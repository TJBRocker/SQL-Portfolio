# Case Study #7 - Balanced Tree Clothing Co.

````sql

SELECT *
  FROM DannySQLChallenge7..product_details

SELECT *
  FROM DannySQLChallenge7..product_hierarchy

SELECT *
  FROM DannySQLChallenge7..product_prices

SELECT *
  FROM DannySQLChallenge7..sales

````

<img width="606" alt="image" src="https://user-images.githubusercontent.com/59825363/219478121-36e10c21-b039-4cf9-ada8-216c1a7ad9fd.png">

<img width="216" alt="image" src="https://user-images.githubusercontent.com/59825363/219478192-8effe1fa-8b03-4c3c-8f0c-f7de0ed4515b.png">

<img width="163" alt="image" src="https://user-images.githubusercontent.com/59825363/219478253-c8ebc32b-a8dc-4f22-b8b5-db39ca6de205.png">

<img width="365" alt="image" src="https://user-images.githubusercontent.com/59825363/219478282-fae0975c-07f3-4568-9e24-e54c753151ee.png">


## High Level Sales Analysis

### 1. What was the total quantity sold for all products?

````sql

SELECT SUM(qty) AS total_sold
  FROM DannySQLChallenge7..sales

````

<img width="250" alt="image" src="https://user-images.githubusercontent.com/59825363/219477894-fae2f0a3-824e-493f-a7a7-e8b3b6775dfd.png">

### 2. What is the total generated revenue for all products before discounts?

````sql

SELECT SUM(qty*price) AS sales_before_discounts
  FROM DannySQLChallenge7..sales

````
<img width="250" alt="image" src="https://user-images.githubusercontent.com/59825363/219478420-b8a27ccf-63cd-4ebd-bbc2-9b57eebf011a.png">


### 3. What was the total discount amount for all products?

````sql

SELECT SUM(qty*price*discount*0.01) AS total_discounts
  FROM DannySQLChallenge7..sales

````
<img width="250" alt="image" src="https://user-images.githubusercontent.com/59825363/219478509-221e0586-71ed-4167-ba1d-342e0ed58544.png">


## Transaction Analysis

### 1. How many unique transactions were there?

````sql

SELECT COUNT (DISTINCT txn_id) AS unique_trans
  FROM DannySQLChallenge7..sales

````

<img width="250" alt="image" src="https://user-images.githubusercontent.com/59825363/219478658-d7bc0ef9-36a8-4cda-a13b-c058ac323beb.png">


### 2. What is the average unique products purchased in each transaction?

````sql

WITH CTE AS(

  SELECT txn_id,
	     COUNT(DISTINCT prod_id) AS unique_products
    FROM DannySQLChallenge7..sales
GROUP BY txn_id
)

SELECT AVG(unique_products) AS average_unique_products
  FROM CTE

````
<img width="250" alt="image" src="https://user-images.githubusercontent.com/59825363/219488998-7c7547ba-11a7-456a-92db-154652421727.png">


### 3. What are the 25th, 50th and 75th percentile values for the revenue per transaction?

````sql

WITH CTE AS(

SELECT txn_id, SUM(qty*price) AS rev
  FROM DannySQLChallenge7..sales
GROUP BY txn_id
)

SELECT TOP 1
	   PERCENTILE_CONT(0.25) WITHIN GROUP(ORDER BY rev) OVER() AS '25th percentile',
	   PERCENTILE_CONT(0.50) WITHIN GROUP(ORDER BY rev) OVER() AS '50th percentile',
	   PERCENTILE_CONT(0.75) WITHIN GROUP(ORDER BY rev) OVER() AS '75th percentile'
  FROM CTE

````

<img width="235" alt="image" src="https://user-images.githubusercontent.com/59825363/219489378-b1647add-00f9-4014-8d53-86713568a908.png">


### 4. What is the average discount value per transaction?

````sql



````

### 5. What is the percentage split of all transactions for members vs non-members?

````sql



````

### 6. What is the average revenue for member transactions and non-member transactions?

````sql



````

## Product Analysis

### 1. What are the top 3 products by total revenue before discount?

````sql



````


### 2. What is the total quantity, revenue and discount for each segment?

````sql



````


### 3. What is the top selling product for each segment?

````sql



````


### 4. What is the total quantity, revenue and discount for each category?

````sql



````


### 5. What is the top selling product for each category?

````sql



````


### 6. What is the percentage split of revenue by product for each segment?

````sql



````


### 7. What is the percentage split of revenue by segment for each category?

````sql



````


### 8. What is the percentage split of total revenue by category?

````sql



````


### 9. What is the total transaction “penetration” for each product? (hint: penetration = number of transactions where at least 1 quantity of a product was purchased divided by total number of transactions)

````sql



````


### 10. What is the most common combination of at least 1 quantity of any 3 products in a 1 single transaction?

````sql



````


## Reporting Challenge

### Write a single SQL script that combines all of the previous questions into a scheduled report that the Balanced Tree team can run at the beginning of each month to calculate the previous month’s values.

### Imagine that the Chief Financial Officer (which is also Danny) has asked for all of these questions at the end of every month.

### He first wants you to generate the data for January only - but then he also wants you to demonstrate that you can easily run the samne analysis for February without many changes (if at all).

### Feel free to split up your final outputs into as many tables as you need - but be sure to explicitly reference which table outputs relate to which question for full marks :)

## Bonus

### Use a single SQL query to transform the 'product_hierarchy' and 'product_prices' datasets to the product_details table.

### Hint: you may want to consider using a recursive CTE to solve this problem!

