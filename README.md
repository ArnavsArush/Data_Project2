# CUSTOMER & SALES DATA ANALYSIS w/ SQL 

## INTRODUCTION
This project focuses on analyzing a firm’s customer and sales data to extract meaningful business insights using SQL. The analysis covers key metrics such as customer retention, average order value, and overall sales performance. Advanced concepts including CTEs & subqueries were used to transform raw transactional data into actionable insights.

SQL CODE FILE: [QUERY_FILE](/Query_file) 
               

## SKILLS SHOWCASED
- **Common table expressions (CTEs)**: 
CTEs (Common Table Expressions):
Used CTEs to break down complex queries into logical steps for improved readability and maintainability.Created intermediate result sets to calculate metrics like customer retention, repeat purchases, and total revenue before final analysis.

- **Subqueries**: implemented subqueries to filter high-value customers based on total spending and order frequency.

- **Date Functions**: Applied date functions like **DATE_TRUNC** & **EXTRACT** to analyze trends like **change over time**, including monthly and yearly sales performance. Calculated customer tenure, order gaps, and purchase frequency using timestamp differences.

- **Aggregations & Grouping**: Performed aggregations (SUM, COUNT, AVG) to compute KPIs such as total quantity sold, total orders, average order value, and customer lifetime value. Grouped data by customer, product, and time periods to uncover meaningful sales and retention insights.

- **CASE Expressions**: Implemented CASE statements to segment customers into categories such as VIP, REGULAR & NEW etc.

- **Window Functions**: Used window functions (e.g., LAG, LEAD, running totals) to analyze change over time in sales, enabling period-over-period comparisons such as year-over-year growth.

## TOOLS I USED
- **SQL**: The backbone of my analysis, allowing me to query the database and unearth critical insights.
- **PostgreSQL**: The chosen database management system to store database.
- **Visual Code Studio**: My go-to software for data analysis i.e to execute queries.
- **Github**: Essential for sharing my SQL scripts and analysis.

## THE ANALYSIS
- Q1 Analyze sales performance over the years?
```SQL
SELECT EXTRACT(YEAR FROM shipping_date) AS shipping_year, 
SUM(sales_amount) AS total_sales
FROM sales
GROUP BY shipping_year;
```

- Q2 Find running total of sales amount by month?
```SQL
SELECT shipping_month, total_sales,
SUM(total_sales) OVER(ORDER BY shipping_month) AS running_total_sales
FROM(
SELECT DATE_TRUNC('MONTH', shipping_date) AS shipping_month, 
SUM(sales_amount) AS total_sales
FROM sales
GROUP BY DATE_TRUNC('MONTH', shipping_date)
ORDER BY DATE_TRUNC('MONTH', shipping_date));
```

