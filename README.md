# <img src="https://media0.giphy.com/media/pylpD8AoQCf3CQ1oO2/giphy.gif" width=30 height=30> KQL Query Quest (work in progress)

Your self-guide to crafting better queries. Uncover tips, tricks, and insights to level up your skills 🚀

KQL stands for "Kusto Query Language." It's a query language developed by Microsoft for use with Azure Data Explorer and other Microsoft services like Azure Monitor and Azure Log Analytics.

In simple terms, KQL is a language that allows you to ask questions and extract information from large datasets efficiently. It enables you to search, filter, and manipulate data to find specific answers to your queries.

For example, if you're analyzing server logs in a cloud environment, you can use this to write queries that help you find relevant information such as any errors that occurred, resource usage patterns, or trends over time.

In summary, KQL is a powerful tool for analyzing large datasets and extracting useful insights from them.

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

<img align="center" alt="GIF" src="https://media0.giphy.com/media/v1.Y2lkPTc5MGI3NjExMmQ1Z3RiaXplc25qczcxZXQwNTJrejdncDZyYWJnNWV1aGlod2QyNSZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/fUjauzDjbxd77fPyzp/giphy.gif" />

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

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


