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

<img width="400" alt="image" src="https://user-images.githubusercontent.com/59825363/219489378-b1647add-00f9-4014-8d53-86713568a908.png">


### 4. What is the average discount value per transaction?

````sql

WITH CTE AS(

SELECT txn_id, SUM(qty*price*discount*0.01) AS total_discount
  FROM DannySQLChallenge7..sales
GROUP BY txn_id
)

SELECT ROUND(AVG(total_discount),2) AS average_discount
  FROM CTE

````

<img width="250" alt="image" src="https://user-images.githubusercontent.com/59825363/219489527-dffbb8d0-70e3-4b74-8c69-a61f5dfd1680.png">


### 5. What is the percentage split of all transactions for members vs non-members?

````sql

WITH CTE AS(

SELECT DISTINCT txn_id, 
	   CASE WHEN TRIM(member) = 't' THEN 1 ELSE 0 END AS member_true,
	   CASE WHEN TRIM(member) = 'f' THEN 1 ELSE 0 END AS member_false
  FROM DannySQLChallenge7..sales
)

SELECT CAST(ROUND(100.0*SUM(member_true)/COUNT(*),2) AS float) AS percent_members,
	   CAST(ROUND(100.0*SUM(member_false)/COUNT(*),2)AS float) AS percent_non_members
  FROM CTE

````

<img width="400" alt="image" src="https://user-images.githubusercontent.com/59825363/219489601-6bc61809-44b7-4bee-b37f-9ca0ea8a81ab.png">


### 6. What is the average revenue for member transactions and non-member transactions?

````sql

WITH CTE AS(

SELECT txn_id,
	   SUM(CASE WHEN TRIM(member) = 't' THEN price*qty END) AS members_rev,
	   SUM(CASE WHEN TRIM(member) = 't' THEN ((1-discount*0.01)*price*qty) END) AS members_inc,
	   SUM(CASE WHEN TRIM(member) = 'f' THEN price*qty END) AS non_members_rev,
	   SUM(CASE WHEN TRIM(member) = 'f' THEN ((1-discount*0.01)*price*qty) END) AS non_members_inc
  FROM DannySQLChallenge7..sales
GROUP BY txn_id
)

SELECT ROUND(AVG(members_rev),2) AS avg_members_rev,
	   ROUND(AVG(members_inc),2) AS avg_members_inc,
	   ROUND(AVG(non_members_rev),2) AS avg_non_members_rev,
	   ROUND(AVG(non_members_inc),2) AS avg_non_members_inc
  FROM CTE

````

<img width="500" alt="image" src="https://user-images.githubusercontent.com/59825363/219489671-b6a11d6c-82be-4d4a-ae44-48c51516dba4.png">


## Product Analysis

### 1. What are the top 3 products by total revenue before discount?

````sql

SELECT TOP 3
	   s.prod_id,
	   pd.product_name,
	   SUM(s.qty*s.price) AS rev_bf_discount
  FROM DannySQLChallenge7..sales AS s
  JOIN DannySQLChallenge7..product_details AS pd ON pd.product_id = s.prod_id
GROUP BY s.prod_id, pd.product_name
ORDER BY rev_bf_discount DESC

````

<img width="400" alt="image" src="https://user-images.githubusercontent.com/59825363/219489769-82bdaadf-98bb-4d19-8bcb-96bae06d735c.png">

### 2. What is the total quantity, revenue and discount for each segment?

````sql

SELECT pd.segment_id,
	   pd.segment_name,
	   SUM(s.qty) AS quantity,
	   SUM(s.price*s.qty) AS revenue,
	   ROUND(SUM(s.price*s.qty*s.discount*0.01),2) AS total_discount
  FROM DannySQLChallenge7..sales AS s
  JOIN DannySQLChallenge7..product_details AS pd ON pd.product_id = s.prod_id
GROUP BY pd.segment_id, pd.segment_name

````

<img width="303" alt="image" src="https://user-images.githubusercontent.com/59825363/219489871-998f01bb-69f0-42ed-b90b-258b74cfdec8.png">

### 3. What is the top selling product for each segment?

````sql

