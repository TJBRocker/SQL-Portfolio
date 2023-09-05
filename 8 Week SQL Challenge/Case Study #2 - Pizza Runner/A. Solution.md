# Case Study #2 - Pizza Runner

## A. Pizza Metrics

### 1.  How many pizzas were ordered?

````sql
SELECT COUNT(pizza_id) AS total_pizzas_ordered
  FROM DannySQLChallenge2..customer_orders
````

<img width="200" alt="image" src="https://user-images.githubusercontent.com/59825363/197053965-de0e20b6-6e29-4a54-8434-4f7becb720ef.png">


### 2.  How many unique customer orders were made?

````sql
SELECT COUNT(DISTINCT order_id) AS unique_orders
  FROM DannySQLChallenge2..customer_orders
````

<img width="200" alt="image" src="https://user-images.githubusercontent.com/59825363/197054465-b5932611-d312-4d31-baa1-505342ced90a.png">


### 3.  How many successful orders were delivered by each runner?

````sql
SELECT runner_id, 
       COUNT(runner_id) AS successful_orders
  FROM DannySQLChallenge2..runner_order_new
 WHERE  distance <> ''
 GROUP BY runner_id
````
<img width="200" alt="image" src="https://user-images.githubusercontent.com/59825363/197054509-89b0b195-6f56-46dd-8017-88b30eff79c6.png">



### 4.  How many of each type of pizza was delivered?

````sql
SELECT pn.pizza_name, COUNT(co.pizza_id) AS pizza_order_count
  FROM DannySQLChallenge2..customer_orders AS co
  JOIN DannySQLChallenge2..pizza_names AS pn ON pn.pizza_id = co.pizza_id
  JOIN DannySQLChallenge2..runner_order_new AS ron ON co.order_id = ron.order_id
 WHERE cancellation = ''
GROUP BY pn.pizza_id, pn.pizza_name


````
![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/055335db-784b-42d8-afed-28fb5d4af1da)



### 5.  How many Vegetarian and Meatlovers were ordered by each customer?

````sql
  SELECT customer_id, pn.pizza_name, 
	 COUNT(co.pizza_id) AS pizza_order_count
    FROM DannySQLChallenge2..customer_orders AS co
    JOIN DannySQLChallenge2..pizza_names AS pn ON pn.pizza_id = co.pizza_id
GROUP BY customer_id, pn.pizza_id, pn.pizza_name;

````

![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/26aaa7d1-f543-4446-ae19-2396993c7e78)

Excluding cancellations

````sql
SELECT customer_id, pn.pizza_name, COUNT(co.pizza_id) AS pizza_order_count
  FROM DannySQLChallenge2..customer_orders AS co
  JOIN DannySQLChallenge2..pizza_names AS pn ON pn.pizza_id = co.pizza_id
  JOIN DannySQLChallenge2..runner_order_new AS ron ON co.order_id = ron.order_id
 WHERE cancellation = ''
GROUP BY customer_id, pn.pizza_id, pn.pizza_name

````
![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/9f61051a-295e-46db-89ff-88561eb8fd45)


### 6.  What was the maximum number of pizzas delivered in a single order?

````sql
WITH pizza_orders
AS(
  SELECT order_id, 
  	 COUNT(order_id) AS pizzas_per_order
    FROM DannySQLChallenge2..customer_orders
GROUP BY order_id
)

SELECT max(pizzas_per_order) AS max_ordered
  FROM pizza_orders
  
````
![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/91ab5cba-b2f3-43cf-a78b-2de43ec84009)

### 7.  For each customer, how many delivered pizzas had at least 1 change and how many had no changes?

````sql
SELECT co.customer_id,
	   SUM(CASE WHEN co.exclusions <> '' OR co.extras <> '' THEN 1 ELSE 0 END) AS pizza_changes,
	   SUM(CASE WHEN co.exclusions = '' AND co.extras = '' THEN 1 ELSE 0 END) AS no_changes
  FROM DannySQLChallenge2..customer_orders AS co
  JOIN DannySQLChallenge2..runner_order_new AS ro ON ro.order_id = co.order_id
