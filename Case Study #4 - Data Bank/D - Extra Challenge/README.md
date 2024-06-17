# 📊 D. Extra Challenge
<p align="center">
<img src="../../img/4.png" align="center" width="400" height="400" >

## 📚 Table of Contents

* [Case Study Questions](#-case-study-questions)
* [My Solution](#-my-solution)
* [Case Study #4 - Data Bank](#-case-study-4---data-bank)

## ❓ Case Study Questions

- [If the annual interest rate is set at 6% and the Data Bank team wants to reward its customers by increasing their data allocation based off the interest calculated on a daily basis at the end of each day, how much data would be required for this option on a monthly basis?](#if-the-annual-interest-rate-is-set-at-6-and-the-data-bank-team-wants-to-reward-its-customers-by-increasing-their-data-allocation-based-off-the-interest-calculated-on-a-daily-basis-at-the-end-of-each-day-how-much-data-would-be-required-for-this-option-on-a-monthly-basis)

## 💡 My Solution

### If the annual interest rate is set at 6% and the Data Bank team wants to reward its customers by increasing their data allocation based off the interest calculated on a daily basis at the end of each day, how much data would be required for this option on a monthly basis?

```SQL
WITH customer_id_cte AS (
    SELECT 
        DISTINCT customer_id -- Select distinct customer IDs
    FROM 
        data_bank.customer_transactions -- From the customer transactions table
), 
date_cte AS (
    SELECT 
        GENERATE_SERIES(MIN(txn_date), MAX(txn_date), '1 day') AS txn_date -- Generate a series of dates from the minimum to the maximum transaction date with a 1-day interval
    FROM 
        data_bank.customer_transactions -- From the customer transactions table
), 
dummy_transaction_cte AS (
    SELECT 
        *, -- Select all columns
        NULL AS txn_type, -- Add a NULL transaction type for dummy transactions
        0 AS txn_amount -- Add a dummy transaction amount
    FROM 
        customer_id_cte -- Cross join customer IDs
    CROSS JOIN 
        date_cte -- Cross join generated dates
), 
all_transactions_cte AS (
    SELECT 
        customer_id, -- Select customer ID
        txn_date, -- Select transaction date
        data_bank.customer_transactions.txn_type, -- Select transaction type from original transactions
        COALESCE(dummy_transaction_cte.txn_amount + data_bank.customer_transactions.txn_amount, 0) AS txn_amount -- Sum dummy and real transaction amounts, using COALESCE to handle nulls
    FROM 
        dummy_transaction_cte
    LEFT JOIN 
        data_bank.customer_transactions USING (customer_id, txn_date) -- Left join to get actual transactions
    ORDER BY 
        customer_id, txn_date -- Order by customer ID and transaction date
), 
running_balance_cte AS (
    SELECT 
        DISTINCT customer_id, -- Select distinct customer IDs
        txn_date, -- Select transaction date
        TO_CHAR(txn_date, 'MM') AS txn_month_no, -- Extract month number from transaction date
        TO_CHAR(txn_date, 'MONTH') AS txn_month, -- Extract month name from transaction date
        SUM(
            CASE 
                WHEN txn_type = 'deposit' THEN txn_amount -- Add amount for deposit transactions
                ELSE -txn_amount -- Subtract amount for non-deposit transactions
            END
        ) OVER(
            PARTITION BY customer_id -- Partition by customer ID to calculate running balance per customer
            ORDER BY txn_date -- Order by transaction date for running balance calculation
            ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
        ) AS running_balance -- Calculate running balance including the impact of each transaction
    FROM 
        all_transactions_cte -- From the union of dummy and real transactions
), 
daily_interest_cte AS (
    SELECT 
        customer_id, -- Select customer ID
        txn_date, -- Select transaction date
        txn_month_no, -- Select month number
        txn_month, -- Select month name
        COALESCE(
            LAG(running_balance, 1) OVER(
                PARTITION BY customer_id -- Partition by customer ID
                ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
            ) * 0.06 / EXTRACT(
                DAY FROM DATE_TRUNC('year', txn_date) + INTERVAL '1 YEAR' - INTERVAL '1 DAY' - DATE_TRUNC('year', txn_date)
            ), 
            0
        ) AS daily_interest -- Calculate daily interest based on the previous day's running balance
    FROM 
        running_balance_cte -- From the running balance CTE
), 
daily_interest_accrual_cte AS (
    SELECT 
        customer_id, -- Select customer ID
        txn_date, -- Select transaction date
        txn_month_no, -- Select month number
        txn_month, -- Select month name
        SUM(daily_interest) OVER(
            PARTITION BY customer_id -- Partition by customer ID
            ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
        ) AS accrued_interest -- Calculate accrued interest by summing daily interest over time
    FROM 
        daily_interest_cte -- From the daily interest CTE
), 
month_end_balance_with_interest_cte AS (
    SELECT 
        DISTINCT customer_id, -- Select distinct customer IDs
        txn_month_no, -- Select month number
        txn_month, -- Select month name
        LAST_VALUE(running_balance) OVER(
            PARTITION BY customer_id, txn_month_no, txn_month -- Partition by customer ID and month
            ORDER BY running_balance_cte.txn_date -- Order by transaction date
            ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
        ) + LAST_VALUE(accrued_interest) OVER(
            PARTITION BY customer_id, txn_month_no, txn_month -- Partition by customer ID and month
            ORDER BY daily_interest_accrual_cte.txn_date -- Order by transaction date
            ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
        ) AS month_end_balance_with_interest -- Calculate month-end balance including accrued interest
    FROM 
        running_balance_cte -- From the running balance CTE
    LEFT JOIN 
        daily_interest_accrual_cte USING (customer_id, txn_month_no, txn_month) -- Join with daily interest accrual CTE
    ORDER BY 
        customer_id, txn_month_no, txn_month -- Order by customer ID, month number, and month name
) 
SELECT 
    txn_month_no, -- Select month number
    txn_month, -- Select month name
    SUM(month_end_balance_with_interest) AS total_data_allocation -- Sum of month-end balances with interest as the total data allocation
FROM 
    month_end_balance_with_interest_cte -- From the month-end balance with interest CTE
GROUP BY 
    txn_month_no, -- Group by month number
    txn_month -- Group by month name
ORDER BY 
    txn_month_no, -- Order by month number
    txn_month; -- Order by month name
```

| txn_month_no | txn_month | total_data_allocation |
| ------------ | --------- | --------------------- |
| 01           | JANUARY   | 125822.17528767131    |
| 02           | FEBRUARY  | -11555.784547945195   |
| 03           | MARCH     | -183315.1023561642    |
| 04           | APRIL     | -240513.27369863016   |

## 📊 Case Study #4 - Data Bank

Curious for more? Get your hands on all the sections [here](../README.md).

© 2024 [Shiv Katira](https://github.com/shivkatira)