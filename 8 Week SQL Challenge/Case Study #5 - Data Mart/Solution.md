# Case Study #5 - Data Mart

## 1. Data Cleansing Steps

In a single query, perform the following operations and generate a new table in the `data_mart` schema named `cleaned_weekly_sales`
- Convert the `week_date` to a `DATE` format

- Add a `week_number` as the second column for each `week_date` value, for example any value from the 1st of January to 7th of January will be 1, 8th to 14th will be 2 etc

- Add a `month_number` with the calendar month for each `week_date` value as the 3rd column

- Add a calendar_year column as the 4th column containing either 2018, 2019 or 2020 values

- Add a new column called `age_band` after the original `segment` column using the following mapping on the number inside the `segment` value

|segment|age_band|
|----|----|
|1|Young Adults|
|2|Middle Aged|
|3 or 4|Retirees|

- Add a new `demographic` column using the following mapping for the first letter in the `segment` values:

|segment|demographic|
|----|----|
|C|	Couples|
|F	|Families|

- Ensure all `null` string values with an `"unknown"` string value in the original `segment` column as well as the new `age_band` and `demographic` columns

- Generate a `new avg_transaction` column as the `sales` value divided by `transactions` rounded to 2 decimal places for each record

````sql
  SELECT *
    FROM DannySQLChallenge5..weekly_sales
ORDER BY 1
````
<img width="500" alt="image" src="https://user-images.githubusercontent.com/59825363/199274703-7f2c30ea-178a-4c13-83ff-b6edac716fd8.png">


````sql
SELECT week_date, 
       DATEPART(week,week_date) AS week_number,
       DATEPART(month,week_date) AS month_number,
       DATEPART(year,week_date) AS calender_year,
       TRIM(region) AS region,
       platform,
       REPLACE(segment,'null','unknown') AS segment,
       customer_type,
       CASE WHEN RIGHT(segment,1) = '1' THEN 'Young Adults'
            WHEN RIGHT(segment,1) = '2' THEN 'Middle Aged'
            WHEN RIGHT(segment,1) IN ('3','4')  THEN 'Retirees'
            ELSE 'unknown'
            END AS age_band,
       CASE WHEN LEFT(TRIM(segment),1) = 'c' THEN 'Couples'
            WHEN LEFT(TRIM(segment),1) = 'f' THEN 'Families'
            ELSE 'unknown'
            END AS demographic,
       CAST(sales AS FLOAT) AS sales,
       CAST(transactions AS FLOAT) AS transactions,
       ROUND((CAST(sales AS FLOAT))/(CAST(transactions AS FLOAT)),2) AS avg_transaction
  INTO DannySQLChallenge5..cleaned_weekly_sales
  FROM DannySQLChallenge5..weekly_sales
````

````sql
SELECT *
  FROM DannySQLChallenge5..cleaned_weekly_sales
 
````
![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/b091bcc2-0eb2-4ab8-a25e-4019e5ca3303)

## 2. Data Exploration

### 1.  What day of the week is used for each `week_date` value?

````sql
SELECT DISTINCT DATENAME(weekday,week_date) AS day_name
  FROM DannySQLChallenge5..cleaned_weekly_sales
````

![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/f10afc0e-2f29-4e74-a5c5-09ff68c09f5d)



### 2.  What range of week numbers are missing from the dataset?

````sql

   SELECT value as missing_weeks
     FROM generate_series(1,52,1) AS gs
LEFT JOIN (SELECT DISTINCT week_number FROM DannySQLChallenge5..cleaned_weekly_sales) AS wn ON wn.week_number = gs.value
    WHERE week_number is NULL

````

![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/f3423ff8-a82e-4d11-a911-d4e7f3ae3e28)

### 3.  How many total transactions were there for each year in the dataset?

````sql

  SELECT calender_year,
         COUNT(transactions) AS trans_count
    FROM DannySQLChallenge5..cleaned_weekly_sales
GROUP BY calender_year
ORDER BY calender_year

````
![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/26bdbe9f-6e28-4e0f-a703-99b1155795fe)


### 4.  What is the total sales for each region for each month?

````sql

   SELECT region, 
          calender_year,
	  DATENAME(month, week_date) AS month_name, 
	  SUM(sales) AS total_sales
    FROM DannySQLChallenge5..cleaned_weekly_sales
GROUP BY region, calender_year, month_number, DATENAME(month, week_date)
ORDER BY region, calender_year, month_number, DATENAME(month, week_date)

````
![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/339b3615-c2aa-4bfd-ab5b-f917f31eeb26)


### 5.  What is the total count of transactions for each platform

````sql

  SELECT platform, 
         COUNT(transactions) AS total_trans
    FROM DannySQLChallenge5..cleaned_weekly_sales
GROUP BY platform
ORDER BY platform

````
![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/842098c1-b921-4ff8-bf82-73c0af8ff0b1)

### 6.  What is the percentage of sales for Retail vs Shopify for each month?

````sql

