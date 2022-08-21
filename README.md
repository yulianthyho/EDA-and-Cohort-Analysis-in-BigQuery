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






