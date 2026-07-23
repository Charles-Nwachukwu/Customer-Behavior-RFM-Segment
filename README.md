# Customer-Behavior-RFM-Segment
RFM customer segmentation on a real UK e-commerce dataset (2010-2011, 4,338 customers). Segmented customers by Recency, Frequency, and Monetary value using DAX in Power BI to identify Champions, At-Risk, and high-value Big Spenders. Includes churn analysis, country breakdown, and revenue-by-segment insights.
# Executive Summary
This project applies RFM (Recency, Frequency, Monetary) segmentation to a real e-commerce transaction dataset covering December 2010 to December 2011. The goal was to move beyond simple totals and understand customer behavior at a granular level  identifying which customers drive the business, which are at risk of leaving, and where revenue is genuinely concentrated. Across 4,338 customers, the analysis found that a small number of highly engaged customers (Champions) generate the majority of revenue, that repeat customers account for 93% of total revenue despite being a minority of transactions, and that customer value does not always correlate with visible activity some of the highest-spending customers order the least often.
# Project Overview
The project follows a complete analytics workflow: sourcing a real transactional dataset, cleaning and aggregating it to the customer level, scoring and segmenting customers using the RFM framework, validating every metric against the raw data, and presenting findings through an interactive Power BI dashboard. The intent was to demonstrate not just the final dashboard, but the full reasoning behind each step including assumptions made, limitations encountered, and errors caught and corrected along the way.
# Data Source
The dataset originates from the UCI Machine Learning Repository's Online Retail dataset  real transaction records from a UK-based, non-store online retailer specializing in all-occasion gifts (many customers are wholesalers rather than individual consumers).

Raw dataset size: 541,909 transaction-level rows

Time span: 1 December 2010 – 9 December 2011

Raw fields: InvoiceNo, StockCode, Description, Quantity, InvoiceDate, UnitPrice, CustomerID, Country

Final customer-level dataset: 4,338 unique customers, 12 fields
# Problem Statement
Raw transaction data on its own does not answer the questions a business actually needs answered:

Who are our most valuable customers?

Which customers are at risk of leaving, and how many have already gone quiet?

Where geographically is revenue concentrated, and how reliable are those figures given very different sample sizes by country? 

