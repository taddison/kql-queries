## Log Analytics Usage

```kql
workspace("primaryWorkspace").Usage
| where TimeGenerated > startofday(ago(30d))
| where IsBillable == true
| extend Workspace = "Primary"
| summarize sum(Quantity) by Solution, bin(TimeGenerated, 1d), DataType, Workspace
| union (
workspace("secondaryWorkspace").Usage
| where TimeGenerated > startofday(ago(30d))
| where IsBillable == true
| extend Workspace = "Secondary"
| summarize sum(Quantity) by Solution, bin(TimeGenerated, 1d), DataType, Workspace
)
| render timechart 
```kql

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