WHERE distance <> '' AND cancellation = ''
GROUP BY customer_id
````

![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/a0e25951-aaab-4b4d-a2f5-36c1ec418d12)

### 8.  How many pizzas were delivered that had both exclusions and extras?

````sql
SELECT COUNT(DISTINCT co.order_id) AS excl_extras
  FROM DannySQLChallenge2..customer_orders AS co
  JOIN DannySQLChallenge2..runner_order_new AS ro ON ro.order_id = co.order_id
WHERE distance <> '' AND exclusions <> '' AND extras <> ''
````

![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/6737c855-fe81-49c0-a173-721905e41515)

### 9.  What was the total volume of pizzas ordered for each hour of the day?

````sql
 SELECT DATEPART(HOUR, order_time) AS hour, COUNT(co.order_id) AS pizzas_ordered
  FROM DannySQLChallenge2..customer_orders AS co
  JOIN DannySQLChallenge2..runner_order_new AS ro ON ro.order_id = co.order_id
 GROUP BY DATEPART(HOUR, order_time)
````

![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/2d200234-7562-4915-b182-3573adcf00b0)


### 10. What was the volume of orders for each day of the week?

````sql
 SET datefirst 1;
   SELECT 
 	  DATEPART(WEEKDAY, order_time) AS day_num, 
	  DATENAME(WEEKDAY, order_time) AS weekday,
          COUNT(co.order_id) AS pizzas_ordered 
     FROM DannySQLChallenge2..customer_orders AS co
     JOIN DannySQLChallenge2..runner_order_new AS ro ON ro.order_id = co.order_id
 GROUP BY DATEPART(WEEKDAY, order_time), DATENAME(WEEKDAY, order_time)
````

![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/e8dfba34-02b0-46fb-a2a7-17cec400974c)

## B. Runner and Customer Experience

### 1.  How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)

````sql
SET datefirst 1;
  SELECT DATEPART(week,registration_date) AS start_week, 
  	 COUNT(runner_id) AS runner_count
    FROM DannySQLChallenge2..runners
GROUP BY DATEPART(week,registration_date)
````

![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/8ad2b4f6-3511-4f5d-97fe-d176f193a2e1)

### 2.  What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?

````sql
WITH time_taken AS
(
SELECT DISTINCT(ron.order_id),runner_id, 
DATEDIFF(MINUTE, order_time, pickup_time) AS pickup_time
  FROM DannySQLChallenge2..runner_order_new AS ron
  JOIN DannySQLChallenge2..customer_orders AS co ON co.order_id = ron.order_id
 WHERE cancellation = ''
)

SELECT runner_id, 
       CONCAT(AVG(pickup_time),' minutes') AS average_pickup
  FROM time_taken
GROUP BY runner_id
````
![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/ad3010ec-d912-42c6-a323-5448efcc5c79)

### 3.  Is there any relationship between the number of pizzas and how long the order takes to prepare?

````sql
WITH CTE AS(

SELECT DISTINCT ron.order_id, runner_id, DATEDIFF(MINUTE, order_time, pickup_time) AS pickup_time, COUNT(co.pizza_id) AS pizza_count
  FROM DannySQLChallenge2..runner_order_new AS ron
  JOIN DannySQLChallenge2..customer_orders AS co ON co.order_id = ron.order_id
 WHERE cancellation = ''
 GROUP BY ron.order_id, runner_id, DATEDIFF(MINUTE, order_time, pickup_time)
)

SELECT pizza_count, CONCAT(AVG(pickup_time), ' minutes') AS avg_prep
  FROM CTE
GROUP BY pizza_count
````
![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/f1c1a439-ad8e-4c78-8ec5-8b7c5534e4db)


### 4.  What was the average distance travelled for each customer?

````sql
WITH distance_by_order
AS(
SELECT DISTINCT(co.order_id), co.customer_id, distance
  FROM DannySQLChallenge2..customer_orders AS co
  JOIN DannySQLChallenge2..runner_order_new AS ron ON ron.order_id=co.order_id 
 WHERE cancellation = ''
)