This project was built to turn a raw transaction log into direct, decision-relevant answers to those questions  while being explicit about what the data can and cannot support.
# Tools and Methodology
#### Data Collection
Source data was obtained directly from the UCI Machine Learning Repository as a CSV export of real transaction records, spanning roughly 13 months of trading activity.
#### Data Cleaning and Preparation
Performed in Power Query (Excel). Four filters were applied to the raw data before any analysis:
* 	Removed cancelled orders (InvoiceNo values starting with 'C')
* 	Removed rows with no CustomerID, since RFM scoring requires attributing every transaction to a customer
* 	Removed non-positive Quantity values (returns and data entry errors)
*   Removed non-positive UnitPrice values (free items and data entry errors)
#### Data Transformation and Processing
Cleaned transaction-level data was aggregated from ~500,000 line items into one row per customer, computing Recency (days since last purchase, measured from a fixed snapshot date of 10 December 2011), Frequency (count of distinct orders), Monetary value (total revenue), Average Order Value, Customer Lifespan, Repeat Customer status, and Churn Status (no purchase within 90 days of last order).
#### Calculated Fields and DAX Measures
Within Power BI, each customer was scored 1–5 on Recency, Frequency, and Monetary value using rank-based quintiles (RANKX combined with CEILING logic), then assigned to one of seven behavioral segments using a prioritized SWITCH(TRUE()) statement. Supporting DAX measures included Weighted AOV (total revenue ÷ total orders, rather than a simple average of per-customer averages), Churn Rate, Total Customers, and a Small Sample Flag used to visually warn against over-interpreting country-level figures built on very few customers.
#### Data Validation
Every dashboard figure was cross-checked against independent Python calculations on the same underlying dataset before being accepted  including the churn rate, weighted AOV, segment counts, segment-level revenue, and segment-level order volume. This process caught and corrected a classification gap that had left 17.7% of customers unclassified in an early version of the segmentation logic, and confirmed that all final segment counts sum exactly to the full 4,338-customer base.
#### Data Modeling
The Power BI data model is built on a single fact table at the customer grain, with calculated columns (R-Score, F-Score, M-Score, Segment) sitting alongside the base aggregated fields. Scores were deliberately built as calculated columns rather than measures, since each customer requires one fixed value per row rather than a value that changes with filter context.
#### Data Visualization
The final Power BI dashboard presents five top-line KPI cards, customer behavior by repeat-purchase and churn status, a country-level revenue and churn table (filtered to the top 10 by revenue, with small-sample warnings), and four parallel views of the seven RFM segments by customer count, revenue, weighted AOV, and total order volume.
# Exploratory Data Analysis
#### Key Patterns
* 34.4% of customers are one-time buyers (Frequency = 1) — over a third of the customer base has never returned for a second order
* 93% of total revenue is generated by repeat customers, despite one-time buyers making up over a third of the customer count
* Customer value and order frequency are related but not interchangeable  some of the highest-spending individual customers order infrequently
#### Distribution
Monetary value is heavily right-skewed: the mean customer spend is $2,054, but the median is only $674 — meaning a relatively small number of high-spending customers pull the average well above what a typical customer actually spends. The same right-skew pattern holds for order Frequency (mean 4.27 orders, median 2 orders).
|Metric | Mean |	Median |	Min |	Max |
| ---      | --- | --- | --- | --- |
|Monetary ($) |	2,054.27 |	674.48 |	3.75 | 280,206.02 |
|Frequency (orders) |	4.27	| 2	| 1 |	209 |
|Recency (days) |	92.54 |	51 |	1 |	374 |
#### Trends
The customer-level dataset records only each customer's first and last purchase date, not every individual transaction date so month-by-month sales trend or seasonality analysis is not possible from this file. A transaction-level view (with a date on every order) would be required to examine trends such as seasonal spikes; this is noted as a boundary of the current dataset rather than a gap in the analysis performed.
#### Outliers
Two individual customers stand out clearly from the rest of the base:
* Customer 16446 (United Kingdom) generated $168,472.50 in revenue from just 2 orders  an average order value of roughly $84,236, far above any typical customer, consistent with a large one-off wholesale purchase
* Customer 14646 (Netherlands) generated the single highest total revenue in the dataset $280,206.02 across 73 orders which materially explains why the Netherlands shows an unusually high country-level average order value despite having only 9 total customers
These outliers are a direct, concrete reason the dashboard flags small-sample countries: a single high-value customer can visibly distort a country-level average when the underlying customer count is small.
#### Correlation
A correlation check across the three RFM dimensions shows a moderate positive relationship between order Frequency and Monetary value (0.55)  customers who order more often do tend to spend more in total, which is expected. Recency shows only a weak relationship with both Frequency (-0.26) and Monetary value (-0.12), meaning how recently a customer purchased is not a strong standalone predictor of how much they spend or how often they buy.
|      |	Recency|	Frequency|	Monetary|
|----- | ----- |-----|-----|
|Recency |	1.00 |-0.26	| -0.12 |
|Frequency	|-0.26 |	1.00	| 0.55 |
|Monetary |	-0.12|	0.55 |	1.00 |
# Key Insights
* Repeat customers are the business they generate 93% of revenue from a minority of total transactions
* Champions (1,035 customers) generate $5.8M in revenue the clear core of the business
* A small, quiet segment (Big Spenders At Risk, 374 customers) holds the highest average order value of any segment ($777.67) despite low visible activity an easily overlooked, high-value group
* Hibernating customers (951 people, the second-largest segment) contribute the least revenue of any segment, indicating a large population that is both inactive and was never high-value
* Country-level figures for accounts under 20 customers should be read with caution individual outlier customers can swing those numbers substantially
# Visual / Dashboard Preview
Final Power BI dashboard, showing top-line KPIs, churn and repeat-customer behavior, the country revenue table, and the four-part RFM segment breakdown

## Customer Behavior RFM Dashboard

![Customer Behavior RFM Dashboard](Customer%20Behavior%20RFM%20Dashboard.png)
























