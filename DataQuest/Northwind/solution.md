# DataQuest Northwind Traders

## Scenario

You are a Data Analyst at Northwind Traders, an international gourmet food distributor. Management is looking to you for insights to make strategic decisions in several aspects of the business. The projects focus on:

-  Evaluating employee performance to boost productivity,
-  Understanding product sales and category performance to optimize inventory and marketing strategies,
-  Analyzing sales growth to identify trends, monitor company progress, and make more accurate forecasts,
-  And evaluating customer purchase behavior to target high-value customers with promotional incentives.

Using the SQL window functions on the [Northwind database](https://github.com/pthom/northwind_psql/tree/master), you will provide these essential insights to management, contributing significantly to the company's strategic decisions.

## Database Schema

Throughout this guided project, you’ll frequently need to reference the schema diagram while crafting your queries. The database schema provides an overview of the Northwind database's tables, columns, relationships, and constraints, making it an essential resource for constructing accurate and efficient SQL queries. The Northwind database has over a dozen tables, most of which we won't need for this project. We have included a modified diagram with the necessary tables below, but if you would like to refer to the original schema, you can do so [here](https://github.com/pthom/northwind_psql/blob/master/ER.png).

![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/2f036c29-8313-416e-a4d0-c3dd32a70ca1)

### Questions & Solutions

Familiarize yourself with the Northwind company by writing the following queries. Consider saving them as views to help with the rest of the project:

-  Combine orders and customers tables to get more detailed information about each order.
-  Combine order_details, products, and orders tables to get detailed order information, including the product name and quantity.
-  Combine employees and orders tables to see who is responsible for each order.

#### Part 1: Monthly Sales

-  Join the Orders and Order_Details tables to bring together the data you need.
-  Group by the month of the Order_Date and calculate the total sales for each month. Use the DATE_TRUNC function to truncate -the Order_Date to the nearest month.
-  Use the SUM function with an OVER clause to calculate the running total of sales by month.

````sql

WITH CTE AS(
SELECT CAST (CONCAT('01-',FORMAT(order_date, 'MM-yyyy')) AS DATE) AS month_extract, 
	   SUM((1-discount)*unit_price*quantity) AS sales
  FROM DQ..order_details AS ord_det
  JOIN DQ..orders AS ord ON ord_det.order_id = ord.order_id
GROUP BY CAST (CONCAT('01-',FORMAT(order_date, 'MM-yyyy')) AS DATE)
)
SELECT *, SUM(sales) OVER(ORDER BY month_extract) AS running_total
  FROM CTE

````

![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/860557cc-2fff-44e2-8bd4-64f96c931b23)


#### Part 2: Month-Over-Month Sales

-  Create a CTE that calculates the total sales for each month.
-  Create a second CTE that uses the LAG function with an OVER clause to get the total sales of the previous month.
-  In your main query, calculate the month-over-month sales growth rate.

````sql

WITH CTE AS(
SELECT CAST (CONCAT('01-',FORMAT(order_date, 'MM-yyyy')) AS DATE) AS month_extract, 
	   SUM((1-discount)*unit_price*quantity) AS sales
  FROM DQ..order_details AS ord_det
  JOIN DQ..orders AS ord ON ord_det.order_id = ord.order_id
GROUP BY CAST (CONCAT('01-',FORMAT(order_date, 'MM-yyyy')) AS DATE)
), CTE2 AS(
SELECT *, SUM(sales) OVER(ORDER BY month_extract) AS running_total, LAG(sales, 1) OVER( ORDER BY month_extract) AS prev_month_sales
  FROM CTE
)
SELECT month_extract AS month, ROUND(100.0 * (sales-prev_month_sales) / prev_month_sales,2) AS growth
  FROM CTE2

````

![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/85a4fddb-722f-4b60-b161-00b46b77a475)

#### Part 3: Identifying High-Value Customers

-  Create a CTE that includes customer identification and calculates the value of each of their orders.
-  In the main query use the CTE to categorize each order as 'Above Average' or 'Average/Below Average' using a CASE statement.
-  As an extension, try counting how many orders are 'Above Average' for each customer.

````sql

WITH CTE AS (
SELECT ord.customer_id,ord_det.order_id, ROUND(SUM((1-discount)*unit_price*quantity),0) AS order_value
  FROM DQ..order_details AS ord_det
  JOIN DQ..orders AS ord ON ord_det.order_id = ord.order_id
  JOIN DQ..customers AS cust ON ord.customer_id = cust.customer_id
GROUP BY ord.customer_id, ord_det.order_id
)
SELECT *, 
	   CASE WHEN order_value > AVG(order_value) OVER() THEN 'above average'
			ELSE 'below average'
			END AS classification
  FROM CTE

````

![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/88e2d4a5-75c0-4d2f-9a4c-84e9e19b1072)


````sql

WITH CTE AS (
SELECT ord.customer_id,ord_det.order_id, ROUND(SUM((1-discount)*unit_price*quantity),0) AS order_value
  FROM DQ..order_details AS ord_det
  JOIN DQ..orders AS ord ON ord_det.order_id = ord.order_id
  JOIN DQ..customers AS cust ON ord.customer_id = cust.customer_id
GROUP BY ord.customer_id, ord_det.order_id
), CTE2 AS(
SELECT *,
	   CASE WHEN order_value > AVG(order_value) OVER() THEN 'above average'
			ELSE 'below average'
			END AS classification
  FROM CTE
) 
SELECT customer_id, COUNT(*) AS above_avg_count
  FROM CTE2
WHERE classification = 'above average'
GROUP BY customer_id
ORDER BY above_avg_count DESC

````
![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/aa1c11b5-f673-40a5-bf4f-650db6561ca9)

#### Part 4: Percentage of Sales for Each Category

Find the percentage of total sales for each product category.

-  Create a CTE that calculates the total sales for each product category.
-  Use your CTE in the main query to calculate the percentage of total sales for each product category.

````sql

WITH CTE AS(
SELECT cat.category_id, category_name, SUM(ROUND((1-discount)*ord_det.unit_price*quantity,0)) AS sales
  FROM DQ..order_details AS ord_det
  JOIN DQ..products AS prod ON ord_det.product_id = prod.product_id
  JOIN DQ..categories AS cat ON cat.category_id = prod.category_id
GROUP BY cat.category_id, category_name
)
SELECT *, ROUND(100.0 *sales/ SUM(sales) OVER(),2) AS sales_percentage
  FROM CTE
ORDER BY sales_percentage DESC

````
![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/f09d5042-bb2d-43d7-9c6e-afae011fd14e)

#### Part 5: Top Products Per Category

-  Create a CTE that calculates the total sales for each product.
-  Use the ROW_NUMBER function with an OVER clause in the main query to assign a row number to each product within each category based on the total sales.
-  Use a WHERE clause in the main query to filter out the products that have a row number greater than 3.

````sql

WITH CTE AS(
SELECT cat.category_id, category_name, prod.product_id, product_name, SUM(ROUND((1-discount)*ord_det.unit_price*quantity,0)) AS sales
  FROM DQ..order_details AS ord_det
  JOIN DQ..products AS prod ON ord_det.product_id = prod.product_id
  JOIN DQ..categories AS cat ON cat.category_id = prod.category_id
GROUP BY cat.category_id, category_name, prod.product_id, product_name
), CTE2 AS(
SELECT *, ROW_NUMBER() OVER(PARTITION BY category_id ORDER BY SALES DESC) AS sales_rank
  FROM CTE
)
SELECT *
  FROM CTE2
 WHERE sales_rank <= 3 

````

![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/b7e8c52b-b5b2-471b-ae45-31af9d96f1cc)

#### Bonus

-  Identify the top 20% of customers by total purchase volume.
````sql

WITH CTE AS(
SELECT ord.customer_id, SUM(quantity) AS item_volume
  FROM DQ..order_details AS ord_det
  JOIN DQ..orders AS ord ON ord_det.order_id = ord.order_id
  JOIN DQ..customers AS cust ON ord.customer_id = cust.customer_id
GROUP BY ord.customer_id

), CTE2 AS(
SELECT *, PERCENT_RANK() OVER(ORDER BY item_volume DESC) AS percentile
  FROM CTE
)

SELECT customer_id, item_volume
  FROM CTE2
 WHERE percentile <=0.2

````
![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/d8925201-9a58-42a9-9745-40dd847c9359)

-  

````sql



````
-  
````sql



````

-  

````sql



````

````sql



````