SELECT customer_id, CONCAT(AVG(CAST(distance AS float)),' km') AS average_distance
FROM distance_by_order
GROUP BY customer_id
````
![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/689f4d2b-6dae-4c90-9087-e381e3e78291)

### 5.  What was the difference between the longest and shortest delivery times for all orders?

````sql
SELECT  CONCAT(MAX(CAST(duration AS float)) - MIN(CAST(duration AS float)), ' minutes') AS delivery_time_difference
  FROM DannySQLChallenge2..runner_order_new
WHERE distance <> ''
````

![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/85d1dde3-adbc-48f5-a4b5-5b34a16c7f65)


### 6.  What was the average speed for each runner for each delivery and do you notice any trend for these values?

````sql
WITH runner_speed
AS(
SELECT  runner_id, ron.order_id, ROUND(CAST(ron.distance AS float) / (CAST(ron.duration AS float)/60),2) AS avg_speed
  FROM DannySQLChallenge2..runner_order_new AS ron
WHERE distance <> ''
)

SELECT runner_id, 
	  CONCAT(ROUND(AVG(avg_speed),1),' km/h') AS avg_speed, 
	  CONCAT(MAX(avg_speed)-MIN(avg_speed),' km/h')  AS speed_variation
  FROM runner_speed
GROUP BY runner_id
````

![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/08303629-4026-45a3-bda4-75f323685c1c)

### 7.  What is the successful delivery percentage for each runner?

````sql
SELECT runner_id, CONCAT( 
       SUM(100*CASE 
		   WHEN distance = '' 
		   THEN 0
		   ELSE 1 
		   END) / COUNT(*),'%') AS success_rate
  FROM DannySQLChallenge2..runner_order_new
GROUP BY runner_id
````
![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/a1bd5071-3b57-475c-be3f-84c87ed2b234)


## C. Ingredient Optimisation

### 1.  What are the standard ingredients for each pizza?

````sql
DROP TABLE IF EXISTS dannySQLchallenge2..cleaned_pizza_toppings
 CREATE TABLE dannySQLchallenge2..cleaned_pizza_toppings
 (pizza_id INT,
  topping_id INT,
  topping_name VARCHAR(50)
)

INSERT INTO DannySQLChallenge2..cleaned_pizza_toppings
SELECT pr.pizza_id, 
       pt.topping_id, 
       pt.topping_name
  FROM DannySQLChallenge2..pizza_recipes AS pr
 CROSS APPLY STRING_SPLIT(CAST(toppings AS varchar),',') AS val
  JOIN DannySQLChallenge2..pizza_toppings AS pt ON TRIM(val.value) = pt.topping_id

SELECT *
  FROM DannySQLChallenge2..cleaned_pizza_toppings
  
  <img width="200" alt="image" src="https://user-images.githubusercontent.com/59825363/197058880-46c2c342-f044-430b-90ac-0c484a91b12c.png">

SELECT cpt.pizza_id, pizza_name, S
       STRING_AGG(topping_name, ', ') AS toppings
  FROM DannySQLChallenge2..cleaned_pizza_toppings AS cpt
  JOIN DannySQLChallenge2..pizza_names AS pn ON cpt.pizza_id = pn.pizza_id
GROUP BY cpt.pizza_id, pizza_name
````
![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/7f5a7cf0-4173-4c1b-a7d5-d59b900195a5)


### 2.  What was the most commonly added extra?

````sql
WITH extras_count AS
(
SELECT TRIM(value) AS topping_id, 
       CAST(topping_name AS varchar(100)) AS topping_name
  FROM DannySQLChallenge2..customer_orders
 CROSS APPLY STRING_SPLIT(extras,',') AS val
  JOIN DannySQLChallenge2..pizza_toppings AS pt ON pt.topping_id = TRIM(val.value)
 WHERE value <> ''
)

SELECT topping_id, topping_name , 
       COUNT(*) AS item_count
  FROM extras_count
GROUP BY topping_id, topping_name
````
![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/2edd787b-59cc-4df2-9a4a-f197a578ab1e)


### 3.  What was the most common exclusion?

