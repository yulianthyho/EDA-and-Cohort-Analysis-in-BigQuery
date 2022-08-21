# TheLook Ecommerce : Exploratory Data Analysis (EDA) and Cohort Analysis in BigQuery 🛒👚
Managed to do EDA and find out the user retention for the company to see the monthly retention until three following months.

- Tools : SQL in BiqQuery and Google Sheets
- Check out my query [here](https://console.cloud.google.com/bigquery?sq=1072889830804:e1c2f3c2c8194a669cdfdbdd7ec7643b)

## Overview of Dataset
TheLook is a fictitious eCommerce clothing site developed by the Looker team. The dataset contains information about customers, products, orders, logistics, web events and digital marketing campaigns. The contents of this dataset are synthetic, and are provided to industry practitioners for the purpose of product discovery, testing, and evaluation. (source : thelook-ecommerce [dataset](https://console.cloud.google.com/marketplace/product/bigquery-public-data/thelook-ecommerce?q=search&referrer=search&project=sincere-torch-350709)) 

## Exploratory Data Analysis
### 1. Monthly Orders and Users
Get the total users who completed the order and total orders per month (Jan 2019-Apr 2022)
 ``` sql
 SELECT
    DATE_TRUNC(DATE(created_at),month) as month,
    COUNT(DISTINCT user_id) as total_user,
    COUNT(order_id) as total_order
  FROM `bigquery-public-data.thelook_ecommerce.orders`
  WHERE DATE(created_at) BETWEEN '2019-01-01' AND '2022-04-30'
  AND status = 'Complete'
  GROUP BY 1
  ORDER BY 1
  ```
 **Table results:**
 
![image](https://user-images.githubusercontent.com/100077706/185785356-ef9d19d9-f664-481d-af5f-79e6a30bf06b.png)

**Insight** : From Jan 2019- April 2022 we can see that the total order and total user relatively growing over years.

![image](https://user-images.githubusercontent.com/100077706/185786090-16553264-4610-415d-8bca-6a399365954f.png)


### 2. Average Order Value (AOV) and Distinct User
Get average order value and total number of unique users, grouped by month  (Jan 2019-Apr 2022)
```sql
SELECT 
DATE_TRUNC(DATE(oi.created_at),MONTH) AS month_year,
COUNT(DISTINCT oi.user_id) AS distinct_users,
ROUND(SUM(sale_price)/ COUNT(DISTINCT oi.order_id),2) AS average_order_value
FROM `bigquery-public-data.thelook_ecommerce.order_items` AS oi
INNER JOIN `bigquery-public-data.thelook_ecommerce.orders` AS o
ON oi.order_id = o.order_id
WHERE DATE(oi.created_at) BETWEEN '2019-01-01' AND '2022-04-30'
AND oi.status = 'Complete'
GROUP BY month_year
ORDER BY month_year
```
 **Table results:**
 
 ![image](https://user-images.githubusercontent.com/100077706/185785701-36c7dba5-3fe0-4229-a068-eb57756b72ad.png)
 
**Insight** : From Jan 2019- April 2022 we can see that the number of user increase over the year.

![image](https://user-images.githubusercontent.com/100077706/185786232-79007d02-afd7-4d2d-ab45-311c2875a2e2.png)

 
 ### 3. Customer Age Category
 Find the first and last name of users from the youngest and oldest age of each gender (Jan 2019-Apr 2022)
 ```sql
 SELECT
first_name,
last_name,
gender,
MIN(age) OVER(PARTITION BY gender) AS youngest_age
FROM `bigquery-public-data.thelook_ecommerce.users` 
WHERE DATE(created_at) BETWEEN '2019-01-01' AND '2022-04-30'
AND age IN (SELECT MIN(age) 
              FROM `bigquery-public-data.thelook_ecommerce.users`)
UNION ALL
SELECT
first_name,
last_name,
gender,
MAX(age) OVER(PARTITION BY gender) AS oldest_age
FROM `bigquery-public-data.thelook_ecommerce.users` 
WHERE DATE(created_at) BETWEEN '2019-01-01' AND '2022-04-30'
AND age IN (SELECT MAX(age) 
              FROM `bigquery-public-data.thelook_ecommerce.users`)
ORDER BY youngest_age
```
**Table results:**

![image](https://user-images.githubusercontent.com/100077706/185785810-32dff763-c021-48ae-a80c-fe5f1974dfe8.png)

**Insight**

![image](https://user-images.githubusercontent.com/100077706/185786288-53101157-fbb5-45c1-a420-a5f189aa8405.png)

### 4. Top 5 Monthly Product
Get the top 5 most profitable product and its profit detail breakdown by month
```sql
WITH profitable as 
(
  SELECT 
    DISTINCT oi.product_id, product_name,
    DATE_TRUNC (date(oi.created_at),month) AS month,
    ROUND(SUM(sale_price),2) AS sales,
    ROUND(SUM(cost),2) AS cost,
    ROUND((SUM(sale_price) - SUM(cost)),2) AS profit
  FROM `bigquery-public-data.thelook_ecommerce.order_items` AS oi
  JOIN `bigquery-public-data.thelook_ecommerce.inventory_items` AS i
  ON oi.product_id = i.product_id
  WHERE DATE(oi.created_at) BETWEEN '2019-01-01'AND '2022-04-30'
  AND oi.status= 'Complete'
  GROUP BY 1,2,3
  ORDER BY month
),
ranks as
(
  SELECT
    month,
    product_id,
    product_name,
    sales,
    cost,
    profit,
    RANK() OVER (PARTITION BY month ORDER BY profit DESC) AS rank_per_month
  FROM profitable
  ORDER BY profitable.month)

SELECT ranks.*
FROM ranks
WHERE rank_per_month <=5
```
**Table results:**
![image](https://user-images.githubusercontent.com/100077706/185785892-10a63ad4-846d-4b89-be68-ea517c997f55.png)

### 5. Month to Date Revenue per Category
A query to get Month to Date of total revenue in each product categories of past 3 months (current date 15 April 2022), breakdown by date
```sql
WITH products as 
(
  SELECT 
    DISTINCT category AS product_categories,
    DATE_TRUNC(DATE(oi.created_at),day) AS dates, 
    DATE_TRUNC(DATE(oi.created_at),month) AS month, 
    ROUND(SUM(sale_price),2) AS revenue
  FROM `bigquery-public-data.thelook_ecommerce.order_items` AS oi
  JOIN `bigquery-public-data.thelook_ecommerce.products` as p
  ON oi.product_id= p.id
  WHERE DATE(oi.created_at) BETWEEN '2022-01-15'AND '2022-04-15'
  AND oi.status = 'Complete'
  GROUP BY 1,2,3
  ),
revenues as
(
  SELECT 
    dates,
    product_categories,
    revenue,
    SUM(revenue) OVER (PARTITION BY product_categories, month ORDER BY dates) AS running_total_revenue
  FROM products
  ORDER BY dates DESC)

SELECT
  dates,
  product_categories,
  revenue,
  ROUND(running_total_revenue,2) AS running_total_revenue
FROM revenues
ORDER BY 2, dates desc
```
**Table results:**

![image](https://user-images.githubusercontent.com/100077706/185785979-deae9ddd-cb94-4f4b-bc6a-5855e792359f.png)

### 6. Business Growth by Users and Revenue
Monthly growth of TPO (#of completed orders) and TPV (#of revenue) in percentage breakdown by product categories, ordered by time descendingly. 
```sql
WITH CTE as 
(
  SELECT 
    DISTINCT category,
    COUNT(oi.order_id) AS total_order,
    DATE_TRUNC(DATE(oi.created_at),month) AS month,
    ROUND(SUM(sale_price),2) AS revenue
  FROM `bigquery-public-data.thelook_ecommerce.order_items` AS oi
  JOIN `bigquery-public-data.thelook_ecommerce.products` as p
  ON oi.product_id = p.id
  WHERE DATE(oi.created_at) BETWEEN '2019-01-01' AND '2022-04-30'
  AND oi.status= 'Complete'
  GROUP BY month, category
  ORDER BY month DESC, category
),
before AS 
(
  SELECT 
    month,
    category,
    total_order,
    LEAD(total_order) OVER (PARTITION BY category ORDER BY month DESC) AS total_order_last_month,
    revenue,
    LEAD(revenue) OVER(PARTITION BY category ORDER BY month DESC) AS total_revenue_last_month
  FROM cte
  ORDER BY month DESC, category
),
growth AS
(
SELECT 
  month,
  category,
  total_order,
  total_order_last_month,
  ROUND((total_order-total_order_last_month)/total_order_last_month*100,2) AS order_growth,
  revenue,
  total_revenue_last_month,
  ROUND((revenue-total_revenue_last_month)/total_revenue_last_month*100,2) as revenue_growth
  FROM before
  ORDER BY month DESC, category
  )

SELECT 
  month, 
  category,
  order_growth,
  revenue_growth
FROM growth
ORDER BY category, month DESC
```
**Table results:**

![image](https://user-images.githubusercontent.com/100077706/185786450-54650b16-f44e-47b1-a341-4f7b9c3f4aaf.png)

**Insights :**
- Each order growth and revenue growth per product category is fluctuative over months.
- There are several additional categories as the months go by. Starting 17 category in Jan 2019 to 26 categories as of April 2022

**Recommendation:**
- Since fashion always changing and following on whats on trends, TheLook Ecommerce can consider about purchasing behaviour according to seasonality. Whats on high demand on some months or period, doesn’t mean that the category will have an increase in purchases in other months. TheLook can optimized their marketing campaign on products that will have an increase in purchase and by offering discount or deals.

**Notes :**
- There are 26 categories of products in TheLook Ecommerce
- The time frame for analysis is from Jan 2019 until Apr 2022.

### 7. Cohort Analysis
Monthly retention cohorts (the groups, or cohorts, can be defined based upon the date that a user purchased a product) and then how many of them (%) coming back for the following month in 2019-2022 ( 3 Months as M1, M2, and M3)
```sql
WITH cohort_items AS
( SELECT user_id, 
  MIN(DATE_TRUNC(DATE(created_at), month)) AS cohort_month,
  FROM `bigquery-public-data.thelook_ecommerce.orders` as o
  GROUP BY user_id
  ORDER BY cohort_month ),
user_activity AS 
( SELECT o.user_id,
  DATE_DIFF(DATE(DATE_TRUNC(created_at,month)),c.cohort_month,month)
  AS month_number
  FROM `bigquery-public-data.thelook_ecommerce.orders` as o
  LEFT JOIN cohort_items AS c
  ON o.user_id = c.user_id
  WHERE DATE(created_at) BETWEEN '2019-01-01' AND '2022-06-30'
  GROUP BY user_id, month_number
  ORDER BY month_number),
cohort_size AS 
( SELECT cohort_month, 
  COUNT(cohort_month) AS num_users
  FROM cohort_items
  GROUP BY cohort_month
  ORDER BY cohort_month),
 retention_table AS 
( SELECT cohort_month, 
  COUNT(CASE WHEN month_number =1 THEN cohort_month END) 
  as M1,
  COUNT (CASE WHEN month_number =2 THEN cohort_month END)
  as M2,
  COUNT (CASE WHEN month_number=3 THEN cohort_month END)
  as M3
  FROM cohort_items as C 
  LEFT JOIN user_activity as U
  ON U.user_id= C.user_id
  GROUP BY 1
  ORDER BY 1)

SELECT 
  r.cohort_month AS Month, 
  c.num_users as M,
  M1, M2, M3
FROM retention_table AS r
LEFT JOIN cohort_size as c 
ON r.cohort_month = c.cohort_month
WHERE r.cohort_month IS NOT NULL
ORDER BY 1
```

**Table results:**

![image](https://user-images.githubusercontent.com/100077706/185786546-769946ef-647a-4f11-a5ef-9e646c5ccbb1.png)

This query result was then exported to Google Sheets to converted into pivot table for analysis. Link to Google Sheets [here](https://docs.google.com/spreadsheets/u/0/d/12F72qbAIN-YHdByaH9e6i9YlQLe2yKXqqzqUGkeqmXY/edit)

![image](https://user-images.githubusercontent.com/100077706/185786624-1eb170a4-ca82-41ae-9a35-5c7e017e0fb5.png)

**Insights :**
- In the first month, retention value is low with an average 4.71%
- Overall during the three following month, the monthly retention generally declined reaching to 4,30% and 4.16% in the second and third month.
- The highest churn is from January 2019 where there is no user retention in the second and third month, and only 6.67% who come back on the first month.
- The highest retention is in April 2022 although it is dropped and decreasing along the month.

**Recommendation :**
- In order to reduce churn rate, TheLook Ecommerce can consider to figuring out marketing and campaign strategy to retain new user, increase user engagement and maintain the loyal user.
- The decreasing of user monthly retention can be caused by the lack of engagement of user and the website. TheLook can optimized their communication channel (email, social media etc), offering discount or deals, and improving service and quality. 

**Notes :**
- Cohorts are divided by month, initial start date can be defined using first purchase from each user.
- The scope of cohort analysis is for users who purchases first time at website between 2019-2022, and to see the monthly retention until three following months.












