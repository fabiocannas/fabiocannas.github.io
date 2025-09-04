---
layout: single
classes: wide
title: "Retrieve PowerBI Audit Logs Using Exchange Online Management PowerShell Module in Azure Automation Account"
date: 2025-09-03
last_modified_at: 2025-09-03
permalink: "/2025/09/03/retrieve-powerbi-audit-logs-using-exo-management-powershell-module-in-azure-automation-account/"
category: [PowerShell, Azure Automation]
tags: [Azure Automation Account, PowerBI, Exchange Online PowerShell, PowerShell, Microsoft Fabric]
toc: true
toc_label: "Retrieve PowerBI Audit Logs Using Exchange Online Management PowerShell Module in Azure Automation Account"
toc_icon: "book"
toc_sticky: true
---
In this article I explain how to retrieve the PowerBI audit logs using Exchange Online Management PowerShell module in Azure Automation Account.

## Prerequisites

- EntraID Global Administrator role;
- Microsoft Graph PowerShell SDK;
- An Azure Automation Account with an associated system managed identity;
- PowerShell v7.2;
- An Azure Storage Account with a Fileshare to persist audit logs;
- The Automation Account's identity must be assigned with the following RBAC roles on storage account:
  - Reader and data access;
  - Storage File Data SMB Share Contributor.
- Microsoft Fabric audit logs must be enabled, <a href="https://learn.microsoft.com/en-us/purview/audit-search#before-you-search-the-audit-log" target="_blank" rel="noopener noreferrer">read more</a>.

## Creation Of A Role Group In Exchange Online
**IMPORTANT:** Access to enable/disable auditing and access to audit cmdlets requires permissions from the Exchange Admin center.
{: .notice--warning}

The access to Exchange Online and his commandlets must be restricted to reduce the risk of data compromise.
To do so, it is necessary to create a role group in Exchange Online Admin center that will give the Automation Account the permission to read the audit logs.

To create a role group in Exchange Online, log in to the Exchange Admin Center and from the “admin roles” page, create a role group as follows:

| Name              | Permissions         |
|-------------------|---------------------| 
| Audit Logs Reader | View-Only Audit Logs|

## Assign "Audit Logs Reader" Role Group To The Automation Account's System Managed Identity

```bash
Connect-MgGraph -Scopes AppRoleAssignment.ReadWrite.All,Application.Read.All
 
$EntraIDApp = Get-MgServicePrincipal -Filter "DisplayName eq '<automation_account_name>'"

Connect-ExchangeOnline -UserPrincipalName <global_admin_user>

New-ServicePrincipal -DisplayName "<automation_account_name>" -AppId "<app_ID>" -ServiceId $ EntraIDApp.Id
 
Add-RoleGroupMember -Identity "Audit Logs Reader" -Member "<app_ID>"
```

## Assign The Exchange.ManageAsApp EntraID Permission To The Automation Account's System Managed Identity

To allow the Azure Automation Account to access resources in Exchange, assign the following EntraID Application API permission:

*Office 365 Exchange Online* > *Exchange.ManageAsApp*

```bash
Connect-MgGraph -Scopes AppRoleAssignment.ReadWrite.All,Application.Read.All

$TenantID="<tenant_id>"
$DisplayNameOfMSI="<automation_account_name>"
$Office365ExchangeOnlineAppId = "00000002-0000-0ff1-ce00-000000000000" 
$ExchangeManageAsAppRoleID = "dc50a0fb-09a3-484d-be87-e023b12c6440" 

$MSI = (Get-MgServicePrincipal -Filter "displayName eq '$DisplayNameOfMSI'").Id
$ResourceID = (Get-MgServicePrincipal -Filter "AppId eq '$Office365ExchangeOnlineAppId").Id

New-MgServicePrincipalAppRoleAssignment -ServicePrincipalId $MSI -PrincipalId $MSI -AppRoleId $ExchangeManageAsAppRoleID -ResourceId $ResourceID
```

