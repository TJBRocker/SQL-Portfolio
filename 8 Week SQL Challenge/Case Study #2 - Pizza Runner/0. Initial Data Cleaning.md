# Case Study #2 - Pizza Runner

## Data Cleaning

After an initial look through the data, it was clear that there was some inconsistencies which would need to be addressed before I was able to begin this challenge.

### Customer Orders table

Looking at the `customer_orders` table, I had to get rid of the nulls and NaN in order to address some of the inconsistencies.

|order_id|	customer_id|	pizza_id	|exclusions|	extras|	order_time|
|--|--|--|--|--|--|
1|	101|	1|	 ||	 	2021-01-01 18:05:02
2|	101|	1	 	 |||	2021-01-01 19:00:52
3|	102|	1|	 |	| 	2021-01-02 23:51:23
3|	102|	2|	 |	NaN	|2021-01-02 23:51:23
4|	103|	1|	4|	 	|2021-01-04 13:23:46
4|	103|	1|	4|	 	|2021-01-04 13:23:46
4|	103|	2|	4|	 	|2021-01-04 13:23:46
5|	104|	1|null|	1	|2021-01-08 21:00:29
6|	101|	2|null|	null	|2021-01-08 21:03:13
7|	105|	2|null|	1|	2021-01-08 21:20:29
8|	102|	1|null|	null	|2021-01-09 23:54:33
9|	103|	1|	4	|1, 5	|2021-01-10 11:22:59
10|	104|	1|	null|	null	|2021-01-11 18:34:49
10|	104|	1|	2, 6|	1, 4	|2021-01-11 18:34:49

To do so I used the following scripts:

````sql

UPDATE DannySQLChallenge2..customer_orders
   SET extras = ''
 WHERE extras IS NULL

 UPDATE DannySQLChallenge2..customer_orders
    SET exclusions = ''
  WHERE exclusions LIKE 'null'

UPDATE DannySQLChallenge2..customer_orders
   SET extras = ''
 WHERE extras LIKE 'null'
 ````
 Producing a nice clean table
 
 |order_id|	customer_id|	pizza_id	|exclusions|	extras|	order_time|
|--|--|--|--|--|--|
1|	101|	1|	 ||	 	2021-01-01 18:05:02
2|	101|	1	 	 |||	2021-01-01 19:00:52
3|	102|	1|	 |	| 	2021-01-02 23:51:23
3|	102|	2|	 |		|2021-01-02 23:51:23
4|	103|	1|	4|	 	|2021-01-04 13:23:46
4|	103|	1|	4|	 	|2021-01-04 13:23:46
4|	103|	2|	4|	 	|2021-01-04 13:23:46
5|	104|	1||	1	|2021-01-08 21:00:29
6|	101|	2||		|2021-01-08 21:03:13
7|	105|	2||	1|	2021-01-08 21:20:29
8|	102|	1||		|2021-01-09 23:54:33
9|	103|	1|	4	|1, 5	|2021-01-10 11:22:59
10|	104|	1|	|		|2021-01-11 18:34:49
10|	104|	1|	2, 6|	1, 4	|2021-01-11 18:34:49
 
 
### Runner Orders Table

order_id	runner_id	pickup_time	distance	duration	cancellation
1	1	2021-01-01 18:15:34	20km	32 minutes	 
2	1	2021-01-01 19:10:54	20km	27 minutes	 
3	1	2021-01-03 00:12:37	13.4km	20 mins	NaN
4	2	2021-01-04 13:53:03	23.4	40	NaN
5	3	2021-01-08 21:10:57	10	15	NaN
6	3	null	null	null	Restaurant Cancellation
7	2	2020-01-08 21:30:45	25km	25mins	null
8	2	2020-01-10 00:15:02	23.4 km	15 minute	null
9	2	null	null	null	Customer Cancellation
10	1	2020-01-11 18:50:20	10km	10minutes	null
 
 Similarly to the `customer_orders` table, there were many inconsitencies with this table:
 -  Distances were non uniform, some contained 'km' others didn't, so these needed getting rid of
 -  Durations were also non uniform, any text needed removing
