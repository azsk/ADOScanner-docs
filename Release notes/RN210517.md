# 210517 (AzSK.ADO v.1.7.0)

## Feature Updates

### ADO Security Scanner PowerShell Module:
* [Preview] OAuth app-based Continuous Assurance (CA) using Azure Functions: 
    * Added support to query the health and other info about the OAuth app-based setup using ```Get-AzSKADOContinuousAssurance``` cmdlet. 
    * OAuth app refresh token can now be renewed on demand using -RefreshOAuthToken switch in ```Update-AzSKADOContinuousAssurance``` cmdlet.
    * ```Install-AzSKADOContinuousAssurance``` cmdlet will accept app id and client secret only in secure string format.
    * Added support to auto-renew refresh token stored in key vault every 30 days. Earlier, it was being renewed during every scan.
    * This is currently in preview. Detailed instructions are available [here](https://aka.ms/adoscanner/ca/oauth).

* Auto bug logging: 
    * Bugs can now be auto-closed towards end of the scan using ```–AutoCloseBugs``` switch in scan cmdlet. Earlier, bugs were auto closed only when bug logger was triggered.
    * Added policy project to scan command in bug description if specified while logging bugs. 

* Added ```–UseGraphAccess``` switch in scan cmdlet to correctly evaluate controls requiring graph access. This capability is used to distinguish between human and service accounts in admin roles and exclude service accounts from consideration for the controls below:
    * ADO_Organization_BCDR_Min_Admin_Count
    * ADO_Organization_AuthZ_Limit_Admin_Count
    * ADO_Project_BCDR_Min_Admin_Count
    * ADO_Project_AuthZ_Limit_Admin_Count

* ADO organization and user info: 
    *Added support to list (a) all resources in an org and (b) authorized OAuth apps and personal access tokens (PAT) of a user, via the ```Get-AzSKADOInfo``` command using ```OrganizationInfo``` and ```UserInfo``` types, respectively.
    ```Powershell
        Get-AzSKADOInfo -OrganizationName $orgName –InfoType OrganizationInfo
        Get-AzSKADOInfo -OrganizationName $orgName -ProjectName $projName –InfoType UserInfo
    ```
### Security controls updates
* New controls:
    * The ```ADO_ServiceConnection_Cannot_Use_Disallowed_Environment``` control restricts service connections from connecting to certain Azure environments as configured by the org.
    * The ```ADO_Project_Check_Inactive_Project``` control flags projects with no sign of development activity in recent past. This control evaluates an aggregation of rules that infer inactivity of pipelines, service connections, agent pools etc.

* Changes to existing controls:
    * Fixed an issue in the ```ADO_Release_DP_No_PlainText_Secrets_In_Definition``` control which was taking longer time to evaluate in scenarios where pipeline definition was referencing deleted variable groups.
    * The ```ADO_Project_DP_Inactive_Repos``` control will treat disabled repositories as inactive. 

### Other improvements/bug fixes
* Resolved an issue where stateful attestations were not being respected in CA scans in Linux environment due to encoding discrepancy.
* Added capability to extend attestation expiry of resources that need by-design exemptions from the backend.



