# E-Commerce Sales Analysis

**Author:** Sasmita Sahoo 
**Date:** 07-01-2025

---

# Introduction

This project analyzes sales data from Kaggle about an e-commerce platform to uncover insights, trends, and recommendations using MSSQL for data extraction and Excel for analysis and visualization. The analysis includes:

- Understanding sales performance
- Identifying trends over time
- Analyzing customer purchasing behavior
- Product category performance evaluation
- Recommendations for business growth

# Data Sources

The analysis uses the following datasets:

1. **Order List:** Contains information about orders, including order ID, order date, customer name, state, and city.
2. **Order Details:** Includes details of each order, such as amount, profit, quantity, category, and sub-category.
3. **Sales Target:** Provides monthly sales targets by category.

# Prerequisites

Ensure the following tools are set up:

1. **MSSQL Server:** To query and extract the data.
2. **Excel:** For data cleaning, pivot tables, and visualizations.
3. **SQL Server Management Studio (SSMS):** To manage and execute SQL queries.

# Data Extraction

## Connecting to MSSQL Server

Use the following steps to connect to the MSSQL server:

1. Open SQL Server Management Studio (SSMS).
2. Connect to your database instance.
3. Use the provided SQL queries to extract the required data.

Example query to extract sales data:

```sql
SELECT 
    order_id, 
    product_id, 
    customer_id, 
    sales_amount, 
    order_date 
FROM 
    sales_data;
```

Save the extracted data as CSV files for further processing in Excel.

# Data Cleaning and Preparation

1. **Load Data:** Import the CSV files into Excel.
2. **Data Cleaning:**
   - Remove duplicates.
   - Format date columns.
   - Handle missing values by filling or removing them.
3. **Merging Datasets:** Used MSSQL to merge the datasets based on common variable.

![image](https://github.com/user-attachments/assets/48ba9581-80bc-41e7-be03-511390adfbc6)

# Analysis

## Query 1: Customer Segmentation Using RFM Model

### SQL Code
For convenience, a view called "combined_orders" was created by joining the "order_details" and "order_list" tables. Another view called "customer_grouping" was created using a subquery. 

In the inner query:
- The `STR_TO_DATE()` function converts "order_date" from string to date format.
- The `DATEDIFF()` function calculates the recency for each customer by finding the difference between the dataset's final date (31st March 2019) and the latest transaction date for each customer.
- The `NTILE()` function ranks customers into groups (1-5) for recency, frequency, and monetary values, where higher ranks represent more desirable attributes (e.g., recent purchases, higher frequency, or spending).

In the outer query:
- A `CASE` statement groups customers into segments based on their RFM scores.

### Result
RFM model insights:
- Almost 50% of customers are "loyal" or "champions." Retention strategies include personalized offers and rewards.
- "Potential loyalists" (14.46%) can be converted into loyal customers through loyalty programs.
- "Hibernating" customers need standard communication and attractive offers to re-engage.
- Personalized recommendations and targeted communications can retain "at-risk" and "attention-needed" customers.

## Query 2: Summary Statistics

### SQL Code
To find the number of unique orders, customers, cities, and states:

```sql
SELECT 
    COUNT(DISTINCT order_id) AS num_orders,
    COUNT(DISTINCT customer_name) AS num_customers,
    COUNT(DISTINCT city) AS num_cities,
    COUNT(DISTINCT state) AS num_states
FROM combined_orders;
```

### Result
- 500 orders
- 332 customers
- 24 cities
- 19 states

## Query 3: New Customers in 2019

### SQL Code
Identify new customers in 2019:

```sql
SELECT 
    customer_name, city, state, SUM(amount) AS total_spent
FROM combined_orders
WHERE YEAR(order_date) = 2019 
  AND customer_name NOT IN (
    SELECT customer_name 
    FROM combined_orders
    WHERE YEAR(order_date) = 2018
  )
GROUP BY customer_name, city, state
ORDER BY total_spent DESC;
```

### Result
- Two new customers are from Delhi, indicating high purchasing power.

## Query 4: Top 10 Profitable States and Cities

### SQL Code

```sql
SELECT 
    state, city, SUM(profit) AS total_profit, COUNT(DISTINCT product_id) AS products_sold, COUNT(DISTINCT customer_name) AS customers
FROM combined_orders
GROUP BY state, city
ORDER BY total_profit DESC;
```

### Result
- Most profitable cities: Pune, Indore, Allahabad, and Delhi.

## Query 5: First Order in Each State

### SQL Code

```sql
WITH ranked_orders AS (
    SELECT 
        state, order_id, order_date, customer_name,
        ROW_NUMBER() OVER (PARTITION BY state ORDER BY order_date ASC) AS rank
    FROM combined_orders
)
SELECT state, order_id, order_date, customer_name
FROM ranked_orders
WHERE rank = 1
ORDER BY order_id;
```

### Result
- Delhi is the last state where operations began but generates high profits.

## Query 6: Orders and Sales by Weekday

### SQL Code

```sql
SELECT 
    DAYNAME(order_date) AS weekday,
    COUNT(order_id) AS num_orders,
    SUM(amount) AS total_sales
FROM combined_orders
GROUP BY weekday
ORDER BY FIELD(weekday, 'Sunday', 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday');
```

### Result
- Highest sales occur on Sunday; highest orders occur on Monday.

## Query 7: Monthly Profitability and Quantity

### SQL Code

```sql
SELECT 
    CONCAT(YEAR(order_date), '-', MONTHNAME(order_date)) AS month,
    SUM(profit) AS total_profit,
    SUM(quantity) AS total_quantity
FROM combined_orders
GROUP BY YEAR(order_date), MONTH(order_date)
ORDER BY YEAR(order_date), MONTH(order_date);
```

### Result
- Losses from April to September 2018; profits surge from October 2018 onwards.

## Query 8: Sales vs. Target by Category

### SQL Code

```sql
SELECT 
    category, 
    CASE 
        WHEN SUM(amount) >= target THEN 'Hit'
        ELSE 'Fail'
    END AS target_status,
    COUNT(*) AS occurrences
FROM sales_vs_target
GROUP BY category, target_status;
```

### Result
- Salespeople often fail to meet targets for furniture and clothing.

## Query 9: Sales and Profit by Category and Sub-Category

### SQL Code

```sql
SELECT 
    category, sub_category,
    SUM(amount) AS total_sales,
    SUM(profit) AS total_profit,
    SUM(quantity) AS total_quantity,
    MAX((amount - profit) / quantity) AS max_cost,
    MAX(amount / quantity) AS max_price
FROM combined_orders
GROUP BY category, sub_category
ORDER BY total_sales DESC;
```

### Result
- Best-selling sub-categories: saree, handkerchief, stole.
- Losses in electronic games; focus should shift to printers and accessories.
  
# Conclusion

This project demonstrates how MSSQL and Excel can be used to derive actionable insights from e-commerce sales data. Future work can include integrating advanced tools like Power BI or Tableau for enhanced visualizations.
