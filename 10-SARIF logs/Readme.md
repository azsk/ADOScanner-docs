## Generating Logs in SARIF Format


Static Analysis Results Interchange Format is a static tools format approved as an OASIS Standard. ADO Scanner provides support to generate SARIF logs in both CA and SDL scanning modes. The switch to generate SARIF logs is -GenerateSarifLogs. An alias -gsl can also be used to get the same functionality. This switch can be added on top of any existing GADS command.

```PowerShell
        $orgName = '<name of ADO org>'
	Get-AzSKADOInfo -OrganizationName $orgName -ProjectName $projName -GenerateSarifLogs
```

These logs can be used with any SARIF tool available online. They can be used with SARIF Viewers to enhance user viewing and understanding of control failures. It can also be used with the SARIF npm sdk to log bugs on work Items, match control results with previous results and many other features.

# Log bugs as work items.

Image of Web SARIF Viewer 

Link to github repo for SARIF Sdk 