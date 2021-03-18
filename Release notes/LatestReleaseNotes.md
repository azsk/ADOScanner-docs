# 210315 (AzSK.ADO v.1.5.0)

## Feature Updates

### ADO Security Scanner PowerShell Module:
* [Preview] OAuth app-based Continuous Assurance (CA) using Azure Functions: 
    * Added the ability to scan ADO resources using an OAuth app for continuous assurance. This is being designed as an alternative to the current personal access token (PAT) based access model. 
    * This is currently in preview and we are working on fine-tuning the OAuth scopes needed by the app. Detailed instructions will be posted [here](https://aka.ms/adoscanner/ca/oauth) by tomorrow.

* Secure score for ADO extensions: The ADO_Organization_SI_Review_Installed_Extensions control now supports a scoring scheme for installed extensions to serve as guidance for organization admins for security evaluation and cleanup of installed extensions. 
    * The secure score is evaluated as a weighted sum of the following parameters: extensions that (a) have not been updated by publishers for more than 2 years, (b) have sensitive access permissions, (c) are not production ready/in preview, (d) are developed by top publishers in marketplace, and (e) have private visibility. 
    * ```[orgName]_ExtensionInfo.csv```  (present in the scan report folder) will contain a column representing the secure score for each extension in the org.
        ```Powershell
        gads -oz $org -ControlIds 'ADO_Organization_SI_Review_Installed_Extensions' -IncludeAdminControls -DetailedScan
        ```
* Multi-project control attestation: Added support to store control attestation details of resources (located across multiple projects) in a single repo in a designated project. Earlier, each project required its own repo for storing attestation info.
* Auto bug logging improvements: Added support for org-specific customization of bug description (to better represent compliance drive requirements such as documentation, support, reference, FAQ etc.). 

### Security controls updates
* New controls:
    * None in this release.

* Changes to existing controls:
    * Service connection controls have been expanded to cover all service connection types in ADO. Earlier, the scanner used to evaluate only 10 types (Azure classic, Azure RM, git, GitHub, Azure Repos/TFS, npm, NuGet, ESRP CodeSigning, ESRP Malware Scanning and generic).

    * Updated recommendation of the control ADO_ServiceConnection_AuthZ_Dont_Grant_All_Pipelines to better describe the steps especially for Azure-based service connections. With this change, a user who doesn’t have access to the Azure subscription can still fix the issue.
* Other improvements/bug fixes
    * Replaced many portal-based APIs used across the module with their documented counterparts.
    * Fixed an issue in bug logging to correctly use the specified area path mentioned in the command. Earlier, irrespective of whether a particular area path was specified, bugs were logged for resources in area paths linked to their respective service.
    * Added support to send info about inactivity period (number of days from the last known usage) of agent pools, pipelines and service connections using the ‘InactiveFromDays’ field in LA events.

