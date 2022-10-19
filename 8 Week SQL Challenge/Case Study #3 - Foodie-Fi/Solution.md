# Case Study #3 - Foodie-Fi

## Case Study Questions
This case study is split into an initial data understanding question before diving straight into data analysis questions before finishing with 1 single extension challenge.

## Data Analysis Questions:

Initially I like to start off by viewing the tables, and in this instance joining them together to see what the data looks like:

````sql
SELECT *
  FROM DannySQLChallenge3..plans

SELECT TOP 5 *
  FROM DannySQLChallenge3..subscriptions

SELECT TOP 5 customer_id, sub.plan_id, pl.plan_name, start_date, price
  FROM DannySQLChallenge3..subscriptions AS sub
  JOIN DannySQLChallenge3..plans AS pl ON sub.plan_id=pl.plan_id
````

|plan_id|plan_name|price|
|-------|-------|-------|
|1|	basic monthly|	9.90|
|2|	pro monthly|	19.90|
|3|	pro annual|	199.00|
|4|	churn|	NULL|


|customer_id|plan_id|start_date|
|-------|-------|-------|
|1	|0	|2020-08-01|
|1	|1	|2020-08-08|
|2	|0	|2020-09-20|
|2	|3	|2020-09-27|
|3	|0	|2020-01-13|

|customer_id|plan_id|plan_name|start_date|price|
|-------|-------|-------|-------|-------|
|1|	0| trial|	2020-08-01|	0.00|
|1|	1| basic monthly|	2020-08-08	|9.90|
|2|	0	|trial|	2020-09-20	|0.00|
|2|	3	|pro annual|	2020-09-27	|199.00|
|3|	0	|trial|	2020-01-13|	0.00|


### 1.  How many customers has Foodie-Fi ever had?

Fairly straightforward start requiring me to count the distinct customers from the subscriptions list

````sql
SELECT COUNT(DISTINCT customer_id) AS total_customers
  FROM DannySQLChallenge3..subscriptions
````
|total_customers|
|-------|
|1000|

### 2.  What is the monthly distribution of trial plan `start_date` values for our dataset - use the start of the month as the group by value

Started by extracting the datepart and name from the start date to allow me to group the month values togetherm before then counting the number of distinct customers

````sql
  SELECT DATEPART(month, start_date) AS month_num,
	 DATENAME(month, start_date) AS month,
	 COUNT(DISTINCT customer_id) AS customer_count
    FROM DannySQLChallenge3..subscriptions AS sub
    JOIN DannySQLChallenge3..plans AS pl ON sub.plan_id=pl.plan_id
   WHERE sub.plan_id = 0
GROUP BY DATEPART(month, start_date), DATENAME(month, start_date)
ORDER BY month_num, month
````
|customer_id|plan_id|start_date|
|-------|-------|-------|
|1|	January|88|
|2|	February|68|
|3|	March	|94|
|4|	April	|81|
|5|	May	|88|
|6|	June	|79|
|7|	July	|89|
|8|	August	|88|
|9|	September|87|
|10|	October	|79|
|11|	November|75|
|12|	December|84|


### 3.  What plan `start_date` values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name

Required me to aggregate a `COUNT` based on the `plan_name`, but filtering on the year 2020 using the `DATEPART` function

````sql
SELECT pl.plan_name, 
       COUNT(*) AS plan_count
  FROM DannySQLChallenge3..subscriptions AS sub
  JOIN DannySQLChallenge3..plans AS pl ON sub.plan_id=pl.plan_id
 WHERE DATEPART(year,start_date) > 2020
GROUP BY pl.plan_id, pl.plan_name
````
|plan_name|plan_count|
|-------|-------|
|basic monthly	|8|
|churn	|71|
|pro annual|63|
|pro monthly|60|

### 4.  What is the customer count and percentage of customers who have churned rounded to 1 decimal place?

This question required me to use a subquery in order to produce a percentage of churn agaisnt total count.

````sql
SELECT COUNT(*) AS churn_count,
       ROUND(100.0 * COUNT(*) / 
                            (SELECT COUNT(DISTINCT customer_id) 
                               FROM DannySQLChallenge3..subscriptions),1) 
                               AS churn_percentage
  FROM DannySQLChallenge3..subscriptions AS sub
  JOIN DannySQLChallenge3..plans AS pl ON sub.plan_id=pl.plan_id
 WHERE plan_name = 'Churn'
````

|churn_count|churn_percentage|
|-------|-------|
|307|30.7|

### 5.  How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?

This is now where the challenge started to ramp up in difficulty, requiring me to think before committing to code!

Thought process behind this was:
-	I needed to determine the different steps in customers behaviours 
-	From there I could then filter on the second 
-	I would also have to use a subquery, similar to the above example to determine the percentage

How I did this!
-	I produced a CTE including `ROW_NUMBER` window function paritioned by the `customer_id` ordered by date ascending.
	- By doing this ranks the customers progression through different plans
-	I was then able to filter on the CTE by `plan_name` *churn* and `rank` *2*
	- This is filtering on customers who after their trial then progressed to churn on their account


