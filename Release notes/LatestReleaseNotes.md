# 220216 (AzSK.ADO v.1.16.0)

## Feature Updates

### ADO Security Scanner PowerShell Module

* Automated bulk remediation extended for:

  * ``ADO_AgentPool_DP_No_Secrets_In_Capabilities:``
  * ``ADO_AgentPool_DP_Enable_Auto_Update``

### Security controls updates

* New controls:
    The following new controls have been added :

  * ``ADO_ServiceConnection_AuthZ_Enable_Branch_Control``: Allow service connections to be accessed only by select branches.
  * ``ADO_VariableGroup_AuthZ_Enable_Branch_Control``: Allow variable groups to be accessed only by select branches.
  * ``ADO_SecureFile_AuthZ_Enable_Branch_Control``: Allow secure files to be accessed only by select branches.
  * ``ADO_Repository_AuthZ_Enable_Branch_Control``: Allow repositories to be accessed only by select branches.
  * ``ADO_Build_DP_Configure_YAML_CI_Triggers``: Use CI triggers to allow YAML CI only from select branches.

### Other Improvements/Bug fixes

* Service tree mapping of secure files and variable groups has been increased by 80% and 55% respectively by using Cloudmine data. The scan time for their mapping generation has also been improved.
* Standalone bug logging now supports resolution of identities from alternate mail addresses while fetching assignees.
* Additional Info for user baseline controls added.
