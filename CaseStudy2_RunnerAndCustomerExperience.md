# 8 Weeks SQL Challenge - 2nd Week

### This [case study](https://8weeksqlchallenge.com/case-study-2/)  is provided by Danny Ma 

### This is the continuation of a case study, see the description, problem statement, tables and the first questions in [Repository](https://github.com/vieluo/-MYSQL-Case-Study-2/blob/main/CaseStudy2_PizzaMetrics.md)

#

## Case Study Questions -- Runner and Customer Experience

1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)
2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?
3. Is there any relationship between the number of pizzas and how long the order takes to prepare?
4. What was the average distance travelled for each customer?
5. What was the difference between the longest and shortest delivery times for all orders?
6. What was the average speed for each runner for each delivery and do you notice any trend for these values?
7. What is the successful delivery percentage for each runner?

## Solution -- Runner and Customer Experience

### 1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)

```sql
SELECT reg_week, COUNT(runner_id) AS num_runner
FROM (
SELECT runner_id, 
CASE 
WHEN registration_date >='2021-01-01' AND registration_date < '2021-01-08' THEN 'week 1'
WHEN registration_date >='2021-01-08' AND registration_date < '2021-01-15' THEN 'week 2'
WHEN registration_date >='2021-01-15' AND registration_date < '2021-01-22' THEN 'week 3'
ELSE 'week 4'
END AS reg_week
FROM runners
) AS runner_reg_week
GROUP BY reg_week
```

| reg_week | num_runner
| --- | ---
| week 1 | 2
| week 2 | 1
| week 3 | 1

### 2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?
Since the duration type in the table is string, we need to change it to numeric type in order to calculate it.

```sql
ALTER TABLE runner_orders
ADD duration_min INTEGER
```
```sql
UPDATE runner_orders
SET duration_min = LEFT(duration,2)
```
```sql
ALTER TABLE runner_orders
DROP COLUMN duration
```
```sql
SELECT runner_id, ROUND(AVG(duration_min),2) AS avg_duration_min
FROM runner_orders
WHERE cancellation IS NULL
GROUP BY runner_id
```

| runner_id | avg_duration_min
| --- | ---
| 1 | 22.25
| 2 | 26.67
| 3 | 15.00


### 3. Is there any relationship between the number of pizzas and how long the order takes to prepare?

The preperation time is pickup time - order time, and we need to change the data type of pickup time to date since is it as string.

```sql
ALTER TABLE runner_orders
MODIFY COLUMN pickup_time TIMESTAMPS
```
```sql
WITH temp1 AS (
SELECT customer_orders.order_id, customer_orders.pizza_id, customer_orders.order_time, runner_orders.pickup_time
FROM customer_orders
JOIN runner_orders
ON customer_orders.order_id = runner_orders.order_id
WHERE pickup_time IS NOT NULL
),
temp2 AS (
SELECT order_id, COUNT(pizza_id) AS num_pizza
FROM temp1
GROUP BY order_id
),
temp3 AS (
SELECT order_id, TIMESTAMPDIFF(minute, order_time, pickup_time) AS prepare_time
FROM temp1
)
SELECT temp2.num_pizza, temp3.prepare_time
FROM temp2
JOIN temp3
ON temp2.order_id = temp3.order_id
ORDER BY prepare_time
```
| num_pizza | prepare_time
| --- | ---
| 1 | 10
| 1 | 10
| 1 | 10
| 1 | 10
| 2 | 15
| 2 | 15
| 1 | 20
| 2 | 21
| 2 | 21
| 3 | 29
| 3 | 29
| 3 | 29

We can see that for most of the orders, the prepare time is longer when there is more pizzas ordered. 


### 4. What was the average distance travelled for each customer?

```sql
SELECT customer_id, ROUND (AVG (distance),2) AS avg_distance_by_customer
FROM(
SELECT customer_orders.order_id,customer_orders.customer_id,runner_orders.duration_min, runner_orders.distance, runner_orders.cancellation
FROM customer_orders
JOIN runner_orders
ON customer_orders.order_id = runner_orders.order_id
WHERE runner_orders.cancellation IS NULL
) AS temp
GROUP BY customer_id
```
| customer_id | avg_distance_by_customer
| --- | ---
| 101 | 20
| 102 | 16.73
| 103 | 23.4
| 104 | 10
| 105 | 25

### 5. What was the difference between the longest and shortest delivery times for all orders?

```sql
SELECT MAX(duration_min) AS max_duration, MIN(duration_min) AS min_duration, MAX(duration_min)-MIN(duration_min) AS different_duration
FROM(
SELECT customer_orders.order_id,customer_orders.customer_id,runner_orders.duration_min, runner_orders.distance, runner_orders.cancellation
FROM customer_orders
JOIN runner_orders
ON customer_orders.order_id = runner_orders.order_id
WHERE runner_orders.cancellation IS NULL
) AS temp
```
| max_duration | min_duration | different_duration
| --- | --- | ---
| 40 | 10 | 30

### 6. What was the average speed for each runner for each delivery and do you notice any trend for these values?
Since the data type of distance is VARCHAR we need to alter it to number in order to calculate the speed. 

```sql
UPDATE runner_orders
SET distance = REPLACE (distance, 'km','')
```
```sql
ALTER TABLE runner_orders
MODIFY COLUMN distance INTEGER
```
```sql
SELECT order_id, runner_id, ROUND (distance*1000/duration_min,2) AS speed_m_per_min
FROM casestudy2.runner_orders
WHERE cancellation IS NULL
```
| order_id | runner_id | speed_m_per_min
| --- | --- | ---
| 1 | 1 | 625.00
| 2 | 1 | 740.74
| 3 | 1 | 650.00
| 4 | 2 | 575.00
| 5 | 3 | 666.67
| 7 | 2 | 1000.00
| 8 | 2 | 1533.33
| 10 | 1 | 1000.00

```sql
SELECT runner_id, ROUND(AVG(distance*1000/duration_min),2) AS avg_speed_m_per_min
FROM runner_orders
WHERE cancellation IS NULL
GROUP BY runner_id
```
| runner_id | avg_speed_m_per_min
| --- | --- 
| 1 | 753.94
| 2 | 1036.11
| 3 | 666.67

We can say that runner 2 has faster speed compare to runner 1 and 3.

### 7. What is the successful delivery percentage for each runner?

```sql
WITH temp1 AS (
SELECT runner_id, COUNT(runner_id) AS total_order_by_runner
FROM runner_orders
GROUP BY runner_id
),
temp2 AS (
SELECT runner_id, COUNT(runner_id) AS total_finished_order_by_runner
FROM runner_orders
WHERE cancellation IS NULL
GROUP BY runner_id
),
temp3 AS (
SELECT temp1.runner_id, temp1.total_order_by_runner, temp2.total_finished_order_by_runner
FROM temp1
JOIN temp2 ON
temp1.runner_id = temp2.runner_id
)
SELECT runner_id, ROUND(total_finished_order_by_runner/total_order_by_runner*100,0) AS successful_delivery_percentage
FROM temp3
```
| runner_id | successful_delivery_percentage
| --- | --- 
| 1 | 100
| 2 | 75
| 3 | 50