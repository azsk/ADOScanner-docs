# Continuous Assurance

## Contents

  -  [Overview](README.md#overview)
  -  [Automated Scanning using Azure functions](Readme.md#Automated-Scanning-using-Azure-functions)
     * [Setting up Continuous Assurance - Step by Step](README.md#overview)
     * [Updating an existing Continuous Assurance setup](README.md#)
     * [Getting details about a Continuous Assurance setup](README.md#)
     * [Continuous Assurance using containers - how it works (under the covers)](README.md#)
  -  [Automated Scanning using ADO extensionn](Readme.md#Automated-Scanning-using-ADO-extension)
     * [Setting up Continuous Assurance - Step by Step](README.md#setting-up-continuous-assurance---step-by-step-1)
     * [Customizing your PAT with minimum required privileges for ADO Connection](README.md#-customizing-your-pat-with-minimum-required-privileges-for-azure-devops-connection)
     * [Visualize security scan results](README.md#visualize-security-scan-results)

----------------------------------------------

## Overview

The basic idea behind Continuous Assurance (CA) is to setup the ability to check for "drift" from what is considered a secure snapshot of a system. Support for Continuous Assurance lets us treat security truly as a 'state' as opposed to a 'point in time' achievement. This is particularly important in today's context when 'continuous change' has become a norm.

Besides 'drift tracking' there are also two other aspects of "staying secure" in operations. First of them is the simple concept that if new, more secure options become available for a feature, it should be possible to detect that a particular application or solution can benefit from them and notify/alert the owners concerned. In a way this can be thought of as facilitating "positive" security drift. The other aspect is about supporting "operational hygiene". In this area, we will add the ability to remind a team about the security hygiene tasks that they need to periodically perform (key rotation, access reviews,  removing inactive/dormant power users, etc.).

----------------------------------------------

## Automated Scanning using Azure functions

An Azure-based continuous assurance scanning solution for ADO can be setup in subscription. It will run ADO security scanner inside a container image and the scanning infrastructure of this containerized model will be hosted in an Azure resource group. This provides an alternate option to running the scanner via ADO pipeline extension and is designed to be more suitable for larger environments.

### Setting up Continuous Assurance - Step by Step
In this section, we will walk through the steps of setting up a Azure DevOps Organization for Continuous Assurance coverage in a subscription. 

To get started, we need the following:
1. The user setting up Continuous Assurance needs to have 'Owner' access to the subscription. 

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
|ExtendedCommand|(Optional) Extended command to narrow down the target scan|TRUE|FALSE|
|ScanIntervalInHours|(Optional) Overrides the default scan interval (24hrs) with the custom provided value.|FALSE|24|
|CreateLAWorkspace|(Optional) Switch to create and map new Log Analytics workspace with CA setup|TRUE|FALSE|


**Step-2: Verifying that CA Setup is complete**  
**1:** In the Azure portal, select the host subscription that was used above and search for resource group. You should see a Function App created by the name starting with 'ADOScannerFA'. There will also be some other resources App Service Plan, Key vault to store PAT secret, Application Insights, Storage to store scan logs, Log Analytics workspace (created only if CreateLAWorkspace switch is used while installation )

**2:** Click on 'Functions' tile. It should show the following function: 
	
![09_CA_FunctionApp](../Images/09_CA_FunctionApp.png)


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
	
![09_CA_Storage_Logs](../Images/09_CA_Storage_Logs.png)

**2:** Verify that data is being sent to the target Log Analytics workspace   

1. Go to the Log Analytics workspace that we used to setup CA above.
2. Navigate to 'Logs' window, and enter below query
  AzSK_ADO_CL | where Source_s == "CA"
3. You should see results similar to the below:
	
![09_CA_Laws_Query](../Images/09_CA_Laws_Query.png)

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
- Managed Identity of Function app :- This is used at the runtime to fetch pat from keyvault and store can logs in storage account. It has Get,List access on keyvault and contributor access on storage account   

----------------------------------------------

## Automated Scanning using ADO extension


Security Scanner for ADO is also available as a native ADO extension that can be set up to scan your ADO artifacts periodically so as to achieve "continuous assurance". The extension also comes packaged with widgets that you can use to visualize scan results by relevance to stakeholder groups (such as org admin, project owners, build/release owners etc.) in dashboards.

The basic idea behind Continuous Assurance (CA) is to setup periodic security scan and if new, more secure options become available for a feature, it should be possible to detect so that an application or solution can benefit from them and notify/alert the owners concerned.

Scan is performed via security scanner task in the pipeline and results can be visualized via dashboard by adding ADO security scanner widgets into the Azure DevOps project's dashboard. Pipeline can be setup with the trigger to run periodically and provide continuous assurance.

### Setting up Continuous Assurance - Step by Step

In this section, we will walk through the steps of setting up an Azure DevOps pipeline for ADO Continuous Assurance coverage.
To get started, we need the following 

__Prerequisite:__

- ADO organization and project 
- "Project Collection Administrator" or "Owner" permission to perform below task:

  * Install "ADO Security Scanner" extension

  * Setup pipeline with scanner task.
    
  * Create dashboard to visualize scan results



#### Install "ADO Security Scanner" extension for your Azure DevOps Organization


Extension has been published to the Visual Studio marketplace gallery under "Azure DevOps > Azure Pipeline" category. You can now install this extension from the Marketplace directly (https://marketplace.visualstudio.com/items?itemName=azsdktm.ADOSecurityScanner).

Refer doc [here](https://docs.microsoft.com/en-us/azure/devops/marketplace/install-extension?view=azure-devops&tabs=browser) for more about installing extensions for org

  ![Extension Details](../Images/09_ADO_ExtensionDetails.png) 


#### Adding ADO Security Scanner in the pipeline

This part assumes that you are familiar with Azure DevOps build tasks and pipelines at a basic level. Our goal is to show how ADO Security Scanner can be added into the build/release workflow.

__Step-1__: Create a build pipeline or open an existing one.

__Step-2__: Add "ADO Security Scanner" task to the pipeline

Click on "Add Tasks" and select "Azure DevOps (ADO) Security Verification".

![Add scanner task](../Images/09_ADO_AddADOScannerTask.png)

__Step-3__: Specify the input parameters for the task.
The "ADO Security Scanner" task starts showing in the "Run on Agent" list and displays some configuration inputs that are required for the task to run. These are none other than the familiar options we have been specifying while running the ADO scan manually - you can choose to specify the target org, projects, builds and releases based on how your org resources are organized.

![Add task inputs](../Images/09_ADO_AddTaskDetails.png)

> **Note:** This task also requires Azure DevOps connection containing org details and PAT to scan the required resources. Refer doc [here](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops&tabs=preview-page) to create token and provide it as part of connection


<html>
<head>

</head><body>
<H3> Customizing your PAT with minimum required privileges for Azure DevOps Connection</H3>

Here is a scope-wise list of minimum privileges that needs to be assigned to your PAT to ensure a smooth experience of the security scan.

<table><tr><th>Scope</th><th>Privilege</th></tr>
<tr><td>
Agent Pools
</td><td>Read</tr>

<tr><td>
Auditing
</td><td>Read Audit Log</tr>

<tr><td>
Build
</td><td>Read</tr>

<tr><td>
Entitlements
</td><td>Read</tr>

<tr><td>
Extension Data
</td><td>Read & write</tr>

<tr><td>
Extensions
</td><td>Read</tr>

<tr><td>
Graph
</td><td>Read</tr>

<tr><td>
Identity
</td><td>Read</tr>


<tr><td>
Member Entitlement Management
</td><td>Read</tr>

<tr><td>
Project and Team
</td><td>Read</tr>

<tr><td>
Release
</td><td>Read</tr>

<tr><td>
Secure Files
</td><td>Read</tr>

<tr><td>
Service Connections
</td><td>Read</tr>

tr><td>
Task Groups
</td><td>Read</tr>

<tr><td>
Tokens
</td><td>Read & manage</tr>

<tr><td>
User Profile
</td><td>Read</tr>

<tr><td>
Variable Groups
</td><td>Read</tr>

<tr><td>
Work Items
</td><td>Read & write</tr>

</table>
<table>
</table>
</body></html>


![Add Service connection](../Images/09_ADO_AddServiceConnection.png)


__! Important__ : Make sure you **DO NOT** select  checkbox for "Grant access permission to all pipelines" before saving service connection. 

> **Note**: ADO Security Scanner extension enables you to leverage some of the advanced capabilities of scanner while running in adhoc mode. You could scan for only preview baseline controls in your build pipeline, or you could just scan for controls with specific severity etc. These advanced features are available to customers through ADO variables. For example, use *ExtendedCommand* variable in the pipeline with its value as *-Severity 'High'* to scan controls with high severity.


__Step-4__: Click "Save & queue"

![Add Service connection](../Images/09_ADO_TriggerPipeline.png)

Task will install latest ADO scanner module and start scanning based on input parameters. 

![Scan Image](../Images/09_ADO_ScanImage-1.png)

At the end, it will show the summary of scan and store the result in extension storage.

![Scan Image](../Images/09_ADO_ScanImage-2.png)

__Step-4__: Setup scheduled trigger for pipeline

Once you are able to successfully run the ADO scan using ADO pipeline, you can configure scheduled trigger to get latest visibility of security on resources

![Schedule Trigger](../Images/09_ADO_ScheduleTrigger.png)

----------------------------------------------

### Visualize security scan results 

Once scan is completed as part of pipeline, results can be visualized with the help of project dashboard.

Extension mainly provides two widgets that can be added as part of dashboard

__* Org Level Security Scan Summary__: Displays org level security control evaluation summary. This dashboard helps org owners to take action based on control failures.

__* Project Component Security Scan Summary__: Displays project components (Build/Release/Connections) security control evaluation summary.

__Steps__:

1. Go to project dashboard under your organization and create new dashboard for org level summary

    ![Create Dashboard](../Images/09_ADO_AddDashboard.png)

2. Click edit or add widget > Search for "__Org Level Security Scan Summary__" > Click 'Add' followed by "Done Editing"

    ![Configure Widget](../Images/09_ADO_AddOrgSummaryWidget.png)

3. Dashboard will start displaying scanned results 

    ![org Level Summary](../Images/09_ADO_OrgLevelDashboard.png)

Step 1,2 & 3 needs to be repeated to add "__Project Component Security Scan Summary__"

![Schedule Trigger](../Images/09_ADO_ProjectComponentLevl.png)


> **Note:**  Dashboard created will be visible to all users which are part of project.

> **Note:**  Dashboard reflects updates only upon pipeline execution. Local scan results don't reflect automatically. If you have remediated a control, make sure you run the pipeline to reflect the updated control results on dashboard.

