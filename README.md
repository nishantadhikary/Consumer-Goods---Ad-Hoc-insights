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

### 2. **Unique Product Increase in 2021 vs. 2020**
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

