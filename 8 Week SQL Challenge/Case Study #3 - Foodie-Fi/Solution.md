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

### 2.  What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value

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


````sql
SELECT pl.plan_name, 
       COUNT(*) AS plan_count
  FROM DannySQLChallenge3..subscriptions AS sub
  JOIN DannySQLChallenge3..plans AS pl ON sub.plan_id=pl.plan_id
 WHERE DATEPART(year,start_date) > 2020
GROUP BY pl.plan_id, pl.plan_name
````
### 4.  What is the customer count and percentage of customers who have churned rounded to 1 decimal place?

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
### 5.  How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?

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
### 6.  What is the number and percentage of customer plans after their initial free trial?

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
                                   FROM DannySQLChallenge3..subscriptions),2)
                                 AS conversion_percentage
  FROM new_plan
 WHERE plan_id = 0
GROUP BY new_plan_id, new_plan
ORDER BY new_plan_id, new_plan
````
### 7.  What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?

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

### 8.  How many customers have upgraded to an annual plan in 2020?

````sql
SELECT pl.plan_name, 
       COUNT(*) AS plan_count
  FROM DannySQLChallenge3..subscriptions AS sub
  JOIN DannySQLChallenge3..plans AS pl ON sub.plan_id=pl.plan_id
 WHERE DATEPART(year,start_date) = 2020 AND plan_name LIKE '%annual%'
GROUP BY pl.plan_id, pl.plan_name
````

### 9.  How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?

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

### 10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)

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
	   WHEN upgrade_days BETWEEN 301 AND 300 THEN 'k. 301 - 330 days'
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

### 11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?

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
