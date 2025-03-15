#-------------------------------------WALMART SALES DATA ANALYSIS------------------------------------------
#--------------------------------------------PART-1(Easy)-------------------------------------------------

CREATE DATABASE IF NOT EXISTS WalmartSalesData;
USE  WalmartSalesData;

CREATE TABLE IF NOT EXISTS sales(
	invoice_id VARCHAR(30) PRIMARY KEY NOT NULL,
    branch VARCHAR(5) NOT NULL,
    city VARCHAR(30) NOT NULL,
    customer_type VARCHAR(30) NOT NULL,
    gender VARCHAR(10) NOT NULL,
    product_line VARCHAR(100) NOT NULL,
    unit_price DECIMAL(10,2) NOT NULL,
    quantity INT NOT NULL,
    VAT FLOAT(6,4) NOT NULL,
    total DECIMAL(10,2) NOT NULL,
    date DATE ,
    time TIME NOT NULL,
    payment_method VARCHAR(30) NOT NULL,
    cogs DECIMAL(10,2) NOT NULL,
    gross_margin_percentage FLOAT(11,9) NOT NULL,
    gross_income DECIMAL(10,2) NOT NULL,
    rating FLOAT(2,1) NOT NULL
);

#----------------------------------------------------------------------------------
# -----------------------------FEATURE ENGINEERING---------------------------------
# -----------------------ADD A NEW COLUMN "time_of_day"----------------------------
SELECT time,
(	CASE
	WHEN time BETWEEN "00:00:00" AND "12:00:00" THEN "Morning"
    WHEN time BETWEEN "12:01:00" AND "16:00:00" THEN "Afternoon"
    WHEN time BETWEEN "16:01:00" AND "20:00:00" THEN "Evening"
    ELSE "Night"
    END
) AS time_of_day FROM sales;
ALTER TABLE sales ADD COLUMN time_of_day VARCHAR(10);
UPDATE sales SET time_of_day = 
(CASE
	WHEN time BETWEEN "00:00:00" AND "12:00:00" THEN "Morning"
    WHEN time BETWEEN "12:01:00" AND "16:00:00" THEN "Afternoon"
    WHEN time BETWEEN "16:01:00" AND "20:00:00" THEN "Evening"
    ELSE "Night"
    END
);

# -----------------------ADD A NEW COLUMN "day_name"-----------------------------------

SELECT date,DAYNAME(date) FROM sales AS day_name;
ALTER TABLE sales ADD COLUMN day_name VARCHAR(10);
UPDATE sales SET day_name = DAYNAME(date);

# -----------------------ADD A NEW COLUMN "month_name"-------------------------------

SELECT date,MONTHNAME(date) FROM sales;
ALTER TABLE sales ADD COLUMN month_name VARCHAR(10);
UPDATE sales SET month_name = MONTHNAME(date);

#------------------------------------------------------------------------------------
# -------------------------EXPLORATORY DATA ANALYSIS(EDA)----------------------------

#---------------------------------Generic Question-----------------------------------
# How many unique cities does the data have?
SELECT DISTINCT city FROM SALES;

# In which city is each branch?
SELECT branch,city FROM sales;

#-------------------------------------Product----------------------------------------
# How many unique product lines does the data have?
SELECT COUNT(DISTINCT product_line) FROM sales;

# What is the most common payment method?
SELECT DISTINCT payment_method,COUNT(payment_method) AS no_of_payment_method
FROM sales
GROUP BY payment_method
ORDER BY no_of_payment_method DESC;

# What is the most selling product line?
SELECT DISTINCT product_line,COUNT(product_line) AS no_of_product_line
FROM sales
GROUP BY product_line
ORDER BY no_of_product_line DESC;

# What is the total revenue by month?
SELECT month_name,SUM(total) AS total_revenue
FROM sales
GROUP BY month_name
ORDER BY total_revenue DESC;

# What month had the largest COGS?
SELECT month_name,MAX(cogs) AS largest_cogs
FROM sales
GROUP BY month_name
ORDER BY largest_cogs DESC;

# What product line had the largest revenue?
SELECT product_line,SUM(total) AS sum_total_revenue,MAX(total) AS largest_revenue
FROM sales
GROUP BY product_line
ORDER BY sum_total_revenue ASC;

# What is the city with the largest revenue?
SELECT city,branch,MAX(total) AS largest_revenue
FROM sales
GROUP BY city,branch;

# What product line had the largest VAT?
SELECT product_line,city,SUM(VAT) AS total_vat,MAX(total) AS largest_vat
FROM sales
GROUP BY product_line,city
ORDER BY largest_vat ASC;

# Fetch each product line and add a column to those product line showing "Good", "Bad". Good if its greater than average sales
SELECT product_line,
	CASE
		WHEN SUM(product_line) > (SELECT AVG(product_line) FROM sales) THEN "Good"
        ELSE "Bad"
	END AS product_line_status
FROM sales
GROUP BY product_line;

# Which branch sold more products than average product sold?
SELECT branch,city,quantity
FROM sales 
GROUP BY branch,city,quantity
HAVING SUM(quantity) > (SELECT AVG(quantity) FROM sales);

# What is the most common product line by gender?
SELECT gender,COUNT(gender) AS cnt_gender,product_line
FROM sales 
GROUP BY gender,product_line
ORDER BY cnt_gender DESC;

