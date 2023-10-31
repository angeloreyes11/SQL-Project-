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
![](https://i.imgur.com/dE2GrNz.png)

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

![](https://i.imgur.com/ggeW9qH.png)

This following query filters out all the NULL values to output only unique and present values. Both queries resulted in 16 rows which indicates that our view table was correct since there were no change in total rows.


My next step in validating the data is to find the consistency of some data and values across different tables. Here, I do a double check on all products sku to see if they match up with other sku numbers found in other tables. 

```SQL
SELECT 
	vas."productsku" AS allsession_productsku,
	vp."sku" AS products_sku,
	vsr."productsku" AS salesreport_productsku
FROM view_allsessions AS vas
JOIN view_products AS vp ON vas."productsku" = vp."sku"
JOIN view_salesreport AS vsr ON vas."productsku" = vsr."productsku"
GROUP BY
	allsession_productsku, 
 	products_sku, 
  	salesreport_productsku
```

![](https://i.imgur.com/aBYoOez.png)

This next QA query calculates the count associated with the product sku to check that a unique sku is given to a specific product name. 


```SQL
SELECT
	COUNT(DISTINCT "productsku") AS SKU_count,
	"v2ProductName"	
FROM view_allsessions
GROUP BY "v2ProductName"
ORDER BY SKU_count 
```

![](https://i.imgur.com/O6JOb7A.png)


Another check I wanted to test is to see if the data numbers made sense. For instance, to validate the transaction revenue numbers to see if it falls out of range. As a potential problem might arise if there are some values that skews our results, ie. values below 0 and over 1000. 

```SQL
SELECT 
	"totaltransactionrevenue"
FROM 
	view_allsessions
WHERE 
	totaltransactionrevenue < 0 OR totaltransactionrevenue > 1000
```

![](https://i.imgur.com/uvvvTTx.png)

We see that there were no rows generated that falls out of our range. 


Finally our last check is to see if any calculations adds up. For example here in our view_analytics table we have unit_price and units_sold columns. Assuming if data entry and data gathering was all correct, multiplying those two columns should equal out to the revenue column.

```SQL
SELECT 
	ROUND("unit_price", 2) AS rounded_unit_price, 
	"units_sold",
	ROUND("unit_price" * "units_sold", 2) AS calculated_revenue,
	"revenue"
FROM view_analytics 
```

![](https://i.imgur.com/1IRWvEg.png)

The result of our query was a surpise. It seems like multiplying those two columns did not result to the values seen in the revenue column. We can conclude that there might be errors in the data intake portion, or rounding errors when cleaning up our tables.
