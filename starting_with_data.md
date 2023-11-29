**Question 1: The first thing I'd like to understand is how our various marketing platforms are doing. To begin, I'd like to understand what percentage of our channel grouping is most effective at getting customers to the platform.**

~~~~sql
SELECT 
	channel_grouping, 
	count(*),
	count(*) * 100.0 / sum(count(*)) OVER () AS percent_total
FROM analytics
GROUP BY channel_grouping
ORDER BY count(*) DESC
~~~~

 Channel | Percentage Total | 
----------|--------------------|
|Organic Search| 49.13%|
|Referral|23.54%|
|Direct|16.73%|
|Social|4.39%|
|Paid Search|4.38%|
|Affiliates|1.01%|
|Display|0.80%|
|Other|0.00%|

Based on this we can see that Organic Search accounts for 50% of our total traffic and Refferal accounts for another 25%. If I had additional information such as marketing spend across these categories we could look at how effective these tools are at generating traffic. 



**Question 2: Since we don't have marketing spend, I'd like to evaluate the effectiveness of each channel by comparing them to the average number of page views each medium receives**

~~~~sql
SELECT channel_grouping, avg(page_views) AS avg_views FROM analytics
GROUP BY channel_grouping
ORDER BY avg_views DESC
~~~~


| Channel | Average Page Views | 
----------|--------------------|
|Referral| 18.78|
|Paid Search|15.62|
|Direct|13.52|
Organic Search|11.66|
|Display|10.38|
|Affiliates|9.93|
|Social|8.20|
|Other|8|

Based on the table above, we can see that Referrals and Paid Search account for the highest number of average page views. Therefore, if the marketing cost between Organic Search and Referrals are similar, we can conclude the Referrals provide us greater impact in terms of average page views. 

**Question 3: How do different channel mediums affect both average and total revenue generated?**

~~~~sql
SELECT channel_grouping, avg(revenue) FROM analytics
WHERE revenue IS NOT NULL
GROUP BY channel_grouping
ORDER BY avg(revenue) DESC
~~~~

 Channel |Average Revenue | 
----------|--------------------|
|Display|$245.69|
|Referral|$87.60|
|Direct|$57.26|
|Social|$48.94|
|Paid Search|$46.47|
|Organic Search|$34.50|
|Affiliates|$30.04|

Based on this we can see that Display offers the highest average revenue followed by Referral.

~~~~sql
SELECT channel_grouping, sum(revenue) FROM analytics
WHERE revenue IS NOT NULL
GROUP BY channel_grouping
ORDER BY sum(revenue) DESC
~~~~

 Channel |Total Revenue | 
----------|--------------------|
|Referral|$196,394|
|Direct|$29,370|
|Organic Search|$26,084|
|Display|$5,896|
|Paid Search|$5,438|
|Social|$832|
|Affiliates|$480|

Based on total revenue, we can see that Referrals account for an overwhelming amount of our sales, whereas Organic Search accounts for a much lower percentage, implying that page views do not result in higher revenue. 

**Question 4: Is there any correlation between page visits and average or total revenue?**

Beginning with average revenue and comparing it against page views we see no discernable trend. 

~~~~sql
SELECT page_views, avg(revenue) AS revenue FROM analytics
WHERE revenue IS NOT NULL
GROUP BY page_views
ORDER BY revenue DESC
~~~~

 Page Views |Average Revenue | 
----------|--------------------|
|47|$266.58|
|52|$129.37|
|9|$123.61|
|13|$123.18|
|11|$118.63|

We can also look at total revenue to see if there is a trend between number of page views to convert a customer from prospecting to purchasing.

~~~~sql
SELECT page_views, sum(revenue) AS revenue FROM analytics
WHERE revenue IS NOT NULL
GROUP BY page_views
ORDER BY revenue DESC
~~~~

Based on this, we can see total revenue peaking between 10-20 page visits, providing some preliminary information around the peak distribution of page visits and revenue. This makes sense intuitively as there is likely a rate of diminishing returns for customers who continuously visit a page but do not make a purchase (ie. window shoppers)

Page Views |Total Revenue | 
----------|--------------------|
|13|$18,846|
|15|$18,153|
|14|$17,992|
|12|$17,387|
|18|$15,039|

**Question 5: Is there any correlation between quantity ordered and sentiment score?**

~~~~sql
SELECT name, total_ordered, (sentiment_score * sentiment_magnitude) AS overall_sentiment FROM salesreport
ORDER BY total_ordered DESC
LIMIT 5
~~~~

Based on the output of this data, there is no visibile correlation between the quantity of items ordered and their overall sentiment. Therefore, we can conclude that this is not an important factor for the customers of our business based on the data.
