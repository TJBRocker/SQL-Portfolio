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

SELECT CONCAT(CAST(ROUND(100.0*COUNT(DISTINCT visit_id)/
			 (SELECT COUNT(DISTINCT visit_id) FROM DannySQLChallenge6..events),2) 
			  AS FLOAT),'%') AS percentage_purchase
  FROM DannySQLChallenge6..events AS e
  JOIN DannySQLChallenge6..event_identifier AS ei ON ei.event_type = e.event_type
 WHERE e.event_type = 3

````

![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/2b33aeea-2d73-4822-b79a-c245d2043435)


6.  What is the percentage of visits which view the checkout page but do not have a purchase event?

````sql

WITH CTE AS(
SELECT visit_id,
	   CASE WHEN e.event_type = 3 THEN 1 ELSE 0 END AS purchased,
	   CASE WHEN ph.page_id = 12 THEN 1 ELSE 0 END AS checkout
  FROM DannySQLChallenge6..events AS e
  JOIN DannySQLChallenge6..page_hierarchy AS ph ON e.page_id = ph.page_id
  JOIN DannySQLChallenge6..event_identifier AS ei ON ei.event_type = e.event_type
  )

SELECT CONCAT(CAST(ROUND(100.0*(SUM(checkout) - SUM(purchased)) 
	   / (COUNT(DISTINCT visit_id)),2) AS FLOAT),'%') AS checkout_visit_without_purchase
  FROM CTE


````

![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/09c853b8-36f5-44f5-8977-e5731d3dc40e)


7.  What are the top 3 pages by number of views?

````sql

  SELECT TOP 3 ph.page_id,page_name, COUNT(*) AS page_views
    FROM DannySQLChallenge6..events AS e
    JOIN DannySQLChallenge6..page_hierarchy AS ph ON e.page_id = ph.page_id
    JOIN DannySQLChallenge6..event_identifier AS ei ON ei.event_type = e.event_type
   WHERE event_name = 'Page View'
GROUP BY ph.page_id,page_name
ORDER BY 3 DESC

````

![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/af02ca4d-1902-484a-b886-bea33c54e03f)

I'm going to exclude any pages that are not product pages

````sql

  SELECT TOP 3 ph.page_id,page_name, COUNT(*) AS page_views
    FROM DannySQLChallenge6..events AS e
    JOIN DannySQLChallenge6..page_hierarchy AS ph ON e.page_id = ph.page_id
    JOIN DannySQLChallenge6..event_identifier AS ei ON ei.event_type = e.event_type
   WHERE event_name = 'Page View' AND e.page_id NOT IN (1,2,12)
GROUP BY ph.page_id,page_name
ORDER BY 3 DESC

````

![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/f0c83f48-53d1-4c8a-ae3a-a9734afb693d)

8.  What is the number of views and cart adds for each product category?

````sql

  SELECT product_category,
	     SUM(CASE WHEN e.event_type = 1 THEN 1 ELSE 0 END) AS views_count,
	     SUM(CASE WHEN e.event_type = 2 THEN 1 ELSE 0 END) AS cart_count
    FROM DannySQLChallenge6..events AS e
    JOIN DannySQLChallenge6..page_hierarchy AS ph ON e.page_id = ph.page_id
    JOIN DannySQLChallenge6..event_identifier AS ei ON ei.event_type = e.event_type
   WHERE product_category IS NOT NULL
GROUP BY product_category
ORDER BY product_category


````

![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/1633ff86-ce33-4048-a970-af4d03ad4516)


9.  What are the top 3 products by purchases?

````sql

WITH purchases_id AS(
SELECT visit_id
  FROM DannySQLChallenge6..events AS e
  JOIN DannySQLChallenge6..event_identifier AS ei ON ei.event_type = e.event_type
 WHERE LOWER(event_name) = 'purchase'
)

SELECT TOP 3 page_name AS product_purchased,
	   COUNT(*) number_purchased
  FROM DannySQLChallenge6..events AS e
  JOIN DannySQLChallenge6..page_hierarchy AS ph ON e.page_id = ph.page_id
  JOIN DannySQLChallenge6..event_identifier AS ei ON ei.event_type = e.event_type
WHERE LOWER(event_name) = 'add to cart' AND visit_id IN (SELECT * FROM purchases_id)
GROUP BY page_name
ORDER BY number_purchased DESC

````

![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/d19c5251-86ad-40c7-9cc8-e2db5d6dbb54)

### B. Product Funnel Analysis

Using a single SQL query - create a new output table which has the following details:

-  How many times was each product viewed?
-  How many times was each product added to cart?
-  How many times was each product added to a cart but not purchased (abandoned)?
-  How many times was each product purchased?

I initially did this as one big table and then split it out into two, I really think that one table is the way forward as you can perform more queries that way

Combo table:

