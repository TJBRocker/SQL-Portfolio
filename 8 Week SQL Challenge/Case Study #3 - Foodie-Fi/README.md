# Case Study #3 - Foodie-Fi:

![image](https://user-images.githubusercontent.com/59825363/196279368-adc19fa7-3464-49a5-86cc-cab984fa5045.png)

**_My solution_** can be found [HERE!](https://github.com/TJBRocker/SQL-Portfolio/blob/main/8%20Week%20SQL%20Challenge/Case%20Study%20%233%20-%20Foodie-Fi/Solution.md)

## Task
Subscription based businesses are super popular and Danny realised that there was a large gap in the market - he wanted to create a new streaming service that only had food related content - something like Netflix but with only cooking shows!

Danny created Foodie-Fi with a data driven mindset and wanted to ensure all future investment decisions and new features were decided using data. This case study focuses on using subscription style digital data to answer important business questions.

## Entity Relationship Diagram

![image](https://user-images.githubusercontent.com/59825363/196281717-b07e8778-e39c-47f3-a590-befe8be45cec.png)

## Case Study Questions

###  A.Data Analysis Questions

1.  How many customers has Foodie-Fi ever had?
2.  What is the monthly distribution of `trial` plan `start_date` values for our dataset - use the start of the month as the group by value
3.  What plan `start_date` values occur after the year 2020 for our dataset? Show the breakdown by count of events for each `plan_name`
4.  What is the customer count and percentage of customers who have churned rounded to 1 decimal place?
5.  How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?
6.  What is the number and percentage of customer plans after their initial free trial?
7.  What is the customer count and percentage breakdown of all 5 `plan_name` values at `2020-12-31`?
8.  How many customers have upgraded to an annual plan in 2020?
9.  How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?
10.  Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)
11.  How many customers downgraded from a pro monthly to a basic monthly plan in 2020?

### B. Challenge Payment Question

The Foodie-Fi team wants you to create a new `payments` table for the year 2020 that includes amounts paid by each customer in the `subscriptions` table with the following requirements:

1.  monthly payments always occur on the same day of month as the original `start_date` of any monthly paid plan
2.  upgrades from basic to monthly or pro plans are reduced by the current paid amount in that month and start immediately
3.  upgrades from pro monthly to pro annual are paid at the end of the current billing period and also starts at the end of the month period
4.  once a customer churns they will no longer make payments

### C. Outside The Box Questions

The following are open ended questions which might be asked during a technical interview for this case study - there are no right or wrong answers, but answers that make sense from both a technical and a business perspective make an amazing impression!

1.  How would you calculate the rate of growth for Foodie-Fi?
2.  What key metrics would you recommend Foodie-Fi management to track over time to assess performance of their overall business?
3.  What are some key customer journeys or experiences that you would analyse further to improve customer retention?
4.  If the Foodie-Fi team were to create an exit survey shown to customers who wish to cancel their subscription, what questions would you include in the survey?
5.  What business levers could the Foodie-Fi team use to reduce the customer churn rate? How would you validate the effectiveness of your ideas?
