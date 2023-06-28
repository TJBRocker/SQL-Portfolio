# Code Challenge

### 1. Data Integrity Checking & Cleanup

- Alphabetically list all of the country codes in the continent_map table that appear more than once. Display any values where country_code is null as country_code = "FOO" and make this row appear first in the list, even though it should alphabetically sort to the middle. Provide the results of this query as your answer.

````sql

WITH CTE AS(
SELECT COALESCE(country_code, 'FOO') AS country_code, COUNT(*) AS country_count
  FROM Braintree..continent_map
 GROUP BY country_code
)

SELECT *
  FROM CTE
 WHERE country_count > 1

````

Or

````sql

   SELECT COALESCE(country_code, 'FOO') AS country_code, COUNT(*) AS country_count
     FROM Braintree..continent_map
 GROUP BY country_code
   HAVING COUNT(*) > 1

````

<img width="250" alt="image" src="https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/abe3e50b-9a55-4108-9d19-52d0f5a35455">


- For all countries that have multiple rows in the continent_map table, delete all multiple records leaving only the 1 record per country. The record that you keep should be the first one when sorted by the continent_code alphabetically ascending. Provide the query/ies and explanation of step(s) that you follow to delete these records.


For this you would typically use ```` DELETE FROM [table] WHERE [condition]```` but for my approach, I wanted to keep the structural integrity of the database so I created a new table
````sql

SELECT country_code, continent_code
  INTO Braintree..new_continent_map
FROM(

  SELECT COALESCE(country_code, 'FOO') AS country_code, 
	     continent_code,
		 ROW_NUMBER() OVER(PARTITION BY country_code ORDER BY country_code, continent_code) AS ID
    FROM Braintree..continent_map
) AS temp
WHERE ID = 1

````

and as you can see there are no countries with more than one ID:

````sql

SELECT country_code, COUNT(*)  AS country_count
  FROM Braintree..new_continent_map
GROUP BY country_code
HAVING COUNT(*) > 1

````
<img width="250" alt="image" src="https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/e7d62ef9-7afa-4cac-a289-7ee2c2e783f6">


### 2. List the countries ranked 10-12 in each continent by the percent of year-over-year growth descending from 2011 to 2012.

  The percent of growth should be calculated as: ((2012 gdp - 2011 gdp) / 2011 gdp)

  The list should include the columns:

- rank
- continent_name
- country_code
- country_name
- growth_percent

````sql

WITH CTE AS(
SELECT ct.country_code, cs.country_name, cts.continent_code, cts.continent_name,
	   SUM(CASE WHEN year = '2011' THEN gdp_per_capita
			ELSE 0 END) AS gdp_2011,
	   SUM(CASE WHEN year = '2012' THEN gdp_per_capita
			ELSE 0 END) AS gdp_2012
  FROM Braintree..per_capita AS ct
  JOIN Braintree..countries AS cs ON ct.country_code = cs.country_code
  JOIN Braintree..new_continent_map AS cm ON cm.country_code = ct.country_code
  JOIN Braintree..continents AS cts ON cts.continent_code = cm.continent_code
GROUP BY ct.country_code, cs.country_name, cts.continent_code, cts.continent_name
),
CTE1 AS(
SELECT *, CAST(ROUND((gdp_2012-gdp_2011)/(NULLIF(gdp_2011,0))*100.0,2)AS FLOAT) AS growth_percent
  FROM CTE
),
CTE2 AS(
SELECT *, ROW_NUMBER() OVER(PARTITION BY continent_code ORDER BY growth_percent DESC) AS growth_rank
  FROM CTE1
)
SELECT country_code,country_name, continent_code, continent_name, growth_percent, growth_rank
  FROM CTE2 AS ct
 WHERE growth_rank BETWEEN 10 AND 12

````

<img width="600" alt="image" src="https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/ae104484-b21e-4cae-9058-82206f182009">


### 3. For the year 2012, create a 3 column, 1 row report showing the percent share of gdp_per_capita for the following regions:

