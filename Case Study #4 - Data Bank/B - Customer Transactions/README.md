# 📊 B. Customer Transactions
<p align="center">
<img src="../../img/4.png" align="center" width="400" height="400" >

## 📚 Table of Contents

* [Case Study Questions](#-case-study-questions)
* [My Solution](#-my-solution)
* [Case Study #4 - Data Bank](#-case-study-4---data-bank)

## ❓ Case Study Questions

1. [What is the unique count and total amount for each transaction type?](#q1-what-is-the-unique-count-and-total-amount-for-each-transaction-type)
2. [What is the average total historical deposit counts and amounts for all customers?](#q2-what-is-the-average-total-historical-deposit-counts-and-amounts-for-all-customers)
3. [For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?](#q3-for-each-month---how-many-data-bank-customers-make-more-than-1-deposit-and-either-1-purchase-or-1-withdrawal-in-a-single-month)
4. [What is the closing balance for each customer at the end of the month?](#q4-what-is-the-closing-balance-for-each-customer-at-the-end-of-the-month)
5. [What is the percentage of customers who increase their closing balance by more than 5%?](#q5-what-is-the-percentage-of-customers-who-increase-their-closing-balance-by-more-than-5)

## 💡 My Solution

### Q1. What is the unique count and total amount for each transaction type?

```SQL
SELECT 
    txn_type, -- Select transaction type
    COUNT(*) AS unique_transaction_count, -- Count the number of unique transactions for each type
    SUM(txn_amount) AS total_transaction_amount -- Calculate the total transaction amount for each type
FROM 
    data_bank.customer_transactions -- Use customer_transactions table to get transaction data
GROUP BY 
    txn_type; -- Group results by transaction type to aggregate data
```

| txn_type   | unique_transaction_count | total_transaction_amount |
| ---------- | ------------------------ | ------------------------ |
| purchase   | 1617                     | 806537                   |
| deposit    | 2671                     | 1359168                  |
| withdrawal | 1580                     | 793003                   |

### Q2. What is the average total historical deposit counts and amounts for all customers?

```SQL
WITH cte AS (
    SELECT 
        customer_id, -- Select customer ID
        COUNT(customer_id) AS deposit_count, -- Count the number of deposits per customer
        SUM(txn_amount) AS total_deposit_amount -- Calculate the total deposit amount per customer
    FROM 
        data_bank.customer_transactions -- Use customer_transactions table to get transaction data
    WHERE 
        txn_type = 'deposit' -- Filter transactions to include only deposits
    GROUP BY 
        customer_id -- Group by customer ID to aggregate deposit data
)
SELECT 
    ROUND(AVG(deposit_count), 0) AS avg_deposit_count, -- Calculate the average deposit count across all customers
    ROUND(AVG(total_deposit_amount), 2) AS avg_deposit_amount -- Calculate the average deposit amount across all customers
FROM 
    cte; -- Use the common table expression to get the aggregated deposit data
```

| avg_deposit_count | avg_deposit_amount |
| ----------------- | ------------------ |
| 5                 | 2718.34            |

### Q3. For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?

```SQL
WITH cte AS (
    SELECT 
        TO_CHAR(txn_date, 'mm') AS month_no, -- Extract the month number from the transaction date
        TO_CHAR(txn_date, 'MONTH') AS txn_month, -- Extract the month name from the transaction date
        customer_id, -- Select customer ID
        SUM(CASE WHEN txn_type = 'deposit' THEN 1 ELSE 0 END) AS deposit_count, -- Count the number of deposits per customer
        SUM(CASE WHEN txn_type = 'withdrawal' THEN 1 ELSE 0 END) AS withdrawal_count, -- Count the number of withdrawals per customer
        SUM(CASE WHEN txn_type = 'purchase' THEN 1 ELSE 0 END) AS purchase_count -- Count the number of purchases per customer
    FROM 
        data_bank.customer_transactions -- Use customer_transactions table to get transaction data
    GROUP BY 
        month_no, txn_month, customer_id -- Group by month number, month name, and customer ID to aggregate data
    ORDER BY 
        month_no -- Order results by month number
)
SELECT 
    txn_month, -- Select the month name
    COUNT(*) AS customer_count -- Count the number of customers meeting the criteria
FROM 
    cte -- Use the common table expression to get the aggregated transaction data
WHERE 
    (deposit_count > 1) -- Filter for customers with more than 1 deposit
    AND 
    (withdrawal_count = 1 OR purchase_count = 1) -- Filter for customers with either 1 withdrawal or 1 purchase
GROUP BY 
    month_no, txn_month; -- Group results by month number and month name
```

| txn_month | customer_count |
| --------- | -------------- |
| JANUARY   | 115            |
| FEBRUARY  | 108            |
| MARCH     | 113            |
| APRIL     | 50             |

### Q4. What is the closing balance for each customer at the end of the month?

```SQL
WITH customer_id_cte AS (
    SELECT DISTINCT customer_id -- Get distinct customer IDs
    FROM data_bank.customer_transactions
), 
date_cte AS (
    SELECT DISTINCT date_trunc('month', txn_date) AS txn_date -- Get distinct transaction dates truncated to the month
    FROM data_bank.customer_transactions
), 
dummy_transaction_cte AS (
    SELECT 
        *, 
        'dummy' AS txn_type, -- Add dummy transaction type
        0 AS txn_amount -- Set transaction amount to 0 for dummy transactions
    FROM 
        customer_id_cte 
        CROSS JOIN date_cte -- Cross join to get all combinations of customer_id and txn_date
), 
all_transactions_cte AS (
    SELECT * 
    FROM dummy_transaction_cte -- Select all from dummy transactions
    UNION ALL 
    SELECT * 
    FROM data_bank.customer_transactions -- Union with actual customer transactions
), 
running_balance_cte AS (
    SELECT 
        customer_id, -- Select customer ID
        TO_CHAR(txn_date, 'MM') AS txn_month_no, -- Extract month number from transaction date
        TO_CHAR(txn_date, 'MONTH') AS txn_month, -- Extract month name from transaction date
        txn_date, -- Select transaction date
        SUM(
            CASE 
                WHEN txn_type = 'deposit' THEN txn_amount -- Add amount for deposits
                ELSE -txn_amount -- Subtract amount for other transactions
            END
        ) OVER(
            PARTITION BY customer_id 
            ORDER BY txn_date 
            ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
        ) AS running_balance -- Calculate running balance
    FROM 
        all_transactions_cte -- Use the combined transactions data
)
SELECT DISTINCT 
    customer_id, -- Select customer ID
    txn_month_no, -- Select month number
    txn_month, -- Select month name
    LAST_VALUE(running_balance) OVER(
        PARTITION BY customer_id, txn_month 
        ORDER BY txn_date 
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS month_end_balance -- Get the closing balance for the month
FROM 
    running_balance_cte -- Use the running balance data
ORDER BY 
    customer_id, txn_month_no; -- Order by customer ID and month number
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
 - Because presenting the entire output is impractical, the above table only displays the sample rows.

### Q5. What is the percentage of customers who increase their closing balance by more than 5%?

```SQL
WITH net_monthly_balance_cte AS (
    SELECT 
        customer_id, -- Select customer ID
        TO_CHAR(txn_date, 'MM') AS txn_month_no, -- Extract month number from transaction date
        TO_CHAR(txn_date, 'MONTH') AS txn_month, -- Extract month name from transaction date
        SUM(
            CASE 
                WHEN txn_type <> 'deposit' THEN -txn_amount -- Subtract amount for non-deposit transactions
                ELSE txn_amount -- Add amount for deposit transactions
            END
        ) AS net_monthly_balance -- Calculate net monthly balance
    FROM 
        data_bank.customer_transactions
    GROUP BY 
        customer_id, txn_month_no, txn_month -- Group by customer ID, month number, and month name
    ORDER BY 
        customer_id, txn_month_no -- Order by customer ID and month number
), 
running_balance_cte AS (
    SELECT 
        customer_id, -- Select customer ID
        txn_month_no, -- Select month number
        txn_month, -- Select month name
        SUM(net_monthly_balance) OVER(
            PARTITION BY customer_id 
            ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
        ) AS running_balance -- Calculate running balance
    FROM 
        net_monthly_balance_cte
), 
opening_closing_balance AS (
    SELECT 
        customer_id, -- Select customer ID
        txn_month_no, -- Select month number
        txn_month, -- Select month name
        COALESCE(
            LAG(running_balance, 1) OVER(
                PARTITION BY customer_id 
                ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
            ), 0
        ) AS opening_balance, -- Calculate opening balance
        running_balance AS closing_balance -- Select closing balance
    FROM 
        running_balance_cte
), 
pct_balance_cte AS (
    SELECT 
        *, -- Select all columns
        CASE 
            WHEN opening_balance = 0 THEN 100 
            ELSE (closing_balance - opening_balance) * 100.0 / ABS(opening_balance) -- Calculate percentage balance change
        END AS pct_balance_change -- Percentage balance change
    FROM 
        opening_closing_balance
) 
SELECT 
    ROUND(
        COUNT(DISTINCT customer_id) * 100.0 / (
            SELECT COUNT(DISTINCT customer_id) 
            FROM data_bank.customer_transactions
        ), 2
    ) AS pct_customers_with_5_pct_increase -- Percentage of customers with at least 5% increase in balance
FROM 
    pct_balance_cte 
WHERE 
    pct_balance_change >= 5 -- Filter for customers with at least 5% balance increase
    AND txn_month_no = (SELECT MAX(txn_month_no) FROM pct_balance_cte); -- Consider only the latest month
```

| pct_customers_with_5_pct_increase |
| --------------------------------- |
| 23.40                             |

**Attention:**
 - Because it is unclear which month we are expected to compute the increase in balance amount, we are simply using the opening and closing balances from the most recent month in the dataset.


## 📊 Case Study #4 - Data Bank

Curious for more? Get your hands on all the sections [here](../README.md).

© 2024 [Shiv Katira](https://github.com/shivkatira)