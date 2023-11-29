**QA Check #1:**
One risk area is to determine if the sales data is accurately captured and displayed at an SKU level. To check this, I compared the total number of items ordered between the "salesbysku" and "salesreport" tables by joining the two tables using SKUs and then using a CASE WHEN function to compare the values in each row. Based on the result, all 454 rows returned "True" giving confidence that the SKU sales data was accurately captured

~~~~sql
SELECT COUNT(*),
CASE WHEN sbs.total_ordered = sr.total_ordered THEN True
ELSE False
END As ordercheck
FROM salesbysku sbs
JOIN salesreport sr USING (sku)
GROUP BY ordercheck
~~~~

**QA Check #2:**
Another risk area is whether our pricing is consistent throughout the database. To check this I compared the salesbysku with the allsessions table to see if there were any discrepencies. Based on the table results of the query below, 484 rows were returned with multiple pricing options per SKU in certain cases. 

~~~~sql
SELECT DISTINCT sku, price FROM salesbysku
JOIN allsessions USING (sku)
ORDER BY sku DESC
~~~~

To remedy this I decided to take the average price and create a new pricing table that can be referenced by our E-Commerce website moving forward when pricing it's items by joining the allsessions or analytics table with the SKU data from the new pricing table

~~~~sql
CREATE TABLE new_pricing AS
SELECT sku, CAST(AVG(price) AS DECIMAL(10,2)) AS average_price
FROM salesbysku
JOIN allsessions USING (sku)
GROUP BY sku
ORDER BY sku DESC;
~~~~

**QA Check #3:** Based on the new table created, check whether each SKU has an associated price to it

Looking at the newly created table, we can see that all SKU items beginning with "918" that do not follow the "GGO" leading format do not have a price associated with them. To double check if any items have been ordered we can use the following syntax.

~~~~sql
SELECT sku, total_ordered, name, CAST(AVG(price) AS DECIMAL(10,2)) AS average_price
FROM salesbysku
JOIN allsessions USING (sku)
GROUP BY sku, name, total_ordered
ORDER BY total_ordered, average_price
~~~~

Since none of these items have been ordered, we can clean up our table by removing the SKUs that do not have a price associated with them as they are likely outdated or errors

~~~~sql
DELETE FROM new_pricing
WHERE average_price = 0
~~~~

**QA Check #4:** Check that visitor IDs follow a logical order structure and are not scattered

To look at visitid we can select the distinct visitid's from the allsessions table and order them chronoligcally to see if they follow a pattern.

~~~~sql
SELECT DISTINCT visitid FROM allsessions
ORDER BY visitid
~~~~

We can see that the visitid's begin with 14700XX and chronoligically increase. A similar check can be done by looking at the other extreme of the table using the DESC ordering

~~~~sql
SELECT DISTINCT visitid FROM allsessions
ORDER BY visitid DESC
~~~~

On the other end we can see that the latest visitid was 150165XX and is descending in a logical fashion. Therefore, there are no concerns about using the visitid based on the QA

**QA Check #5:** Check that dates fall into a logical pattern based on tables

~~~~sql
SELECT date, count(*) FROM allsessions
GROUP BY date
ORDER BY date
~~~~

A similar check can be completed for the analytics table

~~~~sql
SELECT date, count(*) FROM analytics
GROUP BY date
ORDER BY date
~~~~

We can see that all data falls between 2016-2017 for the allsessions table and between May - August 2017 for the analytics table. While the discrepency in collection data remains unknown, at least the data is spread logically meaning the limited data can be trusted

**QA Check #6:** Check that time on site are logically linked

To check this, I looked at where time on site is Null, implying the customer either clicked and immediately exited the webpage and compared this against the revenue bucket. In this case, the resulting query returned zero results (as expected) implying that no revenue was generated when time on site was NULL.

~~~~sql
SELECT * FROM analytics
WHERE time_on_site IS NULL
AND revenue IS NOT NULL
~~~~

**QA Check #7:** Validate that each SKU and name is only counted once in the products table
~~~~sql
SELECT sku, name, count(name) FROM products
GROUP BY sku, name
ORDER BY count(name) DESC
~~~~

The result of the query shows that each SKU/Name pairing has a count of 1 implying that no duplicates exist in the database

**QA Check #6:** Validate that restock_lead_time is not zero in the products table

~~~~sql
SELECT * FROM products
WHERE restock_lead_time = 0
~~~~

The resulting query returns zero results implying that we do not have any restocking lead times that are equal to zero
