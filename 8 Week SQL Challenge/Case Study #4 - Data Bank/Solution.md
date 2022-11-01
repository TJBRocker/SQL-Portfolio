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
         COUNT(customer_id) AS customer_count
    FROM DannySQLChallenge4..customer_nodes AS cn
    JOIN DannySQLChallenge4..regions AS r ON  r.region_id = cn.region_id
GROUP BY cn.region_id, region_name
ORDER BY cn. region_id, region_name
````

### How many days on average are customers reallocated to a different node?

````sql

````

### What is the median, 80th and 95th percentile for this same reallocation days metric for each region?

````sql

````

## B. Customer Transactions

### What is the unique count and total amount for each transaction type?

````sql

````

### What is the average total historical deposit counts and amounts for all customers?

````sql

````

### For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?

````sql

````

### What is the closing balance for each customer at the end of the month?

````sql

````

### What is the percentage of customers who increase their closing balance by more than 5%?

````sql

````


