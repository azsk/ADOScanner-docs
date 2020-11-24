# AzSK.ADO PowerShell module

## Contents
 -  [Import ADO module](Readme.md#import-ado-module)
 -  [Begin scanning your Organisation and Project](Readme.md#begin-scanning-your-organisation-and-project)
 -  [Support](Readme.md#Support)


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

Refer [link](/ControlCoverage) for current control coverage for Azure DevOps

### Support
- For any other issues or feedback please drop a mail to <a href="mailto:azskadosup@microsoft.com">ADO Scanner Support</a>
