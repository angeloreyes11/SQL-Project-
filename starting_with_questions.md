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

In the following data set we found out there were 80 rows that yieleded a total transation revenue.
When we query that first syntax we noticed the top 5 counties that had the highest transaction revenue where from the United States.
However, in the data set, 4 out of these top 5 highest revenue had missing data which omitted the city. Using the where operator
we can clean the data to remove these invalid information which resulted in a slightly different top 5 highest revenue data. The resulted
table then had Atlanta, Sunnyvale, Tel Aviv-Yafo, Los Angeles, and Seattle as the top 5 cities with the highest revenue on site.
While Israel making an appearance in the top 5 as the country with the highest revenue on site, while United States rounding out
the other 4 spots in the top 5.

![](https://i.imgur.com/a7Dvu6W.png)

**Question 2: What is the average number of products ordered from visitors in each city and country?**


SQL Queries:



Answer:





**Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**


SQL Queries:



Answer:





**Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**


SQL Queries:



Answer:





**Question 5: Can we summarize the impact of revenue generated from each city/country?**

SQL Queries:



Answer:







