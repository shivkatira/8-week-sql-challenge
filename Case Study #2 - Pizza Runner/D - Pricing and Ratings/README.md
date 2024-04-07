# 🍕 D. Pricing and Ratings
<p align="center">
<img src="../../img/2.png" align="center" width="400" height="400" >

## 📚 Table of Contents

* [Case Study Questions](#❓-case-study-questions)
* [My Solution](#💡-my-solution)
* [Case Study #2 - Pizza Runner](#🍕-case-study-2---pizza-runner)

## ❓ Case Study Questions

1. [If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?](#q1-if-a-meat-lovers-pizza-costs-12-and-vegetarian-costs-10-and-there-were-no-charges-for-changes---how-much-money-has-pizza-runner-made-so-far-if-there-are-no-delivery-fees)
2. [What if there was an additional $1 charge for any pizza extras?](#q2-what-if-there-was-an-additional-1-charge-for-any-pizza-extras)
    - Add cheese is $1 extra
3. [The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.](#q3-the-pizza-runner-team-now-wants-to-add-an-additional-ratings-system-that-allows-customers-to-rate-their-runner-how-would-you-design-an-additional-table-for-this-new-dataset---generate-a-schema-for-this-new-table-and-insert-your-own-data-for-ratings-for-each-successful-customer-order-between-1-to-5)
4. [Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?](#q4-if-a-meat-lovers-pizza-was-12-and-vegetarian-10-fixed-prices-with-no-cost-for-extras-and-each-runner-is-paid-030-per-kilometre-traveled---how-much-money-does-pizza-runner-have-left-over-after-these-deliveries)
    - customer_id
    - order_id
    - runner_id
    - rating
    - order_time
    - pickup_time
    - Time between order and pickup
    - Delivery duration
    - Average speed
    - Total number of pizzas
5. [If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?](#q5-if-a-meat-lovers-pizza-was-12-and-vegetarian-10-fixed-prices-with-no-cost-for-extras-and-each-runner-is-paid-030-per-kilometre-traveled---how-much-money-does-pizza-runner-have-left-over-after-these-deliveries)

## 💡 My Solution

### Q1. If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?

```SQL
SELECT 
    SUM(
        CASE 
            WHEN pizza_id = 1 THEN 12 -- Calculate total earnings for Meat Lovers pizza
            ELSE 10 -- Calculate total earnings for Vegetarian pizza
        END
    ) AS earnings -- Sum the total earnings
FROM 
    pizza_runner.customer_orders -- Access the customer_orders table to get order information
    LEFT JOIN pizza_runner.runner_orders USING(order_id) -- Join with the runner_orders table using order_id
WHERE 
    distance IS NOT NULL; -- Filter out null distances to ensure only successful deliveries are considered
```

| earnings |
| -------- |
| 138      |

### Q2. What if there was an additional $1 charge for any pizza extras?
- Add cheese is $1 extra

```SQL
SELECT 
    SUM( -- Calculate total earnings
        CASE 
            WHEN pizza_id = 1 THEN 12 -- If the pizza is Meat Lovers, price is $12
            ELSE 10 -- If the pizza is Vegetarian, price is $10
        END 
        + COALESCE( array_length(STRING_TO_ARRAY(extras, ','), 1)::NUMERIC, 0) -- Add $1 for each extra (e.g., cheese)
    ) AS total_earnings -- Alias for the total earnings
FROM 
    pizza_runner.customer_orders -- Access customer orders table
    LEFT JOIN pizza_runner.runner_orders USING(order_id) -- Join with runner orders using order ID
WHERE 
    distance IS NOT NULL; -- Filter out orders with null distance
```

| total_earnings |
| -------------- |
| 142            |

### Q3. The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.

```SQL
-- Drop the table if it already exists
DROP TABLE IF EXISTS pizza_runner.customer_ratings;

-- Create the new table for customer ratings
CREATE TABLE pizza_runner.customer_ratings (
    order_id INTEGER, -- Order ID for the customer order
    ratings INTEGER -- Rating given by the customer (between 1 to 5)
);

-- Insert sample data for ratings for each successful customer order
INSERT INTO pizza_runner.customer_ratings (order_id, ratings) 
VALUES 
    (1, 5), 
    (2, 1), 
    (3, 5), 
    (4, 2), 
    (5, 3), 
    (7, 4), 
    (8, 2), 
    (10, 5);

-- Retrieve and display the data from the customer_ratings table
SELECT * FROM pizza_runner.customer_ratings;
```

### Q4. Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?
- customer_id
- order_id
- runner_id
- rating
- order_time
- pickup_time
- Time between order and pickup
- Delivery duration
- Average speed
- Total number of pizzas

```SQL
SELECT 
    customer_id, -- Customer ID
    order_id, -- Order ID
    runner_id, -- Runner ID
    ratings, -- Rating
    order_time, -- Order time
    TO_TIMESTAMP(pickup_time, 'YYYY-MM-DD HH24:MI:SS') AS pickup_time, -- Pickup time
    EXTRACT(MINUTE FROM AGE(TO_TIMESTAMP(pickup_time, 'YYYY-MM-DD HH24:MI:SS'), order_time)) AS time_between_order_to_pickup, -- Time between order and pickup
    duration, -- Delivery duration
    ROUND(AVG(distance::NUMERIC /(duration::NUMERIC / 60)), 2) AS average_speed, -- Average speed
    COUNT(pizza_id) AS total_number_of_pizzas -- Total number of pizzas
FROM 
    pizza_runner.runner_orders
LEFT JOIN 
    pizza_runner.customer_orders USING (order_id)
LEFT JOIN 
    pizza_runner.customer_ratings USING (order_id)
WHERE 
    distance IS NOT NULL -- Consider only successful deliveries
GROUP BY 
    customer_id, order_id, runner_id, ratings, order_time, pickup_time, duration
ORDER BY 
    customer_id, order_id, runner_id, ratings, order_time, pickup_time, duration;
```

| customer_id | order_id | runner_id | ratings | order_time               | pickup_time              | time_between_order_to_pickup | duration | average_speed | total_number_of_pizzas |
| ----------- | -------- | --------- | ------- | ------------------------ | ------------------------ | ---------------------------- | -------- | ------------- | ---------------------- |
| 101         | 1        | 1         | 5       | 2020-01-01T18:05:02.000Z | 2020-01-01T18:15:34.000Z | 10                           | 32       | 37.50         | 1                      |
| 101         | 2        | 1         | 1       | 2020-01-01T19:00:52.000Z | 2020-01-01T19:10:54.000Z | 10                           | 27       | 44.44         | 1                      |
| 102         | 3        | 1         | 5       | 2020-01-02T23:51:23.000Z | 2020-01-03T00:12:37.000Z | 21                           | 20       | 40.20         | 2                      |
| 102         | 8        | 2         | 2       | 2020-01-09T23:54:33.000Z | 2020-01-10T00:15:02.000Z | 20                           | 15       | 93.60         | 1                      |
| 103         | 4        | 2         | 2       | 2020-01-04T13:23:46.000Z | 2020-01-04T13:53:03.000Z | 29                           | 40       | 35.10         | 3                      |
| 104         | 5        | 3         | 3       | 2020-01-08T21:00:29.000Z | 2020-01-08T21:10:57.000Z | 10                           | 15       | 40.00         | 1                      |
| 104         | 10       | 1         | 5       | 2020-01-11T18:34:49.000Z | 2020-01-11T18:50:20.000Z | 15                           | 10       | 60.00         | 2                      |
| 105         | 7        | 2         | 4       | 2020-01-08T21:20:29.000Z | 2020-01-08T21:30:45.000Z | 10                           | 25       | 60.00         | 1                      |

### Q5. If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?

```SQL
WITH order_costs AS (
    SELECT 
        order_id, 
        SUM(CASE WHEN pizza_id = 1 THEN 12 ELSE 10 END) AS pizza_cost, -- Calculate total pizza cost based on type
        AVG(distance::NUMERIC * 0.3) AS runner_fee -- Calculate runner fees based on distance
    FROM 
        pizza_runner.customer_orders 
    LEFT JOIN 
        pizza_runner.runner_orders USING (order_id) 
    WHERE 
        distance IS NOT NULL -- Consider only successful deliveries
    GROUP BY 
        order_id
)
SELECT 
    ROUND(SUM(pizza_cost - runner_fee), 2) AS net_earnings -- Calculate net earnings
FROM 
    order_costs;
```

| net_earnings |
| ------------ |
| 94.44        |


## 🍕 Case Study #2 - Pizza Runner

Curious for more? Get your hands on all the sections [here](../README.md).

© 2024 [Shiv Katira](https://github.com/shivkatira)