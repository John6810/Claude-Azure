---
name: grafana-dashboards
description: >
  Reference for Azure Managed Grafana dashboards — data sources, ARG/KQL/PromQL patterns,
  JSON structure, naming convention. Load on demand when working on Grafana panels.
---

# Grafana Dashboards — Reference

## Data Sources

| Type | Use for | UID pattern |
|------|---------|-------------|
| Azure Monitor | Metrics, Log Analytics (KQL), AzureDiagnostics | `azure-monitor-<env>` |
| Azure Resource Graph | Inventory, drift, cross-sub queries | `azure-monitor-<env>` (same plugin) |
| Prometheus (Managed) | Cluster/node/pod metrics from ama-metrics | `prometheus-<env>` |

**Hardcode UIDs** in dashboard JSON. Template `${datasource}` variables break provisioning and
cross-dashboard linking — never use them.

## ARG Panels

```json
{
  "datasource": { "type": "grafana-azure-monitor-datasource", "uid": "azure-monitor-prod" },
  "queryType": "Azure Resource Graph",
  "subscriptions": ["<sub-id-mgm>", "<sub-id-api>"],
  "azureResourceGraph": { "query": "resources | where type == 'microsoft.keyvault/vaults' | project name, tags" }
}
```

`subscriptions` MUST be an explicit array — omitting it queries none, not all.

## KQL Gotcha (AzureDiagnostics)

DNS fields are dynamic strings and require `_s` suffix (`QueryName_s`, `DnsQuery_s`). Without
`_s` the column is `null`. Same rule for NSG flow (`rule_s`, `direction_s`).

## KQL & PromQL Patterns

```kusto
ContainerLogV2 | where PodNamespace == "ingress" | where LogMessage contains "error" | take 100
AzureNetworkAnalytics_CL | where SubType_s == "FlowLog" | where FlowStatus_s == "D"
Perf | where ObjectName == "K8SNode" and CounterName == "cpuUsageNanoCores" | summarize avg(CounterValue) by Computer, bin(TimeGenerated, 5m)
```
```promql
100 * (1 - avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])))
increase(kube_pod_container_status_restarts_total[1h]) > 0
container_memory_working_set_bytes / on(pod,container) kube_pod_container_resource_limits{resource="memory"}
```

## Panel Skeleton & Naming

```json
{ "id": 1, "title": "...", "type": "timeseries", "gridPos": { "h": 8, "w": 12, "x": 0, "y": 0 },
  "targets": [{ "datasource": { "uid": "prometheus-prod" }, "expr": "..." }] }
```

Dashboards: `{domain}-{env}-{scope}` — e.g. `aks-prod-cluster-overview`, `network-nprd-palo-alto`.
Folder = domain. Tag with `env:<env>` and `owner:<team>`.
