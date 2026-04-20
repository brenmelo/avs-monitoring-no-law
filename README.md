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

This workbook intentionally trades several capabilities for zero data cost. If you need any of the following, a LAW-based workbook is the right choice.

### Visualization limitations

- **No pie charts for live utilization metrics.** Azure Workbooks' free metrics widget (`MetricsItem/2.0`) only renders line, area, bar, and scatter — not pie. Pie/donut visualizations require a `KqlItem` Resource Graph or LAW query, but the free metrics API can't be queried with KQL. The Overview tab uses pies for inventory dimensions (hosts by cluster, SKU, region) instead.
- **No "Used vs. Free" donut math.** Computing `Used / Capacity * 100` across two metrics into a single pie slice requires KQL against `AzureMetrics` in LAW. Without LAW you get separate Used and Capacity line charts on the Storage tab.
- **No cross-resource aggregation in a single chart.** Free metrics charts are per-resource and split by dimension; you can't `summarize` across multiple AVS instances in one visual without LAW.

### Data & retention limitations

- **No VMware syslog visibility** (vCenter, ESXi, vSAN, NSX) — requires the `AVSSyslog` table in LAW
- **No NSX Distributed Firewall packet logs / audit trail** — requires the `allLogs` diagnostic category routed to LAW
- **No vSAN component-level health, capacity tier breakdown, or dedupe ratio history** — only exposed via syslog → LAW
- **Metric history capped at 93 days** — Azure Monitor platform metrics retention; LAW can be configured up to 12 years
- **Single chart capped at 30 days** — pan the time picker to view older windows; LAW has no per-chart cap
- **No 1-minute granularity beyond 30 days** — the metrics API down-samples; LAW preserves your ingested resolution
- **No long-term trending or capacity forecasting** — needs LAW retention + `series_decompose_forecast()` KQL

### Alerting & analytics limitations

- **No log-search alerts** that combine metrics with logs (e.g., "CPU > 80% AND audit log shows config change") — log alerts require LAW
- **No multi-resource KQL alerts** correlating events across multiple AVS instances
- **No anomaly detection on metrics** — `series_decompose_anomalies()` and ML KQL functions need LAW
- **No custom KPIs from joined data** (e.g., joining metrics with Azure Activity events) — needs LAW
- **No forensic / post-incident investigation of audit events** — the Azure Activity log is free for 90 days but can't be retained or queried with KQL beyond that without LAW

### Security & compliance limitations

- **No security analytics on AVS** (Microsoft Sentinel, custom detections) — Sentinel sits on top of LAW
- **No long-term audit retention for compliance** (SOC 2, PCI, HIPAA) — typically requires 1+ year LAW retention
- **No NSX firewall rule-hit telemetry** for zero-trust validation — requires syslog → LAW

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
