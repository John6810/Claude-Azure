---
name: private-endpoint-patterns
description: >
  Private Endpoint deployment patterns — subresource names, DNS zone names, cross-sub DNS
  wiring, lifecycle gotcha. Load when creating or reviewing PE deployments.
---

# Private Endpoint — Patterns

## Subresource Names & Private DNS Zones

| Service | Subresource | Private DNS zone |
|---------|-------------|------------------|
| Key Vault | `vault` | `privatelink.vaultcore.azure.net` |
| ACR | `registry` | `privatelink.azurecr.io` |
| Storage — blob | `blob` | `privatelink.blob.core.windows.net` |
| Storage — file | `file` | `privatelink.file.core.windows.net` |
| Storage — table | `table` | `privatelink.table.core.windows.net` |
| Storage — queue | `queue` | `privatelink.queue.core.windows.net` |
| AKS API server | `management` | `privatelink.germanywestcentral.azmk8s.io` |
| PostgreSQL Flexible | `postgresqlServer` | `privatelink.postgres.database.azure.com` |
| Log Analytics (AMPLS) | `azuremonitor` | `privatelink.oms.opinsights.azure.com` (+ 4 others) |
| Cosmos DB | `Sql` | `privatelink.documents.azure.com` |

## Cross-Sub DNS Wiring

- Private DNS Zones live **centralized** in the `connectivity` subscription.
- PE deployment in a workload sub needs a cross-sub `dependency {}` on the DNS zone
  in connectivity (absolute path — gotcha #6).
- ALZ DINE `Deploy-Private-DNS-*` auto-creates the `privateDnsZoneGroup` child
  resource → **gotcha #10**: PE module MUST declare
  `lifecycle { ignore_changes = [private_dns_zone_group] }`.

## Naming

Pattern: `pep-{acr}-{env}-{region}-{service}-{workload}` — e.g. `pep-api-prod-gwc-acr-001`.
One PE per subresource type per target resource.

## Approval Flow

- Auto-approved when PE and target resource are in the same tenant.
- Manual approval required for cross-tenant PEs (rare in this project).
