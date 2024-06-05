# 🍜 Case Study #1 - Danny's Diner
<p align="center">
<img src="../img/1.png" align="center" width="400" height="400" >

## 📚 Table of Contents
* [Background](#-background)
* [Full Problem](#-full-problem)
* [Case Study Questions](#-case-study-questions)
* [My Solution](#-my-solution)
* [All Case Studies](#-all-case-studies)

## 📌 Background

Danny's passion for Japanese cuisine led him to open Danny's Diner in 2021, specializing in sushi, curry, and ramen. With basic operational data in hand, the diner seeks assistance in leveraging data to optimize business performance.

## 🧩 Full Problem

Check out full problem [here](https://8weeksqlchallenge.com/case-study-1/).

## ❓ Case Study Questions

1. [What is the total amount each customer spent at the restaurant?](#q1-what-is-the-total-amount-each-customer-spent-at-the-restaurant)
2. [How many days has each customer visited the restaurant?](#q2-how-many-days-has-each-customer-visited-the-restaurant)
3. [What was the first item from the menu purchased by each customer?](#q3-what-was-the-first-item-from-the-menu-purchased-by-each-customer)
4. [What is the most purchased item on the menu and how many times was it purchased by all customers?](#q4-what-is-the-most-purchased-item-on-the-menu-and-how-many-times-was-it-purchased-by-all-customers)
5. [Which item was the most popular for each customer?](#q5-which-item-was-the-most-popular-for-each-customer)
6. [Which item was purchased first by the customer after they became a member?](#q6-which-item-was-purchased-first-by-the-customer-after-they-became-a-member)
7. [Which item was purchased just before the customer became a member?](#q7-which-item-was-purchased-just-before-the-customer-became-a-member)
8. [What is the total items and amount spent for each member before they became a member?](#q8-what-is-the-total-items-and-amount-spent-for-each-member-before-they-became-a-member)
9. [If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?](#q9-if-each-1-spent-equates-to-10-points-and-sushi-has-a-2x-points-multiplier---how-many-points-would-each-customer-have)
10. [In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?](#q10-in-the-first-week-after-a-customer-joins-the-program-including-their-join-date-they-earn-2x-points-on-all-items-not-just-sushi---how-many-points-do-customer-a-and-b-have-at-the-end-of-january)

### Bonus Questions

* [Join All The Things - Create a table that has these columns: customer_id, order_date, product_name, price, member (Y/N).](#join-all-the-things---create-a-table-that-has-these-columns-customer_id-order_date-product_name-price-member-yn)
* [Rank All The Things - Based on the table above, add one column: ranking.](#rank-all-the-things---based-on-the-table-above-add-one-column-ranking)

## 💡 My Solution

### Q1. What is the total amount each customer spent at the restaurant?

```SQL
SELECT 
    customer_id, -- Select customer ID
    SUM(price) AS total_amount_spent -- Calculate total amount spent by each customer
FROM 
    dannys_diner.sales -- Use sales table to get customer transactions
    LEFT JOIN dannys_diner.menu USING(product_id) -- Join with menu table using product ID to get prices
GROUP BY 
    customer_id -- Group results by customer ID to aggregate spending
ORDER BY 
    customer_id; -- Sort results by customer ID for organized presentation
```

| customer_id | total_amount_spent |
| ----------- | ------------------ |
| A           | 76                 |
| B           | 74                 |
| C           | 36                 |

### Q2. How many days has each customer visited the restaurant?

```SQL
SELECT 
    customer_id, -- Select customer ID
    COUNT(DISTINCT order_date) AS visit_count -- Count distinct order dates to calculate visit count for each customer
FROM 
    dannys_diner.sales -- Use sales table to get customer transactions
GROUP BY 
    customer_id -- Group results by customer ID
ORDER BY 
    customer_id; -- Sort results by customer ID for organized presentation
```

| customer_id | visit_count |
| ----------- | ----------- |
| A           | 4           |
| B           | 6           |
| C           | 2           |

### Q3. What was the first item from the menu purchased by each customer?

``` SQL
WITH cte AS (
    SELECT 
        customer_id, -- Select customer ID
        RANK() OVER (PARTITION BY customer_id ORDER BY order_date) AS ranking, -- Rank the purchases for each customer by order date
        product_id 
    FROM 
        dannys_diner.sales -- Use sales table to get customer transactions
) 
SELECT 
    customer_id, -- Select customer ID
    product_name -- Select product name
FROM 
    cte 
    LEFT JOIN dannys_diner.menu USING (product_id) -- Join with menu table using product ID to get product names
WHERE 
    ranking = 1 -- Filter to include only the first-ranked product for each customer
GROUP BY 
    customer_id, 
    product_name -- Group results by customer ID and product name
ORDER BY 
    customer_id, 
    product_name; -- Sort results by customer ID and product name
```

| customer_id | product_name |
| ----------- | ------------ |
| A           | curry        |
| A           | sushi        |
| B           | curry        |
| C           | ramen        |

### Q4. What is the most purchased item on the menu and how many times was it purchased by all customers?

```SQL
SELECT 
    product_name, -- Select product name
    COUNT(product_name) AS purchase_count -- Count occurrences of each product to find the purchase count
FROM 
    dannys_diner.sales -- Use sales table to get customer transactions
    LEFT JOIN dannys_diner.menu USING(product_id) -- Join with menu table using product ID to get product names
GROUP BY 
    product_name -- Group results by product name
ORDER BY 
    purchase_count DESC -- Sort purchase counts in descending order
LIMIT 1; -- Limit results to the top-most purchased item
```

| product_name | purchase_count |
| ------------ | -------------- |
| ramen        | 8              |

### Q5. Which item was the most popular for each customer?

```SQL
WITH cte AS (
    SELECT 
        customer_id, -- Select customer ID
        product_name, -- Select product name
        COUNT(product_id) AS product_purchase_count, -- Count occurrences of each product for each customer
        RANK() OVER (PARTITION BY customer_id ORDER BY COUNT(product_id) DESC) AS ranking -- Rank the products by purchase count for each customer
    FROM 
        dannys_diner.sales -- Use sales table to get customer transactions
        LEFT JOIN dannys_diner.menu USING (product_id) -- Join with menu table using product ID to get product names
    GROUP BY 
        customer_id, 
        product_name -- Group results by customer ID and product name
) 
SELECT 
    customer_id, -- Select customer ID
    product_name, -- Select product name
    product_purchase_count -- Select product purchase count
FROM 
    cte 
WHERE 
    ranking = 1 -- Filter to include only the most popular product for each customer
ORDER BY 
    customer_id; -- Sort results by customer ID
```

| customer_id | product_name | product_purchase_count |
| ----------- | ------------ | ---------------------- |
| A           | ramen        | 3                      |
| B           | ramen        | 2                      |
| B           | curry        | 2                      |
| B           | sushi        | 2                      |
| C           | ramen        | 3                      |

### Q6. Which item was purchased first by the customer after they became a member?

```SQL
WITH cte AS (
    SELECT 
        customer_id, -- Select customer ID
        product_name, -- Select product name
        RANK() OVER (PARTITION BY customer_id ORDER BY order_date) AS ranking -- Rank the purchases for each customer by order date
    FROM 
        dannys_diner.sales AS sales -- Use sales table to get customer transactions
        LEFT JOIN dannys_diner.menu AS menu USING (product_id) -- Join with menu table using product ID to get product names
    WHERE 
        order_date >= ( -- Filter to include purchases after the customer became a member
            SELECT 
                join_date 
            FROM 
                dannys_diner.members AS members 
            WHERE 
                members.customer_id = sales.customer_id
        )
) 
SELECT 
    customer_id, -- Select customer ID
    product_name -- Select product name
FROM 
    cte 
WHERE 
    ranking = 1 -- Filter to include only the first-ranked product for each customer after becoming a member
ORDER BY 
    customer_id; -- Sort results by customer ID
```

| customer_id | product_name |
| ----------- | ------------ |
| A           | curry        |
| B           | sushi        |

### Q7. Which item was purchased just before the customer became a member?

``` SQL
WITH cte AS (
    SELECT 
        customer_id, -- Select customer ID
        product_name, -- Select product name
        RANK() OVER (PARTITION BY customer_id ORDER BY order_date DESC) AS ranking -- Rank the purchases for each customer by order date in descending order
    FROM 
        dannys_diner.sales AS sales -- Use sales table to get customer transactions
        LEFT JOIN dannys_diner.menu AS menu USING (product_id) -- Join with menu table using product ID to get product names
    WHERE 
        order_date < ( -- Filter to include purchases before the customer became a member
            SELECT 
                join_date 
            FROM 
                dannys_diner.members AS members 
            WHERE 
                members.customer_id = sales.customer_id
        )
) 
SELECT 
    customer_id, -- Select customer ID
    product_name -- Select product name
FROM 
    cte 
WHERE 
    ranking = 1 -- Filter to include only the first-ranked product for each customer before becoming a member
ORDER BY 
    customer_id; -- Sort results by customer ID
```

| customer_id | product_name |
| ----------- | ------------ |
| A           | sushi        |
| A           | curry        |
| B           | sushi        |

### Q8. What is the total items and amount spent for each member before they became a member?

```SQL
WITH cte AS (
    SELECT 
        customer_id, -- Select customer ID
        product_id, -- Select product ID
        price -- Select price
    FROM 
        dannys_diner.sales AS sales -- Use sales table to get customer transactions
        LEFT JOIN dannys_diner.menu AS menu USING (product_id) -- Join with menu table using product ID to get prices
    WHERE 
        order_date < ( -- Filter to include purchases before the customer became a member
            SELECT 
                join_date 
            FROM 
                dannys_diner.members AS members 
            WHERE 
                members.customer_id = sales.customer_id
        )
) 
SELECT 
    customer_id, -- Select customer ID
    COUNT(product_id) AS total_items_purchased, -- Count the total items purchased
    SUM(price) AS total_amount_spent -- Calculate the total amount spent
FROM 
    cte -- Use the common table expression
GROUP BY 
    customer_id -- Group results by customer ID
ORDER BY 
    customer_id; -- Sort results by customer ID
```

| customer_id | total_items_purchased | total_amount_spent |
| ----------- | --------------------- | ------------------ |
| A           | 2                     | 25                 |
| B           | 3                     | 40                 |

### Q9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

```SQL
WITH cte AS (
    SELECT 
        customer_id, -- Select customer ID
        CASE 
            WHEN customer_id IN (SELECT customer_id FROM dannys_diner.members) AND product_name = 'sushi' THEN 20 * price -- Calculate points with 2x multiplier for sushi purchases by members
            WHEN customer_id IN (SELECT customer_id FROM dannys_diner.members) AND product_name != 'sushi' THEN 10 * price -- Calculate points for non-sushi purchases by members
            ELSE 0 -- Non-member purchases earn 0 points
        END AS points -- Calculate points for each transaction
    FROM 
        dannys_diner.sales -- Use sales table to get customer transactions
        LEFT JOIN dannys_diner.menu USING (product_id) -- Join with menu table using product ID
)
SELECT 
    customer_id, -- Select customer ID
    SUM(points) AS total_points_earned -- Calculate total points earned by each customer
FROM 
    cte -- Use the common table expression
GROUP BY 
    customer_id -- Group results by customer ID
ORDER BY 
    customer_id; -- Sort results by customer ID
```
| customer_id | total_points_earned |
| ----------- | ------------------- |
| A           | 860                 |
| B           | 940                 |
| C           | 0                   |

### Q10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

```SQL
SELECT 
    customer_id, -- Select customer ID
    SUM(
        CASE 
            WHEN customer_id IN (SELECT customer_id FROM dannys_diner.members) AND product_name = 'sushi' THEN 20 * price -- Calculate points with 2x multiplier for sushi purchases by members
            WHEN customer_id IN (SELECT customer_id FROM dannys_diner.members) AND order_date BETWEEN join_date AND join_date + 6 THEN 20 * price -- Calculate points with 2x multiplier for all purchases in the first week after joining
            WHEN customer_id IN (SELECT customer_id FROM dannys_diner.members) THEN 10 * price -- Calculate points with normal multiplier for subsequent purchases by members
            ELSE 0 -- Non-member purchases earn 0 points
        END
    ) AS points_earned -- Calculate points for each transaction
FROM 
    dannys_diner.sales -- Use sales table to get customer transactions
    LEFT JOIN dannys_diner.menu USING (product_id) -- Join with menu table using product ID
    LEFT JOIN dannys_diner.members USING (customer_id) -- Join with members table using customer ID
WHERE 
    order_date BETWEEN join_date AND '2021-01-31' -- Filter transactions within the first week after joining and January 2021
GROUP BY 
    customer_id -- Group results by customer ID
ORDER BY 
    customer_id; -- Sort results by customer ID
```
| customer_id | points_earned |
| ----------- | ------------- |
| A           | 1020          |
| B           | 320           |

### Join All The Things - Create a table that has these columns: customer_id, order_date, product_name, price, member (Y/N).

```SQL
CREATE TABLE full_view AS (
    SELECT 
        customer_id, -- Select customer ID
        order_date, -- Select order date
        product_name, -- Select product name
        price, -- Select product price
        CASE 
            WHEN order_date < join_date THEN 'N' -- If order date is before join date, customer is not a member
            WHEN order_date >= join_date THEN 'Y' -- If order date is on or after join date, customer is a member
            ELSE 'N' -- Default to 'N' if no other condition matches
        END AS member -- Determine membership status based on order date and join date
    FROM 
        dannys_diner.sales -- Use sales table for customer transactions
        LEFT JOIN dannys_diner.menu USING (product_id) -- Join with menu table using product ID to get product details
        LEFT JOIN dannys_diner.members USING (customer_id) -- Join with members table using customer ID to get membership details
    ORDER BY 
        customer_id, -- Sort results by customer ID for organized presentation
        order_date -- Sort results by order date for organized presentation
);

SELECT 
    * 
FROM 
    full_view; -- Select all columns from the newly created full_view table
```

| customer_id | order_date               | product_name | price | member |
| ----------- | ------------------------ | ------------ | ----- | ------ |
| A           | 2021-01-01T00:00:00.000Z | sushi        | 10    | N      |
| A           | 2021-01-01T00:00:00.000Z | curry        | 15    | N      |
| A           | 2021-01-07T00:00:00.000Z | curry        | 15    | Y      |
| A           | 2021-01-10T00:00:00.000Z | ramen        | 12    | Y      |
| A           | 2021-01-11T00:00:00.000Z | ramen        | 12    | Y      |
| A           | 2021-01-11T00:00:00.000Z | ramen        | 12    | Y      |
| B           | 2021-01-01T00:00:00.000Z | curry        | 15    | N      |
| B           | 2021-01-02T00:00:00.000Z | curry        | 15    | N      |
| B           | 2021-01-04T00:00:00.000Z | sushi        | 10    | N      |
| B           | 2021-01-11T00:00:00.000Z | sushi        | 10    | Y      |
| B           | 2021-01-16T00:00:00.000Z | ramen        | 12    | Y      |
| B           | 2021-02-01T00:00:00.000Z | ramen        | 12    | Y      |
| C           | 2021-01-01T00:00:00.000Z | ramen        | 12    | N      |
| C           | 2021-01-01T00:00:00.000Z | ramen        | 12    | N      |
| C           | 2021-01-07T00:00:00.000Z | ramen        | 12    | N      |

### Rank All The Things - Based on the table above, add one column: ranking.

```SQL
WITH cte AS (
    SELECT 
        customer_id, -- Select customer ID
        order_date, -- Select order date
        product_name, -- Select product name
        price, -- Select price
        CASE 
            WHEN order_date < join_date THEN 'N' -- If order date is before join date, mark as non-member
            WHEN order_date >= join_date THEN 'Y' -- If order date is on or after join date, mark as member
            ELSE 'N' -- Otherwise, mark as non-member
        END AS member -- Create a column indicating if the customer is a member (Y/N)
    FROM 
        dannys_diner.sales -- Use sales table to get customer transactions
        LEFT JOIN dannys_diner.menu USING (product_id) -- Join with menu table using product ID
        LEFT JOIN dannys_diner.members USING (customer_id) -- Join with members table using customer ID
    ORDER BY 
        customer_id, order_date -- Order by customer ID and order date
)
SELECT 
    *, -- Select all columns from the common table expression
    CASE 
        WHEN member = 'N' THEN NULL -- If not a member, set ranking as NULL
        ELSE RANK() OVER (PARTITION BY customer_id, member ORDER BY order_date) -- Calculate ranking within each customer's membership status
    END AS ranking -- Assign a ranking to each row
FROM 
    cte; -- Use the common table expression
```

| customer_id | order_date               | product_name | price | member | ranking |
| ----------- | ------------------------ | ------------ | ----- | ------ | ------- |
| A           | 2021-01-01T00:00:00.000Z | sushi        | 10    | N      | null    |
| A           | 2021-01-01T00:00:00.000Z | curry        | 15    | N      | null    |
| A           | 2021-01-07T00:00:00.000Z | curry        | 15    | Y      | 1       |
| A           | 2021-01-10T00:00:00.000Z | ramen        | 12    | Y      | 2       |
| A           | 2021-01-11T00:00:00.000Z | ramen        | 12    | Y      | 3       |
| A           | 2021-01-11T00:00:00.000Z | ramen        | 12    | Y      | 3       |
| B           | 2021-01-01T00:00:00.000Z | curry        | 15    | N      | null    |
| B           | 2021-01-02T00:00:00.000Z | curry        | 15    | N      | null    |
| B           | 2021-01-04T00:00:00.000Z | sushi        | 10    | N      | null    |
| B           | 2021-01-11T00:00:00.000Z | sushi        | 10    | Y      | 1       |
| B           | 2021-01-16T00:00:00.000Z | ramen        | 12    | Y      | 2       |
| B           | 2021-02-01T00:00:00.000Z | ramen        | 12    | Y      | 3       |
| C           | 2021-01-01T00:00:00.000Z | ramen        | 12    | N      | null    |
| C           | 2021-01-01T00:00:00.000Z | ramen        | 12    | N      | null    |
| C           | 2021-01-07T00:00:00.000Z | ramen        | 12    | N      | null    |

## 🏡 All Case Studies

Curious for more? Get your hands on all the case studies [here](../README.md).

© 2024 [Shiv Katira](https://github.com/shivkatira)
