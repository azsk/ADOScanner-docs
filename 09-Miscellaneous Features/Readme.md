## Miscellaneous Features

- [ADO Scanner information helper command](Readme.md#ado-scanner-information-helper-command)
- [Execute SVTs using "-AllowLongRunningScan" switch](Readme.md#Execute-SVTs-using--AllowLongRunningScan-switch)
- [Scan using -PolicyProject parameter](Readme.md#Scan-using--PolicyProject-parameter)
- [Scan using -Service Id parameter](Readme.md#Scan-using--ServiceId-parameter)

# ADO Scanner information helper command
### Overview

This command provides overall information about the ADO scanner which includes security controls information (severity, description, rationale, baseline etc.) and host information (ADO scanner settings/configuration, logged-in ADO user context etc.). 'Get-AzSKADOInfo' command can be used with 'InfoType' parameter to fetch information.

### Control information 

Run below command to get information about Azure DevOps security control(s). Control summary will be displayed on PS console by default. To get control information on PS console use -Verbose argument. 

```PowerShell
        $orgName = '<name of ADO org>'
	Get-AzSKADOInfo -OrganizationName $orgName `
                -InfoType 'ControlInfo' `
                [-ResourceTypeName <ResourceTypeName>] `
                [-ControlIds <ControlIds>] `
                [-UseBaselineControls] `
				[-UsePreviewBaselineControls] `
		[-ControlSeverity <ControlSeverity>] `
		[-ControlIdContains <ControlIdContains>] `
		[-Verbose]
```

|Param Name|Purpose|Required?|Default value|
|----|----|----|----|
|ResourceTypeName|Friendly name of resource type. E.g., Organization, Project, Build, Release etc.|FALSE|All|
|ControlIds|Comma-separated list of Control Ids|FALSE|None|
|UseBaselineControls|The flag used to get details of controls defined in baseline|FALSE|None|
|UsePreviewBaselineControls|The flag used to get details of controls defined in preview baseline|FALSE|None|
|ControlSeverity|Filter by severity of control E.g., Critical, High, Medium, Low|FALSE|None|
|ControlIdContains|Filter by ControlId(s) contains keyword|FALSE|None|
|Verbose|Get information on PS console|FALSE|None|


Below is the sample output:

Output of control details summary

![GADI_ControlInfo_Summary_PS](../Images/GADI_ControlInfo.png)  


### Host information  

Run below command to get information about,
* Loaded PS modules in PS session
* Logged in user's details
* AzSK ADO settings
* AzSK ADO configurations
* ADO context

```PowerShell
	$orgName = '<name of ADO org>'
	Get-AzSKADOInfo -OrganizationName $orgName `
                -InfoType 'HostInfo'
```

Below is the sample output:

![GADI_HostInfo_Summary_PS](../Images/GADI_HostInfo.png)  

----------------------------------------------

### Execute SVTs using "-AllowLongRunningScan" switch

To scan large number of project component (more then 1000 resources default value), you need to supply an additional switch parameter -AllowLongRunningScan in the command.
```PowerShell
Get-AzSKADOSecurityStatus -OrganizationName "<OrganizationName>" -ProjectNames "<PRJ1, PRJ2,...etc.>" -AllowLongRunningScan
```
Allowing scan for more then 1000 resources can be configured through the organization policy by updating 'IsAllowLongRunningScan' and 'LongRunningScanCheckPoint' properties in the ControlSettings.json file. 
If 'IsAllowLongRunningScan' is set to true, then by using '-AllowLongRunningScan' switch parameter, AzSK.ADO allows scan for resources count which is set in 'LongRunningScanCheckPoint'. If 'IsAllowLongRunningScan' value is set to false it does not allow scan for more then resources count set in 'LongRunningScanCheckPoint'.

----------------------------------------------

## Scan using -PolicyProject parameter

 Using -PolicyProject parameter you can specify the name of the project to read and write attestation details and fetch organization policy for organization.
 
 For example: 
```PowerShell  
#Using PolicyProject parameter
$orgName = '<Organization name>'
$policyProject = '<Name of the project hosting organization policy with which the scan should run.>'
Get-AzSKADOSecurityStatus -OrganizationName $orgName -PolicyProject $policyProject

#Attesting a control using PolicyProject parameter
Get-AzSKADOSecurityStatus -OrganizationName $orgName -PolicyProject $policyProject -ControlsToAttest NotAttested -ResourceTypeName Organization
```
----------------------------------------------

# Scan using -ServiceId parameter

Using the -ServiceId flag one can scan resources associated with a service in an organization. This is applicable if the organization provides a mapping of services to ADO resources (e.g., via a ‘Service Tree’ type repository of service metadata). The parameter can be used as follows:
```PowerShell 
Get-AzSKADOSecurityStatus -OrganizationName $orgName `
                             -ProjectNames $projectName `
                             -ServiceId $serviceId
```

