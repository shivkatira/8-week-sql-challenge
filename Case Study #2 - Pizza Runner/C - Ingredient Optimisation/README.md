# 🍕 C. Ingredient Optimisation
<p align="center">
<img src="../../img/2.png" align="center" width="400" height="400" >

## 📚 Table of Contents

* [Case Study Questions](#❓-case-study-questions)
* [My Solution](#💡-my-solution)
* [Case Study #2 - Pizza Runner](#🍕-case-study-2---pizza-runner)

## ❓ Case Study Questions

1. [What are the standard ingredients for each pizza?](#q1-what-are-the-standard-ingredients-for-each-pizza)
2. [What was the most commonly added extra?](#q2-what-was-the-most-commonly-added-extra)
3. [What was the most common exclusion?](#q3-what-was-the-most-common-exclusion)
4. [Generate an order item for each record in the customers_orders table in the format of one of the following:](#q4-generate-an-order-item-for-each-record-in-the-customers_orders-table-in-the-format-of-one-of-the-following)
    - Meat Lovers
    - Meat Lovers - Exclude Beef
    - Meat Lovers - Extra Bacon
    - Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers
5. [Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients](#q5-generate-an-alphabetically-ordered-comma-separated-ingredient-list-for-each-pizza-order-from-the-customer_orders-table-and-add-a-2x-in-front-of-any-relevant-ingredients)
    - For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"
6. [What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?](#q6-what-is-the-total-quantity-of-each-ingredient-used-in-all-delivered-pizzas-sorted-by-most-frequent-first)

## 💡 My Solution

### Q1. What are the standard ingredients for each pizza?

```SQL
WITH cte AS (
    SELECT 
        pizza_name, -- Name of the pizza
        UNNEST(STRING_TO_ARRAY(toppings, ','))::NUMERIC AS topping_id -- Extract topping IDs from the toppings list
    FROM 
        pizza_runner.pizza_recipes -- Access the table containing pizza recipes
    LEFT JOIN 
        pizza_runner.pizza_names USING (pizza_id) -- Join with pizza names table using pizza ID
)
-- Select the pizza name and the aggregated list of distinct toppings for each pizza
SELECT 
    pizza_name, -- Name of the pizza
    STRING_AGG(DISTINCT topping_name, ',') AS toppings -- Aggregate the distinct topping names into a comma-separated list
FROM 
    cte -- Use the common table expression (CTE) containing pizza data
LEFT JOIN 
    pizza_runner.pizza_toppings USING (topping_id) -- Join with pizza toppings table using topping ID
GROUP BY 
    pizza_name -- Group the results by pizza name
ORDER BY 
    pizza_name; -- Sort the results by pizza name
```

| pizza_name | toppings                                                       |
| ---------- | -------------------------------------------------------------- |
| Meatlovers | BBQ Sauce,Bacon,Beef,Cheese,Chicken,Mushrooms,Pepperoni,Salami |
| Vegetarian | Cheese,Mushrooms,Onions,Peppers,Tomato Sauce,Tomatoes          |

### Q2. What was the most commonly added extra?

```SQL
WITH cte AS (
    SELECT 
        UNNEST(STRING_TO_ARRAY(extras, ','))::NUMERIC AS topping_id -- Extract topping IDs from the extras list
    FROM 
        pizza_runner.customer_orders -- Access the table containing customer orders
    WHERE 
        extras IS NOT NULL -- Filter out orders without extras
)
-- Select the topping name and count of occurrences for each extra
SELECT 
    topping_name, -- Name of the extra topping
    COUNT(topping_id) AS count_extra -- Count of occurrences for each extra topping
FROM 
    cte -- Use the common table expression (CTE) containing extra toppings data
LEFT JOIN 
    pizza_runner.pizza_toppings USING (topping_id) -- Join with pizza toppings table using topping ID
GROUP BY 
    topping_name -- Group the results by topping name
ORDER BY 
    count_extra DESC -- Sort the results by count of occurrences in descending order
LIMIT 1; -- Limit the result to the most commonly added extra
```

| topping_name | count_extra |
| ------------ | ----------- |
| Bacon        | 4           |

### Q3. What was the most common exclusion?

```SQL
WITH cte AS (
    SELECT 
        UNNEST(STRING_TO_ARRAY(exclusions, ','))::NUMERIC AS topping_id -- Extract topping IDs from the exclusions list
    FROM 
        pizza_runner.customer_orders -- Access the table containing customer orders
    WHERE 
        exclusions IS NOT NULL -- Filter out orders without exclusions
)
-- Select the topping name and count of occurrences for each exclusion
SELECT 
    topping_name, -- Name of the excluded topping
    COUNT(topping_id) AS count_exclusion -- Count of occurrences for each exclusion topping
FROM 
    cte -- Use the common table expression (CTE) containing exclusion toppings data
LEFT JOIN 
    pizza_runner.pizza_toppings USING (topping_id) -- Join with pizza toppings table using topping ID
GROUP BY 
    topping_name -- Group the results by topping name
ORDER BY 
    count_exclusion DESC -- Sort the results by count of occurrences in descending order
LIMIT 1; -- Limit the result to the most common exclusion
```
| topping_name | count_exclusion |
| ------------ | --------------- |
| Cheese       | 4               |

### Q4. Generate an order item for each record in the customers_orders table in the format of one of the following:
- Meat Lovers
- Meat Lovers - Exclude Beef
- Meat Lovers - Extra Bacon
- Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers

```SQL
-- Common Table Expressions (CTEs) to extract exclusions and extras from customer orders
WITH 
    -- CTE to extract exclusion toppings
    exclusions_id_cte AS (
        SELECT 
            order_id, customer_id, pizza_id, 
            UNNEST(STRING_TO_ARRAY(exclusions, ','))::NUMERIC AS exclusions_topping_id 
        FROM 
            pizza_runner.customer_orders 
        WHERE 
            exclusions IS NOT NULL
    ), 
    -- CTE to extract extra toppings
    extras_id_cte AS (
        SELECT 
            order_id, customer_id, pizza_id, 
            UNNEST(STRING_TO_ARRAY(extras, ','))::NUMERIC AS extras_topping_id 
        FROM 
            pizza_runner.customer_orders 
        WHERE 
            extras IS NOT NULL
    ), 
    -- CTE to aggregate exclusion toppings
    exclusions_cte AS (
        SELECT 
            order_id, customer_id, pizza_id, 
            STRING_AGG(DISTINCT topping_name, ', ') AS exclusions_topping_name 
        FROM 
            exclusions_id_cte 
        LEFT JOIN 
            pizza_runner.pizza_toppings ON exclusions_id_cte.exclusions_topping_id = pizza_toppings.topping_id 
        GROUP BY 
            order_id, customer_id, pizza_id
    ), 
    -- CTE to aggregate extra toppings
    extras_cte AS (
        SELECT 
            order_id, customer_id, pizza_id, 
            STRING_AGG(DISTINCT topping_name, ', ') AS extras_topping_name 
        FROM 
            extras_id_cte 
        LEFT JOIN 
            pizza_runner.pizza_toppings ON extras_id_cte.extras_topping_id = pizza_toppings.topping_id 
        GROUP BY 
            order_id, customer_id, pizza_id
    )
-- Main query to generate order items with exclusions and extras
SELECT 
    order_id, customer_id, 
    CONCAT(
        pizza_name, 
        CASE 
            WHEN exclusions_topping_name IS NULL THEN '' 
            ELSE CONCAT(' - Exclude ', exclusions_topping_name) 
        END, 
        CASE 
            WHEN extras_topping_name IS NULL THEN '' 
            ELSE CONCAT(' - Extra ', extras_topping_name) 
        END
    ) AS order_details 
FROM 
    pizza_runner.customer_orders 
LEFT JOIN 
    pizza_runner.pizza_names USING (pizza_id) 
LEFT JOIN 
    exclusions_cte USING (order_id, customer_id, pizza_id) 
LEFT JOIN 
    extras_cte USING (order_id, customer_id, pizza_id) 
ORDER BY 
    order_id, customer_id;
```

| order_id | customer_id | order_details                                                   |
| -------- | ----------- | --------------------------------------------------------------- |
| 1        | 101         | Meatlovers                                                      |
| 2        | 101         | Meatlovers                                                      |
| 3        | 102         | Meatlovers                                                      |
| 3        | 102         | Vegetarian                                                      |
| 4        | 103         | Meatlovers - Exclude Cheese                                     |
| 4        | 103         | Meatlovers - Exclude Cheese                                     |
| 4        | 103         | Vegetarian - Exclude Cheese                                     |
| 5        | 104         | Meatlovers - Extra Bacon                                        |
| 6        | 101         | Vegetarian                                                      |
| 7        | 105         | Vegetarian - Extra Bacon                                        |
| 8        | 102         | Meatlovers                                                      |
| 9        | 103         | Meatlovers - Exclude Cheese - Extra Bacon, Chicken              |
| 10       | 104         | Meatlovers - Exclude BBQ Sauce, Mushrooms - Extra Bacon, Cheese |
| 10       | 104         | Meatlovers - Exclude BBQ Sauce, Mushrooms - Extra Bacon, Cheese |

### Q5. Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients
- For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"

```SQL
-- Common Table Expressions (CTEs) to extract toppings, extras, and exclusions
WITH 
    -- CTE to extract pizza toppings
    pizza_toppings_id_cte AS (
        SELECT 
            order_id, customer_id, pizza_id, 
            UNNEST(STRING_TO_ARRAY(toppings, ','))::NUMERIC AS topping_id 
        FROM 
            pizza_runner.pizza_recipes 
        LEFT JOIN 
            pizza_runner.pizza_names USING (pizza_id) 
        LEFT JOIN 
            pizza_runner.customer_orders USING (pizza_id)
    ), 
    -- CTE to extract extra toppings
    extras_id_cte AS (
        SELECT 
            order_id, customer_id, pizza_id, 
            UNNEST(STRING_TO_ARRAY(extras, ','))::NUMERIC AS topping_id 
        FROM 
            pizza_runner.customer_orders 
        WHERE 
            extras IS NOT NULL
    ), 
    -- CTE to extract exclusion toppings
    exclusions_id_cte AS (
        SELECT 
            order_id, customer_id, pizza_id, 
            UNNEST(STRING_TO_ARRAY(exclusions, ','))::NUMERIC AS topping_id 
        FROM 
            pizza_runner.customer_orders 
        WHERE 
            exclusions IS NOT NULL
    ), 
    -- CTE to merge all toppings (including extras and excluding exclusions)
    all_ingredients_id_cte AS (
        (SELECT * FROM pizza_toppings_id_cte EXCEPT SELECT * FROM exclusions_id_cte)
        UNION ALL
        (SELECT * FROM extras_id_cte)
    ), 
    -- CTE to count and aggregate topping names
    all_ingredients_with_count_cte AS (
        SELECT 
            order_id, customer_id, pizza_name, topping_name, COUNT(topping_name) AS topping_count 
        FROM 
            all_ingredients_id_cte 
        LEFT JOIN 
            pizza_runner.pizza_toppings USING (topping_id) 
        LEFT JOIN 
            pizza_runner.pizza_names USING (pizza_id) 
        GROUP BY 
            order_id, customer_id, pizza_name, topping_name
    )
-- Main query to generate ordered ingredient lists
SELECT 
    order_id, 
    customer_id, 
    pizza_name, 
    CONCAT(
        pizza_name, ': ', 
        STRING_AGG(
            CONCAT((CASE WHEN topping_count > 1 THEN '2x' ELSE '' END), topping_name), ', '
        )
    ) AS topping_name_with_count 
FROM 
    all_ingredients_with_count_cte 
GROUP BY 
    order_id, customer_id, pizza_name;
```

| order_id | customer_id | pizza_name | topping_name_with_count                                                             |
| -------- | ----------- | ---------- | ----------------------------------------------------------------------------------- |
| 1        | 101         | Meatlovers | Meatlovers: BBQ Sauce, Bacon, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami   |
| 2        | 101         | Meatlovers | Meatlovers: BBQ Sauce, Bacon, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami   |
| 3        | 102         | Meatlovers | Meatlovers: BBQ Sauce, Bacon, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami   |
| 3        | 102         | Vegetarian | Vegetarian: Cheese, Mushrooms, Onions, Peppers, Tomato Sauce, Tomatoes              |
| 4        | 103         | Meatlovers | Meatlovers: BBQ Sauce, Bacon, Beef, Chicken, Mushrooms, Pepperoni, Salami           |
| 4        | 103         | Vegetarian | Vegetarian: Mushrooms, Onions, Peppers, Tomato Sauce, Tomatoes                      |
| 5        | 104         | Meatlovers | Meatlovers: BBQ Sauce, 2xBacon, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami |
| 6        | 101         | Vegetarian | Vegetarian: Cheese, Mushrooms, Onions, Peppers, Tomato Sauce, Tomatoes              |
| 7        | 105         | Vegetarian | Vegetarian: Bacon, Cheese, Mushrooms, Onions, Peppers, Tomato Sauce, Tomatoes       |
| 8        | 102         | Meatlovers | Meatlovers: BBQ Sauce, Bacon, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami   |
| 9        | 103         | Meatlovers | Meatlovers: BBQ Sauce, 2xBacon, Beef, 2xChicken, Mushrooms, Pepperoni, Salami       |
| 10       | 104         | Meatlovers | Meatlovers: 2xBacon, Beef, 2xCheese, Chicken, Pepperoni, Salami                     |

### Q6. What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?

```SQL
-- Common Table Expressions (CTEs) to extract toppings, extras, and exclusions
WITH 
    -- CTE to extract pizza toppings
    pizza_toppings_id_cte AS (
        SELECT 
            order_id, 
            UNNEST(STRING_TO_ARRAY(toppings, ','))::NUMERIC AS topping_id 
        FROM 
            pizza_runner.pizza_recipes 
        LEFT JOIN 
            pizza_runner.customer_orders USING (pizza_id)
    ), 
    -- CTE to extract extra toppings
    extras_id_cte AS (
        SELECT 
            order_id, 
            UNNEST(STRING_TO_ARRAY(extras, ','))::NUMERIC AS topping_id 
        FROM 
            pizza_runner.customer_orders 
        WHERE 
            extras IS NOT NULL
    ), 
    -- CTE to extract exclusion toppings
    exclusions_id_cte AS (
        SELECT 
            order_id, 
            UNNEST(STRING_TO_ARRAY(exclusions, ','))::NUMERIC AS topping_id 
        FROM 
            pizza_runner.customer_orders 
        WHERE 
            exclusions IS NOT NULL
    ), 
    -- CTE to merge all toppings (including extras and excluding exclusions)
    all_ingredients_id_cte AS (
        (SELECT * FROM pizza_toppings_id_cte EXCEPT SELECT * FROM exclusions_id_cte)
        UNION ALL
        (SELECT * FROM extras_id_cte)
    ), 
    -- CTE to count and aggregate topping names
    all_ingredients_with_count_cte AS (
        SELECT 
            order_id, 
            topping_name, 
            COUNT(topping_name) AS topping_count 
        FROM 
            all_ingredients_id_cte 
        LEFT JOIN 
            pizza_runner.pizza_toppings USING (topping_id) 
        GROUP BY 
            order_id, topping_name
    )
-- Main query to calculate total quantity of each ingredient
SELECT 
    topping_name, 
    SUM(topping_count) AS total_qty 
FROM 
    all_ingredients_with_count_cte 
LEFT JOIN 
    pizza_runner.runner_orders USING (order_id) 
WHERE 
    cancellation IS NULL 
GROUP BY 
    topping_name 
ORDER BY 
    total_qty DESC;
```
| topping_name | total_qty |
| ------------ | --------- |
| Bacon        | 10        |
| Cheese       | 8         |
| Chicken      | 7         |
| Mushrooms    | 7         |
| Pepperoni    | 7         |
| Salami       | 7         |
| Beef         | 7         |
| BBQ Sauce    | 6         |
| Tomatoes     | 3         |
| Onions       | 3         |
| Peppers      | 3         |
| Tomato Sauce | 3         |

## 🍕 Case Study #2 - Pizza Runner

Curious for more? Get your hands on all the sections [here](../README.md).

© 2024 [Shiv Katira](https://github.com/shivkatira)