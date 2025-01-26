# Customer Segmentation data analysis for Ecommerce Sales using SQL
![logo](https://github.com/amandamu0780/SQL_RFM_Ecommerce/blob/main/concepto-de-compras-en-linea.jpg)
## Objectives:
This project aims to demonstrate my intermediate SQL skills by identifying key customers and their buying behavior for an e-commerce company. It provides insights into geographic data, the most profitable times of the year, and product performance. The purpose of this work is to inform potential areas of focus on specific customers, products, and locations. The following provides detailed objectives, business problems, and its solutions and findings.
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
