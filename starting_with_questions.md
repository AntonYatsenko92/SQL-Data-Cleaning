Answer the following questions and provide the SQL queries used to find the answer.

    
**Question 1: Which cities and countries have the highest level of transaction revenues on the site?**

To approach this I first began by looking at the allsessions table. Unfortunately the data here was pretty sparse, with only 81 revenue columns captured and 53 calculated revenue columns when multiplying price * quantity. Based on this I also chose to drop the revenue column (SQL code displayed in the data cleaning MD file).

~~~~sql
SELECT quantity*price AS revenue FROM allsessions
WHERE quantity*price IS NOT NULL
~~~~

Therefore I assumed that visitid is unique to each user (either via IP address or some cookies based tracker) and used visitid to join the allsessions and analytics tables. I was confident this would yield a better answeer as the allsessions table had a distinct count of 14,556 visitid's and the analytics table had a distinct count of 37,877. 

~~~~sql 
SELECT COUNT(DISTINCT visitid) FROM allsessions
~~~~
~~~~sql
SELECT COUNT(DISTINCT visitid) FROM analytics
~~~~

Based on this I combined the two tables below and sorted the information by country and city which displays that the city with greatest revenue is "Sunnyvale" in the United States.

~~~~sql
SELECT country, city, sum(revenue) AS revenue
FROM allsessions
JOIN analytics USING (visitid)
GROUP BY country, city
HAVING city is NOT NULL AND (sum(revenue)) IS NOT NULL
ORDER BY revenue DESC
~~~~

To double check that the country was the greatest revenue in the United States I removed the city column and organized the data using the query below.

~~~~sql
SELECT country, sum(revenue) AS revenue
FROM allsessions
JOIN analytics USING (visitid)
GROUP BY country
HAVING country is NOT NULL AND (sum(revenue)) IS NOT NULL
ORDER BY revenue DESC
~~~~

Therefore, the city with the greatest revenue is Sunnyvale and the country is the United States.

**Question 2: What is the average number of products ordered from visitors in each city and country?**

To calculate quantity of products ordered, I used the data from the analytics table based on the assumption that revenue divided by unit price would give us an estimated_quantity of items ordered per session that generated a sale. Similar to Question #1, I joined the two tables using visitid treating it as the unique identifier of each user. Based on the following code we can identify the average number of products ordered per city. Not surprisingly, Sunnyvale is the city with the highest average order quantity of ~2.55 products.

~~~~sql
SELECT city, avg(estimated_quantity) FROM 
(SELECT visitid, revenue, unit_price, (revenue/unit_price) AS estimated_quantity FROM analytics
WHERE (revenue/unit_price) IS NOT NULL)
JOIN allsessions USING (visitid)
GROUP BY city
~~~~

Finally, we can remove the city column and identify the average products ordered on a country by country basis. This shows that the United States is the country with the highest average product amount with 2.25 items. 

~~~~sql
SELECT country, avg(estimated_quantity) AS products_ordered FROM 
(SELECT visitid, revenue, unit_price, (revenue/unit_price) AS estimated_quantity FROM analytics
WHERE (revenue/unit_price) IS NOT NULL)
JOIN allsessions USING (visitid)
WHERE country IS NOT NULL
GROUP BY country
ORDER BY products_ordered DESC
~~~~

**Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**

Due to a lack of data my approach was to combine the limited quantity information in the "allsessions" table along with dividing revenue by unit price from the "analytics" table and joining the two. Combining the two tables and linking them together only yielded 33 results, of which the majority for country fell into "Houseware" with the leading country being the United States.

~~~~sql
SELECT country, category, count(*) FROM
(SELECT country, city, estimated_quantity, category FROM 
(SELECT visitid, revenue, unit_price, (revenue/unit_price) AS estimated_quantity FROM analytics
WHERE (revenue/unit_price) IS NOT NULL)
JOIN allsessions USING (visitid)

UNION 

SELECT country, city, quantity, category FROM allsessions
WHERE quantity IS NOT NULL)
GROUP BY country, category
ORDER BY count(*) DESC
~~~~

