## Log Analytics Usage

```kql
workspace("primaryWorkspace").Usage
| where TimeGenerated > startofday(ago(30d))
| where IsBillable
| extend Workspace = "Primary"
| summarize sum(Quantity) by Solution, bin(TimeGenerated, 1d), DataType, Workspace
| union (
workspace("secondaryWorkspace").Usage
| where TimeGenerated > startofday(ago(30d))
| where IsBillable
| extend Workspace = "Secondary"
| summarize sum(Quantity) by Solution, bin(TimeGenerated, 1d), DataType, Workspace
)
| render timechart
```

Drilling in to a single data type to investigate a spike in e.g. Event Log data...

```kql
Event
| where TimeGenerated > startofday(ago(30d))
| where _IsBillable == true 
| extend computerName = tolower(tostring(split(Computer, '.')[0]))
| where computerName != "" 
| summarize TotalVolumeBytes=sum(_BilledSize) by computerName, bin(TimeGenerated, 1d)
| render timechart
```

Which solutions and data types make up the majority of the data usage?

```kql
Usage
| where TimeGenerated > ago(30d)
| where IsBillable
| top-nested 1 of 'All' by AllData = round(sum(Quantity),0)
, top-nested 2 of Solution with others='Others' by SolutionTotal = round(sum(Quantity),0)
, top-nested 2 of DataType with others = 'Others' by SolutionDataTotal = round(sum(Quantity),0)
| where AllData != 0
| extend SolutionPct = round((SolutionDataTotal / SolutionTotal) * 100,1)
| extend OverallPct = round((SolutionDataTotal / AllData) * 100,1)
| project AllData, SolutionTotal, Solution, DataType, SolutionPct, SolutionDataTotal, OverallPct
| order by OverallPct desc
```
