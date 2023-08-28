# Case Study #7 - Balanced Tree Clothing Co.:

![image](https://user-images.githubusercontent.com/59825363/196279707-16808857-f6e5-4931-a7f8-891424d80ccc.png)

## Task

Balanced Tree Clothing Company prides themselves on providing an optimised range of clothing and lifestyle wear for the modern adventurer!

Danny, the CEO of this trendy fashion company has asked you to assist the team’s merchandising teams analyse their sales performance and generate a basic financial report to share with the wider business.

## Entity Relationship Diagram

![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/21cd548e-ba13-4f66-ba5c-6c3a03c7558b)

## Case Study Questions

### High Level Sales Analysis

1.  What was the total quantity sold for all products?
2.  What is the total generated revenue for all products before discounts?
3.  What was the total discount amount for all products?

### Transaction Analysis

1.  How many unique transactions were there?
2.  What is the average unique products purchased in each transaction?
3.  What are the 25th, 50th and 75th percentile values for the revenue per transaction?
4.  What is the average discount value per transaction?
5.  What is the percentage split of all transactions for members vs non-members?
6.  What is the average revenue for member transactions and non-member transactions?

### Product Analysis

1.  What are the top 3 products by total revenue before discount?
2.  What is the total quantity, revenue and discount for each segment?
3.  What is the top selling product for each segment?
4.  What is the total quantity, revenue and discount for each category?
5.  What is the top selling product for each category?
6.  What is the percentage split of revenue by product for each segment?
7.  What is the percentage split of revenue by segment for each category?
8.  What is the percentage split of total revenue by category?
9.  What is the total transaction “penetration” for each product? (hint: penetration = number of transactions where at least 1 quantity of a product was purchased divided by total number of transactions)
10.  What is the most common combination of at least 1 quantity of any 3 products in a 1 single transaction?

### Reporting Challenge

Write a single SQL script that combines all of the previous questions into a scheduled report that the Balanced Tree team can run at the beginning of each month to calculate the previous month’s values.

Imagine that the Chief Financial Officer (which is also Danny) has asked for all of these questions at the end of every month.

He first wants you to generate the data for January only - but then he also wants you to demonstrate that you can easily run the samne analysis for February without many changes (if at all).

Feel free to split up your final outputs into as many tables as you need - but be sure to explicitly reference which table outputs relate to which question for full marks :)
