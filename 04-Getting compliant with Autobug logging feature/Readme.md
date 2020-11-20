### [Auto Bug Logging](Readme.md#auto-bug-logging)
- [Overview](Readme.md#Overview)
- [Starting bug logging](Readme.md#starting-bug-logging)  
- [Setting up host project for organization specific controls](Readme.md#setting-up-host-project-for-organization-specific-controls)  
- [Defining area and iteration path](Readme.md#defining-area-and-iteration-path)  
- [Determining the assignee of the bug](Readme.md#determining-the-assignee-of-the-bug) 
- [Customizing resolved bug behaviour](Readme.md#customizing-resolved-bug-behaviour) 
- [Auto close bugs](Readme.md#auto-close-bugs) 
- [Permissions required for bug logging](Readme.md#permissions-required-for-bug-logging) 

# Auto bug logging
## Overview

The auto bug logging feature facilitates user to identify and keep  track of security control failures in ADO resources. Whenever a control failure is surfaced by the ADO Security Scanner, a bug will be logged in the ADO work items that would contain all the relevant details and remediation steps. The bug is assigned to the most relevant user (admin/creator/last known consumer) for the resource.

Duplicate bugs are not logged again and are available in the summary as "active" bugs. 
Upon the completion of all control scans, all passing controls whose bugs had been logged previously are closed automatically as well.

## Starting bug logging

The bug logging feature is implemented via a new switch called *-AutoBugLog*. All controls that emit an evaluation status as "failed" or "verify" are considered as valid targets for bug logging. A typical security scan command with auto bug logging feature would look as :

```Powershell
#Setting attestation host project and auto bug logging for organization controls
Get-AzSKADOSecurityStatus -OrganizationName "<OrganizationName>"  -ProjectNames "<ProjectName>" -ResourceTypeName Project -AutoBugLog All -AreaPath "<Area Path>" -IterationPath "<Iteration Path"
```
To leverage the bug logging feature, following are the scan parameters that need to be included along with any security scan command.
| Param Name | Purpose | Required?| Possible Values| 
|--|--|--|--|
|  AutoBugLog|To enable bug logging and identify which subset of control failures are to be logged as bugs  | TRUE| All, BaselineControls, PreviewBaselineControls| 
|  AreaPath|To specify the area path where the bugs are to be logged  | FALSE| Valid area path in ADO| 
|  IterationPath|To specify the iteration path where the bugs are to be logged  | FALSE| Valid iteration path in ADO| 

The switch *-AutoBugLog*  takes up three values that specify which subset of control failures are to be logged. These are:
| AutoBugLog option |  Description|
|--|--|
| All | Log all control failures |
| BaselineControls | Log only baseline control failures|
| PreviewBaselineControls | Log only preview baseline control failures|

## Setting up host project for organization specific controls

All organization control failures are logged in the work items of the host project. This host project is the same one which is used in the attestation process. If you haven't configured host project during attestation, you can do it during bug logging as well using the flag *-AttestationHostProjectName*. Read about attestation [here](Readme.md#control-attestation-1).
> **Note:** Attestation host project can only be set once, so this project will be used for both attestation and bug logging and can't be updated later.

```Powershell
#Setting attestation host project and auto bug logging for organization controls
Get-AzSKADOSecurityStatus -OrganizationName "<OrganizationName>" -AttestationHostProjectName "<Host Project Name>" -AutoBugLog All
```
See the examples below for auto bug logging of organization, project, build, release, service connection and agent pools control failures :
```Powershell
#Auto bug logging for organization controls
Get-AzSKADOSecurityStatus -OrganizationName "<OrganizationName>" -ResourceTypeName Organization -AutoBugLog All

#Auto bug logging for organization baseline controls
Get-AzSKADOSecurityStatus -OrganizationName "<OrganizationName>" -ResourceTypeName Organization -AutoBugLog BaselineControls

#Auto bug logging for organization preview baseline controls
Get-AzSKADOSecurityStatus -OrganizationName "<OrganizationName>" -ResourceTypeName Organization -AutoBugLog PreviewBaselineControls

#Auto bug logging for project controls
Get-AzSKADOSecurityStatus -OrganizationName "<OrganizationName>" -ProjectNames "<ProjectName>" -ResourceTypeName Project -AutoBugLog All

#Auto bug logging for build controls
Get-AzSKADOSecurityStatus -OrganizationName "<OrganizationName>" -ProjectNames "<ProjectName>" -BuildNames "<BuildName>" -ResourceTypeName Build -AutoBugLog All

#Auto bug logging for release controls
Get-AzSKADOSecurityStatus -OrganizationName "<OrganizationName>" -ProjectNames "<ProjectName>" -ReleaseNames "<ReleaseName>" -ResourceTypeName Release -AutoBugLog All

#Auto bug logging for agent pool controls
Get-AzSKADOSecurityStatus -OrganizationName "<OrganizationName>" -ProjectNames "<ProjectName>" -AgentPoolNames "<AgentPoolName>" -ResourceTypeName AgentPool -AutoBugLog All

#Auto bug logging for service connection controls
Get-AzSKADOSecurityStatus -OrganizationName "<OrganizationName>" -ProjectNames "<ProjectName>" -ServiceConnectionNames "<ServiceConnectionName>" -ResourceTypeName ServiceConnection -AutoBugLog All
```
## Defining area and iteration path

The user can control the area and iteration paths, where the bugs are to be logged, using two methods:

 1. **Specifying paths using scan parameters :** *-AreaPath* and *-IterationPath* are the two scan parameters to identify area and iteration paths respectively. These can be used along with the *-AutoBugLog* switch in a scan command.
  See the following examples:
```Powershell
#Specifying area and iteration paths
Get-AzSKADOSecurityStatus -OrganizationName "<OrganizationName>" -ProjectNames "<ProjectName" -AutoBugLog All -AreaPath "<AreaPath>" -IterationPath "<IterationPath>"
```
 2. **Customizing area and iteration path using org policy :** To leverage this method, make sure the org policy setup is complete in your project. Read more about org policy [here](Readme.md#setting-up-org-policy).
	 - Copy the ControlSettings.json from the AzSK.ADO installation to your org-policy repo.
	 - Remove everything except the "*BugLogAreaPath*" and the "*BugLogIterationPath*" line while keeping the JSON object hierarchy/structure intact.
```json
{
	"BugLogging": {
	  "BugLogAreaPath": "ProjectName\\AreaPath",
	  "BugLogIterationPath": "RootDefaultProject"
	}
}
```
3.  Commit the file.    
4.  Add an entry for *ControlSettings.json* in *ServerConfigMetadata.json* (in the repo) as shown below.

![Bug Logging Org Policy](../Images/ADO_BugLogging_OrgPolicy.png)

By default the values for both of the paths are RootDefaultProject that correspond to the root level of your project work items. In ADO that means the project name that has been supplied is the area and iteration path. While specifying any other path, make sure you escape characters such as "\\" to sanitize your JSON.

If no scan parameters are provided, the paths declared in the control settings will be used. If both scan parameters and org policies are specified, the org policies are overridden and paths mentioned in scan parameter are used.
### The bug logging summary
After the scan is complete and bugs have been logged, you will get a summary as follows:

![Bug Logging Summary](../Images/ADO_BugLogging_BugSummary.png)

There are three kinds of bugs shown in the summary:

 - **New Bug** : For freshly encountered control failures
 - **Active Bug** : For control failures that had been already logged and the controls are still failing.
 -  **Resolved Bug** : For bugs that had been resolved before, but the control failure has resurfaced. These bugs will be  reactivated by the scanner.
 
The details for all these bugs can be found in *BugSummary.json* file that is stored in the folder : *%LOCALAPPDATA%\Microsoft\AzSK.ADOLogs\Org_[yourOrganizationName]*. The information conveyed from this JSON looks as follows:

![Bug Logging JSON](../Images/ADO_BugLogging_BugJSON.png)

A sample bug template is shown as below :

![Bug Logging Template](../Images/ADO_BugLogging_BugTemplate.png)

The following information is conveyed by the bug template:
-   The control that has failed    
-   The control description    
-   Rationale behind why the control is important    
-   Recommendation on how to fix it    
-   Control scan command to reproduce the steps on your end    
-   Detailed logs of the command result    
-   Severity of the failing control    
-   The person to whom the bug is assigned    
-   The area and iteration paths where the bug is logged

## Determining the assignee of the bug

The bugs are assigned according to the resource type as follows:
| Feature| Assignee  |
|--|--|
| Organization | The user running the scan. Bug is logged only if the user is PCA (Project collection administrator) |
| Project | The user running the scan. Bug is logged only if the user is PA (Project administrator) |
| Build | Latest user to trigger the pipeline. If there has been no run history, it is assigned to creator|
|Release| Latest user to trigger the pipeline. If there has been no run history, it is assigned to creator |
| Service Connection| Owner of the service connection |
| Agent Pool | Owner of the agent pool |
| Variable Group | Owner of the variable group |


## Customizing resolved bug behaviour

Any bug that has been resolved before can be reactivated if the control failure resurfaces. Additionally, instead of reactivating, you can log the control failure as a newly created fresh bug and ignore the previously resolved bug. This can be changed using the org policies as follows:
1. **To reactivate resolved bugs :** This is the default behaviour and can be implemented by setting "*ResolvedBugBehaviour*" as "*ReactiveOldBug*" in your control settings using org policy.
```json
{
	"BugLogging": {
  	"ResolvedBugLogBehaviour": "ReactiveOldBug"
	}
}
```
2. **To create a new bug :** This can be implemented by setting "*ResolvedBugBehaviour*" as "*CreateNewBug*" in your control settings using org policy.
```json
{
	"BugLogging": {
	  "ResolvedBugLogBehaviour": "CreateNewBug"
	}
}
```
After setting any one of the above policies, commit the file and add an entry for *ControlSettings.json* in *ServerConfigMetadata.json* (in the repo).

## Auto close bugs

Using the *-AutoBugLog* switch, the scanner also evaluates all the passing control scans and checks for their corresponding bugs in the ADO. If such bugs are found, they are closed. This ensures only those bugs remain in your ADO work item whose control failures are to be fixed. 
For bugs logged against organization and project control failures, the user can control whether to auto close them or to leave them for manual inspection.
This can be controlled via the org policy as follows:

```json
{
	"BugLogging": {
	  "AutoCloseProjectBug": true,
	  "AutoCloseOrgBug": true
	}
}
```
The *AutoCloseProjectBug* and *AutoCloseOrgBug* can be set to true, in which case bugs corresponding to organization and project control failures will be auto closed. If this is set to false, all bugs except organization and project level bugs will be closed. By default the behaviour is to auto close every bug.

## Permissions required for bug logging

Bug logging is supported for organization and project controls only with admin privileges on organization and project, respectively.
>-   In order to log bugs for organization control, user needs to be a member of the group 'Project Collection Administrators'.    
>-   In order to log bugs for project control, user needs to be a member of the group 'Project Administrators' of that particular project.

[Back to top...](Readme.md#auto-bug-logging)