WITH cte AS(
  SELECT calender_year,
         month_number,
         DATENAME(month,week_date) AS month_name,
         SUM(CASE WHEN platform = 'Shopify' THEN sales ELSE 0 END) AS shopify_sales,
         SUM(CASE WHEN platform = 'retail' THEN sales ELSE 0 END) AS retail_sales,
         SUM(sales) AS total_sales
    FROM DannySQLChallenge5..cleaned_weekly_sales
GROUP BY calender_year,month_number,DATENAME(month,week_date)
)

  SELECT calender_year,
         month_name,
         ROUND(100.0*shopify_sales/total_sales,2) AS shopify_percent,
         ROUND(100.0*retail_sales/total_sales,2) As retail_percent
    FROM cte
ORDER BY calender_year, month_number, month_name

````
![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/b3ba5f8e-2478-4a80-8c21-e0f5974f6241)

### 7.  What is the percentage of sales by demographic for each year in the dataset?

````sql
WITH cte AS(
  SELECT calender_year,
	 SUM(CASE WHEN demographic = 'couples' THEN sales ELSE 0 END) AS couples_sales,
	 SUM(CASE WHEN demographic = 'families' THEN sales ELSE 0 END) AS families_sales,
	 SUM(CASE WHEN demographic = 'unknown' THEN sales ELSE 0 END) AS unknown_sales,
	 SUM(sales) AS total_sales
    FROM DannySQLChallenge5..cleaned_weekly_sales
GROUP BY calender_year
)

  SELECT calender_year,
         ROUND(100.0*couples_sales/total_sales,2) AS couples_percent,
	 ROUND(100.0*families_sales/total_sales,2) AS families_percent,
	 ROUND(100.0*unknown_sales/total_sales,2) AS unknown_percent
    FROM cte
ORDER BY calender_year
````
![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/67259938-fc69-4eda-b27e-120a7c655c38)

### 8.  Which `age_band` and `demographic` values contribute the most to Retail sales?

````sql
SELECT age_band,
       demographic,
       SUM(sales) AS total_sales,
       ROUND(100.0*SUM(sales)/(SELECT SUM(sales) FROM DannySQLChallenge5..cleaned_weekly_sales),2) AS sales_percent
  FROM DannySQLChallenge5..cleaned_weekly_sales
 WHERE platform = 'retail'
GROUP BY age_band, demographic
ORDER BY SUM(sales) DESC
````
![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/3bb9f995-76a4-417f-9c4b-b0b1aeb0c981)

### 9.  Can we use the `avg_transaction` column to find the average transaction size for each year for Retail vs Shopify? If not - how would you calculate it instead?

````sql
  SELECT calender_year,
	 platform,
	 ROUND(AVG(avg_transaction),2) AS original_calc,
	 ROUND(SUM(sales)/SUM(transactions),2) AS new_calc,
	 ROUND((SUM(sales)/SUM(transactions)-AVG(avg_transaction)),2) AS difference,
	 CONCAT(ROUND(100.0*(SUM(sales)/SUM(transactions)-AVG(avg_transaction))/AVG(avg_transaction),2),'%') AS percentage_difference
    FROM DannySQLChallenge5..cleaned_weekly_sales
GROUP BY calender_year, platform
ORDER BY platform, calender_year
````
![image](https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/15682ed8-077c-4ad0-9b4c-6d3b86415fe7)

## 3. Before & After Analysis

This technique is usually used when we inspect an important event and want to inspect the impact before and after a certain point in time.

Taking the `week_date` value of `2020-06-15` as the baseline week where the Data Mart sustainable packaging changes came into effect.

We would include all `week_date` values for `2020-06-15` as the start of the period **after** the change and the previous `week_date` values would be **before**

Using this analysis approach - answer the following questions:

### 1.  What is the total sales for the 4 weeks before and after `2020-06-15`? What is the growth or reduction rate in actual values and percentage of sales?

````sql
WITH cte
AS
(
 SELECT 
	CASE WHEN week_number BETWEEN 21 AND 24 THEN sales ELSE 0 END AS before_date_sales,
	CASE WHEN week_number BETWEEN 25 AND 28 THEN sales ELSE 0 END AS after_date_sales
  FROM DannySQLChallenge5..cleaned_weekly_sales
 WHERE calender_year = 2020 AND week_number BETWEEN 21 AND 28
)

SELECT SUM(before_date_sales) AS before_event_sales,
       SUM(after_date_sales) AS after_event_sales,
       SUM(after_date_sales) - SUM(before_date_sales) AS difference,
       CONCAT(ROUND(100.0*(SUM(after_date_sales) - SUM(before_date_sales))/SUM(before_date_sales),2),'%') AS percentage_difference
  FROM cte
````
<img width="450" alt="image" src="https://user-images.githubusercontent.com/59825363/199278123-75dd9563-656b-4b3d-90bb-6759d9f4bc03.png">


### 2.  What about the entire 12 weeks before and after?

````sql
WITH cte
AS
(
 SELECT 
       CASE WHEN week_number BETWEEN 13 AND 24 THEN sales ELSE 0 END AS before_date_sales,
       CASE WHEN week_number BETWEEN 25 AND 36 THEN sales ELSE 0 END AS after_date_sales
  FROM DannySQLChallenge5..cleaned_weekly_sales
 WHERE calender_year = 2020 AND week_number BETWEEN 13 AND 36
)

