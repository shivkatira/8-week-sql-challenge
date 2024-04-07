# 🍕 A. Pizza Metrics
<p align="center">
<img src="../../img/2.png" align="center" width="400" height="400" >

## 📚 Table of Contents

* [Case Study Questions](#❓-case-study-questions)
* [My Solution](#💡-my-solution)
* [Case Study #2 - Pizza Runner](#🍕-case-study-2---pizza-runner)

## ❓ Case Study Questions

1. [How many pizzas were ordered?](#q1-how-many-pizzas-were-ordered)
2. [How many unique customer orders were made?](#q2-how-many-unique-customer-orders-were-made)
3. [How many successful orders were delivered by each runner?](#q3-how-many-successful-orders-were-delivered-by-each-runner)
4. [How many of each type of pizza was delivered?](#q4-how-many-of-each-type-of-pizza-was-delivered)
5. [How many Vegetarian and Meatlovers were ordered by each customer?](#q5-how-many-vegetarian-and-meatlovers-were-ordered-by-each-customer)
6. [What was the maximum number of pizzas delivered in a single order?](#q6-what-was-the-maximum-number-of-pizzas-delivered-in-a-single-order)
7. [For each customer, how many delivered pizzas had at least 1 change and how many had no changes?](#q7-for-each-customer-how-many-delivered-pizzas-had-at-least-1-change-and-how-many-had-no-changes)
8. [How many pizzas were delivered that had both exclusions and extras?](#q8-how-many-pizzas-were-delivered-that-had-both-exclusions-and-extras)
9. [What was the total volume of pizzas ordered for each hour of the day?](#q9-what-was-the-total-volume-of-pizzas-ordered-for-each-hour-of-the-day)
10. [What was the volume of orders for each day of the week?](#q10-what-was-the-volume-of-orders-for-each-day-of-the-week)

## 💡 My Solution

### Q1. How many pizzas were ordered?

```SQL
SELECT 
    COUNT(*) AS total_pizzas_ordered -- Count the number of pizzas ordered
FROM 
    pizza_runner.customer_orders; -- Access the customer orders table to count pizzas
```

| total_pizzas_ordered |
| -------------------- |
| 14                   |

### Q2. How many unique customer orders were made?

```SQL
SELECT 
    COUNT(DISTINCT order_id) AS unique_customer_orders -- Count the distinct order IDs to determine the number of unique customer orders
FROM 
    pizza_runner.customer_orders; -- Access the customer orders table
```

| unique_customer_orders |
| ---------------------- |
| 10                     |

### Q3. How many successful orders were delivered by each runner?

```SQL
SELECT 
    runner_id, -- Identify the runner ID
    COUNT(order_id) AS number_of_successful_orders -- Count the successful orders
FROM 
    pizza_runner.runner_orders 
WHERE 
    cancellation IS NULL -- Consider only successful orders
GROUP BY 
    runner_id -- Group results by runner ID
ORDER BY 
    runner_id; -- Sort results by runner ID
```

| runner_id | number_of_successful_orders |
| --------- | --------------------------- |
| 1         | 4                           |
| 2         | 3                           |
| 3         | 1                           |

### Q4. How many of each type of pizza was delivered?

```SQL
SELECT 
    pizza_name, -- Identify the pizza name
    COUNT(pizza_id) AS pizza_count -- Count the occurrences of each pizza type
FROM 
    pizza_runner.customer_orders 
LEFT JOIN 
    pizza_runner.runner_orders USING(order_id) -- Join customer orders with runner orders
LEFT JOIN 
    pizza_runner.pizza_names USING (pizza_id) -- Join with pizza names to get pizza names
WHERE 
    cancellation IS NULL -- Consider only successful deliveries
GROUP BY 
    pizza_name -- Group results by pizza name
ORDER BY 
    pizza_name; -- Sort results by pizza name
```

| pizza_name | pizza_count |
| ---------- | ----------- |
| Meatlovers | 9           |
| Vegetarian | 3           |

### Q5. How many Vegetarian and Meatlovers were ordered by each customer?

```SQL
SELECT 
    customer_id, -- Identify the customer ID
    SUM(CASE WHEN pizza_name = 'Meatlovers' THEN 1 ELSE 0 END) AS Meatlovers, -- Count Meatlovers pizzas
    SUM(CASE WHEN pizza_name = 'Vegetarian' THEN 1 ELSE 0 END) AS Vegetarian -- Count Vegetarian pizzas
FROM 
    pizza_runner.customer_orders 
LEFT JOIN 
    pizza_runner.pizza_names USING(pizza_id) -- Join with pizza names to get pizza types
GROUP BY 
    customer_id -- Group results by customer ID
ORDER BY 
    customer_id; -- Sort results by customer ID
```

| customer_id | meatlovers | vegetarian |
| ----------- | ---------- | ---------- |
| 101         | 2          | 1          |
| 102         | 2          | 1          |
| 103         | 3          | 1          |
| 104         | 3          | 0          |
| 105         | 0          | 1          |

### Q6. What was the maximum number of pizzas delivered in a single order?

```SQL
SELECT 
    order_id, -- Identify the order ID
    COUNT(order_id) AS pizzas_per_order -- Count the number of pizzas per order
FROM 
    pizza_runner.customer_orders 
LEFT JOIN 
    pizza_runner.runner_orders USING(order_id) -- Join orders with runner deliveries
WHERE 
    cancellation IS NULL -- Exclude cancelled orders
GROUP BY 
    order_id -- Group results by order ID
ORDER BY 
    pizzas_per_order DESC -- Sort in descending order of pizzas per order
LIMIT 
    1; -- Limit to only the first row with the maximum pizzas per order
```

| order_id | pizzas_per_order |
| -------- | ---------------- |
| 4        | 3                |

### Q7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?

```SQL
SELECT 
    customer_id, -- Identify the customer ID
    SUM(CASE WHEN (exclusions IS NOT NULL) OR (extras IS NOT NULL) THEN 1 ELSE 0 END) AS at_least_one_change, -- Count pizzas with at least one change
    SUM(CASE WHEN (exclusions IS NULL) AND (extras IS NULL) THEN 1 ELSE 0 END) AS no_changes -- Count pizzas with no changes
FROM 
    pizza_runner.customer_orders 
LEFT JOIN 
    pizza_runner.runner_orders USING(order_id) -- Join customer orders with runner deliveries
WHERE 
    cancellation IS NULL -- Exclude cancelled orders
GROUP BY 
    customer_id -- Group results by customer ID
ORDER BY 
    customer_id; -- Sort results by customer ID
```

| customer_id | at_least_one_change | no_changes |
| ----------- | ------------------- | ---------- |
| 101         | 0                   | 2          |
| 102         | 0                   | 3          |
| 103         | 3                   | 0          |
| 104         | 2                   | 1          |
| 105         | 1                   | 0          |

### Q8. How many pizzas were delivered that had both exclusions and extras?

```SQL
SELECT 
    SUM(CASE WHEN (exclusions IS NOT NULL) AND (extras IS NOT NULL) THEN 1 ELSE 0 END) AS count_of_pizzas_with_exclusions_and_extras -- Sum pizzas with both exclusions and extras
FROM 
    pizza_runner.customer_orders 
LEFT JOIN 
    pizza_runner.runner_orders USING(order_id) -- Join customer orders with runner deliveries
WHERE 
    cancellation IS NULL; -- Exclude cancelled orders
```

| count_of_pizzas_with_exclusions_and_extras |
| ------------------------------------------ |
| 1                                          |

### Q9. What was the total volume of pizzas ordered for each hour of the day?

```SQL
SELECT 
    EXTRACT(HOUR FROM order_time) AS hour_of_day, -- Extract the hour from the order time
    COUNT(*) AS count_pizza_order -- Count the number of pizza orders for each hour
FROM 
    pizza_runner.customer_orders -- Access the customer orders table
GROUP BY 
    hour_of_day -- Group the results by hour of the day
ORDER BY 
    hour_of_day; -- Sort the results by hour of the day
```

| hour_of_day | count_pizza_order |
| ----------- | ----------------- |
| 11          | 1                 |
| 13          | 3                 |
| 18          | 3                 |
| 19          | 1                 |
| 21          | 3                 |
| 23          | 3                 |

### Q10. What was the volume of orders for each day of the week?

```SQL
SELECT 
    TO_CHAR(order_time, 'DAY') AS day_of_week, -- Extract the day of the week from the order time
    COUNT(*) AS count_pizza_order -- Count the number of pizza orders for each day of the week
FROM 
    pizza_runner.customer_orders -- Access the customer orders table
GROUP BY 
    day_of_week; -- Group the results by day of the week
```

| day_of_week | count_pizza_order |
| ----------- | ----------------- |
| WEDNESDAY   | 5                 |
| THURSDAY    | 3                 |
| FRIDAY      | 1                 |
| SATURDAY    | 5                 |

## 🍕 Case Study #2 - Pizza Runner

Curious for more? Get your hands on all the sections [here](../README.md).

© 2024 [Shiv Katira](https://github.com/shivkatira)