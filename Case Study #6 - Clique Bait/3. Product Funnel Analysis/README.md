# 🪝 3. Product Funnel Analysis
<p align="center">
<img src="../../img/6.png" align="center" width="400" height="400" >

## 📚 Table of Contents

* [Case Study Questions](#-case-study-questions)
* [My Solution](#-my-solution)
* [Case Study #6 - Clique Bait](#-case-study-6---clique-bait)

## ❓ Case Study Questions

* [Using a single SQL query - create a new output table which has the following details:](#using-a-single-sql-query---create-a-new-output-table-which-has-the-following-details)
    - How many times was each product viewed?
    - How many times was each product added to cart?
    - How many times was each product added to a cart but not purchased (abandoned)?
    - How many times was each product purchased?
* [Additionally, create another table which further aggregates the data for the above points but this time for each product category instead of individual products.](#additionally-create-another-table-which-further-aggregates-the-data-for-the-above-points-but-this-time-for-each-product-category-instead-of-individual-products)

1. [Which product had the most views, cart adds and purchases?](#q1-which-product-had-the-most-views-cart-adds-and-purchases)
2. [Which product was most likely to be abandoned?](#q2-which-product-was-most-likely-to-be-abandoned)
3. [Which product had the highest view to purchase percentage?](#q3-which-product-had-the-highest-view-to-purchase-percentage)
4. [What is the average conversion rate from view to cart add?](#q4-what-is-the-average-conversion-rate-from-view-to-cart-add)
5. [What is the average conversion rate from cart add to purchase?](#q5-what-is-the-average-conversion-rate-from-cart-add-to-purchase)

## 💡 My Solution

### Using a single SQL query - create a new output table which has the following details:
- How many times was each product viewed?
- How many times was each product added to cart?
- How many times was each product added to a cart but not purchased (abandoned)?
- How many times was each product purchased?

```SQL
CREATE TABLE clique_bait.product_funnel AS (
    WITH converted_visit_id_cte AS (
        SELECT 
            DISTINCT visit_id -- Select distinct visit IDs for purchase events
        FROM 
            clique_bait.events -- Use events table to get event data
        WHERE 
            event_type = 3 -- Filter for purchase events
    ),
    page_views_cte AS (
        SELECT 
            product_category, -- Select product category
            page_name, -- Select page name
            COUNT(visit_id) AS page_views_count -- Count visit IDs for page view events
        FROM 
            clique_bait.events -- Use events table to get event data
            LEFT JOIN clique_bait.page_hierarchy USING (page_id) -- Join with page_hierarchy table using page_id to get page names
        WHERE 
            event_type = 1 -- Filter for page view events
            AND product_id IS NOT NULL -- Filter out rows with null product IDs
        GROUP BY 
            product_category, page_name -- Group results by product category and page name
        ORDER BY 
            product_category, page_name -- Order results by product category and page name
    ),
    add_to_cart_cte AS (
        SELECT 
            product_category, -- Select product category
            page_name, -- Select page name
            COUNT(visit_id) AS add_to_cart_count -- Count visit IDs for add to cart events
        FROM 
            clique_bait.events -- Use events table to get event data
            LEFT JOIN clique_bait.page_hierarchy USING (page_id) -- Join with page_hierarchy table using page_id to get page names
        WHERE 
            event_type = 2 -- Filter for add to cart events
            AND product_id IS NOT NULL -- Filter out rows with null product IDs
        GROUP BY 
            product_category, page_name -- Group results by product category and page name
        ORDER BY 
            product_category, page_name -- Order results by product category and page name
    ),
    purchase_cte AS (
        SELECT 
            product_category, -- Select product category
            page_name, -- Select page name
            COUNT(visit_id) AS purchase_count -- Count visit IDs for purchase events
        FROM 
            clique_bait.events -- Use events table to get event data
            LEFT JOIN clique_bait.page_hierarchy USING (page_id) -- Join with page_hierarchy table using page_id to get page names
        WHERE 
            visit_id IN (SELECT visit_id FROM converted_visit_id_cte) -- Filter for visit IDs that had purchase events
            AND event_type = 2 -- Filter for add to cart events
            AND product_id IS NOT NULL -- Filter out rows with null product IDs
        GROUP BY 
            product_category, page_name -- Group results by product category and page name
        ORDER BY 
            product_category, page_name -- Order results by product category and page name
    )
    SELECT 
        product_category, -- Select product category
        page_name, -- Select page name
        page_views_count, -- Select page views count
        add_to_cart_count, -- Select add to cart count
        add_to_cart_count - purchase_count AS cart_abandonment_count, -- Calculate cart abandonment count
        purchase_count -- Select purchase count
    FROM 
        page_views_cte -- Use page views CTE for initial data
        LEFT JOIN add_to_cart_cte USING (product_category, page_name) -- Join with add to cart CTE on product category and page name
        LEFT JOIN purchase_cte USING (product_category, page_name) -- Join with purchase CTE on product category and page name
);

SELECT * FROM clique_bait.product_funnel; -- Select all records from the newly created product funnel table
```

| product_category | page_name      | page_views_count | add_to_cart_count | cart_abandonment_count | purchase_count |
| ---------------- | -------------- | ---------------- | ----------------- | ---------------------- | -------------- |
| Fish             | Kingfish       | 1559             | 920               | 213                    | 707            |
| Fish             | Salmon         | 1559             | 938               | 227                    | 711            |
| Fish             | Tuna           | 1515             | 931               | 234                    | 697            |
| Luxury           | Black Truffle  | 1469             | 924               | 217                    | 707            |
| Luxury           | Russian Caviar | 1563             | 946               | 249                    | 697            |
| Shellfish        | Abalone        | 1525             | 932               | 233                    | 699            |
| Shellfish        | Crab           | 1564             | 949               | 230                    | 719            |
| Shellfish        | Lobster        | 1547             | 968               | 214                    | 754            |
| Shellfish        | Oyster         | 1568             | 943               | 217                    | 726            |


### Additionally, create another table which further aggregates the data for the above points but this time for each product category instead of individual products.

```SQL
CREATE TABLE clique_bait.product_category_funnel AS (
    SELECT 
        product_category, -- Select product category
        SUM(page_views_count) AS total_page_views, -- Sum of page views for each product category
        SUM(add_to_cart_count) AS total_add_to_cart, -- Sum of add to cart counts for each product category
        SUM(cart_abandonment_count) AS total_cart_abandonment, -- Sum of cart abandonment counts for each product category
        SUM(purchase_count) AS total_purchases -- Sum of purchase counts for each product category
    FROM 
        clique_bait.product_funnel -- Use the product funnel table for source data
    GROUP BY 
        product_category -- Group results by product category
    ORDER BY 
        product_category -- Order results by product category
);

SELECT * FROM clique_bait.product_category_funnel; -- Select all records from the newly created product category funnel table
```

| product_category | total_page_views | total_add_to_cart | total_cart_abandonment | total_purchases |
| ---------------- | ---------------- | ----------------- | ---------------------- | --------------- |
| Fish             | 4633             | 2789              | 674                    | 2115            |
| Luxury           | 3032             | 1870              | 466                    | 1404            |
| Shellfish        | 6204             | 3792              | 894                    | 2898            |

### Q1. Which product had the most views, cart adds and purchases?

```SQL
-- CTE to rank products by page views
WITH product_views_ranking_cte AS (
    SELECT 
        product_category, -- Select product category
        page_name, -- Select page name
        page_views_count, -- Select page views count
        DENSE_RANK() OVER (ORDER BY page_views_count DESC ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS ranking -- Rank products by page views count in descending order
    FROM 
        clique_bait.product_funnel -- Use the product_funnel table for data
    ORDER BY 
        page_views_count -- Order by page views count
)
-- Select the product with the most views
SELECT 
    product_category, -- Select product category
    page_name, -- Select page name
    page_views_count -- Select page views count
FROM 
    product_views_ranking_cte -- Use the CTE with product views ranking
WHERE 
    ranking = 1; -- Filter for the highest rank, i.e., the product with the most views
```

| product_category | page_name | page_views_count |
| ---------------- | --------- | ---------------- |
| Shellfish        | Oyster    | 1568             |

```SQL
-- CTE to rank products by add to cart count
WITH add_to_cart_ranking_cte AS (
    SELECT 
        product_category, -- Select product category
        page_name, -- Select page name
        add_to_cart_count, -- Select add to cart count
        DENSE_RANK() OVER (ORDER BY add_to_cart_count DESC ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS ranking -- Rank products by add to cart count in descending order
    FROM 
        clique_bait.product_funnel -- Use the product_funnel table for data
    ORDER BY 
        add_to_cart_count -- Order by add to cart count
)
-- Select the product with the most cart adds
SELECT 
    product_category, -- Select product category
    page_name, -- Select page name
    add_to_cart_count -- Select add to cart count
FROM 
    add_to_cart_ranking_cte -- Use the CTE with add to cart ranking
WHERE 
    ranking = 1; -- Filter for the highest rank, i.e., the product with the most cart adds
```

| product_category | page_name | add_to_cart_count |
| ---------------- | --------- | ----------------- |
| Shellfish        | Lobster   | 968               |

```SQL
-- CTE to rank products by purchase count
WITH purchase_ranking_cte AS (
    SELECT 
        product_category, -- Select product category
        page_name, -- Select page name
        purchase_count, -- Select purchase count
        DENSE_RANK() OVER (ORDER BY purchase_count DESC ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS ranking -- Rank products by purchase count in descending order
    FROM 
        clique_bait.product_funnel -- Use the product_funnel table for data
    ORDER BY 
        purchase_count -- Order by purchase count
)
-- Select the product with the most purchases
SELECT 
    product_category, -- Select product category
    page_name, -- Select page name
    purchase_count -- Select purchase count
FROM 
    purchase_ranking_cte -- Use the CTE with purchase ranking
WHERE 
    ranking = 1; -- Filter for the highest rank, i.e., the product with the most purchases
```

| product_category | page_name | purchase_count |
| ---------------- | --------- | -------------- |
| Shellfish        | Lobster   | 754            |

### Q2. Which product was most likely to be abandoned?

```SQL
WITH cart_abandonment_ranking_cte AS (
    SELECT 
        product_category, -- Select product category
        page_name, -- Select page name
        cart_abandonment_count, -- Select the count of cart abandonments
        DENSE_RANK() OVER (ORDER BY cart_abandonment_count DESC ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS ranking -- Rank products by cart abandonment count in descending order
    FROM 
        clique_bait.product_funnel -- Use the product_funnel table for data
)
SELECT 
    product_category, -- Select product category
    page_name, -- Select page name
    cart_abandonment_count -- Select the count of cart abandonments
FROM 
    cart_abandonment_ranking_cte -- Use the common table expression with ranking
WHERE 
    ranking = 1; -- Filter for the highest rank, i.e., the most likely to be abandoned
```

| product_category | page_name      | cart_abandonment_count |
| ---------------- | -------------- | ---------------------- |
| Luxury           | Russian Caviar | 249                    |

### Q3. Which product had the highest view to purchase percentage?

```SQL
-- Common Table Expression to calculate the conversion rate for each product
WITH conversion_rate_cte AS (
    SELECT 
        product_category, 
        page_name, 
        ROUND(purchase_count * 100.0 / page_views_count, 2) AS conversion_rate -- Calculate conversion rate as percentage
    FROM 
        clique_bait.product_funnel
    ORDER BY 
        conversion_rate -- Order by conversion rate in ascending order
), 
-- CTE to rank products by their conversion rate
conversion_rate_ranking_cte AS (
    SELECT 
        *, 
        DENSE_RANK() OVER (ORDER BY conversion_rate DESC ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS ranking -- Rank products by conversion rate in descending order
    FROM 
        conversion_rate_cte
    ORDER BY 
        conversion_rate
)
SELECT 
    product_category, 
    page_name, 
    conversion_rate -- Select the product with the highest conversion rate
FROM 
    conversion_rate_ranking_cte
WHERE 
    ranking = 1; -- Filter to the top-ranked product
```

| product_category | page_name | conversion_rate |
| ---------------- | --------- | --------------- |
| Shellfish        | Lobster   | 48.74           |

### Q4. What is the average conversion rate from view to cart add?

```SQL
SELECT 
    ROUND(AVG(add_to_cart_count * 100.0 / page_views_count), 2) AS average_view_to_cart_add_rate -- Calculate the average conversion rate from view to cart add, rounded to two decimal places
FROM 
    clique_bait.product_funnel; -- Use the product_funnel table to get the necessary counts
```

| average_view_to_cart_add_rate |
| ----------------------------- |
| 60.95                         |

### Q5. What is the average conversion rate from cart add to purchase?

```SQL
SELECT 
    ROUND(AVG(purchase_count * 100.0 / add_to_cart_count), 2) AS average_cart_to_purchase_rate -- Calculate the average conversion rate from cart add to purchase, rounded to two decimal places
FROM 
    clique_bait.product_funnel; -- Use the product_funnel table to get the necessary counts
```

| average_cart_to_purchase_rate |
| ----------------------------- |
| 75.93                         |

## 🪝 Case Study #6 - Clique Bait

Curious for more? Get your hands on all the sections [here](../README.md).

© 2024 [Shiv Katira](https://github.com/shivkatira)