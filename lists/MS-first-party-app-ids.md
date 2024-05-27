# Microsoft first-pary application IDs for Entra

Source: <https://learn.microsoft.com/en-us/troubleshoot/azure/entra/entra-id/governance/verify-first-party-apps-sign-in>

## Background and usage

Microsoft Entra Logs (e.g. SignIn and MicrosoftGraphActivityLogs) contain application IDs that are not necessarily from your own tenant, because these are Microsoft applications acting on behalf of your users. While application IDs are precise, they are not neccessarily very understandable for humans.

Luckily, KQL as used e.g. in Azure Sentinel or Advanced Hunting can ingest external data using the [`externaldata`](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/externaldata-operator?pivots=azuredataexplorer) operator.

Sample KQL query for Azure Monitor

```kql
// load external data from github
let ExternalData=(externaldata(ApplicationName:string,ApplicationID:string)["https://raw.githubusercontent.com/mlcirosec/Entra/main/lists/MS-first-party-app-ids.csv"] with (ignoreFirstRecord=true));
// enrich standard table with external data
MicrosoftGraphActivityLogs
| where TimeGenerated > ago(1d)
| join kind=leftouter ExternalData on $left.AppId == $right.ApplicationID
| project-away ApplicationID
| take 100
```

This query first loads the list of documented Microsoft first party app IDs from the above github-location and stores the result in the new table `ExternalData` naming the two columns `ApplicationName` and `ApplicationID` respectively.

After that, the MS graph activity logs are queried and the results joined with the information from the external data table. Since the `ApplicationID` column would be redundant, we remove it using `project-away`.

**Note**
When entering this query, the MS query editor may underline the _externaldata_ keyword. Hovering over it, shows a hint that this function is not suitable for large datasets. The referenced list however should not be treated as a large dataset, since it only holds a few hundred lines.
