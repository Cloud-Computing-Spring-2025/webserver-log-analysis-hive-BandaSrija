# Web-Server-Log-Analysis

### Overview

This project analyzes web server logs using Apache Hive to extract meaningful insights such as traffic trends, most visited pages, status code distribution, and suspicious activities. The data is stored in an external Hive table and queried using SQL-like commands.



### Steps to Execute
### Create table

CREATE EXTERNAL TABLE IF NOT EXISTS new_web_server_logs (
    ip STRING,
    `timestamp1` STRING,  
    url STRING,
    status INT,
    user_agent STRING
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE
LOCATION '/user/hive/web_logs/';


### Loading data

LOAD DATA INPATH '/user/hue/web_server_logs.csv' INTO TABLE web_server_logs;


###  Count Total Web Requests: Calculate the total number of requests processed.

SELECT CONCAT('Total Web Requests: ', COUNT(*)) AS total_requests
FROM web_server_logs;


###  Analyze Status Codes: Determine the frequency of HTTP status codes (e.g., 200, 404, 500).

SELECT status, COUNT(*) AS count
FROM web_server_logs
GROUP BY status
ORDER BY count DESC;


###  Identify Most Visited Pages: Extract the top 3 most visited URLs.

SELECT url, COUNT(*) AS count
FROM web_server_logs
GROUP BY url
ORDER BY count DESC
LIMIT 3;


### Traffic Source Analysis: Identify the most common user agents (browsers).

SELECT user_agent, COUNT(*) AS request_count
FROM web_server_logs
GROUP BY user_agent
ORDER BY request_count DESC
LIMIT 3;


###  Detect Suspicious Activity: Identify IP addresses with more than 3 failed requests (status 404 or 500)

SELECT ip, COUNT(*) AS failed_requests
FROM web_server_logs
WHERE status IN (404, 500)
GROUP BY ip
HAVING COUNT(*) > 3;



###  Analyze Traffic Trends: Calculate the number of requests per minute to observe traffic patterns.

SELECT SUBSTRING(timestamp1, 1, 16) AS minute_time, COUNT(*) AS request_count
FROM web_server_logs
GROUP BY SUBSTRING(timestamp1, 1, 16)
ORDER BY minute_time;


### Sample Input Data (web_server_logs.csv)
ip,timestamp1,url,status,user_agent
192.168.1.1,2024-02-01 10:15:30,/home,200,Mozilla/5.0
192.168.1.2,2024-02-01 10:15:45,/products,200,Chrome/90.0
192.168.1.3,2024-02-01 10:16:10,/home,200,Safari/13.1
192.168.1.4,2024-02-01 10:16:30,/checkout,404,Mozilla/5.0
192.168.1.5,2024-02-01 10:16:50,/home,200,Chrome/90.0
192.168.1.10,2024-02-01 10:17:05,/products,500,Mozilla/5.0
192.168.1.15,2024-02-01 10:17:20,/checkout,500,Safari/13.1


### Expected Output Examples

Total Web Requests:  
Total Requests: 7  

Status Code Analysis:
200: 4  
404: 1  
500: 2  

Identify Most Visited Pages
Most Visited Pages:
/home: 3  
/products: 2  
/checkout: 2  

Traffic Source Analysis:
Mozilla/5.0: 3  
Chrome/90.0: 2  
Safari/13.1: 2  

Suspicious IP Addresses:
192.168.1.10: 1 failed request  
192.168.1.15: 1 failed request  

Traffic Trend Over Time:
2024-02-01 10:15: 2 requests  
2024-02-01 10:16: 3 requests  
2024-02-01 10:17: 2 requests