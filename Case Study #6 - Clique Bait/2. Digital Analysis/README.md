# 🎣 2. Digital Analysis
<p align="center">
<img src="../../img/6.png" align="center" width="400" height="400" >

## 📚 Table of Contents

* [Case Study Questions](#-case-study-questions)
* [My Solution](#-my-solution)
* [Case Study #6 - Clique Bait](#-case-study-6---clique-bait)

## ❓ Case Study Questions

1. [How many users are there?](#q1-how-many-users-are-there)
2. [How many cookies does each user have on average?](#q2-how-many-cookies-does-each-user-have-on-average)
3. [What is the unique number of visits by all users per month?](#q3-what-is-the-unique-number-of-visits-by-all-users-per-month)
4. [What is the number of events for each event type?](#q4-what-is-the-number-of-events-for-each-event-type)
5. [What is the percentage of visits which have a purchase event?](#q5-what-is-the-percentage-of-visits-which-have-a-purchase-event)
6. [What is the percentage of visits which view the checkout page but do not have a purchase event?](#q6-what-is-the-percentage-of-visits-which-view-the-checkout-page-but-do-not-have-a-purchase-event)
7. [What are the top 3 pages by number of views?](#q7-what-are-the-top-3-pages-by-number-of-views)
8. [What is the number of views and cart adds for each product category?](#q8-what-is-the-number-of-views-and-cart-adds-for-each-product-category)
9. [What are the top 3 products by purchases?](#q9-what-are-the-top-3-products-by-purchases)

## 💡 My Solution

### Q1. How many users are there?

```SQL
SELECT 
    COUNT(DISTINCT user_id) AS user_count -- Count the distinct number of users
FROM 
    clique_bait.users; -- Use the users table to find the count of distinct user IDs
```

| user_count |
| ---------- |
| 500        |

### Q2. How many cookies does each user have on average?

```SQL
WITH cookie_count_cte AS (
    SELECT 
        user_id, -- Select user ID
        COUNT(DISTINCT cookie_id) AS cookie_count -- Count distinct cookies per user
    FROM 
        clique_bait.users -- Use the users table to get user and cookie data
    GROUP BY 
        user_id -- Group by user ID to count cookies per user
)
SELECT 
    ROUND(AVG(cookie_count), 3) AS average_cookie_count -- Calculate the average number of cookies per user and round to 3 decimal places
FROM 
    cookie_count_cte; -- Use the CTE to get the average cookie count
```

| average_cookie_count |
| -------------------- |
| 3.564                |

### Q3. What is the unique number of visits by all users per month?

```SQL
SELECT 
    TO_CHAR(event_time, 'MM') AS month_number, -- Extract the month number from event_time
    TO_CHAR(event_time, 'MONTH') AS month_name, -- Extract the full month name from event_time
    COUNT(DISTINCT visit_id) AS distinct_visits -- Count distinct visits per month
FROM 
    clique_bait.events -- Use the events table to get visit data
GROUP BY 
    month_number, -- Group by month number to aggregate visits
    month_name -- Group by month name for readability
ORDER BY 
    month_number; -- Order results by month number for sequential display
```

| month_number | month_name | distinct_visits |
| ------------ | ---------- | --------------- |
| 01           | JANUARY    | 876             |
| 02           | FEBRUARY   | 1488            |
| 03           | MARCH      | 916             |
| 04           | APRIL      | 248             |
| 05           | MAY        | 36              |

### Q4. What is the number of events for each event type?

```SQL
SELECT 
    event_type, -- Select the event type
    event_name, -- Select the event name
    COUNT(*) AS total_occurrences -- Count the total occurrences of each event type
FROM 
    clique_bait.events -- Use the events table to get event data
    LEFT JOIN clique_bait.event_identifier USING(event_type) -- Join with event_identifier table to get event names
GROUP BY 
    event_type, -- Group by event type to aggregate occurrences
    event_name -- Group by event name for readability
ORDER BY 
    event_type; -- Order results by event type for organized presentation
```

| event_type | event_name    | total_occurrences |
| ---------- | ------------- | ----------------- |
| 1          | Page View     | 20928             |
| 2          | Add to Cart   | 8451              |
| 3          | Purchase      | 1777              |
| 4          | Ad Impression | 876               |
| 5          | Ad Click      | 702               |

### Q5. What is the percentage of visits which have a purchase event?

```SQL
WITH unique_purchase_count_cte AS (
    SELECT 
        COUNT(DISTINCT visit_id) AS unique_purchases -- Count distinct visit IDs where event type is purchase
    FROM 
        clique_bait.events -- Use events table to get event data
    WHERE 
        event_type = 3 -- Filter for purchase events
)
SELECT 
    ROUND( 
        (SELECT unique_purchases FROM unique_purchase_count_cte) * 100.0 / COUNT(DISTINCT visit_id), 2 
    ) AS conversion_rate -- Calculate percentage of visits with a purchase event
FROM 
    clique_bait.events; -- Use events table to get total visits
```

| conversion_rate |
| --------------- |
| 49.86           |

### Q6. What is the percentage of visits which view the checkout page but do not have a purchase event?

```SQL
WITH unique_purchase_count_cte AS (
    SELECT 
        COUNT(DISTINCT visit_id) AS unique_purchases -- Count distinct visit IDs where event type is purchase
    FROM 
        clique_bait.events -- Use events table to get event data
    WHERE 
        event_type = 3 -- Filter for purchase events
)
SELECT 
    ROUND(
        100.0 - ( (SELECT unique_purchases FROM unique_purchase_count_cte) * 100.0 / COUNT(DISTINCT visit_id) ), 2 
    ) AS checkout_abandonment_rate -- Calculate percentage of visits that view checkout page but do not have a purchase event
FROM 
    clique_bait.events -- Use events table to get event data
WHERE 
    page_id = 12; -- Filter for checkout page views
```

| checkout_abandonment_rate |
| ------------------------- |
| 15.50                     |

### Q7. What are the top 3 pages by number of views?

```SQL
SELECT 
    page_name, -- Select page name
    COUNT(DISTINCT visit_id) AS unique_visit_count -- Count distinct visit IDs for each page
FROM 
    clique_bait.events -- Use events table to get event data
    LEFT JOIN clique_bait.page_hierarchy USING(page_id) -- Join with page_hierarchy table using page_id to get page names
GROUP BY 
    page_name -- Group results by page name to aggregate visit counts
ORDER BY 
    unique_visit_count DESC -- Order results by unique visit count in descending order
LIMIT 3; -- Limit results to top 3 pages
```

| page_name    | unique_visit_count |
| ------------ | ------------------ |
| All Products | 3174               |
| Checkout     | 2103               |
| Home Page    | 1782               |

### Q8. What is the number of views and cart adds for each product category?

```SQL
WITH view_count_cte AS (
    SELECT 
        product_category, -- Select product category
        COUNT(visit_id) AS view_count -- Count visit IDs for views
    FROM 
        clique_bait.events -- Use events table to get event data
        LEFT JOIN clique_bait.page_hierarchy USING (page_id) -- Join with page_hierarchy table using page_id to get product categories
    WHERE 
        event_type = 1 -- Filter for view events
    GROUP BY 
        product_category -- Group results by product category to aggregate view counts
), 
add_to_cart_count_cte AS (
    SELECT 
        product_category, -- Select product category
        COUNT(visit_id) AS add_to_cart_count -- Count visit IDs for cart adds
    FROM 
        clique_bait.events -- Use events table to get event data
        LEFT JOIN clique_bait.page_hierarchy USING (page_id) -- Join with page_hierarchy table using page_id to get product categories
    WHERE 
        event_type = 2 -- Filter for add to cart events
    GROUP BY 
        product_category -- Group results by product category to aggregate add to cart counts
) 
SELECT 
    * -- Select all columns
FROM 
    view_count_cte -- Use view count CTE for view counts
    LEFT JOIN add_to_cart_count_cte USING (product_category) -- Join with add to cart count CTE using product category to get add to cart counts
WHERE 
    product_category IS NOT NULL -- Filter out rows with NULL product category
ORDER BY 
    product_category; -- Order results by product category for organized presentation
```

| product_category | view_count | add_to_cart_count |
| ---------------- | ---------- | ----------------- |
| Fish             | 4633       | 2789              |
| Luxury           | 3032       | 1870              |
| Shellfish        | 6204       | 3792              |

### Q9. What are the top 3 products by purchases?

```SQL
WITH purchase_sessions_cte AS (
    SELECT 
        DISTINCT visit_id -- Select distinct visit IDs
    FROM 
        clique_bait.events -- Use events table to get event data
    WHERE 
        event_type = 3 -- Filter for purchase events
)
SELECT 
    page_name, -- Select page name
    COUNT(CASE WHEN event_type = 2 THEN visit_id END) AS purchase_count -- Count visit IDs for add to cart events, treating them as purchases
FROM 
    clique_bait.events -- Use events table to get event data
    LEFT JOIN clique_bait.page_hierarchy USING (page_id) -- Join with page_hierarchy table using page_id to get page names
WHERE 
    visit_id IN (SELECT * FROM purchase_sessions_cte) -- Filter for visit IDs that had purchase events
    AND event_type = 2 -- Filter for add to cart events
GROUP BY 
    page_name -- Group results by page name to aggregate purchase counts
ORDER BY 
    purchase_count DESC -- Order results by purchase count in descending order to get top products
LIMIT 3; -- Limit results to the top 3 products
```

| page_name | purchase_count |
| --------- | -------------- |
| Lobster   | 754            |
| Oyster    | 726            |
| Crab      | 719            |

## 🎣 Case Study #6 - Clique Bait

Curious for more? Get your hands on all the sections [here](../README.md).

© 2024 [Shiv Katira](https://github.com/shivkatira)