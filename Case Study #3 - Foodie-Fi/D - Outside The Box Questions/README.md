# 🥑 D. Outside The Box Questions
<p align="center">
<img src="../../img/3.png" align="center" width="400" height="400" >

## 📚 Table of Contents

* [Case Study Questions](#-case-study-questions)
* [My Solution](#-my-solution)
* [Case Study #3 - Foodie-Fi](#-case-study-3---foodie-fi)

## ❓ Case Study Questions

1. [How would you calculate the rate of growth for Foodie-Fi?](#q1-how-would-you-calculate-the-rate-of-growth-for-foodie-fi)
2. [What key metrics would you recommend Foodie-Fi management to track over time to assess performance of their overall business?](#q2-what-key-metrics-would-you-recommend-foodie-fi-management-to-track-over-time-to-assess-performance-of-their-overall-business)
3. [What are some key customer journeys or experiences that you would analyse further to improve customer retention?](#q3-what-are-some-key-customer-journeys-or-experiences-that-you-would-analyse-further-to-improve-customer-retention)
4. [If the Foodie-Fi team were to create an exit survey shown to customers who wish to cancel their subscription, what questions would you include in the survey?](#q4-if-the-foodie-fi-team-were-to-create-an-exit-survey-shown-to-customers-who-wish-to-cancel-their-subscription-what-questions-would-you-include-in-the-survey)

## 💡 My Solution

### Q1. How would you calculate the rate of growth for Foodie-Fi?

```SQL
WITH revenue_cte AS (
    SELECT 
        TO_CHAR(payment_date, 'YYYY-MM') AS period, -- Format payment date to year-month period
        TO_CHAR(payment_date, 'MONTH-YYYY') AS month, -- Format payment date to month-year
        SUM(amount) AS revenue -- Calculate total revenue for each period
    FROM 
        foodie_fi.payment -- Use payment table
    WHERE 
        payment_date <= (SELECT MAX(start_date) FROM foodie_fi.subscriptions) -- Filter for payments till the last date of transactions
    GROUP BY 
        period, month -- Group by formatted period and month
    ORDER BY 
        period -- Order by period
) 
SELECT 
    period, -- Select period (year-month)
    month, -- Select month (month-year)
    revenue, -- Select calculated revenue
    ROUND(
        COALESCE(
            revenue * 100.0 / LAG(revenue, 1) OVER(ORDER BY period ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING), 
            200
        ) - 100, 
        2
    ) AS growth_rate -- Calculate growth rate as percentage change from previous period
FROM 
    revenue_cte -- Use revenue_cte for data
ORDER BY 
    period; -- Order results by period
```

| period  | month          | revenue  | growth_rate |
| ------- | -------------- | -------- | ----------- |
| 2020-01 | JANUARY  -2020 | 1272.10  | 100.00      |
| 2020-02 | FEBRUARY -2020 | 2723.10  | 114.06      |
| 2020-03 | MARCH    -2020 | 4104.40  | 50.73       |
| 2020-04 | APRIL    -2020 | 5615.70  | 36.82       |
| 2020-05 | MAY      -2020 | 6848.20  | 21.95       |
| 2020-06 | JUNE     -2020 | 8140.70  | 18.87       |
| 2020-07 | JULY     -2020 | 9583.10  | 17.72       |
| 2020-08 | AUGUST   -2020 | 11244.20 | 17.33       |
| 2020-09 | SEPTEMBER-2020 | 12119.60 | 7.79        |
| 2020-10 | OCTOBER  -2020 | 14050.10 | 15.93       |
| 2020-11 | NOVEMBER -2020 | 12039.20 | -14.31      |
| 2020-12 | DECEMBER -2020 | 12655.70 | 5.12        |
| 2021-01 | JANUARY  -2021 | 13313.60 | 5.20        |
| 2021-02 | FEBRUARY -2021 | 10689.10 | -19.71      |
| 2021-03 | MARCH    -2021 | 9595.00  | -10.24      |
| 2021-04 | APRIL    -2021 | 10142.70 | 5.71        |

**Attention:**
- We are using table `payment` constructed in [C. Challenge Payment Question](../C%20-%20Challenge%20Payment%20Question/README.md).

### Q2. What key metrics would you recommend Foodie-Fi management to track over time to assess performance of their overall business?

```SQL
WITH subscriber_base AS (
    SELECT 
        customer_id, -- Select customer ID
        plan_id, -- Select plan ID
        plan_name, -- Select plan name
        GENERATE_SERIES(
            start_date, -- Generate series starting from the start date
            CASE 
                WHEN plan_id = 4 THEN start_date -- For churned customers, stop at start date
                ELSE COALESCE(
                    LEAD(start_date, 1) OVER(PARTITION BY customer_id ORDER BY start_date), 
                    (SELECT MAX(start_date) FROM foodie_fi.subscriptions)
                ) -- For others, stop at next start date or max start date
            END,
            CASE 
                WHEN plan_id = 3 THEN INTERVAL '1 YEAR' -- For annual plans, use yearly interval
                ELSE INTERVAL '1 MONTH' -- For others, use monthly interval
            END
        ) AS payment_date -- Generated payment dates
    FROM 
        foodie_fi.subscriptions -- Use subscriptions table
        LEFT JOIN foodie_fi.plans USING (plan_id) -- Join with plans table using plan_id
), 
subscriber_base_by_month_by_plan AS (
    SELECT 
        TO_CHAR(payment_date, 'YYYY-MM') AS period, -- Convert payment date to YYYY-MM format
        TO_CHAR(payment_date, 'MONTH-YYYY') AS month, -- Convert payment date to MONTH-YYYY format
        SUM(CASE WHEN plan_id = 0 THEN 1 ELSE 0 END) AS trial_subscriber_base, -- Count trial subscribers
        SUM(CASE WHEN plan_id = 1 THEN 1 ELSE 0 END) AS basic_monthly_subscriber_base, -- Count basic monthly subscribers
        SUM(CASE WHEN plan_id = 2 THEN 1 ELSE 0 END) AS pro_monthly_subscriber_base, -- Count pro monthly subscribers
        SUM(CASE WHEN plan_id = 3 THEN 1 ELSE 0 END) AS pro_annual_subscriber_base, -- Count pro annual subscribers
        SUM(CASE WHEN plan_id = 4 THEN 1 ELSE 0 END) AS churned_subscriber_base -- Count churned subscribers
    FROM 
        subscriber_base -- Use subscriber base CTE
    GROUP BY 
        period, month -- Group by period and month
    ORDER BY 
        period, month -- Order by period and month
), 
newly_added_subscriber_base AS (
    SELECT 
        period, -- Select period
        month, -- Select month
        trial_subscriber_base - COALESCE(LAG(trial_subscriber_base, 1) OVER(ORDER BY period), 0) AS newly_added_trial_subscriber_base, -- Calculate newly added trial subscribers
        basic_monthly_subscriber_base - COALESCE(LAG(basic_monthly_subscriber_base, 1) OVER(ORDER BY period), 0) AS newly_added_basic_monthly_subscriber_base, -- Calculate newly added basic monthly subscribers
        pro_monthly_subscriber_base - COALESCE(LAG(pro_monthly_subscriber_base, 1) OVER(ORDER BY period), 0) AS newly_added_pro_monthly_subscriber_base, -- Calculate newly added pro monthly subscribers
        pro_annual_subscriber_base - COALESCE(LAG(pro_annual_subscriber_base, 1) OVER(ORDER BY period), 0) AS newly_added_pro_annual_subscriber_base, -- Calculate newly added pro annual subscribers
        churned_subscriber_base - COALESCE(LAG(churned_subscriber_base, 1) OVER(ORDER BY period), 0) AS newly_churned_subscriber_base -- Calculate newly churned subscribers
    FROM 
        subscriber_base_by_month_by_plan -- Use subscriber base by month by plan CTE
) 
SELECT 
    period, -- Select period
    month, -- Select month
    trial_subscriber_base, -- Select trial subscriber base
    newly_added_trial_subscriber_base, -- Select newly added trial subscribers
    CASE 
        WHEN trial_subscriber_base = 0 THEN 0 
        ELSE ROUND(newly_added_trial_subscriber_base * 100.0 / trial_subscriber_base, 2) 
    END AS trial_subscriber_base_growth_rate, -- Calculate trial subscriber growth rate
    basic_monthly_subscriber_base, -- Select basic monthly subscriber base
    newly_added_basic_monthly_subscriber_base, -- Select newly added basic monthly subscribers
    CASE 
        WHEN basic_monthly_subscriber_base = 0 THEN 0 
        ELSE ROUND(newly_added_basic_monthly_subscriber_base * 100.0 / basic_monthly_subscriber_base, 2) 
    END AS basic_monthly_subscriber_base_growth_rate, -- Calculate basic monthly subscriber growth rate
    pro_monthly_subscriber_base, -- Select pro monthly subscriber base
    newly_added_pro_monthly_subscriber_base, -- Select newly added pro monthly subscribers
    CASE 
        WHEN pro_monthly_subscriber_base = 0 THEN 0 
        ELSE ROUND(newly_added_pro_monthly_subscriber_base * 100.0 / pro_monthly_subscriber_base, 2) 
    END AS pro_monthly_subscriber_base_growth_rate, -- Calculate pro monthly subscriber growth rate
    pro_annual_subscriber_base, -- Select pro annual subscriber base
    newly_added_pro_annual_subscriber_base, -- Select newly added pro annual subscribers
    CASE 
        WHEN pro_annual_subscriber_base = 0 THEN 0 
        ELSE ROUND(newly_added_pro_annual_subscriber_base * 100.0 / pro_annual_subscriber_base, 2) 
    END AS pro_annual_subscriber_base_growth_rate, -- Calculate pro annual subscriber growth rate
    churned_subscriber_base, -- Select churned subscriber base
    newly_churned_subscriber_base, -- Select newly churned subscribers
    CASE 
        WHEN churned_subscriber_base = 0 THEN 0 
        ELSE ROUND(newly_churned_subscriber_base * 100.0 / churned_subscriber_base, 2) 
    END AS churned_subscriber_base_growth_rate -- Calculate churned subscriber growth rate
FROM 
    subscriber_base_by_month_by_plan -- Use subscriber base by month by plan CTE
    LEFT JOIN newly_added_subscriber_base USING (period, month); -- Join with newly added subscriber base on period and month
```
| period  | month          | trial_subscriber_base | newly_added_trial_subscriber_base | trial_subscriber_base_growth_rate | basic_monthly_subscriber_base | newly_added_basic_monthly_subscriber_base | basic_monthly_subscriber_base_growth_rate | pro_monthly_subscriber_base | newly_added_pro_monthly_subscriber_base | pro_monthly_subscriber_base_growth_rate | pro_annual_subscriber_base | newly_added_pro_annual_subscriber_base | pro_annual_subscriber_base_growth_rate | churned_subscriber_base | newly_churned_subscriber_base | churned_subscriber_base_growth_rate |
| ------- | -------------- | --------------------- | --------------------------------- | --------------------------------- | ----------------------------- | ----------------------------------------- | ----------------------------------------- | --------------------------- | --------------------------------------- | --------------------------------------- | -------------------------- | -------------------------------------- | -------------------------------------- | ----------------------- | ----------------------------- | ----------------------------------- |
| 2020-01 | JANUARY  -2020 | 88                    | 88                                | 100.00                            | 31                            | 31                                        | 100.00                                    | 29                          | 29                                      | 100.00                                  | 2                          | 2                                      | 100.00                                 | 9                       | 9                             | 100.00                              |
| 2020-02 | FEBRUARY -2020 | 68                    | -20                               | -29.41                            | 67                            | 36                                        | 53.73                                     | 57                          | 28                                      | 49.12                                   | 5                          | 3                                      | 60.00                                  | 9                       | 0                             | 0.00                                |
| 2020-03 | MARCH    -2020 | 94                    | 26                                | 27.66                             | 109                           | 42                                        | 38.53                                     | 87                          | 30                                      | 34.48                                   | 7                          | 2                                      | 28.57                                  | 13                      | 4                             | 30.77                               |
| 2020-04 | APRIL    -2020 | 81                    | -13                               | -16.05                            | 137                           | 28                                        | 20.44                                     | 116                         | 29                                      | 25.00                                   | 11                         | 4                                      | 36.36                                  | 18                      | 5                             | 27.78                               |
| 2020-05 | MAY      -2020 | 88                    | 7                                 | 7.95                              | 166                           | 29                                        | 17.47                                     | 147                         | 31                                      | 21.09                                   | 13                         | 2                                      | 15.38                                  | 21                      | 3                             | 14.29                               |
| 2020-06 | JUNE     -2020 | 79                    | -9                                | -11.39                            | 192                           | 26                                        | 13.54                                     | 174                         | 27                                      | 15.52                                   | 16                         | 3                                      | 18.75                                  | 19                      | -2                            | -10.53                              |
| 2020-07 | JULY     -2020 | 89                    | 10                                | 11.24                             | 202                           | 10                                        | 4.95                                      | 200                         | 26                                      | 13.00                                   | 20                         | 4                                      | 20.00                                  | 28                      | 9                             | 32.14                               |
| 2020-08 | AUGUST   -2020 | 88                    | -1                                | -1.14                             | 222                           | 20                                        | 9.01                                      | 244                         | 44                                      | 18.03                                   | 24                         | 4                                      | 16.67                                  | 13                      | -15                           | -115.38                             |
| 2020-09 | SEPTEMBER-2020 | 87                    | -1                                | -1.15                             | 221                           | -1                                        | -0.45                                     | 278                         | 34                                      | 12.23                                   | 25                         | 1                                      | 4.00                                   | 23                      | 10                            | 43.48                               |
| 2020-10 | OCTOBER  -2020 | 79                    | -8                                | -10.13                            | 222                           | 1                                         | 0.45                                      | 304                         | 26                                      | 8.55                                    | 32                         | 7                                      | 21.88                                  | 26                      | 3                             | 11.54                               |
| 2020-11 | NOVEMBER -2020 | 75                    | -4                                | -5.33                             | 234                           | 12                                        | 5.13                                      | 314                         | 10                                      | 3.18                                    | 20                         | -12                                    | -60.00                                 | 32                      | 6                             | 18.75                               |
| 2020-12 | DECEMBER -2020 | 84                    | 9                                 | 10.71                             | 240                           | 6                                         | 2.50                                      | 338                         | 24                                      | 7.10                                    | 20                         | 0                                      | 0.00                                   | 25                      | -7                            | -28.00                              |
| 2021-01 | JANUARY  -2021 | 0                     | -84                               | 0                                 | 217                           | -23                                       | -10.60                                    | 348                         | 10                                      | 2.87                                    | 26                         | 6                                      | 23.08                                  | 19                      | -6                            | -31.58                              |
| 2021-02 | FEBRUARY -2021 | 0                     | 0                                 | 0                                 | 182                           | -35                                       | -19.23                                    | 337                         | -11                                     | -3.26                                   | 22                         | -4                                     | -18.18                                 | 18                      | -1                            | -5.56                               |
| 2021-03 | MARCH    -2021 | 0                     | 0                                 | 0                                 | 154                           | -28                                       | -18.18                                    | 330                         | -7                                      | -2.12                                   | 16                         | -6                                     | -37.50                                 | 21                      | 3                             | 14.29                               |
| 2021-04 | APRIL    -2021 | 0                     | 0                                 | 0                                 | 135                           | -19                                       | -14.07                                    | 329                         | -1                                      | -0.30                                   | 24                         | 8                                      | 33.33                                  | 13                      | -8                            | -61.54                              |


### Q3. What are some key customer journeys or experiences that you would analyse further to improve customer retention?

```SQL
WITH unstacked_journey AS (
    SELECT 
        customer_id, -- Select customer ID
        CASE 
            WHEN plan_id = 0 THEN start_date -- Assign start_date to trial if plan_id is 0
            ELSE NULL 
        END AS trial, -- Define trial start date
        CASE 
            WHEN plan_id = 1 THEN start_date -- Assign start_date to basic_monthly if plan_id is 1
            ELSE NULL 
        END AS basic_monthly, -- Define basic_monthly start date
        CASE 
            WHEN plan_id = 2 THEN start_date -- Assign start_date to pro_monthly if plan_id is 2
            ELSE NULL 
        END AS pro_monthly, -- Define pro_monthly start date
        CASE 
            WHEN plan_id = 3 THEN start_date -- Assign start_date to pro_annual if plan_id is 3
            ELSE NULL 
        END AS pro_annual, -- Define pro_annual start date
        CASE 
            WHEN plan_id = 4 THEN start_date -- Assign start_date to churn if plan_id is 4
            ELSE NULL 
        END AS churn -- Define churn start date
    FROM 
        foodie_fi.subscriptions -- Use subscriptions table
), 
customer_journey AS (
    SELECT 
        DISTINCT customer_id, -- Select distinct customer ID
        MIN(trial) OVER(PARTITION BY customer_id) AS trial, -- Get trial date
        MIN(basic_monthly) OVER(PARTITION BY customer_id) AS basic_monthly, -- Get basic monthly date
        MIN(pro_monthly) OVER(PARTITION BY customer_id) AS pro_monthly, -- Get pro monthly date
        MIN(pro_annual) OVER(PARTITION BY customer_id) AS pro_annual, -- Get pro annual date
        MIN(churn) OVER(PARTITION BY customer_id) AS churn -- Get churn date
    FROM 
        unstacked_journey -- Use unstacked_journey CTE
    ORDER BY 
        customer_id -- Order by customer ID
)
SELECT 
    * 
FROM 
    customer_journey; -- Select from customer_journey CTE
```
| customer_id | trial                    | basic_monthly            | pro_monthly              | pro_annual               | churn                    |
| ----------- | ------------------------ | ------------------------ | ------------------------ | ------------------------ | ------------------------ |
| 1           | 2020-08-01T00:00:00.000Z | 2020-08-08T00:00:00.000Z | null                     | null                     | null                     |
| 2           | 2020-09-20T00:00:00.000Z | null                     | null                     | 2020-09-27T00:00:00.000Z | null                     |
| 3           | 2020-01-13T00:00:00.000Z | 2020-01-20T00:00:00.000Z | null                     | null                     | null                     |
| 4           | 2020-01-17T00:00:00.000Z | 2020-01-24T00:00:00.000Z | null                     | null                     | 2020-04-21T00:00:00.000Z |
| 5           | 2020-08-03T00:00:00.000Z | 2020-08-10T00:00:00.000Z | null                     | null                     | null                     |
| 6           | 2020-12-23T00:00:00.000Z | 2020-12-30T00:00:00.000Z | null                     | null                     | 2021-02-26T00:00:00.000Z |
| 7           | 2020-02-05T00:00:00.000Z | 2020-02-12T00:00:00.000Z | 2020-05-22T00:00:00.000Z | null                     | null                     |
| 8           | 2020-06-11T00:00:00.000Z | 2020-06-18T00:00:00.000Z | 2020-08-03T00:00:00.000Z | null                     | null                     |
| 9           | 2020-12-07T00:00:00.000Z | null                     | null                     | 2020-12-14T00:00:00.000Z | null                     |
| 10          | 2020-09-19T00:00:00.000Z | null                     | 2020-09-26T00:00:00.000Z | null                     | null                     |

**Attention:**
 - Because presenting the entire output is impractical, the above table only displays the sample rows.
 - The above table gives an overall view of each customer in a compact manner. We can use the same CTE to go further into the customer's journey.

 ```SQL
 SELECT 
    ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM customer_journey), 2) AS pct_customers_downgrading_plan -- Calculate the percentage of customers downgrading their plan
FROM 
    customer_journey -- Use the customer journey table
WHERE 
    trial > basic_monthly -- Condition to check if a customer downgraded from basic monthly to trial
    OR basic_monthly > pro_monthly -- Condition to check if a customer downgraded from pro monthly to basic monthly 
    OR pro_monthly > pro_annual; -- Condition to check if a customer downgraded from pro annual to pro monthly
 ```

| pct_customers_downgrading_plan |
| ------------------------------ |
| 0.00                           |

```SQL
SELECT 
    ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM customer_journey), 2) AS pct_customers_not_starting_journey_with_trial -- Calculate the percentage of customers not starting their journey with a trial
FROM 
    customer_journey -- Use the customer journey table
WHERE 
    trial IS NULL; -- Condition to check if a customer did not start with a trial plan
```

| pct_customers_not_starting_journey_with_trial |
| --------------------------------------------- |
| 0.00                                          |

```SQL
SELECT 
    ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM customer_journey), 2) AS pct_customers_using_only_trial -- Calculate the percentage of customers using only the trial plan
FROM 
    customer_journey -- Use the customer journey table
WHERE 
    trial IS NOT NULL -- Condition to check if a customer started with a trial plan
    AND basic_monthly IS NULL -- Condition to check if a customer did not use the basic monthly plan
    AND pro_monthly IS NULL -- Condition to check if a customer did not use the pro monthly plan
    AND pro_annual IS NULL; -- Condition to check if a customer did not use the pro annual plan
```

| pct_customers_using_only_trial |
| ------------------------------ |
| 9.20                           |

```SQL
SELECT ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM customer_journey), 2) AS pct_customers_using_single_plan -- Calculate the percentage of customers using a single plan
FROM customer_journey -- Use the customer journey table
WHERE 
    (basic_monthly IS NOT NULL AND pro_monthly IS NULL AND pro_annual IS NULL) -- Condition to check if customer is using basic monthly plan only
    OR (basic_monthly IS NULL AND pro_monthly IS NOT NULL AND pro_annual IS NULL) -- Condition to check if customer is using pro monthly plan only
    OR (basic_monthly IS NULL AND pro_monthly IS NULL AND pro_annual IS NOT NULL); -- Condition to check if customer is using pro annual plan only
```

| pct_customers_using_single_plan |
| ------------------------------- |
| 50.80                           |

```SQL
SELECT 
    ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM customer_journey), 2) AS pct_customers_upgrading_in_the_same_calendar_month -- Calculate the percentage of customers upgrading in the same calendar month
FROM 
    customer_journey -- Use the customer journey table
WHERE 
    TO_CHAR(trial, 'YYYY-MM') = TO_CHAR(basic_monthly, 'YYYY-MM') -- Condition to check if the trial plan was upgraded to basic monthly in the same calendar month
    OR TO_CHAR(trial, 'YYYY-MM') = TO_CHAR(pro_monthly, 'YYYY-MM') -- Condition to check if the trial plan was upgraded to pro monthly in the same calendar month
    OR TO_CHAR(trial, 'YYYY-MM') = TO_CHAR(pro_annual, 'YYYY-MM') -- Condition to check if the trial plan was upgraded to pro annual in the same calendar month
    OR TO_CHAR(basic_monthly, 'YYYY-MM') = TO_CHAR(pro_monthly, 'YYYY-MM') -- Condition to check if the basic monthly plan was upgraded to pro monthly in the same calendar month
    OR TO_CHAR(basic_monthly, 'YYYY-MM') = TO_CHAR(pro_annual, 'YYYY-MM') -- Condition to check if the basic monthly plan was upgraded to pro annual in the same calendar month
    OR TO_CHAR(pro_monthly, 'YYYY-MM') = TO_CHAR(pro_annual, 'YYYY-MM'); -- Condition to check if the pro monthly plan was upgraded to pro annual in the same calendar month
```

| pct_customers_upgrading_in_same_the_calendar_month |
| -------------------------------------------------- |
| 72.50                                              |

```SQL
SELECT 
    ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM customer_journey), 2) AS pct_customer_churning_in_the_same_calendar_month -- Calculate the percentage of customers churning in the same calendar month
FROM 
    customer_journey -- Use the customer journey table
WHERE 
    TO_CHAR(trial, 'YYYY-MM') = TO_CHAR(churn, 'YYYY-MM') -- Condition to check if the trial plan was churned in the same calendar month
    OR TO_CHAR(basic_monthly, 'YYYY-MM') = TO_CHAR(churn, 'YYYY-MM') -- Condition to check if the basic monthly plan was churned in the same calendar month
    OR TO_CHAR(pro_monthly, 'YYYY-MM') = TO_CHAR(churn, 'YYYY-MM') -- Condition to check if the pro monthly plan was churned in the same calendar month
    OR TO_CHAR(pro_annual, 'YYYY-MM') = TO_CHAR(churn, 'YYYY-MM'); -- Condition to check if the pro annual plan was churned in the same calendar month
```

| pct_customer_churning_in_the_same_calendar_month |
| ------------------------------------------------ |
| 9.10                                             |

### Q4. If the Foodie-Fi team were to create an exit survey shown to customers who wish to cancel their subscription, what questions would you include in the survey?

We can ask below questions in the exit survey to better understand customer motivation.

**Section 1: Satisfaction and Experience**

1. Did the content meet your expectations?
    - Not at all
    - Slightly
    - Moderately
    - Very much
    - Completely

2. Do you feel that the plan you subscribed to was value for money?
    - Strongly Disagree
    - Disagree
    - Neutral
    - Agree
    - Strongly Agree

3. Is the platform interface user-friendly?
    - Disagree
    - Strongly Disagree
    - Neutral
    - Agree
    - Strongly Agree

4. How would you describe your experience with the platform? (Open-ended)

5. Would you consider rejoining the platform in the future?
    - Strongly Disagree
    - Disagree
    - Neutral
    - Agree
    - Strongly Agree

6. I would consider rejoining the platform only if: (Open-ended)

**Section 2: Recommendations and Ratings**

7. Would you recommend the platform to your friends or family?
    - Yes
    - No

8. How would you rate the content quality?
    - 1 - Poor
    - 2 - Fair
    - 3 - Good
    - 4 - Very Good
    - 5 - Excellent

9. How would you rate the overall platform?
    - 1 - Poor
    - 2 - Fair
    - 3 - Good
    - 4 - Very Good
    - 5 - Excellent

**Section 3: Usage Patterns**

10. Which of the following streaming services do you currently use? (Select all that apply)
    - Netflix
    - Hulu
    - Disney+
    - Amazon Prime Video
    - Apple TV+
    - HBO Max
    - Other (Please specify): _______

11. On average, how many hours do you spend watching digital content per day?
    - Less than 1 hour
    - 1-2 hours
    - 3-4 hours
    - 5-6 hours
    - More than 6 hours

**Section 4: Demographics and Verification**

12. What is your age group?
    - Under 18
    - 18-24
    - 25-34
    - 35-44
    - 45-54
    - 55-64
    - 65 or older

13. What is your gender?
    - Male
    - Female
    - Non-binary/Third gender
    - Prefer not to say

14. How long were you subscribed to the platform?
    - Less than 1 month
    - 1-3 months
    - 4-6 months
    - 7-12 months
    - More than 1 year

15. How often did you use the platform?
    - Daily
    - Several times a week
    - Weekly
    - Several times a month
    - Rarely

**Section 5: Additional Feedback**

16. Do you have any additional comments or suggestions for us? (Open-ended)

## Q5. What business levers could the Foodie-Fi team use to reduce the customer churn rate? How would you validate the effectiveness of your ideas?

**Analysis of Customer Behavior:**

In the analysis of customer behavior, it was observed that no customers downgraded their plans and approximately half of the customers retained a single plan after the trial period. These insights, combined with data from the exit survey, provide a comprehensive understanding of customer experiences with the platform’s interface, content quality, perceived value for money, re-subscription barriers, digital content consumption patterns, and competitive landscape.

**Strategic Interventions:**

Based on these insights, the following strategic interventions can be implemented:

1. Enhancement of Platform Interface:

    - Action: Address gaps in customer expectations regarding the platform interface by benchmarking against competitors and implementing UI/UX improvements.
    - Rationale: An enhanced user interface can increase user satisfaction and reduce churn.

2. Content Quality Improvement:

    - Action: If content quality is identified as an issue, introduce offers to upgrade plans or expand the content library.
    - Rationale: High-quality, diverse content can improve user engagement and retention.

3. Perceived Value Adjustment:

    - Action: For customers perceiving the service as not providing value for money, offer the option to downgrade rather than cancel subscriptions, or introduce targeted promotions.
    - Rationale: Offering flexible pricing options can retain customers who might otherwise leave.

4. Re-engagement Offers:

    - Action: Provide special offers to former subscribers to incentivize them to rejoin the platform.
    - Rationale: Re-engagement strategies can bring back previous customers, reducing overall churn.

5. Validation of Effectiveness:

    - The effectiveness of these strategies can be validated through data analysis using SQL queries, as outlined in the solution to [question 3](#q3-what-are-some-key-customer-journeys-or-experiences-that-you-would-analyse-further-to-improve-customer-retention).

By systematically implementing and evaluating these strategies, Foodie-Fi can effectively reduce customer churn and enhance overall customer satisfaction.

## 🥑 Case Study #3 - Foodie-Fi

Curious for more? Get your hands on all the sections [here](../README.md).

© 2024 [Shiv Katira](https://github.com/shivkatira)