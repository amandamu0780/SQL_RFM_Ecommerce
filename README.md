# Customer Segmentation data analysis for Ecommerce Sales using SQL
![logo](https://github.com/amandamu0780/SQL_RFM_Ecommerce/blob/main/concepto-de-compras-en-linea.jpg)
## Objectives:
This project aims to demonstrate my intermediate SQL skills by identifying key customers and their buying behavior for an e-commerce company utilizing RFM analysis. It provides insights into geographic data, the most profitable times of the year, and product performance. The purpose of this work is to inform potential areas of focus on specific customers, products, and locations. The following provides detailed objectives, business problems, and its solutions and findings.
## Dataset:
The data of this project was sourced from Kaggle:[Ecommerce_Data](https://www.kaggle.com/datasets/benroshan/ecommerce-data)
## Importing the data
```sql
CREATE TABLE list_orders (
	    Order_id VARCHAR(25),
	    Order_date DATE,
	    Customer_name VARCHAR(25),
	    State VARCHAR(25),
	    City VARCHAR(25)
);

CREATE TABLE order_details (
		Order_id VARCHAR(25),
		Amount DECIMAL (10,2),
		Profit DECIMAL(10,2),
		Quantity INT,
		Category VARCHAR (25),
		Sub_category VARCHAR (25)
);

CREATE TABLE sales_target (
		Month_order VARCHAR(25),
		Category VARCHAR (25),
		Target DECIMAL (10,2)
 ```

## Business Problems and Solutions
1. Find most profitable Seasons in India
```sql
WITH months AS 
(
SELECT 
EXTRACT (month FROM lo.order_date) AS month,
od.profit
FROM order_details od
JOIN list_orders lo
ON od.order_id = lo.order_id
GROUP BY 1,2
)
SELECT 
SUM(profit) AS total_profit,
CASE 
WHEN month IN (2,3) THEN 'Spring'
WHEN month IN (4,5) THEN 'Summer'
WHEN month IN (6,7,8,9) THEN 'Monsoon'
WHEN month IN (10,11) THEN 'Fall'
WHEN month IN (12,1) THEN 'Winter'
END AS Season
FROM months
GROUP BY 2
ORDER BY 1 DESC;
```
2. Find 10 most profitable products
```sql
SELECT 
od.category,
od.sub_category,
SUM(od.profit)
FROM order_details od
JOIN list_orders lo
ON od.order_id = lo.order_id
GROUP BY 1,2
ORDER BY 3 DESC
LIMIT 10;
```
## Customer Segmentation
3. How can we segment our customer based in a RFM scores to identify different costumer groups

Calculating RFM scores
```sql
SELECT
        lo.customer_name,
        od.order_id AS order_id,
        AGE(CURRENT_DATE, MAX(lo.order_date)) AS Recency,
        COUNT(od.order_id) AS Frequency,
        SUM(od.amount) AS Monetary,
        NTILE(5) OVER (ORDER BY AGE(CURRENT_DATE, MAX(lo.order_date)) DESC) AS R,
        NTILE(5) OVER (ORDER BY COUNT(od.order_id) ASC) AS F,
        NTILE(5) OVER (ORDER BY SUM(od.amount) ASC) AS M
    FROM 
        order_details od
    INNER JOIN 
        list_orders lo ON od.order_id = lo.order_id
    GROUP BY 
        lo.customer_name, od.order_id
```
Combining RFM scores
```sql
WITH Rfm AS (
    SELECT
        lo.customer_name,
        od.order_id AS order_id,
        AGE(CURRENT_DATE, MAX(lo.order_date)) AS Recency,
        COUNT(od.order_id) AS Frequency,
        SUM(od.amount) AS Monetary,
        NTILE(5) OVER (ORDER BY AGE(CURRENT_DATE, MAX(lo.order_date)) DESC) AS R,
        NTILE(5) OVER (ORDER BY COUNT(od.order_id) ASC) AS F,
        NTILE(5) OVER (ORDER BY SUM(od.amount) ASC) AS M
    FROM 
        order_details od
    INNER JOIN 
        list_orders lo ON od.order_id = lo.order_id
    GROUP BY 
        lo.customer_name, od.order_id
)
SELECT 
		order_id,
        customer_name,
        CONCAT(R, F, M) AS Rfm_score
    FROM 
        Rfm 
    ORDER BY 
        Rfm_score DESC
```
Categorizing Customers based in their RFM scores
```sql
WITH Rfm AS (
    SELECT
        lo.customer_name,
        od.order_id AS order_id,
        AGE(CURRENT_DATE, MAX(lo.order_date)) AS Recency,
        COUNT(od.order_id) AS Frequency,
        SUM(od.amount) AS Monetary,
        NTILE(5) OVER (ORDER BY AGE(CURRENT_DATE, MAX(lo.order_date)) DESC) AS R,
        NTILE(5) OVER (ORDER BY COUNT(od.order_id) ASC) AS F,
        NTILE(5) OVER (ORDER BY SUM(od.amount) ASC) AS M
    FROM 
        order_details od
    INNER JOIN 
        list_orders lo ON od.order_id = lo.order_id
    GROUP BY 
        lo.customer_name, od.order_id
),
Rfm_scores AS (
    SELECT 
		order_id,
        customer_name,
        CONCAT(R, F, M) AS Rfm_score
    FROM 
        Rfm 
    ORDER BY 
        Rfm_score DESC
)
SELECT
	COUNT(order_id) AS num_customers,
    CASE 
        WHEN Rfm_score IN ('444', '445', '454', '455', '544', '545', '554', '555') THEN 'VIP Customers'
        WHEN Rfm_score IN ('114', '115', '124', '125', '214', '215', '224', '225') THEN 'At Risk'
        WHEN Rfm_score IN ('411', '412', '421', '422', '511', '512', '521', '522') THEN 'New Customers'
        WHEN Rfm_score IN ('443', '453', '543', '553') THEN 'Loyal Customers'
        WHEN Rfm_score IN ('111', '112', '121', '122', '211', '212', '221', '222') THEN 'Lost'
        ELSE 'Other customer'
    END AS Segment
FROM 
    Rfm_scores
GROUP BY 2
ORDER BY 1 DESC;
```
Anlyzing which category is the most popular for each of the customers segments
```sql
WITH Rfm AS (
    SELECT
		od.category,
        lo.customer_name,
        od.order_id AS order_id,
        AGE(CURRENT_DATE, MAX(lo.order_date)) AS Recency,
        COUNT(od.order_id) AS Frequency,
        SUM(od.amount) AS Monetary,
        NTILE(5) OVER (ORDER BY AGE(CURRENT_DATE, MAX(lo.order_date)) DESC) AS R,
        NTILE(5) OVER (ORDER BY COUNT(od.order_id) ASC) AS F,
        NTILE(5) OVER (ORDER BY SUM(od.amount) ASC) AS M
    FROM 
        order_details od
    INNER JOIN 
        list_orders lo ON od.order_id = lo.order_id
    GROUP BY 
        1,2,3
),
Rfm_scores AS (
    SELECT 
		category,
		order_id,
        customer_name,
        CONCAT(R, F, M) AS Rfm_score
    FROM 
        Rfm 
    ORDER BY 
        Rfm_score DESC
)
SELECT
	category,
	COUNT(DISTINCT order_id) AS num_purchases,
    CASE 
        WHEN Rfm_score IN ('444', '445', '454', '455', '544', '545', '554', '555') THEN 'VIP Customers'
        WHEN Rfm_score IN ('114', '115', '124', '125', '214', '215', '224', '225') THEN 'At Risk'
        WHEN Rfm_score IN ('411', '412', '421', '422', '511', '512', '521', '522') THEN 'New Customers'
        WHEN Rfm_score IN ('443', '453', '543', '553') THEN 'Loyal Customers'
        WHEN Rfm_score IN ('111', '112', '121', '122', '211', '212', '221', '222') THEN 'Lost'
        ELSE 'Other customer'
    END AS Segment
FROM 
    Rfm_scores
GROUP BY 1,3
ORDER BY 3 DESC;
```
Most profitable Segments
```sql
WITH Rfm AS (
    SELECT
		od.profit,
		od.category,
        lo.customer_name,
        od.order_id AS order_id,
        AGE(CURRENT_DATE, MAX(lo.order_date)) AS Recency,
        COUNT(od.order_id) AS Frequency,
        SUM(od.amount) AS Monetary,
        NTILE(5) OVER (ORDER BY AGE(CURRENT_DATE, MAX(lo.order_date)) DESC) AS R,
        NTILE(5) OVER (ORDER BY COUNT(od.order_id) ASC) AS F,
        NTILE(5) OVER (ORDER BY SUM(od.amount) ASC) AS M
    FROM 
        order_details od
    INNER JOIN 
        list_orders lo ON od.order_id = lo.order_id
    GROUP BY 
        1,2,3,4
),
Rfm_scores AS (
    SELECT
		profit,
		category,
		order_id,
        customer_name,
        CONCAT(R, F, M) AS Rfm_score
    FROM 
        Rfm 
    ORDER BY 
        Rfm_score DESC
)
SELECT
	SUM(profit) AS profit,
    CASE 
        WHEN Rfm_score IN ('444', '445', '454', '455', '544', '545', '554', '555') THEN 'VIP Customers'
        WHEN Rfm_score IN ('114', '115', '124', '125', '214', '215', '224', '225') THEN 'At Risk'
        WHEN Rfm_score IN ('411', '412', '421', '422', '511', '512', '521', '522') THEN 'New Customers'
        WHEN Rfm_score IN ('443', '453', '543', '553') THEN 'Loyal Customers'
        WHEN Rfm_score IN ('111', '112', '121', '122', '211', '212', '221', '222') THEN 'Lost'
        ELSE 'Regular Customer'
    END AS Segment
FROM 
    Rfm_scores
GROUP BY 2
ORDER BY 1 DESC;
```
