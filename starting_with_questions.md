Answer the following questions and provide the SQL queries used to find the answer.

    
**Question 1: Which cities and countries have the highest level of transaction revenues on the site?**


SQL Queries:

-- Uncleaned data --

```SQL
SELECT 
	"city",
	"country",
	CAST("totalTransactionRevenue" AS INTEGER)/1000000 AS total_transaction_revenue
FROM 	"all_sessions"
WHERE 
	"transactions" IS NOT NULL AND 
	"totalTransactionRevenue" IS NOT NULL
GROUP BY
	"total_transaction_revenue",
	"city",
	"country"	
ORDER BY 
	"total_transaction_revenue" DESC 


-- CLEANING DATA WITH THE WHERE OPERATOR --


SELECT 
	"city",
	"country",
	CAST("totalTransactionRevenue" AS INTEGER)/1000000 AS total_transaction_revenue
FROM 	"all_sessions"
WHERE 
	"transactions" IS NOT NULL AND 
	"totalTransactionRevenue" IS NOT NULL AND 
	"city" <> 'not available in demo dataset'
GROUP BY
	"total_transaction_revenue",
	"city",
	"country"	
ORDER BY 
	"total_transaction_revenue" DESC 
LIMIT 5  

Answer:

In the following data set we found out there were 80 rows that yieleded a total transation revenue. When we query that first syntax 
we noticed the top 5 counties that had the highest transaction revenue where from the United States. However, in the data set, 4 out of these top 5 highest revenue 
had missing data which omitted the city. Using the where operator we can clean the data to remove these invalid information which resulted in a slightly different top 5 
highest revenue data. The resulted table then had Atlanta, Sunnyvale, Tel Aviv-Yafo, Los Angeles, and Seattle as the top 5 cities with the highest revenue on site. While 
Israel making an appearance in the top 5 as the country with the highest revenue on site, while United States rounding out the other 4 spots in the top 5.

![](https://i.imgur.com/a7Dvu6W.png)
![](https://i.imgur.com/ONBoDwX.png)

**Question 2: What is the average number of products ordered from visitors in each city and country?**


SQL Queries:

-- AVERAGE PRODUCTS ORDERED PER COUNTRY AND OVERALL COMBINED AVERAGE OF ALL COUNTRIES. --

SELECT DISTINCT
	"country",
	ROUND(AVG(CAST("total_ordered" AS INTEGER)), 2) AS avg_number_of_products
FROM 	"sales_by_sku" AS sbs
JOIN 	all_sessions AS a
ON 	a."productSKU" = sbs."productSKU"
WHERE 	a."country" <> '(not set)'
GROUP BY "country"
ORDER BY "country"


SELECT
	ROUND(AVG(avg_number_of_products), 2) AS overall_country_average
FROM (
	SELECT 
        	ROUND(AVG(CAST("total_ordered" AS INTEGER)), 2) AS avg_number_of_products
	FROM 	"sales_by_sku" AS sbs
    	JOIN 	all_sessions AS a ON a."productSKU" = sbs."productSKU"
	GROUP BY "country"
) AS CountryAverage


-- AVERAGE PRODUCTS ORDERED PER CITY AND OVERALL COMBINED AVERAGE OF ALL CITIES. --

SELECT DISTINCT
	"city",
	ROUND(AVG(CAST("total_ordered" AS INTEGER)), 2) AS avg_number_of_products
FROM 	"sales_by_sku" AS sbs
JOIN 	all_sessions AS a
ON 	a."productSKU" = sbs."productSKU"
WHERE	 a."city" <> '(not set)'
GROUP BY "city"
ORDER BY "city"


SELECT ROUND(AVG(avg_number_of_products), 2) AS overall_city_average
FROM (
	SELECT 
        	ROUND(AVG(CAST("total_ordered" AS INTEGER)), 2) AS avg_number_of_products
	FROM 	"sales_by_sku" AS sbs
	JOIN 	all_sessions AS a ON a."productSKU" = sbs."productSKU"
    	GROUP BY "city"
) AS CityAverage



Answer:

In the resulting queries we found out there were 226 distinct cities that ordered products. We can see in the second query that the average ordered products for the city
is 22.85 products ordered. While we can see with the "country" query that we there are 126 countries that had placed orders. Combined, there were a total of 19.75 products ordered
for each country combined.


![](https://i.imgur.com/9UQ88Zc.png)
![](https://i.imgur.com/AjPmo7K.png)
![](https://i.imgur.com/WMaaDtq.png)
![](https://i.imgur.com/m3yLrHX.png)

**Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**


SQL Queries:

-- country --
-- max --

WITH CountryCategoryCount AS (
	SELECT 
		a."country",
		a."v2ProductCategory" AS product_category,
		COUNT(*) AS category_count
	FROM all_sessions AS a
	WHERE 
		a."country" <> '(not set)' AND
		a."city" <> 'not available in demo dataset' AND
		a."v2ProductCategory" <> '(not set)' AND
		a."v2ProductCategory" <> '${escCatTitle}' AND 
		a."v2ProductName" <> '(not set)'
	GROUP BY a."country", a."v2ProductCategory"
)
SELECT 
	"country",
	product_category,
	MAX(category_count) AS max_category_count
FROM CountryCategoryCount
GROUP BY "country", product_category
ORDER BY max_category_count DESC, "country"

-- min -- 

WITH CountryCategoryCount AS (
	SELECT 
		a."country",
		a."v2ProductCategory" AS product_category,
		COUNT(*) AS category_count
	FROM all_sessions AS a
	WHERE 
		a."country" <> '(not set)' AND
		a."city" <> 'not available in demo dataset' AND
		a."v2ProductCategory" <> '(not set)' AND
		a."v2ProductCategory" <> '${escCatTitle}' AND 
		a."v2ProductName" <> '(not set)'
	GROUP BY a."country", a."v2ProductCategory"
)
SELECT 
	"country",
	product_category,
	MIN(category_count) AS min_category_count
FROM CountryCategoryCount 
GROUP BY "country", product_category
ORDER BY min_category_count, "country"

-- city --

-- max -- 

WITH CityCategoryCount AS (
	SELECT 
		a."city",
		a."v2ProductCategory" AS product_category,
		COUNT(*) AS category_count
	FROM all_sessions AS a
	WHERE 
		a."city" <> '(not set)' AND 
		a."city" <> 'not available in demo dataset' AND
		a."v2ProductCategory" <> '(not set)' AND
		a."v2ProductCategory" <> '${escCatTitle}' AND 
		a."v2ProductName" <> '(not set)'
	GROUP BY a."city", a."v2ProductCategory" 
)
SELECT 
	"city",
	product_category,
	MAX(category_count) AS max_category_count
FROM CityCategoryCount 
GROUP BY "city", product_category
ORDER BY max_category_count DESC, "city"

-- min -- 

WITH CityCategoryCount AS (
	SELECT 
		a."city",
		a."v2ProductCategory" AS product_category,
		COUNT(*) AS category_count
	FROM all_sessions AS a
	WHERE 
		a."city" <> '(not set)' AND 
		a."city" <> 'not available in demo dataset' AND
		a."v2ProductCategory" <> '(not set)' AND
		a."v2ProductCategory" <> '${escCatTitle}' AND 
		a."v2ProductName" <> '(not set)'
	GROUP BY a."city", a."v2ProductCategory"
)
SELECT 
	"city",
	product_category,
	MIN(category_count) AS min_category_count
FROM CityCategoryCount 
GROUP BY "city", product_category
ORDER BY min_category_count, "city"


Answer:

To understand this data and to answer this question, I used a min and max function in order to see if there are any patterns we can find from countries and cities 
that might have ordered a specific product category. The country that ordered the most was the United States, which filled up the top 10 product ordered of 10 types of different
categories. While Japan ordered the least amount of the "lifestyle" category. The city of Mountain View had the most number of products ordered which majory ordered Men's T-shirts. 
While the city of Minato had the lowest count which make sense since we saw Japan oredered the least amount.

![](https://i.imgur.com/8H6pZQz.png)
![](https://i.imgur.com/KB8sWmk.png)
![](https://i.imgur.com/7oUfRdf.png)
![](https://i.imgur.com/Adxe23B.png)


**Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**


SQL Queries:

-- Country -- 

WITH RankedProducts AS (
    SELECT 
        a."city",
        a."country",
        p."name" AS top_selling_product,
		a."v2ProductCategory" AS product_category,
        CAST(sr."total_ordered" AS INTEGER) AS total_ordered_quantity,
        ROW_NUMBER() OVER(PARTITION BY a."country" ORDER BY CAST(sr."total_ordered" AS INTEGER) DESC) AS country_rank
    FROM all_sessions AS a
    JOIN sales_report AS sr ON a."productSKU" = sr."productSKU"
    JOIN products AS p ON a."productSKU" = p."SKU"
    WHERE 
        a."city" <> '(not set)' AND
        a."country" <> '(not set)' AND
        a."city" <> 'not available in demo dataset' AND
        a."country" <> '(not set)' AND 
        p."name" <> '(not set)'AND
		a."v2ProductCategory" <> '(not set)'
)
SELECT 
    "city",
    "country",
    top_selling_product,
	product_category,
    total_ordered_quantity
FROM RankedProducts
WHERE country_rank = 1 AND
	total_ordered_quantity > 0
GROUP BY  total_ordered_quantity, "country","city", top_selling_product, product_category
ORDER BY  total_ordered_quantity DESC, "country", top_selling_product;

-- City --

WITH RankedProducts AS (
    SELECT 
        a."city",
        a."country",
        p."name" AS top_selling_product,
		a."v2ProductCategory" AS product_category,
        CAST(sr."total_ordered" AS INTEGER) AS total_ordered_quantity,
        ROW_NUMBER() OVER(PARTITION BY a."city" ORDER BY CAST(sr."total_ordered" AS INTEGER) DESC) AS city_rank
    FROM all_sessions AS a
    JOIN sales_report AS sr ON a."productSKU" = sr."productSKU"
    JOIN products AS p ON a."productSKU" = p."SKU"
    WHERE 
        a."city" <> '(not set)' AND
        a."country" <> '(not set)' AND
        a."city" <> 'not available in demo dataset' AND
        a."country" <> '(not set)' AND 
        p."name" <> '(not set)'
)
SELECT 
    "city",
    "country",
    top_selling_product,
	product_category,
    total_ordered_quantity
FROM RankedProducts
WHERE city_rank = 1 AND
	total_ordered_quantity > 0
GROUP BY  total_ordered_quantity, "country","city", top_selling_product, product_category
ORDER BY total_ordered_quantity DESC, "city", top_selling_product;


Answer:

Out of the 55 rows generated when analyzing the most ordered product by country, we can see some patterns. It looks like Asian countries such as Indonesia, Maylasia, 
Taiwan, Thailand, and Vietnam had their top selling product as insulated water bottles or in that same category. Maybe this is due to their warmer climate as people from these Asian 
countries want their water to be cool for longer periods of time. Another pattern that was found was that Ballpoint LED Light Pen were the top sellers from this site, accumlating "456"
numbers of pens sold. This can be expanded to every product sold where each product has the same amount ordered for each different product for each countries. Indicating that there
are a cap of how many items are sold. This may be due to stock limitations. When we take a look at top products sold for each city, we can conclude that the Ballpoint LED Light Pen 
was sold the most just like our country table and that it also has 456 pens sold for each city. We can also see a pattern where majority of these pens were sold in the United States.

![](https://i.imgur.com/5Ef4jj1.png)
![](https://i.imgur.com/poxMpfC.png)


**Question 5: Can we summarize the impact of revenue generated from each city/country?**

SQL Queries:

WITH RevenueGenerated AS (
	SELECT
		a."city",
		a."country",
		a."productSKU",
		SUM(CAST(a."totalTransactionRevenue" AS INTEGER))/1000000 AS sum_total_revenue,
		AVG(SUM(CAST(a."totalTransactionRevenue" AS INTEGER))/1000000) OVER () AS avg_total_revenue
	FROM all_sessions As a
	WHERE 
		a."totalTransactionRevenue" IS NOT NULL AND
		a."city" IS NOT NULL AND
		a."city" <> '(not set)' AND
		a."city" <> 'not available in demo dataset' AND
     	a."country" IS NOT NULL AND   
		a."country" <> '(not set)' AND
    	a."country" <> '(not set)' 
	GROUP BY "city", "country", a."productSKU"
)
SELECT 
	rg."city",
	rg. "country",
	p."name",
	rg.sum_total_revenue,
	ROUND(AVG(avg_total_revenue),2) AS avg_total_revenue
FROM RevenueGenerated AS rg
JOIN products AS p ON rg."productSKU" = p."SKU"
GROUP BY "city", "country", sum_total_revenue, p."name"
ORDER BY sum_total_revenue DESC



Answer:

To answer this question we used a sum aggregate function and compared it with the average for total revenue. We can see that the revenue that the top 15 cities make 
are above the average total revenue of all cities that we had filtered out. Meaning that the 32 remaining cities are not making enough revenue compared to the average of the cities.

![](https://i.imgur.com/BLjGiI3.png)




