# Case Study #5 Data Mart:

![image](https://user-images.githubusercontent.com/59825363/196279535-0af81f3b-e8fc-45ac-a67b-f4380bd4fa17.png)

## Task

Data Mart is Danny’s latest venture and after running international operations for his online supermarket that specialises in fresh produce - Danny is asking for your support to analyse his sales performance.

In June 2020 - large scale supply changes were made at Data Mart. All Data Mart products now use sustainable packaging methods in every single step from the farm all the way to the customer.

Danny needs your help to quantify the impact of this change on the sales performance for Data Mart and it’s separate business areas.

The key business question he wants you to help him answer are the following:

- What was the quantifiable impact of the changes introduced in June 2020?
- Which platform, region, segment and customer types were the most impacted by this change?
- What can we do about future introduction of similar sustainability updates to the business to minimise impact on sales?


## Entity Relationship Diagram

![image](https://user-images.githubusercontent.com/59825363/196281958-c4fe3c35-e2bf-4306-a3be-7ce27a031a38.png)

## Case Study Questions

### 1. Data Cleansing Steps

In a single query, perform the following operations and generate a new table in the `data_mart` schema named `clean_weekly_sales`:

-  Convert the `week_date` to a `DATE` format
- 
-  Add a `week_number` as the second column for each `week_date` value, for example any value from the 1st of January to 7th of January will be 1, 8th to 14th will be 2 etc

-  Add a `month_number` with the calendar month for each `week_date` value as the 3rd column

-  Add a `calendar_year` column as the 4th column containing either 2018, 2019 or 2020 values

-  Add a new column called `age_band` after the original `segment` column using the following mapping on the number inside the `segment` value

| :-: |
|segment|age_band|  
|------|------|
|1|Young Adults|
|2|Middled Aged|
|3 or 4|Retirees|
