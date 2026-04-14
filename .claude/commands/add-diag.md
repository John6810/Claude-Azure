# Add Diagnostic Settings

Create `diag-{workload}` deployment for `$ARGUMENTS`.

## Instructions

1. Read reference: `landing-zone/corporate/apimanager/diag-api/terragrunt.hcl`
2. Read `modules/DiagnosticSettings/variables.tf` for input structure
3. Identify resources to monitor in target workload
4. Create `landing-zone/{path}/diag-{workload}/terragrunt.hcl`

## Log Categories

| Resource | Logs | Metrics |
|----------|------|---------|
| VNet | VMProtectionAlerts | AllMetrics |
| NSG | NetworkSecurityGroupEvent, NetworkSecurityGroupRuleCounter | — |
| Key Vault | AuditEvent, AzurePolicyEvaluationDetails | AllMetrics |
| AKS | kube-apiserver, kube-audit-admin, kube-controller-manager, kube-scheduler, cluster-autoscaler, guard | AllMetrics |
| Storage | StorageRead, StorageWrite, StorageDelete | AllMetrics, Transaction |

LAW is centralized in Management sub — use cross-sub dependency to `alz-management`.