````sql
WITH churn_table AS
(
SELECT customer_id, 
       sub.plan_id, 
       pl.plan_name, 
       start_date, 
       ROW_NUMBER() OVER(PARTITION BY customer_id ORDER BY start_date) AS rank
  FROM DannySQLChallenge3..subscriptions AS sub
  JOIN DannySQLChallenge3..plans AS pl ON sub.plan_id=pl.plan_id
)

SELECT COUNT(*) AS immediate_churn_count,
       ROUND(100.0 * COUNT(*) 
       / (SELECT COUNT(DISTINCT customer_id) 
            FROM DannySQLChallenge3..subscriptions),0) 
            AS immediate_churn_percentage
  FROM churn_table
 WHERE plan_name = 'churn' AND rank = 2
````
|immediate_churn_count|immediate_churn_percentage|
|-------|-------|
|92|9|

### 6.  What is the number and percentage of customer plans after their initial free trial?

Similar to the question above, this involved me using a window function, in this case `LEAD`. This technique allows the user to choose a number of values ahead in a chosen column, depending on partitioning.

The make up of my soultion is as follows:

-	A CTE containing a `LEAD` window function chooses the subsequent `plan_name` and `plan_id`
-	Using the CTE I
	- Filter on initial 'plan_id' by the trial plan
	- `GROUP BY` the new plan
	- `COUNT` the new plans
	- Determine a percentage of new plan from the overal customer population via a subquery

````sql
WITH new_plan
AS(
SELECT customer_id, 
       sub.plan_id, 
       pl.plan_name, 
       start_date, 
       LEAD(plan_name,1) OVER(PARTITION BY customer_id ORDER BY start_date) AS new_plan,
       LEAD(sub.plan_id,1) OVER(PARTITION BY customer_id ORDER BY start_date) AS new_plan_id
  FROM DannySQLChallenge3..subscriptions AS sub
  JOIN DannySQLChallenge3..plans AS pl ON sub.plan_id=pl.plan_id
)

SELECT new_plan, 
       COUNT(*) AS plan_count,
       ROUND(100.0 * COUNT(*) / (SELECT COUNT(DISTINCT customer_id) 
                                   FROM DannySQLChallenge3..subscriptions),1)
                                 AS conversion_percentage
  FROM new_plan
 WHERE plan_id = 0
GROUP BY new_plan_id, new_plan
ORDER BY new_plan_id, new_plan
````
|new_plan|plan_count|conversion_percentage|
|-------|-------|-------|
|basic monthly|	546|	54.6|
|pro monthly|	325|	32.5|
|pro annual|	37|	3.7|
|churn|	92|	9.2|

### 7.  What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?

Another window function! This question had me stumped for a little while. I realised however, by ranking each plan but this time by date descending, I could determine the current plan for each customer. From there I simplying had to filter on this value and do some counting!

````sql
WITH current_plan
AS(
SELECT customer_id,
       start_date,
       plan_name,
       sub.plan_id,
       RANK() OVER(PARTITION BY customer_id ORDER BY start_date DESC) AS plan_rank
  FROM DannySQLChallenge3..subscriptions AS sub
  JOIN DannySQLChallenge3..plans AS pl ON sub.plan_id=pl.plan_id
 WHERE start_date < ='2020-12-31'
 )

   SELECT plan_name, 
          COUNT(*) AS plan_count,
          CAST(100.0*COUNT(*)/(SELECT COUNT(*) 
                                 FROM current_plan 
		                WHERE plan_rank = 1) 
				AS float) AS percentage
    FROM current_plan
   WHERE plan_rank = 1
GROUP BY plan_id, plan_name
ORDER BY plan_id, plan_name
````
|plan_name|plan_count|percentage|
|-------|-------|-------|
|trial|	19|	1.9|
|basic monthly|	224|	22.4|
|pro monthly|	326|	32.6|
|pro annual|	195|	19.5|
|churn|	236|	23.6|

### 8.  How many customers have upgraded to an annual plan in 2020?

Fairly straightforward filtering and aggregation example, especially in comparison to the last few questions!

````sql
SELECT pl.plan_name, 
       COUNT(*) AS plan_count
  FROM DannySQLChallenge3..subscriptions AS sub
  JOIN DannySQLChallenge3..plans AS pl ON sub.plan_id=pl.plan_id
 WHERE DATEPART(year,start_date) = 2020 AND plan_name LIKE '%annual%'
GROUP BY pl.plan_id, pl.plan_name
````
|plan_name|plan_count|
|-------|-------|
|pro annual|	195|

### 9.  How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?

This solution is broken down as follows:
-	Two CTE's, determine the two plan groups
	-	The first is a table of customers on trial plans and their `start_date`
	-	The second a table of customers on the annual plan and their new `start_date` or the date at which they converted to a annual plan.
-	A inner join of the two CTEs, in order for all customers to be joined who over time period are both a trial user and one who later converted to a annual plan
-	Finally using `DATEDIFF` to calculate the average difference
-	I looked at the question ahead and determined the minimum and maximum days also so I can create the buckets required for that problem.

