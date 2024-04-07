# 🍕 Data Cleaning
<p align="center">
<img src="../../img/2.png" align="center" width="400" height="400" >

## 📚 Table of Contents

* [My Solution](#💡-my-solution)
* [Case Study #2 - Pizza Runner](#🍕-case-study-2---pizza-runner)

## 💡 My Solution

```SQL
-- Update customer_orders table to handle null values and data types
UPDATE 
    pizza_runner.customer_orders 
SET 
    -- Set exclusions to NULL if it's an empty string or 'null'
    exclusions = (
        CASE 
            WHEN exclusions = '' OR exclusions = 'null' THEN NULL 
            ELSE exclusions 
        END
    ), 
    -- Set extras to NULL if it's an empty string or 'null'
    extras = (
        CASE 
            WHEN extras = '' OR extras = 'null' THEN NULL 
            ELSE extras 
        END
    );

-- Update runner_orders table to handle null values and data types
UPDATE 
    pizza_runner.runner_orders 
SET 
    -- Set pickup_time to NULL if it's 'null'
    pickup_time = (
        CASE 
            WHEN pickup_time = 'null' THEN NULL 
            ELSE pickup_time 
        END
    ), 
    -- Set distance to NULL if it's 'null' and remove non-numeric characters
    distance = (
        CASE 
            WHEN distance = 'null' THEN NULL 
            ELSE REGEXP_REPLACE(distance, '[^\.\d]', '', 'g')::NUMERIC 
        END
    ), 
    -- Set duration to NULL if it's 'null' and remove non-numeric characters
    duration = (
        CASE 
            WHEN duration = 'null' THEN NULL 
            ELSE REGEXP_REPLACE(duration, '\D', '', 'g')::NUMERIC 
        END
    ), 
    -- Set cancellation to NULL if it's 'NaN', empty string, or 'null'
    cancellation = (
        CASE 
            WHEN cancellation = 'NaN' OR cancellation = '' OR cancellation = 'null' THEN NULL 
            ELSE cancellation 
        END
    );
```

## 🍕 Case Study #2 - Pizza Runner

Curious for more? Get your hands on all the sections [here](../README.md).

© 2024 [Shiv Katira](https://github.com/shivkatira)