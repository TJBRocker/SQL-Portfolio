# Case Study #4 - Data Bank


## A. Customer Nodes Exploration

### How many unique nodes are there on the Data Bank system?

````sql
  SELECT DISTINCT node_id AS unique_nodes
    FROM DannySQLChallenge4..customer_nodes
ORDER BY 1
````
<img width="200" alt="image" src="https://user-images.githubusercontent.com/59825363/199301530-82d251ab-7364-4b30-ad61-25f91bdecf87.png">


### What is the number of nodes per region?

````sql
  SELECT cn.region_id,
         region_name,
         COUNT(node_id) AS node_count
    FROM DannySQLChallenge4..customer_nodes AS cn
    JOIN DannySQLChallenge4..regions AS r ON  r.region_id = cn.region_id
GROUP BY cn.region_id, region_name
ORDER BY cn. region_id, region_name
````
<img width="300" alt="image" src="https://user-images.githubusercontent.com/59825363/199302124-49dfe1f8-e865-4b11-bb14-ebb8da4186f8.png">


### How many customers are allocated to each region?

````sql
  SELECT cn.region_id,
         region_name,
         COUNT(DISTINCT customer_id) AS customer_count
    FROM DannySQLChallenge4..customer_nodes AS cn
    JOIN DannySQLChallenge4..regions AS r ON  r.region_id = cn.region_id
GROUP BY cn.region_id, region_name
ORDER BY cn. region_id, region_name
````
<img width="300" alt="image" src="https://user-images.githubusercontent.com/59825363/199303734-744ea1be-c73b-48ad-8cd1-3cb05bcc4796.png">


### How many days on average are customers reallocated to a different node?

````sql
SELECT AVG(DATEDIFF(DAY,start_date,end_date)) AS avg_reallocation_time
  FROM DannySQLChallenge4..customer_nodes
````
<img width="200" alt="image" src="https://user-images.githubusercontent.com/59825363/199304178-9f19b993-962e-4b70-a7a1-f5eccd34afd1.png">

Initially the above produced a large reallocation time, under further investigation the largest end date was in the year 9999-12-31! This must be for customer/nodes that haven't been rellocated so I'll exclude these.

````sql
SELECT ROUND(AVG(DATEDIFF(DAY,start_date,end_date)),2) AS avg_reallocation_time
  FROM DannySQLChallenge4..customer_nodes
 WHERE end_date != '9999-12-31'
````
<img width="200" alt="image" src="https://user-images.githubusercontent.com/59825363/199304372-66f1586a-bbfe-4123-a594-af5175f8929e.png">

After looking into this further, it turns out that some customer stay within the same node, example below

````sql
  SELECT *, 
         DATEDIFF(DAY,start_date,end_date) AS reallocation_time
    FROM DannySQLChallenge4..customer_nodes
   WHERE end_date != '9999-12-31' AND customer_id = 2
ORDER BY customer_id, node_id, region_id
````
<img width="450" alt="image" src="https://user-images.githubusercontent.com/59825363/199304656-ed199af7-729d-46eb-8042-31ca5c81ff57.png">

So I had to readjust my answer, first trialling a aggregate function

````sql
 SELECT customer_id, 
        node_id, 
        SUM(DATEDIFF(DAY,start_date,end_date)) AS reallocation_time
    FROM DannySQLChallenge4..customer_nodes
   WHERE end_date != '9999-12-31'
GROUP BY customer_id, node_id
ORDER BY customer_id, node_id
````
<img width="300" alt="image" src="https://user-images.githubusercontent.com/59825363/199305315-84ff002a-f14a-419b-90e0-943266a3f3b5.png">


I then used this to create a temp table for later

````sql
DROP TABLE IF EXISTS #reallocation_table
CREATE TABLE #reallocation_table
( customer_id VARCHAR(50),
  node_id int,
  region_id int,
  reallocation_days int
)

INSERT INTO #reallocation_table
  SELECT customer_id, 
         node_id, 
         region_id,
         SUM(DATEDIFF(DAY,start_date,end_date)) AS reallocation_time
    FROM DannySQLChallenge4..customer_nodes
   WHERE end_date != '9999-12-31'
GROUP BY customer_id, node_id,region_id
ORDER BY customer_id, node_id,region_id
````


````sql
SELECT AVG(reallocation_days) AS avg_reallocation_time
  FROM #reallocation_table
````
<img width="250" alt="image" src="https://user-images.githubusercontent.com/59825363/199305481-b0376c08-90b1-4f92-98a7-ce70ed788e9d.png">


### What is the median, 80th and 95th percentile for this same reallocation days metric for each region?

