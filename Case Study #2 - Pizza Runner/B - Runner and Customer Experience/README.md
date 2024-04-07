# 🍕 B. Runner and Customer Experience
<p align="center">
<img src="../../img/2.png" align="center" width="400" height="400" >

## 📚 Table of Contents

* [Case Study Questions](#❓-case-study-questions)
* [My Solution](#💡-my-solution)
* [Case Study #2 - Pizza Runner](#🍕-case-study-2---pizza-runner)

## ❓ Case Study Questions

1. [How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)](#q1-how-many-runners-signed-up-for-each-1-week-period-ie-week-starts-2021-01-01)
2. [What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?](#q2-what-was-the-average-time-in-minutes-it-took-for-each-runner-to-arrive-at-the-pizza-runner-hq-to-pickup-the-order)
3. [Is there any relationship between the number of pizzas and how long the order takes to prepare?](#q3-is-there-any-relationship-between-the-number-of-pizzas-and-how-long-the-order-takes-to-prepare)
4. [What was the average distance travelled for each customer?](#q4-what-was-the-average-distance-travelled-for-each-customer)
5. [What was the difference between the longest and shortest delivery times for all orders?](#q5-what-was-the-difference-between-the-longest-and-shortest-delivery-times-for-all-orders)
6. [What was the average speed for each runner for each delivery and do you notice any trend for these values?](#q6-what-was-the-average-speed-for-each-runner-for-each-delivery-and-do-you-notice-any-trend-for-these-values)
7. [What is the successful delivery percentage for each runner?](#q7-what-is-the-successful-delivery-percentage-for-each-runner)

## 💡 My Solution

### Q1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)

```SQL
SELECT 
    TO_CHAR(registration_date, 'w') AS week_no, -- Extract the week number from the registration date
    COUNT(registration_date) AS registrations -- Count the number of registrations for each week
FROM 
    pizza_runner.runners -- Access the table containing runner data
GROUP BY 
    week_no -- Group the results by week number
ORDER BY 
    week_no; -- Order the results by week number
```

| week_no | registrations |
| ------- | ------------- |
| 1       | 2             |
| 2       | 1             |
| 3       | 1             |

### Q2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?

```SQL
SELECT 
    runner_id, -- Select the runner ID
    ROUND( -- Round the result to the nearest integer
        AVG( -- Calculate the average time
            EXTRACT( -- Extract a part of the timestamp
                MINUTE FROM AGE( -- Extract the minute component from the time difference
                    TO_TIMESTAMP(pickup_time, 'YYYY-MM-DD HH24:MI:SS'), -- Convert the pickup time to a timestamp
                    order_time -- Calculate the time difference between the pickup time and the order time
                )
            ) -- Extract the minutes
        )::NUMERIC, -- Cast the result to numeric
        0 -- No decimal places
    ) AS average_time_to_arrive -- Alias for the result
FROM 
    pizza_runner.runner_orders -- Access the table containing runner orders
LEFT JOIN 
    pizza_runner.customer_orders USING(order_id) -- Join with customer orders to get order time
WHERE 
    pickup_time IS NOT NULL -- Exclude null pickup times
GROUP BY 
    runner_id -- Group results by runner ID
ORDER BY 
    runner_id; -- Sort results by runner ID
```

| runner_id | average_time_to_arrive |
| --------- | ---------------------- |
| 1         | 15                     |
| 2         | 23                     |
| 3         | 10                     |

### Q3. Is there any relationship between the number of pizzas and how long the order takes to prepare?

```SQL
WITH cte AS (
    SELECT 
        COUNT(*) AS pizza_count, -- Count the number of pizzas in each order
        AVG(EXTRACT(MINUTE FROM AGE(TO_TIMESTAMP(pickup_time, 'YYYY-MM-DD HH24:MI:SS'), order_time))::NUMERIC) AS average_preparation_time -- Calculate the average order preparation time
    FROM 
        pizza_runner.customer_orders -- Access the table containing customer orders
    LEFT JOIN 
        pizza_runner.runner_orders USING (order_id) -- Join with runner orders table using order ID
    WHERE 
        pickup_time IS NOT NULL -- Ensure pickup time is not null to consider successfully delivered orders
    GROUP BY 
        order_id -- Group the results by order ID to aggregate orders
)
SELECT 
    pizza_count, -- Number of pizzas in each order
    ROUND(AVG(average_preparation_time), 0) AS pizza_preparation_time -- Calculate the average preparation time for each pizza count
FROM 
    cte -- Use the common table expression (CTE) containing aggregated data
GROUP BY 
    pizza_count -- Group the results by pizza count
ORDER BY 
    pizza_count; -- Sort the results by pizza count
```

| pizza_count | pizza_preparation_time |
| ----------- | ---------------------- |
| 1           | 12                     |
| 2           | 18                     |
| 3           | 29                     |

### Q4. What was the average distance travelled for each customer?

```SQL
SELECT 
    customer_id, -- Select the customer ID
    ROUND( AVG(distance::NUMERIC), 2) AS average_distance_travelled -- Calculate the average distance travelled, rounding to 2 decimal places
FROM 
    pizza_runner.runner_orders -- Access the table containing runner orders
LEFT JOIN 
    pizza_runner.customer_orders USING(order_id) -- Join with customer orders to link orders with customers
GROUP BY 
    customer_id -- Group results by customer ID
ORDER BY 
    customer_id; -- Sort results by customer ID
```

| customer_id | average_distance_travelled |
| ----------- | -------------------------- |
| 101         | 20.00                      |
| 102         | 16.73                      |
| 103         | 23.40                      |
| 104         | 10.00                      |
| 105         | 25.00                      |

### Q5. What was the difference between the longest and shortest delivery times for all orders?

```SQL
SELECT 
    MAX(duration)::NUMERIC - MIN(duration)::NUMERIC AS difference_between_longest_and_shortest_delivery_time -- Calculate the difference between the maximum and minimum delivery times
FROM 
    pizza_runner.runner_orders; -- Access the table containing runner orders
```

| difference_between_longest_and_shortest_delivery_time |
| ----------------------------------------------------- |
| 30                                                    |

### Q6. What was the average speed for each runner for each delivery and do you notice any trend for these values?

```SQL
SELECT 
    runner_id, -- Identify the runner ID
    order_id, -- Identify the order ID
    ROUND(distance::NUMERIC / (duration::NUMERIC / 60), 2) AS speed -- Calculate the speed (distance divided by duration) and round to 2 decimal places
FROM 
    pizza_runner.runner_orders -- Access the table containing runner orders
WHERE 
    distance IS NOT NULL; -- Exclude records where distance is NULL
```

| runner_id | order_id | speed |
| --------- | -------- | ----- |
| 1         | 1        | 37.50 |
| 1         | 2        | 44.44 |
| 1         | 3        | 40.20 |
| 2         | 4        | 35.10 |
| 3         | 5        | 40.00 |
| 2         | 7        | 60.00 |
| 2         | 8        | 93.60 |
| 1         | 10       | 60.00 |

### Q7. What is the successful delivery percentage for each runner?

```SQL
SELECT 
    runner_id, -- Identify the runner ID
    ROUND(100.0 * SUM(CASE WHEN distance IS NOT NULL THEN 1 ELSE 0 END) / COUNT(*), 2) AS successful_delivery_rate -- Calculate the percentage of successful deliveries
FROM 
    pizza_runner.runner_orders -- Access the table containing runner orders
GROUP BY 
    runner_id -- Group the results by runner ID
ORDER BY 
    runner_id; -- Sort the results by runner ID
```

| runner_id | successful_delivery_rate |
| --------- | ------------------------ |
| 1         | 100.00                   |
| 2         | 75.00                    |
| 3         | 50.00                    |

## 🍕 Case Study #2 - Pizza Runner

Curious for more? Get your hands on all the sections [here](../README.md).

© 2024 [Shiv Katira](https://github.com/shivkatira)