````sql
WITH exclusions_list
AS
(
SELECT TRIM(value) AS topping_id, 
	   CAST(topping_name AS varchar(100)) AS topping_name
  FROM DannySQLChallenge2..customer_orders
	   CROSS APPLY STRING_SPLIT(exclusions, ',') AS val
  JOIN DannySQLChallenge2..pizza_toppings AS pt ON pt.topping_id = TRIM(val.value)
 WHERE value <> ''
)

SELECT topping_id, topping_name, count(*) AS total
  FROM exclusions_list
GROUP BY topping_id, topping_name
ORDER BY total DESC
````
![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/07c20721-a39d-4d0f-bb60-4501b08a02df)

### 4.  Generate an order item for each record in the customers_orders table in the format of one of the following:
  - Meat Lovers
  - Meat Lovers - Exclude Beef
  - Meat Lovers - Extra Bacon
  - Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers

````sql
WITH pizza_numbers
AS
(
SELECT order_id, customer_id, pizza_name, exclusions, extras,
	   ROW_NUMBER() OVER(ORDER BY order_id) AS pizza_number
  FROM DannySQLChallenge2..customer_orders AS co
JOIN DannySQLChallenge2..pizza_names AS pn ON pn.pizza_id = co.pizza_id
), 
excl_tab AS(
SELECT co.order_id, co.customer_id,pizza_number, pizza_name, 
	   CASE WHEN exclusions <>'' THEN '- Exclude ' ELSE '' END AS text_addition, 
	   TRIM(excl1.value) AS exclusions_extras
  FROM pizza_numbers AS co
  CROSS APPLY STRING_SPLIT(exclusions, ',') AS excl1
), ex_tab AS
(
SELECT co.order_id, co.customer_id,pizza_number, pizza_name,
	   CASE WHEN extras <> '' THEN '- Extras ' ELSE '' END AS text_addition, 
	   TRIM(extr1.value) AS exclusions_extras
  FROM pizza_numbers AS co
  CROSS APPLY STRING_SPLIT(extras, ',') AS extr1
), combo_tab AS
(
SELECT order_id, customer_id, pizza_number, pizza_name, text_addition, exclusions_extras, topping_id, topping_name
  FROM excl_tab
 JOIN DannySQLChallenge2..cleaned_pizza_toppings AS cpt ON cpt.topping_id = exclusions_extras
 WHERE text_addition <> ''
UNION
SELECT order_id, customer_id, pizza_number, pizza_name, text_addition, exclusions_extras, topping_id, topping_name
  FROM ex_tab
JOIN DannySQLChallenge2..cleaned_pizza_toppings AS cpt ON cpt.topping_id = exclusions_extras
 WHERE text_addition <> ''
), combo_tab2 AS
(
SELECT pizza_number, TRIM(pizza_name) AS pizza_name, TRIM(text_addition) AS text_addition, STRING_AGG(topping_name, ', ') AS agg
  FROM combo_tab
GROUP BY pizza_number, TRIM(pizza_name), TRIM(text_addition)
), combo_tab3 AS
(
SELECT order_id, customer_id, pns.pizza_name,pns.pizza_number, CONCAT(text_addition,' ', agg) AS agg
  FROM pizza_numbers AS pns
LEFT JOIN combo_tab2 AS ct2 ON ct2.pizza_number = pns.pizza_number
), combo_tab4 AS
(
SELECT order_id, customer_id,pizza_name, STRING_AGG(agg,' ') AS agg
  FROM combo_tab3
GROUP BY order_id, customer_id, pizza_name, pizza_number
)
SELECT order_id, customer_id, CONCAT(pizza_name,' ',agg) AS pizza_order
  FROM combo_tab4
````
![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/81208024-5baa-4312-9bc3-6501d6c713a3)

### 5.	Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients. For example: ""Meat Lovers: 2xBacon, Beef, ... , Salami""

