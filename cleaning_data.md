**What issues will you address by cleaning the data?**

**Cleaning Process**

During the cleaning process of the different tables, I initially noticed right away that there were alot of null values, improper formatting, and irrelavent columns. I tackled this task as I went through the questions from "starting_with_data" and "starting_with_questions". As I worked through those questions, there were some columns that had to be cleaned up along the way. In the section below, I will list down what neccesary changes I had to do in order to help me get through and undertand those questions. Note that some columns were not used and had to be dropped. Creating views were the choice of action while performing the cleaning process as it was advised not to touch the actual data. 

Queries:
Below, provide the SQL queries you used to clean your data.

_**Table 1: "all_sessions" table.**_

There were plenty of issues with the all_seasons table. Since this was the main table to use in order for me to answer the questions provided for this project, there were a lot of data cleaning that had to be done. The process of cleaning this data was to create a view that had all the neccessary columns and clean it up using data transformation and data cleaning process we had learned in our lectures. In the space below will be a query to create a view of the cleanned all_sessions table.Then one by one we will break it down to understand what were the steps in cleaning the data.

```SQL
CREATE OR REPLACE VIEW view_allsessions AS
SELECT DISTINCT	ON 	
	("fullVisitorId", "visitId") "fullVisitorId", "visitId", 
	"channelGrouping",
	"country", 
	"city",
	CAST("totalTransactionRevenue" AS INTEGER)/1000000 AS totalTransactionRevenue,
	to_char(to_date("date", 'YYYYMMDD'),'YYYY-MM-DD') AS formatted_date, 
	CAST("productPrice" AS INTEGER)/1000000 AS productPrice,
	LPAD(CASE 
		 	WHEN "productSKU" LIKE 'GGOE%' THEN "productSKU"
		 	ELSE 'GGOE' || REPLACE ("productSKU", ' ','')
		 END, 11, '0') AS productSKU,
	"v2ProductName", 
	"v2ProductCategory"
FROM all_sessions
WHERE 
	"country" <> '(not set)' AND
	"city" <> '(not set)' AND 
	"city" <> 'not available in demo dataset' AND 
	"totalTransactionRevenue" IS NOT NULL AND 
	"v2ProductCategory" <> '(not set)' AND 
	"v2ProductCategory" <> '${escCatTitle}'
```

