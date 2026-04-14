---
description: Terraform and Terragrunt conventions for this Landing Zone
paths:
  - "**/*.tf"
  - "**/*.hcl"
---

# Terraform / Terragrunt Rules

- Provider: azurerm ~> 4.0, terraform >= 1.5.0, time >= 0.9.0
- Always `terraform fmt` after editing .tf files
- Always `terraform validate` before committing
- Never hardcode environment values — use includes (`include.root.inputs.environment`)
- Tags must include `include.root.inputs.common_tags`
- SubnetWithNsg output keys = FULL subnet name (e.g. `snet-api-prod-gwc-nodes`), never short name
- Stack modules: direct resource blocks only, no `module {}` calls
- Mock outputs: declare ALL keys used by dependents + `mock_outputs_allowed_terraform_commands = ["validate", "plan", "init"]`
- Cross-subscription deps: `"${get_repo_root()}/landing-zone/platform/..."` (absolute path)
