# AtliQ Hardware SQL Insights

## Problem Statement

AtliQ Hardware is experiencing rapid growth and has started integrating data analytics into its operations. However, they still rely on Excel for most of their analytics, which leads to insufficient insights for management. The management team tasked junior data analysts with providing 10 ad-hoc insights using SQL. This repository contains the SQL queries that fulfill these requests, based on data stored in various tables.

### Tables used:
- `dim_customer`
- `fact_sales_monthly`
- `dim_product`
- `fact_manufacturing_cost`
- `fact_pre_invoice_deductions`
- `fact_gross_price`

## SQL Queries

### 1. List of Markets for "Atliq Exclusive" in the APAC Region
**Request:** Provide the list of markets in which the customer "Atliq Exclusive" operates in the APAC region.

```sql
SELECT DISTINCT market 
FROM dim_customer
WHERE customer = 'Atliq Exclusive' 
AND region = 'APAC';
```

### 2. Unique Product Increase in 2021 vs. 2020
**Request:** Calculate the percentage increase in unique products between 2021 and 2020.

```sql
WITH cte AS (
    SELECT COUNT(DISTINCT product_code) AS product_count_2021 
    FROM fact_sales_monthly
    WHERE fiscal_year = 2021
),
cte1 AS (
    SELECT COUNT(DISTINCT product_code) AS product_count_2020 
    FROM fact_sales_monthly
    WHERE fiscal_year = 2020
),
cte2 AS (
    SELECT *, product_count_2021 - product_count_2020 AS chg 
    FROM cte, cte1
)
SELECT *, ROUND(chg/product_count_2020*100, 2) AS chg_pct 
FROM cte2;
```
### 3. Unique Product Count per Segment
**Request:** Provide a report with all unique product counts for each segment, sorted in descending order.

```sql
SELECT segment, COUNT(DISTINCT product_code) AS product_count 
FROM dim_product
GROUP BY segment
ORDER BY product_count DESC;
```
### 4. Segment with Highest Increase in Unique Products (2021 vs. 2020)
**Request:** Identify the segment with the most increase in unique products from 2020 to 2021.

```sql
WITH cte AS (
    SELECT segment, COUNT(DISTINCT product_code) AS product_count_2021 
    FROM dim_product AS dp
    JOIN fact_sales_monthly AS f USING (product_code)
    WHERE fiscal_year = 2021
    GROUP BY segment
),
cte1 AS (
    SELECT segment, COUNT(DISTINCT product_code) AS product_count_2020 
    FROM dim_product AS dp
    JOIN fact_sales_monthly AS f USING (product_code)
    WHERE fiscal_year = 2020
    GROUP BY segment
)
SELECT *, product_count_2021 - product_count_2020 AS difference 
FROM cte
JOIN cte1 USING (segment)
ORDER BY difference DESC;
```
### 5. Products with Highest and Lowest Manufacturing Costs
**Request:** Get the product with the highest and lowest manufacturing costs.

```sql
WITH cte AS (
    SELECT dp.product_code, dp.product, ROUND(m.manufacturing_cost, 2) AS max_manufacturing_cost 
    FROM fact_manufacturing_cost AS m 
    JOIN dim_product AS dp ON dp.product_code = m.product_code
    WHERE manufacturing_cost = (SELECT MAX(manufacturing_cost) FROM fact_manufacturing_cost)
),
cte1 AS (
    SELECT dp.product_code, dp.product, ROUND(m.manufacturing_cost, 2) AS min_manufacturing_cost 
    FROM fact_manufacturing_cost AS m 
    JOIN dim_product AS dp ON dp.product_code = m.product_code
    WHERE manufacturing_cost = (SELECT MIN(manufacturing_cost) FROM fact_manufacturing_cost)
)
SELECT * FROM cte
UNION
SELECT * FROM cte1;
```
### 6. Top 5 Customers with Highest Average Discounts (India, 2021)
**Request:** Identify the top 5 customers who received the highest average pre-invoice discount percentage in India during 2021.

