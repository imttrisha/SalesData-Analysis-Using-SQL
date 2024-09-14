# Supermarket Sales Data Analysis

## Project Overview
This project is an end-to-end solution for analyzing supermarket sales data using SQL for database management and Power BI for visualizing insights. The dataset consists of 1000 rows of sales transactions and includes information like invoice ID, branch, product line, customer details, and financial metrics.

## Table of Contents
1. [Database Schema](#database-schema)
2. [Feature Engineering](#feature-engineering)
3. [Data Queries](#data-queries)
4. [Power BI Integration](#power-bi-integration)

## Database Schema
The SQL script creates a `sales` table with the following fields:

- **invoice_id**: Unique identifier for each transaction.
- **branch**: The branch where the sale occurred.
- **city**: City of the branch.
- **customer_type**: Whether the customer is a member or non-member.
- **gender**: Gender of the customer.
- **product_line**: The product category.
- **unit_price**: Price per unit of the product.
- **quantity**: Number of items purchased.
- **tax_pct**: VAT percentage for the transaction.
- **total**: Total transaction amount, including taxes.
- **date**: Date of the transaction.
- **time**: Time of the transaction.
- **payment**: Payment method used (e.g., credit card, cash).
- **cogs**: Cost of goods sold.
- **gross_margin_pct**: Gross margin percentage.
- **gross_income**: Gross income from the sale.
- **rating**: Customer satisfaction rating.

Here is the SQL command to create the table:
```sql
CREATE TABLE IF NOT EXISTS sales (
    invoice_id VARCHAR(30) NOT NULL PRIMARY KEY,
    branch VARCHAR(5) NOT NULL,
    city VARCHAR(30) NOT NULL,
    customer_type VARCHAR(30) NOT NULL,
    gender VARCHAR(30) NOT NULL,
    product_line VARCHAR(100) NOT NULL,
    unit_price DECIMAL(10,2) NOT NULL,
    quantity INT NOT NULL,
    tax_pct FLOAT NOT NULL,
    total DECIMAL(12,4) NOT NULL,
    date DATETIME NOT NULL,
    time TIME NOT NULL,
    payment VARCHAR(15) NOT NULL,
    cogs DECIMAL(10,2) NOT NULL,
    gross_margin_pct FLOAT,
    gross_income DECIMAL(12,4),
    rating FLOAT
);

## Feature Engineering
-**Outlier Detection**
SELECT 
    invoice_id, unit_price, quantity
FROM sales
WHERE unit_price > (SELECT AVG(unit_price) + 3 * STDDEV(unit_price) FROM sales)
OR unit_price < (SELECT AVG(unit_price) - 3 * STDDEV(unit_price) FROM sales)
OR quantity > (SELECT AVG(quantity) + 3 * STDDEV(quantity) FROM sales)
OR quantity < (SELECT AVG(quantity) - 3 * STDDEV(quantity) FROM sales);

-**Add new column sales month**
ALTER TABLE sales ADD COLUMN sales_month INT;
UPDATE sales SET sales_month = MONTH(date);

-**Add new column day of week**
ALTER TABLE sales ADD COLUMN day_of_week VARCHAR(10);
UPDATE sales SET day_of_week = DAYNAME(date);

-**Add new column time of day**
ALTER TABLE sales ADD COLUMN time_of_day VARCHAR(20);
UPDATE sales
SET time_of_day = (
	CASE
		WHEN `time` BETWEEN "00:00:00" AND "12:00:00" THEN "Morning"
        WHEN `time` BETWEEN "12:01:00" AND "16:00:00" THEN "Afternoon"
        ELSE "Evening"
    END
);

## Data Queries

-**How many unique cities does the data have?**
SELECT 
	DISTINCT city
FROM sales;

-**In which city is each branch?**
SELECT 
	DISTINCT city,
    branch
FROM sales;

-**How many unique product lines does the data have?**
SELECT
	DISTINCT product_line
FROM sales;

-**What is the most selling product line?**
SELECT
	SUM(quantity) as qty,
    product_line
FROM sales
GROUP BY product_line
ORDER BY qty DESC limit 1;

-**What is the total revenue by month?**
SELECT
	sales_month AS month,
	SUM(total) AS total_revenue
FROM sales
GROUP BY sales_month 
ORDER BY total_revenue;

-**What month had the largest COGS?**
SELECT
	sales_month AS month,
	SUM(cogs) AS cogs
FROM sales
GROUP BY sales_month 
ORDER BY cogs;

-**Which branch sold more products than average product sold?**
SELECT 
	branch, 
    SUM(quantity) AS qnty
FROM sales
GROUP BY branch
HAVING SUM(quantity) > (SELECT AVG(quantity) FROM sales) limit 1;

-**What product line had the largest revenue?**
SELECT
	product_line,
	SUM(total) as total_revenue
FROM sales
GROUP BY product_line
ORDER BY total_revenue DESC limit 1;

-**What product line had the largest VAT?**
SELECT
	product_line,
	AVG(tax_pct) as avg_tax
FROM sales
GROUP BY product_line
ORDER BY avg_tax DESC limit 1;

-**What is the most common product line by gender?**
SELECT
	gender,
    product_line,
    COUNT(gender) AS total_cnt
FROM sales
GROUP BY gender, product_line
ORDER BY total_cnt DESC;

-**What is the average rating of each product line?**
SELECT
	ROUND(AVG(rating), 2) as avg_rating,
    product_line
FROM sales
GROUP BY product_line
ORDER BY avg_rating DESC;

-**What is the most common customer type?**
SELECT
	customer_type,
	count(*) as count
FROM sales
GROUP BY customer_type
ORDER BY count DESC;

-**Which customer type buys the most?**
SELECT
	customer_type,
    COUNT(*)
FROM sales
GROUP BY customer_type;


-**What is the gender of most of the customers?**
SELECT
	gender,
	COUNT(*) as gender_cnt
FROM sales
GROUP BY gender
ORDER BY gender_cnt DESC;

-**What is the gender distribution per branch?**
SELECT
	gender,
	COUNT(*) as gender_cnt
FROM sales
WHERE branch = "B"
GROUP BY gender
ORDER BY gender_cnt DESC;

-**Which day fo the week has the best avg ratings?**
SELECT
	day_of_week,
	AVG(rating) AS avg_rating
FROM sales
GROUP BY day_of_week 
ORDER BY avg_rating DESC;

-**Number of sales made in each time of the day per weekday?** 
SELECT
	time_of_day,
	COUNT(*) AS total_sales
FROM sales
WHERE day_of_week = "Sunday"
GROUP BY time_of_day 
ORDER BY total_sales DESC;

-**Which of the customer types brings the most revenue?**
SELECT
	customer_type,
	SUM(total) AS total_revenue
FROM sales
GROUP BY customer_type
ORDER BY total_revenue;

-- ------------------------------Complex Queries------------------------------------------------------
-**Calculate the total revenue generated by each customer type over their entire purchasing history.**
SELECT 
    customer_type, 
    SUM(total) AS total_revenue, 
    COUNT(DISTINCT invoice_id) AS total_transactions
FROM sales
GROUP BY customer_type;

-**Analyze how sales for each product line have changed over different months**
SELECT 
    sales_month, 
    product_line, 
    SUM(total) AS monthly_revenue
FROM sales
GROUP BY sales_month, product_line
ORDER BY sales_month, product_line;

-**Find the top product lines by revenue in each branch.**
WITH BranchRevenue AS (
    SELECT 
        branch, 
        product_line, 
        SUM(total) AS total_revenue
    FROM sales
    GROUP BY branch, product_line
)
SELECT 
    branch, 
    product_line, 
    total_revenue
FROM BranchRevenue
WHERE (branch, total_revenue) IN (
    SELECT 
        branch, 
        MAX(total_revenue)
    FROM BranchRevenue
    GROUP BY branch
);

-**Revenue by Gender and Payment Method**
SELECT 
    gender, 
    payment, 
    SUM(total) AS total_revenue
FROM sales
GROUP BY gender, payment
ORDER BY gender, total_revenue DESC;
