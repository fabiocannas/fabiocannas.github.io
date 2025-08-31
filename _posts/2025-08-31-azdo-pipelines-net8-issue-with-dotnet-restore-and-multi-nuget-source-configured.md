---
layout: single
classes: wide
title: "Dotnet 8 Issue With Dotnet Restore And Multi Nuget Source Configured"
date: 2025-08-31
permalink: "/2025/08/31/dotnet8-issue-with-dotnet-restore-and-multi-nuget-source-configured/"
category: [Devops]
tags: [dotnet, CICD pipelines, Development, .NET, .NET 8, Nuget, Devops, Azure Devops]
toc: true
toc_label: "Dotnet 8 Issue With Dotnet Restore And Multi Nuget Source Configured"
toc_icon: "book"
---
For some time now, an issue has been identified in .NET 8 that affects developers using multiple NuGet package sources, particularly when one or more sources require authentication. The issue manifests as authentication failures during dotnet restore operations.

## Issue Details
The problem occurs when the nuget.config file contains multiple package sources, such as:

```xml
<packageSources>
  <add key="nuget.org" value="https://api.nuget.org/v3/index.json" protocolVersion="3" />
  <add key="externalSourceThatRequireAuth" value="https://[...]/nuget/v3/index.json"/>
  <add key="Microsoft Visual Studio Offline Packages" value="C:\Program Files (x86)\Microsoft SDKs\NuGetPackages\" />
</packageSources>
```

When running dotnet restore in .NET 8 with multiple configured NuGet sources, developers encounter the following error:
> C:\Program Files\dotnet\sdk\8.0.100\NuGet.targets(156,5): error : Unable to load the service index for source https://[...]/nuget/v3/index.json. 
> C:\Program Files\dotnet\sdk\8.0.100\NuGet.targets(156,5): error : Response status code does not indicate success: 401 (Unauthorized). 

I myself have encountered this issue on some customer Azure Devops pipelines and, by force of circumstances, I ended up on the NuGet [Github issue 13129](https://github.com/NuGet/Home/issues/13129).

## Workarounds

Couple of workarounds have been identified that successfully resolve the issue:

- Installation of the latest version of Azure Artifact Credential Provider;

- Using VSS_NUGET_EXTERNAL_FEED_ENDPOINTS environment variable for DotNetCoreCLI@2 task in Azure Devops Pipelines.

I chose the second solution, because I wasn't very confident in updating the Azure artifact credential provider on my customer (a big one) self-hosted Azure Devops Agent, used daily by all development teams of that company.

## Using VSS_NUGET_EXTERNAL_FEED_ENDPOINTS environment variable for DotNetCoreCLI@2 task in Azure Devops Pipelines
Here is how to use the VSS_NUGET_EXTERNAL_FEED_ENDPOINTS environment variable in DotNetCoreCLI@2 task.

```yml
- task: DotNetCoreCLI@2
  inputs:
    command: "restore"
    projects: '<sln_file_path>'
    feedsToUse: 'config'
    nugetConfigPath: '<nuget_config_path>'
    restoreArguments: '--no-cache'
  env:
    VSS_NUGET_EXTERNAL_FEED_ENDPOINTS: $(FEED_ENDPOINT)
```
> The $(FEED_ENDPOINT) variable holds a secret value.

[VSS_NUGET_EXTERNAL_FEED_ENDPOINTS](https://github.com/microsoft/artifacts-credprovider#other-automated-build-scenarios) is a JSON that contains an array of service endpoints, usernames and access tokens to authenticate endpoints in nuget.config

```json
{"endpointCredentials": [{"endpoint":"http://example.index.json", "username":"optional", "password":"accesstoken"}]}
```

As you can see from the example, this solution requires an Azure Devops [Personal Access Token](https://learn.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops&tabs=Windows).
The PAT must have Packaging (Read) permission, to allow consuming packages from the Azure Artifacts feed.

## Important Note
Normally, you should not use a PAT to access Azure Devops Artifact feeds in Azure Devops Pipelines.
In fact, it is enough that the AZDO project *Build Service* has read permissions on the feed in order to be able to consume its packages.