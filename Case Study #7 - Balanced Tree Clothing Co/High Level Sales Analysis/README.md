# 🏔️ High Level Sales Analysis
<p align="center">
<img src="../../img/7.png" align="center" width="400" height="400" >

## 📚 Table of Contents

* [Case Study Questions](#-case-study-questions)
* [My Solution](#-my-solution)
* [Case Study #7 - Balanced Tree Clothing Co](#️-case-study-7---balanced-tree-clothing-co)

## ❓ Case Study Questions

1. [What was the total quantity sold for all products?](#q1-what-was-the-total-quantity-sold-for-all-products)
2. [What is the total generated revenue for all products before discounts?](#q2-what-is-the-total-generated-revenue-for-all-products-before-discounts)
3. [What was the total discount amount for all products?](#q3-what-was-the-total-discount-amount-for-all-products)

## 💡 My Solution

### Q1. What was the total quantity sold for all products?

```SQL
SELECT 
    SUM(qty) AS total_quantity_sold -- Calculate total quantity sold for all products
FROM 
    balanced_tree.sales; -- Use sales table to get quantity sold data
```

| total_quantity_sold |
| ------------------- |
| 45216               |

### Q2. What is the total generated revenue for all products before discounts?

```SQL
SELECT 
    SUM(qty * price) AS total_revenue -- Calculate total generated revenue for all products before discounts
FROM 
    balanced_tree.sales; -- Use sales table to get quantity and price data
```

| total_revenue |
| ------------- |
| 1289453       |

### Q3. What was the total discount amount for all products?

```SQL
SELECT 
    ROUND(SUM(qty * price * discount / 100.0), 2) AS total_discount_amount -- Calculate total discount amount for all products
FROM 
    balanced_tree.sales; -- Use sales table to get quantity, price, and discount data
```

| total_discount |
| -------------- |
| 156229.14      |

## 🏔️ Case Study #7 - Balanced Tree Clothing Co

Curious for more? Get your hands on all the sections [here](../README.md).

© 2024 [Shiv Katira](https://github.com/shivkatira)