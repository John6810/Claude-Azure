# Azure Landing Zone — Project Context

Terragrunt-based Azure Landing Zone (hub-and-spoke, Palo Alto NVA).
Region: Germany West Central (`gwc`). Tenant: `090a1bf9-58cc-49fa-8a9e-3f7b0a100fa9`.
Environments: `prod` (10.238.0.0/16) / `nprd` (10.239.0.0/16).
Subscriptions: `mgm`, `con`, `idt`, `sec`, `api`.
Language: English.

## Build & Test

```bash
export TG_ENV="nprd"          # or "prod" (default: prod)
terragrunt init               # Initialize
terragrunt plan               # Plan single module
terragrunt apply              # Apply single module
terragrunt run-all apply      # Apply all (ordered by deps)
terraform validate && terraform fmt -check
```

## Naming

Pattern: `{prefix}-{acr}-{env}-{region}-{workload}` — e.g. `rg-api-prod-gwc-aks`.
ACR exception: `cr{acr}{env}{region}{workload}` (no hyphens).
Key Vault max 24 chars. `kv-api-prod-gwc-` = 16 chars → workload max 8.

## Terragrunt Pattern

Every deployment: `include "root"` + `include "sub"` + `terraform { source }` + `dependency` with `mock_outputs` + `inputs` from includes.
Environment switch: `export TG_ENV="nprd"` or `export TG_ENV="prod"`.

## Critical Gotchas — ALWAYS VERIFY

1. **SubnetWithNsg keys** = full subnet name, NOT short name
2. **Stack modules** (KeyVaultStack, AzureMonitorWorkspace) — no `module {}` calls, use direct resource blocks
3. **Key Vault name** max 24 characters total
4. **Mock outputs** — declare ALL keys used by dependent modules, include `mock_outputs_allowed_terraform_commands`
5. **Palo Alto default route** — all spokes: `0.0.0.0/0` → PROD `10.238.200.36` / NPRD `10.239.200.36`
6. **Cross-sub deps** — absolute path: `"${get_repo_root()}/landing-zone/platform/..."`
7. **AKS private_dns_zone_id** = `"None"` (ALZ policy manages DNS in connectivity)
8. **Palo Alto custom roles** — name must include `${local.prefix}-${var.workload}` to avoid env conflicts
9. **azurerm v4 + AKS** — KMS Private and VNet Integration not manageable via provider → `az aks update` + `lifecycle { ignore_changes }`
10. **ALZ DINE + PEs** — Deploy-Private-DNS-* auto-creates privateDnsZoneGroup → PE module needs `lifecycle { ignore_changes = [private_dns_zone_group] }`
11. **D2s_v5 + Ephemeral OS** — NOT supported (insufficient temp storage) → use D4s_v5+ or D2ds_v5

## Module Pattern

All modules: `version.tf` + `variables.tf` + `main.tf` + `output.tf`.
Naming: `time_static` → `locals { computed_name, name }` → resource.
Standard variables: `name` (nullable override), `subscription_acronym`, `environment`, `region_code`, `location`, `workload`, `tags`.

## Key References

- `_global/networks_corp.hcl` — IP addressing (source of truth)
- `_global/sub_platform.hcl` / `_global/sub_corp.hcl` — subscription IDs
- `config/prod.hcl` / `config/nprd.hcl` — environment tags
- `DEPLOYMENT-GUIDE.md` — full deployment procedure (French)
- `landing-zone/platform/convention-entra-id-rbac.md` — RBAC naming convention
