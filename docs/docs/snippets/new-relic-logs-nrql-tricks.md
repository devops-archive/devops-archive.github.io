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
WITH aparse(message, '* - - [*] "* * *" * * "*" "*"') AS (Client_IP, LogTime, Method, URL, Protocol, Status, Bytes, Referer, UserAgent)
SELECT 
  Client_IP AS 'IP Address',
  LogTime AS 'Log Timestamp',
  Method AS 'HTTP Method',
  URL AS 'Requested URL',
  Status AS 'Status Code',
  Referer AS 'Referer',
  UserAgent AS 'UserAgent'
WHERE filePath = '/var/log/platform/magento_cloud_project_id/access.log'
  AND message LIKE '123.12.123.123%'
  AND message NOT LIKE '%/graphql'
SINCE 12 hours ago 
LIMIT 5
```

### Using capture()

Using the NRQL `capture()` function:

```
FROM Log
WITH capture(message, r'^(?P<Client_IPADDR>[^ ]+) - - \[(?P<Timestamp>[^\]]+)\] "(?P<Method>[A-Z]+) (?P<URL>[^ ]+) [^"]+" (?P<Status>\d{3}) (?P<Bytes>[^ ]+) "(?P<Referer>[^ ]+)" "(?P<UserAgent>([^ ]+) [^"]+)"') AS parsedLog
SELECT 
  parsedLog[Client_IPADDR] AS 'IP Address',
  parsedLog[Timestamp] AS 'Log Timestamp',
  parsedLog[Method] AS 'HTTP Method',
  parsedLog[URL] AS 'Requested URL',
  parsedLog[Status] AS 'Status Code',
  parsedLog[Referer] AS 'HTTP Referer',
  parsedLog[UserAgent] AS 'User Agent'
WHERE filePath = '/var/log/platform/magento_cloud_project_id/access.log'
  AND parsedLog[Client_IPADDR] = '123.12.123.123'
  AND parsedLog[URL] NOT LIKE '%/graphql'
SINCE 30 minutes ago
LIMIT 5
```