````sql
WITH recipe_split AS
(
     SELECT pr.pizza_id, pizza_name, TRIM(split.value) AS topping_id
       FROM DannySQLChallenge2..pizza_recipes AS pr
	   JOIN DannySQLChallenge2..pizza_names AS pn ON pn.pizza_id = pr.pizza_id
CROSS APPLY STRING_SPLIT(CAST(toppings AS nvarchar), ',') AS split
), pizza_listed AS(

SELECT DISTINCT rs.pizza_id, pizza_name, rs.topping_id, topping_name, 1 AS topping_count
  FROM recipe_split AS rs
  JOIN DannySQLChallenge2..cleaned_pizza_toppings AS cpt ON cpt.topping_id = rs.topping_id
), pizza_numbers AS
(
SELECT order_id, customer_id, co.pizza_id, pizza_name, exclusions, extras,
	   ROW_NUMBER() OVER(ORDER BY order_id) AS pizza_number
  FROM DannySQLChallenge2..customer_orders AS co
JOIN DannySQLChallenge2..pizza_names AS pn ON pn.pizza_id = co.pizza_id
), exclusions_tab AS
(
SELECT co.order_id, co.customer_id,pizza_number, pizza_name, 
	   TRIM(excl1.value) AS extra_excl, CASE WHEN TRIM(excl1.value) <> ''  THEN -1 ELSE 0 END AS topping_addition_subtraction
  FROM pizza_numbers AS co
  CROSS APPLY STRING_SPLIT(exclusions, ',') AS excl1
 WHERE exclusions <> ''
), extras_tab AS
(
SELECT co.order_id, co.customer_id,pizza_number, pizza_name, 
	   TRIM(extra1.value) AS extra_excl, CASE WHEN TRIM(extra1.value) <> ''  THEN 1 ELSE 0 END AS topping_addition_subtraction
  FROM pizza_numbers AS co
  CROSS APPLY STRING_SPLIT(extras, ',') AS extra1
 WHERE extras <> ''
 ), excl_additions_table AS
(
 SELECT order_id, customer_id, pizza_number, pizza_name, extra_excl AS topping_id, topping_name, topping_addition_subtraction
   FROM exclusions_tab AS etab
   JOIN DannySQLChallenge2..cleaned_pizza_toppings AS cpt ON cpt.topping_id = etab.extra_excl
  UNION
 SELECT order_id, customer_id, pizza_number, pizza_name, extra_excl AS topping_id, topping_name, topping_addition_subtraction
   FROM extras_tab AS et
   JOIN DannySQLChallenge2..cleaned_pizza_toppings AS cpt ON cpt.topping_id = et.extra_excl
), count_table AS
(
SELECT order_id, customer_id, pizza_number, pn.pizza_name, CAST(topping_id AS float) AS topping_id, topping_name, topping_count
  FROM pizza_numbers AS pn
  JOIN pizza_listed AS pl ON pl.pizza_id = pn.pizza_id
 UNION ALL
SELECT *
  FROM excl_additions_table
), total_toppings AS
(
SELECT order_id, customer_id, pizza_number, pizza_name, topping_name, SUM(topping_count) AS topping_count
  FROM count_table
GROUP BY order_id, customer_id, pizza_number, pizza_name, topping_name
HAVING SUM(topping_count) > 0 
), topping_string_name AS
(
SELECT *, CASE WHEN topping_count>1 THEN CONCAT(topping_count,'x',topping_name) ELSE topping_name END AS topping_string
  FROM total_toppings
)

  SELECT order_id, customer_id, CONCAT(pizza_name,':',STRING_AGG(topping_string,', ')) AS pizza_ingredients
    FROM topping_string_name
GROUP BY order_id, customer_id, pizza_name, pizza_number
````

![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/0e29570f-a3b3-4d78-9e80-0d8be44c27b1)

	
### 6.	What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?

````sql

````


## D. Pricing and Ratings

### 1.	If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?

````sql
SELECT CONCAT('$',SUM(
       CASE 
       WHEN pizza_id = 1 
       THEN 12 
       ELSE 10 
       END)) AS revenue
  FROM DannySQLChallenge2..customer_orders AS co
  JOIN DannySQLChallenge2..runner_order_new AS ron ON ron.order_id=co.order_id
 WHERE cancellation = ''
````

