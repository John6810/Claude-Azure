---
description: Azure DevOps Pipeline conventions for this Landing Zone
paths:
  - ".azure-pipelines/**"
  - "**/*pipeline*.yml"
---

# Azure Pipelines Rules

- WIF only — no secrets stored, OIDC token exchange
- `ARM_*` variables must be passed as `env:` on each script step (not inherited)
- `TG_ENV` must be exported before every terragrunt command
- `--non-interactive` mandatory in CI
- `-detailed-exitcode` on plan: exit 2 = changes (normal), exit 1 = error
- Deployment jobs required for approval gates (`jobs.deployment` with `environment:`)
- `fetchDepth: 0` on checkout for Terragrunt git operations
- `persistCredentials: false` on checkout to limit token exposure
- Binary caching via `Cache@2` with key `tools | TF_VERSION | TG_VERSION`
- Deployment folder is `landing-zone/`, NOT `live/`