````sql
WITH trial_period 
AS(
SELECT customer_id, start_date
  FROM DannySQLChallenge3..subscriptions
 WHERE plan_id = 0
),
annual_plans
AS(
SELECT customer_id, start_date AS annual_purch_date
  FROM DannySQLChallenge3..subscriptions
 WHERE plan_id = 3
)

SELECT AVG(DATEDIFF(day,tp.start_date,ap.annual_purch_date)) AS avg_plan_conversion, 
       MIN(DATEDIFF(day,tp.start_date,ap.annual_purch_date)) AS min_conversion, 
       MAX(DATEDIFF(day,tp.start_date,ap.annual_purch_date)) AS max_conversion
  FROM annual_plans AS ap
  JOIN trial_period AS tp on tp.customer_id = ap.customer_id
````
|avg_plan_conversion|min_conversion|max_conversion|
|-------|-------|-------|
|104|	7|	346|

### 10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)

I'm convinced there's an easier way, but I haven't figured one out yet so I created a large `CASE` statement to answer the problem.

````sql
WITH trial_period 
AS(
SELECT customer_id, 
       start_date
  FROM DannySQLChallenge3..subscriptions
 WHERE plan_id = 0
),
annual_plans
AS(
SELECT customer_id, 
       start_date AS annual_purch_date
  FROM DannySQLChallenge3..subscriptions
 WHERE plan_id = 3
),
days_to_annual
AS(
SELECT DATEDIFF(day,tp.start_date,ap.annual_purch_date) AS upgrade_days
  FROM annual_plans AS ap
  JOIN trial_period AS tp on tp.customer_id = ap.customer_id
),
case_statement
AS(
SELECT CASE
	   WHEN upgrade_days < 31 THEN 'a. 0 - 30 days'
	   WHEN upgrade_days BETWEEN 31 AND 60 THEN 'b. 31 - 60 days'
	   WHEN upgrade_days BETWEEN 61 AND 90 THEN 'c. 61 - 90 days'
	   WHEN upgrade_days BETWEEN 91 AND 120 THEN 'd. 91 - 120 days'
	   WHEN upgrade_days BETWEEN 121 AND 150 THEN 'e. 121 - 150 days'
	   WHEN upgrade_days BETWEEN 151 AND 180 THEN 'f. 151 - 180 days'
	   WHEN upgrade_days BETWEEN 181 AND 210 THEN 'g. 181 - 210 days'
	   WHEN upgrade_days BETWEEN 211 AND 240 THEN 'h. 211 - 240 days'
	   WHEN upgrade_days BETWEEN 241 AND 270 THEN 'i. 241 - 270 days'
	   WHEN upgrade_days BETWEEN 271 AND 300 THEN 'j. 271 - 300 days'
	   WHEN upgrade_days BETWEEN 301 AND 330 THEN 'k. 301 - 330 days'
	   WHEN upgrade_days BETWEEN 331 AND 360 THEN 'l. 331 - 360 days'
	   WHEN upgrade_days >= 360 THEN 'm. 360+ days'
	   END AS upgrade_days_breakdown
  FROM days_to_annual
)

SELECT TRIM(SUBSTRING(upgrade_days_breakdown,3,LEN(upgrade_days_breakdown))) AS analysis, 
       COUNT(*) AS count
  FROM case_statement
GROUP BY upgrade_days_breakdown
ORDER BY upgrade_days_breakdown
````

|analysis|count|
|-------|-------|
|0 - 30 days|	49|
|31 - 60 days|	24|
|61 - 90 days|	34|
|91 - 120 days|	35|
|121 - 150 days|	42|
|151 - 180 days|	36|
|181 - 210 days|	26|
|211 - 240 days|	4|
|241 - 270 days|	5|
|271 - 300 days|	1|
|301 - 330 days|	1|
|331 - 360 days|	1|

### 11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?

this problem was similar to Q6 and the I was able to use that intuition to complete.


````sql
WITH plan_analysis
AS(
SELECT customer_id, 
       sub.plan_id, 
       pl.plan_name, 
       start_date, 
       LEAD(sub.plan_id,1) OVER(PARTITION BY customer_id ORDER BY start_date) AS new_plan_id,
       LEAD(plan_name,1) OVER(PARTITION BY customer_id ORDER BY start_date) AS new_plan,
       LEAD(start_date,1) OVER(PARTITION BY customer_id ORDER BY start_date) AS new_start_date
  FROM DannySQLChallenge3..subscriptions AS sub
  JOIN DannySQLChallenge3..plans AS pl ON sub.plan_id=pl.plan_id
)

SELECT COUNT(*) AS number_downgraded
  FROM plan_analysis
 WHERE plan_id = 3 AND new_plan_id = 1 
       AND DATEPART(year, start_date) = 2000 
       AND DATEPART(year, new_start_date) = 2000
````
This was a surprising answer, and at first I was convinced it was incorrect!

|number_downgraded|
|-------|
|0 |
