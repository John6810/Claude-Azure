# Contributing

Changes to agents, rules, commands, or `CLAUDE.md` change Claude's behaviour. Behaviour is hard
to review by reading a diff — you have to run the prompts and compare the output. This guide
describes the process we use to do that reliably.

## TDD for Prompts

Treat every behavioural change like a code change with tests. The flow is:

```
BASELINE  →  CHANGE  →  RETEST  →  COMPARE  →  DOCUMENT
```

1. **BASELINE.** Before you change anything, pick the relevant scenarios in `tests/scenarios/` and
   run them against the current `main`. Save the responses in `tests/results/<scenario-id>/baseline.md`.
2. **CHANGE.** Make your edit to the agent / rule / command / `CLAUDE.md`. Keep the diff small and
   single-purpose.
3. **RETEST.** Re-run the same scenarios in a fresh Claude Code session (no carry-over context).
   Save responses in `tests/results/<scenario-id>/post-change.md`.
4. **COMPARE.** For each scenario, walk through the Expected Behaviour checklist and the
   Anti-Patterns list. Did the change improve the targeted metric without regressing anything else?
5. **DOCUMENT.** Update the scenario file's `Baseline` and `Post-Change` blocks, set a `Verdict`
   (`PASS` / `REGRESSION` / `MIXED`), and reference the commit hash.

Do not merge a behavioural change without at least one updated scenario.

## Starter Test Scenarios

The eight scenarios below exercise the core agents. New scenarios go in `tests/scenarios/` using
the `NN-short-name.md` convention.

| #  | Target Agent           | What it exercises                                       |
|----|------------------------|---------------------------------------------------------|
| 01 | `architecte`           | KeyVault deployment from scratch                        |
| 02 | `module-creator`       | New ACR module (4-file pattern)                         |
| 03 | `deployment-reviewer`  | Review a terragrunt.hcl with deliberate errors         |
| 04 | `network-planner`      | IP plan for an AKS spoke                                |
| 05 | `troubleshooter`       | Diagnose `Unsupported attribute` error                  |
| 06 | `test-creator`         | Generate `.tftest.hcl` for KeyVault module              |
| 07 | `cicd-azure-devops`    | Add a new workload to the pipeline                      |
| 08 | `architecte`           | AVM vs custom module decision                           |

Scenarios 01 and 08 ship as fully fleshed-out examples. The others should be added as the relevant
agents are reviewed.

## Scenario File Format

Every scenario follows the same structure so that diffs across runs are mechanical to compare.

```markdown
# NN — Short Name

**Target Agent:** `<agent-name>`
**Model tier:** opus | sonnet | haiku
**Goal:** one sentence describing the intended capability.

## Prompt

> The exact prompt to paste into Claude Code. No paraphrasing.

## Expected Behaviour (checklist)

- [ ] Concrete, verifiable item #1
- [ ] Concrete, verifiable item #2
- [ ] ...

## Anti-Patterns (must NOT do)

- Do not <thing>.
- Do not <thing>.

## Baseline (commit `<hash>`)

Summary of what Claude produced on `main` before the change. Paste the response or link to
`tests/results/NN/baseline.md`. Note any checklist items missed and any anti-patterns triggered.

## Post-Change (commit `<hash>`)

Same format. Highlight what changed, both improvements and regressions.

## Verdict

`PASS` / `REGRESSION` / `MIXED` — with a one-line reason.
```

## Writing Guidelines

These keep agents and rules cheap, accurate, and mergeable.

### Imperative voice

Write instructions as commands. `Verify naming length before writing the file.` not
`The agent should probably check naming length.` Claude follows imperative instructions more
reliably than descriptive ones.

### Concrete examples beat abstract advice

`KV name max 24 chars — 'kv-api-prod-gwc-' = 16 chars → workload max 8.` is better than
`Be careful with Key Vault name limits.` Show, then explain.

### No duplication

Every fact has one home:

- Naming convention → `naming-checker.md`
- Gotchas → `CLAUDE.md` (numbered)
- Pipeline rules → `rules/pipeline.md`
- Deployment order → `architecte.md`

If you find yourself copy-pasting between agents, refactor: extract to the canonical home and have
the others reference by name or by gotcha number.

### Token budgets

Keep things small. Long prompts cost money on every turn and bury the important parts.

| File type           | Target size      | Hard cap        |
|---------------------|------------------|-----------------|
| `CLAUDE.md`         | ~50 lines        | 60 lines        |
| Agent definition    | 100–200 lines    | 250 lines       |
| Rule file           | 30–50 lines      | 80 lines        |
| Command             | 20–40 lines      | 60 lines        |

If an agent grows past its cap, split it or push detail into a referenced rule/skill.

### Frontmatter is contract

`name`, `description`, `model`, and `tools` in the YAML frontmatter define when the agent is
auto-invoked and what it can do. Do not change them casually — a change to `description` will
re-route work between agents.

## Adding a Gotcha

Production gotchas have to flow through three places consistently. When you discover a new one:

1. **Add it to `CLAUDE.md`** in the `## Critical Gotchas — ALWAYS VERIFY` section. Use the next
   number. Keep the entry to one or two lines. This is the canonical home.
2. **Reference it from `deployment-reviewer.md`** so pre-apply audits catch it. Reference by
   number; do not copy the description.
3. **Add a check to `test-creator.md`** under `### Gotcha-Specific Tests` so the relevant module
   gets a `.tftest.hcl` block that fails if the gotcha is reintroduced.
4. **Add or update a scenario** in `tests/scenarios/` if the gotcha changes how an existing
   scenario should be answered. Re-baseline.

If a gotcha becomes obsolete (provider fix, ALZ policy change), remove it from `CLAUDE.md` and
keep the number — do not renumber, it would invalidate existing references.

## Review Checklist

Before opening a PR, walk through:

- [ ] `CLAUDE.md` still under 60 lines.
- [ ] No fact duplicated across files (naming, gotchas, deployment order, pipeline rules).
- [ ] Frontmatter (`name`, `description`, `model`, `tools`) unchanged unless the change is
      explicitly about routing.
- [ ] Token budgets respected (see table above).
- [ ] Imperative voice; concrete examples for non-obvious instructions.
- [ ] At least one scenario updated with `Baseline` and `Post-Change` results.
- [ ] If a new gotcha was added: `CLAUDE.md` + `deployment-reviewer.md` + `test-creator.md` all
      updated coherently.
- [ ] No secrets, internal hostnames, or non-project IDs committed. `CLAUDE.md` intentionally contains the project tenant ID and subscription acronyms — these are not secrets.
- [ ] `terraform fmt` clean on any `.tf` files touched.