## Use A PowerShell Runbook In Azure Automation Account To Extract PowerBI Audit Logs

To implement the runbook, I used the script you find on <a href="https://learn.microsoft.com/en-us/purview/audit-log-search-script#step-2-modify-and-run-the-script-to-retrieve-audit-records" target="_blank" rel="noopener noreferrer">Microsoft Learn</a> as a basis.

Here are the changes I made:
- Added parameters to execute as runbook in Azure Automation Account;
- Connection to Exchange Online;
- Disconnection from Exchange Online without a confirmation prompt;
- Added <a href="https://learn.microsoft.com/en-us/azure/automation/manage-runbooks#retry-logic-in-runbook-to-avoid-transient-failures" target="_blank" rel="noopener noreferrer">retry logic</a>;
- Added logic to save export file to Azure Files.

```bash
[CmdletBinding()]
param(
 [Parameter(Mandatory=$False)]
 [string]$TenantID,
 [Parameter(Mandatory=$True)]
 [ValidateNotNullOrEmpty()]
 [string]$SubscriptionID,
 [Parameter(Mandatory=$True)]
 [ValidateNotNullOrEmpty()]
 [string]$ResourceGroupName,
 [Parameter(Mandatory=$True)]
 [ValidateNotNullOrEmpty()]
 [string]$StorageAccountName,
 [Parameter(Mandatory=$True)]
 [ValidateNotNullOrEmpty()]
 [string]$FileShareName,
 [Parameter(Mandatory=$True)]
 [ValidateNotNullOrEmpty()]
 [string]$FolderName,
 [Parameter(Mandatory=$False)]
 [ValidateNotNullOrEmpty()]
 [string]$RecordType = "PowerBIAudit"
)
$ErrorActionPreference = "Stop"

$startTime = (Get-Date)
filter timestamp {"[$(Get-Date -Format G)]: $_"}
Write-Output "Script started" | timestamp
$Stoploop = $false
$Retrycount = 0
$MaxRetry = 3
$WaitTimeInSeconds = 30
$outputFile = ""
do {
    try {
        Import-Module ExchangeOnlineManagement

        Connect-ExchangeOnline -ManagedIdentity -Organization "<organization_name>"

        [DateTime]$start = [DateTime]::UtcNow.AddDays(-1).Date
        [DateTime]$end = $start.Date.AddHours(23).AddMinutes(59).AddSeconds(59).AddMilliseconds(999)
        $resultSize = 5000
        $intervalMinutes = 60
        $year = $startTime.ToString("yyyy")
        $month = $startTime.ToString("MM")
        $day = $startTime.ToString("dd")
        $outputFile = "$env:TEMP\AuditLogRecords_$year$month$day.csv"

        #Start script
        [DateTime]$currentStart = $start
        [DateTime]$currentEnd = $end

        Write-Output "Retrieving audit records for the date range between $($start) and $($end), RecordType=$RecordType, ResultsSize=$resultSize" | timestamp

        $totalCount = 0
        while ($true)
        {
            $currentEnd = $currentStart.AddMinutes($intervalMinutes)
            If ($currentEnd -gt $end)
            {
                $currentEnd = $end
            }

            If ($currentStart -eq $currentEnd)
            {
                break
            }

            $sessionID = [Guid]::NewGuid().ToString() + "_" +  "ExtractLogs" + (Get-Date).ToString("yyyyMMddHHmmssfff")
            Write-Output "Retrieving audit records for activities performed between $($currentStart) and $($currentEnd)" | timestamp
            $currentCount = 0
            do
            {
              $results = Search-UnifiedAuditLog -StartDate $currentStart -EndDate $currentEnd -RecordType $RecordType -SessionId $sessionID -SessionCommand ReturnLargeSet -ResultSize $resultSize
              if (($results | Measure-Object).Count -ne 0)
              {
                  $results | export-csv -Path $outputFile -Append -NoTypeInformation
                  $currentTotal = $results[0].ResultCount
                  $totalCount += $results.Count
                  $currentCount += $results.Count
                  if ($currentTotal -eq $results[$results.Count - 1].ResultIndex)
                  {
                      Write-Output "Successfully retrieved $($currentTotal) audit records for the current time range. Moving on to the next interval."
                      ""
                      break
                  }
              }    
            }
            while (($results | Measure-Object).Count -ne 0)
            $currentStart = $currentEnd
        }

        #Silently disconnect from Exchange Online without a confirmation prompt
        Disconnect-ExchangeOnline -Confirm:$false

        Write-Output "Script complete! Finished retrieving audit records for the date range between $($start) and $($end). Total count: $totalCount" | timestamp
        If($totalCount -gt 0) {
            $directoryPath = "$FolderName/$year-$month-$day"
            $FileName = "auditLogs.csv"
            
            If ([string]::IsNullOrEmpty($TenantID)) {
                $TenantID = Get-AutomationVariable -Name "DefaultTenantId"
            }

            Write-Output "Logging in to Azure using Automation Account identity" | timestamp
            Connect-AzAccount -Identity -Tenant $TenantID -Subscription $SubscriptionID

            Write-Output "Upload files to file share" | timestamp  

            $ctx = (Get-AzStorageAccount -ResourceGroupName $ResourceGroupName -Name $StorageAccountName).Context

            $folderExists = $false
            try {
                Get-AzStorageFile -Context $ctx -ShareName $FileShareName -Path $directoryPath
                $folderExists = $true
                Write-Output "Directory $directoryPath already exists." | timestamp
            }
            catch {
                Write-Output "Directory $directoryPath does not exist, creating it..." | timestamp
                $folderExists = $false
            }

            if (-not $folderExists) {
                New-AzStorageDirectory -Context $ctx -ShareName $FileShareName -Path $directoryPath
            }

            Set-AzStorageFileContent -Context $ctx -ShareName $FileShareName -Source $outputFile -Path "$directoryPath/$FileName" -Force
        }

        $Stoploop = $true
    }
    catch {
        if ($Retrycount -gt $MaxRetry)
        {
            Write-Output "Export failed $MaxRetry times and we will not try again." | timestamp
            $Stoploop = $true
            Write-Error -Message $_.Exception | timestamp
            throw $_.Exception
        }
        else  
        {
            Write-Output -Message $_.Exception.Message | timestamp
            Write-Output "Export failed. Retrying in $WaitTimeInSeconds seconds..." | timestamp
            Start-Sleep -Seconds $WaitTimeInSeconds
            $Retrycount = $Retrycount + 1
        }
    }
    finally{
        if (Test-Path $outputFile) {
            Remove-Item -Path $outputFile -Force
        }
    }
}
While ($Stoploop -eq $false)

$duration = NEW-TIMESPAN –Start $startTime –End (Get-Date)
Write-Output "Done in $([int]$duration.TotalMinutes) minute(s) and $([int]$duration.Seconds) second(s)" | timestamp
```

## Important Notes
- You can retrieve audit logs through the Office 365 Management Activity API or the audit log search tool in the Microsoft Purview portal;
- This runbook extracts sensitive data, which must be managed carefully, especially when used in an enterprise environment;
- Access to Exchange Online Management PowerShell module must be controlled and limited. It is good practice to <a href="https://learn.microsoft.com/en-us/powershell/exchange/disable-access-to-exchange-online-powershell?view=exchange-ps" target="_blank" rel="noopener noreferrer">disable EXO PowerShell module</a> on accounts that do not need to use them.
  
Here is how to disable Exchange Online PowerShell module access for users who don't have any directory roles (admin roles) in Microsoft 365:

```bash
Connect-MgGraph

Get-MgUser | ForEach-Object {
 if (-not (Get-MgUserMemberOf -UserId $_.UserPrincipalName | Where-Object { $_.'@odata.type' -eq '#microsoft.graph.directoryRole' })) {
    Set-User -Identity $_.UserPrincipalName -EXOModuleEnabled $false
  }
}
```