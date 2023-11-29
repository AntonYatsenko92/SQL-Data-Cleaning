What issues will you address by cleaning the data?

##All Sessions Dataset

Replacing "not available in demo dataset" with NULL Values to better get a count of country and city sales spread

~~~~sql
UPDATE allsessions
SET city = NULL
WHERE city = 'not available in demo dataset'
~~~~

Drop columns from dataset that do not add much value as they are either mostly or all NULL values. The following columns were dropped using the syntax below but changing column names (showing single example for conciseness)

1) Currency Code
2) Item Quantity
3) Item Revenue
4) Transaction Revenue
5) Transaction ID
6) Search Keyword
7) Product Revenue

~~~~sql
ALTER TABLE allsessions
DROP COLUMN product_revenue
~~~~

The price column was updated by dividing by 1,000,000 to better reflect the real item price by removing the lagging 0's and then putting the price in dollars and cents.

~~~~sql
UPDATE allsessions
SET price = price / 1000000
~~~~

The revenue column was dropped from this dataset because the total revenue from this column was ~$16K vs ~$2.6M from the analytics table. Furthermore, this column only had 81/15,000 rows complete so it was not reliable

~~~~sql
ALTER TABLE allsessions
DROP COLUMN revenue
~~~~ 

##Analytics Dataset

The unit_price column was updated by dividing by 1,000,000 to better reflect the real item price by removing the lagging 0's and then putting the price in dollars and cents.

~~~~sql
UPDATE analytics
SET unit_price = unit_price / 1000000
~~~~

Remove all rows that have a unit price of $0 as this does not accurately reflect an item in the catalogue

~~~~sql
DELETE FROM analytics
WHERE unit_price = 0
~~~~




