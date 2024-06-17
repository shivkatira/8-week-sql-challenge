# 📊 C. Data Allocation Challenge
<p align="center">
<img src="../../img/4.png" align="center" width="400" height="400" >

## 📚 Table of Contents

* [Case Study Questions](#-case-study-questions)
* [My Solution](#-my-solution)
* [Case Study #4 - Data Bank](#-case-study-4---data-bank)

## ❓ Case Study Questions

- [Running customer balance column that includes the impact each transaction](#Running-customer-balance-column-that-includes-the-impact-each-transaction)
- [Customer balance at the end of each month](#Customer-balance-at-the-end-of-each-month)
- [Minimum, average and maximum values of the running balance for each customer](#Minimum-average-and-maximum-values-of-the-running-balance-for-each-customer)
- [Option 1: data is allocated based off the amount of money at the end of the previous month](#option-1-data-is-allocated-based-off-the-amount-of-money-at-the-end-of-the-previous-month)
- [Option 2: data is allocated on the average amount of money kept in the account in the previous 30 days](#option-2-data-is-allocated-on-the-average-amount-of-money-kept-in-the-account-in-the-previous-30-days)
- [Option 3: data is updated real-time](#option-3-data-is-updated-real-time)

## 💡 My Solution

### Running customer balance column that includes the impact each transaction

```SQL
SELECT 
    *, -- Select all columns from the transactions table
    SUM(
        CASE 
            WHEN txn_type != 'deposit' THEN -txn_amount -- Subtract transaction amount for non-deposit transactions
            ELSE txn_amount -- Add transaction amount for deposit transactions
        END
    ) OVER(
        PARTITION BY customer_id -- Partition by customer ID to calculate running balance per customer
        ORDER BY txn_date -- Order by transaction date for running balance calculation
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_balance -- Calculate running balance including the impact of each transaction
FROM 
    data_bank.customer_transactions -- From the customer transactions table
ORDER BY 
    customer_id, -- Order results by customer ID
    txn_date; -- Order results by transaction date for each customer
```

| customer_id | txn_date                 | txn_type   | txn_amount | running_balance |
| ----------- | ------------------------ | ---------- | ---------- | --------------- |
| 1           | 2020-01-02T00:00:00.000Z | deposit    | 312        | 312             |
| 1           | 2020-03-05T00:00:00.000Z | purchase   | 612        | -300            |
| 1           | 2020-03-17T00:00:00.000Z | deposit    | 324        | 24              |
| 1           | 2020-03-19T00:00:00.000Z | purchase   | 664        | -640            |
| 2           | 2020-01-03T00:00:00.000Z | deposit    | 549        | 549             |
| 2           | 2020-03-24T00:00:00.000Z | deposit    | 61         | 610             |
| 3           | 2020-01-27T00:00:00.000Z | deposit    | 144        | 144             |
| 3           | 2020-02-22T00:00:00.000Z | purchase   | 965        | -821            |
| 3           | 2020-03-05T00:00:00.000Z | withdrawal | 213        | -1034           |
| 3           | 2020-03-19T00:00:00.000Z | withdrawal | 188        | -1222           |
| 3           | 2020-04-12T00:00:00.000Z | deposit    | 493        | -729            |

**Attention:**
- Because presenting the entire output is impractical, the above table only displays the first 10 rows.

### Customer balance at the end of each month

```SQL
WITH customer_id_cte AS (
    SELECT DISTINCT 
        customer_id -- Select distinct customer IDs
    FROM 
        data_bank.customer_transactions
), 
date_cte AS (
    SELECT DISTINCT 
        DATE_TRUNC('month', txn_date) AS txn_date -- Extract distinct month start dates from transaction dates
    FROM 
        data_bank.customer_transactions
), 
dummy_transaction_cte AS (
    SELECT 
        *, 
        NULL AS txn_type, -- Add a dummy transaction type
        0 AS txn_amount -- Set dummy transaction amount to 0
    FROM 
        customer_id_cte 
    CROSS JOIN 
        date_cte -- Cross join to create dummy transactions for each customer and month
), 
all_transactions_cte AS (
    SELECT 
        * 
    FROM 
        dummy_transaction_cte 
    UNION ALL 
    SELECT 
        * 
    FROM 
        data_bank.customer_transactions -- Combine dummy transactions with actual transactions
), 
running_balance_cte AS (
    SELECT 
        customer_id, 
        TO_CHAR(txn_date, 'MM') AS txn_month_no, -- Extract the month number from the transaction date
        TO_CHAR(txn_date, 'MONTH') AS txn_month, -- Extract the month name from the transaction date
        txn_date, 
        SUM(
            CASE 
                WHEN txn_type = 'deposit' THEN txn_amount -- Add transaction amount if type is 'deposit'
                ELSE -txn_amount -- Subtract transaction amount for other types
            END
        ) OVER(
            PARTITION BY customer_id -- Partition by customer ID to calculate running balance
            ORDER BY txn_date -- Order by transaction date for running balance calculation
            ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
        ) AS running_balance -- Calculate running balance for each customer
    FROM 
        all_transactions_cte
) 
SELECT DISTINCT 
    customer_id, 
    txn_month_no, 
    txn_month, 
    LAST_VALUE(running_balance) OVER(
        PARTITION BY customer_id, txn_month -- Partition by customer ID and month to get the last running balance of the month
        ORDER BY txn_date 
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS month_end_balance -- Get the closing balance for each customer at the end of each month
FROM 
    running_balance_cte 
ORDER BY 
    customer_id, 
    txn_month_no; -- Order results by customer ID and month number
```

| customer_id | txn_month_no | txn_month | month_end_balance |
| ----------- | ------------ | --------- | ----------------- |
| 1           | 01           | JANUARY   | 312               |
| 1           | 02           | FEBRUARY  | 312               |
| 1           | 03           | MARCH     | -640              |
| 1           | 04           | APRIL     | -640              |
| 2           | 01           | JANUARY   | 549               |
| 2           | 02           | FEBRUARY  | 549               |
| 2           | 03           | MARCH     | 610               |
| 2           | 04           | APRIL     | 610               |
| 3           | 01           | JANUARY   | 144               |
| 3           | 02           | FEBRUARY  | -821              |
| 3           | 03           | MARCH     | -1222             |
| 3           | 04           | APRIL     | -729              |

**Attention:**
- Because presenting the entire output is impractical, the above table only displays the first 10 rows.

### Minimum, average and maximum values of the running balance for each customer

```SQL
WITH running_balance_cte AS (
    SELECT 
        *, -- Select all columns from the transactions table
        SUM(
            CASE 
                WHEN txn_type != 'deposit' THEN -txn_amount -- Subtract transaction amount for non-deposit transactions
                ELSE txn_amount -- Add transaction amount for deposit transactions
            END
        ) OVER(
            PARTITION BY customer_id -- Partition by customer ID to calculate running balance per customer
            ORDER BY txn_date -- Order by transaction date for running balance calculation
            ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
        ) AS running_balance -- Calculate running balance including the impact of each transaction
    FROM 
        data_bank.customer_transactions -- From the customer transactions table
    ORDER BY 
        customer_id, -- Order results by customer ID
        txn_date -- Order results by transaction date for each customer
)

SELECT 
    *, -- Select all columns from the CTE
    MAX(running_balance) OVER (
        PARTITION BY customer_id -- Partition by customer ID to calculate maximum running balance per customer
        ORDER BY txn_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS max_running_balance, -- Calculate the maximum running balance for each customer

    ROUND(
        AVG(running_balance) OVER (
            PARTITION BY customer_id -- Partition by customer ID to calculate average running balance per customer
            ORDER BY txn_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
        ), 0
    ) AS avg_running_balance, -- Calculate the average running balance for each customer

    MIN(running_balance) OVER (
        PARTITION BY customer_id -- Partition by customer ID to calculate minimum running balance per customer
        ORDER BY txn_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS min_running_balance -- Calculate the minimum running balance for each customer

FROM 
    running_balance_cte; -- From the running balance CTE
```

| customer_id | txn_date                 | txn_type   | txn_amount | running_balance | max_running_balance | avg_running_balance | min_running_balance |
| ----------- | ------------------------ | ---------- | ---------- | --------------- | ------------------- | ------------------- | ------------------- |
| 1           | 2020-01-02T00:00:00.000Z | deposit    | 312        | 312             | 312                 | 312                 | 312                 |
| 1           | 2020-03-05T00:00:00.000Z | purchase   | 612        | -300            | 312                 | 6                   | -300                |
| 1           | 2020-03-17T00:00:00.000Z | deposit    | 324        | 24              | 312                 | 12                  | -300                |
| 1           | 2020-03-19T00:00:00.000Z | purchase   | 664        | -640            | 312                 | -151                | -640                |
| 2           | 2020-01-03T00:00:00.000Z | deposit    | 549        | 549             | 549                 | 549                 | 549                 |
| 2           | 2020-03-24T00:00:00.000Z | deposit    | 61         | 610             | 610                 | 580                 | 549                 |
| 3           | 2020-01-27T00:00:00.000Z | deposit    | 144        | 144             | 144                 | 144                 | 144                 |
| 3           | 2020-02-22T00:00:00.000Z | purchase   | 965        | -821            | 144                 | -339                | -821                |
| 3           | 2020-03-05T00:00:00.000Z | withdrawal | 213        | -1034           | 144                 | -570                | -1034               |
| 3           | 2020-03-19T00:00:00.000Z | withdrawal | 188        | -1222           | 144                 | -733                | -1222               |
| 3           | 2020-04-12T00:00:00.000Z | deposit    | 493        | -729            | 144                 | -732                | -1222               |

**Attention:**
- Because presenting the entire output is impractical, the above table only displays the first 10 rows.

### Option 1: data is allocated based off the amount of money at the end of the previous month

```SQL
WITH customer_id_cte AS (
    SELECT 
        DISTINCT customer_id -- Select distinct customer IDs
    FROM 
        data_bank.customer_transactions -- From the customer transactions table
), 
date_cte AS (
    SELECT 
        DISTINCT date_trunc('month', txn_date) AS txn_date -- Select distinct transaction dates truncated to the month
    FROM 
        data_bank.customer_transactions -- From the customer transactions table
), 
dummy_transaction_cte AS (
    SELECT 
        *, -- Select all columns
        'dummy' AS txn_type, -- Add a dummy transaction type
        0 AS txn_amount -- Add a dummy transaction amount
    FROM 
        customer_id_cte -- Cross join customer IDs
    CROSS JOIN 
        date_cte -- Cross join truncated dates
), 
all_transactions_cte AS (
    SELECT 
        * -- Select all columns from the dummy transactions
    FROM 
        dummy_transaction_cte
    UNION ALL
    SELECT 
        * -- Select all columns from the original transactions table
    FROM 
        data_bank.customer_transactions
), 
running_balance_cte AS (
    SELECT 
        customer_id, -- Select customer ID
        TO_CHAR(txn_date, 'MM') AS txn_month_no, -- Extract month number from transaction date
        TO_CHAR(txn_date, 'MONTH') AS txn_month, -- Extract month name from transaction date
        txn_date, -- Select transaction date
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
month_end_balance_cte AS (
    SELECT 
        DISTINCT customer_id, -- Select distinct customer IDs
        txn_month_no, -- Select month number
        txn_month, -- Select month name
        LAST_VALUE(running_balance) OVER(
            PARTITION BY customer_id, txn_month -- Partition by customer ID and month to get the last value of running balance
            ORDER BY txn_date ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
        ) AS month_end_balance -- Get the last running balance value for each customer at the end of each month
    FROM 
        running_balance_cte -- From the running balance CTE
    ORDER BY 
        customer_id, -- Order results by customer ID
        txn_month_no -- Order results by month number
) 
SELECT 
    txn_month_no, -- Select month number
    txn_month, -- Select month name
    COALESCE(
        LAG(SUM(month_end_balance), 1) OVER(
            ORDER BY txn_month_no, txn_month -- Order by month number and month name
            ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
        ), 0
    ) AS total_data_allocation -- Calculate data allocation based on the previous month's end balance
FROM 
    month_end_balance_cte -- From the month-end balance CTE
GROUP BY 
    txn_month_no, -- Group by month number
    txn_month -- Group by month name
ORDER BY 
    txn_month_no, -- Order results by month number
    txn_month; -- Order results by month name
```

| txn_month_no | txn_month | total_data_allocation |
| ------------ | --------- | --------------------- |
| 01           | JANUARY   | 0                     |
| 02           | FEBRUARY  | 121259                |
| 03           | MARCH     | -15366                |
| 04           | APRIL     | -184590               |

### Option 2: data is allocated on the average amount of money kept in the account in the previous 30 days

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
last_30_days_avg_balance_cte AS (
    SELECT 
        customer_id, -- Select customer ID
        txn_date, -- Select transaction date
        TO_CHAR(txn_date, 'MM') AS txn_month_no, -- Extract month number from transaction date
        TO_CHAR(txn_date, 'MONTH') AS txn_month, -- Extract month name from transaction date
        ROUND(
            AVG(running_balance) OVER(
                PARTITION BY customer_id -- Partition by customer ID
                ROWS BETWEEN 30 PRECEDING AND CURRENT ROW -- Calculate average running balance over the previous 30 days
            ), 
            2
        ) AS avg_30_days_balance -- Round the average balance to 2 decimal places
    FROM 
        running_balance_cte -- From the running balance CTE
    ORDER BY 
        customer_id, txn_date -- Order by customer ID and transaction date
), 
month_end_balance_cte AS (
    SELECT 
        DISTINCT customer_id, -- Select distinct customer IDs
        txn_month_no, -- Select month number
        txn_month, -- Select month name
        LAST_VALUE(avg_30_days_balance) OVER(
            PARTITION BY customer_id, txn_month -- Partition by customer ID and month
            ORDER BY txn_date -- Order by transaction date
            ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
        ) AS month_end_balance -- Calculate the last value of the 30-day average balance for each month
    FROM 
        last_30_days_avg_balance_cte -- From the last 30 days average balance CTE
    ORDER BY 
        customer_id, txn_month_no, txn_month -- Order by customer ID, month number, and month name
) 
SELECT 
    txn_month_no, -- Select month number
    txn_month, -- Select month name
    SUM(month_end_balance) AS total_data_allocation -- Sum of month-end balances as the total data allocation
FROM 
    month_end_balance_cte -- From the month-end balance CTE
GROUP BY 
    txn_month_no, -- Group by month number
    txn_month -- Group by month name
ORDER BY 
    txn_month_no, -- Order by month number
    txn_month; -- Order by month name
```

| txn_month_no | txn_month | total_data_allocation |
| ------------ | --------- | --------------------- |
| 01           | JANUARY   | 94224.48              |
| 02           | FEBRUARY  | 68486.58              |
| 03           | MARCH     | -92733.40             |
| 04           | APRIL     | -223345.44            |

### Option 3: data is updated real-time

```SQL
WITH customer_id_cte AS (
    SELECT 
        DISTINCT customer_id -- Select distinct customer IDs
    FROM 
        data_bank.customer_transactions -- From the customer transactions table
), 
date_cte AS (
    SELECT 
        DISTINCT date_trunc('month', txn_date) AS txn_date -- Select distinct transaction dates truncated to the month
    FROM 
        data_bank.customer_transactions -- From the customer transactions table
), 
dummy_transaction_cte AS (
    SELECT 
        *, -- Select all columns
        'dummy' AS txn_type, -- Add a dummy transaction type
        0 AS txn_amount -- Add a dummy transaction amount
    FROM 
        customer_id_cte -- Cross join customer IDs
    CROSS JOIN 
        date_cte -- Cross join truncated dates
), 
all_transactions_cte AS (
    SELECT 
        * -- Select all columns from the dummy transactions
    FROM 
        dummy_transaction_cte
    UNION ALL
    SELECT 
        * -- Select all columns from the original transactions table
    FROM 
        data_bank.customer_transactions
), 
running_balance_cte AS (
    SELECT 
        customer_id, -- Select customer ID
        TO_CHAR(txn_date, 'MM') AS txn_month_no, -- Extract month number from transaction date
        TO_CHAR(txn_date, 'MONTH') AS txn_month, -- Extract month name from transaction date
        txn_date, -- Select transaction date
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
month_end_balance_cte AS (
    SELECT 
        DISTINCT customer_id, -- Select distinct customer IDs
        txn_month_no, -- Select month number
        txn_month, -- Select month name
        LAST_VALUE(running_balance) OVER(
            PARTITION BY customer_id, txn_month -- Partition by customer ID and month to get the last value of running balance
            ORDER BY txn_date ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
        ) AS month_end_balance -- Get the last running balance value for each customer at the end of each month
    FROM 
        running_balance_cte -- From the running balance CTE
    ORDER BY 
        customer_id, -- Order results by customer ID
        txn_month_no -- Order results by month number
) 

SELECT 
    txn_month_no, -- Select month number
    txn_month, -- Select month name
    SUM(month_end_balance) AS total_data_allocation -- Sum the month-end balances to get total data allocation
FROM 
    month_end_balance_cte -- From the month-end balance CTE
GROUP BY 
    txn_month_no, -- Group by month number
    txn_month -- Group by month name
ORDER BY 
    txn_month_no, -- Order results by month number
    txn_month; -- Order results by month name
```

| txn_month_no | txn_month | total_data_allocation |
| ------------ | --------- | --------------------- |
| 01           | JANUARY   | 121259                |
| 02           | FEBRUARY  | -15366                |
| 03           | MARCH     | -184590               |
| 04           | APRIL     | -240531               |


## 📊 Case Study #4 - Data Bank

Curious for more? Get your hands on all the sections [here](../README.md).

© 2024 [Shiv Katira](https://github.com/shivkatira)