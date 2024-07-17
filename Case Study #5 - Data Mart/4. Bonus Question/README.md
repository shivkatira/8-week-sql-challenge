# 🛒 4. Bonus Question
<p align="center">
<img src="../../img/5.png" align="center" width="400" height="400" >

## 📚 Table of Contents

* [Case Study Questions](#-case-study-questions)
* [My Solution](#-my-solution)
* [Case Study #5 - Data Mart](#-case-study-5---data-mart)

## ❓ Case Study Questions

* [Which areas of the business have the highest negative impact in sales metrics performance in 2020 for the 12 week before and after period?](#which-areas-of-the-business-have-the-highest-negative-impact-in-sales-metrics-performance-in-2020-for-the-12-week-before-and-after-period)
    - `region`
    - `platform`
    - `age_band`
    - `demographic`
    - `customer_type`

* [Do you have any further recommendations for Danny’s team at Data Mart or any interesting insights based off this analysis?](#do-you-have-any-further-recommendations-for-dannys-team-at-data-mart-or-any-interesting-insights-based-off-this-analysis)

## 💡 My Solution

### Which areas of the business have the highest negative impact in sales metrics performance in 2020 for the 12 week before and after period?

- `region`

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
                region, -- Select the region
                SUM(
                    CASE 
                        WHEN calendar_year = (SELECT baseline_calendar_year FROM baseline_cte) 
                        AND week_number::NUMERIC BETWEEN 
                            (SELECT baseline_week_number::NUMERIC - 12 FROM baseline_cte) 
                            AND 
                            (SELECT baseline_week_number::NUMERIC - 1 FROM baseline_cte) 
                        THEN sales 
                        ELSE 0 
                    END 
                ) AS twelve_weeks_before_sales, -- Calculate sales for 12 weeks before the baseline date for each region
                SUM(
                    CASE 
                        WHEN calendar_year = (SELECT baseline_calendar_year FROM baseline_cte) 
                        AND week_number::NUMERIC BETWEEN 
                            (SELECT baseline_week_number::NUMERIC FROM baseline_cte) 
                            AND 
                            (SELECT baseline_week_number::NUMERIC + 11 FROM baseline_cte) 
                        THEN sales 
                        ELSE 0 
                    END 
                ) AS twelve_weeks_after_sales -- Calculate sales for 12 weeks after the baseline date for each region
            FROM 
                data_mart.clean_weekly_sales 
            GROUP BY 
                region -- Group results by region
            ORDER BY 
                region -- Order results by region
        )
    SELECT 
        region, -- Select the region
        twelve_weeks_after_sales - twelve_weeks_before_sales AS absolute_sales_change, -- Calculate the absolute sales change for 12 weeks
        ROUND(
            (twelve_weeks_after_sales - twelve_weeks_before_sales) * 100.0 / twelve_weeks_before_sales, 2
        ) AS pct_sales_change -- Calculate the percentage sales change for 12 weeks
    FROM 
        comparison_cte
    ORDER BY 
        pct_sales_change; -- Order the final results by percentage sales change
    ```

    | region        | absolute_sales_change | pct_sales_change |
    | ------------- | --------------------- | ---------------- |
    | ASIA          | -53436845             | -3.26            |
    | OCEANIA       | -71321100             | -3.03            |
    | SOUTH AMERICA | -4584174              | -2.15            |
    | CANADA        | -8174013              | -1.92            |
    | USA           | -10814843             | -1.60            |
    | AFRICA        | -9146811              | -0.54            |
    | EUROPE        | 5152392               | 4.73             |

- `platform`

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
                platform, -- Select the platform
                SUM(
                    CASE 
                        WHEN calendar_year = (SELECT baseline_calendar_year FROM baseline_cte) 
                        AND week_number::NUMERIC BETWEEN 
                            (SELECT baseline_week_number::NUMERIC - 12 FROM baseline_cte) 
                            AND 
                            (SELECT baseline_week_number::NUMERIC - 1 FROM baseline_cte) 
                        THEN sales 
                        ELSE 0 
                    END 
                ) AS twelve_weeks_before_sales, -- Calculate sales for 12 weeks before the baseline date for each platform
                SUM(
                    CASE 
                        WHEN calendar_year = (SELECT baseline_calendar_year FROM baseline_cte) 
                        AND week_number::NUMERIC BETWEEN 
                            (SELECT baseline_week_number::NUMERIC FROM baseline_cte) 
                            AND 
                            (SELECT baseline_week_number::NUMERIC + 11 FROM baseline_cte) 
                        THEN sales 
                        ELSE 0 
                    END 
                ) AS twelve_weeks_after_sales -- Calculate sales for 12 weeks after the baseline date for each platform
            FROM 
                data_mart.clean_weekly_sales 
            GROUP BY 
                platform -- Group results by platform
            ORDER BY 
                platform -- Order results by platform
        )
    SELECT 
        platform, -- Select the platform
        twelve_weeks_after_sales - twelve_weeks_before_sales AS absolute_sales_change, -- Calculate the absolute sales change for 12 weeks
        ROUND(
            (twelve_weeks_after_sales - twelve_weeks_before_sales) * 100.0 / twelve_weeks_before_sales, 2
        ) AS pct_sales_change -- Calculate the percentage sales change for 12 weeks
    FROM 
        comparison_cte
    ORDER BY 
        pct_sales_change; -- Order the final results by percentage sales change
    ```

    | platform | absolute_sales_change | pct_sales_change |
    | -------- | --------------------- | ---------------- |
    | Retail   | -168083834            | -2.43            |
    | Shopify  | 15758440              | 7.18             |

- `age_band`

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
                age_band, -- Select the age band
                SUM(
                    CASE 
                        WHEN calendar_year = (SELECT baseline_calendar_year FROM baseline_cte) 
                        AND week_number::NUMERIC BETWEEN 
                            (SELECT baseline_week_number::NUMERIC - 12 FROM baseline_cte) 
                            AND 
                            (SELECT baseline_week_number::NUMERIC - 1 FROM baseline_cte) 
                        THEN sales 
                        ELSE 0 
                    END 
                ) AS twelve_weeks_before_sales, -- Calculate sales for 12 weeks before the baseline date for each age band
                SUM(
                    CASE 
                        WHEN calendar_year = (SELECT baseline_calendar_year FROM baseline_cte) 
                        AND week_number::NUMERIC BETWEEN 
                            (SELECT baseline_week_number::NUMERIC FROM baseline_cte) 
                            AND 
                            (SELECT baseline_week_number::NUMERIC + 11 FROM baseline_cte) 
                        THEN sales 
                        ELSE 0 
                    END 
                ) AS twelve_weeks_after_sales -- Calculate sales for 12 weeks after the baseline date for each age band
            FROM 
                data_mart.clean_weekly_sales 
            GROUP BY 
                age_band -- Group results by age band
            ORDER BY 
                age_band -- Order results by age band
        )
    SELECT 
        age_band, -- Select the age band
        twelve_weeks_after_sales - twelve_weeks_before_sales AS absolute_sales_change, -- Calculate the absolute sales change for 12 weeks
        ROUND(
            (twelve_weeks_after_sales - twelve_weeks_before_sales) * 100.0 / twelve_weeks_before_sales, 2
        ) AS pct_sales_change -- Calculate the percentage sales change for 12 weeks
    FROM 
        comparison_cte
    ORDER BY 
        pct_sales_change; -- Order the final results by percentage sales change
    ```

    | age_band     | absolute_sales_change | pct_sales_change |
    | ------------ | --------------------- | ---------------- |
    | unknown      | -92393021             | -3.34            |
    | Middle Aged  | -22994292             | -1.97            |
    | Retirees     | -29549521             | -1.23            |
    | Young Adults | -7388560              | -0.92            |

- `demographic`

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
                demographic, -- Select the demographic
                SUM(
                    CASE 
                        WHEN calendar_year = (SELECT baseline_calendar_year FROM baseline_cte) 
                        AND week_number::NUMERIC BETWEEN 
                            (SELECT baseline_week_number::NUMERIC - 12 FROM baseline_cte) 
                            AND 
                            (SELECT baseline_week_number::NUMERIC - 1 FROM baseline_cte) 
                        THEN sales 
                        ELSE 0 
                    END 
                ) AS twelve_weeks_before_sales, -- Calculate sales for 12 weeks before the baseline date for each demographic
                SUM(
                    CASE 
                        WHEN calendar_year = (SELECT baseline_calendar_year FROM baseline_cte) 
                        AND week_number::NUMERIC BETWEEN 
                            (SELECT baseline_week_number::NUMERIC FROM baseline_cte) 
                            AND 
                            (SELECT baseline_week_number::NUMERIC + 11 FROM baseline_cte) 
                        THEN sales 
                        ELSE 0 
                    END 
                ) AS twelve_weeks_after_sales -- Calculate sales for 12 weeks after the baseline date for each demographic
            FROM 
                data_mart.clean_weekly_sales 
            GROUP BY 
                demographic -- Group results by demographic
            ORDER BY 
                demographic -- Order results by demographic
        )
    SELECT 
        demographic, -- Select the demographic
        twelve_weeks_after_sales - twelve_weeks_before_sales AS absolute_sales_change, -- Calculate the absolute sales change for 12 weeks
        ROUND(
            (twelve_weeks_after_sales - twelve_weeks_before_sales) * 100.0 / twelve_weeks_before_sales, 2
        ) AS pct_sales_change -- Calculate the percentage sales change for 12 weeks
    FROM 
        comparison_cte
    ORDER BY 
        pct_sales_change; -- Order the final results by percentage sales change
    ```

    | demographic | absolute_sales_change | pct_sales_change |
    | ----------- | --------------------- | ---------------- |
    | unknown     | -92393021             | -3.34            |
    | Families    | -42320015             | -1.82            |
    | Couples     | -17612358             | -0.87            |

- `customer_type`

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
                customer_type, -- Select the customer type
                SUM(
                    CASE 
                        WHEN calendar_year = (SELECT baseline_calendar_year FROM baseline_cte) 
                        AND week_number::NUMERIC BETWEEN 
                            (SELECT baseline_week_number::NUMERIC - 12 FROM baseline_cte) 
                            AND 
                            (SELECT baseline_week_number::NUMERIC - 1 FROM baseline_cte) 
                        THEN sales 
                        ELSE 0 
                    END 
                ) AS twelve_weeks_before_sales, -- Calculate sales for 12 weeks before the baseline date for each customer type
                SUM(
                    CASE 
                        WHEN calendar_year = (SELECT baseline_calendar_year FROM baseline_cte) 
                        AND week_number::NUMERIC BETWEEN 
                            (SELECT baseline_week_number::NUMERIC FROM baseline_cte) 
                            AND 
                            (SELECT baseline_week_number::NUMERIC + 11 FROM baseline_cte) 
                        THEN sales 
                        ELSE 0 
                    END 
                ) AS twelve_weeks_after_sales -- Calculate sales for 12 weeks after the baseline date for each customer type
            FROM 
                data_mart.clean_weekly_sales 
            GROUP BY 
                customer_type -- Group results by customer type
            ORDER BY 
                customer_type -- Order results by customer type
        )
    SELECT 
        customer_type, -- Select the customer type
        twelve_weeks_after_sales - twelve_weeks_before_sales AS absolute_sales_change, -- Calculate the absolute sales change for 12 weeks
        ROUND(
            (twelve_weeks_after_sales - twelve_weeks_before_sales) * 100.0 / twelve_weeks_before_sales, 2
        ) AS pct_sales_change -- Calculate the percentage sales change for 12 weeks
    FROM 
        comparison_cte
    ORDER BY 
        pct_sales_change; -- Order the final results by percentage sales change
    ```

    | customer_type | absolute_sales_change | pct_sales_change |
    | ------------- | --------------------- | ---------------- |
    | Guest         | -77202666             | -3.00            |
    | Existing      | -83872973             | -2.27            |
    | New           | 8750245               | 1.01             |

### Do you have any further recommendations for Danny’s team at Data Mart or any interesting insights based off this analysis?

#### Insights

##### Quantifiable Impact of Sustainable Packaging (June 2020):

- Sales Consistency: Annual sales figures exhibit stability, with negligible variation across years.
- Seasonal Trends: Sales from April to August consistently surpass those from March and September across all regions.
- Platform Distribution: A dominant 97% of sales are processed through the retail platform, whereas Shopify accounts for the remaining 3%. However, the average transaction value on Shopify is quintuple that of retail transactions.
- Demographic Contribution: Couples contribute 28% to total sales, families 32%, and the unknown demographic group 40%. Retiree families lead sales within known segments.
- Geographical Reception: The sustainable packaging initiative has been positively received in Europe. Conversely, regions such as Asia, Oceania, and South Africa exhibit significant reluctance.
- Platform-Specific Impact: Sustainable packaging introduction has resulted in increased sales on Shopify but a decrease on the retail platform.
- Age Group Response: While sales have generally declined post-change, young adults show the least impact, with middle-aged, retirees, and unknown age groups showing higher adverse effects.
- Demographic Group Impact: Couples demonstrate the least sales reduction, whereas families and unknown demographics are more negatively affected.
- Customer Type Response: New customers exhibit a preference for sustainable packaging, unlike existing customers and guests who show a negative response.

#### Recommendations

##### Customer Registration and Data Enhancement:

- Loyalty Programs: Implement loyalty programs to incentivize customer registration on both Retail and Shopify platforms. This initiative aims to reduce the unknown values in demographic and age data, facilitating more granular insights.

##### Strategic Focus Areas:

1. Regional Packaging Strategy:

    - Europe: Maintain sustainable packaging for all products.
    - Other Regions: Apply sustainable packaging selectively to products preferred by Couples and Young Adults, as these segments are less averse to the change.

2. Platform-Specific Strategy:

    - Shopify: Continue using sustainable packaging for all products sold on Shopify, leveraging the higher transaction value and positive reception.

3. Decision-Making Framework:

    - Customer Type Neutrality: Avoid basing strategic packaging decisions on customer type (new, existing, guest), due to the complexity in predicting preferences accurately. Instead, focus on demographic and age-based preferences.

##### Additional Recommendations:

- Marketing and Education: Develop targeted marketing campaigns to educate and promote the benefits of sustainable packaging, especially in regions and demographics showing resistance.
- Feedback Mechanisms: Establish robust feedback loops with customers to continually assess and address their concerns regarding sustainable packaging.
- Product Diversification: Consider diversifying product offerings with both sustainable and traditional packaging options, to cater to varied customer preferences while gradually encouraging sustainable choices.

#### Conclusion:

The implementation of sustainable packaging has had varied impacts across different platforms, regions, demographics, and customer types. By strategically focusing on areas of strength and leveraging loyalty programs, Data Mart can mitigate negative impacts and continue its commitment to sustainability without compromising sales performance. Further, ongoing analysis and customer engagement will be crucial in fine-tuning these strategies to optimize outcomes.

## 🛒 Case Study #5 - Data Mart

Curious for more? Get your hands on all the sections [here](../README.md).

© 2024 [Shiv Katira](https://github.com/shivkatira)