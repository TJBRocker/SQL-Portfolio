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



````

````sql



````

````sql



````

````sql



````

````sql



````

````sql



````

