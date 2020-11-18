# AzSK.ADO PowerShell module

## Contents
 -  [Import ADO module](Readme.md#import-ado-module)
 -  [Scan your Azure DevOps resources](Readme.md#scan-your-azure-devops-resources)
 -  [Execute SVTs using "-IncludeAdminControls" switch](Readme.md#execute-svts-using--includeadmincontrols-switch)
 -  [Execute SVTs using "-DetailedScan" switch](Readme.md#execute-svts-using--detailedscan-switch)
 -  [Execute SVTs using "-UsePartialCommits" switch](Readme.md#execute-svts-using--usepartialcommits-switch)

## Import ADO module
Firstly ADO module should be imported in the powershell session before using the scan commands.To import module ADO module run below command.
```PowerShell
Import-Module AzSK.ADO
```
## Scan your Azure DevOps resources

Run the command below after replacing `<OrganizationName>` with your Azure DevOps org Name 
and `<PRJ1, PRJ2, ..`> with a comma-separated list of project names where your Azure DevOps resources are hosted.
You will get Organization name from your ADO organization url e.g. http://sampleadoorg.visualstudio.com. In this 'sampleadoorg' is org name.

```PowerShell
Get-AzSKADOSecurityStatus -OrganizationName "<OrganizationName>" -ProjectNames "<PRJ1, PRJ2,...etc.>"
```

Command also supports other parameters of filtering resources.
For instance, you can also make use of the 'BuildNames','ReleaseNames' to filter specific resource

```PowerShell

#Scan organization
Get-AzSKADOSecurityStatus -OrganizationName "<OrganizationName>"

#Scan organization and Project
Get-AzSKADOSecurityStatus -OrganizationName "<OrganizationName>" -ProjectNames "<PRJ1,PRJ2,etc>" 

#Scan organization, project and builds
Get-AzSKADOSecurityStatus -OrganizationName "<OrganizationName>" -ProjectNames "PRJ1" -BuildNames "<BLD1, BLD2,...etc.>" 

#Scan organization, project and releases
Get-AzSKADOSecurityStatus -OrganizationName "<OrganizationName>" -ProjectNames "PRJ1" -ReleaseNames "<RLS1, RLS2,...etc.>" 

#Scan organization, project, all builds and releases
Get-AzSKADOSecurityStatus -OrganizationName "<OrganizationName>" -ProjectNames "PRJ1" -BuildNames "*" -ReleaseNames "*" 

#Scan organization, project and service connections
Get-AzSKADOSecurityStatus -OrganizationName "<OrganizationName>" -ProjectNames "PRJ1" -ServiceConnectionNames "<SER1, SER2,...ect.>"

#Scan organization, project and agent pools
Get-AzSKADOSecurityStatus -OrganizationName "<OrganizationName>" -ProjectNames "PRJ1" -AgentPoolNames "<AGP1, AGP2,...etc.>"

#Scan organization, project and Variable groups
Get-AzSKADOSecurityStatus -OrganizationName "<OrganizationName>" -ProjectNames "PRJ1" -VariableGroupNames "<VGN1,VGN2>"

#Scan organization, project, all builds, releases, service connectiopns and agent pools
Get-AzSKADOSecurityStatus -OrganizationName "<OrganizationName>" -ProjectNames "PRJ1" -BuildNames "*" -ReleaseNames "*" -ServiceConnectionNames "*" -AgentPoolNames "*"

#Scan organization, project, all builds, releases and variable groups
Get-AzSKADOSecurityStatus -OrganizationName "<OrganizationName>" -ProjectNames "PRJ1" -BuildNames "*" -ReleaseNames "*" -VariableGroupNames "*"

#Scan organization and Project with PAT prompted
Get-AzSKADOSecurityStatus -OrganizationName "<OrganizationName>" -ProjectNames "PRJ1" -BuildNames "*" -PromptForPAT

#Scan organization and Project with PAT token URL
Get-AzSKADOSecurityStatus -OrganizationName "<OrganizationName>" -ProjectNames "PRJ1" -ReleaseNames "*" -PATTokenURL

#Scan all supported artifacts
Get-AzSKADOSecurityStatus -OrganizationName "<OrganizationName>" -ScanAllArtifacts

#Scan projects 
Get-AzSKADOSecurityStatus -OrganizationName "<OrganizationName>" -ProjectNames "<PRJ1,PRJ2,etc>" -ResourceTypeName Project

#Scan builds 
Get-AzSKADOSecurityStatus -OrganizationName "<OrganizationName>" -ProjectNames "PRJ1" -ResourceTypeName Build

#Scan releases 
Get-AzSKADOSecurityStatus -OrganizationName "<OrganizationName>" -ProjectNames "PRJ1"  -ResourceTypeName Release

#Scan service connections 
Get-AzSKADOSecurityStatus -OrganizationName "<OrganizationName>" -ProjectNames "PRJ1"  -ResourceTypeName ServiceConnection

#Scan agent pools 
Get-AzSKADOSecurityStatus -OrganizationName "<OrganizationName>" -ProjectNames "PRJ1"  -ResourceTypeName AgentPool

#Scan Variable group
Get-AzSKADOSecurityStatus -OrganizationName "<OrganizationName>" -ProjectNames "PRJ1"  -ResourceTypeName VariableGroup

#Scan using service Id
Get-AzSKADOSecurityStatus -OrganizationName "<OrganizationName>" -ProjectNames "PRJ1"  -ServiceId "<service Id>"

#Scan resources for baseline controls only
Get-AzSKADOSecurityStatus -OrganizationName "<OrganizationName>" -ProjectNames "<PRJ1,PRJ2,etc>" -ubc

#Scan resources with severity
Get-AzSKADOSecurityStatus -OrganizationName "<OrganizationName>" -ProjectNames "<PRJ1,PRJ2,etc>" -Severity "High/Medium/Low"
```

The outcome of the security scan/analysis is printed on the console during SVT execution and a CSV and LOG files are 
also generated for subsequent use.

The CSV file and LOG file are generated under a org-specific sub-folder in the folder  
*%LOCALAPPDATA%\Microsoft\AzSK.ADOLogs\Org_[yourOrganizationName]*  
E.g.  
C:\Users\<UserName>\AppData\Local\Microsoft\AzSK.ADOLogs\Org_[yourOrganizationName]\20181218_103136_GADS

Refer [doc](../02-Secure-Development#understand-the-scan-reports) for understanding the scan report and [link](./ControlCoverage) for current control coverage for Azure DevOps

To scan large number of project component (more then 1000 resources default value), you need to supply an additional switch parameter -AllowLongRunningScan in the command.
```PowerShell
Get-AzSKADOSecurityStatus -OrganizationName "<OrganizationName>" -ProjectNames "<PRJ1, PRJ2,...etc.>" -AllowLongRunningScan
```
Allowing scan for more then 1000 resources can be configured through the organization policy by updating 'IsAllowLongRunningScan' and 'LongRunningScanCheckPoint' properties in the ControlSettings.json file. 
If 'IsAllowLongRunningScan' is set to true, then by using '-AllowLongRunningScan' switch parameter, AzSK.ADO allows scan for resources count which is set in 'LongRunningScanCheckPoint'. If 'IsAllowLongRunningScan' value is set to false it does not allow scan for more then resources count set in 'LongRunningScanCheckPoint'.

### Execute SVTs using "-IncludeAdminControls" switch

By default organization and project control is not including in scan for non-admin users. To scan organization and project controls, non-admin users needs to add a switch '-IncludeAdminControls' with scan command. 

```PowerShell
#Scan organization and Project (non-admin users)
Get-AzSKADOSecurityStatus -OrganizationName "<OrganizationName>" -ProjectNames "<PRJ1,PRJ2,etc>" -ScanAllArtifacts -IncludeAdminControls
```
----------------------------------------------

### Execute SVTs using "-DetailedScan" switch

A special flag -DetailedScan in the scan command which can be used to tell the scanner to query and display richer information when evaluating certain controls. This is “off by default” and helps us scan RBAC controls at scale by avoiding API calls that can be deferred to a fix stage. 
```PowerShell
Get-AzSKADOSecurityStatus -OrganizationName "<OrganizationName>" -ScanAllArtifacts -DetailedScan
```
Detailed information is also generated when -ControlIds or -ControlsToAttest flag is used. At present, the following controls support this flag: 
- ADO_Build_AuthZ_Grant_Min_RBAC_Access
- ADO_Release_AuthZ_Grant_Min_RBAC_Access
- ADO_Organization_AuthZ_Justify_Guest_Identities

----------------------------------------------

### Execute SVTs using "-UsePartialCommits" switch

The Get-AzSKADOSecurityStatus command now supports checkpointing via a "-UsePartialCommits" switch. When this switch is used, the command periodically persists scan progress to disk. That way, if the scan is interrupted or an error occurs, a future retry can resume from the last saved state. This capability also helps in Continuous Assurance scans if scan gets suspended due to any unforeseen reason.The cmdlet below checks security control state via a "-UsePartialCommits" switch:
Get-AzSKADOSecurityStatus-OrganizationName "<OrganizationName>" -ScanAllArtifacts -UsePartialCommits
