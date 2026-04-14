---
name: kql-patterns
description: >
  KQL query patterns for Log Analytics, Azure Resource Graph, and ADX. Load when writing
  KQL queries, DCR transformKql filters, or Grafana KQL panels.
---

# KQL Patterns — Reference

## Engines

| Engine | Tables | Notes |
|--------|--------|-------|
| Log Analytics (LAW) | `AzureDiagnostics`, `ContainerLogV2`, `Perf`, `SigninLogs` | `ago()` + time filters required |
| Azure Resource Graph | `resources`, `resourcecontainers`, `advisorresources` | No time filter, no `ago()` |
| ADX (FinOps Hub) | `CostDetails`, `BillingAccount` | FOCUS schema columns |

## AzureDiagnostics Field Suffixes (#1 gotcha)

Dynamic columns require a type suffix or they return `null` — strings `_s`
(`QueryName_s`, `DnsQuery_s`, `rule_s`, `direction_s`), numbers `_d`, booleans `_b`.
Always verify the suffix in the LAW schema explorer before shipping a query.

## DCR transformKql

Filter pattern (reduce volume, keep security-relevant failures):

```kusto
source | where Category != "NonInteractiveUserSignInLogs" or ResultType != "0"
```

Basic Logs tier reduces cost on high-volume tables — limitations: no joins, 8-day retention.

## Common Queries

```kusto
// Pod logs
ContainerLogV2 | where PodNamespace == "<ns>" | where LogMessage contains "error" | take 100

// NSG denied flows
AzureNetworkAnalytics_CL | where SubType_s == "FlowLog" and FlowStatus_s == "D"
| summarize count() by SrcIP_s, DestIP_s, DestPort_d

// Failed sign-ins by app + CA failures
SigninLogs | where ResultType != 0
| summarize Failures=count() by AppDisplayName, ConditionalAccessStatus

// AKS NotReady nodes
KubeNodeInventory | where Status !contains "Ready"
| summarize arg_max(TimeGenerated, *) by Computer | project Computer, Status, ClusterName

// Inventory by RG (ARG)
resources | summarize count() by resourceGroup, type | order by count_ desc

// Defender alerts
SecurityAlert | where AlertSeverity in ("High","Medium") and Status != "Resolved"
| project TimeGenerated, AlertSeverity, AlertName, CompromisedEntity
```
