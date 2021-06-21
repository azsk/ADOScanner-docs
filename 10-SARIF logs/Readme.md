## Generating Logs in SARIF Format


Static Analysis Results Interchange Format is a static tools logging format approved as an OASIS Standard. ADO Scanner provides support to generate SARIF logs in both CA and SDL scanning modes. The switch to generate SARIF logs is -GenerateSarifLogs. An alias -gsl can also be used to achieve the same functionality. This switch can be added on top of any existing gads command. The logs are published as "logs.sarif" in the security results folder.   

```PowerShell
        $orgName = '<name of ADO org>'
	Get-AzSKADOInfo -OrganizationName $orgName -ProjectName $projName -GenerateSarifLogs
```

These logs can be used with any SARIF tool available online. They can be used with [SARIF viewers](https://sarifweb.azurewebsites.net/#Viewers) to enhance user viewing and understanding of control failures. It can also be used with the SARIF [Software Development Kits](https://github.com/microsoft/sarif-sdk) to perform many functions like logging bugs and comparing control results with previous logs.

# Log bugs as work items.

Image of Web SARIF Viewer 

Image of SARIF Design document


Link to github repo for SARIF Sdk 