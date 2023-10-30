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

Question 2: Are there any seasonal trends in sales or revenues? Which months have the highest sales?

SQL Queries:

```SQL
SELECT
	to_char(to_date("date", 'YYYYMMDD'),'YYYY-MM-DD') AS formatted_date, 
	EXTRACT(YEAR FROM to_date("date", 'YYYYMMDD')) AS year,
	EXTRACT(MONTH FROM to_date("date", 'YYYYMMDD')) AS month,
	CASE
		WHEN EXTRACT(MONTH FROM to_date("date", 'YYYYMMDD')) IN (12,1,2) THEN 'WINTER'
		WHEN EXTRACT(MONTH FROM to_date("date", 'YYYYMMDD')) IN (3,4,5) THEN 'SPRING'
		WHEN EXTRACT(MONTH FROM to_date("date", 'YYYYMMDD')) IN (6,7,8) THEN 'SUMMER'
		WHEN EXTRACT(MONTH FROM to_date("date", 'YYYYMMDD')) IN (9,10,11) THEN 'FALL'
		ELSE 'OTHER'
	END AS seasons,
	SUM(CAST("totalTransactionRevenue" AS INTEGER))/1000000 AS total_revenue
FROM 
	all_sessions
WHERE
	"date" is not null AND 
	"totalTransactionRevenue" IS NOT NULL AND 
	"transactions" IS NOT NULL 
GROUP BY
	formatted_date, year, month
ORDER BY
	total_revenue DESC
```

**Sum of revenue per seasons**  

```SQL
WITH RevenueBySeasons AS (
	SELECT
		to_char(to_date("date", 'YYYYMMDD'),'YYYY-MM-DD') AS formatted_date, 
		EXTRACT(YEAR FROM to_date("date", 'YYYYMMDD')) AS year,
		EXTRACT(MONTH FROM to_date("date", 'YYYYMMDD')) AS month,
		CASE
			WHEN EXTRACT(MONTH FROM to_date("date", 'YYYYMMDD')) IN (12,1,2) THEN 'WINTER'
			WHEN EXTRACT(MONTH FROM to_date("date", 'YYYYMMDD')) IN (3,4,5) THEN 'SPRING'
			WHEN EXTRACT(MONTH FROM to_date("date", 'YYYYMMDD')) IN (6,7,8) THEN 'SUMMER'
			WHEN EXTRACT(MONTH FROM to_date("date", 'YYYYMMDD')) IN (9,10,11) THEN 'FALL'
			ELSE 'OTHER'
		END AS seasons,
		SUM(CAST("totalTransactionRevenue" AS INTEGER))/1000000 AS total_revenue
	FROM 
		all_sessions
	WHERE
		"date" is not null AND 
		"totalTransactionRevenue" IS NOT NULL AND 
		"transactions" IS NOT NULL 
	GROUP BY
		formatted_date, year, month
	ORDER BY
		total_revenue DESC
)
SELECT 
	seasons,
	SUM(total_revenue) AS total_season_revenue
FROM 
	RevenueBySeasons
GROUP BY
	seasons
ORDER BY 
	total_season_revenue DESC
```

Answer: When we categorized the months into the four seasons we can understand which seasons produced the most revenue. The winter season collected 15201 total revenue 
which amounted to the most out of all the other seasons. Spring came second in total revenue gained. While Fall was last place only amounting to 5119 total revenue. Although this
answered the question, another question we could have asked is to see the correlation between the seasons and products and see why the winter season produced more revenue than others.

![](https://i.imgur.com/WT59XE4.png)


![](https://i.imgur.com/tIRzgbf.png)


Question 3: What are the top-selling products in each category?

SQL Queries:

```SQL
WITH ProductRanked AS (
	SELECT 
		p."SKU" AS SKU,
		p."name" AS product_name, 
		a."v2ProductCategory" AS category,
		CAST(sbs."total_ordered" AS INTEGER) AS total_ordered,
		RANK() OVER(PARTITION BY a."v2ProductCategory" ORDER BY sbs."total_ordered" DESC) AS sales_rank,
		CASE
			WHEN a."v2ProductCategory"  = '${escCatTitle}' THEN 'HOME/NEST/NEST-USA/'
			WHEN a."v2ProductCategory"  = '(not set)' THEN 'HOME/ACCESSORIES/DRINKWARE/'
			ELSE a."v2ProductCategory"
		END AS updated_category
	FROM products AS p 
	JOIN sales_by_sku AS sbs ON p."SKU" = sbs."productSKU"
	JOIN all_sessions AS a ON p."SKU" = a."productSKU"	
)
SELECT
	SKU,
	product_name,
    UPPER(updated_category) AS category,
    total_ordered 
FROM ProductRanked
WHERE sales_rank = 1 
GROUP BY updated_category,SKU,product_name, total_ordered 
ORDER BY total_ordered DESC, updated_category
```

Answer: Based on the results we can see that from the drinkware category was the Android 17oz Stainless Steel Sport Bottle. Then for the NEST category, the top selling product 
is seen to be the Learning Thermostat 3rd Gen-USA - Stainless Steel. Then for the notebook/journal category the top selling product is the Hard Cover Journal.

![](https://i.imgur.com/iHdaPiR.png)