(i) Asia, (ii) Europe, (iii) the Rest of the World. Your result should look something like:

| Asia | Europe | Rest of World |
| ----------- | ----------- | ----------- |
| 25.0%|	25.0%	|50.0%  |

````sql

SELECT cts.continent_name, 
	   ROUND(100.0*SUM(gdp_per_capita)/(SELECT SUM(gdp_per_capita) FROM Braintree..per_capita  WHERE year = '2012'),2) AS sum_gdp_per_capita
  FROM Braintree..per_capita AS ct
  JOIN Braintree..countries AS cs ON ct.country_code = cs.country_code
  JOIN Braintree..new_continent_map AS cm ON cm.country_code = ct.country_code
  JOIN Braintree..continents AS cts ON cts.continent_code = cm.continent_code
 WHERE year = '2012'
GROUP BY cts.continent_name

````

Initially tried the above, but then realised it doesn't work as there are some countries in the there which are just regions and not real countries, will show this using left joins:

````sql

   SELECT DISTINCT ct.country_code, country_name
     FROM Braintree..per_capita AS ct
LEFT JOIN Braintree..countries AS cs ON ct.country_code = cs.country_code
LEFT JOIN Braintree..new_continent_map AS cm ON cm.country_code = ct.country_code
LEFT JOIN Braintree..continents AS cts ON cts.continent_code = cm.continent_code
    WHERE continent_name IS NULL

````

<img width="320" alt="image" src="https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/6dca0d26-4c97-4e89-aca8-1244a1712db7">

Can get around this by using 'inner' joins or standard joins. Will create either a CTE or table to use throughout queries

````sql

WITH CTE AS(
SELECT gdp_per_capita, ct.country_code, cs.country_name, CASE WHEN cts.continent_name = 'Asia' THEN cts.continent_name
			WHEN cts.continent_name = 'Europe' THEN cts.continent_name
			ELSE 'Rest of World' END AS continent_group
  FROM Braintree..per_capita AS ct
  JOIN Braintree..countries AS cs ON ct.country_code = cs.country_code
  JOIN Braintree..new_continent_map AS cm ON cm.country_code = ct.country_code
  JOIN Braintree..continents AS cts ON cts.continent_code = cm.continent_code
 WHERE year = '2012'
), CTE2 AS(
SELECT continent_group,
       ROUND(100.0*SUM(gdp_per_capita)/(SELECT SUM(gdp_per_capita) FROM CTE),2) AS sum_gdp_per_capita
  FROM CTE
GROUP BY continent_group)
SELECT SUM(CASE WHEN continent_group = 'Asia' THEN sum_gdp_per_capita ELSE 0 END) AS Asia,
	   SUM(CASE WHEN continent_group = 'Europe' THEN sum_gdp_per_capita ELSE 0 END) AS Europe,
	   SUM(CASE WHEN continent_group = 'Rest of World' THEN sum_gdp_per_capita ELSE 0 END) AS Rest_of_World
  FROM CTE2

````

<img width="250" alt="image" src="https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/825072c7-c54e-42aa-8686-96ee92d14741">


### 4a. What is the count of countries and sum of their related gdp_per_capita values for the year 2007 where the string 'an' (case insensitive) appears anywhere in the country name?

````sql

SELECT COUNT(*) AS country_count, ROUND(SUM(gdp_per_capita),2) AS total_gdp
  FROM Braintree..per_capita AS ct
  JOIN Braintree..countries AS cs ON ct.country_code = cs.country_code
  JOIN Braintree..new_continent_map AS cm ON cm.country_code = ct.country_code
  JOIN Braintree..continents AS cts ON cts.continent_code = cm.continent_code
 WHERE year = '2007' AND LOWER(country_name) LIKE '%an%'

````


<img width="250" alt="image" src="https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/fe70fa29-9fd6-4d82-a1c5-d82f728e6d0e">


### 4b. Repeat question 4a, but this time make the query case sensitive.