SELECT SUM(before_date_sales) AS before_event_sales,
       SUM(after_date_sales) AS after_event_sales,
       SUM(after_date_sales) - SUM(before_date_sales) AS difference,
       CONCAT(ROUND(100.0*(SUM(after_date_sales) - SUM(before_date_sales))/SUM(before_date_sales),2),'%') AS percentage_difference
  FROM cte
````
<img width="450" alt="image" src="https://user-images.githubusercontent.com/59825363/199278330-d11ad2e4-98cd-4025-b362-c2d9bfe2fb01.png">


### 3.  How do the sale metrics for these 2 periods before and after compare with the previous years in 2018 and 2019?

````sql
WITH cte
AS
(
 SELECT calender_year,
        CASE WHEN week_number BETWEEN 21 AND 24 THEN sales ELSE 0 END AS before_date_sales,
        CASE WHEN week_number BETWEEN 25 AND 28 THEN sales ELSE 0 END AS after_date_sales
  FROM DannySQLChallenge5..cleaned_weekly_sales
 WHERE week_number BETWEEN 21 AND 28
)

  SELECT calender_year,
         SUM(before_date_sales) AS before_event_sales,
         SUM(after_date_sales) AS after_event_sales,
         SUM(after_date_sales) - SUM(before_date_sales) AS difference,
         CONCAT(ROUND(100.0*(SUM(after_date_sales) - SUM(before_date_sales))/SUM(before_date_sales),2),'%') AS percentage_difference
    FROM cte
GROUP BY calender_year
ORDER BY calender_year
````
<img width="500" alt="image" src="https://user-images.githubusercontent.com/59825363/199278648-e1960d36-c250-47b7-aba6-a01254de7cbe.png">


````sql
WITH cte
AS
(
 SELECT calender_year,
        CASE WHEN week_number BETWEEN 13 AND 24 THEN sales ELSE 0 END AS before_date_sales,
        CASE WHEN week_number BETWEEN 25 AND 36 THEN sales ELSE 0 END AS after_date_sales
  FROM DannySQLChallenge5..cleaned_weekly_sales
 WHERE week_number BETWEEN 13 AND 36
)

   SELECT calender_year,
	  SUM(before_date_sales) AS before_event_sales,
	  SUM(after_date_sales) AS after_event_sales,
	  SUM(after_date_sales) - SUM(before_date_sales) AS difference,
	  CONCAT(ROUND(100.0*(SUM(after_date_sales) - SUM(before_date_sales))/SUM(before_date_sales),2),'%') AS percentage_difference
    FROM cte
GROUP BY calender_year
ORDER BY calender_year
````
<img width="500" alt="image" src="https://user-images.githubusercontent.com/59825363/199278803-7c7604f0-7124-491e-8dd7-18d87e82637b.png">


## 4. Bonus Question
Which areas of the business have the highest negative impact in sales metrics performance in 2020 for the 12 week before and after period?

- region
- platform
- age_band
- demographic
- customer_type

Do you have any further recommendations for Danny’s team at Data Mart or any interesting insights based off this analysis?

````sql
WITH cte
AS
(
 SELECT region,
	   CASE WHEN week_number BETWEEN 13 AND 24 THEN sales ELSE 0 END AS before_date_sales,
	   CASE WHEN week_number BETWEEN 25 AND 36 THEN sales ELSE 0 END AS after_date_sales
  FROM DannySQLChallenge5..cleaned_weekly_sales
 WHERE week_number BETWEEN 13 AND 36
)

SELECT region,
	   SUM(before_date_sales) AS before_event_sales,
	   SUM(after_date_sales) AS after_event_sales,
	   SUM(after_date_sales) - SUM(before_date_sales) AS difference,
	   CONCAT(ROUND(100.0*(SUM(after_date_sales) - SUM(before_date_sales))/SUM(before_date_sales),2),'%') AS percentage_difference
  FROM cte
GROUP BY region
ORDER BY region
````

This query is repeated just by replacing whichever field we are interested in. 

Region:

<img width="450" alt="image" src="https://user-images.githubusercontent.com/59825363/199300266-b08b277a-30df-4755-96b8-cdc2ab3bbab2.png">

platform:

<img width="450" alt="image" src="https://user-images.githubusercontent.com/59825363/199300335-7c52f25f-8c9a-4465-8b5a-50af1f3336ae.png">

age_band:

<img width="450" alt="image" src="https://user-images.githubusercontent.com/59825363/199300388-6e390e93-cd3c-48a9-8120-54b4182f6749.png">

demographic:

![image](https://user-images.githubusercontent.com/59825363/199300423-2a5b293c-54b9-4e3e-a6f6-6330487605bb.png)

customer_type:

<img width="450" alt="image" src="https://user-images.githubusercontent.com/59825363/199300450-e33c7497-7101-407d-9c9b-9d5f7fb0b159.png">

