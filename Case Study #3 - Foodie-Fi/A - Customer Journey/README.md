# 🥑 A. Customer Journey
<p align="center">
<img src="../../img/3.png" align="center" width="400" height="400" >

## 📚 Table of Contents

* [Case Study Questions](#-case-study-questions)
* [My Solution](#-my-solution)
* [Case Study #3 - Foodie-Fi](#-case-study-3---foodie-fi)

## ❓ Case Study Questions

- [Based off the 8 sample customers provided in the sample from the subscriptions table, write a brief description about each customer’s onboarding journey.](#based-off-the-8-sample-customers-provided-in-the-sample-from-the-subscriptions-table-write-a-brief-description-about-each-customers-onboarding-journey)

## 💡 My Solution

### Based off the 8 sample customers provided in the sample from the subscriptions table, write a brief description about each customer’s onboarding journey.

```SQL
SELECT 
    customer_id, -- Identify the customer
    plan_name, -- Name of the subscription plan
    start_date, -- Start date of the subscription
    RANK() OVER(PARTITION BY customer_id ORDER BY start_date) AS customer_journey_stage -- Rank the subscription stages for each customer
FROM 
    foodie_fi.subscriptions -- Access the subscriptions table
LEFT JOIN 
    foodie_fi.plans USING (plan_id) -- Join with plans table to get plan details
ORDER BY 
    customer_id, plan_id, start_date, customer_journey_stage; -- Order results for clarity
```

| customer_id | plan_name     | start_date               | customer_journey_stage |
| ----------- | ------------- | ------------------------ | ---------------------- |
| 1           | trial         | 2020-08-01T00:00:00.000Z | 1                      |
| 1           | basic monthly | 2020-08-08T00:00:00.000Z | 2                      |
| 2           | trial         | 2020-09-20T00:00:00.000Z | 1                      |
| 2           | pro annual    | 2020-09-27T00:00:00.000Z | 2                      |
| 3           | trial         | 2020-01-13T00:00:00.000Z | 1                      |
| 3           | basic monthly | 2020-01-20T00:00:00.000Z | 2                      |
| 4           | trial         | 2020-01-17T00:00:00.000Z | 1                      |
| 4           | basic monthly | 2020-01-24T00:00:00.000Z | 2                      |
| 4           | churn         | 2020-04-21T00:00:00.000Z | 3                      |
| 5           | trial         | 2020-08-03T00:00:00.000Z | 1                      |

**Attention:**
- Because presenting the entire output is impractical, the above table only displays the first 10 rows.

## 🥑 Case Study #3 - Foodie-Fi

Curious for more? Get your hands on all the sections [here](../README.md).

© 2024 [Shiv Katira](https://github.com/shivkatira)