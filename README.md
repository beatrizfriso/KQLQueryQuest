# KQLQueryQuest

<img src="https://media0.giphy.com/media/pylpD8AoQCf3CQ1oO2/giphy.gif" width=30 height=30> Your self-guide to crafting better queries. Uncover tips, tricks, and insights to level up your KQL skills 🚀

- Counting Successful Requests
```
let startDateTime = datetime(2024-04-01);
let endDateTime = datetime(2024-04-04);
let successStatusCode = 200;
requests
| where timestamp >= startDateTime and timestamp <= endDateTime
| where resultCode == successStatusCode
| summarize Count = count() by bin(timestamp, 1d)
| order by timestamp asc
```

- Count the number of failed requests in the last hour
```
let agoDuration = 1h;
let failedStatusCode = 500;
requests
| where timestamp >= ago(agoDuration)
| where resultCode == failedStatusCode
| summarize Count = count()
```

- Count the number of failed requests in the last day
```
let agoDuration = 1d;
let failedStatusCode = 500;
requests
| where timestamp >= ago(agoDuration)
| where resultCode == failedStatusCode
| summarize Count = count()
```


