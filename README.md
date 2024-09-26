# AtliQ Hardware SQL Insights

## Problem Statement

AtliQ Hardware, a rapidly growing consumer electronics company, relies heavily on Excel for data analytics, which has led to insufficient insights. The management team requested 10 ad-hoc insights using SQL to improve decision-making, and this repository contains the SQL queries that fulfill those requests.

## Ad-Hoc Requests and SQL Solutions

1. **List of Markets for "Atliq Exclusive" in APAC Region**
   - Query to find the list of markets in which the customer "Atliq Exclusive" operates in the APAC region.
   ```sql
   SELECT DISTINCT market 
   FROM dim_customer
   WHERE customer = 'Atliq Exclusive' 
     AND region = 'APAC';
