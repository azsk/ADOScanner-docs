## Generating Logs in SARIF Format


[Static Analysis Results Interchange Format](https://sarifweb.azurewebsites.net/) is a static tools' logging format approved as an OASIS Standard. ADO Scanner provides support to generate SARIF logs in both CA and SDL scanning modes. The switch to generate SARIF logs is -GenerateSarifLogs. An alias -gsl can also be used to achieve the same functionality. This switch can be added on top of any existing gads command. The logs are published as "logs.sarif" in the security results folder.   

```PowerShell
        $orgName=""
        $projName=""
	Get-AzSKADOInfo -OrganizationName $orgName -ProjectName $projName -GenerateSarifLogs
```

These logs can be used with any SARIF tool available online. They can be used with [SARIF viewers](https://sarifweb.azurewebsites.net/#Viewers) to enhance user viewing and understanding of control failures. It can also be used with the SARIF [Software Development Kits](https://github.com/microsoft/sarif-sdk) to perform many functions like logging bugs and comparing control results with previous logs.

#  Important SARIF commands for npm SDK

This requires setting up the SDK first. After the setup is complete you can run the following commands:

Creating Work Items using SARIF logs: 
```PowerShell
        $env:SarifWorkItemFilingPat= "PAT Token"
        npx @microsoft/sarif-multitool file-work-items pathToSarifLogs/logs.sarif --host-uri https://dev.azure.com/OrgName/ProjName --split PerResult

```

Compare previous run SARIF logs with freshly generated SARIF logs
```PowerShell
    npx @microsoft/sarif-multitool match-results-forward newSARIFLogs.sarif -r oldSarifLogs.sarif -o ComparedLogsOutput.sarif
```

For other useful commands with SARIF files please refer [here](https://github.com/microsoft/sarif-sdk/blob/main/docs/multitool-usage.md)

Image of SARIF in WebViewer

Image of SARIF Design document