````sql
SELECT DISTINCT
       rt.region_id,
       region_name,
       PERCENTILE_DISC(0.50) WITHIN GROUP (ORDER BY reallocation_days) OVER(PARTITION BY rt.region_id) AS discrete_median,
       PERCENTILE_DISC(0.80) WITHIN GROUP (ORDER BY reallocation_days) OVER(PARTITION BY rt.region_id) AS discrete_80th,
       PERCENTILE_DISC(0.95) WITHIN GROUP (ORDER BY reallocation_days) OVER(PARTITION BY rt.region_id) AS discrete_95th,
       PERCENTILE_CONT(0.50) WITHIN GROUP (ORDER BY reallocation_days) OVER(PARTITION BY rt.region_id) AS cont_median,
       PERCENTILE_CONT(0.80) WITHIN GROUP (ORDER BY reallocation_days) OVER(PARTITION BY rt.region_id) AS cont_80th,
       PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY reallocation_days) OVER(PARTITION BY rt.region_id) AS cont_95th	
  FROM #reallocation_table AS rt
  JOIN DannySQLChallenge4..regions AS r ON r.region_id = rt.region_id
````
<img width="600" alt="image" src="https://user-images.githubusercontent.com/59825363/199305637-8112137f-1399-4a7b-a5bd-183caf7a9d64.png">



## B. Customer Transactions

### What is the unique count and total amount for each transaction type?

````sql
  SELECT txn_type,
         COUNT(*) AS transaction_count,
         SUM(txn_amount) AS transaction_amount
    FROM DannySQLChallenge4..Customer_Transactions
GROUP BY txn_type
ORDER BY txn_type
````
<img width="350" alt="image" src="https://user-images.githubusercontent.com/59825363/199305832-71516b4c-6483-4acd-a9ed-f067172f590d.png">

### What is the average total historical deposit counts and amounts for all customers?

````sql
WITH cust_desposits
AS
(
  SELECT customer_id, COUNT(*) AS trans_count, SUM(txn_amount) AS total_deposits
    FROM DannySQLChallenge4..Customer_Transactions
   WHERE txn_type = 'deposit'
GROUP BY customer_id

)

SELECT AVG(trans_count) AS avg_historical_desposits,
	   AVG(total_deposits) AS avg_deposits
  FROM cust_desposits
````
<img width="250" alt="image" src="https://user-images.githubusercontent.com/59825363/199305952-ccc17196-6869-46b7-82ea-81d774d0f100.png">



### For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?

````sql
WITH bank_hist
AS
(
SELECT *,
       DATEPART(month,txn_date) AS month_num,
       DATENAME(month,txn_date) AS month,
      CASE 
          WHEN txn_type = 'deposit' 
          THEN 1
          ELSE 0
      END AS deposit_made,
      CASE 
          WHEN txn_type = 'purchase' 
          THEN 1
          WHEN txn_type = 'withdrawal'
          THEN 1
          ELSE 0
      END AS purchase_or_withdrawal	
  FROM DannySQLChallenge4..Customer_Transactions
),
customer_usage
AS
(
   SELECT customer_id,
         month_num,
         month,
         SUM(deposit_made) AS total_desposits,
         SUM(purchase_or_withdrawal) AS total_outs
    FROM bank_hist
GROUP BY customer_id, month_num, month
)

  SELECT month_num,
         month,
         COUNT (customer_id) AS cust_count
    FROM customer_usage
   WHERE total_desposits > 1 AND total_outs > 1
GROUP BY month_num, month
ORDER BY month_num, month
````
<img width="300" alt="image" src="https://user-images.githubusercontent.com/59825363/199310236-f5bd684a-620d-43bc-aee6-d542d9ae20a7.png">


### What is the closing balance for each customer at the end of the month?

````sql
WITH
updated_trans
AS
(
    SELECT customer_id,
          SUM(CASE 
                 WHEN txn_type = 'deposit' THEN txn_amount
                 ELSE -txn_amount
             END) AS new_trans,
         DATEPART(month,txn_date) AS month_num,
         DATENAME(month,txn_date) AS month
    FROM DannySQLChallenge4..Customer_Transactions
GROUP BY customer_id, DATEPART(month,txn_date), DATENAME(month,txn_date)
)

SELECT customer_id, 
       month_num, 
       month,
       new_trans,
       SUM(new_trans) OVER(PARTITION BY customer_id ORDER BY month_num ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS closing_balance
  FROM updated_trans
````
<img width="450" alt="image" src="https://user-images.githubusercontent.com/59825363/199310755-a25b1142-597c-4146-935c-972be9157ac3.png">


### What is the percentage of customers who increase their closing balance by more than 5%?

````sql

````


