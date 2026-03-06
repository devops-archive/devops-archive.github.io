# New Relic Logs - NRQL Tricks

To list all traffic from the individual IP Address based on the count of traffic received.

```
FROM Log 
SELECT count(*) 
WHERE filePath = '/var/log/platform/magento_cloud_project_id/access.log' 
  AND ADBE_Environment = 'production' 
FACET capture(message, r'^(?P<client_ip>[^\s]+).*') 
SINCE 1 hour ago
```


To list all HTTP 500 errors from individual IP Address to "/graphql" URL based on the count of request received.

```
FROM Log 
SELECT count(*) 
WHERE filePath = '/var/log/platform/qzwe4gmdjp2mg/access.log' 
  AND ADBE_Environment = 'production'
  AND message LIKE '%/graphql%'
  AND message LIKE '%" 500 %'
FACET capture(message, r'^(?P<client_ip>[^\s]+).*') 
SINCE 1 hour ago
```
