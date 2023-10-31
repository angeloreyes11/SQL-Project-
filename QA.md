**What are your risk areas? Identify and describe them.**

QA Process:
Describe your QA process and include the SQL queries used to execute it.

During my quality assurance process, there were a few things I wanted to check to make sure that the data cleaning and transformation I did made sense. The first step in my QA proces was to check that the primary keys of my tables which were "fullVisitorId" and "visitId" were all outputting unique values. 

```SQL
SELECT 
	vas."fullVisitorId", 
	vas."visitId" AS all_sessions_visitId,
	van."visitId" AS analytics_visitId
FROM view_allsessions AS vas
JOIN view_analytics AS van 
ON vas."visitId" = van."visitId"
```

This query resulted in 3 columns, "fullVisitorId", "all_seasions_visitId", "analytics_visitId" and outputted 16 rows when running this query.

```SQL
SELECT 
	vas."fullVisitorId", 
	vas."visitId" AS all_sessions_visitId,
	van."visitId" AS analytics_visitId
FROM view_allsessions AS vas
JOIN view_analytics AS van 
ON vas."visitId" = van."visitId"
WHERE 
	vas."visitId" IS NOT NULL AND 
	van."visitId" IS NOT NULL
```


This following query filters out all the NULL values to output only unique and present values. Both queries resulted in 16 rows which indicates that our view table was correct since there were no change in total rows.





