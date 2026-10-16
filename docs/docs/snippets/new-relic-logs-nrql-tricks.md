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
WHERE filePath = '/var/log/platform/magento_cloud_project_id/access.log' 
  AND ADBE_Environment = 'production'
  AND message LIKE '%/graphql%'
  AND message LIKE '%" 500 %'
FACET capture(message, r'^(?P<client_ip>[^\s]+).*') 
SINCE 1 hour ago
```

## Modify the returned output

If we need to modify the output returned by the above NRQL query—for example, to transform the raw access log in the `message` field into a tabular view containing only the fields we are interested in—we can reformat the output accordingly.

### Using aparse()

Using the NRQL `aparse()` function.

```
FROM Log
WITH aparse(message, '* - - [*] "* * *" * *') AS (Client_IP, LogTime, Method, URL, Protocol, Status, Bytes)
SELECT 
  LogTime AS 'Log Timestamp',
  Method AS 'HTTP Method',
  URL AS 'Requested URL',
  Status AS 'Status Code',
  capture(message, r'.*?"\s*"(?P<User_Agent>[^"]+)"$') AS 'User Agent'
WHERE filePath = '/var/log/platform/magento_cloud_project_id/access.log'
  AND message LIKE '123.12.123.123%'
  AND message NOT LIKE '%cookware.html%'
SINCE 12 hours ago 
LIMIT 5
```

### Using capture()

Uisng the NRQL `capture()` function:

```
FROM Log
WITH capture(message, r'^[^ ]+ - - \[(?P<Timestamp>[^\]]+)\] "(?P<Method>[A-Z]+) (?P<URL>[^ ]+) [^"]+" (?P<Status>\d{3}) .*') AS parsedLog
SELECT 
  parsedLog[Timestamp] AS 'Log Timestamp',
  parsedLog[Method] AS 'HTTP Method',
  parsedLog[URL] AS 'Requested URL',
  parsedLog[Status] AS 'Status Code',
  capture(message, r'.*?"\s*"(?P<User_Agent>[^"]+)"$') AS 'User Agent'
WHERE filePath = '/var/log/platform/magento_cloud_project_id/access.log'
  AND message LIKE '123.12.123.123%'
  AND message NOT LIKE '%cookware.html%'
SINCE 12 hours ago 
LIMIT 5
```
