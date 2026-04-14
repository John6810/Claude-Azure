# Add Resource Locks

Create `locks-{workload}` deployment for `$ARGUMENTS`.

## Instructions

1. Read reference: `landing-zone/corporate/apimanager/locks-api/terragrunt.hcl`
2. Identify Resource Groups to lock
3. Create terragrunt.hcl with `modules/ResourceLock`

## Key Rules

- `enable_locks = true` in production, `false` only for `terraform destroy`
- `CanNotDelete` on RG locks ALL resources inside
- Recommended: network watcher RG, KV RG, AKS RG, ACR RG
