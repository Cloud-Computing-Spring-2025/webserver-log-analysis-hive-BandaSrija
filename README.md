# Web-Server-Log-Analysis

### Overview

This project analyzes web server logs using Apache Hive to extract meaningful insights such as traffic trends, most visited pages, 
status code distribution, and suspicious activities. The data is stored in an external Hive table and queried using SQL-like commands.

### Implementation 

The analysis is performed using the following queries:

Count Total Web Requests: Counts the total number of requests received.

Analyze Status Codes: Identifies the frequency of HTTP status codes.

Identify Most Visited Pages: Extracts the top three most visited URLs.

Traffic Source Analysis: Identifies the most common user agents (browsers).

Detect Suspicious Activity: Finds IP addresses with more than three failed requests (404 or 500 errors).

Analyze Traffic Trends: Computes the number of requests per minute to observe traffic patterns.


### Implementation Approach

The implementation of this project follows a structured approach to analyze web server logs using Apache Hive. 
Each query serves a specific purpose in extracting valuable insights from raw log data. Below is a breakdown of how each 
analysis task is implemented:

# 1. Creating an External Table

The new_web_server_logs table is created using Apache Hive.
It stores log data with relevant attributes such as ip, timestamp1, url, status, and user_agent.
The table is external, meaning it does not manage the data directly but references files stored in HDFS.

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


2. Loading Data into Hive Table

The LOAD DATA command imports CSV log data into the Hive table.
The CSV file is stored in HDFS (/user/hue/web_server_logs.csv).

LOAD DATA INPATH '/user/hue/web_server_logs.csv' INTO TABLE web_server_logs;

3. Counting Total Web Requests

A simple COUNT(*) query is used to determine the total number of processed requests.

SELECT CONCAT('Total Web Requests: ', COUNT(*)) AS total_requests
FROM web_server_logs;


4. Analyzing Status Codes

This query groups logs by HTTP status codes (e.g., 200, 404, 500) and counts occurrences.
The ORDER BY clause ensures that the most frequent statuses appear first.

SELECT status, COUNT(*) AS count
FROM web_server_logs
GROUP BY status
ORDER BY count DESC;

5. Identifying the Most Visited Pages

The GROUP BY and COUNT(*) functions are used to rank the most frequently visited URLs.
The LIMIT 3 clause ensures that only the top 3 pages are displayed.

SELECT url, COUNT(*) AS count
FROM web_server_logs
GROUP BY url
ORDER BY count DESC
LIMIT 3;

6. Analyzing Traffic Sources (User Agents)

The user_agent column identifies different browsers used by visitors.
The query counts occurrences of each user agent and sorts them in descending order.

SELECT user_agent, COUNT(*) AS request_count
FROM web_server_logs
GROUP BY user_agent
ORDER BY request_count DESC
LIMIT 3;

7. Detecting Suspicious Activity

This query identifies IP addresses that have more than 3 failed requests (status 404 or 500).
The HAVING COUNT(*) > 3 condition filters out normal traffic.

SELECT ip, COUNT(*) AS failed_requests
FROM web_server_logs
WHERE status IN (404, 500)
GROUP BY ip
HAVING COUNT(*) > 3;

8. Analyzing Traffic Trends

The timestamp1 field (in YYYY-MM-DD HH:MM:SS format) is truncated to the minute level (YYYY-MM-DD HH:MM).
This allows counting the number of requests per minute to observe traffic patterns over time.

SELECT SUBSTRING(timestamp1, 1, 16) AS minute_time, COUNT(*) AS request_count
FROM web_server_logs
GROUP BY SUBSTRING(timestamp1, 1, 16)
ORDER BY minute_time;


### Challenges Faced

Timestamp Parsing Issues: Formatting inconsistencies in timestamps required adjustments in query logic.

Data Ingestion Errors: Some records contained missing fields, requiring data cleansing before loading into Hive.

Query Optimization: Performance tuning was necessary for large datasets by using partitioning and indexing.


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