````sql

 DROP TABLE IF EXISTS DannySQLChallenge6..combo_funnel
 CREATE TABLE DannySQLChallenge6..combo_funnel
 (product_category VARCHAR(50),
 product_name 
 VARCHAR(50),
 viewed INT,
 added INT,
 abandoned INT,
 purchased INT
 )


WITH CTE AS(
  SELECT visit_id, product_category, page_name AS product_name,
		 CASE WHEN e.event_type = 1 THEN 1 ELSE 0 END AS page_view,
		 CASE WHEN e.event_type = 2 THEN 1 ELSE 0 END AS cart_add,
		 FIRST_VALUE(event_name) OVER(PARTITION BY visit_id ORDER BY sequence_number DESC) AS last_action
    FROM DannySQLChallenge6..events AS e
    JOIN DannySQLChallenge6..page_hierarchy AS ph ON e.page_id = ph.page_id
    JOIN DannySQLChallenge6..event_identifier AS ei ON ei.event_type = e.event_type
)
INSERT INTO DannySQLChallenge6..combo_funnel
SELECT product_category,
	   product_name,
	   SUM(page_view) AS viewed, 
	   SUM(cart_add) AS added,
	   SUM(CASE WHEN cart_add = 1 AND LOWER(last_action) <> 'purchase' THEN 1 ELSE 0 END) AS abandoned,
	   SUM(CASE WHEN LOWER(last_action) = 'purchase' AND cart_add =  1 THEN 1 ELSE 0 END) AS purchased
  FROM CTE
 WHERE product_category IS NOT NULL
GROUP BY product_category, product_name
ORDER BY product_category, product_name

````

![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/9a8dc888-f75f-4474-a4bc-fa33f758555c)


Product funnel analysis:

````sql

 DROP TABLE IF EXISTS DannySQLChallenge6..product_funnel
 CREATE TABLE DannySQLChallenge6..product_funnel
 (
 product_name VARCHAR(50),
 viewed INT,
 added INT,
 abandoned INT,
 purchased INT
 )
 
 WITH CTE AS(
  SELECT visit_id, product_category, page_name AS product_name,
		 CASE WHEN e.event_type = 1 THEN 1 ELSE 0 END AS page_view,
		 CASE WHEN e.event_type = 2 THEN 1 ELSE 0 END AS cart_add,
		 FIRST_VALUE(event_name) OVER(PARTITION BY visit_id ORDER BY sequence_number DESC) AS last_action
    FROM DannySQLChallenge6..events AS e
    JOIN DannySQLChallenge6..page_hierarchy AS ph ON e.page_id = ph.page_id
    JOIN DannySQLChallenge6..event_identifier AS ei ON ei.event_type = e.event_type
)
INSERT INTO DannySQLChallenge6..product_funnel
SELECT 
	   product_name,	   
	   SUM(page_view) AS viewed, 
	   SUM(cart_add) AS added,
	   SUM(CASE WHEN cart_add = 1 AND LOWER(last_action) <> 'purchase' THEN 1 ELSE 0 END) AS abandoned,
	   SUM(CASE WHEN LOWER(last_action) = 'purchase' AND cart_add =  1 THEN 1 ELSE 0 END) AS purchased
  FROM CTE
 WHERE product_category IS NOT NULL
GROUP BY  product_name
ORDER BY product_name

````

![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/85ceb700-c6a5-42af-be43-dc08235ca8e8)


Category funnel:

````sql

 DROP TABLE IF EXISTS DannySQLChallenge6..category_funnel
 CREATE TABLE DannySQLChallenge6..category_funnel
 (
 product_category VARCHAR(50),
 viewed INT,
 added INT,
 abandoned INT,
 purchased INT
 )

 WITH CTE AS(
  SELECT visit_id, product_category, page_name AS product_name,
		 CASE WHEN e.event_type = 1 THEN 1 ELSE 0 END AS page_view,
		 CASE WHEN e.event_type = 2 THEN 1 ELSE 0 END AS cart_add,
		 FIRST_VALUE(event_name) OVER(PARTITION BY visit_id ORDER BY sequence_number DESC) AS last_action
    FROM DannySQLChallenge6..events AS e
    JOIN DannySQLChallenge6..page_hierarchy AS ph ON e.page_id = ph.page_id
    JOIN DannySQLChallenge6..event_identifier AS ei ON ei.event_type = e.event_type
)

INSERT INTO DannySQLChallenge6..category_funnel
SELECT 
	   product_category,	   
	   SUM(page_view) AS viewed, 
	   SUM(cart_add) AS added,
	   SUM(CASE WHEN cart_add = 1 AND LOWER(last_action) <> 'purchase' THEN 1 ELSE 0 END) AS abandoned,
	   SUM(CASE WHEN LOWER(last_action) = 'purchase' AND cart_add =  1 THEN 1 ELSE 0 END) AS purchased
  FROM CTE
 WHERE product_category IS NOT NULL
