---
title: "App Service with Azure Files mount as local share, using private connectivity. Did you know?"
date: 2025-08-05
---
You are working on an Azure infrastructure that must implement private connectivity.

You create an App Service that needs a Azure Files mounted as local share.
Both App Service and Storage account use private connectivity only.

**[App Service's virtual network integration](https://learn.microsoft.com/en-us/azure/app-service/overview-vnet-integration) is active, so the app can access resources in your virtual network.
Inbound private access to the app and to the storage account is granted through [private endpoints](https://learn.microsoft.com/en-us/azure/private-link/private-endpoint-overview).**

You already configured the fileshare mount on App Service, but the app is unable to read from the fileshare.

Why?

![Math lady meme](https://media1.giphy.com/media/v1.Y2lkPTc5MGI3NjExdGlmb2gxMHRqZ2R4eGRiN2QxYTZucmRhNjZlaTlhZDZyMWV3cW05OSZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/WRQBXSCnEFJIuxktnw/giphy.gif)

The reasons could be many, but one important thing to know is the following: 
**with virtual network integration on your app, the mounted drive uses an RFC1918 IP address and not an IP address from your virtual network.
In order to enable routing through your virtual network you have to set the following app setting in App Service's environment variables:**

```bash
WEBSITE_CONTENTOVERVNET = 1
```

Sources:

- [Manage Azure App Service virtual network integration routing](https://learn.microsoft.com/en-us/azure/app-service/configure-vnet-integration-routing#content-share)

- [Mount Azure Storage as a local share in App Service](https://learn.microsoft.com/en-us/azure/app-service/configure-connect-to-azure-storage?tabs=basic%2Cportal&pivots=container-linux)