# What is the average rating of each product line?
SELECT product_line,AVG(rating) AS avg_rating
FROM sales 
GROUP BY product_line
ORDER BY avg_rating DESC;

#--------------------------------------Sales-----------------------------------------

# Number of sales made in each time of the day per weekday
SELECT SUM(quantity) AS qty ,day_name AS days,time_of_day
FROM sales 
GROUP BY days,time_of_day
ORDER BY qty DESC;

# Which of the customer types brings the most revenue?
SELECT customer_type,SUM(total) AS most_revenue
FROM sales 
GROUP BY customer_type
ORDER BY most_revenue DESC;

# Which city has the largest tax percent/ VAT (Value Added Tax)?
SELECT city,ROUND(MAX(VAT),3) AS largest_tax
FROM sales 
GROUP BY city
ORDER BY largest_tax DESC;

# Which customer type pays the most in VAT?
SELECT customer_type,ROUND(MAX(VAT),3) AS largest_tax
FROM sales 
GROUP BY customer_type
ORDER BY largest_tax DESC;

#------------------------------------Customer----------------------------------------

# How many unique customer types does the data have?
SELECT DISTINCT customer_type FROM sales ;

# How many unique payment methods does the data have?
SELECT DISTINCT payment_method FROM sales ;

# What is the most common customer type?
SELECT customer_type,COUNT(*) AS cnt
FROM sales
GROUP BY customer_type
ORDER BY cnt ASC ;

# Which customer type buys the most?
SELECT customer_type,COUNT(*),SUM(quantity) AS qty
FROM sales 
GROUP BY customer_type
ORDER BY qty DESC;

# What is the gender of most of the customers?
SELECT customer_type,COUNT(*),gender
FROM sales 
GROUP BY customer_type,gender;

# What is the gender distribution per branch?
SELECT branch,COUNT(*),gender
FROM sales 
GROUP BY branch,gender;

# Which time of the day do customers give most ratings?
SELECT customer_type,rating,COUNT(*) AS cnt,time_of_day
FROM sales 
GROUP BY customer_type,rating,time_of_day
ORDER BY cnt DESC;

# Which time of the day do customers give most ratings per branch?
SELECT customer_type,rating,COUNT(*) AS cnt,time_of_day,branch
FROM sales 
GROUP BY customer_type,rating,time_of_day,branch
ORDER BY customer_type,cnt,branch DESC;

# Which day of the week has the best avg ratings?
SELECT customer_type,ROUND(AVG(rating),2) AS avg_rating,day_name
FROM sales 
GROUP BY customer_type,day_name
ORDER BY avg_rating DESC;

# Which day of the week has the best average ratings per branch?
SELECT customer_type,ROUND(AVG(rating),2) AS avg_rating,day_name,branch
FROM sales 
GROUP BY customer_type,day_name,branch
ORDER BY avg_rating,branch DESC;

#-----------------------------------------------------END-----------------------------------------------------------
#-------------------------------------WALMART SALES DATA ANALYSIS--------------------------------------------
#-----------------------------------------PART-2(Hard)-----------------------------------------------

# Using the existing database
USE  WalmartSalesData;

# Identify Peak Sales Time Slot for Each Branch
SELECT branch,
    EXTRACT(HOUR FROM time) AS hour_slot, # extract day,hr,min,month,year from given time
    SUM(total) AS total_sales
FROM sales
GROUP BY branch,hour_slot
ORDER BY branch, total_sales DESC;

# Calculate the Average Gross Margin by Branch and Product Line for Each Month
SELECT branch, product_line,
    EXTRACT(YEAR FROM date) AS year,
    EXTRACT(MONTH FROM date) AS month,
    AVG(gross_margin_percentage) AS avg_gross_margin
FROM sales
GROUP BY branch, product_line, year, month
ORDER BY branch, product_line, year, month;

# Find the Most Popular Payment Method by Gender and City
SELECT gender,city,payment_method,
    COUNT(*) AS payment_method_count
FROM sales
GROUP BY gender, city,payment_method
ORDER BY gender, city, payment_method_count DESC;

# Determine the Variance in Customer Ratings for Each Product Line
SELECT product_line,
    VARIANCE(rating) AS rating_variance # computes the variance of a numeric column
FROM sales
GROUP BY product_line
ORDER BY rating_variance DESC;

# Calculate Monthly Sales Growth Rate for Each Branch
SELECT
    branch_id,
    DATE_TRUNC('month', sale_date) AS sale_month,
    SUM(sale_amount) AS monthly_sales,
    LAG(SUM(sale_amount), 1) OVER (PARTITION BY branch_id ORDER BY DATE_TRUNC('month', sale_date)) AS prev_month_sales,
    (SUM(sale_amount) - LAG(SUM(sale_amount), 1) OVER (PARTITION BY branch_id ORDER BY DATE_TRUNC('month', sale_date))) / 
    LAG(SUM(sale_amount), 1) OVER (PARTITION BY branch_id ORDER BY DATE_TRUNC('month', sale_date)) * 100 AS growth_rate
FROM sales
GROUP BY branch_id, DATE_TRUNC('month', sale_date)
ORDER BY branch_id, sale_month;

# (SUM(sale_amount) - LAG(SUM(sale_amount), 1)) / LAG(SUM(sale_amount), 1) * 100. Computes the growth rate

#--------------------------------------------------END----------------------------------------------------------