````sql

SELECT COUNT(*) AS country_count, ROUND(SUM(gdp_per_capita),2) AS total_gdp
  FROM Braintree..per_capita AS ct
  JOIN Braintree..countries AS cs ON ct.country_code = cs.country_code
  JOIN Braintree..new_continent_map AS cm ON cm.country_code = ct.country_code
  JOIN Braintree..continents AS cts ON cts.continent_code = cm.continent_code
 WHERE year = '2007' AND country_name COLLATE Latin1_General_CS_AS LIKE '%an%'

````


<img width="250" alt="image" src="https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/f0d39da3-beb3-4fd5-8d47-61de6ec15f76">


### 5. Find the sum of gpd_per_capita by year and the count of countries for each year that have non-null gdp_per_capita where (i) the year is before 2012 and (ii) the country has a null gdp_per_capita in 2012. Your result should have the columns:

- year
- country_count
- total

````sql

WITH CTE AS(
SELECT ct.country_code
  FROM Braintree..per_capita AS ct
  JOIN Braintree..countries AS cs ON ct.country_code = cs.country_code
  JOIN Braintree..new_continent_map AS cm ON cm.country_code = ct.country_code
  JOIN Braintree..continents AS cts ON cts.continent_code = cm.continent_code
WHERE year = '2012' AND gdp_per_capita IS NULL
)
SELECT year, COUNT(*) AS country_count, ROUND(SUM(gdp_per_capita),2) AS total
  FROM Braintree..per_capita AS ct
  JOIN Braintree..countries AS cs ON ct.country_code = cs.country_code
  JOIN Braintree..new_continent_map AS cm ON cm.country_code = ct.country_code
  JOIN Braintree..continents AS cts ON cts.continent_code = cm.continent_code
WHERE YEAR < 2012 AND gdp_per_capita IS NOT NULL AND ct.country_code IN (SELECT * FROM CTE)
GROUP BY YEAR

````


<img width="275" alt="image" src="https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/6e8bd696-b7e4-4de0-b24d-ea14d58a6ac0">


### 6. All in a single query, execute all of the steps below and provide the results as your final answer:

#### a. create a single list of all per_capita records for year 2009 that includes columns:

- continent_name
- country_code
- country_name
- gdp_per_capita

````sql

SELECT continent_name, ct.country_code, country_name, gdp_per_capita
  FROM Braintree..per_capita AS ct
  JOIN Braintree..countries AS cs ON ct.country_code = cs.country_code
  JOIN Braintree..new_continent_map AS cm ON cm.country_code = ct.country_code
  JOIN Braintree..continents AS cts ON cts.continent_code = cm.continent_code
 WHERE year = '2009'

````


<img width="400" alt="image" src="https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/31452d7b-e96e-4b2d-b4b1-71d4bfba49e0">


#### b. order this list by:

- continent_name ascending
- characters 2 through 4 (inclusive) of the country_name descending

````sql

SELECT continent_name, ct.country_code, country_name, gdp_per_capita
  FROM Braintree..per_capita AS ct
  JOIN Braintree..countries AS cs ON ct.country_code = cs.country_code
  JOIN Braintree..new_continent_map AS cm ON cm.country_code = ct.country_code
  JOIN Braintree..continents AS cts ON cts.continent_code = cm.continent_code
 WHERE year = '2009'
ORDER BY continent_name ASC, SUBSTRING(country_name, 2,3) DESC

````


<img width="388" alt="image" src="https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/b45c833d-3fdc-40db-8a30-fd6f7e6cde82">


#### c. create a running total of gdp_per_capita by continent_name

````sql

