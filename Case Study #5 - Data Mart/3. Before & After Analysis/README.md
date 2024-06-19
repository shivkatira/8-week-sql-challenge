# 🧺 3. Before & After Analysis
<p align="center">
<img src="../../img/5.png" align="center" width="400" height="400" >

## 📚 Table of Contents

* [Case Study Questions](#-case-study-questions)
* [My Solution](#-my-solution)
* [Case Study #5 - Data Mart](#-case-study-5---data-mart)

## ❓ Case Study Questions

1. [What is the total sales for the 4 weeks before and after 2020-06-15? What is the growth or reduction rate in actual values and percentage of sales?](#q1-what-is-the-total-sales-for-the-4-weeks-before-and-after-2020-06-15-what-is-the-growth-or-reduction-rate-in-actual-values-and-percentage-of-sales)
2. [What about the entire 12 weeks before and after?](#q2-what-about-the-entire-12-weeks-before-and-after)
3. [How do the sale metrics for these 2 periods before and after compare with the previous years in 2018 and 2019?](#q3-how-do-the-sale-metrics-for-these-2-periods-before-and-after-compare-with-the-previous-years-in-2018-and-2019)

## 💡 My Solution

### Q1. What is the total sales for the 4 weeks before and after 2020-06-15? What is the growth or reduction rate in actual values and percentage of sales?

```SQL
WITH 
    baseline_cte AS (
        SELECT 
            DISTINCT week_date AS baseline_week_date, -- Select baseline week date
            week_number AS baseline_week_number, -- Select baseline week number
            month_number AS baseline_month_number, -- Select baseline month number
            calendar_year AS baseline_calendar_year -- Select baseline calendar year
        FROM 
            data_mart.clean_weekly_sales 
        WHERE 
            week_date = TO_DATE('2020-06-15', 'YYYY-MM-DD') -- Filter for the specific baseline date
    ),
    comparison_cte AS (
        SELECT 
            SUM(
                CASE 
                    WHEN calendar_year = (SELECT baseline_calendar_year FROM baseline_cte) 
                    AND week_number::NUMERIC BETWEEN 
                        (SELECT baseline_week_number::NUMERIC-4 FROM baseline_cte) 
                        AND 
                        (SELECT baseline_week_number::NUMERIC-1 FROM baseline_cte) 
                    THEN sales 
                    ELSE 0 
                END 
            ) AS four_weeks_before_sales, -- Calculate sales for 4 weeks before baseline date
            SUM(
                CASE 
                    WHEN calendar_year = (SELECT baseline_calendar_year FROM baseline_cte) 
                    AND week_number::NUMERIC BETWEEN 
                        (SELECT baseline_week_number::NUMERIC FROM baseline_cte) 
                        AND 
                        (SELECT baseline_week_number::NUMERIC+3 FROM baseline_cte) 
                    THEN sales 
                    ELSE 0 
                END 
            ) AS four_weeks_after_sales -- Calculate sales for 4 weeks after baseline date
        FROM 
            data_mart.clean_weekly_sales
    )
SELECT 
    four_weeks_after_sales - four_weeks_before_sales AS absolute_sales_change, -- Calculate the absolute change in sales
    ROUND(
        (four_weeks_after_sales - four_weeks_before_sales) * 100.0 / four_weeks_before_sales, 2
    ) AS pct_sales_change -- Calculate the percentage change in sales
FROM 
    comparison_cte;
```

| absolute_sales_change | pct_sales_change |
| --------------------- | ---------------- |
| -26884188             | -1.15            |

### Q2. What about the entire 12 weeks before and after?

```SQL
WITH 
    baseline_cte AS (
        SELECT 
            DISTINCT week_date AS baseline_week_date, -- Select baseline week date
            week_number AS baseline_week_number, -- Select baseline week number
            month_number AS baseline_month_number, -- Select baseline month number
            calendar_year AS baseline_calendar_year -- Select baseline calendar year
        FROM 
            data_mart.clean_weekly_sales 
        WHERE 
            week_date = TO_DATE('2020-06-15', 'YYYY-MM-DD') -- Filter for the specific baseline date
    ),
    comparison_cte AS (
        SELECT 
            SUM(
                CASE 
                    WHEN calendar_year = (SELECT baseline_calendar_year FROM baseline_cte) 
                    AND week_number::NUMERIC BETWEEN 
                        (SELECT baseline_week_number::NUMERIC-12 FROM baseline_cte) 
                        AND 
                        (SELECT baseline_week_number::NUMERIC-1 FROM baseline_cte) 
                    THEN sales 
                    ELSE 0 
                END 
            ) AS twelve_weeks_before_sales, -- Calculate sales for 12 weeks before baseline date
            SUM(
                CASE 
                    WHEN calendar_year = (SELECT baseline_calendar_year FROM baseline_cte) 
                    AND week_number::NUMERIC BETWEEN 
                        (SELECT baseline_week_number::NUMERIC FROM baseline_cte) 
                        AND 
                        (SELECT baseline_week_number::NUMERIC+11 FROM baseline_cte) 
                    THEN sales 
                    ELSE 0 
                END 
            ) AS twelve_weeks_after_sales -- Calculate sales for 12 weeks after baseline date
        FROM 
            data_mart.clean_weekly_sales
    )
SELECT 
    twelve_weeks_after_sales - twelve_weeks_before_sales AS absolute_sales_change, -- Calculate the absolute change in sales
    ROUND(
        (twelve_weeks_after_sales - twelve_weeks_before_sales) * 100.0 / twelve_weeks_before_sales, 2
    ) AS pct_sales_change -- Calculate the percentage change in sales
FROM 
    comparison_cte;
```

| absolute_sales_change | pct_sales_change |
| --------------------- | ---------------- |
| -152325394            | -2.14            |

### Q3. How do the sale metrics for these 2 periods before and after compare with the previous years in 2018 and 2019?

```SQL
WITH 
    baseline_cte AS (
        SELECT 
            DISTINCT week_date AS baseline_week_date, -- Select the baseline week date
            week_number AS baseline_week_number, -- Select the baseline week number
            month_number AS baseline_month_number, -- Select the baseline month number
            calendar_year AS baseline_calendar_year -- Select the baseline calendar year
        FROM 
            data_mart.clean_weekly_sales 
        WHERE 
            week_date = TO_DATE('2020-06-15', 'YYYY-MM-DD') -- Filter for the specific baseline date
    ),
    comparison_cte AS (
        SELECT 
            calendar_year, -- Select the calendar year
            SUM(
                CASE 
                    WHEN week_number::NUMERIC BETWEEN 
                        (SELECT baseline_week_number::NUMERIC - 4 FROM baseline_cte) 
                        AND 
                        (SELECT baseline_week_number::NUMERIC - 1 FROM baseline_cte) 
                    THEN sales 
                    ELSE 0 
                END 
            ) AS four_weeks_before_sales, -- Calculate sales for 4 weeks before the baseline date
            SUM(
                CASE 
                    WHEN week_number::NUMERIC BETWEEN 
                        (SELECT baseline_week_number::NUMERIC FROM baseline_cte) 
                        AND 
                        (SELECT baseline_week_number::NUMERIC + 3 FROM baseline_cte) 
                    THEN sales 
                    ELSE 0 
                END 
            ) AS four_weeks_after_sales, -- Calculate sales for 4 weeks after the baseline date
            SUM(
                CASE 
                    WHEN week_number::NUMERIC BETWEEN 
                        (SELECT baseline_week_number::NUMERIC - 12 FROM baseline_cte) 
                        AND 
                        (SELECT baseline_week_number::NUMERIC - 1 FROM baseline_cte) 
                    THEN sales 
                    ELSE 0 
                END 
            ) AS twelve_weeks_before_sales, -- Calculate sales for 12 weeks before the baseline date
            SUM(
                CASE 
                    WHEN week_number::NUMERIC BETWEEN 
                        (SELECT baseline_week_number::NUMERIC FROM baseline_cte) 
                        AND 
                        (SELECT baseline_week_number::NUMERIC + 11 FROM baseline_cte) 
                    THEN sales 
                    ELSE 0 
                END 
            ) AS twelve_weeks_after_sales -- Calculate sales for 12 weeks after the baseline date
        FROM 
            data_mart.clean_weekly_sales
        GROUP BY 
            calendar_year -- Group results by calendar year
        ORDER BY 
            calendar_year -- Order results by calendar year
    )
SELECT 
    calendar_year, -- Select the calendar year
    four_weeks_after_sales - four_weeks_before_sales AS absolute_sales_change_four_weeks, -- Calculate the absolute sales change for 4 weeks
    ROUND(
        (four_weeks_after_sales - four_weeks_before_sales) * 100.0 / four_weeks_before_sales, 2
    ) AS pct_sales_change_four_weeks, -- Calculate the percentage sales change for 4 weeks
    twelve_weeks_after_sales - twelve_weeks_before_sales AS absolute_sales_change_twelve_weeks, -- Calculate the absolute sales change for 12 weeks
    ROUND(
        (twelve_weeks_after_sales - twelve_weeks_before_sales) * 100.0 / twelve_weeks_before_sales, 2
    ) AS pct_sales_change_twelve_weeks -- Calculate the percentage sales change for 12 weeks
FROM 
    comparison_cte
ORDER BY 
    calendar_year; -- Order the final results by calendar year
```

| calendar_year | absolute_sales_change_four_weeks | pct_sales_change_four_weeks | absolute_sales_change_twelve_weeks | pct_sales_change_twelve_weeks |
| ------------- | -------------------------------- | --------------------------- | ---------------------------------- | ----------------------------- |
| 2018          | -3936687                         | -0.19                       | 617804389                          | 10.54                         |
| 2019          | 2336594                          | 0.10                        | -20740294                          | -0.30                         |
| 2020          | -26884188                        | -1.15                       | -152325394                         | -2.14                         |

## 🧺 Case Study #5 - Data Mart

Curious for more? Get your hands on all the sections [here](../README.md).

© 2024 [Shiv Katira](https://github.com/shivkatira)