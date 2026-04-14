---
name: monitoring-stack
description: >
  Observability stack architecture — data flow from AKS through DCR/AMW to Grafana, plus AMBA
  alerts, Defender, and diagnostic settings. Load when working on monitoring, alerts, or
  diagnostic settings.
---

# Monitoring Stack — Reference

## Data Flow (Prometheus)

```
AKS (ama-metrics agent)
  → DCR (dcr-{acr}-{env}-{region}-prometheus)
  → Azure Monitor Workspace (amw-mgm-{env}-gwc-01)
  → Recording Rules (Node + K8s rule groups)
  → Grafana (amg-mgm-{env}-01) via azure_monitor_workspace_integrations
```

## Component Inventory

| Component | Name pattern | Sub | Purpose |
|-----------|--------------|-----|---------|
| LAW | `law-mgm-{env}-gwc-01` | mgm | Centralized logs |
| AMW | `amw-mgm-{env}-gwc-01` | mgm | Prometheus metrics |
| Grafana | `amg-mgm-{env}-01` | mgm | Dashboards |
| DCR | `dcr-{acr}-{env}-{region}-prometheus` | per-workload | AKS metric collection |
| AMBA | alert rules at MG level | mgmt group | ALZ baseline alerts |
| AMPLS | Azure Monitor Private Link Scope | mgm | Private-only ingestion |
| Container Insights | AKS `monitor_metrics {}` block | per-AKS | Logs → LAW |
| Defender for Containers | `azurerm_security_center_subscription_pricing` | per-sub | Runtime + posture |

## Diagnostic Settings (target = centralized LAW in mgm, cross-sub dep)

| Resource | Log categories | Metrics |
|----------|----------------|---------|
| VNet | `VMProtectionAlerts` | `AllMetrics` |
| NSG | `NetworkSecurityGroupEvent`, `NetworkSecurityGroupRuleCounter` | — |
| Key Vault | `AuditEvent`, `AzurePolicyEvaluationDetails` | `AllMetrics` |
| AKS | `kube-apiserver`, `kube-audit-admin`, `kube-controller-manager`, `kube-scheduler`, `cluster-autoscaler`, `guard` | `AllMetrics` |
| Storage | `StorageRead`, `StorageWrite`, `StorageDelete` | `AllMetrics`, `Transaction` |

## Key Learnings

- **AMPLS + Palo Alto `azure-advanced-metrics`:** plugin may not support Private Link — NAT Gateway fallback required for egress.
- **NonInteractiveUserSignInLogs:** high volume — filter via DCR `transformKql` + Basic Logs tier (see `kql-patterns`).
- **Container Insights vs Managed Prometheus:** CI owns logs (`ContainerLogV2`); Managed Prometheus owns metrics (`ama-metrics`). Enable both.
