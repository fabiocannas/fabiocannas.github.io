---
title: "Stopping Azure Application Gateway to save costs: a quick win for your Azure bill."
date: 2025-08-18
---
![Azure Application Gateway](https://learn.microsoft.com/en-us/azure/application-gateway/media/application-gateway-url-route-overview/figure1-720.png)

*[What is Azure Application Gateway?](https://learn.microsoft.com/en-us/azure/application-gateway/overview)*

Azure Application Gateway is a powerful load balancing service, but it can be as well one of those services that quietly accumulates costs even when you are not actively using it. If you are running development environments, proof-of-concepts, or seasonal applications, stopping your Application Gateway when it's not needed can lead to significant cost savings.

## How does the Application Gateway impact on Azure costs?
Unlike some Azure services that only charge for usage, Application Gateway has a fixed hourly charge simply for being provisioned. Even if no traffic is flowing through it, you are still paying for the gateway to be available. For the Web Application Firewall V2 tier, this can be around €300 per month (as of August 18, 2025, for the Italy North region) just for having it running.

## How to Stop Azure Application Gateway
>**The Azure Application Gateway cannot be stopped from the Azure portal, you have to use az cli/Powershell or the [REST API](https://learn.microsoft.com/en-us/rest/api/application-gateway/application-gateways/stop?view=rest-application-gateway-2024-05-01&tabs=HTTP).**

### Using Azure CLI

```bash
az network application-gateway stop --ids <agw_id>
```

or

```bash
az network application-gateway stop \
  --name <agw_name> \
  --resource-group <rg_name>
```

### Using PowerShell

```bash
$AppGw = Get-AzApplicationGateway -Name <agw_name> -ResourceGroupName <rg_name>
Stop-AzApplicationGateway -ApplicationGateway $AppGw
```

## Important Considerations
What happens when you stop the Azure Application Gateway:
- All traffic routing through the gateway will be interrupted
- All configurations are preserved

![plot](https://github.com/fabiocannas/fabiocannas.github.io/blob/main/_posts/2025-07-18-Stopping-Azure-Application-Gateway-to-Save-Costs/2025-07-18-Stopping-Azure-Application-Gateway-to-Save-Costs.png?raw=true)

*Check Azure Application Gateway operational state in "Properties" blade.*

You can restart it anytime with the same settings.

## Restart the Azure Application Gateway when you need it

### Using Azure CLI

```bash
az network application-gateway start --ids <agw_id>
```

or

```bash
az network application-gateway start \
  --name <agw_name> \
  --resource-group <rg_name>
```

### Using PowerShell

```bash
$AppGw = Get-AzApplicationGateway -Name <agw_name> -ResourceGroupName <rg_name>
Start-AzApplicationGateway -ApplicationGateway $AppGw
```

**Note that it typically takes a few minutes for the Application Gateway to fully start and begin accepting traffic.**

For non-production environments or applications with predictable downtime, stopping Azure Application Gateway is one of the easiest ways to reduce Azure costs. The savings can be substantial, especially when multiplied across multiple environments or extended periods.