# Add RBAC Assignments

Create `rbac-{workload}` deployment for `$ARGUMENTS`.

## Instructions

1. Read reference: `landing-zone/corporate/apimanager/rbac-api/terragrunt.hcl`
2. Read `landing-zone/platform/convention-entra-id-rbac.md` for naming
3. Create terragrunt.hcl with `modules/RbacAssignments`

## PIM Matrix

| Role | Prod | Nprd |
|------|------|------|
| Reader | Permanent (RBAC) | Permanent (RBAC) |
| Contributor | PIM | Permanent (RBAC) |
| Owner/Admin | PIM | Permanent (RBAC) |

Use `local.admin_prefix` to switch: `GRP_AZ_PIM_` in prod, `GRP_AZ_RBAC_` in nprd.
Entra ID group pattern: `GRP_AZ_{Type}_{Scope}_{Resource}_{Env}_{Role}`.
CI/CD SPs: direct assignments, not via groups.
