---
title: "How to Monitor and Cap Log Analytics Workspace Ingestion Across Your Azure Tenant"
date: 2025-07-31
---
The Azure Log Analytics Workspace is a powerful tool for collecting and analyzing telemetry data in Azure Monitor. 
However, uncontrolled data ingestion can lead to unexpected costs. In this post, we’ll explore how to use Azure Resource Graph (ARG) to identify Log Analytics Workspaces across your tenant and monitor their ingestion volume, helping you implement capping strategies to stay within budget.

### Why Cap Log Analytics Workspace Ingestion?
Azure charges for Log Analytics Workspace based on the volume of data ingested. Without proper controls, ingestion can spike due to:
- Excessive data from monitored resources
- Misconfigured diagnostic settings
- Unexpected changes in workloads
  
Capping ingestion helps:
- Prevent budget overruns
- Maintain predictable costs

### How to Identify Log Analytics Workspaces Using Azure Resource Graph
Use an Azure Resource Graph query to list all Log Analytics Workspaces and check which of them has capping enabled:

```bash
Resources
| where type == "microsoft.operationalinsights/workspaces"
| extend dailyQuotaGb = properties.workspaceCapping.dailyQuotaGb
| project name, location, dailyQuotaGb, subscriptionId, id
```

> **Note:** records for which 'dailyQuotaGb' column has '-1' value have no cap enabled!

You can also create a Log Search Alert to audit Log Analytics Workspace periodically:

```bash
arg("").Resources
| where type == "microsoft.operationalinsights/workspaces"
| extend dailyQuotaGb = properties.workspaceCapping.dailyQuotaGb
| project name, location, dailyQuotaGb, subscriptionId, id
```

To know  more about Azure Monitor Alerts with ARG queries see [Azure Resource Graph Alerts query quickstart](https://learn.microsoft.com/en-us/azure/governance/resource-graph/alerts-query-quickstart?tabs=azure-resource-graph)

### How To Set Daily Cap in Log Analytics Workspace
In the Azure Portal, go to your instance of Log Analytics Workspace:
- From the left menu, select **Usage and estimated costs**.
- Select **Daily Cap**.
- Select **ON** and set the cap in **GB/day** (the minimum value is 0.023).

![plot](https://github.com/fabiocannas/fabiocannas.github.io/blob/main/_posts/2025-07-31-How_to_Monitor_and_Cap_Log_Analytics_Workspace_Ingestion/2025-07-31-How_to_Monitor_and_Cap_Log_Analytics_Workspace_Ingestion.png?raw=true)

If you prefer to use Azure Resource Manager APIs, here is the link to Log Analytics Workspace's **Create Or Update** documentation:
[Azure Resource Manager - Workspaces - Create Or Update](https://learn.microsoft.com/en-us/rest/api/loganalytics/workspaces/create-or-update?view=rest-loganalytics-2025-02-01)

### Monitor Ingestion Volume Using Log Analytics queries
Create a Log Search Alert To track ingestion volume, by querying the **Usage** table:
```bash
Usage
| where IsBillable
| summarize DataGB = sum(Quantity / 1000)
| where DataGB > 1
```

You can also create a Log Search Alert to notify you when ingestion exceeds a defined limit:
```bash
_LogOperation | where Category =~ "Ingestion" | where Detail contains "OverQuota"
```

Alerts can be configured to run daily and trigger actions like emails or automation.

Capping Log Analytics Workspace ingestion is essential for cost control in Azure Monitor. By combining Azure Resource Graph for discovery and Log Analytics queries for monitoring, you can build a proactive strategy to manage ingestion across your tenant.