WITH CTE AS (
SELECT pd.segment_id,
	   pd.segment_name,
	   pd.product_id,
	   pd.product_name,
	   SUM(s.qty) AS quantity,
	   SUM(s.price*s.qty) AS revenue,
	   ROUND(SUM(s.price*s.qty*s.discount*0.01),2) AS total_discount,
	   ROW_NUMBER() OVER(PARTITION BY pd.segment_name ORDER BY SUM(s.qty) DESC) AS quantity_rank,
	   ROW_NUMBER() OVER(PARTITION BY pd.segment_name ORDER BY SUM(s.qty*s.price) DESC) AS revenue_rank
  FROM DannySQLChallenge7..sales AS s
  JOIN DannySQLChallenge7..product_details AS pd ON pd.product_id = s.prod_id
GROUP BY pd.segment_id, pd.segment_name, pd.product_id, pd.product_name
)

SELECT segment_name,
	   product_name,
	   quantity
  FROM CTE
 WHERE quantity_rank = 1

````

<img width="400" alt="image" src="https://user-images.githubusercontent.com/59825363/219489962-a0807541-32ca-4d92-95c6-7844584b3cd1.png">

````sql

 WITH CTE AS (
SELECT pd.segment_id,
	   pd.segment_name,
	   pd.product_id,
	   pd.product_name,
	   SUM(s.qty) AS quantity,
	   SUM(s.price*s.qty) AS revenue,
	   ROUND(SUM(s.price*s.qty*s.discount*0.01),2) AS total_discount,
	   ROW_NUMBER() OVER(PARTITION BY pd.segment_name ORDER BY SUM(s.qty) DESC) AS quantity_rank,
	   ROW_NUMBER() OVER(PARTITION BY pd.segment_name ORDER BY SUM(s.qty*s.price) DESC) AS revenue_rank
  FROM DannySQLChallenge7..sales AS s
  JOIN DannySQLChallenge7..product_details AS pd ON pd.product_id = s.prod_id
GROUP BY pd.segment_id, pd.segment_name, pd.product_id, pd.product_name
)

SELECT segment_name,
	   product_name,
	   revenue
  FROM CTE
 WHERE revenue_rank = 1

````

<img width="400" alt="image" src="https://user-images.githubusercontent.com/59825363/219490028-bcfff046-8722-4829-97db-606649143cff.png">

### 4. What is the total quantity, revenue and discount for each category?

````sql

SELECT pd.category_id,
	   pd.category_name,
	   SUM(s.qty) AS quantity,
	   SUM(s.price*s.qty) AS revenue,
	   ROUND(SUM(s.price*s.qty*s.discount*0.01),2) AS total_discount
  FROM DannySQLChallenge7..sales AS s
  JOIN DannySQLChallenge7..product_details AS pd ON pd.product_id = s.prod_id
GROUP BY pd.category_id, pd.category_name
ORDER BY pd.category_id

````

<img width="400" alt="image" src="https://user-images.githubusercontent.com/59825363/219490089-118bcb64-d4b7-4b39-a658-e2c1cb93e9be.png">

### 5. What is the top selling product for each category?

````sql

WITH CTE AS(

SELECT pd.segment_id,
	   pd.segment_name,
	   pd.product_id,
	   pd.product_name,
	   pd.category_id,
	   pd.category_name,
	   SUM(s.qty) AS quantity,
	   SUM(s.price*s.qty) AS revenue,
	   ROUND(SUM(s.price*s.qty*s.discount*0.01),2) AS total_discount,
	   ROW_NUMBER() OVER(PARTITION BY pd.category_name ORDER BY SUM(s.qty) DESC) AS quantity_rank,
	   ROW_NUMBER() OVER(PARTITION BY pd.category_name ORDER BY SUM(s.qty*s.price) DESC) AS revenue_rank
  FROM DannySQLChallenge7..sales AS s
  JOIN DannySQLChallenge7..product_details AS pd ON pd.product_id = s.prod_id
GROUP BY pd.segment_id, pd.segment_name, pd.product_id, pd.product_name, pd.category_id, pd.category_name
)

SELECT category_name,
       product_name,
	   quantity,
	   revenue
  FROM CTE
 WHERE quantity_rank = 1

````

<img width="400" alt="image" src="https://user-images.githubusercontent.com/59825363/219490203-71d2a18e-c7df-47d0-97fb-3eb37d76692d.png">

