# 8 Weeks SQL Challenge - 2nd Week

### This [case study](https://8weeksqlchallenge.com/case-study-2/)  is provided by Danny Ma 

## Introduction
Did you know that over 115 million kilograms of pizza is consumed daily worldwide??? (Well according to Wikipedia anyway…)

Danny was scrolling through his Instagram feed when something really caught his eye - “80s Retro Styling and Pizza Is The Future!”

Danny was sold on the idea, but he knew that pizza alone was not going to help him get seed funding to expand his new Pizza Empire - so he had one more genius idea to combine with it - he was going to Uberize it - and so Pizza Runner was launched!

Danny started by recruiting “runners” to deliver fresh pizza from Pizza Runner Headquarters (otherwise known as Danny’s house) and also maxed out his credit card to pay freelance developers to build a mobile app to accept orders from customers.


## Problem Statement
Because Danny had a few years of experience as a data scientist - he was very aware that data collection was going to be critical for his business’ growth.

He has prepared for us an entity relationship diagram of his database design but requires further assistance to clean his data and apply some basic calculations so he can better direct his runners and optimise Pizza Runner’s operations.

All datasets exist within the pizza_runner database schema - be sure to include this reference within your SQL scripts as you start exploring the data and answering the case study questions.

*See the complete tables [here](https://8weeksqlchallenge.com/case-study-2/)*


## Case Study Questions
This case study has LOTS of questions - they are broken up by area of focus including:
* Pizza Metrics
*  and Customer Experience
* Ingredient Optimisation
* Pricing and Ratings
* Bonus DML Challenges (DML = Data Manipulation Language)

## Create tables

```sql
CREATE TABLE IF NOT EXISTS runners (
  runner_id INTEGER,
  registration_date DATE
);
INSERT INTO runners
  (runner_id, registration_date)
VALUES
  (1, '2021-01-01'),
  (2, '2021-01-03'),
  (3, '2021-01-08'),
  (4, '2021-01-15');
```

```sql
CREATE TABLE customer_orders (
  order_id INTEGER,
  customer_id INTEGER,
  pizza_id INTEGER,
  exclusions VARCHAR(4),
  extras VARCHAR(4),
  order_time TIMESTAMP
);

INSERT INTO customer_orders
  (order_id, customer_id, pizza_id, exclusions, extras, order_time)
VALUES
  ('1', '101', '1', '', '', '2020-01-01 18:05:02'),
  ('2', '101', '1', '', '', '2020-01-01 19:00:52'),
  ('3', '102', '1', '', '', '2020-01-02 23:51:23'),
  ('3', '102', '2', '', NULL, '2020-01-02 23:51:23'),
  ('4', '103', '1', '4', '', '2020-01-04 13:23:46'),
  ('4', '103', '1', '4', '', '2020-01-04 13:23:46'),
  ('4', '103', '2', '4', '', '2020-01-04 13:23:46'),
  ('5', '104', '1', 'null', '1', '2020-01-08 21:00:29'),
  ('6', '101', '2', 'null', 'null', '2020-01-08 21:03:13'),
  ('7', '105', '2', 'null', '1', '2020-01-08 21:20:29'),
  ('8', '102', '1', 'null', 'null', '2020-01-09 23:54:33'),
  ('9', '103', '1', '4', '1, 5', '2020-01-10 11:22:59'),
  ('10', '104', '1', 'null', 'null', '2020-01-11 18:34:49'),
  ('10', '104', '1', '2, 6', '1, 4', '2020-01-11 18:34:49');
```
```sql
CREATE TABLE runner_orders (
  order_id INTEGER,
  runner_id INTEGER,
  pickup_time VARCHAR(19),
  distance VARCHAR(7),
  duration VARCHAR(10),
  cancellation VARCHAR(23)
);

INSERT INTO runner_orders
  (order_id, runner_id, pickup_time, distance, duration, cancellation)
VALUES
  ('1', '1', '2020-01-01 18:15:34', '20km', '32 minutes', ''),
  ('2', '1', '2020-01-01 19:10:54', '20km', '27 minutes', ''),
  ('3', '1', '2020-01-03 00:12:37', '13.4km', '20 mins', NULL),
  ('4', '2', '2020-01-04 13:53:03', '23.4', '40', NULL),
  ('5', '3', '2020-01-08 21:10:57', '10', '15', NULL),
  ('6', '3', 'null', 'null', 'null', 'Restaurant Cancellation'),
  ('7', '2', '2020-01-08 21:30:45', '25km', '25mins', 'null'),
  ('8', '2', '2020-01-10 00:15:02', '23.4 km', '15 minute', 'null'),
  ('9', '2', 'null', 'null', 'null', 'Customer Cancellation'),
  ('10', '1', '2020-01-11 18:50:20', '10km', '10minutes', 'null');
```

```sql
CREATE TABLE pizza_name (
pizza_id INTEGER,
pizza_name VARCHAR(50)
);
INSERT INTO pizza_name
(pizza_id, pizza_name)
VALUES
('1', 'Meat Lovers'),
('2', 'Vegetarian');
```

```sql
CREATE TABLE pizza_recipes (
pizza_id INTEGER,
toppings TEXT
);

INSERT INTO pizza_recipes
(pizza_id, toppings)
VALUES 
('1', '1,2,3,4,5,6,8,10'),
('2', '4,6,7,9,11,12');
```

```sql
CREATE TABLE pizza_toppings (
  topping_id INTEGER,
  topping_name TEXT
);

INSERT INTO pizza_toppings
  (topping_id, topping_name)
VALUES
  (1, 'Bacon'),
  (2, 'BBQ Sauce'),
  (3, 'Beef'),
  (4, 'Cheese'),
  (5, 'Chicken'),
  (6, 'Mushrooms'),
  (7, 'Onions'),
  (8, 'Pepperoni'),
  (9, 'Peppers'),
  (10, 'Salami'),
  (11, 'Tomatoes'),
  (12, 'Tomato Sauce');
```

## Case Study Questions -- Pizza Metrics

  1. How many pizzas were ordered?
  2. How many unique customer orders were made?
  3. How many successful orders were delivered by each runner?
  4. How many of each type of pizza was delivered?
  5. How many Vegetarian and Meatlovers were ordered by each customer?
  6. What was the maximum number of pizzas delivered in a single order?
  7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
  8. How many pizzas were delivered that had both exclusions and extras?
  9. What was the total volume of pizzas ordered for each hour of the day?
  10. What was the volume of orders for each day of the week?

## Solution -- Pizza Metrics
### 1. How many pizzas were ordered?

Since there are some null values in the tables, we need to clean them up.

```sql
UPDATE customer_orders
SET extras = null
WHERE extras = 'null' OR extras = ''
```
```sql
UPDATE customer_orders
SET exclusions = null
WHERE exclusions IN ('null','')
```
```sql
SELECT COUNT(order_id) AS num_order
FROM customer_orders
```

| num_order
| ---
| 14


### 2. How many unique customer orders were made?
```sql
SELECT COUNT(pizza_id)
FROM (
SELECT distinct pizza_id, exclusions, extras
FROM customer_orders
) AS unique_pizza
```

| unique_pizza
| ---
| 8

###  3. How many successful orders were delivered by each runner?
Since there are some null values in the tables, we need to clean them up.

```sql
UPDATE runner_orders
SET cancellation = null
WHERE cancellation = 'null' 
```
```sql
UPDATE runner_orders
SET duration = null
WHERE duration= 'null' 
```
```sql
UPDATE runner_orders
SET distance = null
WHERE distance= 'null'
```
```sql
UPDATE runner_orders
SET pickup_time = null
WHERE pickup_time= 'null'
```
```sql
SELECT runner_id, COUNT(order_id) AS num_order
FROM runner_orders
WHERE duration IS NOT NULL
GROUP BY runner_id
```

| runner_id | num_order
| --- | ---
| 1 | 4
| 2 | 3
| 3 | 1

###  4. How many of each type of pizza was delivered?
```sql
SELECT pizza_id, COUNT(pizza_id) AS num_order
FROM (
SELECT customer_orders.order_id, customer_orders.pizza_id, runner_orders.runner_id, runner_orders.cancellation
FROM customer_orders
LEFT JOIN runner_orders
ON customer_orders.order_id = runner_orders.order_id) AS order_pizzas
WHERE cancellation IS NULL
GROUP BY pizza_id
```

| pizza_id | num_order
| --- | ---
| 1 | 9
| 2 | 3

###  6. What was the maximum number of pizzas delivered in a single order?
```sql
SELECT max(num_order) AS max_order
FROM (
SELECT order_id, COUNT(pizza_id) AS num_order
FROM (
SELECT customer_orders.order_id, customer_orders.pizza_id, runner_orders.runner_id, runner_orders.cancellation
FROM customer_orders
LEFT JOIN runner_orders
ON customer_orders.order_id = runner_orders.order_id) AS order_pizzas
WHERE cancellation IS NULL
GROUP BY order_id 
) AS order_pizzas_num
```
| max_order
| ---
| 3

###  7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?

Pizzas had no changes:
```sql
WITH delivered_order AS(
SELECT customer_orders.order_id, customer_orders.customer_id, customer_orders.pizza_id, customer_orders.exclusions, customer_orders.extras, runner_orders.cancellation
FROM customer_orders
LEFT JOIN runner_orders
ON customer_orders.order_id = runner_orders.order_id
WHERE cancellation IS NULL)

SELECT customer_id, COUNT(order_id) AS num_pizzas
FROM delivered_order
WHERE exclusions IS NULL AND extras IS NULL
GROUP BY customer_id
ORDER BY num_pizzas DESC
```
| customer_id | num_pizzas
| --- | ---
| 102 | 3
| 101 | 2
| 103 | 1

Pizzas had at least 1 change:
```sql
WITH delivered_order AS(
SELECT customer_orders.order_id, customer_orders.customer_id, customer_orders.pizza_id, customer_orders.exclusions, customer_orders.extras, runner_orders.cancellation
FROM customer_orders
LEFT JOIN runner_orders
ON customer_orders.order_id = runner_orders.order_id
WHERE cancellation IS NULL)

SELECT customer_id, COUNT(order_id) AS num_pizzas
FROM delivered_order
WHERE exclusions IS NOT NULL OR extras IS NOT NULL
GROUP BY customer_id
ORDER BY num_pizzas DESC
```
| customer_id | num_pizzas
| --- | ---
| 103 | 3
| 104 | 2
| 105 | 1

###  8. How many pizzas were delivered that had both exclusions and extras?

```sql
WITH delivered_order AS(
SELECT customer_orders.order_id, customer_orders.customer_id, customer_orders.pizza_id, customer_orders.exclusions, customer_orders.extras, runner_orders.cancellation
FROM customer_orders
LEFT JOIN runner_orders
ON customer_orders.order_id = runner_orders.order_id
WHERE cancellation IS NULL)

SELECT customer_id, COUNT(order_id) AS num_pizzas
FROM delivered_order
WHERE exclusions IS NOT NULL AND extras IS NOT NULL
GROUP BY customer_id
ORDER BY num_pizzas DESC
```
| customer_id | num_pizzas
| --- | ---
| 104 | 1

###  9. What was the total volume of pizzas ordered for each hour of the day?
```sql
SELECT order_hour, COUNT(order_hour) AS order_by_hour
FROM (
SELECT order_id, pizza_id, HOUR(order_time) AS order_hour
FROM customer_orders
) AS customer_orders_hour
GROUP BY order_hour
ORDER BY order_by_hour DESC
```

| order_hour | order_by_hour
| --- | ---
| 18 | 3
| 23 | 3
| 13 | 3
| 21 | 3
| 19 | 1
| 11 | 1

### 10. What was the volume of orders for each day of the week?
```sql
WITH customer_order_weekday AS (
SELECT order_id, pizza_id, 
CASE
  WHEN weekday(order_time) = 1 THEN 'Sunday'
  WHEN weekday(order_time) = 2 THEN 'Monday'
  WHEN weekday(order_time) = 3 THEN 'Tuesday'
  WHEN weekday(order_time) = 4 THEN 'Wednesday'
  WHEN weekday(order_time) = 5 THEN 'Thursday'
  WHEN weekday(order_time) = 6 THEN 'Friday'
  ELSE 'Saturday'
  END AS order_weekday
FROM customer_orders
)

SELECT order_weekday, count(order_id) AS order_by_weekday
FROM customer_order_weekday
GROUP BY order_weekday
ORDER BY order_by_weekday DESC
```

| order_weekday | order_by_weekday
| --- | ---
| Monday | 5
| Thursday | 5
| Tuesday | 3
| Wednesday | 1