Finally, replacing country with city in the above query allowed categorization by city. The results were similar to those found in country with Sunnyvale accounting for all 13/33 "Houseware" purchases. Therefore, based on the limited sales data available, the most popular category for this store is Houseware from both a city and country point of view.

~~~~sql
SELECT country, category, count(*) FROM
(SELECT country, city, estimated_quantity, category FROM 
(SELECT visitid, revenue, unit_price, (revenue/unit_price) AS estimated_quantity FROM analytics
WHERE (revenue/unit_price) IS NOT NULL)
JOIN allsessions USING (visitid)

UNION 

SELECT country, city, quantity, category FROM allsessions
WHERE quantity IS NOT NULL)
GROUP BY country, category
ORDER BY count(*) DESC
~~~~

**Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**

To determine the top selling product a similar approach was used above, combining the limited quantity sales data between the allsessions and analytics table and then filtering by SKU and cross-referencing it back to the products table. SKU was used since the initial QA determined that the SKU is a more reliable indicator whereas product names have some variability in naming convention.

~~~~sql
SELECT country, name FROM
(SELECT country, sku, sum(quantity) FROM
((SELECT country, city, quantity, SKU FROM allsessions
WHERE quantity IS NOT NULL

UNION 

SELECT country, city, (unit_price/revenue), sku AS estimated_quantity FROM allsessions
JOIN analytics USING (visitid)
WHERE (unit_price/revenue) IS NOT NULL))
WHERE country IS NOT NULL
GROUP BY country, sku
ORDER BY sum(quantity) DESC)
JOIN products USING(sku)
LIMIT 1
~~~~

Therefore on a country level, the highest selling product is the "Leatherette Journal" based on analyzing the quantity sold. The SQL code can be updated by grouping by city instead of country to determine which city had the highest sales of product. 

~~~~sql
SELECT city, name, quantity FROM
(SELECT city, sku, sum(quantity) AS quantity FROM
((SELECT country, city, quantity, SKU FROM allsessions
WHERE quantity IS NOT NULL

UNION 

SELECT country, city, (unit_price/revenue), sku AS estimated_quantity FROM allsessions
JOIN analytics USING (visitid)
WHERE (unit_price/revenue) IS NOT NULL))
WHERE country IS NOT NULL
GROUP BY city, sku
ORDER BY sum(quantity) DESC)
JOIN products USING(sku)
WHERE city IS NOT NULL
~~~~

The data changes when looking at quantity sold at a city level. The main reason for this is alot of the country orders had a NULL city value, which was filtered out by adding the "WHERE city IS NOT NULL" filter. Based on this we can see the top selling product is "Dress Socks" in Madrid. There was no discernable pattern identified as most product ranged in the 1-2 quantity sold and were largely concentrated in the United States, with Madrid being an outlier. 

**Question 5: Can we summarize the impact of revenue generated from each city/country?**

A similar approach was used to combine the allsessions and analytics tables to generate revenue. Based on the data, the United States accounted for the greatest amount of revenue with $7,048. The remaining countries offer negligible revenue in comparison. 

~~~~sql
SELECT country, sum(revenue) FROM
(SELECT country, city, quantity*price AS revenue FROM allsessions
WHERE quantity*price IS NOT NULL

UNION 

SELECT country, city, revenue FROM analytics
JOIN allsessions USING (visitid)
WHERE revenue IS NOT NULL)
GROUP BY country
ORDER BY sum(revenue) DESC
~~~~

Replacing country with city in the above query yields the impact of revenue on a city by city basis. Once again, Sunnyvale accounts for the highest revenue generated with $919.63 followed by Mountain View, Salem, New York, and San Jose. The majority of these cities are in the United States, which gives confidence that the United States amounts for the majority of revenue for this company.

~~~~sql
SELECT city, sum(revenue) FROM
(SELECT country, city, quantity*price AS revenue FROM allsessions
WHERE quantity*price IS NOT NULL

UNION 

SELECT country, city, revenue FROM analytics
JOIN allsessions USING (visitid)
WHERE revenue IS NOT NULL)
WHERE city IS NOT NULL
GROUP BY city
ORDER BY sum(revenue) DESC
~~~~








