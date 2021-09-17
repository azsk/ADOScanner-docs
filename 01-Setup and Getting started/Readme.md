# AzSK.ADO PowerShell module

## Contents

  -  [Overview](Readme.md#overview)
  -  [Setup](Readme.md#Setup)
  	 -  [Installation Guide](Readme.md#installation-guide)
  	 -  [Auto Update](Readme.md#auto-update)
  -  [Getting Started](Readme.md#getting-started)
  	 -  [Import ADO module](Readme.md#import-ado-module)
 	 -  [Begin scanning your Organisation and Project](Readme.md#begin-scanning-your-organisation-and-project)
  -  [FAQs](Readme.md#faqs)
  -  [Support](Readme.md#Support)
 
  
  
----------------------------------------------

## Overview
Security Scanner for Azure DevOps (ADO) helps you keep your ADO artifacts such as various org/project settings, build/release configurations, service connections, agent pools, etc. configured securely. You can run the ADO Security Scanner standalone in a PowerShell console or in an ADO pipeline via a marketplace extension.

Security Scanner for Azure DevOps (ADO) performs security scanning for core areas of Azure DevOps like Organization, Projects, Users, Pipelines (Build & Release), Connections and Agent Pools. 

> At its core, the Security Scanner for ADO is a PowerShell module. This can be run locally from the PS console after installation. This is as simple as running PS in non-Admin mode and running the cmds as shown below:

----------------------------------------------

### Setup 

## Installation Guide

>**Pre-requisites**:
> - PowerShell 5.0 or higher. 

1. First verify that prerequisites are already installed:  
    Ensure that you have PowerShell version 5.0 or higher by typing **$PSVersionTable** in the PowerShell ISE console window and looking at the PSVersion in the output as shown below.) 
 If the PSVersion is older than 5.0, update PowerShell from [here](https://www.microsoft.com/en-us/download/details.aspx?id=54616). 
 
 <kbd>
   <img src="../Images/00_PS_Version.png" alt="PowerShell version">
</kbd>


2. Install the Security Scanner for Azure DevOps (AzSK.ADO) PS module:  
	  
```PowerShell
  Install-Module AzSK.ADO -Scope CurrentUser -AllowClobber -Force
```
------------------------------------------------

### Auto Update
As Azure DevOps features evolve and add more security controls, "AzSK.ADO" also evolves every month respecting the latest security features.
It is always recommended to scan your organization with the latest AzSK.ADO module, thus ensuring to evaluate latest security controls that are available through the module.
"AzSK.ADO" module provides different auto update capabilities w.r.t different stages of devops. More details are below:

**Adhoc Scans:**
Users running the older version of AzSK.ADO scan from their local machine will get a warning as shown in the image below.
It would also provide the user with required instructions to upgrade the module.
<kbd>	
![Install_Autoupdate](../Images/Updatecommand.PNG) 
</kbd>
The users can leverage the auto update feature which has been introduced from the AzSDK.ADO version 1.15.x.
As shown in the image above, user can go with update by running the command below:

```PowerShell
  Set-AzSKADOPolicySettings -AutoUpdate On|Off
```

User needs to close and reopen a fresh session once the command is run.
Going forward, if the latest version of AzSK.ADO is released, then during execution of any AzSK.ADO command it would start the auto update workflow automatically 
as shown in the image below:
<kbd>
![Install_Autoupdate_Workflow](../Images/AutoUpdate.png)
</kbd>
Step 1: It would take user consent before it starts the auto update workflow. (1 in the image above) <br/>
Step 2: Users need to close all the displayed PS sessions. Typically open PS sessions would lock the module and fail the installation. (2 in the image above) <br/>
Step 3: Even the current session must be closed. It would again take the user consent before it starts the auto update flow to avoid the loss of any unsaved work. (3 in the image above)

**ADOScanner Extension:**
No impact to default behavior of ADOScanner extension. It always runs the scan with the latest version available in the PS Gallery. 

----------------------------------------------

## Getting Started

## Import ADO module
Firstly ADO module should be imported in the powershell session before using the scan commands. To import ADO module run below command.
```PowerShell
Import-Module AzSK.ADO
```
## Begin scanning your Organisation and Project

Run the command below after replacing `<OrganizationName>` with your Azure DevOps org Name 
and `<PRJ1, PRJ2, ..`> with a comma-separated list of project names where your Azure DevOps resources are hosted.
You will get Organization name from your ADO organization url e.g. http://sampleadoorg.visualstudio.com. In this 'sampleadoorg' is org name.

```PowerShell
Get-AzSKADOSecurityStatus -OrganizationName "<OrganizationName>" -ProjectNames "<PRJ1, PRJ2,...etc.>"
```

The outcome of the security scan/analysis is printed on the console during SVT execution and a CSV and LOG files are 
also generated for subsequent use.

The CSV file and LOG file are generated under a org-specific sub-folder in the folder  
*%LOCALAPPDATA%\Microsoft\AzSK.ADOLogs\Org_[yourOrganizationName]*  
E.g.  
C:\Users\<UserName>\AppData\Local\Microsoft\AzSK.ADOLogs\Org_[yourOrganizationName]\20181218_103136_GADS

## Customize location for scan reports
The location to save scan reports can be customized using the command below:
```Powershell
Set-AzSKADOUserPreference -OutputFolderPath '<Custom folder path>'
```
The location to save scan report can be reset to default using command below:
```Powershell
Set-AzSKADOUserPreference -ResetOutputFolderPath
```

Refer [link](/ControlCoverage) for current control coverage for Azure DevOps

### FAQs

#### Error message: "Running scripts is disabled on this system..."
This is an indication that PowerShell script loading and execution is disabled on your machine. You will need to enable it before the ADOScanner installation script (which itself is a PowerShell script) can run. 
```PowerShell
Get-ExecutionPolicy -Scope CurrentUser
```
If you run above command in the PS console, you will likely see that the policy level is either 'Restricted' or 'Undefined'. For AzSK cmdlets to run, it needs to be set to 'RemoteSigned'.
To resolve this issue run the following command in your PS console:
```PowerShell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```
The execution policy setting will be remembered and all future PS consoles opened in non-Admin (CurrentUser) mode will apply the 'RemoteSigned' execution policy.

#### Error message: The remote server returned an error: (401) Unauthorized
If you encounter the error message `Organization not found: Incorrect organization name or you do not have necessary permission to access the organization. InvalidOperation: The remote server returned an error: (401) Unauthorized`, this could mean that the logged in identity does not have access to the organization. 

To ensure the tool use the correct identity, you can force the sign-in dialog to appear by setting a variable. 
Here's how you can accomplish this: 

```PowerShell
$AzSKADOLoginUI = 1       ## Helps you ensure you log in with the correct identity
Import-Module AzSK.ADO
# > Run your commands again now.
```

#### Error message: "Cannot process argument transformation on parameter 'ResourceTypeName'. Cannot convert value....."

While using ADOScanner as pipeline extension ,If you encounter the error message Cannot process argument transformation on parameter 'ResourceTypeName'. Cannot convert value..., this could mean that the pipeline got corrupted due to known.
To fix this issue, Please clone you pipeline and run again.

#### How to register with PSGallery
Use below command :
```PowerShell
Register-PSRepository -Name PSGallery -SourceLocation https://www.powershellgallery.com/api/v2/ -InstallationPolicy Trusted
```

### Support
- For any other issues or feedback please drop a mail to <a href="mailto:azskadosup@microsoft.com">ADO Scanner Support</a>
