# 🪝 4. Campaigns Analysis
<p align="center">
<img src="../../img/6.png" align="center" width="400" height="400" >

## 📚 Table of Contents

* [Case Study Questions](#-case-study-questions)
* [My Solution](#-my-solution)
* [Case Study #6 - Clique Bait](#-case-study-6---clique-bait)

## ❓ Case Study Questions

* [Generate a table that has 1 single row for every unique visit_id record and has the following columns:](#generate-a-table-that-has-1-single-row-for-every-unique-visit_id-record-and-has-the-following-columns)
    - user_id
    - visit_id
    - visit_start_time: the earliest event_time for each visit
    - page_views: count of page views for each visit
    - cart_adds: count of product cart add events for each visit
    - purchase: 1/0 flag if a purchase event exists for each visit
    - campaign_name: map the visit to a campaign if the visit_start_time falls between the start_date and end_date
    - impression: count of ad impressions for each visit
    - click: count of ad clicks for each visit
    - (Optional column) cart_products: a comma separated text value with products added to the cart sorted by the order they were added to the cart (hint: use the sequence_number)

* [Identifying users who have received impressions during each campaign period and comparing each metric with other users who did not have an impression event](#identifying-users-who-have-received-impressions-during-each-campaign-period-and-comparing-each-metric-with-other-users-who-did-not-have-an-impression-event)
* [Does clicking on an impression lead to higher purchase rates?](#does-clicking-on-an-impression-lead-to-higher-purchase-rates)
* [What is the uplift in purchase rate when comparing users who click on a campaign impression versus users who do not receive an impression? What if we compare them with users who just an impression but do not click?](#what-is-the-uplift-in-purchase-rate-when-comparing-users-who-click-on-a-campaign-impression-versus-users-who-do-not-receive-an-impression-what-if-we-compare-them-with-users-who-just-an-impression-but-do-not-click)
* [What metrics can you use to quantify the success or failure of each campaign compared to eachother?](#what-metrics-can-you-use-to-quantify-the-success-or-failure-of-each-campaign-compared-to-eachother)

## 💡 My Solution

### Generate a table that has 1 single row for every unique visit_id record and has the following columns:
- user_id
- visit_id
- visit_start_time: the earliest event_time for each visit
- page_views: count of page views for each visit
- cart_adds: count of product cart add events for each visit
- purchase: 1/0 flag if a purchase event exists for each visit
- campaign_name: map the visit to a campaign if the visit_start_time falls between the start_date and end_date
- impression: count of ad impressions for each visit
- click: count of ad clicks for each visit
- (Optional column) cart_products: a comma separated text value with products added to the cart sorted by the order they were added to the cart (hint: use the sequence_number)

```SQL
CREATE TABLE clique_bait.campaign_analysis AS (
    SELECT 
        user_id, -- Select user ID
        visit_id, -- Select visit ID
        MIN(event_time) AS visit_start_time, -- Get the earliest event time for each visit
        SUM(CASE WHEN event_type = 1 THEN 1 ELSE 0 END) AS page_views, -- Count of page views for each visit
        SUM(CASE WHEN event_type = 2 THEN 1 ELSE 0 END) AS cart_adds, -- Count of product cart add events for each visit
        SUM(CASE WHEN event_type = 3 THEN 1 ELSE 0 END) AS purchase, -- 1/0 flag if a purchase event exists for each visit
        campaign_name, -- Map the visit to a campaign if the visit_start_time falls between the start_date and end_date
        SUM(CASE WHEN event_type = 4 THEN 1 ELSE 0 END) AS impression, -- Count of ad impressions for each visit
        SUM(CASE WHEN event_type = 5 THEN 1 ELSE 0 END) AS click, -- Count of ad clicks for each visit
        STRING_AGG(CASE WHEN event_type = 2 THEN page_name END, ', ' ORDER BY sequence_number) AS cart_products -- Comma-separated text value with products added to the cart sorted by the order they were added to the cart
    FROM 
        clique_bait.users -- Use users table
        LEFT JOIN clique_bait.events USING (cookie_id) -- Join with events table using cookie ID
        LEFT JOIN clique_bait.campaign_identifier ON event_time BETWEEN clique_bait.campaign_identifier.start_date AND clique_bait.campaign_identifier.end_date -- Join with campaign_identifier table to map campaigns
        LEFT JOIN clique_bait.page_hierarchy USING (page_id) -- Join with page_hierarchy table using page ID
    GROUP BY 
        user_id, visit_id, campaign_name -- Group results by user ID, visit ID, and campaign name
    ORDER BY 
        user_id, visit_id, campaign_name -- Sort results by user ID, visit ID, and campaign name for organized presentation
);

SELECT * FROM clique_bait.campaign_analysis;
```

| user_id | visit_id | visit_start_time         | page_views | cart_adds | purchase | campaign_name                     | impression | click | cart_products                                                                         |
| ------- | -------- | ------------------------ | ---------- | --------- | -------- | --------------------------------- | ---------- | ----- | ------------------------------------------------------------------------------------- |
| 1       | 02a5d5   | 2020-02-26T16:57:26.260Z | 4          | 0         | 0        | Half Off - Treat Your Shellf(ish) | 0          | 0     | null                                                                                  |
| 1       | 0826dc   | 2020-02-26T05:58:37.918Z | 1          | 0         | 0        | Half Off - Treat Your Shellf(ish) | 0          | 0     | null                                                                                  |
| 1       | 0fc437   | 2020-02-04T17:49:49.602Z | 10         | 6         | 1        | Half Off - Treat Your Shellf(ish) | 1          | 1     | Tuna, Russian Caviar, Black Truffle, Abalone, Crab, Oyster                            |
| 1       | 30b94d   | 2020-03-15T13:12:54.023Z | 9          | 7         | 1        | Half Off - Treat Your Shellf(ish) | 1          | 1     | Salmon, Kingfish, Tuna, Russian Caviar, Abalone, Lobster, Crab                        |
| 1       | 41355d   | 2020-03-25T00:11:17.860Z | 6          | 1         | 0        | Half Off - Treat Your Shellf(ish) | 0          | 0     | Lobster                                                                               |
| 1       | ccf365   | 2020-02-04T19:16:09.182Z | 7          | 3         | 1        | Half Off - Treat Your Shellf(ish) | 0          | 0     | Lobster, Crab, Oyster                                                                 |
| 1       | eaffde   | 2020-03-25T20:06:32.342Z | 10         | 8         | 1        | Half Off - Treat Your Shellf(ish) | 1          | 1     | Salmon, Tuna, Russian Caviar, Black Truffle, Abalone, Lobster, Crab, Oyster           |
| 1       | f7c798   | 2020-03-15T02:23:26.312Z | 9          | 3         | 1        | Half Off - Treat Your Shellf(ish) | 0          | 0     | Russian Caviar, Crab, Oyster                                                          |
| 2       | 0635fb   | 2020-02-16T06:42:42.735Z | 9          | 4         | 1        | Half Off - Treat Your Shellf(ish) | 0          | 0     | Salmon, Kingfish, Abalone, Crab                                                       |
| 2       | 1f1198   | 2020-02-01T21:51:55.078Z | 1          | 0         | 0        | Half Off - Treat Your Shellf(ish) | 0          | 0     | null                                                                                  |

**Attention:**
- Because presenting the entire output is impractical, the above table only displays the first 10 rows.

### Identifying users who have received impressions during each campaign period and comparing each metric with other users who did not have an impression event

```SQL
WITH without_impression_melted_cte AS (
    SELECT 
        campaign_name, -- Select campaign name
        UNNEST(ARRAY[ '1. average_page_views', '2. average_views_to_cart_rate', '3. average_cart_add_to_purchase_rate', '4. average_cart_abandonment_rate' ]) AS metrics_name, -- Unnest metric names
        UNNEST(ARRAY[ 
            page_views, 
            ROUND(cart_adds * 100.0 / page_views, 2), 
            CASE WHEN cart_adds <> 0 THEN ROUND(purchase * 100.0 / cart_adds, 2) ELSE 0 END, 
            CASE WHEN cart_adds <> 0 THEN 100 - ROUND(purchase * 100.0 / cart_adds, 2) ELSE 0 END 
        ]) AS metrics_value -- Unnest metrics values
    FROM 
        clique_bait.campaign_analysis -- Use campaign_analysis table
    WHERE 
        impression = 0 -- Filter for no impressions
        AND campaign_name IS NOT NULL -- Ensure campaign name is not null
    ORDER BY 
        campaign_name, metrics_name -- Order by campaign name and metrics name
),
without_impression_cte AS (
    SELECT 
        campaign_name, -- Select campaign name
        metrics_name, -- Select metrics name
        ROUND(AVG(metrics_value), 2) AS without_impression -- Calculate average metrics value for users without impressions
    FROM 
        without_impression_melted_cte -- Use the CTE for users without impressions
    GROUP BY 
        campaign_name, metrics_name -- Group by campaign name and metrics name
    ORDER BY 
        campaign_name -- Order by campaign name
),
with_impression_melted_cte AS (
    SELECT 
        campaign_name, -- Select campaign name
        UNNEST(ARRAY[ '1. average_page_views', '2. average_views_to_cart_rate', '3. average_cart_add_to_purchase_rate', '4. average_cart_abandonment_rate' ]) AS metrics_name, -- Unnest metric names
        UNNEST(ARRAY[ 
            page_views, 
            ROUND(cart_adds * 100.0 / page_views, 2), 
            CASE WHEN cart_adds <> 0 THEN ROUND(purchase * 100.0 / cart_adds, 2) ELSE 0 END, 
            CASE WHEN cart_adds <> 0 THEN 100 - ROUND(purchase * 100.0 / cart_adds, 2) ELSE 0 END 
        ]) AS metrics_value -- Unnest metrics values
    FROM 
        clique_bait.campaign_analysis -- Use campaign_analysis table
    WHERE 
        impression > 0 -- Filter for users with impressions
        AND campaign_name IS NOT NULL -- Ensure campaign name is not null
    ORDER BY 
        campaign_name, metrics_name -- Order by campaign name and metrics name
),
with_impression_cte AS (
    SELECT 
        campaign_name, -- Select campaign name
        metrics_name, -- Select metrics name
        ROUND(AVG(metrics_value), 2) AS with_impression -- Calculate average metrics value for users with impressions
    FROM 
        with_impression_melted_cte -- Use the CTE for users with impressions
    GROUP BY 
        campaign_name, metrics_name -- Group by campaign name and metrics name
    ORDER BY 
        campaign_name -- Order by campaign name
)
SELECT 
    *, -- Select all columns
    with_impression - without_impression AS absolute_effectiveness, -- Calculate absolute effectiveness
    ROUND((with_impression - without_impression) * 100.0 / without_impression, 2) AS percentage_effectiveness -- Calculate percentage effectiveness
FROM 
    without_impression_cte -- Use the CTE for users without impressions
    LEFT JOIN with_impression_cte USING (campaign_name, metrics_name); -- Join with the CTE for users with impressions
```

| campaign_name                     | metrics_name                         | without_impression | with_impression | absolute_effectiveness | percentage_effectiveness |
| --------------------------------- | ------------------------------------ | ------------------ | --------------- | ---------------------- | ------------------------ |
| 25% Off - Living The Lux Life     | 1. average_page_views                | 5.12               | 8.64            | 3.52                   | 68.75                    |
| 25% Off - Living The Lux Life     | 2. average_views_to_cart_rate        | 20.98              | 57.12           | 36.14                  | 172.26                   |
| 25% Off - Living The Lux Life     | 3. average_cart_add_to_purchase_rate | 19.00              | 20.60           | 1.60                   | 8.42                     |
| 25% Off - Living The Lux Life     | 4. average_cart_abandonment_rate     | 42.00              | 79.40           | 37.40                  | 89.05                    |
| BOGOF - Fishing For Compliments   | 1. average_page_views                | 4.95               | 8.77            | 3.82                   | 77.17                    |
| BOGOF - Fishing For Compliments   | 2. average_views_to_cart_rate        | 19.42              | 59.86           | 40.44                  | 208.24                   |
| BOGOF - Fishing For Compliments   | 3. average_cart_add_to_purchase_rate | 19.14              | 17.09           | -2.05                  | -10.71                   |
| BOGOF - Fishing For Compliments   | 4. average_cart_abandonment_rate     | 39.84              | 82.91           | 43.07                  | 108.11                   |
| Half Off - Treat Your Shellf(ish) | 1. average_page_views                | 4.96               | 8.51            | 3.55                   | 71.57                    |
| Half Off - Treat Your Shellf(ish) | 2. average_views_to_cart_rate        | 20.67              | 56.95           | 36.28                  | 175.52                   |
| Half Off - Treat Your Shellf(ish) | 3. average_cart_add_to_purchase_rate | 19.66              | 21.17           | 1.51                   | 7.68                     |
| Half Off - Treat Your Shellf(ish) | 4. average_cart_abandonment_rate     | 41.45              | 77.27           | 35.82                  | 86.42                    |

### Does clicking on an impression lead to higher purchase rates?

```SQL
SELECT 
    ROUND(
        SUM(CASE WHEN click = 0 THEN purchase ELSE 0 END) * 100.0 / 
        SUM(CASE WHEN click = 0 THEN page_views ELSE 0 END), 2
    ) AS purchase_rate_without_click, -- Calculate purchase rate without clicks
    ROUND(
        SUM(CASE WHEN click = 1 THEN purchase ELSE 0 END) * 100.0 / 
        SUM(CASE WHEN click = 1 THEN page_views ELSE 0 END), 2
    ) AS purchase_rate_with_click -- Calculate purchase rate with clicks
FROM 
    clique_bait.campaign_analysis; -- Use campaign_analysis table for data
```

| purchase_rate_without_click | purchase_rate_with_click |
| --------------------------- | ------------------------ |
| 7.92                        | 9.80                     |

### What is the uplift in purchase rate when comparing users who click on a campaign impression versus users who do not receive an impression? What if we compare them with users who just an impression but do not click?

```SQL
-- Calculate purchase rate upliftment when comparing users who click on a campaign impression versus users who do not receive an impression
WITH cte AS (
    SELECT 
        ROUND(
            SUM(CASE WHEN impression = 1 AND click = 1 THEN purchase ELSE 0 END) * 100.0 / 
            SUM(CASE WHEN impression = 1 AND click = 1 THEN page_views ELSE 0 END), 2
        ) AS purchase_rate_with_impression_with_click, -- Calculate purchase rate with impression and click
        ROUND(
            SUM(CASE WHEN impression = 0 AND click = 0 THEN purchase ELSE 0 END) * 100.0 / 
            SUM(CASE WHEN impression = 0 AND click = 0 THEN page_views ELSE 0 END), 2
        ) AS purchase_rate_without_impression_without_click -- Calculate purchase rate without impression and click
    FROM 
        clique_bait.campaign_analysis
)
SELECT 
    purchase_rate_without_impression_without_click, -- Select purchase rate without impression and click
    purchase_rate_with_impression_with_click, -- Select purchase rate with impression and click
    purchase_rate_with_impression_with_click - purchase_rate_without_impression_without_click AS absolute_purchase_rate_upliftment, -- Calculate absolute purchase rate upliftment
    ROUND(
        (purchase_rate_with_impression_with_click - purchase_rate_without_impression_without_click) * 100.0 / 
        purchase_rate_without_impression_without_click, 2
    ) AS percentage_purchase_rate_upliftment -- Calculate percentage purchase rate upliftment
FROM 
    no_impression_click_cte;
```

| purchase_rate_without_impression_without_click | purchase_rate_with_impression_with_click | absolute_purchase_rate_upliftment | percentage_purchase_rate_upliftment |
| ---------------------------------------------- | ---------------------------------------- | --------------------------------- | ----------------------------------- |
| 7.74                                           | 9.80                                     | 2.06                              | 26.61                               |

```SQL
-- Calculate purchase rate upliftment when comparing users who click on a campaign impression versus users who just have an impression but do not click
WITH cte AS (
    SELECT 
        ROUND(
            SUM(CASE WHEN impression = 1 AND click = 1 THEN purchase ELSE 0 END) * 100.0 / 
            SUM(CASE WHEN impression = 1 AND click = 1 THEN page_views ELSE 0 END), 2
        ) AS purchase_rate_with_impression_with_click, -- Calculate purchase rate with impression and click
        ROUND(
            SUM(CASE WHEN impression = 1 AND click = 0 THEN purchase ELSE 0 END) * 100.0 / 
            SUM(CASE WHEN impression = 1 AND click = 0 THEN page_views ELSE 0 END), 2
        ) AS purchase_rate_with_impression_without_click -- Calculate purchase rate with impression without click
    FROM 
        clique_bait.campaign_analysis
)
SELECT 
    purchase_rate_with_impression_without_click, -- Select purchase rate with impression without click
    purchase_rate_with_impression_with_click, -- Select purchase rate with impression and click
    purchase_rate_with_impression_with_click - purchase_rate_with_impression_without_click AS absolute_purchase_rate_upliftment, -- Calculate absolute purchase rate upliftment
    ROUND(
        (purchase_rate_with_impression_with_click - purchase_rate_with_impression_without_click) * 100.0 / 
        purchase_rate_with_impression_without_click, 2
    ) AS percentage_purchase_rate_upliftment -- Calculate percentage purchase rate upliftment
FROM 
    impression_click_comparison_cte;
```

| purchase_rate_with_impression_without_click | purchase_rate_with_impression_with_click | absolute_purchase_rate_upliftment | percentage_purchase_rate_upliftment |
| ------------------------------------------- | ---------------------------------------- | --------------------------------- | ----------------------------------- |
| 10.13                                       | 9.80                                     | -0.33                             | -3.26                               |

### What metrics can you use to quantify the success or failure of each campaign compared to eachother?

```SQL
SELECT 
    campaign_name, -- Select campaign name
    ROUND(SUM(click) * 100.0 / SUM(impression), 2) AS impression_click_rate, -- Calculate impression to click rate
    ROUND(SUM(CASE WHEN click > 0 THEN cart_adds ELSE 0 END) * 100.0 / SUM(CASE WHEN click > 0 THEN page_views ELSE 0 END), 2) AS cart_add_rate, -- Calculate cart add rate for users who clicked
    ROUND(SUM(CASE WHEN click > 0 THEN purchase ELSE 0 END) * 100.0 / SUM(CASE WHEN click > 0 THEN cart_adds ELSE 0 END), 2) AS cart_add_to_purchase_rate, -- Calculate cart add to purchase rate for users who clicked
    100 - ROUND(SUM(CASE WHEN click > 0 THEN purchase ELSE 0 END) * 100.0 / SUM(CASE WHEN click > 0 THEN cart_adds ELSE 0 END), 2) AS cart_abandonment_rate -- Calculate cart abandonment rate for users who clicked
FROM 
    clique_bait.campaign_analysis -- Use campaign_analysis table
WHERE 
    campaign_name IS NOT NULL -- Ensure campaign name is not null
GROUP BY 
    campaign_name -- Group by campaign name
ORDER BY 
    campaign_name; -- Order by campaign name
```

| campaign_name                     | impression_click_rate | cart_add_rate | cart_add_to_purchase_rate | cart_abandonment_rate |
| --------------------------------- | --------------------- | ------------- | ------------------------- | --------------------- |
| 25% Off - Living The Lux Life     | 77.88                 | 62.78         | 14.98                     | 85.02                 |
| BOGOF - Fishing For Compliments   | 84.62                 | 64.27         | 15.53                     | 84.47                 |
| Half Off - Treat Your Shellf(ish) | 80.10                 | 62.92         | 15.83                     | 84.17                 |

## 🪝 Case Study #6 - Clique Bait

Curious for more? Get your hands on all the sections [here](../README.md).

© 2024 [Shiv Katira](https://github.com/shivkatira)