### 6. What is the percentage split of revenue by product for each segment?

````sql

SELECT pd.segment_id,
	   pd.segment_name,
	   pd.product_id,
	   pd.product_name,
	   pd.category_id,
	   pd.category_name,
	   SUM(s.qty) AS quantity,
	   SUM(s.price*s.qty) AS revenue,
	   ROUND(SUM(s.price*s.qty*s.discount*0.01),2) AS total_discount,
	   ROW_NUMBER() OVER(PARTITION BY pd.category_name ORDER BY SUM(s.qty) DESC) AS quantity_rank,
	   ROW_NUMBER() OVER(PARTITION BY pd.category_name ORDER BY SUM(s.qty*s.price) DESC) AS revenue_rank
  FROM DannySQLChallenge7..sales AS s
  JOIN DannySQLChallenge7..product_details AS pd ON pd.product_id = s.prod_id
GROUP BY pd.segment_id, pd.segment_name, pd.product_id, pd.product_name, pd.category_id, pd.category_name

````

<img width="727" alt="image" src="https://user-images.githubusercontent.com/59825363/219490273-85c9c0f9-ad0b-4d3d-acf2-240f1a67c587.png">

### 7. What is the percentage split of revenue by segment for each category?

````sql

SELECT pd.segment_name,  
	   pd.category_name,
	   ROUND(100.0*SUM(qty*s.price)/ SUM(SUM(qty*s.price)) OVER(PARTITION BY category_name),2) AS percent_of_rev
  FROM DannySQLChallenge7..sales AS s
  JOIN DannySQLChallenge7..product_details AS pd ON pd.product_id = s.prod_id
GROUP BY pd.segment_name,  pd.category_name

````

<img width="400" alt="image" src="https://user-images.githubusercontent.com/59825363/219490329-f0080b2c-e634-4cc3-92fd-44d11eebaf44.png">

### 8. What is the percentage split of total revenue by category?

````sql

SELECT  
	   pd.category_name,
	   SUM(s.price*s.qty) AS revenue,
	   ROUND(100.0*SUM(s.price*s.qty) / (SELECT SUM(price* qty) FROM DannySQLChallenge7..sales),2) AS percent_of_rev  
  FROM DannySQLChallenge7..sales AS s
  JOIN DannySQLChallenge7..product_details AS pd ON pd.product_id = s.prod_id
GROUP BY  pd.category_name

````

<img width="400" alt="image" src="https://user-images.githubusercontent.com/59825363/219490385-a9d61325-1449-4c2b-b663-1242e9a3c30a.png">

### 9. What is the total transaction “penetration” for each product? (hint: penetration = number of transactions where at least 1 quantity of a product was purchased divided by total number of transactions)

````sql

SELECT 
	   pd.product_name,
	   CAST(ROUND(100.0*COUNT(prod_id) / (SELECT COUNT(DISTINCT txn_id) FROM DannySQLChallenge7..sales),2) AS float) AS penetration
  FROM DannySQLChallenge7..sales AS s
  JOIN DannySQLChallenge7..product_details AS pd ON pd.product_id = s.prod_id
GROUP BY  pd.product_name
ORDER BY 2 DESC

````

<img width="400" alt="image" src="https://user-images.githubusercontent.com/59825363/219490445-34b03b37-a03a-4dab-944f-991f7c1e8efd.png">

### 10. What is the most common combination of at least 1 quantity of any 3 products in a 1 single transaction?

````sql



````


## Reporting Challenge

Write a single SQL script that combines all of the previous questions into a scheduled report that the Balanced Tree team can run at the beginning of each month to calculate the previous month’s values.

Imagine that the Chief Financial Officer (which is also Danny) has asked for all of these questions at the end of every month.

He first wants you to generate the data for January only - but then he also wants you to demonstrate that you can easily run the samne analysis for February without many changes (if at all).

Feel free to split up your final outputs into as many tables as you need - but be sure to explicitly reference which table outputs relate to which question for full marks :)

## Bonus

Use a single SQL query to transform the "product_hierarchy" and "product_prices" datasets to the product_details table.

Hint: you may want to consider using a recursive CTE to solve this problem!

