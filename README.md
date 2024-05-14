# <img src="https://media0.giphy.com/media/pylpD8AoQCf3CQ1oO2/giphy.gif" width=30 height=30> KQL Query Quest (work in progress)

Your self-guide to crafting better queries. Uncover tips, tricks, and insights to level up your skills 🚀

KQL stands for "Kusto Query Language." It's a query language developed by Microsoft for use with Azure Data Explorer and other Microsoft services like Azure Monitor and Azure Log Analytics.

In simple terms, KQL is a language that allows you to ask questions and extract information from large datasets efficiently. It enables you to search, filter, and manipulate data to find specific answers to your queries.

For example, if you're analyzing server logs in a cloud environment, you can use this to write queries that help you find relevant information such as any errors that occurred, resource usage patterns, or trends over time.

In summary, KQL is a powerful tool for analyzing large datasets and extracting useful insights from them.

learn more: https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query

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

- Number of requests per country/region of origin, including the use of Alias for return fields
```
requests
| summarize Qtde=count() by AreaOrigem=client_CountryOrRegion
```

- Pie chart with requests by country/region of origin
```
requests
| summarize Qtde=count() by CidadeOrigem=client_CountryOrRegion 
| render piechart
```

- Simple query with number of requests per city of origin
requests
```
| summarize count() by client_City
```

- Query with number of requests per city of origin including the use of Alias for the requests return fields
```
| summarize Qtde=count() by CidadeOrigem=client_City 
```

- Pie chart with requests by city of origin
```
requests
| summarize Qtde=count() by CidadeOrigem=client_City 
| render piechart
```

- Simple query with number of requests per container
```
requests
| summarize count() by cloud_RoleInstance 
```

- Number of requests per container, including the use of Alias for return fields
```
requests
| summarize Qtde=count() by Container=cloud_RoleInstance 
```

- Number of requests and average response time per container, including the use of Alias for return fields
```
requests
| summarize Qtde=count(), TempoMedio=avg(duration) by Container=cloud_RoleInstance 
```

- Pie chart with requests per container
```
requests
| summarize Qtde=count() by Container=cloud_RoleInstance 
| render piechart
```

- Aggregates the total count of requests by client city and renders the result as a pie chart
```
requests
| summarize totalCount=sum(itemCount) by client_City
| render piechart
```

- Retrieves dependency telemetry data for the "GET Bolsas/Get" operation where the duration of the operation is greater than 150 milliseconds
```
dependencies 
| limit 1000 
| where operation_Name == "GET Bolsas/Get" and duration > 150 
| project operation_Name, timestamp, data, duration 
```

- Counting the total number of requests made, separating them into successful and failed ones. It also calculates the number of impacted users. The results are grouped by operation name, URL, and source, then shown in a pie chart based on the total number of requests
```
requests
| where 
//(operation_Name contains_cs "app-name-xxx" or operation_Name contains_cs "app-name-xxx")
and (url contains "ambiente-hml" or url contains "caogs")
| summarize TotalRequests = count(), 
            SuccessfulRequests = sum(iff(success == true, 1, 0)),
            FailedRequests = sum(iff(success == false, 1, 0)),
            ImpactedUsers = dcount(user_Id) by operation_Name, url, source
| order by TotalRequests desc
| render piechart
```

- Filters request data for the past 30 days and then summarizes it. It counts the total number of requests, separates them into successful and failed requests based on the success field, and groups the results by the operation name. The summarized data is then ordered based on the total number of requests in descending order and displayed in a table format
```
requests
| where timestamp >= ago(30d)
| summarize
    TotalRequests = count(), 
    SuccessfulRequests = countif(success == true),
    FailedRequests = countif(success == false)
    by operation_Name
| order by TotalRequests desc
| render table
```

- This query filters request data from the last 1 day and then summarizes it based on performance buckets. It counts the number of requests in each performance bucket and orders the results in ascending order based on the performance bucket. Finally, the summarized data is visualized as a pie chart
```
requests
| where timestamp > ago(1d)
| summarize count() by performanceBucket
| order by performanceBucket asc
| render piechart 
```

- This query filters request data from the 1 day hours and then calculates the total count of requests. It then categorizes the requests into different performance buckets based on their duration. After summarizing the count of requests in each performance bucket, it calculates the percentage of requests in each bucket relative to the total count. Finally, it projects the bucket names, count, and percentage of requests into a pie chart for visualization.
```
let dataSet = requests
| where timestamp > ago(1d);
let totalCount = toscalar(dataSet | count);
dataSet
| extend requestPerformanceBucket = 
    iff(duration <= 300, "0-300ms",
    iff(duration <= 600, "301-600ms",
    iff(duration <= 1500, "601ms-1.5s",
    iff(duration <= 2000, "1.501s-2s",
    iff(duration <= 4000, "2.001s-4s", ">4s")))))
| summarize Count = count() by requestPerformanceBucket
| extend Percentage = strcat(round(Count / totalCount * 100, 2), "%")
| project Bucket = requestPerformanceBucket, Count, Percentage
| render piechart
```

```
let AllAPIs = datatable(APIName:string) 
[
    "api-abc-qa",
    "api-abc-dev",
    "api-deploy-dev",
    "api-chdeploy-qa"
] | extend APIName = tostring(APIName);
let ConsumedAPIs =
    requests
    | where timestamp > ago(7d)
    | extend APIName = strcat(tostring(split(operation_Name, ";")[0]))
    | summarize
        TotalRequests = count(), 
        SuccessfulRequests = countif(success == true),
        FailedRequests = countif(success == false)
    by APIName;
AllAPIs
| join kind=leftouter (ConsumedAPIs) on APIName
| project APIName, TotalRequests = iff(isnotempty(TotalRequests), TotalRequests, 0), SuccessfulRequests = iff(isnotempty(SuccessfulRequests), SuccessfulRequests, 0), FailedRequests = iff(isnotempty(FailedRequests), FailedRequests, 0)
```