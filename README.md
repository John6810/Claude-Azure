# Claude Code — Azure Landing Zone

Operating an enterprise Azure Landing Zone is repetitive, error-prone, and full of project-specific
tribal knowledge: a 24-character Key Vault limit, a Palo Alto default route hidden in a route table,
a DINE policy that silently mutates your private endpoints. Generic AI assistants do not know any of
this — and a senior cloud architect should not waste cycles re-explaining it on every prompt.

This repository is the **Claude Code configuration** that turns Claude into a domain-aware operator
for a Terragrunt-based Azure Landing Zone (hub-and-spoke, Palo Alto NVA, ALZ policies, WIF pipelines).
It encodes the conventions, gotchas, and decision rules of a production environment so that every
session starts already knowing the rules.

## Why this exists

- **Reduce context cost** — `CLAUDE.md` stays under 60 lines. Specialized knowledge lives in agents,
  rules, and skills loaded only when needed.
- **Avoid hallucinated Terraform** — agents are told to verify against `learn.microsoft.com` and
  `registry.terraform.io` before producing module code.
- **Encode gotchas once** — the 11 production gotchas live in `CLAUDE.md` and are referenced by ID
  in agents, reviews, and tests. No copy-paste, no drift.
- **Pick the right model for the job** — reasoning, execution, and pattern-matching tasks are routed
  to Opus, Sonnet, and Haiku respectively to keep latency and cost under control.

## Agents — by tier

Agents are split by reasoning load. Use the right one and you get faster, cheaper, more accurate output.

### Opus — reasoning, architecture, diagnostics

| Agent | Role |
|-------|------|
| `architecte` | Architecture decisions, terragrunt.hcl creation, deployment order, AVM vs custom arbitration |
| `troubleshooter` | Plan/apply diagnostics, state operations, Azure API errors |
| `policy-manager` | ALZ policies, DINE/Deny analysis, exemptions |
| `upgrade-planner` | Version upgrade impact analysis (read-only) |
| `module-creator` | Terraform module creation (4-file pattern) |

### Sonnet — execution, review, generation

| Agent | Role |
|-------|------|
| `chef-de-projet` | Coordination, scoping — delegates everything, writes no code |
| `cicd-azure-devops` | Azure Pipelines YAML, WIF, approval gates |
| `deployment-reviewer` | Pre-apply audit checklist (read-only) |
| `security-reviewer` | RBAC/PIM, PEs, MIs, KV audit (read-only) |
| `finops` | Cost audit, tags, optimizations (read-only) |
| `documentation` | Per-module README generation |
| `test-creator` | Terraform native tests (`.tftest.hcl`) |
| `k8s-operator` | AKS day-2 ops — kubectl/helm diagnostics, Workload Identity, node pools |

### Haiku — pattern matching, validation

| Agent | Role |
|-------|------|
| `naming-checker` | Naming convention validation (read-only) |
| `network-planner` | IP addressing plan (read-only) |

## Commands

| Command | Description |
|---------|-------------|
| `/new-module {Name}` | Create a Terraform module (4 files) |
| `/new-deployment {path}` | Create a Terragrunt deployment folder |
| `/check-naming {path}` | Verify naming conventions |
| `/add-diag {workload}` | Add Diagnostic Settings |
| `/add-locks {workload}` | Add Resource Locks (CanNotDelete) |
| `/add-rbac {workload}` | Add RBAC Assignments (Entra ID groups + identities) |
| `/generate-docs {Module\|all}` | Generate README documentation |
| `/generate-tests {Module\|all}` | Generate Terraform native tests |
| `/audit-security {path}` | Security audit (RBAC, PE, MI, KV, networking) |
| `/audit-finops {path}` | FinOps audit (tags, costs, optimizations) |
| `/audit-pre-apply {path}` | Pre-apply checklist (mock outputs, naming, routes) |
| `/post-apply {path}` | Post-deployment verification via az CLI |

## Key Design Decisions

- **`CLAUDE.md` < 60 lines.** Loaded on every prompt. Anything that is not strictly required every
  session belongs in an agent, a rule, or a skill — not in the global context.
