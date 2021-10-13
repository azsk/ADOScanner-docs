# AzSK.ADO PowerShell module SDL based scan

## Contents

  -  [Overview](Readme.md#overview)
  -  [Setup](Readme.md#Setup)
  	 -  [Installation Guide](Readme.md#installation-guide)
  -  [Getting Started](Readme.md#getting-started)
 	 -  [Scanning Admin Controls](Readme.md#scanning-admin-controls)
 	 -  [Scanning Non-Admin Controls](Readme.md#scanning-non-admin-controls)
  -  [Execute SVTs for large organizations in batch mode](Readme.md#execute-svts-for-large-organizations-in-batch-mode)
  -  [Org-specific customization](Readme.md#customizing-adoscanner-using-org-policy)
	 -  [Overview](Readme.md#Introduction)
	 -  [Consuming custom org policy](Readme.md#consuming-custom-org-policy)
  -  [Compliance visibility](Readme.md#compliance-visibility)
  	 -  [Using the Log Analytics Workspace Summary for scan logs](Readme.md#using-the-log-analytics-workspace-for-scan-logs)
  	 -  [Using the Log Analytics Workbook for monitoring](Readme.md#using-the-log-analytics-workbook-for-monitoring)
  -  [FAQs](Readme.md#faqs)
  -  [Support](Readme.md#Support)
 
  
  
----------------------------------------------

## Overview
Security Scanner for Azure DevOps (AzSK.ADO) helps you keep your ADO resources such as various org/project settings, build/release configurations, service connections, agent pools, feeds, repositories, securefiles, environments etc. configured securely. 

> At its core, the Security Scanner for ADO is a PowerShell module. This can be run locally from the PS console after installation. This is as simple as running PS in non-Admin mode and running the scan commands.

> The ADO Scanner can be used in manual (command line) mode or as an ADO pipeline extension or as a scheduled Azure Function app for getting continuous assurance. We call these the **SDL** mode, the **CICD** mode and the **CA** mode, respectively.

The purpose of this document is to highlight that, even just using the SDL mode, the ADO Scanner can be leveraged to gain visibility and drive compliance while also using a organization-specific custom policy for control configuration/settings.

----------------------------------------------

## Setup 

### Installation Guide

>**Pre-requisites**:
> - PowerShell 5.0 or higher. 

1. First verify that pre-requisites are  installed:  
    Ensure that you have PowerShell version 5.0 or higher by typing **$PSVersionTable** in the PowerShell ISE console window and looking at the PSVersion in the output as shown below.) 
 If the PSVersion is older than 5.0, update PowerShell from [here](https://www.microsoft.com/en-us/download/details.aspx?id=54616). 
 
 <kbd>
   <img src="../Images/00_PS_Version.png" alt="PowerShell version">
</kbd>


2. Install the ADO Scanner (AzSK.ADO) PowerShell module:  
	  
```PowerShell
  Install-Module AzSK.ADO -Scope CurrentUser -AllowClobber -Force
```

----------------------------------------------

## Getting Started

### Import the ADO Scanner module
To run scans, we need to import AzSK.ADO module in a PowerShell session.

```PowerShell
Import-Module AzSK.ADO
```


## Scanning Admin Controls

Admin controls refer to controls associated with organization or project level settings. These are settings that can typically be modified by Project Collection Administrators (PCA) or Project Administrators (PA) respectively.

> Note: If you are not a member of these roles, you may be able to scan these controls but results may be inaccurate.

Run the command below to scan the admin controls for your ADO org and project(s):
```PowerShell
$orgName = "<OrganizationName>"
$projNames = "<ProjectNames>"
Get-AzSKADOSecurityStatus -OrganizationName $orgName -ProjectNames $projNames -IncludeAdminControls
```

## Scanning Non-Admin Controls

Non-admin controls refer to the controls associated with ADO resources created as part of various engineering activities within an organization/project. These are things like builds, releases, agent pools, service connections, variable groups, repos, feeds, etc.

The Get-AzSKADOSecurityStatus command can be used with various switches to define the target resources and controls for a scan. As an example, to scan _all_ builds in a project, you can run:

```PowerShell
$orgName = "<OrganizationName>"
$projNames = "<ProjectNames>"
Get-AzSKADOSecurityStatus -OrganizationName $orgName -ProjectNames $projNames -ResourceTypeName Build 
```
Typically, you would want to scan all resources in the project and limit the control set scanned to baseline controls only. This can be done via the command below:

#Scan resources for baseline controls only
Get-AzSKADOSecurityStatus -OrganizationName $orgName -ProjectNames $projNames  -ResourceTypeName Build  -ubc

Another useful option is the ability to scan controls that have a specific tag applied to them. For example, to scan all authorization related controls (tag = "AuthZ") for builds, you can run the following command:

```PowerShell
#Scan controls with particular tags only
Get-AzSKADOSecurityStatus -OrganizationName $orgName -ProjectNames $projNames  -ResourceTypeName Build  -FilterTags "AuthZ"
```

> Note: Available tags for controls can be viewed by running "Get-AzSKADOInfo -InfoType ControlInfo".

Check various other parameters supported by the command  [here](https://github.com/azsk/ADOScanner-docs/blob/master/02-%20Running%20ADO%20Scanner%20from%20command%20line/Readme.md).

> **Note:** Use the switch "-AllowLongRunningScan" if the number of resources scanning are more than 1000 to acknowledge the intent of long running scan.

----------------------------------------------

### Execute SVTs using "-UsePartialCommits" switch

The Get-AzSKADOSecurityStatus command now supports checkpointing via a "-UsePartialCommits" switch. When this switch is used, the command periodically persists scan progress to disk. That way, if the scan is interrupted or an error occurs, a future retry can resume from the last saved state. The cmdlet below checks security control state via a "-UsePartialCommits" switch:
```PowerShell
Get-AzSKADOSecurityStatus -OrganizationName $orgName -ScanAllResources -UsePartialCommits
```
----------------------------------------------

### Speed up checkpointed scans with "-DoNotRefetchResources" switch
The "-UsePartialCommits" switch also supports an optional switch: "-DoNotRefetchResources" in SDL mode. When this switch is used, resources are not re-fetched during the continuation of the checkpointed scan (i.e., when the "-upc" switch is used). This efficiently speeds up scans of subsequent batches after the initial one. Currently the resources supported with the switch are Release, Agent Pool, Organization and Project. 

```PowerShell
Get-AzSKADOSecurityStatus -OrganizationName $orgName -ProjectName $projNames -ReleaseNames * -ResourceTypeName Release -UsePartialCommits -DoNotRefetchResources
```

----------------------------------------------

### Execute path based scanning for builds and releases
The Get-AzSKADOSecurityStatus command supports path based scanning by scanning build and release configs constrained to specific build and release folder paths. This is achieved via two switches : "-BuildsFolderPath" and "-ReleasesFolderPath".
```PowerShell
Get-AzSKADOSecurityStatus -OrganizationName $orgName -ProjectName $projNames -ReleaseNames * -ResourceTypeName Release -ReleasesFolderPath "<ReleasesFolderPath>"

Get-AzSKADOSecurityStatus -OrganizationName $orgName -ProjectName $projNames -BuildNames * -ResourceTypeName Build -BuildsFolderPath "<BuildsFolderPath>"
```
Consider the following build folder structure: </br>
<kbd>
<img  src="../Images/02_Folder_Structure.PNG"  alt="Folder structure">
</kbd>
 </br>
To scan builds inside "Folder 1", the path should be given as "Folder 1". This will scan all builds inside this folder (i.e., Build 1, Build 2 and Build 3). To scan all builds inside "Folder 2", the path should be "Folder 1\Folder 2". This will scan Build 1 and Build 2.

----------------------------------------------


### Customize location for scan reports
The location to save scan reports can be customized using the command below:
```Powershell
Set-AzSKADOUserPreference -OutputFolderPath '<Custom folder path>'
```
The location to save scan report can be reset to default using command below:
```Powershell
Set-AzSKADOUserPreference -ResetOutputFolderPath
```

> **Note:** Please start new PS session to validate the settings.

----------------------------------------------

## Execute SVTs for large organizations in batch mode

ADO Scanner supports another command, **Get-AzSKADOSecurityStatusBatchMode (gadsbm)**, to facilitate scanning of large organizations whose scanning may continue across hours on an end. In such scenarios the *gadsbm* command scans your pipelines in batches that reduces the load on your RAM and storage. In case the scan interrupts in between, you can resume the scan from the last unscanned batch without having to go through the process of collecting pipelines again. It uses the -upc switch implicitly  ensuring you double checkpoints - batch wise as well as inside a batch.
 > - Consider scanning your projects in batch mode when the number of pipelines in the project is greater than 20K
 > -  Batch mode currently supports build and release pipelines. For all other resources, you can scan using GADS command
 #### The GADSBM command and its parameters
A typical security scan command in batch mode will look like as follows:
```PowerShell
#-----------Scan given projects-----------------#
GADSBM -OrganizationName "<OrganizationName>" -ProjectNames "<ProjectName1>, <ProjectName2>" -ResourceTypeName Build_Release -BatchSize <BatchSize> -FolderName "<FolderName>" -PATTokenURL "<KeyVaultUrlContainingPAT>"

#----------Scan all projects------------#
GADSBM -OrganizationName "<OrganizationName>" -ProjectNames "*" -ResourceTypeName Build_Release -BatchSize <BatchSize> -FolderName "<FolderName>" -PATTokenURL "<KeyVaultUrlContainingPAT>"
```
> Make sure you have the latest version of AzSK.ADO installed. After installation, if there are any older versions of the module imported in the current session already, you should start the batch mode in a fresh console. This is to make sure that the changes from the latest module take effect.

 Following are the scan parameters that are specific to the GADSBM command :
 
| Parameter name |Purpose  | Required?| Possible values
|--|--|--|--|
| OrganizationName | Name of the organization being scanned |True | Valid organization name
|ProjectName | Name of the project being scanned |True | Valid project name
| PATTokenURL | KeyVault URL for PAT | True | Valid key vault url
| ResourceTypeName | Resource type to be scanned | True | Build, Release, Build_Release|
| FolderName | Name of the folder where batch results are to be stored| True| Folder name (if folder doesn't exists, it will be created)
|BatchSize| The number of pipelines to be scanned in one batch |False| Valid batch size number
|KeepConsoleOpen| Keep powershell console open after a batch completes| False

Apart from the above parameters, you can also use the following parameters that are supported in *gads* as well :

 - UseBaselineControls
 - FilterTags
 - ControlIds
 - BuildsFolderPath
 - ReleasesFolderPath
 - PolicyProject
 - DoNotOpenFolder
 - All bug logging related parameters
 
#### Combining security reports from all batches
The results of each individual batch will be in the respective folders in the folder path as defined above. For a consolidated summary and easy viewing of the logs, you may want to combine the security reports from all batch results folders into one combined security report. You can do this using the _Get-AzSKADOSecurityStatusBatchMode (gadsbmr)_ command.

```Powershell
GADSBMR -OrganizationName "<OrganizationName>" -FolderName "<FolderName>"
```
All security reports will be combined in one security report. The security report will contain the path to the logs for each resource which you can use to analyse the results. 


Check the additional customization and features supported by batch mode [here](https://github.com/azsk/ADOScanner-docs/blob/master/02-%20Running%20ADO%20Scanner%20from%20command%20line/Readme.md#execute-svts-for-large-organizations-in-batch-mode).

----------------------------------------------
## Customizing ADOScanner using org policy

### Introduction

### When and why should I setup org policy

In many contexts you may want to "customize" the behavior of the security scans for your environment. You may want to do things such as: (a) enable/disable
some controls, (b) change control settings to better match specific security policies within your project, (c) change various messages, (d) add additional filter criteria for certain regulatory requirements that teams in your project can leverage, etc. When faced with such a need, you need a way to create and manage
a dedicated policy endpoint customized to the needs of your environment. The organization policy setup feature helps you do that in an automated fashion.

Check the detailed information , setup process and advanced features supported by org policy [here](https://github.com/azsk/ADOScanner-docs/tree/master/08-%20Customizing%20ADOScanner%20for%20your%20org).

----------------------------------------------

### Consuming custom org policy

Follow the steps below for consuming the org policy:

#### 1. Running scan in local machine with custom org policy

 To run scan with custom org policy from any machine, run the below command

```PowerShell
#Run scan cmdlet and validate if it is running with org policy
$orgName = "<Organization name>"
$projName = "<Name of the project hosting org policy with which the scan should run.>"
Get-AzSKADOSecurityStatus -OrganizationName $orgName -ProjectNames $projName

#Using 'PolicyProject' parameter
Get-AzSKADOSecurityStatus -OrganizationName $orgName -PolicyProject $projName

```
> **Note**: Using PolicyProject parameter you can specify the name of the project hosting organization policy with which the scan should run.

#### 2. Local folder-based org policy support

To facilitate use of ADO Scanner locally (from desktop console) for driving compliance for an org, we have added support to run against org policy (custom control settings, etc.) from a local folder instead of reading policy files from ADO policy repo. This capability can be used by org/project admins to evaluate ADOScanner and fine tune its configuration for their org/environment before deploying the org policy as a policy repo.

```PowerShell
#Command to configure a local folder as source of org policy:
Set-AzSKADOPolicySettings -LocalOrgPolicyFolderPath "<Folder path where the org policy is configured>"

#To reset the org policy to default location:
Set-AzSKADOPolicySettings -RestoreDefaultOrgPolicySettings
```
> **Note**: LocalOrgPolicyFolderPath should contain the file ServerConfigMetadata.json  with list of policy files mentioned in it.

----------------------------------------------
## Compliance visibility

The AzSK.ADO Monitoring Solution is deployed to a Log Analytics workspace that can be used for monitoring and generating a dashboard for compliance visibility and security monitoring.

####  Enabling Log Anaytics workspace connectivity into AzSK.ADO and routing AzSK.ADO events to Log Analytics

This section assumes that: a) you have a Log Analytics worskpace b) you have deployed AzSK.ADO monitoring solution.

Check how to create Log Analytics Workspace and deploy AzSK.ADO monitoring solution [here](https://github.com/azsk/ADOScanner-docs/blob/users/sragala/sdle2e-v2/06-%20Tracking%20compliance%20for%20your%20ADO%20environment/README.md#setting-up-the-azskado-monitoring-solution-step-by-step).

**Step-1 :** Connect AzSK.ADO to your Log Analytics workspace for sending AzSK.ADO control evaluation results.

Run the below in a PS session (this assumes that you have the latest AzSK.ADO installed).
```PowerShell
 $wsID = 'LogAnalyticsWorkSpaceID'       #See pictures in [A] above for how to get wsId and shrKey
 $shrKey = 'LogAnalyticsWorkSpaceSharedKey'
	
 Set-AzSKADOMonitoringSettings -WorkspaceID $wsID -SharedKey $shrKey
```
Close the current PS window and start a new one. (This is required for the new settings to take effect.)
After this, all AzSK.ADO cmdlets, SVTs, etc. run on the local machine will start sending events (outcomes of 
security scans) into the Log Analytics repository corresponding to the workspace ID above.

**Step-2 :** Generate some AzSK.ADO events and validate that they are reaching the configured Log Analytics workspace.

Run a few AzSK.ADO cmdlets to generate events for the Log Analytics repo. 
For example, you can run one or both of the following:

```PowerShell
 Get-AzSKADOSecurityStatus -OrganizationName "OrgName" 
 Get-AzSKADOSecurityStatus -OrganizationName "OrgName" -ProjectNames "PrjName"
```

After the above scans finish, Wait for few minutes and go into Log Analytics Workspace Logs and search for 'AzSK_ADO_CL', it should show 
AzSK.ADO events similar to the below ("_CL" stands for "custom log"):

### Using the Log Analytics Workspace for scan logs

####  Viewing raw events from AzSK.ADO

Click on the 'Logs' in the menu bar on the left to open the "Logs" query page.

Enter "AzSK_ADO_CL" in the query field.


You should see data about AzSK.ADO events as query results. (Again, this assumes that by now AzSK.ADO 
control scan results are being sent to this workspace.
<kbd>
![09_Log_Analytics_Workspace_Logs_Query](../Images/09_Log_Analytics_Workspace_Logs_Query.png) 
</kbd>
----------------------------------------------

### Using the Log Analytics Workbook for monitoring

Once the scan events are published to Log Analytics Workspace, the workbook created above will have compliance details with various security trends.
To view the workbook created, Goto : Log Analytics Workspace --> General --> Workbook --> click on the newly created workbook.

#### Workbook Overview tiles

The solution workbook contains multiple blades representing various types of security activity, 
security trends, etc. This view shows up when you click on the view tile and looks like the picture
below:
<kbd>
![09_Log_Analytics_Workbook_Overview](../Images/09_Log_Analytics_Workbook_Overview.png)
</kbd>

The "Help" (Summary) blade provides complete instructions on how to interpret the different
blades in this view. These blades cover the complete picture of baseline security compliance
for your organization and resources. It starts with a couple of blades that display organization
level security issues. The subsequent blades display project level security queries - each 
blade takes a different pivot to show the resource compliance data. 

----------------------------------------------

## FAQs

#### Error message: "Running scripts is disabled on this system..."
This is an indication that PowerShell script loading and execution is disabled on your machine. You will need to enable it before the ADOScanner installation script (which itself is a PowerShell script) can run. 
```PowerShell
Get-ExecutionPolicy -Scope CurrentUser
```
Incase you run above command in the PS console, you will likely see that the policy level is either 'Restricted' or 'Undefined'. For AzSK cmdlets to run, it needs to be set to 'RemoteSigned'.
To resolve this issue run the following command in your PS console:
```PowerShell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```
The execution policy setting will be remembered and all future PS consoles opened in non-Admin (CurrentUser) mode will apply the 'RemoteSigned' execution policy.

#### Error message: The remote server returned an error: (401) Unauthorized
Incase you encounter the error message `Organization not found: Incorrect organization name or you do not have necessary permission to access the organization. InvalidOperation: The remote server returned an error: (401) Unauthorized`, this could mean that the logged in identity does not have access to the organization. 

To ensure the tool use the correct identity, you can force reset the credentials by using "-ResetCredentials" parameter. 
Here's how you can accomplish this: 

```PowerShell
Import-Module AzSK.ADO
#Run the gads command with "-ResetCredentials" parameter
gads -OrganizationName $orgName -ProjectNames $projNames -ResetCredentials

```

#### How to register with PSGallery
Use below command :
```PowerShell
Register-PSRepository -Name PSGallery -SourceLocation https://www.powershellgallery.com/api/v2/ -InstallationPolicy Trusted
```

### Support
- For any other issues or feedback please drop a mail to <a href="mailto:azskadosup@microsoft.com">ADO Scanner Support</a>
