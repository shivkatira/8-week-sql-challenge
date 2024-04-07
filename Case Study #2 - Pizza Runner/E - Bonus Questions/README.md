# 🍕 E. Bonus Questions
<p align="center">
<img src="../../img/2.png" align="center" width="400" height="400" >

## 📚 Table of Contents

* [Case Study Questions](#❓-case-study-questions)
* [My Solution](#💡-my-solution)
* [Case Study #2 - Pizza Runner](#🍕-case-study-2---pizza-runner)

## ❓ Case Study Questions

- [If Danny wants to expand his range of pizzas - how would this impact the existing data design? Write an INSERT statement to demonstrate what would happen if a new Supreme pizza with all the toppings was added to the Pizza Runner menu?](#if-danny-wants-to-expand-his-range-of-pizzas---how-would-this-impact-the-existing-data-design-write-an-insert-statement-to-demonstrate-what-would-happen-if-a-new-supreme-pizza-with-all-the-toppings-was-added-to-the-pizza-runner-menu) 

## 💡 My Solution

### If Danny wants to expand his range of pizzas - how would this impact the existing data design? Write an INSERT statement to demonstrate what would happen if a new Supreme pizza with all the toppings was added to the Pizza Runner menu?

```SQL
-- To expand the range of pizzas, we would need to update the pizza_names table with the new pizza name 
-- and the pizza_recipes table with the toppings for the new pizza.

-- Inserting a new pizza named "Supreme" into the pizza_names table
INSERT INTO pizza_runner.pizza_names("pizza_id", "pizza_name") 
VALUES (3, 'Supreme');

-- Verifying the addition of the new pizza in the pizza_names table
SELECT * FROM pizza_runner.pizza_names;

-- Adding the toppings for the new "Supreme" pizza into the pizza_recipes table
INSERT INTO pizza_runner.pizza_recipes ("pizza_id", "toppings") 
VALUES (3, '1,2,3,4,5,6,7,8,9,10,11,12');

-- Verifying the addition of toppings for the new pizza in the pizza_recipes table
SELECT * FROM pizza_runner.pizza_recipes;
```

| pizza_id | pizza_name |
| -------- | ---------- |
| 1        | Meatlovers |
| 2        | Vegetarian |
| 3        | Supreme    |

| pizza_id | toppings                   |
| -------- | -------------------------- |
| 1        | 1, 2, 3, 4, 5, 6, 8, 10    |
| 2        | 4, 6, 7, 9, 11, 12         |
| 3        | 1,2,3,4,5,6,7,8,9,10,11,12 |

## 🍕 Case Study #2 - Pizza Runner

Curious for more? Get your hands on all the sections [here](../README.md).

© 2024 [Shiv Katira](https://github.com/shivkatira)