- **No duplication.** Naming convention is owned by `naming-checker`. Gotchas are owned by `CLAUDE.md`.
  Pipeline rules are owned by `rules/pipeline.md`. Agents reference; they do not repeat.
- **Right model for the job.** Opus for design and diagnosis, Sonnet for code and review, Haiku for
  validation. This keeps cost predictable and avoids over-thinking trivial pattern matches.
- **Path-scoped rules.** `rules/terraform.md` only loads when editing `*.tf` / `*.hcl`.
  `rules/pipeline.md` only loads under `.azure-pipelines/**`. No wasted tokens.
- **Decision frameworks, not opinions.** The `architecte` agent ships explicit matrices
  (Module vs Inline, AVM vs Custom, count vs for_each, dependency vs data, PE module pattern,
  spoke sizing, single vs split state) so recommendations are reproducible.
- **TDD for prompts.** Behaviour changes are validated against scenarios in `tests/scenarios/` and
  diff'd against a baseline. See `CONTRIBUTING.md`.

## Target Architecture

This config assumes — and is opinionated about — the following landing zone shape:

- **Topology:** hub-and-spoke, single region (`gwc` by default), one hub VNet per environment.
- **Egress:** Palo Alto NVA cluster in the hub. Spokes default-route `0.0.0.0/0` to a fixed Palo IP
  (gotcha #5).
- **IaC:** Terragrunt for orchestration, Terraform modules following the 4-file pattern
  (`version.tf` / `variables.tf` / `main.tf` / `output.tf`).
- **Environments:** `prod` and `nprd`, switched via `TG_ENV` env var.
- **Subscriptions:** `mgm`, `con`, `idt`, `sec`, `api` (extend as needed).
- **Governance:** ALZ policies enabled, including DINE for private endpoints (gotcha #10).
- **Pipelines:** Azure DevOps with Workload Identity Federation (WIF), per-environment service
  connections, manual approval gates on `prod`.

## Setup

1. Clone this repo into the root of your Terragrunt landing-zone project (or merge it in).
2. Copy the local settings template:

   ```bash
   cp .claude/settings.local.json.example .claude/settings.local.json
   ```

3. Open the project in Claude Code. `CLAUDE.md` and `.claude/` are picked up automatically.
4. Agent teams require: `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` (already set in `settings.json`).

### What to customise

| File | What to change | Why |
|------|----------------|-----|
| `CLAUDE.md` | Tenant ID, region code, environments, subscription acronyms, IP ranges | Project-specific identity |
| `CLAUDE.md` (gotchas) | Add/remove entries as you discover them | Drift control |
| `.claude/settings.json` | Hooks (e.g. notification command for non-Windows) | OS / team workflow |
| `.claude/settings.local.json` | Personal paths, model overrides | Per-developer, gitignored |
| `.claude/rules/terraform.md` | House style for HCL | Codify your team's conventions |
| `.claude/agents/network-planner.md` | Hub/spoke ranges, Palo IPs | Match your topology |

## Adapting to your own Landing Zone

If you fork this for a different project:

1. **Rewrite `CLAUDE.md` first.** It is the single source of truth for tenant, region, environments,
   subscriptions, and gotchas. Keep it under 60 lines.
2. **Update `naming-checker`** with your naming convention (prefix order, separator, length limits).
3. **Update `network-planner`** with your IP plan (hub CIDR, spoke CIDRs, NVA addresses).
4. **Re-tune `rules/`** to your house style (provider versions, lint rules, module layout).
5. **Re-baseline the scenarios** in `tests/scenarios/` against your version — see `CONTRIBUTING.md`
   for the TDD-for-prompts process.
6. **Strip the gotchas you do not have**, add your own. Reference them by number from
   `deployment-reviewer` and `test-creator`.

## Hooks

| Event | Trigger | Action |
|-------|---------|--------|
| PostToolUse | Edit/Write on `.tf` files | `terraform fmt` |
| Notification | Claude idle | Windows toast notification |

## License

MPL-2.0 — see `LICENSE`.

---

**Built for real-world Azure Landing Zone operations, not demos.**
