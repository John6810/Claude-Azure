---
name: entra-id-rbac
description: >
  Entra ID RBAC and PIM conventions — group naming, role definitions, PIM matrix, CI/CD SP
  patterns. Load when creating RBAC assignments or auditing access.
---

# Entra ID RBAC — Reference

Full source of truth: `landing-zone/platform/convention-entra-id-rbac.md`.

## Group Naming

Pattern: `GRP_AZ_{Type}_{Scope}_{Resource}_{Env}_{Role}`

- **Type:** `RBAC` (permanent) or `PIM` (just-in-time)
- **Scope:** `MG` | `SUB` | `RG`
- Max 63 characters, **underscore delimiter only**
- Examples:
  - `GRP_AZ_PIM_RG_AksApi_Prod_AKSClusterAdmin`
  - `GRP_AZ_RBAC_RG_KvApi_Nprd_KVAdmin`

## PIM Matrix

| Env | Reader | Contributor / Owner / Admin |
|-----|--------|-----------------------------|
| Prod | permanent RBAC | PIM (just-in-time) |
| Nprd | permanent RBAC | permanent RBAC |

`local.admin_prefix` in `terragrunt.hcl` switches between `GRP_AZ_PIM_` and `GRP_AZ_RBAC_`
based on `TG_ENV`.

## Common Role Definitions

| Role | Typical scope | Use case |
|------|---------------|----------|
| Azure Kubernetes Service RBAC Cluster Admin | RG | AKS admin |
| Azure Kubernetes Service Cluster User Role | RG | AKS read / `kubectl exec` |
| Key Vault Administrator | RG | KV full access |
| Key Vault Secrets User | RG / KV | Workload identity secret read |
| AcrPull | ACR | AKS kubelet image pull |
| Storage Blob Data Contributor | Storage Account | tfstate, FinOps exports |
| Contributor | sub / RG | General resource management |
| User Access Administrator | sub | CI/CD SP for RBAC assignments |
| Managed Identity Operator | RG | AKS control plane on node pool RG |

## CI/CD Service Principals

- **Direct role assignments** — never via Entra groups.
- Minimum: `Contributor` + `User Access Administrator` on target subscription.
- Storage: `Storage Blob Data Contributor` on the tfstate Storage Account.
- **No PIM for SPs** — permanent assignments only (PIM is incompatible with automation).
