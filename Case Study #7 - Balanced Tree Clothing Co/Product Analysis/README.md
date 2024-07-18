# 🏔️ Product Analysis
<p align="center">
<img src="../../img/7.png" align="center" width="400" height="400" >

## 📚 Table of Contents

* [Case Study Questions](#-case-study-questions)
* [My Solution](#-my-solution)
* [Case Study #7 - Balanced Tree Clothing Co](#️-case-study-7---balanced-tree-clothing-co)

## ❓ Case Study Questions

1. [What are the top 3 products by total revenue before discount?](#q1-what-are-the-top-3-products-by-total-revenue-before-discount)
2. [What is the total quantity, revenue and discount for each segment?](#q2-what-is-the-total-quantity-revenue-and-discount-for-each-segment)
3. [What is the top selling product for each segment?](#q3-what-is-the-top-selling-product-for-each-segment)
4. [What is the total quantity, revenue and discount for each category?](#q4-what-is-the-total-quantity-revenue-and-discount-for-each-category)
5. [What is the top selling product for each category?](#q5-what-is-the-top-selling-product-for-each-category)
6. [What is the percentage split of revenue by product for each segment?](#q6-what-is-the-percentage-split-of-revenue-by-product-for-each-segment)
7. [What is the percentage split of revenue by segment for each category?](#q7-what-is-the-percentage-split-of-revenue-by-segment-for-each-category)
8. [What is the percentage split of total revenue by category?](#q8-what-is-the-percentage-split-of-total-revenue-by-category)
9. [What is the total transaction “penetration” for each product? (hint: penetration = number of transactions where at least 1 quantity of a product was purchased divided by total number of transactions)](#q9-what-is-the-total-transaction-penetration-for-each-product-hint-penetration--number-of-transactions-where-at-least-1-quantity-of-a-product-was-purchased-divided-by-total-number-of-transactions)
10. [What is the most common combination of at least 1 quantity of any 3 products in a 1 single transaction?](#q10-what-is-the-most-common-combination-of-at-least-1-quantity-of-any-3-products-in-a-1-single-transaction)

## 💡 My Solution

### Q1. What are the top 3 products by total revenue before discount?

```SQL
WITH 
    revenue_cte AS (
        SELECT 
            product_name, -- Select product name
            SUM(qty * product_details.price) AS revenue -- Calculate total revenue for each product before discount
        FROM 
            balanced_tree.product_details -- Use product details table
            LEFT JOIN balanced_tree.sales ON product_id = prod_id -- Join with sales table to get quantities sold
        GROUP BY 
            product_name -- Group results by product name to aggregate revenue
    ), 
    ranking_cte AS (
        SELECT 
            *, 
            DENSE_RANK() OVER (ORDER BY revenue DESC) AS ranking -- Rank products by total revenue in descending order
        FROM 
            revenue_cte -- Use the revenue CTE to get the total revenue for each product
    )
SELECT 
    product_name, -- Select product name
    revenue -- Select total revenue
FROM 
    ranking_cte -- Use the ranking CTE to get the ranked products
WHERE 
    ranking <= 3 -- Filter to get only the top 3 products
ORDER BY 
    revenue DESC; -- Order the results by revenue in descending order for better readability
```

| product_name                 | revenue |
| ---------------------------- | ------- |
| Blue Polo Shirt - Mens       | 217683  |
| Grey Fashion Jacket - Womens | 209304  |
| White Tee Shirt - Mens       | 152000  |

### Q2. What is the total quantity, revenue and discount for each segment?

```SQL
SELECT 
    segment_name, -- Select segment name
    SUM(qty) AS total_quantity, -- Calculate total quantity sold for each segment
    SUM(qty * product_details.price) AS total_revenue_before_discount, -- Calculate total revenue before discount for each segment
    ROUND(SUM((discount * qty * product_details.price) / 100.0), 2) AS total_discount -- Calculate total discount for each segment
FROM 
    balanced_tree.product_details -- Use product details table
    LEFT JOIN balanced_tree.sales ON product_id = prod_id -- Join with sales table to get quantities sold and discounts
GROUP BY 
    segment_name -- Group results by segment name to aggregate quantity, revenue, and discount
ORDER BY 
    segment_name; -- Order results by segment name for better readability
```

| segment_name | total_quantity | total_revenue_before_discount | total_discount |
| ------------ | -------------- | ----------------------------- | -------------- |
| Jacket       | 11385          | 366983                        | 44277.46       |
| Jeans        | 11349          | 208350                        | 25343.97       |
| Shirt        | 11265          | 406143                        | 49594.27       |
| Socks        | 11217          | 307977                        | 37013.44       |

### Q3. What is the top selling product for each segment?

```SQL
WITH ranking_by_total_qty_cte AS (
    SELECT 
        segment_name, -- Select segment name
        product_name, -- Select product name
        DENSE_RANK() OVER (PARTITION BY segment_name ORDER BY SUM(qty) DESC ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS ranking, -- Rank products within each segment by total quantity sold
        SUM(qty) AS total_quantity -- Calculate total quantity sold for each product within the segment
    FROM 
        balanced_tree.product_details -- Use product details table
        LEFT JOIN balanced_tree.sales ON product_id = prod_id -- Join with sales table to get quantities sold
    GROUP BY 
        segment_name, product_name -- Group results by segment name and product name to aggregate quantities
    ORDER BY 
        segment_name, product_name -- Order results by segment name and product name for better readability
) 
SELECT 
    segment_name, -- Select segment name
    product_name, -- Select product name
    total_quantity -- Select total quantity sold
FROM 
    ranking_by_total_qty_cte -- Use the CTE to get the ranking information
WHERE 
    ranking = 1; -- Filter to get only the top selling product for each segment
```

| segment_name | product_name                  | total_quantity |
| ------------ | ----------------------------- | -------------- |
| Jacket       | Grey Fashion Jacket - Womens  | 3876           |
| Jeans        | Navy Oversized Jeans - Womens | 3856           |
| Shirt        | Blue Polo Shirt - Mens        | 3819           |
| Socks        | Navy Solid Socks - Mens       | 3792           |

### Q4. What is the total quantity, revenue and discount for each category?

```SQL
SELECT 
    category_name, -- Select category name
    SUM(qty) AS total_quantity, -- Calculate total quantity for each category
    SUM(qty * product_details.price) AS total_revenue_before_discount, -- Calculate total revenue before discount for each category
    ROUND(SUM((discount * qty * product_details.price) / 100.0), 2) AS total_discount -- Calculate total discount for each category
FROM 
    balanced_tree.product_details -- Use product details table
    LEFT JOIN balanced_tree.sales ON product_id = prod_id -- Join with sales table to get sales data
GROUP BY 
    category_name -- Group results by category name to aggregate totals
ORDER BY 
    category_name; -- Order results by category name for organized presentation
```

| category_name | total_quantity | total_revenue_before_discount | total_discount |
| ------------- | -------------- | ----------------------------- | -------------- |
| Mens          | 22482          | 714120                        | 86607.71       |
| Womens        | 22734          | 575333                        | 69621.43       |

### Q5. What is the top selling product for each category?

```SQL
WITH ranking_by_total_quantity_cte AS (
    SELECT 
        category_name, -- Select category name
        product_name, -- Select product name
        DENSE_RANK() OVER (PARTITION BY category_name ORDER BY SUM(qty) DESC) AS ranking, -- Rank products by total quantity sold within each category
        SUM(qty) AS total_quantity -- Calculate total quantity sold for each product
    FROM 
        balanced_tree.product_details -- Use product details table
        LEFT JOIN balanced_tree.sales ON product_id = prod_id -- Join with sales table to get sales data
    GROUP BY 
        category_name, -- Group by category name to aggregate totals
        product_name -- Group by product name to aggregate totals
    ORDER BY 
        category_name, -- Order results by category name for organized presentation
        product_name -- Order results by product name for organized presentation
)
SELECT 
    category_name, -- Select category name
    product_name, -- Select product name
    total_quantity -- Select total quantity sold
FROM 
    ranking_by_total_quantity_cte -- Use CTE to get ranking and totals
WHERE 
    ranking = 1; -- Filter to get only the top ranked product for each category
```

| category_name | product_name                 | total_quantity |
| ------------- | ---------------------------- | -------------- |
| Mens          | Blue Polo Shirt - Mens       | 3819           |
| Womens        | Grey Fashion Jacket - Womens | 3876           |

### Q6. What is the percentage split of revenue by product for each segment?

```SQL
WITH total_revenue_by_segment_cte AS (
    SELECT 
        segment_name, -- Select segment name
        SUM(sales.price * qty) AS total_revenue -- Calculate total revenue for each segment
    FROM 
        balanced_tree.product_details -- Use product details table
        LEFT JOIN balanced_tree.sales ON product_details.product_id = sales.prod_id -- Join with sales table to get sales data
    GROUP BY 
        segment_name -- Group by segment name to aggregate totals
)
SELECT 
    segment_name, -- Select segment name
    product_name, -- Select product name
    ROUND(SUM(sales.price * qty) * 100.0 / total_revenue, 2) AS revenue_percentage -- Calculate revenue percentage for each product within the segment
FROM 
    balanced_tree.product_details -- Use product details table
    LEFT JOIN balanced_tree.sales ON product_details.product_id = sales.prod_id -- Join with sales table to get sales data
    LEFT JOIN total_revenue_by_segment_cte USING (segment_name) -- Join with total revenue CTE to get total segment revenue
GROUP BY 
    segment_name, -- Group by segment name to aggregate totals
    total_revenue, -- Group by total revenue to calculate percentage
    product_name -- Group by product name to calculate percentage
ORDER BY 
    segment_name, -- Order results by segment name for organized presentation
    revenue_percentage DESC; -- Order results by revenue percentage in descending order
```

| segment_name | product_name                     | revenue_percentage |
| ------------ | -------------------------------- | ------------------ |
| Jacket       | Grey Fashion Jacket - Womens     | 57.03              |
| Jacket       | Khaki Suit Jacket - Womens       | 23.51              |
| Jacket       | Indigo Rain Jacket - Womens      | 19.45              |
| Jeans        | Black Straight Jeans - Womens    | 58.15              |
| Jeans        | Navy Oversized Jeans - Womens    | 24.06              |
| Jeans        | Cream Relaxed Jeans - Womens     | 17.79              |
| Shirt        | Blue Polo Shirt - Mens           | 53.60              |
| Shirt        | White Tee Shirt - Mens           | 37.43              |
| Shirt        | Teal Button Up Shirt - Mens      | 8.98               |
| Socks        | Navy Solid Socks - Mens          | 44.33              |
| Socks        | Pink Fluro Polkadot Socks - Mens | 35.50              |
| Socks        | White Striped Socks - Mens       | 20.18              |

### Q7. What is the percentage split of revenue by segment for each category?

```SQL
WITH total_revenue_by_category_cte AS (
    SELECT 
        category_name, -- Select category name
        SUM(sales.price * qty) AS total_revenue -- Calculate total revenue for each category
    FROM 
        balanced_tree.product_details -- Use product details table
        LEFT JOIN balanced_tree.sales ON product_details.product_id = sales.prod_id -- Join with sales table to get sales data
    GROUP BY 
        category_name -- Group by category name to aggregate totals
)
SELECT 
    category_name, -- Select category name
    segment_name, -- Select segment name
    ROUND(SUM(sales.price * qty) * 100.0 / total_revenue, 2) AS revenue_percentage -- Calculate revenue percentage for each segment within the category
FROM 
    balanced_tree.product_details -- Use product details table
    LEFT JOIN balanced_tree.sales ON product_details.product_id = sales.prod_id -- Join with sales table to get sales data
    LEFT JOIN total_revenue_by_category_cte USING (category_name) -- Join with total revenue CTE to get total category revenue
GROUP BY 
    category_name, -- Group by category name to aggregate totals
    total_revenue, -- Group by total revenue to calculate percentage
    segment_name -- Group by segment name to calculate percentage
ORDER BY 
    category_name, -- Order results by category name for organized presentation
    segment_name, -- Order results by segment name for further organization
    revenue_percentage DESC; -- Order results by revenue percentage in descending order
```

| category_name | segment_name | revenue_percentage |
| ------------- | ------------ | ------------------ |
| Mens          | Shirt        | 56.87              |
| Mens          | Socks        | 43.13              |
| Womens        | Jacket       | 63.79              |
| Womens        | Jeans        | 36.21              |

### Q8. What is the percentage split of total revenue by category?

```SQL
WITH total_revenue_cte AS (
    SELECT 
        SUM(sales.price * qty) AS total_revenue -- Calculate total revenue from all sales
    FROM 
        balanced_tree.sales -- Use sales table to get sales data
)
SELECT 
    category_name, -- Select category name
    ROUND(SUM(sales.price * qty) * 100.0 / (SELECT total_revenue FROM total_revenue_cte), 2) AS revenue_percentage -- Calculate revenue percentage for each category
FROM 
    balanced_tree.sales -- Use sales table to get sales data
    LEFT JOIN balanced_tree.product_details ON product_details.product_id = sales.prod_id -- Join with product details table to get product information
GROUP BY 
    category_name -- Group by category name to aggregate totals
ORDER BY 
    revenue_percentage DESC; -- Order results by revenue percentage in descending order
```

| category_name | revenue_percentage |
| ------------- | ------------------ |
| Mens          | 55.38              |
| Womens        | 44.62              |

### Q9. What is the total transaction “penetration” for each product? (hint: penetration = number of transactions where at least 1 quantity of a product was purchased divided by total number of transactions)

```SQL
SELECT 
    product_name, -- Select product name
    ROUND(
        SUM(CASE WHEN product_id = prod_id THEN 1 ELSE 0 END) * 100.0 / ( 
            SELECT COUNT(DISTINCT txn_id) FROM balanced_tree.sales -- Calculate total number of unique transactions
        ), 
        2
    ) AS penetration -- Calculate the penetration percentage for each product
FROM 
    balanced_tree.product_details -- Use product details table to get product information
    LEFT JOIN balanced_tree.sales ON product_details.product_id = sales.prod_id -- Join with sales table to match products with transactions
GROUP BY 
    product_name -- Group by product name to aggregate totals
ORDER BY 
    penetration DESC; -- Order results by penetration percentage in descending order
```

| product_name                     | penetration |
| -------------------------------- | ----------- |
| Navy Solid Socks - Mens          | 51.24       |
| Grey Fashion Jacket - Womens     | 51.00       |
| Navy Oversized Jeans - Womens    | 50.96       |
| White Tee Shirt - Mens           | 50.72       |
| Blue Polo Shirt - Mens           | 50.72       |
| Pink Fluro Polkadot Socks - Mens | 50.32       |
| Indigo Rain Jacket - Womens      | 50.00       |
| Khaki Suit Jacket - Womens       | 49.88       |
| Black Straight Jeans - Womens    | 49.84       |
| White Striped Socks - Mens       | 49.72       |
| Cream Relaxed Jeans - Womens     | 49.72       |
| Teal Button Up Shirt - Mens      | 49.68       |

### Q10. What is the most common combination of at least 1 quantity of any 3 products in a 1 single transaction?

```SQL
WITH basket_cte AS (
    SELECT 
        txn_id, -- Select transaction ID
        STRING_AGG(DISTINCT product_name, ', ' ORDER BY product_name) AS basket_items, -- Concatenate distinct product names in each transaction into a single string
        COUNT(DISTINCT product_name) AS basket_size -- Count distinct product names in each transaction
    FROM 
        balanced_tree.sales -- Use sales table to get transaction details
        LEFT JOIN balanced_tree.product_details ON sales.prod_id = product_details.product_id -- Join with product details to get product names
    GROUP BY 
        txn_id -- Group by transaction ID to aggregate products in each transaction
), 
combinations_cte AS (
    SELECT 
        basket_items, -- Select basket items
        COUNT(basket_items) AS most_frequently_purchased_combination -- Count how many times each basket combination occurs
    FROM 
        basket_cte 
    WHERE 
        basket_size = 3 -- Filter for transactions that include exactly 3 distinct products
    GROUP BY 
        basket_items -- Group by basket items to aggregate counts
    ORDER BY 
        most_frequently_purchased_combination DESC -- Order by the count of each basket combination in descending order
) 
SELECT 
    basket_items -- Select basket items
FROM 
    combinations_cte 
WHERE 
    most_frequently_purchased_combination = ( 
        SELECT 
            MAX(most_frequently_purchased_combination) -- Select the highest count of any basket combination
        FROM 
            combinations_cte 
    ) 
ORDER BY 
    basket_items; -- Order results by basket items for organized presentation
```

| basket_items                                                                              |
| ----------------------------------------------------------------------------------------- |
| Black Straight Jeans - Womens, Navy Oversized Jeans - Womens, Teal Button Up Shirt - Mens |
| Black Straight Jeans - Womens, Navy Oversized Jeans - Womens, White Tee Shirt - Mens      |
| Blue Polo Shirt - Mens, Navy Oversized Jeans - Womens, Pink Fluro Polkadot Socks - Mens   |
| Cream Relaxed Jeans - Womens, Navy Oversized Jeans - Womens, White Tee Shirt - Mens       |
| Khaki Suit Jacket - Womens, Navy Oversized Jeans - Womens, White Striped Socks - Mens     |

## 🏔️ Case Study #7 - Balanced Tree Clothing Co

Curious for more? Get your hands on all the sections [here](../README.md).

© 2024 [Shiv Katira](https://github.com/shivkatira)