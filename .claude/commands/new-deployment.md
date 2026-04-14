# Create a New Terragrunt Deployment

Create a deployment folder at `landing-zone/$ARGUMENTS`.

## Instructions

1. Read a similar existing deployment for context (e.g. `landing-zone/corporate/apimanager/rg-aks/`)
2. Read the subscription file (`corporate.hcl`, `connectivity.hcl`, etc.) for available locals
3. Read `_global/networks_corp.hcl` if subnets/IPs needed

## Template

```hcl
include "root" {
  path   = find_in_parent_folders("root.hcl")
  expose = true
}
include "sub" {
  path   = find_in_parent_folders("{subscription}.hcl")
  expose = true
}
terraform {
  source = "${get_repo_root()}/modules/{ModuleName}"
}
dependency "{name}" {
  config_path = "../{folder}"
  mock_outputs = { id = "/subscriptions/.../mock", name = "mock" }
  mock_outputs_allowed_terraform_commands = ["validate", "plan", "init"]
}
inputs = {
  subscription_acronym = include.sub.locals.subscription_acronym
  environment          = include.root.inputs.environment
  region_code          = include.root.inputs.region_code
  location             = include.root.inputs.location
  workload             = "{workload}"
  tags = merge(include.root.inputs.common_tags, {
    Subscription = include.sub.locals.subscription_name
    Workload     = "{description}"
  })
}
```

Refer to CLAUDE.md gotchas for SubnetWithNsg keys, cross-sub deps, mock_outputs.
