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
<img width="900" alt="image" src="https://user-images.githubusercontent.com/59825363/199275893-319fa10e-e45b-4b04-98ab-41ab81db9354.png">



## 2. Data Exploration

### 1.  What day of the week is used for each `week_date` value?

````sql
SELECT DISTINCT DATENAME(weekday,week_date) AS day_name
  FROM DannySQLChallenge5..cleaned_weekly_sales
````

<img width="200" alt="image" src="https://user-images.githubusercontent.com/59825363/199276034-db35a89d-980b-430c-8223-06da18a1e4a7.png">


### 2.  What range of week numbers are missing from the dataset?

````sql



````

### 3.  How many total transactions were there for each year in the dataset?

````sql

  SELECT calender_year, 
	     SUM(transactions) AS trans_count
    FROM DannySQLChallenge5..cleaned_weekly_sales
GROUP BY calender_year
ORDER BY calender_year

````
<img width="250" alt="image" src="https://user-images.githubusercontent.com/59825363/199276198-242e7803-fc4d-4a3e-a4db-7c6bef73ad42.png">


### 4.  What is the total sales for each region for each month?

````sql

   SELECT region, 
	     month_number, 
		 DATENAME(month, week_date) AS month_name, 
		 SUM(sales) AS total_sales
    FROM DannySQLChallenge5..cleaned_weekly_sales
GROUP BY region, month_number, DATENAME(month, week_date)
ORDER BY region, month_number, DATENAME(month, week_date)

````
<img width="350" alt="image" src="https://user-images.githubusercontent.com/59825363/199276325-7811639c-c9c8-4902-84c1-9f3076112226.png">



### 5.  What is the total count of transactions for each platform

````sql



````

### 6.  What is the percentage of sales for Retail vs Shopify for each month?

````sql



````

### 7.  What is the percentage of sales by demographic for each year in the dataset?

````sql



````

### 8.  Which `age_band` and `demographic` values contribute the most to Retail sales?

````sql



````

### 9.  Can we use the `avg_transaction` column to find the average transaction size for each year for Retail vs Shopify? If not - how would you calculate it instead?

````sql



````

## 3. Before & After Analysis

This technique is usually used when we inspect an important event and want to inspect the impact before and after a certain point in time.

Taking the `week_date` value of `2020-06-15` as the baseline week where the Data Mart sustainable packaging changes came into effect.

We would include all `week_date` values for `2020-06-15` as the start of the period **after** the change and the previous `week_date` values would be **before**

Using this analysis approach - answer the following questions:

1.  What is the total sales for the 4 weeks before and after `2020-06-15`? What is the growth or reduction rate in actual values and percentage of sales?
2.  What about the entire 12 weeks before and after?
3.  How do the sale metrics for these 2 periods before and after compare with the previous years in 2018 and 2019?

## 4. Bonus Question
Which areas of the business have the highest negative impact in sales metrics performance in 2020 for the 12 week before and after period?

- region
- platform
- age_band
- demographic
- customer_type

Do you have any further recommendations for Danny’s team at Data Mart or any interesting insights based off this analysis?