GROUP BY  product_category
ORDER BY product_category

````

![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/733fa088-2bdb-49a8-9068-c7cd68dededb)



Additionally, create another table which further aggregates the data for the above points but this time for each product category instead of individual products.

Use your 2 new output tables - answer the following questions:

1.  Which product had the most views, cart adds and purchases?

````sql

  SELECT TOP 1 product_name, SUM(viewed+added+purchased) AS total_actions
    FROM DannySQLChallenge6..combo_funnel
GROUP BY product_name
ORDER BY total_actions DESC

````

![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/dbb929bb-a49f-4c88-b653-75a16fb8f912)


2.  Which product was most likely to be abandoned?

````sql

  SELECT TOP 1 product_name, abandoned
    FROM DannySQLChallenge6..combo_funnel
ORDER BY abandoned DESC


````

![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/7fbc2fa4-4329-4aba-b8d4-fd93c8c1bc72)


3.  Which product had the highest view to purchase percentage?

````sql

  SELECT TOP 1 product_name, 
		 CONCAT(CAST(ROUND(100.0*purchased/viewed,2) AS float),'%') AS view_purch_ratio
    FROM DannySQLChallenge6..combo_funnel
ORDER BY 2 DESC

````

![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/687c66cb-2cab-4516-a19b-1dd09949e294)

4.  What is the average conversion rate from view to cart add?

````sql

  SELECT CONCAT(ROUND(AVG(CAST(100.0*added/viewed AS float)),2),'%')  AS add_conversion
    FROM DannySQLChallenge6..combo_funnel

````

![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/fee5d095-ae10-496d-b5f0-6293d432595e)


5.  What is the average conversion rate from cart add to purchase?

````sql

  SELECT CONCAT(ROUND(AVG(CAST(100.0*added/viewed AS float)),2),'%')  AS add_conversion,
		 CONCAT(ROUND(AVG(CAST(100.0*purchased/added AS float)),2),'%')  AS purch_conversion
    FROM DannySQLChallenge6..combo_funnel

````

![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/72731100-7700-4f86-9a0c-29327e880b05)


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


````sql

DROP TABLE IF EXISTS DannySQLChallenge6..campaign_analysis
CREATE TABLE DannySQLChallenge6..campaign_analysis
 (user_id VARCHAR(50),
 visit_id VARCHAR(255),
 visit_start_time TIMESTAMP,
 page_views INT,
 cart_adds INT,
 purchase INT,
 ad_impressions INT,
 clicks INT,
 campaign_name VARCHAR(50),
 cart_products VARCHAR(255)
 )



 WITH CTE AS ( 
 SELECT  user_id,
		 e.visit_id,
		 STRING_AGG(page_name, ', ') WITHIN GROUP(ORDER BY sequence_number ASC) AS cart_products
    FROM DannySQLChallenge6..events AS e
    JOIN DannySQLChallenge6..page_hierarchy AS ph ON e.page_id = ph.page_id
    JOIN DannySQLChallenge6..event_identifier AS ei ON ei.event_type = e.event_type
	JOIN DannySQLChallenge6..users AS u ON u.cookie_id = e.cookie_id
   WHERE e.event_type = 2
GROUP BY user_id, e.visit_id
)   
INSERT INTO  DannySQLChallenge6..campaign_analysis
SELECT    u.user_id,
		  e.visit_id, 
	      MIN(event_time) AS visit_start_time,
		  SUM(CASE WHEN e.event_type = 1 THEN 1 ELSE 0 END) AS page_views,
		  SUM(CASE WHEN e.event_type = 2 THEN 1 ELSE 0 END) AS cart_adds,
		  SUM(CASE WHEN e.event_type = 3 THEN 1 ELSE 0 END) AS purchase,
		  SUM(CASE WHEN e.event_type = 4 THEN 1 ELSE 0 END) AS ad_impressions,
	 	  SUM(CASE WHEN e.event_type = 5 THEN 1 ELSE 0 END) AS clicks,
		  campaign_name,
		  cart_products
     FROM DannySQLChallenge6..events AS e
     JOIN DannySQLChallenge6..page_hierarchy AS ph ON e.page_id = ph.page_id
     JOIN DannySQLChallenge6..event_identifier AS ei ON ei.event_type = e.event_type
     JOIN DannySQLChallenge6..users AS u ON u.cookie_id = e.cookie_id
 LEFT JOIN DannySQLChallenge6..campaign_identifier AS ci ON e.event_time BETWEEN ci.start_date AND ci.end_date
 LEFT JOIN CTE AS cte ON CTE.visit_id = e.visit_id
 GROUP BY u.user_id, e.visit_id, campaign_name, cart_products
 ORDER BY u.user_id, e.visit_id

````

Snapshot:

![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/98cc5a77-ecc1-4106-85ca-bf7ab98775ef)


````sql



````

````sql



````


