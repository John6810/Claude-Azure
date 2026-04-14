# Post-Apply Verification

Verify resources deployed in `$ARGUMENTS` are correctly configured in Azure.

## Process

1. Read terragrunt.hcl files to identify deployed resources
2. Verify each via `az` CLI (read-only):

| Resource | Key Checks |
|----------|-----------|
| RG | Tags (Environment, CostCenter) |
| Key Vault | RBAC auth, purge protection, public access disabled |
| AKS | Private cluster, RBAC, OIDC, Workload Identity |
| ACR | Public access disabled, admin disabled |
| PEs | All in `Approved` status |
| Route Table | `0.0.0.0/0` → Palo ILB |

3. Produce report: Compliant ✅ / Discrepancies ❌ / Not Found ⚠️

Requires `az login` or WIF credentials. Use `--subscription` if needed.
