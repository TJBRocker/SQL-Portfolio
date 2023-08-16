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



````

3.  What is the unique number of visits by all users per month?

````sql



````

4.  What is the number of events for each event type?

````sql



````

5.  What is the percentage of visits which have a purchase event?

````sql



````

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

