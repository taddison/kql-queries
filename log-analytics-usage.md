## Log Analytics Usage

```kql
workspace("primaryWorkspace").Usage
| where TimeGenerated > ago(21d)
| where IsBillable == true
| extend Workspace = "Primary"
| summarize sum(Quantity) by Solution, bin(TimeGenerated, 1d), DataType, Workspace
| union (
workspace("secondaryWorkspace").Usage
| where TimeGenerated > ago(21d)
| where IsBillable == true
| extend Workspace = "Secondary"
| summarize sum(Quantity) by Solution, bin(TimeGenerated, 1d), DataType, Workspace
)
| render timechart 
```kql