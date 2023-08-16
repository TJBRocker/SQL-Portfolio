### A. Digital Analysis

Using the available datasets - answer the following questions using a single query for each one:

1.  How many users are there?

````sql

SELECT COUNT(DISTINCT user_id) AS total_users
  FROM DannySQLChallenge6..users

````

![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/a2ec1b84-60dc-4e5c-85b6-fc18c8e8fe00)


2.  How many cookies does each user have on average?

````sql

WITH CTE
AS
(
SELECT user_id AS total_users,
	   CAST(COUNT(cookie_id) AS DECIMAL) AS cookies
  FROM DannySQLChallenge6..users
GROUP BY user_id
)

SELECT CAST(ROUND(AVG(cookies),2)AS FLOAT) AS avg_num_cookies
  FROM CTE

````

![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/b28999c9-a96b-4dc7-b339-30ca0657f5d7)


3.  What is the unique number of visits by all users per month?

````sql

SELECT DATEPART(month,LEFT(event_time,10)) AS month_number,
	   DATENAME(month,LEFT(event_time,10)) AS month_name,
	   COUNT(DISTINCT visit_id) AS count
  FROM DannySQLChallenge6..events
GROUP BY DATEPART(month,LEFT(event_time,10)), DATENAME(month,LEFT(event_time,10))
ORDER BY 1,2

````

![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/37456e74-f1b7-4046-93d3-8868a79767db)


4.  What is the number of events for each event type?

````sql

  SELECT e.event_type,
	     event_name,
	     COUNT(visit_id) AS number_of_events
    FROM DannySQLChallenge6..events AS e
    JOIN DannySQLChallenge6..event_identifier AS ei ON ei.event_type = e.event_type
GROUP BY e.event_type, event_name
ORDER BY e.event_type, event_name

````

![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/aabaec5e-c54c-4003-a6f7-0213b010a80f)


5.  What is the percentage of visits which have a purchase event?

````sql

SELECT CONCAT(CAST(ROUND(100.0*COUNT(DISTINCT visit_id)/(SELECT COUNT(DISTINCT visit_id) FROM DannySQLChallenge6..events),2) AS FLOAT),'%') AS percentage_purchase
  FROM DannySQLChallenge6..events AS e
  JOIN DannySQLChallenge6..event_identifier AS ei ON ei.event_type = e.event_type
 WHERE e.event_type = 3

````

![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/2b33aeea-2d73-4822-b79a-c245d2043435)


6.  What is the percentage of visits which view the checkout page but do not have a purchase event?

````sql



````

7.  What are the top 3 pages by number of views?

````sql



````

8.  What is the number of views and cart adds for each product category?

````sql



````

9.  What are the top 3 products by purchases?

````sql



````


### B. Product Funnel Analysis

Using a single SQL query - create a new output table which has the following details:

-  How many times was each product viewed?
-  How many times was each product added to cart?
-  How many times was each product added to a cart but not purchased (abandoned)?
-  How many times was each product purchased?

Additionally, create another table which further aggregates the data for the above points but this time for each product category instead of individual products.

Use your 2 new output tables - answer the following questions:

1.  Which product had the most views, cart adds and purchases?
2.  Which product was most likely to be abandoned?
3.  Which product had the highest view to purchase percentage?
4.  What is the average conversion rate from view to cart add?
5.  What is the average conversion rate from cart add to purchase?

### C. Campaigns Analysis

Generate a table that has 1 single row for every unique `visit_id` record and has the following columns:

-  `user_id`
-  `visit_id`
-  `visit_start_time`: the earliest `event_time` for each visit
-  `page_views`: count of page views for each visit
-  `cart_adds`: count of product cart add events for each visit
-  `purchase`: 1/0 flag if a purchase event exists for each visit
-  `campaign_name`: map the visit to a campaign if the `visit_start_time` falls between the start_date and end_date
-  `impression`: count of ad impressions for each visit
-  `click`: count of ad clicks for each visit
-  (Optional column) `cart_products`: a comma separated text value with products added to the cart sorted by the order they were added to the cart (hint: use the `sequence_number`)