```sql
WITH cte AS (
    SELECT customer_code, ROUND(AVG(pre_invoice_discount_pct) * 100, 2) AS avg_pre_invoice_discount 
    FROM fact_pre_invoice_deductions
    WHERE fiscal_year = 2021
    GROUP BY customer_code
    ORDER BY avg_pre_invoice_discount DESC
)
SELECT dc.customer, cte.*, dc.market 
FROM dim_customer AS dc
JOIN cte USING (customer_code)
WHERE market = 'India'
ORDER BY cte.avg_pre_invoice_discount DESC
LIMIT 5;
```
### 7. Monthly Gross Sales for "Atliq Exclusive"
**Request:** Generate a report showing the gross sales amount for each month for the customer "Atliq Exclusive."

```sql
WITH cte AS (
    SELECT *, ROUND(sold_quantity * gross_price, 2) AS gross_sales_amount 
    FROM fact_sales_monthly
    JOIN fact_gross_price USING (product_code, fiscal_year)
    JOIN dim_customer USING (customer_code)
    WHERE customer = 'Atliq Exclusive'
)
SELECT MONTHNAME(date) AS month, fiscal_year AS year, SUM(gross_sales_amount) AS gross_sales_amount
FROM cte
GROUP BY MONTHNAME(date), fiscal_year;
```
### 8. Quarter with Maximum Sold Quantity in 2020
**Request:** Determine which quarter in 2020 had the maximum total sold quantity.

```sql
WITH cte AS (
    SELECT MONTH(date) AS month_num, SUM(sold_quantity) AS total_sold_quantity 
    FROM fact_sales_monthly
    WHERE fiscal_year = 2020
    GROUP BY MONTH(date)
)
SELECT 
    CASE 
        WHEN month_num IN (9, 10, 11) THEN 'Qtr 1'
        WHEN month_num IN (12, 1, 2) THEN 'Qtr 2'
        WHEN month_num IN (3, 4, 5) THEN 'Qtr 3'
        WHEN month_num IN (6, 7, 8) THEN 'Qtr 4'
    END AS Quarter,
    SUM(total_sold_quantity) AS total_sold_quantity
FROM cte
GROUP BY Quarter
ORDER BY total_sold_quantity DESC;
```
### 9. Channel with Highest Gross Sales (2021)
**Request:** Find out which sales channel contributed the most gross sales in 2021 and its percentage contribution.

```sql
WITH cte AS (
    SELECT dc.channel, f.sold_quantity * gp.gross_price AS gross_sales 
    FROM fact_sales_monthly AS f
    JOIN fact_gross_price AS gp ON f.product_code = gp.product_code AND f.fiscal_year = gp.fiscal_year
    JOIN dim_customer AS dc ON dc.customer_code = f.customer_code
    WHERE f.fiscal_year = 2021
),
cte1 AS (
    SELECT channel, ROUND(SUM(gross_sales) / 1000000, 2) AS gross_sales_mln 
    FROM cte
    GROUP BY channel
)
SELECT *, ROUND(gross_sales_mln * 100 / SUM(gross_sales_mln) OVER(), 2) AS percentage 
FROM cte1;
```
### 10. Top 3 Products by Total Sold Quantity per Division (2021)
**Request:** List the top 3 products by total sold quantity in each division for the fiscal year 2021.

```sql
WITH cte AS (
    SELECT division, product_code, product, SUM(sold_quantity) AS total_sold_quantity 
    FROM dim_product AS dp
    JOIN fact_sales_monthly AS f USING (product_code)
    WHERE fiscal_year = 2021
    GROUP BY division, product_code, product
),
cte1 AS (
    SELECT *, ROW_NUMBER() OVER(PARTITION BY division ORDER BY total_sold_quantity DESC) AS ranking 
    FROM cte
)
SELECT * 
FROM cte1
WHERE ranking <= 3;
```
## Conclusion

The SQL queries provided in this repository address the 10 ad-hoc data analysis requests made by the management of AtliQ Hardware. These queries aim to derive actionable insights from the company's data, such as identifying top-performing products, analyzing sales trends, understanding customer behavior, and more. The results generated from these queries can help AtliQ Hardware make more informed and strategic decisions, ultimately leading to improved business performance.

Each query is carefully crafted to answer specific business questions and leverages various datasets from AtliQ's data warehouse, such as customer information, product details, and sales transactions. These insights will guide management in identifying opportunities for growth and addressing challenges across markets and products.