- Q3 Analyze the yearly performance of products by comparing each product's sales to both its average sales performance &
the previous year's sales.
```SQL
WITH current_year_performance AS(
SELECT EXTRACT (YEAR FROM l.shipping_date) AS sales_year,
r.product_name,
SUM(l.sales_amount) AS current_sales
FROM sales l
LEFT JOIN products r
ON l.product_key = r.product_key
GROUP BY EXTRACT(YEAR FROM l.shipping_date), r.product_name
)
--CURRENT YEAR SALES PERFORMANCE^.

SELECT  sales_year, product_name, current_sales,
--CURRENT YEAR SALES PERFORMANCE VS AVG SALES PERFORMANCE
AVG(current_sales) OVER (PARTITION BY product_name) AS avg_sales,
current_sales - AVG(current_sales) OVER (PARTITION BY product_name) AS deviation,
CASE WHEN current_sales - AVG(current_sales) OVER (PARTITION BY product_name) > 0 THEN 'Above Average'
WHEN current_sales - AVG(current_sales) OVER (PARTITION BY product_name) < 0 THEN 'Below Average'
ELSE 'Average' END avg_change, --END
--CURRENT YEAR SALES PERFORMANCE VS PREVIOUS YEAR SALES PERFORMANCE
LAG(current_sales) OVER( PARTITION BY product_name ORDER BY sales_year) AS previous_year_sales,
current_sales - LAG(current_sales) OVER( PARTITION BY product_name ORDER BY sales_year) AS c_p_deviation,
CASE WHEN current_sales - LAG(current_sales) OVER( PARTITION BY product_name ORDER BY sales_year) > 0 THEN 'Increased'
WHEN current_sales - LAG(current_sales) OVER( PARTITION BY product_name ORDER BY sales_year) < 0 THEN 'Decreased'
ELSE 'No Change' END p_y_change
FROM current_year_performance
ORDER BY product_name;
```
- Q4 WHICH CATEGORIES HAVE THE HIGHEST SALES FROM THE OVERALL SALES?
```SQL
WITH sales_per_category AS(SELECT b.category, SUM(a.sales_amount) AS total_sales
FROM sales a 
LEFT JOIN products b
ON a.product_key = b.product_key
GROUP BY b.category)
--CONTRIBUTION OF EACH CATEGORY TO OVERALL SALES
SELECT category, total_sales,
SUM(total_sales) OVER() AS overall_sales,
(total_sales/ SUM(total_sales) OVER ())* 100 AS sales_contribution 
FROM sales_per_category;
```
- Q5 SEGMENT PRODUCTS INTO COST RANGES AND COUNT HOW MANY PRODUCTS FALL INTO EACH RANGE?
```SQL
WITH cost_ranges AS(
SELECT product_name,cost,
CASE WHEN cost < 100 THEN 'below 100'
WHEN cost BETWEEN 100 AND 500 THEN '100-500'
WHEN cost BETWEEN 500 AND 1000 THEN '500-1000'
ELSE 'above 1000' END AS cost_range
From products)
--COUNT OF PRODUCTS IN EACH COST RANGE
SELECT cost_range, COUNT(product_name) AS product_count
FROM cost_ranges
GROUP BY cost_range;
```


- Q6 GROUP CUSTOMERS INTO THREE SEGMENTS BASED ON THEIR SPENDING BEHAVIOR:

- **VIP**- AT LEAST 12 MONTHS OF HISTORY AND TOTAL SPENDING ABOVE 5000

- **REGULAR**- AT LEAST 12 MONTHS OF HISTORY AND TOTAL SPENDING 5000 OR LESS 

- **NEW**- LESS THAN 12 MONTHS OF HISTORY
```SQL
WITH lifespan AS (SELECT b.customer_key,
SUM( a.sales_amount) AS total_spending,
MIN(a.shipping_date) AS first_purchase_date,
MAX(a.shipping_date) AS last_purchase_date,
(EXTRACT(YEAR FROM MAX(a.shipping_date)) - EXTRACT(YEAR FROM MIN(a.shipping_date))) * 12 +
(EXTRACT(MONTH FROM MAX(a.shipping_date)) - EXTRACT(MONTH FROM MIN(a.shipping_date))) AS months_of_history
FROM customers b 
LEFT JOIN sales a
ON b.customer_key = a.customer_key
GROUP BY b.customer_key)
--CUSTOMER SEGMENTATION BASED ON SPENDING BEHAVIOR
SELECT COUNT(customer_key) AS number_of_customers, 
CASE WHEN months_of_history >= 12 AND total_spending > 5000 THEN 'VIP'
WHEN months_of_history >= 12 AND total_spending <= 5000 THEN 'REGULAR'
ELSE 'NEW' END AS customer_segment
FROM lifespan
GROUP BY customer_segment;
```


## SUMMARY
In simple words what I did in the analysis was:

- Find **Change-over-time** of sales.
- Did **Cumulative analysis** of sales using window functions.
- Did **Performance analysis** of products.
- Did **Part-to-whole analysis** of each category to find out which category out of all is performing best in market. 
- **Segmented customers** into three categories and found insights related to them.

