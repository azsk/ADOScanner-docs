# 211115 (AzSK.ADO v.1.13.0)

## Feature Updates

### ADO Security Scanner PowerShell Module:
* Automated bulk remediation extended for:
    * ADO_Organization_AuthN_Use_ALT_Accounts_For_Admin
    * ADO_Organization_AuthZ_Restrict_Broader_Group_Access_on_Feed
    * ADO_Project_AuthN_Use_ALT_Accounts_For_Admin
    * ADO_ServiceConnection_AuthZ_Dont_Grant_All_Pipelines_Access
    * ADO_SecureFile_AuthZ_Dont_Grant_All_Pipelines_Access
    * ADO_VariableGroup_AuthZ_Dont_Grant_All_Pipelines_Access_On_VG_With_Secrets				

* Incremental Scan has been extended to cover the following resource types in addition to builds and releases:
	* Variable Groups
	* Feeds
	* Repositories
	* Secure files
    * Environments

* Incremental Scan can now be used via ADO Security Scanner extension as well.

### Security controls updates
* New controls:
    The following new control have been added in this release:
	* ``ADO_Organization_AuthZ_Restrict_Feed_Create_Permission``: Allow only limited group of users permission to create feeds in the organization.
	* ``ADO_Feed_SI_Review_Inactive_Feeds``: Inactive feeds must be removed if no more required.

* Changes to existing controls :
	* ``ADO_Organization_AuthN_Use_ALT_Accounts_For_Admin``: The control now evaluates any human accounts added in Project Collection Service Accounts for SC-ALT check, in addition to PCA groups. Service accounts are not evaluated. 

### Other Improvements/Bug fixes
* Corrected logic to determine a service account while bug logging
* While evaluating ``ADO_Build_SI_Dont_Use_Broadly_Editable_Variable_Group control`` if any variable group is already checked for editability, then it will not be checked again for other build resources.