# 🥑 C. Challenge Payment Question
<p align="center">
<img src="../../img/3.png" align="center" width="400" height="400" >

## 📚 Table of Contents

* [Case Study Questions](#-case-study-questions)
* [My Solution](#-my-solution)
* [Case Study #3 - Foodie-Fi](#-case-study-3---foodie-fi)

## ❓ Case Study Questions

- [The Foodie-Fi team wants you to create a new payments table for the year 2020 that includes amounts paid by each customer in the subscriptions table with the following requirements:](#the-foodie-fi-team-wants-you-to-create-a-new-payments-table-for-the-year-2020-that-includes-amounts-paid-by-each-customer-in-the-subscriptions-table-with-the-following-requirements)
    - monthly payments always occur on the same day of month as the original start_date of any monthly paid plan
    - upgrades from basic to monthly or pro plans are reduced by the current paid amount in that month and start immediately
    - upgrades from pro monthly to pro annual are paid at the end of the current billing period and also starts at the end of the month period
    - once a customer churns they will no longer make payments

## 💡 My Solution

### The Foodie-Fi team wants you to create a new payments table for the year 2020 that includes amounts paid by each customer in the subscriptions table with the following requirements:
- monthly payments always occur on the same day of month as the original start_date of any monthly paid plan
- upgrades from basic to monthly or pro plans are reduced by the current paid amount in that month and start immediately
- upgrades from pro monthly to pro annual are paid at the end of the current billing period and also starts at the end of the month period
- once a customer churns they will no longer make payments

```SQL
CREATE TABLE payment AS (
    WITH payment_date_cte AS (
        SELECT 
            customer_id, -- Select customer ID
            plan_id, -- Select plan ID
            plan_name, -- Select plan name
            GENERATE_SERIES(
                start_date, 
                CASE
                    WHEN plan_id = 4 THEN NULL -- No further payments if plan is churned
                    ELSE COALESCE(
                        LEAD(start_date, 1) OVER(PARTITION BY customer_id ORDER BY start_date ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING), 
                        (SELECT MAX(start_date) FROM foodie_fi.subscriptions) -- End date for the paid plans
                    )
                END, 
                CASE 
                    WHEN plan_id = 3 THEN INTERVAL '1 YEAR' -- Payment interval for annual plans
                    ELSE INTERVAL '1 MONTH' -- Payment interval for monthly plans
                END 
            ) AS payment_date, -- Generate payment dates
            price AS payment -- Select payment amount
        FROM 
            foodie_fi.subscriptions -- Use subscriptions table
            LEFT JOIN foodie_fi.plans USING (plan_id) -- Join with plans table to get prices
        ORDER BY 
            customer_id -- Order by customer ID
    ), 
    payment_reference_cte AS (
        SELECT 
            *, 
            COALESCE(
                LAG(payment_date, 1) OVER(PARTITION BY customer_id ORDER BY payment_date ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING), 
                payment_date - INTERVAL '1 MONTH'
            ) AS last_payment_date, -- Get last payment date
            COALESCE(
                LEAD(payment_date, 1) OVER(PARTITION BY customer_id ORDER BY payment_date ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING), 
                payment_date + INTERVAL '1 MONTH'
            ) AS next_payment_date, -- Get next payment date
            LAG(payment, 1) OVER(PARTITION BY customer_id ORDER BY payment_date ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS last_payment -- Get last payment amount
        FROM 
            payment_date_cte -- Use payment_date_cte to reference dates and payments
        WHERE 
            plan_id != 0 -- Exclude trial plan
    ) 
    SELECT 
        customer_id, -- Select customer ID
        plan_id, -- Select plan ID
        plan_name, -- Select plan name
        payment_date, -- Select payment date
        CASE 
            WHEN EXTRACT(MONTH FROM AGE(payment_date, last_payment_date)) < 1 
                AND EXTRACT(DAY FROM payment_date - last_payment_date) > 0 
            THEN payment - last_payment -- Adjust payment if upgrade occurs in the same month
            ELSE payment -- Use full payment amount otherwise
        END AS amount, -- Calculate payment amount
        RANK() OVER(PARTITION BY customer_id ORDER BY payment_date) AS payment_order -- Rank payments by customer and date
    FROM 
        payment_reference_cte -- Use payment_reference_cte for calculations
    WHERE 
        payment_date != next_payment_date -- Ensure payment date is not duplicated
);

SELECT 
    * 
FROM 
    foodie_fi.payment 
WHERE 
    EXTRACT(YEAR FROM payment_date) = '2020'; -- Filter for payments in the year 2020
```

| customer_id | plan_id | plan_name     | payment_date             | amount | payment_order |
| ----------- | ------- | ------------- | ------------------------ | ------ | ------------- |
| 1           | 1       | basic monthly | 2020-08-08T00:00:00.000Z | 9.90   | 1             |
| 1           | 1       | basic monthly | 2020-09-08T00:00:00.000Z | 9.90   | 2             |
| 1           | 1       | basic monthly | 2020-10-08T00:00:00.000Z | 9.90   | 3             |
| 1           | 1       | basic monthly | 2020-11-08T00:00:00.000Z | 9.90   | 4             |
| 1           | 1       | basic monthly | 2020-12-08T00:00:00.000Z | 9.90   | 5             |
| 2           | 3       | pro annual    | 2020-09-27T00:00:00.000Z | 199.00 | 1             |
| 13          | 1       | basic monthly | 2020-12-22T00:00:00.000Z | 9.90   | 1             |
| 15          | 2       | pro monthly   | 2020-03-24T00:00:00.000Z | 19.90  | 1             |
| 15          | 2       | pro monthly   | 2020-04-24T00:00:00.000Z | 19.90  | 2             |
| 16          | 1       | basic monthly | 2020-06-07T00:00:00.000Z | 9.90   | 1             |
| 16          | 1       | basic monthly | 2020-07-07T00:00:00.000Z | 9.90   | 2             |
| 16          | 1       | basic monthly | 2020-08-07T00:00:00.000Z | 9.90   | 3             |
| 16          | 1       | basic monthly | 2020-09-07T00:00:00.000Z | 9.90   | 4             |
| 16          | 1       | basic monthly | 2020-10-07T00:00:00.000Z | 9.90   | 5             |
| 16          | 3       | pro annual    | 2020-10-21T00:00:00.000Z | 189.10 | 6             |
| 18          | 2       | pro monthly   | 2020-07-13T00:00:00.000Z | 19.90  | 1             |
| 18          | 2       | pro monthly   | 2020-08-13T00:00:00.000Z | 19.90  | 2             |
| 18          | 2       | pro monthly   | 2020-09-13T00:00:00.000Z | 19.90  | 3             |
| 18          | 2       | pro monthly   | 2020-10-13T00:00:00.000Z | 19.90  | 4             |
| 18          | 2       | pro monthly   | 2020-11-13T00:00:00.000Z | 19.90  | 5             |
| 18          | 2       | pro monthly   | 2020-12-13T00:00:00.000Z | 19.90  | 6             |
| 19          | 2       | pro monthly   | 2020-06-29T00:00:00.000Z | 19.90  | 1             |
| 19          | 2       | pro monthly   | 2020-07-29T00:00:00.000Z | 19.90  | 2             |
| 19          | 3       | pro annual    | 2020-08-29T00:00:00.000Z | 199.00 | 3             |

**Attention:**
 - Because presenting the entire output is impractical, the above table only displays the sample rows.
 - To make table `payment` reusable for the [D. Outside The Box Questions](../D%20-%20Outside%20The%20Box%20Questions/README.md), the expression `EXTRACT(YEAR FROM payment_date) = '2020'` is used when querying the table rather than when constructing it.

## 🥑 Case Study #3 - Foodie-Fi

Curious for more? Get your hands on all the sections [here](../README.md).

© 2024 [Shiv Katira](https://github.com/shivkatira)