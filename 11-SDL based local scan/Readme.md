# AzSK.ADO PowerShell module SDL based scan

## Contents

  -  [Overview](Readme.md#overview)
  -  [Setup](Readme.md#Setup)
  	 -  [Installation Guide](Readme.md#installation-guide)
  -  [Getting Started](Readme.md#getting-started)
  	 -  [Import ADO module](Readme.md#import-ado-module)
 	 -  [Scanning Admin Controls](Readme.md#scanning-admin-controls)
 	 -  [Scanning Non-Admin Controls](Readme.md#scanning-non-admin-controls)
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
----------------------------------------------

## Getting Started

## Import ADO module
Firstly ADO module should be imported in the powershell session before using the scan commands. To import ADO module run below command.
```PowerShell
Import-Module AzSK.ADO
```
## Scanning Admin Controls

Project Control Administration access (PCA) and Project Administrator access is required to scan Organization and Project Controls respectively. Otherwise, the scan results depends on the access levels of the identity running the scanner.
To scan admin controls, it is always recommended to use "-IncludeAdminControls" switch

Run the command below after replacing `<OrganizationName>` with your Azure DevOps org Name 
and `<PRJ1, PRJ2, ..`> with a comma-separated list of project names where your Azure DevOps resources are hosted.
You will get Organization name from your ADO organization url e.g. http://sampleadoorg.visualstudio.com. In this 'sampleadoorg' is org name.

```PowerShell
Get-AzSKADOSecurityStatus -OrganizationName "<OrganizationName>" -ProjectNames "<PRJ1, PRJ2,...etc.>" -IncludeAdminControls
```

The outcome of the security scan/analysis is printed on the console during SVT execution and a CSV and LOG files are 
also generated for subsequent use.

The CSV file and LOG file are generated under a org-specific sub-folder in the folder  
*%LOCALAPPDATA%\Microsoft\AzSK.ADOLogs\Org_[yourOrganizationName]*  
E.g.  
C:\Users\<UserName>\AppData\Local\Microsoft\AzSK.ADOLogs\Org_[yourOrganizationName]\20181218_103136_GADS

## Scanning Non-Admin Controls

Non-admin controls always refer to the controls associate with any of the resource types other than Organization and project. i.e Build/Release/AgentPool/ServiceConnection/VariableGroup/SecureFile/Repo/Feeds

For example, to scan all builds in a project, run the command below after replacing `<OrganizationName>` with your Azure DevOps org Name 
and `<PRJName> with a  project name where your Azure DevOps resources are hosted.

```PowerShell
Get-AzSKADOSecurityStatus -OrganizationName "<OrganizationName>" -ProjectNames "<PRJName>" -ResourceTypeName Build 
```
Command also supports other parameters of filtering resources.

```PowerShell
#To scan selected builds in a project
Get-AzSKADOSecurityStatus -OrganizationName "<OrganizationName>" -ProjectNames "<PRJName>" -ResourceTypeName Build  -BuildNames "<Build1,Build2...>"

#Scan all supported artifacts
Get-AzSKADOSecurityStatus -OrganizationName "<OrganizationName>" -ScanAllResources

#Scan resources for baseline controls only
Get-AzSKADOSecurityStatus -OrganizationName "<OrganizationName>" -ProjectNames "<PRJName>"  -ResourceTypeName Build  -ubc

#Scan controls with particular tags only
Get-AzSKADOSecurityStatus -OrganizationName "<OrganizationName>" -ProjectNames "<PRJName>"  -ResourceTypeName Build  -FilterTags "<Comma sepated tags to filter>"
```
Check other parameters supported by command  [here](https://github.com/azsk/ADOScanner-docs/blob/master/02-%20Running%20ADO%20Scanner%20from%20command%20line/Readme.md).

> **Note:** Use the switch "-AllowLongRunningScan" if the number of resources scanning are morethan 1000 to acknowledge the acceptance of long running scan.

----------------------------------------------

### Execute SVTs using "-UsePartialCommits" switch

The Get-AzSKADOSecurityStatus command now supports checkpointing via a "-UsePartialCommits" switch. When this switch is used, the command periodically persists scan progress to disk. That way, if the scan is interrupted or an error occurs, a future retry can resume from the last saved state. This capability also helps in Continuous Assurance scans if scan gets suspended due to any unforeseen reason.The cmdlet below checks security control state via a "-UsePartialCommits" switch:
```PowerShell
Get-AzSKADOSecurityStatus-OrganizationName "<OrganizationName>" -ScanAllResources -UsePartialCommits
```
----------------------------------------------

#### Speed up checkpointed scans with "-DoNotRefetchResources" switch
The "-UsePartialCommits" switch also supports an optional switch: "-DoNotRefetchResources" in SDL mode. When this switch is used, resources are not re-fetched during the continuation of the checkpointed scan (i.e., when the "-upc" switch is used). This efficiently speeds up scans of subsequent batches after the initial one. Currently the resources supported with the switch are Release, Agent Pool, Organization and Project. 

```PowerShell
Get-AzSKADOSecurityStatus -OrganizationName "<OrganizationName>" -ProjectName "<ProjectName>" -ReleaseNames * -ResourceTypeName Release -UsePartialCommits -DoNotRefetchResources
```

----------------------------------------------

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

#### How to register with PSGallery
Use below command :
```PowerShell
Register-PSRepository -Name PSGallery -SourceLocation https://www.powershellgallery.com/api/v2/ -InstallationPolicy Trusted
```

### Support
- For any other issues or feedback please drop a mail to <a href="mailto:azskadosup@microsoft.com">ADO Scanner Support</a>
