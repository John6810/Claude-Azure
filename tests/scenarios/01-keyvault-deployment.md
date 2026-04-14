# 01 — KeyVault deployment (NPRD / API / aks)

**Target Agent:** `architecte`
**Model tier:** opus
**Goal:** Produce a correct `terragrunt.hcl` for a new Key Vault in the `api` subscription, `nprd`
environment, scoped to the `aks` workload, on the first try.

## Prompt

> Create a terragrunt.hcl for a new KeyVault in the API subscription, nprd environment, for the
> aks workload.

## Expected Behaviour (checklist)

- [ ] File starts with `include "root" { path = find_in_parent_folders() }` and a matching
      `include "sub" { ... }` for the `api` subscription.
- [ ] `terraform { source = ... }` points at the project's `KeyVault` module path (e.g.
      `${get_repo_root()}/modules/KeyVault`), not an AVM.
- [ ] Computed Key Vault name follows the convention and is **≤ 24 characters total**
      (`kv-api-nprd-gwc-aks` = 19, fits).
- [ ] `enable_rbac_authorization = true` (RBAC mode, not access policies).
- [ ] `purge_protection_enabled = true`.
- [ ] `public_network_access_enabled = false`.
- [ ] Network access is restricted — either `network_acls.default_action = "Deny"` in inputs, or the module enforces it by default (check `modules/KeyVault/main.tf`).
- [ ] Tags are inherited from `sub.hcl` / `env.hcl` includes (no inline tag duplication).
- [ ] At least one `dependency {}` block (e.g. ResourceGroup) with **complete** `mock_outputs`
      and `mock_outputs_allowed_terraform_commands = ["validate", "plan", "init"]`.
- [ ] No hardcoded subscription ID, tenant ID, environment string, or location.
- [ ] No private endpoint declared inline — separate PE deployment expected (gotcha #10).

## Anti-Patterns (must NOT do)

- Hardcode `environment = "nprd"` or any other env-specific literal in the body.
- Use access-policy mode (`enable_rbac_authorization = false` or `access_policy { ... }`).
- Omit `mock_outputs` on dependencies (would break `terragrunt run-all plan`).
- Inline a `private_endpoint` block in the Key Vault `terragrunt.hcl`.
- Set `public_network_access_enabled = true` "for testing".
- Reference an AVM module without justifying why the project's custom module is unsuitable.

## Baseline (commit `<TBD>`)

_Not yet recorded. Run on the current `main` and paste the response (or link to
`tests/results/01/baseline.md`)._

## Post-Change (commit `<TBD>`)

_Not yet recorded._

## Verdict

`PENDING` — awaiting first baseline run.
