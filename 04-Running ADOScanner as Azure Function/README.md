# Running ADOScanner as Azure Function (Continuous Assurance)

## Contents

  -  [Overview](README.md#overview)
  -  [Automated Scanning using Azure functions](README.md#automated-scanning-using-azure-functions)
     * [Setting up ADOScanner using Azure function - Step by Step](README.md#setting-up-adoscanner-using-azure-function---step-by-step)
     * [Updating an existing Continuous Assurance setup](README.md#updating-an-existing-continuous-assurance-setup)
     * [Getting details about a Continuous Assurance setup](README.md#getting-details-about-a-continuous-assurance-setup)
     * [Continuous Assurance using containers - how it works (under the covers)](README.md#continuous-assurance-using-containers---how-it-works-under-the-covers)

----------------------------------------------

## Overview

The basic idea behind Continuous Assurance (CA) is to setup the ability to check for "drift" from what is considered a secure snapshot of a system. Support for Continuous Assurance lets us treat security truly as a 'state' as opposed to a 'point in time' achievement. This is particularly important in today's context when 'continuous change' has become a norm.

Besides 'drift tracking' there are also two other aspects of "staying secure" in operations. First of them is the simple concept that if new, more secure options become available for a feature, it should be possible to detect that a particular application or solution can benefit from them and notify/alert the owners concerned. In a way this can be thought of as facilitating "positive" security drift. The other aspect is about supporting "operational hygiene". In this area, we will add the ability to remind a team about the security hygiene tasks that they need to periodically perform (key rotation, access reviews,  removing inactive/dormant power users, etc.).

----------------------------------------------

## Automated Scanning using Azure functions

An Azure-based continuous assurance scanning solution for ADO can be setup in subscription. It will run ADO security scanner inside a container image and the scanning infrastructure of this containerized model will be hosted in an Azure resource group. This provides an alternate option to running the scanner via ADO pipeline extension and is designed to be more suitable for larger environments.

### Setting up ADOScanner using Azure function - Step by Step
In this section, we will walk through the steps of setting up a Azure DevOps Organization for Continuous Assurance coverage in a subscription. 

To get started, we need the following:
1. The user setting up Continuous Assurance needs to have 'Owner' access to the subscription or Owner access on existing resource group where setup needs to be created. 

2. Target Log Analytics WorkspaceID* and SharedKey. (The Log Analytics workspace can be in a different subscription, see note below)


**Step-1: Setup**  
1. Install latest AzSK.ADO module.
2. Run the '**Install-AzSKADOContinuousAssurance**' command with required parameters given in below table. 

```PowerShell
	Install-AzSKADOContinuousAssurance -SubscriptionId <SubscriptionId> `
                                    [-ResourceGroupName <ResourceGroupName>] `
                                    -OrganizationName <OrganizationName> `
                                    -ProjectName <ProjectName> `
                                    -PATToken <SecureStringPATToken> `
                                    [-Location <Location>] `
                                    [-LAWSId <WorkspaceId>] `
                                    [-LAWSSharedKey <SharedKey>] `
                                    [-AltLAWSId <WorkspaceId>] `
                                    [-AltLAWSSharedKey <SharedKey>] `
                                    [-ExtendedCommand <ExtendedCommand>] `
                                    [-ScanIntervalInHours <ScanIntervalInHours>] `
                                    [-CreateLAWorkspace]
```
Note: 
Updating ExtendedCommand overrides the default '-ScanAllArtifacts' behavior of CA. If you need that, please specify '-saa' switch in CA '-ExtendedCommand'

Here are few basic examples of continuous assurance setup command:

<b>Example 1:</b> Install CA in default resource group 'ADOScannerRG' 
```PowerShell
	Install-AzSKADOContinuousAssurance -SubscriptionId <SubscriptionId> `
                                    -OrganizationName <OrganizationName> `
                                    -PATToken <SecureStringPATToken> `
                                    -Location <Location> `
                                    -LAWSId <WorkspaceId> `
                                    -LAWSSharedKey <SharedKey> `
                                    -AltLAWSId <WorkspaceId> `
                                    -AltLAWSSharedKey <SharedKey> `
                                    -ProjectName <ProjectName> `
                                    -ExtendedCommand <ExtendedCommand> 
```

<b>Example 2:</b> Install CA in a custom resource group and create log analytics workspace in the same resource group
```PowerShell
	Install-AzSKADOContinuousAssurance -SubscriptionId <SubscriptionId> `
                                    -ResourceGroupName <ResourceGroupName> `
                                    -OrganizationName <OrganizationName> `
                                    -ProjectName <ProjectName> `
                                    -PATToken <SecureStringPATToken> `
                                    -Location <Location> `
                                    -CreateLAWorkspace

```


Note:

|Param Name|Purpose|Required?|Default value|
|----|----|----|----|
|SubscriptionId|Subscription ID of the host subscription in which Continuous Assurance setup will be done |TRUE|None|
|ResourceGroupName|(Optional) Name of resource group where Function app will be installed|FALSE|ADOScannerRG|
|OrganizationName|Organization name for which scan will be performed|TRUE|None|
|ProjectName|Project to be scanned|TRUE|None|
|PATToken|PAT token secure string for organization to be scanned|TRUE|None|
|Location|Location in which all resources need to be setup|FALSE|East US|
|LAWSId|(Optional) Workspace ID of Log Analytics workspace which will be used to monitor security scan results|FALSE|None|
|LAWSSharedKey|(Optional) Shared key of Log Analytics workspace which will be used to monitor security scan results|FALSE|None|
|AltLAWSId|(Optional) Alternate workspace ID of Log Analytics workspace which will be used to monitor security scan results|FALSE|None|
|AltLAWSSharedKey|(Optional) Alternate shared key of Log Analytics workspace which will be used to monitor security scan results|FALSE|None|
|ExtendedCommand|(Optional) Extended command to narrow down the target scan|FALSE|FALSE|
|ScanIntervalInHours|(Optional) Overrides the default scan interval (24hrs) with the custom provided value.|FALSE|24|
|CreateLAWorkspace|(Optional) Switch to create and map new Log Analytics workspace with CA setup|FALSE|FALSE|


**Step-2: Verifying that CA Setup is complete**  
**1:** In the Azure portal, select the host subscription that was used above and search for resource group. You should see a Function App created by the name starting with 'ADOScannerFA'. There will also be some other resources App Service Plan, Key vault to store PAT secret, Application Insights, Storage to store scan logs, Log Analytics workspace (created only if CreateLAWorkspace switch is used while installation )

**2:** Click on 'Functions' tile. It should show the following function: 
<kbd>	
![09_CA_FunctionApp](../Images/09_CA_FunctionApp.png)
</kbd>

**3:** Click on 'Configuration' tile. It should show the application settings of the function app. Default schedule is to run scan 20 minutes post Install CA command and then every 24 hours after that.


**Step-4: Verifying CA execution and Log Analytics connectivity**  
Once CA setup is completed successfully, the function app will automatically trigger (once a day) and scan the organization and the specified projects for the organization. The outcomes of these scans will be saved in a storage account created during installation (format : adoscannersa\<YYMMDDHHMMSS> e.g. adoscannersa200815181008)

The results of the control evaluation are also routed to the Log Analytics for viewing via a security dashboard.  
  
Let us verify that the function app output is generated as expected and that the Log Analytics connectivity is setup and working correctly.

**1:** Verify CSV file and LOG file are getting generated 
 
1. Go to Storage Explorer and look for a storage account with a name in adoscannersa<YYMMDDHHMMSS> format in your subscription in 'ADOScannerRG' resource group.
2. Find a blob container called 'ado-scan-logs' in this storage account.
3. There should be a ZIP file named using a timestamp based on the date time for the manual execution in this container (most likely the ZIP file with the most recent creation date). 
4. Download the ZIP file and extract its contents locally.
<kbd>	
<img src="../Images/09_CA_Storage_Logs.png" alt="09_CA_Storage_Logs">
</kbd>
**2:** Verify that data is being sent to the target Log Analytics workspace   

1. Go to the Log Analytics workspace that we used to setup CA above.
2. Navigate to 'Logs' window, and enter below query
  AzSK_ADO_CL | where Source_s == "CA"
3. You should see results similar to the below:
<kbd>	
<img src="../Images/09_CA_Laws_Query.png" alt="09_CA_Laws_Query">
</kbd>

----------------------------------------------

### Updating an existing Continuous Assurance setup

The '**Update-AzSKADOContinuousAssurance**' command can be used to make changes to a previously setup CA configuration.
For instance, you may use it to:
- Update the Pat token, Organization name
- Switch the Log Analytics workspace information that CA should use to send control evaluation events to
- Update Project names to be scanned , extended command

To do any or all of these:
1. Open the PowerShell ISE and login to your Azure account (using **Connect-AzAccount**).  
2. Run the '**Update-AzSKADOContinuousAssurance**' command with required parameters given in below table. 

```PowerShell
Update-AzSKADOContinuousAssurance -SubscriptionId <SubscriptionId> `
                                  -OrganizationName <OrganizationName> `
                                  [-ResourceGroupName <ResourceGroupName>] `
                                  [-LAWSId <WorkspaceId>] `
                                  [-LAWSSharedKey <SharedKey>] `
                                  [-AltLAWSId <WorkspaceId>] `
                                  [-AltLAWSSharedKey <SharedKey>] `
                                  [-PATToken <PATToken>] `
                                  [-ProjectName <ProjectName>] `
                                  [-ExtendedCommand <ExtendedCommand>] `
                                  [-ScanIntervalInHours <ScanIntervalInHours>] `
                                  [-RsrcTimeStamp <RsrcTimeStamp>] `
                                  [-ClearExtendedCommand]

```

----------------------------------------------

### Getting details about a Continuous Assurance setup

Run the 'Get-AzSKADOContinuousAssurance' command as below.
Result will display the current status of CA in your subscription. If CA is not working as expected, it will display remediation steps else it will display a message indicating CA is in healthy state.
Once you follow the remediation steps, run the command again to check if anything is still missing in CA setup. Follow the remediation steps accordingly until the CA state becomes healthy.

Command:
1. Open the PowerShell ISE and login to your Azure account (using **Connect-AzAccount**).  
2. Run the '**Get-AzSKADOContinuousAssurance**' command with required parameters given in below table. 

```PowerShell
Get-AzSKADOContinuousAssurance  -SubscriptionId <SubscriptionId> `
                                -OrganizationName <OrganizationName> `
                                [-ResourceGroupName <ResourceGroupName>] `
                                [-FunctionAppName <FunctionAppName>]
```

----------------------------------------------

### Continuous Assurance using containers - how it works (under the covers)
The CA feature is about tracking configuration drift. This is achieved by enabling support for running AzSK.ADO periodically.

The CA installation script that sets up CA creates the following resources in your subscription:

- Resource group (Name : ADOScannerRG) :- 
To host all the Continuous Assurance artifacts
- Storage account (Format : adoscannersaYYMMDDHHMMSS) :- To store the daily results of CA scans. The storage account is named with a timestamp-suffix applied to 'adoscannersa'(e.g. adoscannersa200815181008)
- Function app (Format : ADOScannerFAYYMMDDHHMMSS) :- To trigger the containerised image and provide input to function from application setings. The function app is named with a timestamp-suffix applied to 'ADOScannerFA'(e.g. ADOScannerFA200815181008)
- Application Insights (Format : ADOScannerFAYYMMDDHHMMSS) :- To collect log, performance of the function in containerised image. The application insights is named with a timestamp-suffix and has same name as function app (e.g. ADOScannerFA200815181008)
- Key Vault (Format : ADOScannerKVYYMMDDHHMMSS) :- To safegaurd the pat token used in running scan. The Key vault is named with a timestamp-suffix applied to 'ADOScannerKV' (e.g. ADOScannerKV200815181008)
- Managed Identity of Function app :- This is used at the runtime to fetch pat from keyvault and store logs in storage account. It has Get,List access on keyvault and contributor access on storage account   
