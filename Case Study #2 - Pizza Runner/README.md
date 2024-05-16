# Case Study #2 - Pizza Runner
## Introduction
Did you know that over **115 million kilograms** of pizza is consumed daily worldwide??? (Well according to Wikipedia anyway…)

Danny was scrolling through his Instagram feed when something really caught his eye - “80s Retro Styling and Pizza Is The Future!”

Danny was sold on the idea, but he knew that pizza alone was not going to help him get seed funding to expand his new Pizza Empire - so he had one more genius idea to combine with it - he was going to Uberize it - and so Pizza Runner was launched!

Danny started by recruiting “runners” to deliver fresh pizza from Pizza Runner Headquarters (otherwise known as Danny’s house) and also maxed out his credit card to pay freelance developers to build a mobile app to accept orders from customers.

## Available Data
Because Danny had a few years of experience as a data scientist - he was very aware that data collection was going to be critical for his business’ growth.

He has prepared for us an entity relationship diagram of his database design but requires further assistance to clean his data and apply some basic calculations so he can better direct his runners and optimise Pizza Runner’s operations.

## Entity Relationship Diagram
![Pizza Runner](https://github.com/Geomatric15/SQL-Weekly-Challenge/assets/167914482/e6fd0e76-c784-446b-89f1-5ed683a9bac3)

## Case Study Questions
> [!NOTE]
> These questions were answered using postgreSQL
### Data Cleaning
Creating new clean temporary tables from the existing tables.

**Runner Orders Table**
```sql
CREATE TEMP TABLE runner_orders_temp AS 
SELECT order_id,
       runner_id,
       CASE WHEN pickup_time LIKE 'null' THEN NULL
            ELSE pickup_time END AS pickup_time,
       CASE WHEN distance LIKE '%km' THEN TRIM('km' FROM distance)
            WHEN distance LIKE 'null' THEN NULL 
            ELSE distance END AS distance,
       CASE WHEN duration LIKE '%minutes' THEN TRIM('minutes' FROM duration)
            WHEN duration LIKE '%mins' THEN TRIM('mins' FROM duration)                                  			    
            WHEN duration LIKE '%minute' THEN TRIM('minute' FROM duration)
            WHEN duration LIKE 'null' THEN NULL
            ELSE duration END AS duration,
       CASE WHEN cancellation LIKE 'null' THEN NULL
            WHEN cancellation LIKE '' THEN NULL
            ELSE cancellation END AS cancellation
FROM runner_orders;

ALTER TABLE runner_orders_temp
ALTER COLUMN pickup_time TYPE TIMESTAMP USING(pickup_time::TIMESTAMP WITHOUT TIME ZONE),
ALTER COLUMN distance TYPE NUMERIC USING(distance::NUMERIC),
ALTER COLUMN duration TYPE NUMERIC USING(duration::NUMERIC);
```

**Customer Orders Table**
```sql
CREATE TEMP TABLE customer_orders_temp AS 
SELECT order_id,
       customer_id,
       pizza_id,
       CASE WHEN exclusion LIKE 'null' THEN NULL
            WHEN exclusion LIKE '' THEN NULL
            ELSE exclusion END AS exclusion,
       CASE WHEN extras LIKE 'null' THEN NULL
            WHEN extras LIKE '' THEN NULL
            ELSE extras END AS extras,
       order_time
FROM customer_orders;
```

### A. Pizza Metrics
#### 1.) How many pizzas were ordered?
```sql
SELECT COUNT(pizza_id) AS pizza_ordered
FROM customer_orders_temp;
```
**Output:**

#### 2.) How many unique customer orders were made?
```sql
SELECT COUNT(DISTINCT order_id) AS unique_ordered
FROM customer_orders_temp;
```
**Output:**

#### 3.) How many successful orders were delivered by each runner?
```sql
SELECT runner_id,
       COUNT(order_id) AS delivered_count
FROM runner_orders_temp
WHERE distance IS NOT NULL
GROUP BY runner_id;
```
**Output:**

#### 4.) How many of each type of pizza was delivered?
```sql
SELECT CDT.pizza_id,
       COUNT(CDT.pizza_id) AS delivered_count
FROM customer_orders_temp CDT
INNER JOIN runner_orders_temp ROT
USING(order_id)
WHERE cancellation IS NULL
GROUP BY CDT.pizza_id;
```
**Output:**

#### 5.) How many Vegetarian and Meatlovers were ordered by each customer?
```sql
SELECT customer_id,
       pizza_id,
       COUNT(pizza_id) AS ordered_count
FROM customer_orders_temp
GROUP BY customer_id, pizza_id
ORDER BY customer_id;
```
**Output:**

#### 6.) What was the maximum number of pizzas delivered in a single order?
```sql
SELECT order_id,
       COUNT(pizza_id) AS total
FROM customer_orders_temp COT
INNER JOIN runner_orders_temp ROT
USING(order_id)
WHERE cancellation IS NULL
GROUP BY order_id
ORDER BY total DESC
LIMIT 1;
```
**Output:**

#### 7.) For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
```sql
SELECT customer_id,
       SUM(CASE WHEN exclusion IS NULL
                 AND extras IS NULL THEN 1 END) AS non_changes,
       SUM(CASE WHEN exclusion IS NOT NULL 
                  OR extras IS NOT NULL THEN 1 END) AS changes
FROM customer_orders_temp
INNER JOIN runner_orders_temp
USING(order_id)
WHERE cancellation IS NULL
GROUP BY customer_id;
```
**Output:**

#### 8.) How many pizzas were delivered that had both exclusions and extras?
```sql
SELECT COUNT(*) delivered_pizza_with_both_changes
FROM customer_orders_temp
INNER JOIN runner_orders_temp
USING(order_id)
WHERE cancellation IS NULL 
  AND exclusion IS NOT NULL
  AND extras IS NOT NULL;
```
**Output:**

#### 9.) What was the total volume of pizzas ordered for each hour of the day?
```sql
SELECT COUNT(*) no_of_orders,
       EXTRACT(HOUR FROM order_time) AS hour
FROM customer_orders_temp
GROUP BY hour
ORDER BY hour ASC;
```
**Output:**

#### 10.) What was the volume of orders for each day of the week?
```sql
SELECT COUNT(*) no_of_orders,
       EXTRACT(DOW FROM order_time) AS day_of_week
FROM customer_orders_temp
GROUP BY day_of_week
ORDER BY day_of_week;
```
**Output:**

