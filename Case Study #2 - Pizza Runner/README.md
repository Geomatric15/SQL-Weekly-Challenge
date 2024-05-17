# Case Study #2 - Pizza Runner
![2](https://github.com/Geomatric15/SQL-Weekly-Challenge/assets/167914482/b63015c8-9fa3-4282-a081-c40579a4232f)

## Introduction
Did you know that over **115 million kilograms** of pizza is consumed daily worldwide??? (Well according to Wikipedia anyway…)

Danny was scrolling through his Instagram feed when something really caught his eye - “80s Retro Styling and Pizza Is The Future!”

Danny was sold on the idea, but he knew that pizza alone was not going to help him get seed funding to expand his new Pizza Empire - so he had one more genius idea to combine with it - he was going to Uberize it - and so Pizza Runner was launched!

Danny started by recruiting “runners” to deliver fresh pizza from Pizza Runner Headquarters (otherwise known as Danny’s house) and also maxed out his credit card to pay freelance developers to build a mobile app to accept orders from customers.

$~$

## Available Data
Because Danny had a few years of experience as a data scientist - he was very aware that data collection was going to be critical for his business’ growth.

He has prepared for us an entity relationship diagram of his database design but requires further assistance to clean his data and apply some basic calculations so he can better direct his runners and optimise Pizza Runner’s operations.

$~$

## Entity Relationship Diagram
![Pizza Runner](https://github.com/Geomatric15/SQL-Weekly-Challenge/assets/167914482/e6fd0e76-c784-446b-89f1-5ed683a9bac3)

$~$

## Case Study Questions
> [!NOTE]
> These questions were answered using postgreSQL

$~$

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
**Output:**
order_id | runner_id |     pickup_time     | distance | duration |      cancellation
| --- | --- | --- | --- | --- | --- |
|        1 |         1 | 2020-01-01 18:15:34 |       20 |       32 | |
|        2 |         1 | 2020-01-01 19:10:54 |       20 |       27 | |
|        3 |         1 | 2020-01-03 00:12:37 |     13.4 |       20 | |
|        4 |         2 | 2020-01-04 13:53:03 |     23.4 |       40 | |
|        5 |         3 | 2020-01-08 21:10:57 |       10 |       15 | |
|        6 |         3 |                     |          |          | Restaurant Cancellation |
|        7 |         2 | 2020-01-08 21:30:45 |       25 |       25 | |
|        8 |         2 | 2020-01-10 00:15:02 |     23.4 |       15 | | 
|        9 |         2 |                     |          |          | Customer Cancellation |
|       10 |         1 | 2020-01-11 18:50:20 |       10 |       10 | |
       
$~$

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
**Output:**
 order_id | customer_id | pizza_id | exclusion | extras |     order_time
| --- | --- | --- | --- | --- | --- |
|        1 |         101 |        1 |           |        | 2020-01-01 18:05:02 |
|        2 |         101 |        1 |           |        | 2020-01-01 19:00:52 |
|        3 |         102 |        1 |           |        | 2020-01-02 23:51:23 |
|        3 |         102 |        2 |           |        | 2020-01-02 23:51:23 |
|        4 |         103 |        1 | 4         |        | 2020-01-04 13:23:46 |
|        4 |         103 |        1 | 4         |        | 2020-01-04 13:23:46 |
|        4 |         103 |        2 | 4         |        | 2020-01-04 13:23:46 | 
|        5 |         104 |        1 |           | 1      | 2020-01-08 21:00:29 | 
|        6 |         101 |        2 |           |        | 2020-01-08 21:03:13 |
|        7 |         105 |        2 |           | 1      | 2020-01-08 21:20:29 |
|        8 |         102 |        1 |           |        | 2020-01-09 23:54:33 |
|        9 |         103 |        1 | 4         | 1, 5   | 2020-01-10 11:22:59 |
|       10 |         104 |        1 |           |        | 2020-01-11 18:34:49 |
|       10 |         104 |        1 | 2, 6      | 1, 4   | 2020-01-11 18:34:49 |
       
$~$

### A. Pizza Metrics
#### 1.) How many pizzas were ordered?
```sql
SELECT COUNT(pizza_id) AS pizza_ordered
FROM customer_orders_temp;
```
**Output:**
| pizza_ordered |
| --- |
| 14 |

#### 2.) How many unique customer orders were made?
```sql
SELECT COUNT(DISTINCT order_id) AS unique_ordered
FROM customer_orders_temp;
```
**Output:**
| unique_ordered |
| --- |
| 10 |

#### 3.) How many successful orders were delivered by each runner?
```sql
SELECT runner_id,
       COUNT(order_id) AS delivered_count
FROM runner_orders_temp
WHERE distance IS NOT NULL
GROUP BY runner_id;
```
**Output:**
| runner_id | delivered_count |
| --- | --- |
| 3 | 1 |
| 2 | 3 |
| 1 | 4 |

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
| pizza_id | delivered_count |
| --- | --- |
| 1 | 9 |
| 2 | 3 |

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
 customer_id | pizza_id | ordered_count
| --- | --- | --- |
| 101 | 2 | 1 |
| 101 | 1 | 2 |
| 102 | 2 | 1 |
| 102 | 1 | 2 |
| 103 | 2 | 1 |
| 103 | 1 | 3 |
| 104 | 1 | 3 |
| 105 | 2 | 1 |

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
 order_id | total
| --- | --- |
| 4 | 3 |

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
 customer_id | non_changes | changes
| --- | --- | --- |
| 101 | 2 |   |
| 102 | 3 |   |
| 103 |   | 3 |
| 104 | 1 | 2 |
| 105 |   | 1 |

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
 | delivered_pizza_with_both_changes |
| --- |
| 1 |

#### 9.) What was the total volume of pizzas ordered for each hour of the day?
```sql
SELECT COUNT(*) no_of_orders,
       EXTRACT(HOUR FROM order_time) AS hour
FROM customer_orders_temp
GROUP BY hour
ORDER BY hour ASC;
```
**Output:**
| no_of_orders | hour |
| --- | --- |
| 1 | 11 | 
| 3 | 13 | 
| 3 | 18 | 
| 1 | 19 |
| 3 | 21 |
| 3 | 23 |

#### 10.) What was the volume of orders for each day of the week?
```sql
SELECT COUNT(*) no_of_orders,
       EXTRACT(DOW FROM order_time) AS day_of_week
FROM customer_orders_temp
GROUP BY day_of_week
ORDER BY day_of_week;
```
**Output:**
| no_of_orders | day_of_week |
| --- | --- |
| 5 | 3 |
| 3 | 4 |
| 1 | 5 |
| 5 | 6 |

$~$

### B. Runner and Customer Experience

#### 1.) How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)
```sql
SELECT EXTRACT(WEEK FROM registration_date) AS week,
       COUNT(runner_id) AS runner_joined
FROM runners
GROUP BY week;
```

#### 2.) What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?
```sql
SELECT ROUND(AVG(time_diff)) AS average_time
FROM (
	SELECT DISTINCT order_id,
		   EXTRACT(MINUTE FROM (pickup_time - order_time)) AS time_diff
	FROM customer_orders_temp
	INNER JOIN runner_orders_temp
	USING(order_id)
	WHERE cancellation IS NULL);
```

#### 3.) Is there any relationship between the number of pizzas and how long the order takes to prepare?
```sql
SELECT number_ordered,
       ROUND(AVG(prep_time))
FROM (
	SELECT order_id,
		   COUNT(order_id) AS number_ordered,
		   EXTRACT(MINUTE FROM (pickup_time - order_time)) AS prep_time
	FROM customer_orders_temp
	INNER JOIN runner_orders_temp
	USING(order_id)
	WHERE cancellation IS NULL
	GROUP BY order_id, prep_time)
GROUP BY number_ordered
ORDER BY number_ordered;
```

#### 4.) What was the average distance travelled for each customer?
```sql
SELECT customer_id,
       ROUND(AVG(distance), 2) AS avg_distance_travelled
FROM runner_orders_temp
INNER JOIN customer_orders_temp
USING(order_id)
WHERE cancellation IS NULL
GROUP BY customer_id;
```

#### 5.) What was the difference between the longest and shortest delivery times for all orders?
```sql
SELECT MAX(duration) - MIN(duration) AS time_difference
FROM customer_orders_temp
INNER JOIN runner_orders_temp
USING(order_id)
WHERE cancellation IS NULL;
```

#### 6.) What was the average speed for each runner for each delivery and do you notice any trend for these values?
```sql
SELECT runner_id, ROUND(AVG(distance), 2) AS distance, ROUND(AVG(duration), 2) AS duration
FROM runner_orders_temp
WHERE cancellation IS NULL
GROUP BY runner_id;
```

#### 7.) What is the successful delivery percentage for each runner?
```sql
SELECT runner_id,
       ROUND(CAST(SUM(CASE WHEN cancellation IS NULL THEN 1 ELSE 0 END) AS DECIMAL) /
	   CAST(COUNT(runner_id) AS DECIMAL) * 100, 2) AS successful_delivery_percentage
FROM runner_orders_temp
GROUP BY runner_id;
```

