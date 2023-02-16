# Case Study #7 - Balanced Tree Clothing Co.

## High Level Sales Analysis

### 1. What was the total quantity sold for all products?

````sql

SELECT SUM(qty) AS total_sold
  FROM DannySQLChallenge7..sales

````
<img width="300" alt="image" src="https://user-images.githubusercontent.com/59825363/219477894-fae2f0a3-824e-493f-a7a7-e8b3b6775dfd.png">



### 2. What is the total generated revenue for all products before discounts?

````sql



````


### 3. What was the total discount amount for all products?

````sql



````

## Transaction Analysis

### 1. How many unique transactions were there?

````sql



````

### 2. What is the average unique products purchased in each transaction?

````sql



````

### 3. What are the 25th, 50th and 75th percentile values for the revenue per transaction?

````sql



````

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

