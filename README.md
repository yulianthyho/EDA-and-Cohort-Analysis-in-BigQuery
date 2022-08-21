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




