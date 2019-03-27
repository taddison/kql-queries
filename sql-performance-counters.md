### Recent Server CPU

```kql
let computer = 'prod-db';
let period = 3d;
let step = 15m;
Perf 
| where TimeGenerated > ago(period)
| where CounterName == "% Processor Time" and ObjectName == "Processor"
| where Computer contains computer
| summarize CPU = avg(CounterValue) by bin(TimeGenerated, step)
| render timechart 
```