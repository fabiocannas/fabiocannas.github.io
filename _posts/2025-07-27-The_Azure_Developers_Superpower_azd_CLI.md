---
title: "The Azure Developer's Superpower: azd CLI"
date: 2025-07-27
---
After a somewhat lengthy pause, it was time to update my blog. 

Following is the content of my session at the Azure Meetup Casteddu, held a few days ago at Sa Manifattura, Cagliari.

![plot](https://github.com/fabiocannas/blog/blob/main/_posts/2025-07-27-The_Azure_Developers_Superpower_azd_CLI/2025-07-27-The_Azure_Developers_Superpower_azd_CLI_intro_slide.jpg?raw=true)

To stay updated about Azure Meetup Casteddu events, I invite you to join the community:

[Join Azure Meetup Casteddu on Whatsapp](https://chat.whatsapp.com/E1m2qrQ4V8E15OLgGAkWFg=)

[Join Azure Meetup Casteddu - Meetup page](https://www.meetup.com/it-IT/azure-meetup-casteddu/)

[Follow Azure Meetup Casteddu on LinkedIn](https://www.linkedin.com/company/azure-meetup-casteddu/posts/?feedView=all)

# Azure Developer CLI (azd)

## The common Azure Development Challenges

Have you ever spent more time configuring Azure deployments than actually writing code?

Well, it happened to me, specially when I was learning Azure.

Before diving into this article, try to answer the following questions:

1. How long does it take you to build a proof of concept?
2. How long does it take you to deploy it on Azure?
3. Can you do it yourself or do you ask your DevOps mates for help?
4. Are you able to implement a robust repeatable deployment process that allows you to share your work with your colleagues so they can work on it and redeploy it in a short time? How long does it take you?

## The Problems We All Face

### Infrastructure Setup Complexity
- Deep knowledge of Azure services is required

### Time spent on DevOps/Platform engineering instead of development 
- Deployment pipeline + IAC headaches

### Environment consistency issues

All of these are factors that can slow down development and increase the risk of errors.

## The Solution: Azure Developer CLI

An open-source tool that accelerates provisioning and deploying app resources on Azure.
azd CLI provides best practice, developer-friendly commands that map to key stages in development workflows.

> **Note:** Azure Developer CLI ≠ Azure CLI

| Tool                | Sample Command                                                                 | Outcome                                                                                   |
|---------------------|------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------|
| Azure Developer CLI | `azd provision`                                                               | Provisions multiple Azure resources required for an app based on project resources and configurations, such as an Azure resource group, an Azure App Service web app and app service plan, an Azure Storage account, and an Azure Key Vault. |
| Azure CLI           | `az webapp create --resource-group myResourceGroup --plan myAppServicePlan --name myWebApp` | Provisions a new web app in the specified resource group and app service plan.            |
| Azure PowerShell    | `New-AzWebApp -ResourceGroupName "myResourceGroup" -Name "myWebApp" -AppServicePlan "myAppServicePlan"` | Provisions a new web app in the specified resource group and app service plan.            |

[source - aka.ms/learn - Compare Azure Developer CLI commands](https://learn.microsoft.com/en-us/azure/developer/azure-developer-cli/azd-commands?WT.mc_id=AZ-MVP-5004796#compare-azure-developer-cli-commands)

## Features

### Supported Azure compute services (host)
| Azure compute service    | Feature Stage  |
| ------------------------ | -------------- |
| Azure App Service        | Stable         |
| Azure Static Web Apps    | Stable         |
| Azure Container Apps     | Beta           |
| Azure Functions          | Stable         |
| Azure Kubernetes Service | Beta (only for projects deployable via `kubectl apply -f`)    |
| Azure Spring Apps        | Beta           |

[source - aka.ms/learn - azd CLI Supported Azure Compute Services](https://learn.microsoft.com/en-us/azure/developer/azure-developer-cli/supported-languages-environments#supported-azure-compute-services-host)

### Supported languages and frameworks
| Language | Feature Stage |
| -------- | -----------   |
| Node.js  | Stable        |
| Python   | Stable        |
| .NET     | Stable        |
| Java     | Stable        |

[source - aka.ms/learn - azd CLI Supported Languages and Framework](https://learn.microsoft.com/en-us/azure/developer/azure-developer-cli/supported-languages-environments#supported-languages-and-framework)

### Template library - [AWESOME-AZD](aka.ms/awesome-azd)
![plot](https://github.com/fabiocannas/blog/blob/main/_posts/2025-07-27-The_Azure_Developers_Superpower_azd_CLI/2025-07-27-The_Azure_Developers_Superpower_azd_CLI_azd_awesome.jpg?raw=true) 

### Develop your own templates
![plot](https://github.com/fabiocannas/blog/blob/main/_posts/2025-07-27-The_Azure_Developers_Superpower_azd_CLI/2025-07-27-The_Azure_Developers_Superpower_azd_CLI_custom_templates.jpg?raw=true) 

[source - aka.ms/learn - Make azd Compatible](https://learn.microsoft.com/en-us/azure/developer/azure-developer-cli/make-azd-compatible)

## Template Structure
- `.azure` folder - Contains essential Azure configurations and environment variables, such as the location to deploy resources or other subscription information
- `infra` folder - Contains all of the Bicep or Terraform infrastructure-as-code files for the azd template
- `src` folder - Contains all of the deployable app source code
- `azure.yaml` file - A configuration file that defines services and maps them to Azure resources

## Workflows

### azd init workflows
1. Scan current directory: Analyzes existing app codebase to generate appropriate configuration
2. Select a template: Clones and initializes a template from gallery
3. Create a minimal project: Initializes basic azure.yaml file
 
### azd up workflow
5. Packaging: prepares the application code and dependencies
6. Provisioning: creates and configures Azure resources
7. Deployment: deploys the packaged application

## azd CLI Zero to Hero in 8 Commands

```bash
# 1. Login to Azure
azd auth login

# 2. Initialize from template
azd init --template todo-csharp-sql

# 3. Provision and deploy
azd up

# 4. View your live app
azd show

# 5. Provision infrastructure (Bicep/Terraform)
azd provision

# 6. Make changes to app code and redeploy
azd deploy

# 7. Monitor and troubleshoot
azd monitor

# 8. Delete resources
azd down
```

Below you will see the commands in action:

### azd auth login - Let's authenticate to Azure
![plot](https://github.com/fabiocannas/blog/blob/main/_posts/2025-07-27-The_Azure_Developers_Superpower_azd_CLI/2025-07-27-The_Azure_Developers_Superpower_azd_CLI_azd_auth_login.jpg?raw=true) 

### azd init - Let's download a template and then initialize the workspace
![plot](https://github.com/fabiocannas/blog/blob/main/_posts/2025-07-27-The_Azure_Developers_Superpower_azd_CLI/2025-07-27-The_Azure_Developers_Superpower_azd_CLI_azd_init.jpg?raw=true)

### azd init - Let's take a look to the .azure folder
![plot](https://github.com/fabiocannas/blog/blob/main/_posts/2025-07-27-The_Azure_Developers_Superpower_azd_CLI/2025-07-27-The_Azure_Developers_Superpower_azd_CLI_azd_init_dotazure_folder.jpg?raw=true)

### azd up - This is my favorite command. Hold on tight! haha
![plot](https://github.com/fabiocannas/blog/blob/main/_posts/2025-07-27-The_Azure_Developers_Superpower_azd_CLI/2025-07-27-The_Azure_Developers_Superpower_azd_CLI_azd_up_1.jpg?raw=true)

![plot](https://github.com/fabiocannas/blog/blob/main/_posts/2025-07-27-The_Azure_Developers_Superpower_azd_CLI/2025-07-27-The_Azure_Developers_Superpower_azd_CLI_azd_up_2.jpg?raw=true)

### azd show - Let's have a look at the apps that we just published
![plot](https://github.com/fabiocannas/blog/blob/main/_posts/2025-07-27-The_Azure_Developers_Superpower_azd_CLI/2025-07-27-The_Azure_Developers_Superpower_azd_CLI_azd_show.jpg?raw=true)

### azd deploy - Let's publish changes to the webapp
![plot](https://github.com/fabiocannas/blog/blob/main/_posts/2025-07-27-The_Azure_Developers_Superpower_azd_CLI/2025-07-27-The_Azure_Developers_Superpower_azd_CLI_azd_deploy.jpg?raw=true)

### azd monitor - Finally, we can monitor our application 
![plot](https://github.com/fabiocannas/blog/blob/main/_posts/2025-07-27-The_Azure_Developers_Superpower_azd_CLI/2025-07-27-The_Azure_Developers_Superpower_azd_CLI_azd_monitor.jpg?raw=true)

![plot](https://github.com/fabiocannas/blog/blob/main/_posts/2025-07-27-The_Azure_Developers_Superpower_azd_CLI/2025-07-27-The_Azure_Developers_Superpower_azd_CLI_azd_monitor_az_portal_dashboard.jpg?raw=true)

`TIP: run the following command to go to Application Insights Live Metrics`

```bash
azd monitor --live
```

### azd down
![plot](https://github.com/fabiocannas/blog/blob/main/_posts/2025-07-27-The_Azure_Developers_Superpower_azd_CLI/2025-07-27-The_Azure_Developers_Superpower_azd_CLI_azd_down.jpg?raw=true)


## CI/CD Pipeline Support

### Pipeline Configuration
The `azd pipeline config` command automates provisioning and deployment using pipeline definition files included in azd templates.

Supports:
- Azure Pipelines
- Github Actions

### Configuration Steps
1. Authenticate with Azure
2. Select CI/CD platform
3. Configure repository
4. Set up service principal
5. Configure authentication:
   - GitHub: OpenID Connect (OIDC) or client credentials
   - Azure Pipelines: Workload identity federation (OIDC) or client credentials
6. Provision pipeline files
7. Set pipeline variables and secrets
8. Commit and push changes
9. Trigger pipeline runs

## Event Hooks

Hooks can execute custom scripts before and after azd commands or service lifecycle events.
Configure in azure.yaml file with OS-specific support (Windows or Posix).

### Command Hooks
- prerestore and postrestore
- preprovision and postprovision
- predeploy and postdeploy
- preup and postup
- predown and postdown

### Service Lifecycle Hooks
- prerestore and postrestore
- prebuild and postbuild
- prepackage and postpackage
- predeploy and postdeploy

## azd compose (alpha)

```bash
azd config set alpha.compose on
```

- `azd add` command creates resources without manual IAC templates
- Infrastructure state is tracked in-memory
- `azd infra gen` or `azd infra synth`  converts state to Bicep files

## azd CLI Extensions (alpha)

```bash
azd config set alpha.extensions on
```

The extensions are modular components that extend azd CLI functionality.

### Extension Sources
Extensions are distributed and managed through extension sources (an equivalent concept to NuGet or NPM feeds):
- Official registry: https://aka.ms/azd/extensions/registry
- Development registry: https://aka.ms/azd/extensions/registry/dev

Extension sources are manifest files providing lists of available extensions, following [the official schema](https://github.com/Azure/azure-dev/blob/main/cli/azd/extensions/registry.schema.json).

## Get Started with azd CLI

### Install azd CLI

```bash
# Windows 
winget install microsoft.azd 

# macOS 
brew tap azure/azd && brew install azd 

# Linux 
curl -fsSL https://aka.ms/install-azd.sh | bash
```

### Try It Out
```bash
azd auth login

azd init --template todo-csharp-sql

azd up
```

## Resources
### Documentation: [aka.ms/azd](aka.ms/azd)

### Templates: [aka.ms/awesome-azd](aka.ms/awesome-azd)

### Blog: [Azure SDK Blog](https://devblogs.microsoft.com/azure-sdk/tag/azure-developer-cli/)

### See you at the next Azure Meetup Casteddu events!

[Join Azure Meetup Casteddu on Whatsapp](https://chat.whatsapp.com/E1m2qrQ4V8E15OLgGAkWFg=)

[Join Azure Meetup Casteddu - Meetup page](https://www.meetup.com/it-IT/azure-meetup-casteddu/)

[Follow Azure Meetup Casteddu on LinkedIn](https://www.linkedin.com/company/azure-meetup-casteddu/posts/?feedView=all)