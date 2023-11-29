# Final-Project-Transforming-and-Analyzing-Data-with-SQL

## Project/Goals

The project had three (3) main goals with a subset of goals built into them:

**Goal #1: Import and Clean the Data** In this step the goal was open and understand the provided data in the .csv files, determine the most relevant data types for import and then import the data using PGAdmin. Once imported, the data was cleaned by removing unnecessary columns and rows, redundant/unnecessary information, and creating new information where necesary


**Goal #2: Validate the Data** Once the data was cleaned and analyzed the next goal was to validate the authenticity of the data prior to proceeding with the analysis. This was done through a number of Quality Assurance (QA) steps such as looking at total orders, SKU/price relationships, date ranges, price ranges, and additional variables. Data that did not make sense (for example SKU's with a price of $0) were omitted where necessary

**Goal #3: Analyze the Data** Once the data was cleaned and validated the final step was to analze the data. The analysis conducted focused on two different aspects:

1) Analyzing product/revenue by country and city
2) Whether channel type and page views are correlated with revenue 

## Process

###Goal #1: Import and Clean the Data

**Step 1:** Open and familarize all data before importing to understand column function and data types. Where applicable empty columns were removed proactively to simply data import into PGAdmin

**Step 2:** Create tables and import data into PGAdmin. The goal was to import the data using relevant column types to minimize downstream casting of data

**Step 3:** Validate data was imported correcting by doing spot check comparisons against .csv files

**Step 4:** Begin cleaning the data. In this step my approach was to use a "top down" beginning with the allsessions table and progressing from most complex to simplest tables. This helped me familiarize myself with how the tables were interlinked. The general format I followed was:

1) Remove unnecessary/redundant columns that were not caught in Step #1. For example, there were multiple revenue columns and the one in the allsessions table was dropped as it did not seem accurate when compared against the total revenue highlighted in the analytics table

2) Clean up data by replacing columns with NULL or other relevant values

3) Modify columns where necessary. For example price and revenue were both divided by 1,000,000

###Goal #2: Validate the Data

**Step 5:** Similar to Goal #1, I took a top-down approach by proceeding with the most complicated table (allsessions) and proceeding through them sequentially in as structured of a manner as possible. Data checks were conducted on all columns to validate data types, information accuracy, and completeness to support the analysis

**Step 6:** Create new tables to account for incomplete data. For example, various SKUs have multiple prices associated with them; to solve this I created a new table that took the average price of the two price discrepencies, given a lack of additional information to validate which price is correct

###Goal #3: Analyze the Data

**Step 7:** Determine which data to analyze. For the structured questions, there was very limited revenue information that linked up to country and city data. Therefore, I decided to use all available revenue information and UNION two different datasets to increase the data set. This only left me with ~80 entries, but it was still significantly better than using either of the two individual datasets (~50 and ~30 rows respectively). I took a similar approach for calculating product quantity by taking the revenue and pricing from the analytics table to create an estimated sales quantity. This increased the data pool available for the analysis

**Step 8:** Conduct analysis by combining relevant tables. In this case the majority of the analysis for the structured questions portion was between the allsessions and analysis tables

**Step 9:** Further analyze data to see what additional insights could be addressed. In this step I looked at what data was available in a meaningful capacity to determine what other insights could be gathered. Since this is an online e-commerce platform I was curious to determine whether channel and page views (significant available datasets) affected revenue as this would help our company optimize our marketing/channel optimization spend

**Step 10:** I created queries to analyze this data and answer some of the questions, knowing the data I was using had been cleaned and validated. This allowed for some insights to be developed and some additional questions answered

## Results

Beginning with the structured questions portion where we looked at product and revenue mix by country and city, the main finding was that the United States and Sunnyvale contributed to the majority of the findings. Furthermore, there was no discernable trend between top-selling product when the data was cut by city and country. The reason for this is likely the sparse data available with approximately only 80 entries that could be tied to country by using the visitid and revenue tables provided across the allsessions and analytics table. Since the dataset was so small, it was prone to outliers (for example one large order) and therefore this data should not be used as to inform any business decisions.

Transitioning to analyzing channel and page view affects on revenue, we can see that Organic Search and Referral drive ~75% of our websites traffic. Breaking this down a level deeper by page views, Referral drives additional page views for our website than organic search. Without understanding what the cost of each referral or Search Engine Optimization click via Organic Search, it is difficult to make inferences but this data provides some quantitative direction as to the effectiveness of each channel in driving traffic to our website.

Diving a level deeper I looked at page view and channel type against average and total revenue. Beginning with channel, average revenue was dominated by Display, however, total revenue was dominated by the Referral channel type, which required further investigation to understand why this was occurring. Furthermore, page views did not offer any correlations except that most total revenue occurred between 13-18 page views, supporting the hypothesis that there is a diminishing return to the number of times a visitor comes to the website and views various pages. 

## Challenges 

The first challenge faced with the project was understanding the provided datasets. While some columns were intuitive there were many columns that were subject to interpretation and ultimatley omitted from the analysis. Combining this with a data structure that didn't fully make sense with having redundancy among columns created confusion. Have a document or guide that provides supporting information, such as a directory, along with target value ranges would have supported creating this database and subsequent analysis.

The second challenge was reconciling redundant or conflicting data. There were several instances (ie. price) that was different while being correlated to the same SKU. An immediate hypothesis was that price discrimination could be occuring based on country or region (which many companies do to maximize profits) however a lack of supporting data (ie. sales via country or city) prevented me from investigating this further. This resulted in me assuming that price is uniform across the organization, which is likely a limiting assumption and not reflective of how an efficient business would operate. Another example would be the varying SKU formats - for example some began with 90XX and others with GGOXX. In this case, I was able to eliminate the former format given they all correlated to a price of 0 dollars, however, having some structure around this could have given me the opportunity to fix the SKUs in greater detail. There are many more examples in the dataset. 

The third and final challenge was dealing with missing values. Without document structure and missing values, it was difficult to conduct meaningful analysis. Specifically, when looking at revenue and product quantity by country/city, working with a dataset of ~80 values made the analysis sensitive to outliers and therefore I would not use the results of this to inform any business decisions. Furthermore, this limited my subsequent analysis as I focused on columns with a meaningful amount of entries to provide some statistically significant results. 

## Future Goals

Each of my goals correlates to one of the challenges outlined above:

My first goal would be to take the time and work with available resources to understand/document the database. This would take significant effort but create a strong foundation to ensuring the dataase is structured properly, each table is designed in a logical way, and additional individuals have access to documentation to be able to interpret the datasets and conduct analysis.

My second goal would be to use this understanding of the datasets to further reconcile and clean up the data. This would include items such as removing additional conflicting information, restructuring the tables in a more organized way, and creating rules/guidelines around data collection moving forward to avoid redundancy/discrepency in the datasets. 

Finally, my third and final goal would be to fill in some of the missing values where applicable. For example, can we use past invoice or PO data to backfill some of the missing city/country data or revenue/sales quantities. If not possible, then perhaps taking the time to gather new information and conducting further validation/QA to ensure the database is operating correctly is critical before using the datasets to inform any business decisions. 
