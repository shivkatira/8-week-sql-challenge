# 📊 A. Customer Nodes Exploration
<p align="center">
<img src="../../img/4.png" align="center" width="400" height="400" >

## 📚 Table of Contents

* [Case Study Questions](#-case-study-questions)
* [My Solution](#-my-solution)
* [Case Study #4 - Data Bank](#-case-study-4---data-bank)

## ❓ Case Study Questions

1. [How many unique nodes are there on the Data Bank system?](#q1-how-many-unique-nodes-are-there-on-the-data-bank-system)
2. [What is the number of nodes per region?](#q2-what-is-the-number-of-nodes-per-region)
3. [How many customers are allocated to each region?](#q3-how-many-customers-are-allocated-to-each-region)
4. [How many days on average are customers reallocated to a different node?](#q4-how-many-days-on-average-are-customers-reallocated-to-a-different-node)
5. [What is the median, 80th and 95th percentile for this same reallocation days metric for each region?](#q5-what-is-the-median-80th-and-95th-percentile-for-this-same-reallocation-days-metric-for-each-region)

## 💡 My Solution

### Q1. How many unique nodes are there on the Data Bank system?

```SQL
SELECT 
    COUNT(DISTINCT node_id) AS unique_nodes -- Count the number of unique nodes in the system
FROM 
    data_bank.customer_nodes; -- Use customer_nodes table to get node data
```

| unique_nodes |
| ------------ |
| 5            |

### Q2. What is the number of nodes per region?

```SQL
SELECT 
    region_name, -- Select region name
    COUNT(node_id) AS node_count -- Count the number of nodes per region
FROM 
    data_bank.customer_nodes -- Use customer_nodes table to get node data
    LEFT JOIN data_bank.regions USING(region_id) -- Join with regions table using region ID to get region names
GROUP BY 
    region_name; -- Group results by region name
```

| region_name | unique_nodes |
| ----------- | ------------ |
| America     | 735          |
| Australia   | 770          |
| Africa      | 714          |
| Asia        | 665          |
| Europe      | 616          |

### Q3. How many customers are allocated to each region?

```SQL
SELECT 
    region_name, -- Select region name
    COUNT(DISTINCT customer_id) AS customer_count -- Count the number of unique customers allocated to each region
FROM 
    data_bank.customer_nodes -- Use customer_nodes table to get customer and region data
    LEFT JOIN data_bank.regions USING(region_id) -- Join with regions table using region_id to get region names
GROUP BY 
    region_name; -- Group results by region name to get customer count per region
```

| region_name | customer_count |
| ----------- | -------------- |
| Africa      | 102            |
| America     | 105            |
| Asia        | 95             |
| Australia   | 110            |
| Europe      | 88             |

### Q4. How many days on average are customers reallocated to a different node?

```SQL
SELECT 
    ROUND(AVG(DATE_PART('DAY', AGE(end_date, start_date)))::NUMERIC, 2) AS average_reallocation_time -- Calculate average days customers are reallocated to a different node, rounded to 2 decimal places
FROM 
    data_bank.customer_nodes; -- Use customer_nodes table to get start and end dates for reallocation
```
| average_reallocation_time |
| ------------------------- |
| 14.40                     |

### Q5. What is the median, 80th and 95th percentile for this same reallocation days metric for each region?

```SQL
SELECT 
    region_name, -- Select region name
    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY DATE_PART('DAY', AGE(end_date, start_date))) AS median_reallocation_time, -- Calculate median reallocation time
    PERCENTILE_CONT(0.8) WITHIN GROUP (ORDER BY DATE_PART('DAY', AGE(end_date, start_date))) AS percentile_80_reallocation_time, -- Calculate 80th percentile reallocation time
    PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY DATE_PART('DAY', AGE(end_date, start_date))) AS percentile_95_reallocation_time -- Calculate 95th percentile reallocation time
FROM 
    data_bank.customer_nodes -- Use customer_nodes table to get start and end dates for reallocation
    LEFT JOIN data_bank.regions USING (region_id) -- Join with regions table using region_id to get region names
GROUP BY 
    region_name; -- Group results by region name to aggregate metrics per region
```
| region_name | median_reallocation_time | percentile_80_reallocation_time | percentile_95_reallocation_time |
| ----------- | ------------------------ | ------------------------------- | ------------------------------- |
| Africa      | 14                       | 24                              | 28                              |
| America     | 15                       | 23                              | 28                              |
| Asia        | 15                       | 23                              | 28                              |
| Australia   | 14                       | 23                              | 28                              |
| Europe      | 14                       | 24                              | 28                              |

## 📊 Case Study #4 - Data Bank

Curious for more? Get your hands on all the sections [here](../README.md).

© 2024 [Shiv Katira](https://github.com/shivkatira)