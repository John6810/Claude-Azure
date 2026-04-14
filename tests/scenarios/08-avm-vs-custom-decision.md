# 08 — AVM vs Custom decision (Storage Account for TF state)

**Target Agent:** `architecte`
**Model tier:** opus
**Goal:** Produce a reasoned, project-aware recommendation between an AVM and a custom module,
explicitly using the decision matrix from `architecte.md`.

## Prompt

> I need to deploy a Storage Account for Terraform state in the management subscription. Should I
> use an AVM or a custom module?

## Expected Behaviour (checklist)

- [ ] Explicitly references the **AVM vs Custom** decision matrix from `architecte.md`.
- [ ] Evaluates each criterion against this specific case:
      - [ ] Naming convention (custom needed: `st{acr}{env}{region}{workload}` style, no hyphens).
      - [ ] `time_static` immutability (custom).
      - [ ] Composition needs (PE, lifecycle, diagnostic settings) — likely custom.
      - [ ] Provider version flexibility.
      - [ ] Maintenance budget.
- [ ] Gives a **clear recommendation** (custom or AVM) — not "it depends" without resolution.
- [ ] Lists trade-offs in both directions, not just the chosen one.
- [ ] Mentions the operational implications: state Storage Account needs immutability of name,
      blob versioning, soft delete, and likely a private endpoint (separate PE deployment, gotcha #10).
- [ ] If AVM is suggested, addresses the naming-convention gap explicitly (e.g. via `name` override).
- [ ] If custom is suggested, acknowledges what the AVM would have given for free
      (community testing, faster bootstrap).

## Anti-Patterns (must NOT do)

- Answer "it depends" without walking the matrix.
- Recommend AVM without addressing how the project's naming convention will be enforced.
- Recommend a custom module without acknowledging the maintenance cost vs AVM.
- Ignore the specific use case (state Storage Account has stricter requirements than a generic SA).
- Suggest inline resources — Storage Account for state is reused across environments, custom
  module is the project default.

## Baseline (commit `<TBD>`)

_Not yet recorded._

## Post-Change (commit `<TBD>`)

_Not yet recorded._

## Verdict

`PENDING` — awaiting first baseline run.
