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
SELECT runner_id, 
       SUM(100*CASE 
		   WHEN distance = '' 
		   THEN 0
		   ELSE 1 
		   END) / COUNT(*) AS success
  FROM DannySQLChallenge2..runner_order_new
GROUP BY runner_id
````
<img width="200" alt="image" src="https://user-images.githubusercontent.com/59825363/197058543-ca416fc5-154f-4db5-9f0a-c4bc477a09d7.png">


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
<img width="650" alt="image" src="https://user-images.githubusercontent.com/59825363/197059058-139e6b03-99f3-43d8-af07-42d71b92cecf.png">



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
<img width="300" alt="image" src="https://user-images.githubusercontent.com/59825363/197070876-2b0c3162-30b5-477f-83da-89959ead7cc1.png">



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

  SELECT topping_id, topping_name, 
     	 count(*) AS total
    FROM exclusions_list
GROUP BY topping_id, topping_name
ORDER BY topping_id,topping_name
````

<img width="300" alt="image" src="https://user-images.githubusercontent.com/59825363/197070983-d15653fa-fbf7-4d87-ba8b-703ecf0ad8b9.png">


### 4.  Generate an order item for each record in the customers_orders table in the format of one of the following:
  - Meat Lovers
  - Meat Lovers - Exclude Beef
  - Meat Lovers - Extra Bacon
  - Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers

````sql

````

	-	
### 5.	Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients

	-	For example: ""Meat Lovers: 2xBacon, Beef, ... , Salami""

````sql

````

	
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

<img width="250" alt="image" src="https://user-images.githubusercontent.com/59825363/197854447-3bb9247a-7092-450b-8bda-72a621b0e1c1.png">


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
<img width="250" alt="image" src="https://user-images.githubusercontent.com/59825363/197854666-acfb6a7a-be0f-4417-abe1-6496187dbf6f.png">


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
<img width="750" alt="image" src="https://user-images.githubusercontent.com/59825363/197854908-7d58901f-6f0f-44e7-bc1e-7a081814308c.png">



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
<img width="350" alt="image" src="https://user-images.githubusercontent.com/59825363/197855242-a4a13ef8-ce14-481f-b6a1-d5c9de481340.png">

