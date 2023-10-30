Question 1: Does the marketing channel affect the revenue? 

SQL Queries:

```SQL
SELECT
	a. "fullVisitorId" AS visitor_id,
	a."channelGrouping" AS channel_marketing,
	CAST("totalTransactionRevenue" AS INTEGER)/1000000 AS total_revenue
FROM 
	all_sessions AS a
JOIN
	analytics AS an ON a."fullVisitorId" = an."fullvisitorId"
WHERE
	a."channelGrouping" IS NOT NULL AND
	a."totalTransactionRevenue" IS NOT NULL 
GROUP BY
	a. "fullVisitorId", channel_marketing, total_revenue
ORDER BY
	total_revenue DESC 

```
Answer: Based on this query, when we isolate the distinct full visitor id we can see that there were more total revenue acquired when visitors were reffered to the site instead for example one 
who would have organically searched for the products manually

![](https://i.imgur.com/7mpwVDW.png)

Question 2: 

SQL Queries:

Answer:



Question 3: 

SQL Queries:

Answer:



Question 4: 

SQL Queries:

Answer:



Question 5: 

SQL Queries:

Answer:
