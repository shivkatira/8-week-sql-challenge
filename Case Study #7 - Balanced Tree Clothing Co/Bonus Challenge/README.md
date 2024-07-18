# 🏔️ Bonus Challenge
<p align="center">
<img src="../../img/7.png" align="center" width="400" height="400" >

## 📚 Table of Contents

* [Case Study Questions](#-case-study-questions)
* [My Solution](#-my-solution)
* [Case Study #7 - Balanced Tree Clothing Co](#️-case-study-7---balanced-tree-clothing-co)

## ❓ Case Study Questions

* [Use a single SQL query to transform the `product_hierarchy` and `product_prices` datasets to the `product_details` table.](#use-a-single-sql-query-to-transform-the-product_hierarchy-and-product_prices-datasets-to-the-product_details-table)

## 💡 My Solution

### Use a single SQL query to transform the `product_hierarchy` and `product_prices` datasets to the `product_details` table.

```SQL
SELECT 
    product_prices.product_id, -- Select product ID
    product_prices.price, -- Select product price
    CONCAT(style_level.level_text, ' ', segment_level.level_text, ' - ', category_level.level_text) AS product_name, -- Concatenate style, segment, and category names to create product name
    category_level.id AS category_id, -- Select category ID
    segment_level.id AS segment_id, -- Select segment ID
    style_level.id AS style_id, -- Select style ID
    category_level.level_text AS category_name, -- Select category name
    segment_level.level_text AS segment_name, -- Select segment name
    style_level.level_text AS style_name -- Select style name
FROM 
    balanced_tree.product_prices -- Use product prices table to get product prices
    LEFT JOIN balanced_tree.product_hierarchy AS style_level ON product_prices.id = style_level.id -- Join with product hierarchy for style level
    LEFT JOIN balanced_tree.product_hierarchy AS segment_level ON style_level.parent_id = segment_level.id -- Join with product hierarchy for segment level
    LEFT JOIN balanced_tree.product_hierarchy AS category_level ON segment_level.parent_id = category_level.id; -- Join with product hierarchy for category level
```

| product_id | price | product_name                     | category_id | segment_id | style_id | category_name | segment_name | style_name          |
| ---------- | ----- | -------------------------------- | ----------- | ---------- | -------- | ------------- | ------------ | ------------------- |
| c4a632     | 13    | Navy Oversized Jeans - Womens    | 1           | 3          | 7        | Womens        | Jeans        | Navy Oversized      |
| e83aa3     | 32    | Black Straight Jeans - Womens    | 1           | 3          | 8        | Womens        | Jeans        | Black Straight      |
| e31d39     | 10    | Cream Relaxed Jeans - Womens     | 1           | 3          | 9        | Womens        | Jeans        | Cream Relaxed       |
| d5e9a6     | 23    | Khaki Suit Jacket - Womens       | 1           | 4          | 10       | Womens        | Jacket       | Khaki Suit          |
| 72f5d4     | 19    | Indigo Rain Jacket - Womens      | 1           | 4          | 11       | Womens        | Jacket       | Indigo Rain         |
| 9ec847     | 54    | Grey Fashion Jacket - Womens     | 1           | 4          | 12       | Womens        | Jacket       | Grey Fashion        |
| 5d267b     | 40    | White Tee Shirt - Mens           | 2           | 5          | 13       | Mens          | Shirt        | White Tee           |
| c8d436     | 10    | Teal Button Up Shirt - Mens      | 2           | 5          | 14       | Mens          | Shirt        | Teal Button Up      |
| 2a2353     | 57    | Blue Polo Shirt - Mens           | 2           | 5          | 15       | Mens          | Shirt        | Blue Polo           |
| f084eb     | 36    | Navy Solid Socks - Mens          | 2           | 6          | 16       | Mens          | Socks        | Navy Solid          |
| b9a74d     | 17    | White Striped Socks - Mens       | 2           | 6          | 17       | Mens          | Socks        | White Striped       |
| 2feb6b     | 29    | Pink Fluro Polkadot Socks - Mens | 2           | 6          | 18       | Mens          | Socks        | Pink Fluro Polkadot |

## 🏔️ Case Study #7 - Balanced Tree Clothing Co

Curious for more? Get your hands on all the sections [here](../README.md).

© 2024 [Shiv Katira](https://github.com/shivkatira)