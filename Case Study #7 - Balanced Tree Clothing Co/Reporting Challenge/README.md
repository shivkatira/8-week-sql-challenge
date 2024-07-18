# 🏔️ Reporting Challenge
<p align="center">
<img src="../../img/7.png" align="center" width="400" height="400" >

## 📚 Table of Contents

* [Case Study Questions](#-case-study-questions)
* [My Solution](#-my-solution)
* [Case Study #7 - Balanced Tree Clothing Co](#️-case-study-7---balanced-tree-clothing-co)

## ❓ Case Study Questions

* [Write a single SQL script that combines all of the previous questions into a scheduled report that the Balanced Tree team can run at the beginning of each month to calculate the previous month’s values.](#write-a-single-sql-script-that-combines-all-of-the-previous-questions-into-a-scheduled-report-that-the-balanced-tree-team-can-run-at-the-beginning-of-each-month-to-calculate-the-previous-months-values)

## 💡 My Solution

### Write a single SQL script that combines all of the previous questions into a scheduled report that the Balanced Tree team can run at the beginning of each month to calculate the previous month’s values.

```SQL
-- Drop the table if it exists and create a new one for the month under analysis
DROP TABLE IF EXISTS month_under_analysis;
CREATE TABLE month_under_analysis (month_value INT, year_value INT);
INSERT INTO month_under_analysis (month_value, year_value) VALUES (1, 2021);
```

**Attention:**
  - Replace `1` and `2021` with the desired `month_value` and `year_value` to produce the insights shown below for that particular month.

```SQL
-- Common Table Expressions (CTEs) for transaction analysis
WITH txn_cte AS (
    SELECT 
        txn_id, 
        SUM(qty) AS qty_per_txn, 
        COUNT(DISTINCT prod_id) AS unique_product_count_per_txn, 
        SUM(qty * price) AS revenue_per_txn, 
        ROUND(SUM(qty * price * discount / 100.0), 2) AS discount_per_txn 
    FROM 
        balanced_tree.sales, 
        month_under_analysis 
    WHERE 
        EXTRACT(MONTH FROM start_txn_time) = month_value AND EXTRACT(YEAR FROM start_txn_time) = year_value 
    GROUP BY 
        txn_id
)
-- Select various transaction metrics
SELECT 
    SUM(qty_per_txn) AS total_qty, 
    SUM(revenue_per_txn) AS total_revenue, 
    SUM(discount_per_txn) AS total_discount, 
    ROUND(AVG(unique_product_count_per_txn), 2) AS avg_unique_product_count, 
    ROUND(AVG(discount_per_txn), 2) AS avg_discount_per_txn, 
    COUNT(DISTINCT txn_id) AS unique_txn, 
    PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY revenue_per_txn) AS percentile_25, 
    PERCENTILE_CONT(0.50) WITHIN GROUP (ORDER BY revenue_per_txn) AS percentile_50, 
    PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY revenue_per_txn) AS percentile_75 
FROM 
    txn_cte;
```

| total_qty | total_revenue | total_discount | avg_unique_product_count | avg_discount_per_txn | unique_txn | percentile_25 | percentile_50 | percentile_75 |
| --------- | ------------- | -------------- | ------------------------ | -------------------- | ---------- | ------------- | ------------- | ------------- |
| 14788     | 420672        | 51589.10       | 5.99                     | 62.31                | 828        | 359           | 496.5         | 645.25        |

```SQL
-- Common Table Expressions (CTEs) for revenue per transaction analysis
WITH revenue_per_txn_cte AS (
    SELECT 
        member, 
        txn_id, 
        SUM(price * qty) AS revenue 
    FROM 
        balanced_tree.sales, 
        month_under_analysis 
    WHERE 
        EXTRACT(MONTH FROM start_txn_time) = month_value AND EXTRACT(YEAR FROM start_txn_time) = year_value 
    GROUP BY 
        member, txn_id
)
-- Select average revenue and transaction contribution percentage
SELECT 
    member, 
    ROUND(AVG(revenue), 2) AS avg_revenue, 
    ROUND(COUNT(DISTINCT txn_id) * 100.0 / (SELECT COUNT(DISTINCT txn_id) FROM revenue_per_txn_cte), 2) AS txn_contribution_pct 
FROM 
    revenue_per_txn_cte 
GROUP BY 
    member 
ORDER BY 
    member DESC;
```

| member | avg_revenue | txn_contribution_pct |
| ------ | ----------- | -------------------- |
| true   | 515.93      | 59.42                |
| false  | 496.53      | 40.58                |

```SQL
-- Common Table Expressions (CTEs) for product analysis
WITH product_analysis_cte AS (
    SELECT 
        category_name, 
        segment_name, 
        product_name, 
        SUM(qty) AS total_qty, 
        SUM(qty * product_details.price) AS total_revenue, 
        ROUND(SUM(qty * product_details.price * discount / 100.0), 2) AS total_discount, 
        ROUND(SUM(CASE WHEN product_id = prod_id THEN 1 ELSE 0 END) * 100.0 / (SELECT COUNT(DISTINCT txn_id) FROM balanced_tree.sales), 2) AS penetration 
    FROM 
        month_under_analysis, 
        balanced_tree.product_details 
    LEFT JOIN 
        balanced_tree.sales ON product_id = prod_id 
    WHERE 
        EXTRACT(MONTH FROM start_txn_time) = month_value AND EXTRACT(YEAR FROM start_txn_time) = year_value 
    GROUP BY 
        category_name, segment_name, product_name 
    ORDER BY 
        category_name, segment_name, product_name
)
-- Select various product analysis metrics
SELECT 
    category_name, 
    segment_name, 
    product_name, 
    total_qty, 
    ROUND(total_qty / (SELECT SUM(total_qty) FROM product_analysis_cte) * 100.0, 2) AS total_qty_pct_contribution, 
    DENSE_RANK() OVER (ORDER BY total_qty DESC ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS total_qty_ranking, 
    total_revenue, 
    ROUND(total_revenue / (SELECT SUM(total_revenue) FROM product_analysis_cte) * 100.0, 2) AS total_revenue_pct_contribution, 
    DENSE_RANK() OVER (ORDER BY total_revenue DESC ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS total_revenue_ranking, 
    total_discount, 
    ROUND(total_discount / (SELECT SUM(total_discount) FROM product_analysis_cte) * 100.0, 2) AS total_discount_pct_contribution, 
    DENSE_RANK() OVER (ORDER BY total_discount DESC ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS total_discount_ranking, 
    penetration, 
    ROUND(penetration / (SELECT SUM(penetration) FROM product_analysis_cte) * 100.0, 2) AS penetration_pct_contribution, 
    DENSE_RANK() OVER (ORDER BY penetration DESC ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS penetration_ranking 
FROM 
    product_analysis_cte 
ORDER BY 
    category_name, segment_name, product_name;
```

| category_name | segment_name | product_name                     | total_qty | total_qty_pct_contribution | total_qty_ranking | total_revenue | total_revenue_pct_contribution | total_revenue_ranking | total_discount | total_discount_pct_contribution | total_discount_ranking | penetration | penetration_pct_contribution | penetration_ranking |
| ------------- | ------------ | -------------------------------- | --------- | -------------------------- | ----------------- | ------------- | ------------------------------ | --------------------- | -------------- | ------------------------------- | ---------------------- | ----------- | ---------------------------- | ------------------- |
| Mens          | Shirt        | Blue Polo Shirt - Mens           | 1214      | 8.21                       | 9                 | 69198         | 16.45                          | 2                     | 8523.78        | 16.52                           | 2                      | 16.52       | 8.33                         | 6                   |
| Mens          | Shirt        | Teal Button Up Shirt - Mens      | 1220      | 8.25                       | 8                 | 12200         | 2.90                           | 12                    | 1539.00        | 2.98                            | 12                     | 16.44       | 8.29                         | 7                   |
| Mens          | Shirt        | White Tee Shirt - Mens           | 1256      | 8.49                       | 5                 | 50240         | 11.94                          | 3                     | 6165.60        | 11.95                           | 3                      | 16.64       | 8.39                         | 5                   |
| Mens          | Socks        | Navy Solid Socks - Mens          | 1264      | 8.55                       | 3                 | 45504         | 10.82                          | 4                     | 5557.32        | 10.77                           | 4                      | 16.80       | 8.47                         | 4                   |
| Mens          | Socks        | Pink Fluro Polkadot Socks - Mens | 1157      | 7.82                       | 10                | 33553         | 7.98                           | 6                     | 4091.61        | 7.93                            | 6                      | 15.84       | 7.99                         | 12                  |
| Mens          | Socks        | White Striped Socks - Mens       | 1150      | 7.78                       | 11                | 19550         | 4.65                           | 9                     | 2357.73        | 4.57                            | 9                      | 15.96       | 8.05                         | 11                  |
| Womens        | Jacket       | Grey Fashion Jacket - Womens     | 1300      | 8.79                       | 1                 | 70200         | 16.69                          | 1                     | 8580.60        | 16.63                           | 1                      | 17.24       | 8.69                         | 2                   |
| Womens        | Jacket       | Indigo Rain Jacket - Womens      | 1225      | 8.28                       | 7                 | 23275         | 5.53                           | 8                     | 2852.28        | 5.53                            | 8                      | 16.28       | 8.21                         | 9                   |
| Womens        | Jacket       | Khaki Suit Jacket - Womens       | 1225      | 8.28                       | 7                 | 28175         | 6.70                           | 7                     | 3438.50        | 6.67                            | 7                      | 16.08       | 8.11                         | 10                  |
| Womens        | Jeans        | Black Straight Jeans - Womens    | 1238      | 8.37                       | 6                 | 39616         | 9.42                           | 5                     | 4863.04        | 9.43                            | 5                      | 16.32       | 8.23                         | 8                   |
| Womens        | Jeans        | Cream Relaxed Jeans - Womens     | 1282      | 8.67                       | 2                 | 12820         | 3.05                           | 11                    | 1595.80        | 3.09                            | 11                     | 17.28       | 8.71                         | 1                   |
| Womens        | Jeans        | Navy Oversized Jeans - Womens    | 1257      | 8.50                       | 4                 | 16341         | 3.88                           | 10                    | 2023.84        | 3.92                            | 10                     | 16.92       | 8.53                         | 3                   |

```SQL
-- Common Table Expressions (CTEs) for revenue by segment analysis
WITH total_revenue_by_segment_cte AS (
    SELECT 
        category_name, 
        segment_name, 
        SUM(sales.price * qty) AS total_revenue 
    FROM 
        month_under_analysis, 
        balanced_tree.product_details 
    LEFT JOIN 
        balanced_tree.sales ON product_details.product_id = sales.prod_id 
    WHERE 
        EXTRACT(MONTH FROM start_txn_time) = month_value AND EXTRACT(YEAR FROM start_txn_time) = year_value 
    GROUP BY 
        category_name, segment_name
)
-- Select revenue percentage by segment
SELECT 
    category_name, 
    segment_name, 
    product_name, 
    ROUND(SUM(sales.price * qty) * 100.0 / total_revenue, 2) AS revenue_pct 
FROM 
    month_under_analysis, 
    balanced_tree.product_details 
LEFT JOIN 
    balanced_tree.sales ON product_details.product_id = sales.prod_id 
LEFT JOIN 
    total_revenue_by_segment_cte USING (category_name, segment_name) 
WHERE 
    EXTRACT(MONTH FROM start_txn_time) = month_value AND EXTRACT(YEAR FROM start_txn_time) = year_value 
GROUP BY 
    category_name, segment_name, total_revenue, product_name 
ORDER BY 
    category_name, segment_name, revenue_pct DESC;
```

| category_name | segment_name | product_name                     | revenue_pct |
| ------------- | ------------ | -------------------------------- | ----------- |
| Mens          | Shirt        | Blue Polo Shirt - Mens           | 52.57       |
| Mens          | Shirt        | White Tee Shirt - Mens           | 38.17       |
| Mens          | Shirt        | Teal Button Up Shirt - Mens      | 9.27        |
| Mens          | Socks        | Navy Solid Socks - Mens          | 46.15       |
| Mens          | Socks        | Pink Fluro Polkadot Socks - Mens | 34.03       |
| Mens          | Socks        | White Striped Socks - Mens       | 19.83       |
| Womens        | Jacket       | Grey Fashion Jacket - Womens     | 57.71       |
| Womens        | Jacket       | Khaki Suit Jacket - Womens       | 23.16       |
| Womens        | Jacket       | Indigo Rain Jacket - Womens      | 19.13       |
| Womens        | Jeans        | Black Straight Jeans - Womens    | 57.60       |
| Womens        | Jeans        | Navy Oversized Jeans - Womens    | 23.76       |
| Womens        | Jeans        | Cream Relaxed Jeans - Womens     | 18.64       |

```SQL
-- Common Table Expressions (CTEs) for segment analysis
WITH segment_analysis_cte AS (
    SELECT 
        category_name, 
        segment_name, 
        product_name, 
        DENSE_RANK() OVER (PARTITION BY segment_name ORDER BY SUM(qty) DESC ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS ranking, 
        SUM(qty) AS total_qty, 
        SUM(qty * product_details.price) AS total_revenue, 
        ROUND(SUM((discount * qty * product_details.price) / 100.0), 2) AS total_discount 
    FROM 
        month_under_analysis, 
        balanced_tree.product_details 
    LEFT JOIN 
        balanced_tree.sales ON product_id = prod_id 
    WHERE 
        EXTRACT(MONTH FROM start_txn_time) = month_value AND EXTRACT(YEAR FROM start_txn_time) = year_value 
    GROUP BY 
        category_name, segment_name, product_name 
    ORDER BY 
        category_name, segment_name, product_name
),
-- Select top selling product per segment
top_selling_product_cte AS (
    SELECT 
        category_name, 
        segment_name, 
        product_name AS top_selling_product 
    FROM 
        segment_analysis_cte 
    WHERE 
        ranking = 1
),
-- Select segment performance metrics
segment_performance_cte AS (
    SELECT 
        category_name, 
        segment_name, 
        SUM(total_qty) AS total_qty, 
        SUM(total_revenue) AS total_revenue, 
        SUM(total_discount) AS total_discount 
    FROM 
        segment_analysis_cte 
    GROUP BY 
        category_name, segment_name
)
-- Select various segment performance metrics
SELECT 
    category_name, 
    segment_name, 
    total_qty, 
    ROUND(total_qty * 100.0 / (SELECT SUM(total_qty) FROM segment_performance_cte), 2) AS total_qty_pct_contribution, 
    DENSE_RANK() OVER (ORDER BY total_qty DESC ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS total_qty_ranking, 
    total_revenue, 
    ROUND(total_revenue * 100.0 / (SELECT SUM(total_revenue) FROM segment_performance_cte), 2) AS total_revenue_pct_contribution, 
    DENSE_RANK() OVER (ORDER BY total_revenue DESC ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS total_revenue_ranking, 
    total_discount, 
    ROUND(total_discount * 100.0 / (SELECT SUM(total_discount) FROM segment_performance_cte), 2) AS total_discount_pct_contribution, 
    DENSE_RANK() OVER (ORDER BY total_discount DESC ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS total_discount_ranking, 
    top_selling_product 
FROM 
    segment_performance_cte 
LEFT JOIN 
    top_selling_product_cte USING (category_name, segment_name) 
ORDER BY 
    category_name, segment_name;
```

| category_name | segment_name | total_qty | total_qty_pct_contribution | total_qty_ranking | total_revenue | total_revenue_pct_contribution | total_revenue_ranking | total_discount | total_discount_pct_contribution | total_discount_ranking | top_selling_product          |
| ------------- | ------------ | --------- | -------------------------- | ----------------- | ------------- | ------------------------------ | --------------------- | -------------- | ------------------------------- | ---------------------- | ---------------------------- |
| Mens          | Shirt        | 3690      | 24.95                      | 3                 | 131638        | 31.29                          | 1                     | 16228.38       | 31.46                           | 1                      | White Tee Shirt - Mens       |
| Mens          | Socks        | 3571      | 24.15                      | 4                 | 98607         | 23.44                          | 3                     | 12006.66       | 23.27                           | 3                      | Navy Solid Socks - Mens      |
| Womens        | Jacket       | 3750      | 25.36                      | 2                 | 121650        | 28.92                          | 2                     | 14871.38       | 28.83                           | 2                      | Grey Fashion Jacket - Womens |
| Womens        | Jeans        | 3777      | 25.54                      | 1                 | 68777         | 16.35                          | 4                     | 8482.68        | 16.44                           | 4                      | Cream Relaxed Jeans - Womens |

```SQL
-- Common Table Expressions (CTEs) for category analysis
WITH category_analysis_cte AS

 (
    SELECT 
        category_name, 
        product_name, 
        DENSE_RANK() OVER (PARTITION BY category_name ORDER BY SUM(qty) DESC ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS ranking, 
        SUM(qty) AS total_qty, 
        SUM(qty * product_details.price) AS total_revenue, 
        ROUND(SUM((discount * qty * product_details.price) / 100.0), 2) AS total_discount 
    FROM 
        month_under_analysis, 
        balanced_tree.product_details 
    LEFT JOIN 
        balanced_tree.sales ON product_id = prod_id 
    WHERE 
        EXTRACT(MONTH FROM start_txn_time) = month_value AND EXTRACT(YEAR FROM start_txn_time) = year_value 
    GROUP BY 
        category_name, product_name 
    ORDER BY 
        category_name, product_name
),
-- Select top selling product per category
top_selling_product_cte AS (
    SELECT 
        category_name, 
        product_name AS top_selling_product 
    FROM 
        category_analysis_cte 
    WHERE 
        ranking = 1
),
-- Select category performance metrics
category_performance_cte AS (
    SELECT 
        category_name, 
        SUM(total_qty) AS total_qty, 
        SUM(total_revenue) AS total_revenue, 
        SUM(total_discount) AS total_discount 
    FROM 
        category_analysis_cte 
    GROUP BY 
        category_name
)
-- Select various category performance metrics
SELECT 
    category_name, 
    total_qty, 
    ROUND(total_qty * 100.0 / (SELECT SUM(total_qty) FROM category_performance_cte), 2) AS total_qty_pct_contribution, 
    DENSE_RANK() OVER (ORDER BY total_qty DESC ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS total_qty_ranking, 
    total_revenue, 
    ROUND(total_revenue * 100.0 / (SELECT SUM(total_revenue) FROM category_performance_cte), 2) AS total_revenue_pct_contribution, 
    DENSE_RANK() OVER (ORDER BY total_revenue DESC ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS total_revenue_ranking, 
    total_discount, 
    ROUND(total_discount * 100.0 / (SELECT SUM(total_discount) FROM category_performance_cte), 2) AS total_discount_pct_contribution, 
    DENSE_RANK() OVER (ORDER BY total_discount DESC ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS total_discount_ranking, 
    top_selling_product 
FROM 
    category_performance_cte 
LEFT JOIN 
    top_selling_product_cte USING (category_name) 
ORDER BY 
    category_name;
```

| category_name | total_qty | total_qty_pct_contribution | total_qty_ranking | total_revenue | total_revenue_pct_contribution | total_revenue_ranking | total_discount | total_discount_pct_contribution | total_discount_ranking | top_selling_product          |
| ------------- | --------- | -------------------------- | ----------------- | ------------- | ------------------------------ | --------------------- | -------------- | ------------------------------- | ---------------------- | ---------------------------- |
| Mens          | 7261      | 49.10                      | 2                 | 230245        | 54.73                          | 1                     | 28235.04       | 54.73                           | 1                      | Navy Solid Socks - Mens      |
| Womens        | 7527      | 50.90                      | 1                 | 190427        | 45.27                          | 2                     | 23354.06       | 45.27                           | 2                      | Grey Fashion Jacket - Womens |

```SQL
-- Common Table Expressions (CTEs) for revenue by category analysis
WITH total_revenue_by_category_cte AS (
    SELECT 
        category_name, 
        SUM(sales.price * qty) AS total_revenue 
    FROM 
        month_under_analysis, 
        balanced_tree.product_details 
    LEFT JOIN 
        balanced_tree.sales ON product_details.product_id = sales.prod_id 
    WHERE 
        EXTRACT(MONTH FROM start_txn_time) = month_value AND EXTRACT(YEAR FROM start_txn_time) = year_value 
    GROUP BY 
        category_name
)
-- Select revenue percentage by category
SELECT 
    category_name, 
    segment_name, 
    ROUND(SUM(sales.price * qty) * 100.0 / total_revenue, 2) AS revenue_pct 
FROM 
    month_under_analysis, 
    balanced_tree.product_details 
LEFT JOIN 
    balanced_tree.sales ON product_details.product_id = sales.prod_id 
LEFT JOIN 
    total_revenue_by_category_cte USING (category_name) 
WHERE 
    EXTRACT(MONTH FROM start_txn_time) = month_value AND EXTRACT(YEAR FROM start_txn_time) = year_value 
GROUP BY 
    category_name, total_revenue, segment_name 
ORDER BY 
    category_name, segment_name, revenue_pct DESC;
```

| category_name | segment_name | revenue_pct |
| ------------- | ------------ | ----------- |
| Mens          | Shirt        | 57.17       |
| Mens          | Socks        | 42.83       |
| Womens        | Jacket       | 63.88       |
| Womens        | Jeans        | 36.12       |

```SQL
-- Common Table Expressions (CTEs) for basket analysis
WITH basket_cte AS (
    SELECT 
        txn_id, 
        STRING_AGG(DISTINCT product_name, ', ' ORDER BY product_name) AS basket_items, 
        COUNT(DISTINCT product_name) AS basket_size 
    FROM 
        month_under_analysis, 
        balanced_tree.sales 
    LEFT JOIN 
        balanced_tree.product_details ON sales.prod_id = product_details.product_id 
    WHERE 
        EXTRACT(MONTH FROM start_txn_time) = month_value AND EXTRACT(YEAR FROM start_txn_time) = year_value 
    GROUP BY 
        txn_id
),
-- Select most frequently purchased combination of 3 products
combinations_cte AS (
    SELECT 
        basket_items, 
        COUNT(basket_items) AS most_frequently_purchased_combination 
    FROM 
        basket_cte 
    WHERE 
        basket_size = 3 
    GROUP BY 
        basket_items 
    ORDER BY 
        most_frequently_purchased_combination DESC
)
-- Select the basket items with the most frequent combination
SELECT 
    basket_items 
FROM 
    combinations_cte 
WHERE 
    most_frequently_purchased_combination = (SELECT MAX(most_frequently_purchased_combination) FROM combinations_cte) 
ORDER BY 
    basket_items;
```

| basket_items                                                                            |
| --------------------------------------------------------------------------------------- |
| Blue Polo Shirt - Mens, Navy Oversized Jeans - Womens, Pink Fluro Polkadot Socks - Mens |
| Cream Relaxed Jeans - Womens, Navy Oversized Jeans - Womens, White Tee Shirt - Mens     |
| Khaki Suit Jacket - Womens, Navy Oversized Jeans - Womens, White Striped Socks - Mens   |

## 🏔️ Case Study #7 - Balanced Tree Clothing Co

Curious for more? Get your hands on all the sections [here](../README.md).

© 2024 [Shiv Katira](https://github.com/shivkatira)