```SQL
SELECT *
FROM view_allsessions
```
![](https://i.imgur.com/CBJU2gD.png)

_**A. "fullVisitorId" & "visitId Columns**_ 

fullVisitorId & visitId columns had repeating values. This was fixed by finding the unique values of these columns using the SELECT DISCINT statement. 

```SQL
CREATE OR REPLACE VIEW view_IDs AS
SELECT DISTINCT	ON 	
	("fullVisitorId", "visitId") "fullVisitorId", "visitId"
FROM
  all_sessions
```

```SQL 
SELECT *
FROM view_IDs
```

![](https://i.imgur.com/On6pMB4.png)

_**B. "country" & "city" Columns**_

In the country and city columns I had found that there were missing data. Some countries and cities had a '(not set)' while some city information were missing which were marked by 'not available in demo dataset' This was cleaned up using the where statment. 

```SQL
CREATE OR REPLACE VIEW view_locations AS
SELECT
	"country", 
	"city"
FROM all_sessions 
WHERE 
	"country" <> '(not set)' AND
	"city" <> '(not set)' AND 
	"city" <> 'not available in demo dataset' 
ORDER BY "country", "city"
```

```SQL 
SELECT *
FROM view_locations
```
![](https://i.imgur.com/X5d8SvP.png)

_**C. "totalTransactionRevenue", "productPrice", "productSKU", "v2ProdutName" & "v2ProductCategory" Columns**_

These columns were used to answer majority of the questions that were asked during the project. We were given a hint that the prices in this dataset were multiplied by 1000000 which was the reason for initially seeing high numbers for these columns. To fix it we would have to divide it by 1000000, but before that since all of the tables were imported as varchars, we would have to convert these columns to INTEGERs using the CAST function. There were also some NULL values in the totaltransactionrevenue column so using the WHERE clause will help eliminate those out. ProductSku column also had some values that were not formatted properly as others. To do this we used the LPAD() function in order to input the strings that are needing to be corrected (using a case statement). Finally the last column that needed to be cleaned was the v2ProductCategory column which contained some missing information. 

```SQL
CREATE OR REPLACE VIEW view_productdetails AS
WITH UniqueVisitors AS (
	SELECT DISTINCT ON ("fullVisitorId") "fullVisitorId",
		CAST("totalTransactionRevenue" AS INTEGER)/1000000 AS totalTransactionRevenue,
		CAST("productPrice" AS INTEGER)/1000000 AS productPrice,
		LPAD(CASE 
				WHEN "productSKU" LIKE 'GGOE%' THEN "productSKU"
				ELSE 'GGOE' || REPLACE ("productSKU", ' ','')
			 END, 11, '0') AS productSKU,
		"v2ProductName" AS productName, 
		"v2ProductCategory" AS productCategory
	FROM all_sessions
	WHERE 
		"totalTransactionRevenue" IS NOT NULL AND 
		"v2ProductCategory" <> '(not set)' AND 
		"v2ProductCategory" <> '${escCatTitle}' 
)
SELECT *
FROM UniqueVisitors
ORDER BY totalTransactionRevenue DESC


```

```SQL
SELECT * 
FROM view_productdetails 
```

![](https://i.imgur.com/Nh40bqw.png)


_**Table 2: "analytics" table.**_

In the same way we cleaned and transformed the all_sessions table, we have done the same for the analytics table. There were similar formatting and filtering out that was needed to be done with this table. Majority of the columns were cleaned in a similar way as the columns for the all_sessions table. 

```SQL
CREATE OR REPLACE VIEW view_analytics AS
WITH RevenueCTE AS (
	SELECT DISTINCT	ON 	
		("visitId") "visitId", 
		"channelGrouping",
		to_char(to_date("date", 'YYYYMMDD'),'YYYY-MM-DD') AS formatted_date, 
		CAST("unit_price" AS INTEGER)/1000000 AS unit_price,
		CAST("units_sold" AS INTEGER) AS units_sold, 
		CAST("pageviews" AS INTEGER) AS page_views,
		CAST("revenue" AS BIGINT)/1000000 AS revenue
	FROM analytics
	WHERE 
		CAST(unit_price AS INTEGER) > 0 AND 
		unit_price IS NOT NULL AND 
		CAST(units_sold AS INTEGER) > 0 AND 
		units_sold IS NOT NULL AND 
		pageviews IS NOT NULL AND 
		revenue IS NOT NULL 
)
SELECT *
FROM RevenueCTE
ORDER BY revenue DESC
```

```SQL
SELECT *
FROM view_analytics
```

![](https://i.imgur.com/EOa1018.png)


_**A. "visitId" Column**_ 

There were multiple repeating values found in this column. To clean it we need to SELECT DISTINCT values in order for us to get unique values for this column.

```SQL
CREATE OR REPLACE VIEW view_analytics AS
SELECT DISTINCT	ON 	
		("visitId") "visitId"
-- Remaining synthax from view_analytics goes here
```

_**B. "date" Column**_ 

Upon first look, we can tell that the date column did not have the proper formatting of YYYY-MM-DD. In order to fix this we would have to use the to_date function in combination to to_char function. 

```SQL
-- synthax before this is seen from the view_analytics goes here.
to_char(to_date("date", 'YYYYMMDD'),'YYYY-MM-DD') AS formatted_date,
-- further syntax from the view_analytics goes here 
```

_**C. "unit_price", "units_sold", "page_views", and "revenue" Columns**_ 

These next 4 columns were cleaned by first converting them into their proper format such as INTEGER or BIGINT. The unit_price and revenue were divided by 1000000 in order to get more easier numbers to read. There were also filtered out using the WHERE clause in order to get filter out the NULL values and as well prices and units sold that were 0 (indicating that they had no transaction). There were also NULL values that had to be filtered out in the page_views column. 


```SQL
-- synthax before this is seen from the view_anaytics table
CAST("unit_price" AS INTEGER)/1000000 AS unit_price,
		CAST("units_sold" AS INTEGER) AS units_sold, 
		CAST("pageviews" AS INTEGER) AS page_views,
		CAST("revenue" AS BIGINT)/1000000 AS revenue
	FROM analytics
	WHERE 
		CAST(unit_price AS INTEGER) > 0 AND 
		unit_price IS NOT NULL AND 
		CAST(units_sold AS INTEGER) > 0 AND 
		units_sold IS NOT NULL AND 
		pageviews IS NOT NULL AND 
		revenue IS NOT NULL 
-- the rest of the synthax goes here from the view_analytics table
```

_**Table 3: "products" table.**_

To clean the products table, our approach was very similar to what we have been doing for the other tables. Some columns were the same in this products table as the other two tables we had already cleaned up. So there were be some similar queries that were already seen above. 

```SQL
CREATE OR REPLACE VIEW view_products AS
SELECT 
	LPAD(CASE 
		WHEN "SKU" LIKE 'GGOE%' THEN "SKU"
		ELSE 'GGOE' || REPLACE ("SKU", ' ','')
	 END, 11, '0') AS SKU,
	"name",
	CAST("orderedQuantity" AS INTEGER) AS orderedQuantity,
	CAST("stockLevel" AS INTEGER) AS stockLevel, 
	CAST("restockingLeadTime" AS INTEGER) AS restockingLeadTime,
	CAST("sentimentScore" AS NUMERIC) AS sentimentScore,
	CAST("sentimentMagnitude" AS NUMERIC) AS sentimentMagnitude
FROM products
WHERE 
	"orderedQuantity" IS NOT NULL AND 
	CAST("orderedQuantity" AS INTEGER) > 0 AND 
	CAST("sentimentScore" AS NUMERIC) IS NOT NULL AND
	CAST("sentimentMagnitude" AS NUMERIC) IS NOT NULL
ORDER BY orderedQuantity DESC
```

```SQL
SELECT *
FROM view_products
```

![](https://i.imgur.com/jeXSLKL.png)

_**A. "SKU" Column**_ 

Same as the productSKU column in our all_session table, we had to format this SKU column to match the same format in the entire column. Again, we used the LPAD() function. 

```SQL
CREATE OR REPLACE VIEW view_products AS
SELECT 
	LPAD(CASE 
		WHEN "SKU" LIKE 'GGOE%' THEN "SKU"
		ELSE 'GGOE' || REPLACE ("SKU", ' ','')
	 END, 11, '0') AS SKU,
	"name",
 -- From view_products table the remaining synthax goes here.
```

_**B. "orderQuantity", "stockLevel", "restockingLeadTime", "sentimentScore", and "sentimentMagnitude" Columns**_ 

These next 5 tables were cleaned up using changing their format using the CAST function and as well as filtering the tables using the WHERE clause.

```SQL
-- the previous synthax from the view_products table goes here
CAST("orderedQuantity" AS INTEGER) AS orderedQuantity,
	CAST("stockLevel" AS INTEGER) AS stockLevel, 
	CAST("restockingLeadTime" AS INTEGER) AS restockingLeadTime,
	CAST("sentimentScore" AS NUMERIC) AS sentimentScore,
	CAST("sentimentMagnitude" AS NUMERIC) AS sentimentMagnitude
FROM products
WHERE 
	"orderedQuantity" IS NOT NULL AND 
	CAST("orderedQuantity" AS INTEGER) > 0 AND 
	CAST("sentimentScore" AS NUMERIC) IS NOT NULL AND
	CAST("sentimentMagnitude" AS NUMERIC) IS NOT NULL
ORDER BY orderedQuantity DESC
```

_**Table 4: "sales_by_sku" table.**_

This was the easiest to clean up as there were only 2 columns in the table. ProductSKU was cleaned up using the LPAD() function as what we used already. While the total_ordered column was cleaned up by filtering irrelavent values such as products with 0 orders and below. 

```SQL
CREATE OR REPLACE VIEW view_salesbysku AS
SELECT
	LPAD(CASE 
		WHEN "productSKU" LIKE 'GGOE%' THEN "productSKU"
		ELSE 'GGOE' || REPLACE ("productSKU", ' ','')
	 END, 11, '0') AS productSKU,
	 CAST("total_ordered" AS INTEGER)
FROM sales_by_sku
WHERE 
	CAST("total_ordered" AS INTEGER) > 0
```

```SQL
SELECT *
FROM view_salesbysku
```

![](https://i.imgur.com/ixZodU3.png)


_**Table 5: "sales_report" table.**_

This sales report table was cleaned in the same way as how we cleaned the products table. ProductSKU was properly fixed in order for all the values to have the same format using the LPAD() function. All the remaining columns were CASTed to its proper format such as INTEGER and NUMERIC. Then the null values were filtered out using the WHERE clause. Also irrelavent data were also filtered out usch as products that had 0 total ordered.

```SQL
CREATE OR REPLACE VIEW view_salesreport AS
SELECT 
	LPAD(CASE 
		WHEN "productSKU" LIKE 'GGOE%' THEN "productSKU"
		ELSE 'GGOE' || REPLACE ("productSKU", ' ','')
	 END, 11, '0') AS productSKU,
	 CAST("total_ordered" AS INTEGER) as total_ordered,
	 "name",
	 CAST("stockLevel" AS INTEGER) AS stockLevel,
	 CAST("restockingLeadTime" AS INTEGER) AS restockingLeadTime,
	 CAST("sentimentScore" AS NUMERIC) AS sentimentScore,
	 CAST("sentimentMagnitude" AS NUMERIC) AS sentimentMagnitude,
	 CAST("ratio" AS NUMERIC) AS ratio
FROM sales_report
WHERE 
	"total_ordered" IS NOT NULL AND 
	CAST("total_ordered" AS INTEGER) > 0 AND 
	CAST("sentimentScore" AS NUMERIC) IS NOT NULL AND
	CAST("sentimentMagnitude" AS NUMERIC) IS NOT NULL
ORDER BY "total_ordered" DESC
```

```SQL
SELECT *
FROM view_salesreport
```

![](https://i.imgur.com/MTzWeSS.png)
