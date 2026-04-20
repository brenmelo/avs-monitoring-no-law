# AVS Monitoring Workbook (No Log Analytics Required)

An Azure Workbook for monitoring **Azure VMware Solution (AVS)** private clouds that runs entirely on **free** Azure data sources. No Log Analytics Workspace, no diagnostic settings, no ingestion charges.

## Why this exists

Most AVS monitoring workbooks require sending platform metrics and VMware syslogs to a Log Analytics Workspace via diagnostic settings, which incurs ingestion + retention costs. This workbook uses only free data sources to give you visibility into your AVS estate at zero data cost.

## Data sources (all free)

| Source | What it provides | Retention |
|---|---|---|
| Azure Resource Graph | Inventory, SKU, host counts, networking, datastore configuration, Arc-enabled VMware vSphere | Live |
| Azure Monitor platform metrics API | All supported AVS cluster + datastore metrics | 93 days |
| Azure Alerts Management | Active and recent alerts on AVS resources | Per alert config |
| Azure Advisor | Cost, reliability, and performance recommendations | Live |

## Supported metrics coverage

Every metric documented in the [AVS supported metrics reference](https://learn.microsoft.com/en-us/azure/azure-monitor/reference/supported-metrics/microsoft-avs-privateclouds-metrics) is charted:

**Cluster (current):** `CpuUsageAverage`, `MemUsageAverage`, `MemOverheadAverage`, `ClusterSummaryEffectiveMemory`, `ClusterSummaryTotalMemCapacityMB`
**Cluster (legacy):** `EffectiveCpuAverage`, `EffectiveMemAverage`, `OverheadAverage`, `TotalMbAverage`, `UsageAverage`
**Datastore (current):** `DiskUsedPercentage`, `DiskUsedLatest`, `DiskCapacityLatest`
**Datastore (legacy):** `UsedLatest`, `CapacityLatest`

All charts are split by `clustername` or `dsname` dimension, with average + peak views where useful.

## What's included

The workbook ships with 9 tabs:

| Tab | Content |
|---|---|
| **Overview** | KPI tiles (private clouds, hosts, clusters, active alerts), private cloud health table, CPU / Memory / Datastore trend charts |
| **CPU** | All CPU metrics (current + legacy), average + peak, split by cluster |
| **Memory** | All memory metrics: usage %, effective memory, total capacity, overhead — current + legacy variants |
| **Storage** | All datastore metrics (current + legacy), vSAN capacity estimate, external datastore inventory (ANF, Elastic SAN, Pure) |
| **Networking** | ExpressRoute Circuits, GlobalReach Connections, ExpressRoute Authorizations |
| **Arc Resources** | Arc-enabled VMware vSphere VMs, OS distribution, guest agent status, Azure extensions inventory |
| **Capacity & Inventory** | Cluster inventory with SKU coloring, SKU distribution, hosts by region |
| **Alerts & Advisor** | Severity tiles, time-filtered alert table, ESLZ alert template deploy links, Azure Advisor recommendations |
| **AVS Map** | Geographic distribution heat-map of AVS Private Cloud instances |

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