WITH CTE AS(
SELECT continent_name, ct.country_code, country_name, ROUND(gdp_per_capita,0) AS gdp_per_capita,
	   SUM(gdp_per_capita)OVER(PARTITION BY continent_name ORDER BY continent_name, SUBSTRING(country_name, 2,3) DESC) AS running_total
  FROM Braintree..per_capita AS ct
  JOIN Braintree..countries AS cs ON ct.country_code = cs.country_code
  JOIN Braintree..new_continent_map AS cm ON cm.country_code = ct.country_code
  JOIN Braintree..continents AS cts ON cts.continent_code = cm.continent_code
 WHERE year = '2009'
 )

 SELECT continent_name, country_code, country_name, gdp_per_capita, ROUND(running_total,2) AS running_total
   FROM CTE

````


<img width="437" alt="image" src="https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/f71d07bb-3c9f-44fd-9f0f-f7783d2eecaf">


#### d. return only the first record from the ordered list for which each continent's running total of gdp_per_capita meets or exceeds $70,000.00 with the following columns:

- continent_name
- country_code
- country_name
- gdp_per_capita
- running_total

````sql

WITH CTE AS(
SELECT continent_name, ct.country_code, country_name, gdp_per_capita,
	   SUM(gdp_per_capita)OVER(PARTITION BY continent_name ORDER BY continent_name, SUBSTRING(country_name, 2,3) DESC) AS running_total
  FROM Braintree..per_capita AS ct
  JOIN Braintree..countries AS cs ON ct.country_code = cs.country_code
  JOIN Braintree..new_continent_map AS cm ON cm.country_code = ct.country_code
  JOIN Braintree..continents AS cts ON cts.continent_code = cm.continent_code
 WHERE year = '2009'
), CTE2 AS (
SELECT *, ROW_NUMBER()OVER(PARTITION BY continent_name ORDER BY continent_name, SUBSTRING(country_name, 2,3) DESC) AS row_num
  FROM CTE
 WHERE running_total >= 70000
)
SELECT continent_name, country_code, country_name, ROUND(gdp_per_capita,2) AS gdp_per_capita, ROUND(running_total,2) AS running_total
  FROM CTE2
 WHERE row_num = 1

````

<img width="443" alt="image" src="https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/cc911b89-543a-4b0f-be70-0faabadbc14d">

### 7. Find the country with the highest average gdp_per_capita for each continent for all years. Now compare your list to the following data set. Please describe any and all mistakes that you can find with the data set below. Include any code that you use to help detect these mistakes.

|rank|	continent_name|	country_code|	country_name|	avg_gdp_per_capita|
| ----------- | ----------- | ----------- | ----------- | ----------- |
|1	|Africa|	SYC|	Seychelles|	$11,348.66|
|1	|Asia|	KWT|	Kuwait|	$43,192.49|
|1	|Europe|	MCO|	Monaco|	$152,936.10|
|1	|North America|	BMU|	Bermuda|	$83,788.48|
|1	|Oceania|	AUS|	Australia|	$47,070.39|
|1	|South America|	CHL|	Chile	|$10,781.71|

````sql

WITH CTE AS(
 SELECT cm.continent_code, continent_name, ct.country_code, country_name, gdp_per_capita
  FROM Braintree..per_capita AS ct
  JOIN Braintree..countries AS cs ON ct.country_code = cs.country_code
  JOIN Braintree..new_continent_map AS cm ON cm.country_code = ct.country_code
  JOIN Braintree..continents AS cts ON cts.continent_code = cm.continent_code
WHERE gdp_per_capita IS NOT NULL
), CTE2 AS(
SELECT continent_code, continent_name, country_code, country_name, ROUND(AVG(gdp_per_capita),2) AS avg_gdp_per_capita
  FROM CTE
GROUP BY continent_code, continent_name, country_code, country_name
), CTE3 AS(
SELECT *, RANK() OVER(PARTITION BY continent_name ORDER BY avg_gdp_per_capita DESC) AS rank
  FROM CTE2
) 
SELECT *
  FROM CTE3
 WHERE rank = 1

````

<img width="511" alt="image" src="https://github.com/TJBRocker/SQL-Portfolio/assets/59825363/eb4116cc-2e6c-4e9a-b3be-79cb31d3dc80">
