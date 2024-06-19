# 🧺 1. Data Cleansing Steps
<p align="center">
<img src="../../img/5.png" align="center" width="400" height="400" >

## 📚 Table of Contents

* [Case Study Questions](#-case-study-questions)
* [My Solution](#-my-solution)
* [Case Study #5 - Data Mart](#-case-study-5---data-mart)

## ❓ Case Study Questions

* [In a single query, perform the following operations and generate a new table in the `data_mart` schema named clean_weekly_sales:](#in-a-single-query-perform-the-following-operations-and-generate-a-new-table-in-the-data_mart-schema-named-clean_weekly_sales)
    - Convert the `week_date` to a DATE format
    - Add a `week_number` as the second column for each `week_date` value, for example any value from the 1st of January to 7th of January will be 1, 8th to 14th will be 2 etc
    - Add a `month_number` with the calendar month for each `week_date` value as the 3rd column
    - Add a `calendar_year` column as the 4th column containing either 2018, 2019 or 2020 values
    - Add a new column called `age_band` after the original `segment` column using the following mapping on the number inside the `segment` value
        | segment | age_band     |
        |---------|--------------|
        | 1       | Young Adults |
        | 2	      | Middle Aged  |
        | 3 or 4  | Retirees     |

    - Add a new `demographic` column using the following mapping for the first letter in the `segment` values:
        | segment | demographic  |
        |---------|--------------|
        | C       | Couples      |
        | F	      | Families     |
    - Ensure all `null` string values with an "unknown" string value in the original `segment` column as well as the new `age_band` and `demographic` columns
    - Generate a new `avg_transaction` column as the sales value divided by transactions rounded to 2 decimal places for each record

## 💡 My Solution

### In a single query, perform the following operations and generate a new table in the `data_mart` schema named clean_weekly_sales:
- Convert the `week_date` to a DATE format
- Add a `week_number` as the second column for each `week_date` value, for example any value from the 1st of January to 7th of January will be 1, 8th to 14th will be 2 etc
- Add a `month_number` with the calendar month for each `week_date` value as the 3rd column
- Add a `calendar_year` column as the 4th column containing either 2018, 2019 or 2020 values
- Add a new column called `age_band` after the original `segment` column using the following mapping on the number inside the `segment` value
    | segment | age_band     |
    |---------|--------------|
    | 1       | Young Adults |
    | 2	      | Middle Aged  |
    | 3 or 4  | Retirees     |

- Add a new `demographic` column using the following mapping for the first letter in the `segment` values:
    | segment | demographic  |
    |---------|--------------|
    | C       | Couples      |
    | F	      | Families     |
- Ensure all `null` string values with an "unknown" string value in the original `segment` column as well as the new `age_band` and `demographic` columns
- Generate a new `avg_transaction` column as the sales value divided by transactions rounded to 2 decimal places for each record

```SQL
CREATE TABLE data_mart.clean_weekly_sales AS (
    WITH week_date_cte AS (
        SELECT 
            TO_DATE(week_date, 'DD/MM/YY') AS week_date, -- Convert week_date to DATE format
            region, 
            platform, 
            segment, 
            customer_type, 
            transactions, 
            sales 
        FROM data_mart.weekly_sales
    )
    SELECT 
        week_date, -- Select converted week_date
        TO_CHAR(week_date, 'WW') AS week_number, -- Convert week_date to week number
        TO_CHAR(week_date, 'MM') AS month_number, -- Convert week_date to month number
        TO_CHAR(week_date, 'YYYY') AS calendar_year, -- Convert week_date to calendar year
        region, 
        platform, 
        CASE 
            WHEN segment NOT LIKE 'C%' AND segment NOT LIKE 'F%' THEN 'unknown' 
            ELSE segment 
        END AS segment, -- Replace null segments with 'unknown'
        COALESCE(
            CASE 
                WHEN segment LIKE '%1' THEN 'Young Adults' 
                WHEN segment LIKE '%2' THEN 'Middle Aged' 
                WHEN segment LIKE '%3' THEN 'Retirees' 
                WHEN segment LIKE '%4' THEN 'Retirees' 
                ELSE NULL 
            END, 'unknown'
        ) AS age_band, -- Map segment to age_band or 'unknown'
        COALESCE(
            CASE 
                WHEN segment LIKE 'C%' THEN 'Couples' 
                WHEN segment LIKE 'F%' THEN 'Families' 
                ELSE NULL 
            END, 'unknown'
        ) AS demographic, -- Map segment to demographic or 'unknown'
        customer_type, 
        transactions, 
        sales, 
        ROUND(sales / transactions, 2) AS avg_transaction -- Calculate average transaction value
    FROM week_date_cte
);

SELECT * FROM data_mart.clean_weekly_sales; -- Select all records from the newly created table
```

| week_date                | week_number | month_number | calendar_year | region        | platform | segment | age_band     | demographic | customer_type | transactions | sales    | avg_transaction |
| ------------------------ | ----------- | ------------ | ------------- | ------------- | -------- | ------- | ------------ | ----------- | ------------- | ------------ | -------- | --------------- |
| 2020-08-31T00:00:00.000Z | 35          | 08           | 2020          | ASIA          | Retail   | C3      | Retirees     | Couples     | New           | 120631       | 3656163  | 30.00           |
| 2020-08-31T00:00:00.000Z | 35          | 08           | 2020          | ASIA          | Retail   | F1      | Young Adults | Families    | New           | 31574        | 996575   | 31.00           |
| 2020-08-31T00:00:00.000Z | 35          | 08           | 2020          | USA           | Retail   | unknown | unknown      | unknown     | Guest         | 529151       | 16509610 | 31.00           |
| 2020-08-31T00:00:00.000Z | 35          | 08           | 2020          | EUROPE        | Retail   | C1      | Young Adults | Couples     | New           | 4517         | 141942   | 31.00           |
| 2020-08-31T00:00:00.000Z | 35          | 08           | 2020          | AFRICA        | Retail   | C2      | Middle Aged  | Couples     | New           | 58046        | 1758388  | 30.00           |
| 2020-08-31T00:00:00.000Z | 35          | 08           | 2020          | CANADA        | Shopify  | F2      | Middle Aged  | Families    | Existing      | 1336         | 243878   | 182.00          |
| 2020-08-31T00:00:00.000Z | 35          | 08           | 2020          | AFRICA        | Shopify  | F3      | Retirees     | Families    | Existing      | 2514         | 519502   | 206.00          |
| 2020-08-31T00:00:00.000Z | 35          | 08           | 2020          | ASIA          | Shopify  | F1      | Young Adults | Families    | Existing      | 2158         | 371417   | 172.00          |
| 2020-08-31T00:00:00.000Z | 35          | 08           | 2020          | AFRICA        | Shopify  | F2      | Middle Aged  | Families    | New           | 318          | 49557    | 155.00          |

**Attention:**
- Because presenting the entire output is impractical, the above table only displays the first 10 rows.

## 🧺 Case Study #5 - Data Mart

Curious for more? Get your hands on all the sections [here](../README.md).

© 2024 [Shiv Katira](https://github.com/shivkatira)