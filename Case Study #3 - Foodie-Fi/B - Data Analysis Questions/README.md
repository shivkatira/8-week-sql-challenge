# 🥑 B. Data Analysis Questions
<p align="center">
<img src="../../img/3.png" align="center" width="400" height="400" >

## 📚 Table of Contents

* [Case Study Questions](#-case-study-questions)
* [My Solution](#-my-solution)
* [Case Study #3 - Foodie-Fi](#-case-study-3---foodie-fi)

## ❓ Case Study Questions

1. [How many customers has Foodie-Fi ever had?](#q1-how-many-customers-has-foodie-fi-ever-had)
2. [What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value](#q2-what-is-the-monthly-distribution-of-trial-plan-start_date-values-for-our-dataset---use-the-start-of-the-month-as-the-group-by-value)
3. [What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name](#q3-what-plan-start_date-values-occur-after-the-year-2020-for-our-dataset-show-the-breakdown-by-count-of-events-for-each-plan_name)
4. [What is the customer count and percentage of customers who have churned rounded to 1 decimal place?](#q4-what-is-the-customer-count-and-percentage-of-customers-who-have-churned-rounded-to-1-decimal-place)
5. [How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?](#q5-how-many-customers-have-churned-straight-after-their-initial-free-trial---what-percentage-is-this-rounded-to-the-nearest-whole-number)
6. [What is the number and percentage of customer plans after their initial free trial?](#q6-what-is-the-number-and-percentage-of-customer-plans-after-their-initial-free-trial)
7. [What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?](#q7-what-is-the-customer-count-and-percentage-breakdown-of-all-5-plan_name-values-at-2020-12-31)
8. [How many customers have upgraded to an annual plan in 2020?](#q8-how-many-customers-have-upgraded-to-an-annual-plan-in-2020)
9. [How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?](#q9-how-many-days-on-average-does-it-take-for-a-customer-to-an-annual-plan-from-the-day-they-join-foodie-fi)
10. [Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)](#q10-can-you-further-breakdown-this-average-value-into-30-day-periods-ie-0-30-days-31-60-days-etc)
11. [How many customers downgraded from a pro monthly to a basic monthly plan in 2020?](#q11-how-many-customers-downgraded-from-a-pro-monthly-to-a-basic-monthly-plan-in-2020)

## 💡 My Solution

### Q1. How many customers has Foodie-Fi ever had?

```SQL
SELECT 
    COUNT(DISTINCT customer_id) AS total_customers -- Count the unique customer IDs
FROM 
    foodie_fi.subscriptions; -- Access the subscriptions table to count customers
```

| total_customers |
| --------------- |
| 1000            |

### Q2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value

```SQL
SELECT 
    TO_CHAR(start_date, 'mm') AS month_number, -- Extract month number from start_date
    TO_CHAR(start_date, 'MONTH') AS month_name, -- Extract month name from start_date
    COUNT(start_date) AS new_trial_subscriptions -- Count the number of new trial subscriptions
FROM 
    foodie_fi.subscriptions -- Access the subscriptions table
WHERE 
    plan_id = 0 -- Filter for trial plan
GROUP BY 
    month_number, month_name -- Group by month number and name
ORDER BY 
    month_number; -- Sort by month number
```

| month_number | month_name | new_trial_subscriptions |
| ------------ | ---------- | ----------------------- |
| 01           | JANUARY    | 88                      |
| 02           | FEBRUARY   | 68                      |
| 03           | MARCH      | 94                      |
| 04           | APRIL      | 81                      |
| 05           | MAY        | 88                      |
| 06           | JUNE       | 79                      |
| 07           | JULY       | 89                      |
| 08           | AUGUST     | 88                      |
| 09           | SEPTEMBER  | 87                      |
| 10           | OCTOBER    | 79                      |
| 11           | NOVEMBER   | 75                      |
| 12           | DECEMBER   | 84                      |

### Q3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name

```SQL
SELECT 
    plan_id, -- Select plan ID
    plan_name, -- Select plan name
    COUNT(plan_id) AS subscription_count -- Count the number of subscriptions for each plan
FROM 
    foodie_fi.subscriptions -- Access the subscriptions table
LEFT JOIN 
    foodie_fi.plans USING(plan_id) -- Join with plans table using plan_id
WHERE 
    EXTRACT(YEAR FROM start_date) > 2020 -- Filter for start_date values after the year 2020
GROUP BY 
    plan_id, plan_name -- Group by plan_id and plan_name
ORDER BY 
    plan_id; -- Sort by plan_id
```

| plan_id | plan_name     | subscription_count |
| ------- | ------------- | ------------------ |
| 1       | basic monthly | 8                  |
| 2       | pro monthly   | 60                 |
| 3       | pro annual    | 63                 |
| 4       | churn         | 71                 |

### Q4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?

```SQL
SELECT 
    COUNT(customer_id) AS churned_customer_count, -- Count the number of churned customers
    ROUND( 
        COUNT(customer_id) * 100.0 / (
            SELECT COUNT(DISTINCT customer_id) AS total_subscribers -- Total number of unique subscribers
            FROM foodie_fi.subscriptions
        ), 1 -- Calculate percentage of churned customers rounded to 1 decimal place
    ) AS churned_customer_percentage 
FROM 
    foodie_fi.subscriptions 
WHERE 
    plan_id = 4; -- Filter for customers with plan_id indicating churn
```

| churned_customer_count | churned_customer_percentage |
| ---------------------- | --------------------------- |
| 307                    | 30.7                        |

### Q5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?

```SQL
WITH customer_plan_journey AS (
    -- Rank the plan switches for each customer
    SELECT 
        customer_id, 
        plan_id, 
        start_date, 
        RANK() OVER (PARTITION BY customer_id ORDER BY start_date) AS plan_switch 
    FROM 
        foodie_fi.subscriptions
)

SELECT 
    plan_name, -- Display the name of the plan
    COUNT(DISTINCT customer_id) AS customer_count, -- Count the number of customers
    ROUND(
        COUNT(DISTINCT customer_id) * 100.0 / (
            SELECT COUNT(DISTINCT customer_id) AS total_subscribers -- Total number of unique subscribers
            FROM foodie_fi.subscriptions
        ), 0 -- Calculate percentage of customers rounded to the nearest whole number
    ) AS customer_percentage 
FROM 
    customer_plan_journey 
    LEFT JOIN foodie_fi.plans USING (plan_id) -- Join with plans table to get plan names
WHERE 
    plan_switch = 2 -- Filter for customers who switched plans
    AND plan_id = 4 -- Filter for churned customers
GROUP BY 
    plan_name; -- Group by plan name for aggregation
```

| plan_name | customer_count | customer_percentage |
| --------- | -------------- | ------------------- |
| churn     | 92             | 9                   |

### Q6. What is the number and percentage of customer plans after their initial free trial?

```SQL
WITH customer_plan_journey AS (
    -- Rank the plan switches for each customer
    SELECT 
        customer_id, 
        plan_id, 
        start_date, 
        RANK() OVER (PARTITION BY customer_id ORDER BY start_date) AS plan_switch 
    FROM 
        foodie_fi.subscriptions
)

SELECT 
    plan_id, -- Display the plan ID
    plan_name, -- Display the plan name
    COUNT(DISTINCT customer_id) AS customer_count, -- Count the number of customers
    ROUND(
        COUNT(DISTINCT customer_id) * 100.0 / (
            SELECT COUNT(DISTINCT customer_id) AS total_subscribers -- Total number of unique subscribers
            FROM foodie_fi.subscriptions
        ), 2 -- Calculate percentage of customers rounded to two decimal places
    ) AS customer_percentage 
FROM 
    customer_plan_journey 
    LEFT JOIN foodie_fi.plans USING (plan_id) -- Join with plans table to get plan names
WHERE 
    plan_switch = 2 -- Filter for plans after the initial free trial
    AND plan_id != 4 -- Exclude churned customers
GROUP BY 
    plan_id, plan_name -- Group by plan ID and name for aggregation
ORDER BY 
    plan_id; -- Sort by plan ID for presentation
```

| plan_id | plan_name     | customer_count | customer_percentage |
| ------- | ------------- | -------------- | ------------------- |
| 1       | basic monthly | 546            | 54.60               |
| 2       | pro monthly   | 325            | 32.50               |
| 3       | pro annual    | 37             | 3.70                |

### Q7. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?

```SQL
WITH customer_plan_journey AS (
    -- Rank the plan switches for each customer considering subscriptions until 2020
    SELECT 
        customer_id, 
        plan_id, 
        start_date, 
        RANK() OVER (PARTITION BY customer_id ORDER BY start_date DESC) AS plan_switch 
    FROM 
        foodie_fi.subscriptions
    WHERE 
        start_date <= '2020-12-31' -- Consider subscriptions until the end of 2020
)

SELECT 
    plan_id, -- Display the plan ID
    plan_name, -- Display the plan name
    COUNT(DISTINCT customer_id) AS customer_count, -- Count the number of customers
    ROUND(
        COUNT(DISTINCT customer_id) * 100.0 / (
            SELECT COUNT(DISTINCT customer_id) AS total_subscribers -- Total number of unique subscribers
            FROM foodie_fi.subscriptions
        ), 2 -- Calculate percentage of customers rounded to two decimal places
    ) AS customer_percentage 
FROM 
    customer_plan_journey 
    LEFT JOIN foodie_fi.plans USING (plan_id) -- Join with plans table to get plan names
WHERE 
    plan_switch = 1 -- Filter for the initial plan selection
GROUP BY 
    plan_id, plan_name -- Group by plan ID and name for aggregation
ORDER BY 
    plan_id; -- Sort by plan ID for presentation
```

| plan_id | plan_name     | customer_count | customer_percentage |
| ------- | ------------- | -------------- | ------------------- |
| 0       | trial         | 19             | 1.90                |
| 1       | basic monthly | 224            | 22.40               |
| 2       | pro monthly   | 326            | 32.60               |
| 3       | pro annual    | 195            | 19.50               |
| 4       | churn         | 236            | 23.60               |

### Q8. How many customers have upgraded to an annual plan in 2020?

```SQL
SELECT 
    COUNT(DISTINCT customer_id) AS upgraded_to_annual_count -- Count the number of customers who upgraded to an annual plan in 2020
FROM 
    foodie_fi.subscriptions 
WHERE 
    plan_id = 3 -- Filter for the annual plan
    AND EXTRACT(YEAR FROM start_date) = 2020; -- Consider subscriptions that started in 2020
```

| upgraded_to_annual_count |
| ------------------------ |
| 195                      |

### Q9. How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?

```SQL
WITH trial_plan_subscribers AS (
    SELECT 
        customer_id, 
        start_date AS joining_date -- Selecting the start date as the joining date for trial plan subscribers
    FROM 
        foodie_fi.subscriptions 
    WHERE 
        plan_id = 0 -- Filtering for trial plan subscribers
), 
annual_plan_subscribers AS (
    SELECT 
        customer_id, 
        start_date AS annual_plan_upgrade_date -- Selecting the start date as the upgrade date for annual plan subscribers
    FROM 
        foodie_fi.subscriptions 
    WHERE 
        plan_id = 3 -- Filtering for annual plan subscribers
) 
SELECT 
    ROUND(AVG(annual_plan_upgrade_date - joining_date), 2) AS avg_days_to_upgrade -- Calculate the average days to upgrade
FROM 
    trial_plan_subscribers 
RIGHT JOIN 
    annual_plan_subscribers USING (customer_id); -- Joining the two CTEs on customer_id to find the upgrade duration
```

| avg_days_to_upgrade |
| ------------------- |
| 104.62              |

### Q10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)

```SQL
WITH trial_plan_subscribers AS (
    SELECT 
        customer_id, 
        start_date AS joining_date -- Selecting the start date as the joining date for trial plan subscribers
    FROM 
        foodie_fi.subscriptions 
    WHERE 
        plan_id = 0 -- Filtering for trial plan subscribers
), 
annual_plan_subscribers AS (
    SELECT 
        customer_id, 
        start_date AS annual_plan_upgrade_date -- Selecting the start date as the upgrade date for annual plan subscribers
    FROM 
        foodie_fi.subscriptions 
    WHERE 
        plan_id = 3 -- Filtering for annual plan subscribers
), 
bins AS (
    SELECT 
        WIDTH_BUCKET(annual_plan_upgrade_date - joining_date, 0, 365, 12) AS avg_days_to_upgrade_bin -- Creating bins for 30-day periods
    FROM 
        trial_plan_subscribers 
    RIGHT JOIN 
        annual_plan_subscribers USING (customer_id) -- Joining the two CTEs on customer_id to find the upgrade duration
) 
SELECT 
    CONCAT((avg_days_to_upgrade_bin - 1) * 30, ' - ', avg_days_to_upgrade_bin * 30, ' days') AS upgrade_duration_range, -- Defining the 30-day period ranges
    COUNT(*) AS customer_count -- Counting the number of customers in each bin
FROM 
    bins 
GROUP BY 
    avg_days_to_upgrade_bin -- Grouping by the upgrade duration bins
ORDER BY 
    avg_days_to_upgrade_bin; -- Sorting the results by the bin
```

| upgrade_duration_range | customer_count |
| ---------------------- | -------------- |
| 0 - 30 days            | 49             |
| 30 - 60 days           | 24             |
| 60 - 90 days           | 35             |
| 90 - 120 days          | 35             |
| 120 - 150 days         | 43             |
| 150 - 180 days         | 37             |
| 180 - 210 days         | 24             |
| 210 - 240 days         | 4              |
| 240 - 270 days         | 4              |
| 270 - 300 days         | 1              |
| 300 - 330 days         | 1              |
| 330 - 360 days         | 1              |

### Q11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?

```SQL
WITH pro_monthly_plan_subscribers AS (
    SELECT 
        customer_id, 
        start_date AS pro_monthly_plan_date -- Selecting the start date as the pro monthly plan date
    FROM 
        foodie_fi.subscriptions 
    WHERE 
        plan_id = 2 -- Filtering for pro monthly plan subscribers
), 
basic_monthly_plan_subscribers AS (
    SELECT 
        customer_id, 
        start_date AS basic_monthly_plan_date -- Selecting the start date as the basic monthly plan date
    FROM 
        foodie_fi.subscriptions 
    WHERE 
        plan_id = 1 -- Filtering for basic monthly plan subscribers
) 
SELECT 
    COUNT(*) AS customer_count -- Counting the number of customers who downgraded
FROM 
    pro_monthly_plan_subscribers 
INNER JOIN 
    basic_monthly_plan_subscribers USING (customer_id) -- Joining the two CTEs on customer_id
WHERE 
    pro_monthly_plan_date < basic_monthly_plan_date -- Filtering for customers who downgraded
AND 
    EXTRACT(YEAR FROM basic_monthly_plan_date) = 2020; -- Filtering for downgrades that happened in 2020
```

| customer_count |
| -------------- |
| 0              |


## 🥑 Case Study #3 - Foodie-Fi

Curious for more? Get your hands on all the sections [here](../README.md).

© 2024 [Shiv Katira](https://github.com/shivkatira)