# AVS Monitoring Workbook (No Log Analytics Required)

An Azure Workbook for monitoring **Azure VMware Solution (AVS)** private clouds that runs entirely on **free** Azure data sources. No Log Analytics Workspace, no diagnostic settings, no ingestion charges.

## Why this exists

Most AVS monitoring workbooks require sending platform metrics and VMware syslogs to a Log Analytics Workspace via diagnostic settings, which incurs ingestion + retention costs. This workbook uses only free data sources to give you visibility into your AVS estate at zero data cost.

## Data sources (all free)

| Source | What it provides | Retention |
|---|---|---|
| Azure Resource Graph | Inventory, SKU, host counts, networking, datastore configuration | Live |
| Azure Monitor platform metrics API | CPU, memory, vSAN, datastore utilization | 93 days |
| Azure Alerts Management | Active and recent alerts on AVS resources | Per alert config |

## What's included

The workbook ships with 5 tabs:

| Tab | Content |
|---|---|
| **Overview** | KPI tiles (private clouds, hosts, clusters, active alerts), private cloud health table, CPU / Memory / vSAN trend charts |
| **Performance** | CPU and memory average + peak, effective memory, memory overhead — split by cluster |
| **Storage** | Datastore Used %, Used GB, Capacity GB charts plus external datastore inventory (ANF, Elastic SAN, Pure Cloud Block Store) |
| **Capacity & Inventory** | Cluster table with SKU coloring, SKU distribution, hosts by region |
| **Alerts** | Severity tiles, time-filtered alert table, deploy links for ESLZ alert templates |

## Deploy

### Option 1: Azure Portal (manual)

1. Open the [Azure Portal](https://portal.azure.com) → **Azure Monitor** → **Workbooks** → **New**
2. Click the `</>` button to open the **Advanced Editor**
3. Switch the Template Type from `Gallery Template` to `Workbook`
4. Paste the contents of [`AVS-Metrics-Workbook-NoLAW.workbook.json`](./AVS-Metrics-Workbook-NoLAW.workbook.json)
5. Click **Apply** and then **Save**

### Option 2: ARM template

Coming soon. For now, use the manual deployment above.

## Limitations vs. LAW-based workbooks

This workbook intentionally trades a few capabilities for zero data cost. If you need any of the following, a LAW-based workbook is the right choice:

- **VMware syslog visibility** (vCenter, ESXi, vSAN, NSX) — requires `AVSSyslog` table in LAW
- **NSX Distributed Firewall logs and audit trail** — requires `allLogs` diagnostic category to LAW
- **Metric history beyond 93 days** — platform metrics API is capped at 93 days
- **Single chart spanning more than 30 days** — chart-level cap; pan the time picker to view older windows
- **Log-search alerts that combine metrics with logs** — requires LAW

## Recommended companion

Pair this workbook with the AVS alert templates from the Enterprise Scale for AVS repo:

- [Deploy AVS metric alerts](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2FEnterprise-Scale-for-AVS%2Fmain%2FBrownField%2FMonitoring%2FAVS-Utilization-Alerts%2FARM%2FAVSMonitor.deploy.json)
- [Deploy AVS service health alerts](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2FEnterprise-Scale-for-AVS%2Fmain%2FBrownField%2FMonitoring%2FAVS-Service-Health%2FARM%2FAVSServiceHealth.deploy.json)

Both deploy alert rules directly against platform metrics, so they complement this workbook without adding any LAW dependency.

## References

- [Azure Monitor platform metrics retention (93 days, free)](https://learn.microsoft.com/en-us/azure/azure-monitor/metrics/data-platform-metrics#retention-of-metrics)
- [Configure VMware syslogs for AVS](https://learn.microsoft.com/en-us/azure/azure-vmware/configure-vmware-syslogs)
- [Azure Monitor Logs cost calculations](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/cost-logs)
- [Azure VMware Solution metrics reference](https://learn.microsoft.com/en-us/azure/azure-vmware/concepts-monitor-repair-private-cloud)

## License

MIT
