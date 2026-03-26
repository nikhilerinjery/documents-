```
fields @timestamp, @message
| filter @message like /ERROR/ and @message like /System.IndexOutOfRangeException/
| sort @timestamp desc
| limit 100
```


service ```/rb/ecs/prod/rbcloud/api_service```
