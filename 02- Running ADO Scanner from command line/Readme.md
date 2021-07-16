# AzSK.ADO PowerShell module

## Contents
 -  [Scan your Azure DevOps resources](Readme.md#scan-your-azure-devops-resources)
 -  [Execute SVTs using "-IncludeAdminControls" switch](Readme.md#execute-svts-using--includeadmincontrols-switch)
 -  [Execute SVTs using "-DetailedScan" switch](Readme.md#execute-svts-using--detailedscan-switch)
 -  [Execute SVTs using "-UsePartialCommits" switch](Readme.md#execute-svts-using--usepartialcommits-switch)
  	 - [Speed up checkpointed scans with "-DoNotRefetchResources" switch](Readme.md#speed-up-checkpointed-scans-with--donotrefetchresources-switch) 
 -  [Execute path based scanning for builds and releases](Readme.md#execute-path-based-scanning-for-builds-and-releases)

## Scan your Azure DevOps resources

Run the command below after replacing `<OrganizationName>` with your Azure DevOps org Name 
and `<PRJ1, PRJ2, ..`> with a comma-separated list of project names where your Azure DevOps resources are hosted.
You will get Organization name from your ADO organization url e.g. "http<i></i>//sampleadoorg.visualstudio.com". In this 'sampleadoorg' is org name.

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
Get-AzSKADOSecurityStatus -OrganizationName "<OrganizationName>" -ScanAllResources

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

### Execute SVTs using "-IncludeAdminControls" switch

By default organization and project control is not including in scan for non-admin users. To scan organization and project controls, non-admin users needs to add a switch '-IncludeAdminControls' with scan command. 

```PowerShell
#Scan organization and Project (non-admin users)
Get-AzSKADOSecurityStatus -OrganizationName "<OrganizationName>" -ProjectNames "<PRJ1,PRJ2,etc>" -ScanAllResources -IncludeAdminControls
```
----------------------------------------------

### Execute SVTs using "-DetailedScan" switch

A special flag -DetailedScan in the scan command which can be used to tell the scanner to query and display richer information when evaluating certain controls. This is “off by default” and helps us scan RBAC controls at scale by avoiding API calls that can be deferred to a fix stage. 
```PowerShell
Get-AzSKADOSecurityStatus -OrganizationName "<OrganizationName>" -ScanAllResources -DetailedScan
```
Detailed information is also generated when -ControlIds or -ControlsToAttest flag is used. At present, the following controls support this flag, there detailed information can be seen below : 
|Control ID|Detailed Information|
|----------|--------------------|
|ADO_Organization_SI_Review_Installed_Extensions|List of extensions that: <br>(a) have not been updated by publishers for more than 2 years.<br> (b) have sensitive access permissions.<br> (c) are not production ready/in preview.<br> (d) are developed by top publishers in marketplace.<br> (e) have private visibility.|
|ADO_Build_AuthZ_Grant_Min_RBAC_Access|List of identities with their name and permissions that have been provided with minimum RBAC access to associated pipeline|
|ADO_Release_AuthZ_Grant_Min_RBAC_Access|List of identities with their name and permissions that have been provided with minimum RBAC access to associated pipeline|
|ADO_Organization_AuthZ_Justify_Guest_Identities|List of guest identities and their respective project entitlements in the organization.|


----------------------------------------------

### Execute SVTs using "-UsePartialCommits" switch

The Get-AzSKADOSecurityStatus command now supports checkpointing via a "-UsePartialCommits" switch. When this switch is used, the command periodically persists scan progress to disk. That way, if the scan is interrupted or an error occurs, a future retry can resume from the last saved state. This capability also helps in Continuous Assurance scans if scan gets suspended due to any unforeseen reason.The cmdlet below checks security control state via a "-UsePartialCommits" switch:
```PowerShell
Get-AzSKADOSecurityStatus-OrganizationName "<OrganizationName>" -ScanAllResources -UsePartialCommits
```

#### Speed up checkpointed scans with "-DoNotRefetchResources" switch
The "-UsePartialCommits" switch also supports an optional switch: "-DoNotRefetchResources" in SDL mode. When this switch is used, resources are not re-fetched during the continuation of the checkpointed scan (i.e., when the "-upc" switch is used). This efficiently speeds up scans of subsequent batches after the initial one. Currently the resources supported with the switch are Release, Agent Pool, Organization and Project. 

```PowerShell
Get-AzSKADOSecurityStatus -OrganizationName "<OrganizationName>" -ProjectName "<ProjectName>" -ReleaseNames * -ResourceTypeName Release -UsePartialCommits -DoNotRefetchResources
```

----------------------------------------------

### Execute path based scanning for builds and releases
The Get-AzSKADOSecurityStatus command supports path based scanning by scanning build and release configs constrained to specific build and release folder paths. This is achieved via two switches : "-BuildsFolderPath" and "-ReleasesFolderPath".
```PowerShell
Get-AzSKADOSecurityStatus -OrganizationName "<OrganizationName>" -ProjectName "<ProjectName>" -ReleaseNames * -ResourceTypeName Release -ReleasesFolderPath "<ReleasesFolderPath>"

Get-AzSKADOSecurityStatus -OrganizationName "<OrganizationName>" -ProjectName "<ProjectName>" -BuildNames * -ResourceTypeName Build -BuildsFolderPath "<BuildsFolderPath>"
```
Consider the following build folder structure: </br>
<kbd>
<img  src="../Images/02_Folder_Structure.PNG"  alt="Folder structure">
</kbd>
 </br>
To scan builds inside "Folder 1", the path should be given as "Folder 1". This will scan all builds inside this folder (i.e., Build 1, Build 2 and Build 3). To scan all builds inside "Folder 2", the path should be "Folder 1\Folder 2". This will scan Build 1 and Build 2.

----------------------------------------------

### Execute SVTs using "-UseGraphAccess" switch

Some AzSK.ADO controls require graph access for correct evaluation. To fetch the graph access token for the user a special flag "-UseGraphAccess" should be used in the scan command. This switch is “off by default” and control results for such controls that depend on AAD group expansion may not be accurate.

```PowerShell
Get-AzSKADOSecurityStatus-OrganizationName "<OrganizationName>" -UseGraphAccess
```
