---
name: azurerm-v4-migration
description: >
  azurerm v3→v4 breaking changes affecting this project's modules. Load when creating/modifying
  modules, diagnosing unexpected plan diffs, or planning upgrades.
---

# azurerm v4 — Breaking Changes (this project)

## Breaking Changes That Impacted Us

| Resource | Change | Affected module | Mitigation |
|----------|--------|-----------------|------------|
| `azurerm_kubernetes_cluster` | `default_node_pool` restructured; `key_management_service` + VNet Integration not manageable | `aks` | `az aks update` out-of-band + `lifecycle { ignore_changes }` (gotcha #9) |
| `azurerm_key_vault` | `enable_rbac_authorization` removed (RBAC is default) | `kv` / `KeyVaultStack` | Drop the variable + input |
| `azurerm_private_endpoint` | `private_dns_zone_group` created by ALZ DINE | `pe` | `lifecycle { ignore_changes = [private_dns_zone_group] }` (gotcha #10) |
| `azurerm_storage_account` | `allow_blob_public_access` → `allow_nested_items_to_be_public` | `st` | Rename attribute |
| `azurerm_*_association` | Several removed, inlined in parent (e.g. subnet↔NSG) | `subnet`, `rt` | Inline `network_security_group_id` / `route_table_id` |
| Provider `features {}` | Simplified (several nested blocks removed) | all `version.tf` | Keep only blocks still supported |

## `lifecycle { ignore_changes }` Patterns

| Resource | Attributes | Reason |
|----------|------------|--------|
| `azurerm_kubernetes_cluster` | `key_management_service`, `azure_active_directory_role_based_access_control`, `vnet_integration` | Managed out-of-band via `az aks update` |
| `azurerm_private_endpoint` | `private_dns_zone_group` | Created by ALZ DINE Deploy-Private-DNS-* |
| Resources with DINE diag settings | `diagnostic_setting` child | DINE policy owns them |

## Version Constraints

- All modules: `azurerm ~> 4.0` in `version.tf`
- `azapi ~> 2.0` used **only** in the AKS module for capabilities not yet in azurerm
