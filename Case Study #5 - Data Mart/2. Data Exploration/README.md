# 🧺 2. Data Exploration
<p align="center">
<img src="../../img/5.png" align="center" width="400" height="400" >

## 📚 Table of Contents

* [Case Study Questions](#-case-study-questions)
* [My Solution](#-my-solution)
* [Case Study #5 - Data Mart](#-case-study-5---data-mart)

## ❓ Case Study Questions

1. [What day of the week is used for each `week_date` value?](#q1-what-day-of-the-week-is-used-for-each-week_date-value)
2. [What range of week numbers are missing from the dataset?](#q2-what-range-of-week-numbers-are-missing-from-the-dataset)
3. [How many total transactions were there for each year in the dataset?](#q3-how-many-total-transactions-were-there-for-each-year-in-the-dataset)
4. [What is the total sales for each `region` for each month?](#q4-what-is-the-total-sales-for-each-region-for-each-month)
5. [What is the total count of transactions for each `platform`](#q5-what-is-the-total-count-of-transactions-for-each-platform)
6. [What is the percentage of sales for Retail vs Shopify for each month?](#q6-what-is-the-percentage-of-sales-for-retail-vs-shopify-for-each-month)
7. [What is the percentage of sales by `demographic` for each year in the dataset?](#q7-what-is-the-percentage-of-sales-by-demographic-for-each-year-in-the-dataset)
8. [Which `age_band` and `demographic` values contribute the most to Retail sales?](#q8-which-age_band-and-demographic-values-contribute-the-most-to-retail-sales)
9. [Can we use the `avg_transaction` column to find the average transaction size for each year for Retail vs Shopify? If not - how would you calculate it instead?](#q9-can-we-use-the-avg_transaction-column-to-find-the-average-transaction-size-for-each-year-for-retail-vs-shopify-if-not---how-would-you-calculate-it-instead)

## 💡 My Solution

### Q1. What day of the week is used for each `week_date` value?

```SQL
SELECT DISTINCT 
    TO_CHAR(week_date, 'DAY') AS week_day -- Convert week_date to the name of the day of the week
FROM 
    data_mart.clean_weekly_sales; -- Select from the clean_weekly_sales table in the data_mart schema
```

| week_day  |
| --------- |
| MONDAY    |

### Q2. What range of week numbers are missing from the dataset?

```SQL
WITH all_week_number_cte AS (
    SELECT 
        GENERATE_SERIES(1, 53)::TEXT AS all_week_number -- Generate a series of week numbers from 1 to 53
)
SELECT 
    all_week_number -- Select all generated week numbers
FROM 
    all_week_number_cte -- From the generated series of week numbers
WHERE 
    all_week_number NOT IN ( 
        SELECT 
            week_number -- Select week numbers present in the dataset
        FROM 
            data_mart.clean_weekly_sales -- From the clean_weekly_sales table in the data_mart schema
    );
```

| all_week_number |
| --------------- |
| 1               |
| 2               |
| 3               |
| 4               |
| 5               |
| 6               |
| 7               |
| 8               |
| 9               |
| 10              |
| 11              |
| 37              |
| 38              |
| 39              |
| 40              |
| 41              |
| 42              |
| 43              |
| 44              |
| 45              |
| 46              |
| 47              |
| 48              |
| 49              |
| 50              |
| 51              |
| 52              |
| 53              |

### Q3. How many total transactions were there for each year in the dataset?

```SQL
SELECT 
    calendar_year, -- Select the calendar year
    SUM(transactions) AS total_transactions -- Calculate the total number of transactions for each year
FROM 
    data_mart.clean_weekly_sales -- From the clean_weekly_sales table in the data_mart schema
GROUP BY 
    calendar_year -- Group results by calendar year to aggregate transactions
ORDER BY 
    calendar_year; -- Sort results by calendar year for organized presentation
```

| calendar_year | total_transactions |
| ------------- | ------------------ |
| 2018          | 346406460          |
| 2019          | 365639285          |
| 2020          | 375813651          |

### Q4. What is the total sales for each `region` for each month?

```SQL
SELECT 
    region, -- Select the region
    month_number, -- Select the month number
    SUM(sales) AS total_sales -- Calculate the total sales for each region and month
FROM 
    data_mart.clean_weekly_sales -- From the clean_weekly_sales table in the data_mart schema
GROUP BY 
    region, -- Group results by region
    month_number -- Group results by month number
ORDER BY 
    region, -- Sort results by region
    month_number; -- Sort results by month number within each region
```

| region        | month_number | total_sales |
| ------------- | ------------ | ----------- |
| AFRICA        | 03           | 567767480   |
| AFRICA        | 04           | 1911783504  |
| AFRICA        | 05           | 1647244738  |
| AFRICA        | 06           | 1767559760  |
| AFRICA        | 07           | 1960219710  |
| AFRICA        | 08           | 1809596890  |
| AFRICA        | 09           | 276320987   |
| ASIA          | 03           | 529770793   |
| ASIA          | 04           | 1804628707  |
| ASIA          | 05           | 1526285399  |
| ASIA          | 06           | 1619482889  |
| ASIA          | 07           | 1768844756  |
| ASIA          | 08           | 1663320609  |
| ASIA          | 09           | 252836807   |
| CANADA        | 03           | 144634329   |
| CANADA        | 04           | 484552594   |
| CANADA        | 05           | 412378365   |
| CANADA        | 06           | 443846698   |
| CANADA        | 07           | 477134947   |
| CANADA        | 08           | 447073019   |
| CANADA        | 09           | 69067959    |
| EUROPE        | 03           | 35337093    |
| EUROPE        | 04           | 127334255   |
| EUROPE        | 05           | 109338389   |
| EUROPE        | 06           | 122813826   |
| EUROPE        | 07           | 136757466   |
| EUROPE        | 08           | 122102995   |
| EUROPE        | 09           | 18877433    |
| OCEANIA       | 03           | 783282888   |
| OCEANIA       | 04           | 2599767620  |
| OCEANIA       | 05           | 2215657304  |
| OCEANIA       | 06           | 2371884744  |
| OCEANIA       | 07           | 2563459400  |
| OCEANIA       | 08           | 2432313652  |
| OCEANIA       | 09           | 372465518   |
| SOUTH AMERICA | 03           | 71023109    |
| SOUTH AMERICA | 04           | 238451531   |
| SOUTH AMERICA | 05           | 201391809   |
| SOUTH AMERICA | 06           | 218247455   |
| SOUTH AMERICA | 07           | 235582776   |
| SOUTH AMERICA | 08           | 221166052   |
| SOUTH AMERICA | 09           | 34175583    |
| USA           | 03           | 225353043   |
| USA           | 04           | 759786323   |
| USA           | 05           | 655967121   |
| USA           | 06           | 703878990   |
| USA           | 07           | 760331754   |
| USA           | 08           | 712002790   |
| USA           | 09           | 110532368   |


### Q5. What is the total count of transactions for each `platform`

```SQL
SELECT 
    platform, -- Select the platform
    SUM(transactions) AS total_transactions -- Calculate the total number of transactions for each platform
FROM 
    data_mart.clean_weekly_sales -- From the clean_weekly_sales table in the data_mart schema
GROUP BY 
    platform -- Group results by platform
ORDER BY 
    platform; -- Sort results by platform
```

| platform | total_transactions |
| -------- | ------------------ |
| Retail   | 1081934227         |
| Shopify  | 5925169            |

### Q6. What is the percentage of sales for Retail vs Shopify for each month?

```SQL
WITH monthly_sales_cte AS (
    SELECT 
        calendar_year, -- Select the calendar year
        month_number, -- Select the month number
        SUM(sales) AS total_monthly_sales -- Calculate the total monthly sales
    FROM 
        data_mart.clean_weekly_sales -- From the clean_weekly_sales table in the data_mart schema
    GROUP BY 
        calendar_year, month_number -- Group results by calendar year and month number
    ORDER BY 
        calendar_year, month_number -- Sort results by calendar year and month number
), 
retail_sales_cte AS (
    SELECT 
        calendar_year, -- Select the calendar year
        month_number, -- Select the month number
        SUM(sales) AS retail_sales -- Calculate the total retail sales
    FROM 
        data_mart.clean_weekly_sales -- From the clean_weekly_sales table in the data_mart schema
    WHERE 
        platform = 'Retail' -- Filter for Retail platform
    GROUP BY 
        calendar_year, month_number -- Group results by calendar year and month number
    ORDER BY 
        calendar_year, month_number -- Sort results by calendar year and month number
), 
shopify_sales_cte AS (
    SELECT 
        calendar_year, -- Select the calendar year
        month_number, -- Select the month number
        SUM(sales) AS shopify_sales -- Calculate the total Shopify sales
    FROM 
        data_mart.clean_weekly_sales -- From the clean_weekly_sales table in the data_mart schema
    WHERE 
        platform = 'Shopify' -- Filter for Shopify platform
    GROUP BY 
        calendar_year, month_number -- Group results by calendar year and month number
    ORDER BY 
        calendar_year, month_number -- Sort results by calendar year and month number
)
SELECT 
    calendar_year, -- Select the calendar year
    month_number, -- Select the month number
    ROUND(retail_sales * 100.0 / total_monthly_sales, 2) AS retail_sales_share, -- Calculate the percentage of retail sales
    ROUND(shopify_sales * 100.0 / total_monthly_sales, 2) AS shopify_sales_share -- Calculate the percentage of Shopify sales
FROM 
    retail_sales_cte -- From the retail sales CTE
    LEFT JOIN shopify_sales_cte USING (calendar_year, month_number) -- Left join with Shopify sales CTE on calendar year and month number
    LEFT JOIN monthly_sales_cte USING (calendar_year, month_number); -- Left join with monthly sales CTE on calendar year and month number
```

| calendar_year | month_number | retail_sales_share | shopify_sales_share |
| ------------- | ------------ | ------------------ | ------------------- |
| 2018          | 03           | 97.92              | 2.08                |
| 2018          | 04           | 97.93              | 2.07                |
| 2018          | 05           | 97.73              | 2.27                |
| 2018          | 06           | 97.76              | 2.24                |
| 2018          | 07           | 97.75              | 2.25                |
| 2018          | 08           | 97.71              | 2.29                |
| 2018          | 09           | 97.68              | 2.32                |
| 2019          | 03           | 97.71              | 2.29                |
| 2019          | 04           | 97.80              | 2.20                |
| 2019          | 05           | 97.52              | 2.48                |
| 2019          | 06           | 97.42              | 2.58                |
| 2019          | 07           | 97.35              | 2.65                |
| 2019          | 08           | 97.21              | 2.79                |
| 2019          | 09           | 97.09              | 2.91                |
| 2020          | 03           | 97.30              | 2.70                |
| 2020          | 04           | 96.96              | 3.04                |
| 2020          | 05           | 96.71              | 3.29                |
| 2020          | 06           | 96.80              | 3.20                |
| 2020          | 07           | 96.67              | 3.33                |
| 2020          | 08           | 96.51              | 3.49                |

### Q7. What is the percentage of sales by `demographic` for each year in the dataset?

```SQL
WITH annual_sales_cte AS (
    SELECT 
        calendar_year, -- Select the calendar year
        SUM(sales) AS total_annual_sales -- Calculate the total annual sales
    FROM 
        data_mart.clean_weekly_sales -- From the clean_weekly_sales table in the data_mart schema
    GROUP BY 
        calendar_year -- Group results by calendar year
    ORDER BY 
        calendar_year -- Sort results by calendar year
), 
couples_sales_cte AS (
    SELECT 
        calendar_year, -- Select the calendar year
        SUM(sales) AS couples_sales -- Calculate the total sales for the Couples demographic
    FROM 
        data_mart.clean_weekly_sales -- From the clean_weekly_sales table in the data_mart schema
    WHERE 
        demographic = 'Couples' -- Filter for Couples demographic
    GROUP BY 
        calendar_year -- Group results by calendar year
    ORDER BY 
        calendar_year -- Sort results by calendar year
), 
families_sales_cte AS (
    SELECT 
        calendar_year, -- Select the calendar year
        SUM(sales) AS families_sales -- Calculate the total sales for the Families demographic
    FROM 
        data_mart.clean_weekly_sales -- From the clean_weekly_sales table in the data_mart schema
    WHERE 
        demographic = 'Families' -- Filter for Families demographic
    GROUP BY 
        calendar_year -- Group results by calendar year
    ORDER BY 
        calendar_year -- Sort results by calendar year
), 
unknown_sales_cte AS (
    SELECT 
        calendar_year, -- Select the calendar year
        SUM(sales) AS unknown_sales -- Calculate the total sales for the unknown demographic
    FROM 
        data_mart.clean_weekly_sales -- From the clean_weekly_sales table in the data_mart schema
    WHERE 
        demographic = 'unknown' -- Filter for unknown demographic
    GROUP BY 
        calendar_year -- Group results by calendar year
    ORDER BY 
        calendar_year -- Sort results by calendar year
)
SELECT 
    calendar_year, -- Select the calendar year
    ROUND(couples_sales * 100.0 / total_annual_sales, 2) AS couples_sales_share, -- Calculate the percentage of sales for Couples demographic
    ROUND(families_sales * 100.0 / total_annual_sales, 2) AS families_sales_share, -- Calculate the percentage of sales for Families demographic
    ROUND(unknown_sales * 100.0 / total_annual_sales, 2) AS unknown_sales_share -- Calculate the percentage of sales for unknown demographic
FROM 
    couples_sales_cte -- From the Couples sales CTE
    LEFT JOIN families_sales_cte USING (calendar_year) -- Left join with Families sales CTE on calendar year
    LEFT JOIN unknown_sales_cte USING (calendar_year) -- Left join with unknown sales CTE on calendar year
    LEFT JOIN annual_sales_cte USING (calendar_year); -- Left join with annual sales CTE on calendar year
```

| calendar_year | couples_sales_share | families_sales_share | unknown_sales_share |
| ------------- | ------------------- | -------------------- | ------------------- |
| 2018          | 26.38               | 31.99                | 41.63               |
| 2019          | 27.28               | 32.47                | 40.25               |
| 2020          | 28.72               | 32.73                | 38.55               |

### Q8. Which `age_band` and `demographic` values contribute the most to Retail sales?

```SQL
SELECT 
    age_band, -- Select the age band
    demographic, -- Select the demographic
    ROUND(
        SUM(sales) * 100.0 / ( 
            SELECT SUM(sales) -- Calculate the total sales
            FROM data_mart.clean_weekly_sales 
            WHERE platform = 'Retail' -- Filter for Retail platform
        ), 2 
    ) AS retail_sales_share -- Calculate the percentage share of Retail sales
FROM 
    data_mart.clean_weekly_sales -- From the clean_weekly_sales table in the data_mart schema
WHERE 
    platform = 'Retail' -- Filter for Retail platform
    AND age_band <> 'unknown' -- Exclude unknown age bands
    AND demographic <> 'unknown' -- Exclude unknown demographics
GROUP BY 
    age_band, demographic -- Group results by age band and demographic
ORDER BY 
    retail_sales_share DESC -- Sort results by retail sales share in descending order
LIMIT 1; -- Limit the results to the top record
```

| age_band | demographic | retail_sales_share |
| -------- | ----------- | ------------------ |
| Retirees | Families    | 16.73              |

### Q9. Can we use the `avg_transaction` column to find the average transaction size for each year for Retail vs Shopify? If not - how would you calculate it instead?

```SQL
WITH 
    -- Calculate annual sales for each calendar year
    annual_sales_cte AS (
        SELECT 
            calendar_year, 
            SUM(sales) AS annual_sales -- Sum of sales for each calendar year
        FROM 
            data_mart.clean_weekly_sales
        GROUP BY 
            calendar_year
        ORDER BY 
            calendar_year
    )
    
-- Query to compare average transaction sizes between Retail and Shopify
SELECT 
    calendar_year, -- Select calendar year
    platform, -- Select platform (Retail or Shopify)
    ROUND( AVG(avg_transaction), 2 ) AS avg_transaction_aggregated, -- Calculate aggregated average transaction size
    ROUND( SUM(sales)/ SUM(transactions), 2 ) AS avg_transaction_calculated -- Calculate average transaction size directly
FROM 
    data_mart.clean_weekly_sales
GROUP BY 
    calendar_year, platform -- Group by calendar year and platform
ORDER BY 
    calendar_year, platform; -- Sort results by calendar year and platform
```

| calendar_year | platform | avg_transaction_aggregated | avg_transaction_calculated |
| ------------- | -------- | -------------------------- | -------------------------- |
| 2018          | Retail   | 42.41                      | 36.00                      |
| 2018          | Shopify  | 187.80                     | 192.00                     |
| 2019          | Retail   | 41.47                      | 36.00                      |
| 2019          | Shopify  | 177.07                     | 183.00                     |
| 2020          | Retail   | 40.14                      | 36.00                      |
| 2020          | Shopify  | 174.40                     | 179.00                     |

## 🧺 Case Study #5 - Data Mart

Curious for more? Get your hands on all the sections [here](../README.md).

© 2024 [Shiv Katira](https://github.com/shivkatira)