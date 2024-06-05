# 🍕 Data Cleaning
<p align="center">
<img src="../../img/2.png" align="center" width="400" height="400" >

## 📚 Table of Contents

* [My Solution](#-my-solution)
* [Case Study #2 - Pizza Runner](#-case-study-2---pizza-runner)

## 💡 My Solution

```SQL
-- Update exclusions and extras in customer_orders table
UPDATE pizza_runner.customer_orders 
SET 
    exclusions = ( 
        CASE 
            WHEN exclusions = '' OR exclusions = 'null' THEN NULL -- Set exclusions to NULL if empty or 'null'
            ELSE exclusions 
        END
    ),
    extras = ( 
        CASE 
            WHEN extras = '' OR extras = 'null' THEN NULL -- Set extras to NULL if empty or 'null'
            ELSE extras 
        END
    );

-- Select all records from customer_orders to verify updates
SELECT * 
FROM pizza_runner.customer_orders;
```

| order_id | customer_id | pizza_id | exclusions | extras | order_time               |
| -------- | ----------- | -------- | ---------- | ------ | ------------------------ |
| 1        | 101         | 1        | null       | null   | 2020-01-01T18:05:02.000Z |
| 2        | 101         | 1        | null       | null   | 2020-01-01T19:00:52.000Z |
| 3        | 102         | 1        | null       | null   | 2020-01-02T23:51:23.000Z |
| 3        | 102         | 2        | null       | null   | 2020-01-02T23:51:23.000Z |
| 4        | 103         | 1        | 4          | null   | 2020-01-04T13:23:46.000Z |
| 4        | 103         | 1        | 4          | null   | 2020-01-04T13:23:46.000Z |
| 4        | 103         | 2        | 4          | null   | 2020-01-04T13:23:46.000Z |
| 5        | 104         | 1        | null       | 1      | 2020-01-08T21:00:29.000Z |
| 6        | 101         | 2        | null       | null   | 2020-01-08T21:03:13.000Z |
| 7        | 105         | 2        | null       | 1      | 2020-01-08T21:20:29.000Z |
| 8        | 102         | 1        | null       | null   | 2020-01-09T23:54:33.000Z |
| 9        | 103         | 1        | 4          | 1, 5   | 2020-01-10T11:22:59.000Z |
| 10       | 104         | 1        | null       | null   | 2020-01-11T18:34:49.000Z |
| 10       | 104         | 1        | 2, 6       | 1, 4   | 2020-01-11T18:34:49.000Z |

```SQL
-- Update various columns in runner_orders table
UPDATE pizza_runner.runner_orders 
SET 
    pickup_time = ( 
        CASE 
            WHEN pickup_time = 'null' THEN NULL -- Set pickup_time to NULL if 'null'
            ELSE pickup_time 
        END
    ),
    distance = ( 
        CASE 
            WHEN distance = 'null' THEN NULL -- Set distance to NULL if 'null'
            ELSE REGEXP_REPLACE(distance, '[^\.\d]', '', 'g')::NUMERIC -- Remove non-numeric characters and convert to NUMERIC
        END
    ),
    duration = ( 
        CASE 
            WHEN duration = 'null' THEN NULL -- Set duration to NULL if 'null'
            ELSE REGEXP_REPLACE(duration, '\D', '', 'g')::NUMERIC -- Remove non-numeric characters and convert to NUMERIC
        END
    ),
    cancellation = ( 
        CASE 
            WHEN cancellation = 'NaN' OR cancellation = '' OR cancellation = 'null' THEN NULL -- Set cancellation to NULL if NaN, empty, or 'null'
            ELSE cancellation 
        END
    );

-- Select all records from runner_orders to verify updates
SELECT * 
FROM pizza_runner.runner_orders;
```

| order_id | runner_id | pickup_time         | distance | duration | cancellation            |
| -------- | --------- | ------------------- | -------- | -------- | ----------------------- |
| 1        | 1         | 2020-01-01 18:15:34 | 20       | 32       | null                    |
| 2        | 1         | 2020-01-01 19:10:54 | 20       | 27       | null                    |
| 3        | 1         | 2020-01-03 00:12:37 | 13.4     | 20       | null                    |
| 4        | 2         | 2020-01-04 13:53:03 | 23.4     | 40       | null                    |
| 5        | 3         | 2020-01-08 21:10:57 | 10       | 15       | null                    |
| 6        | 3         | null                | null     | null     | Restaurant Cancellation |
| 7        | 2         | 2020-01-08 21:30:45 | 25       | 25       | null                    |
| 8        | 2         | 2020-01-10 00:15:02 | 23.4     | 15       | null                    |
| 9        | 2         | null                | null     | null     | Customer Cancellation   |
| 10       | 1         | 2020-01-11 18:50:20 | 10       | 10       | null                    |

## 🍕 Case Study #2 - Pizza Runner

Curious for more? Get your hands on all the sections [here](../README.md).

© 2024 [Shiv Katira](https://github.com/shivkatira)