![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/57348aa6-27cf-4e91-8acf-af9385effe96)

### 2.	What if there was an additional $1 charge for any pizza extras?
	-	Add cheese is $1 extra

````sql
SELECT CONCAT('$', 
       SUM(
	   CASE 
	   WHEN pizza_id = 1 
	   THEN 12 
	   ELSE 10 
	   END) + 
	   (SELECT SUM(LEN(REPLACE(REPLACE(extras,',',''),' ',''))) AS extras_cost
	      FROM DannySQLChallenge2..customer_orders AS co
	      JOIN DannySQLChallenge2..runner_order_new AS ron ON ron.order_id=co.order_id
	     WHERE cancellation = '')) AS total_revenue
  FROM DannySQLChallenge2..customer_orders AS co
  JOIN DannySQLChallenge2..runner_order_new AS ron ON ron.order_id=co.order_id
 WHERE cancellation = ''
````
![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/64f3e35f-0a78-47f3-97ad-f7ea92a2d4a1)

### 3.	The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.
### 4.	Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?
	-	customer_id
	-	order_id
	-	runner_id
	-	rating
	-	order_time
	-	pickup_time
	-	Time between order and pickup
	-	Delivery duration
	-	Average speed
	-	Total number of pizzas

````sql
SELECT customer_id,
       ron.order_id,
       runner_id,
       order_time,
       pickup_time,
       DATEDIFF(MINUTE, order_time, pickup_time) AS time_taken_to_pickup,
       duration AS delivery_duration,
       ROUND(CAST(ron.distance AS float) / (CAST(ron.duration AS float)/60),2) AS avg_speed,
       COUNT(pizza_id) AS number_pizzas
  FROM DannySQLChallenge2..runner_order_new AS ron
  JOIN DannySQLChallenge2..customer_orders AS co ON co.order_id = ron.order_id
 WHERE cancellation = ''
 GROUP BY customer_id, ron.order_id, runner_id, order_time, pickup_time, duration, ROUND(CAST(ron.distance AS float) / (CAST(ron.duration AS float)/60),2)

````
![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/0f4c870a-66b3-4875-9781-d30802578454)

### 5.	If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?

````sql
WITH revenue_tab
AS
(
SELECT SUM(
	   CASE 
	   WHEN pizza_id = 1 
	   THEN 12 
	   ELSE 10 
	    END) AS revenue
  FROM DannySQLChallenge2..customer_orders AS co
  JOIN DannySQLChallenge2..runner_order_new AS ron ON ron.order_id=co.order_id
 WHERE cancellation = ''
),
delivery_cost_tab
AS
(
SELECT DISTINCT ron.order_id,
       distance,
       CAST(distance AS FLOAT) * 0.3 AS delivery_cost
  FROM DannySQLChallenge2..customer_orders AS co
  JOIN DannySQLChallenge2..runner_order_new AS ron ON ron.order_id=co.order_id
 WHERE cancellation = ''
)

SELECT CONCAT('$',revenue - (SELECT SUM(delivery_cost) FROM delivery_cost_tab)) AS total_profits
  FROM revenue_tab
````
<img width="200" alt="image" src="https://user-images.githubusercontent.com/59825363/197855193-7d46fb48-62a4-4feb-96ac-558efe50efd2.png">


## E. Bonus Questions

If Danny wants to expand his range of pizzas - how would this impact the existing data design? Write an INSERT statement to demonstrate what would happen if a new Supreme pizza with all the toppings was added to the Pizza Runner menu?

````sql
INSERT INTO DannySQLChallenge2..pizza_names (pizza_id,pizza_name)
VALUES (3, 'Supreme');

INSERT INTO DannySQLChallenge2..pizza_recipes (pizza_id, toppings)
VALUES (3,'1,2,3,4,5,6,7,8,9,10,11,12');

SELECT *
  FROM DannySQLChallenge2..pizza_names AS pn
  JOIN DannySQLChallenge2..pizza_recipes AS pt ON pt.pizza_id = pn.pizza_id
````
![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/abd4bf82-7262-496e-aeb1-d7563058dd77)


