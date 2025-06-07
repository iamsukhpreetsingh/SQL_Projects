# 🚴‍♀️ Bike Rental Shop Database Analysis 
(source: https://learnsql.com/course/2023-advanced-sql-practice-challenges/november-challenge/introduction/introduction/)

A comprehensive SQL analysis project for a bike rental business, featuring advanced queries for business intelligence, revenue analysis, and operational insights.

##  Project Overview

This project demonstrates advanced SQL techniques through real-world business scenarios for a bike rental shop. The analysis covers inventory management, customer analytics, pricing strategies, and revenue reporting.

##  Database Schema

The database consists of five main tables:

- **`customer`** - Customer information and profiles
- **`bike`** - Bike inventory with categories, pricing, and availability status
- **`rental`** - Rental transactions and history
- **`membership_type`** - Different membership plan types and pricing
- **`membership`** - Customer membership purchases and details

## 🎯 Business Problems Solved

### 1. Inventory Analysis by Category
**Problem**: Emily needs to know how many bikes the shop owns by category (showing only categories with more than 2 bikes).

**Key SQL Concepts**:
- `GROUP BY` with aggregate functions
- `HAVING` clause for filtering aggregated results

**Solution**:
```sql
SELECT category, COUNT(*) AS number_of_bikes 
FROM bike 
GROUP BY category
HAVING COUNT(*) > 2;
```

### 2. Customer Membership Analytics
**Problem**: Generate a comprehensive list of customers with their total membership purchases, including customers with zero memberships.

**Key SQL Concepts**:
- Common Table Expressions (CTEs)
- Window functions with `PARTITION BY`
- `RIGHT JOIN` to include all customers
- `COALESCE` for handling NULL values

**Solution**:
```sql
WITH cte1 AS (
    SELECT * FROM (
        SELECT customer_id, 
        COUNT(membership_type_id) OVER (PARTITION BY customer_id) AS membership_count
        FROM membership 
    )
    GROUP BY customer_id, membership_count
)
SELECT b.name, b.id, COALESCE(a.membership_count, 0) AS membership_count
FROM cte1 a 
RIGHT JOIN customer b ON a.customer_id = b.id
ORDER BY membership_count DESC;
```

### 3. Dynamic Pricing Strategy
**Problem**: Create a winter discount pricing model with category-specific discount rates.

**Key SQL Concepts**:
- Complex `CASE WHEN` statements
- Percentage calculations
- `ROUND` function for price formatting
- Multi-condition logic

**Discount Structure**:
- Electric bikes: 10% hourly, 20% daily
- Mountain bikes: 20% hourly, 50% daily  
- Other bikes: 50% for all rentals

**Solution**:
```sql
SELECT id, category, 
       price_per_hour AS old_price_per_hour, 
       price_per_day AS old_price_per_day,
       ROUND(CASE 
           WHEN category = 'electric' THEN price_per_hour - (0.10 * price_per_hour)
           WHEN category = 'mountain bike' THEN price_per_hour - (0.20 * price_per_hour)
           ELSE price_per_hour - (0.50 * price_per_hour)
       END, 2) AS new_price_per_hour,
       ROUND(CASE 
           WHEN category = 'electric' THEN price_per_day - (0.20 * price_per_day)
           WHEN category = 'mountain bike' THEN price_per_day - (0.50 * price_per_day)
           ELSE price_per_day - (0.50 * price_per_day)
       END, 2) AS new_price_per_day
FROM bike
ORDER BY category;
```

### 4. Operational Status Reporting
**Problem**: Track available vs. rented bikes by category for inventory management.

**Key SQL Concepts**:
- Multiple CTEs for different status types
- Window functions for counting
- `UNION ALL` for combining results
- Complex grouping strategies

**Solution**:
```sql
WITH cte_available AS (
    SELECT category, status,
           COUNT(status) OVER(PARTITION BY category) AS rn
    FROM bike 
    WHERE status = 'available'
),
cte_rented AS (
    SELECT category, status,
           COUNT(status) OVER(PARTITION BY category) AS rn
    FROM bike 
    WHERE status = 'rented'
)
SELECT * FROM (
    SELECT * FROM cte_available
    UNION ALL
    SELECT * FROM cte_rented
) t
GROUP BY category, status, rn;
```

### 5. Revenue Analysis by Time Period
**Problem**: Generate comprehensive revenue reports showing monthly, yearly, and all-time totals.

**Key SQL Concepts**:
- `EXTRACT` function for date manipulation
- Nested CTEs for multi-level aggregation
- Window functions with `PARTITION BY`
- Hierarchical data presentation

**Solution**:
```sql
WITH cte AS (
    SELECT start_timestamp, 
           EXTRACT(YEAR FROM start_timestamp) AS year, 
           EXTRACT(MONTH FROM start_timestamp) AS month, 
           total_paid
    FROM rental
),
per_month AS (
    SELECT year, month, SUM(total_paid) AS revenue_per_month
    FROM cte
    GROUP BY year, month
    ORDER BY year, month
),
per_year AS (
    SELECT year, month, revenue_per_month,
           SUM(revenue_per_month) OVER(PARTITION BY year) AS revenue_per_year
    FROM per_month
)
SELECT year, month, revenue_per_month, revenue_per_year
FROM per_year;
```

### 6. Membership Revenue Breakdown
**Problem**: Analyze membership revenue by year, month, and membership type combinations.

**Key SQL Concepts**:
- Multi-table joins
- Date extraction and grouping
- Revenue aggregation by multiple dimensions

**Solution**:
```sql
SELECT *,
       SUM(price) OVER(PARTITION BY month) AS total_revenue_per_month
FROM (
    SELECT 
        EXTRACT(YEAR FROM a.start_date) AS year,
        EXTRACT(MONTH FROM a.start_date) AS month,
        b.name,
        b.price
    FROM membership a
    JOIN membership_type b ON a.membership_type_id = b.id 
);
```

### 7. Advanced Reporting with Subtotals
**Problem**: Create a comprehensive 2023 membership report with subtotals and grand totals.

**Key SQL Concepts**:
- Advanced window functions
- Multiple aggregation levels
- Sorted results by multiple criteria

**Solution**:
```sql
SELECT 
    b.name AS membership_type_name,
    EXTRACT(MONTH FROM a.start_date) AS month,
    b.price AS price_per_month,
    SUM(b.price) OVER(PARTITION BY name) AS price_per_category,
    SUM(b.price) OVER() AS Total_revenue
FROM membership a
LEFT JOIN membership_type b ON a.membership_type_id = b.id 
ORDER BY name;
```

## 🛠️ Technical Skills Demonstrated

### Advanced SQL Techniques
- **Common Table Expressions (CTEs)** - Multiple and nested CTEs for complex logic
- **Window Functions** - `COUNT() OVER()`, `SUM() OVER()` with partitioning
- **Complex Joins** - RIGHT JOIN, LEFT JOIN for comprehensive data retrieval
- **Aggregate Functions** - GROUP BY, HAVING, COUNT, SUM
- **Date Functions** - EXTRACT for year/month analysis
- **Conditional Logic** - Multi-level CASE WHEN statements
- **Mathematical Operations** - Percentage calculations and rounding

### Data Analysis Concepts
- **Customer Segmentation** - Analyzing customer behavior and membership patterns
- **Inventory Management** - Tracking bike availability and utilization
- **Revenue Analytics** - Time-series analysis and financial reporting
- **Pricing Strategy** - Dynamic discount modeling
- **Business Intelligence** - Operational metrics and KPI tracking

## 📈 Business Impact

This analysis provides Emily's bike rental shop with:

1. **Inventory Optimization** - Understanding which bike categories are most/least stocked
2. **Customer Insights** - Identifying high-value customers and membership patterns  
3. **Revenue Strategy** - Data-driven pricing and discount strategies
4. **Operational Efficiency** - Real-time availability tracking
5. **Financial Planning** - Comprehensive revenue reporting across time periods

---

*The Queries are 100% written by me and this Readme was generated by Claude :).*
