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

$~$

#### 2.) How many unique customer orders were made?
```sql
SELECT COUNT(DISTINCT order_id) AS unique_ordered
FROM customer_orders_temp;
```
**Output:**
| unique_ordered |
| --- |
| 10 |

$~$

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

$~$

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

$~$

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

$~$

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

$~$

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

$~$

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

$~$

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

$~$

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
GROUP BY week
ORDER BY runner_joined DESC;
```
**Output:**
| week | runner_joined |
| --- | --- |
|   53 |             2 |
|    1 |             1 |
|    2 |             1 |

$~$

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
**Output:**
| average_time |
| --- |
| 16 |

$~$

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
**Output:**
 number_ordered | prep_time
| --- | --- |
| 1 |    12 |
| 2 |    18 |
| 3 |    29 |

$~$

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
**Output:**
 customer_id | avg_distance_travelled
| --- | --- |
| 101 | 20.00 |
| 102 | 16.73 |
| 103 | 23.40 |
| 104 | 10.00 |
| 105 | 25.00 |

$~$

#### 5.) What was the difference between the longest and shortest delivery times for all orders?
```sql
SELECT MAX(duration) - MIN(duration) AS time_difference
FROM customer_orders_temp
INNER JOIN runner_orders_temp
USING(order_id)
WHERE cancellation IS NULL;
```
**Ouput:**
| time_difference |
| --- |
| 30 |

$~$

#### 6.) What was the average speed for each runner for each delivery and do you notice any trend for these values?
```sql
SELECT runner_id, ROUND(AVG(distance), 2) AS distance, ROUND(AVG(duration), 2) AS duration
FROM runner_orders_temp
WHERE cancellation IS NULL
GROUP BY runner_id;
```
**Output**
 runner_id | distance | duration
| --- | --- | --- |
| 1 | 15.85 | 22.25 |
| 2 | 23.93 | 26.67 |
| 3 | 10.00 | 15.00 |

$~$

#### 7.) What is the successful delivery percentage for each runner?
```sql
SELECT runner_id,
       ROUND(CAST(SUM(CASE WHEN cancellation IS NULL THEN 1 ELSE 0 END) AS DECIMAL) /
	   CAST(COUNT(runner_id) AS DECIMAL) * 100, 2) AS successful_delivery_percentage
FROM runner_orders_temp
GROUP BY runner_id;
```
**Ouput:**
 runner_id | successful_delivery_percentage
| --- | --- |
| 3 | 50.00 |
| 2 | 75.00 |
| 1 | 100.00 |

$~$

### C. Ingredient Optimisation

#### 1.) What are the standard ingredients for each pizza?
```sql
SELECT pizza_name AS pizza_type,
       STRING_AGG(topping_name, ', ') AS ingredients
FROM (
    SELECT pizza_id, 
        CAST(REGEXP_SPLIT_TO_TABLE(toppings, E',') AS INT) AS toppings
    FROM pizza_recipes
      ) PR
INNER JOIN pizza_names PN 
 ON PR.pizza_id = PN.pizza_id
INNER JOIN pizza_toppings PT
 on PR.toppings = PT.topping_id
GROUP BY pizza_name;
```
**Output:**
pizza_type |                              ingredients
| --- | --- |
| Meatlovers | Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami | 
| Vegetarian | Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce |

$~$

#### 2.) What was the most commonly added extra?
```sql
SELECT topping_name,
       COUNT(topping_name) AS times_added
FROM (
    SELECT CAST(REGEXP_SPLIT_TO_TABLE(extras, E',') AS INT) AS extras
    FROM customer_orders_temp) COT
INNER JOIN pizza_toppings PT
 ON COT.extras = PT.topping_id
GROUP BY topping_name
ORDER BY times_added DESC
LIMIT 1;
```
**Output:**
| topping_name | times_added | 
| --- | --- |
| Bacon | 4 |

$~$

#### 3.) What was the most common exclusion?
```sql
SELECT topping_name,
       COUNT(topping_name) AS times_excluded
FROM (
    SELECT CAST(REGEXP_SPLIT_TO_TABLE(exclusion, E',') AS INT) AS exclusion
    FROM customer_orders_temp) COT
INNER JOIN pizza_toppings PT
 ON COT.exclusion = PT.topping_id
GROUP BY topping_name
ORDER BY times_added DESC
LIMIT 1;
```
| topping_name | times_excluded | 
| --- | --- |
| Cheese | 4 |

$~$

> [!NOTE]
> We will create a new table for the next following questions.

**Creating new table: customer_orders_customized**
```sql
CREATE TEMP TABLE customer_orders_customized AS
SELECT order_id,
       customer_id,
       pizza_id,
       CASE WHEN exclusion LIKE 'null'
                 OR exclusion LIKE '' THEN NULL
            ELSE exclusion END AS exclusion,
       CASE WHEN extras LIKE 'null' 
                 OR extras LIKE '' THEN NULL
            ELSE extras END AS extras,
       order_time,
       unique_number
FROM (
      SELECT order_id,
             customer_id,
             pizza_id,
             REGEXP_SPLIT_TO_TABLE(exclusion, E',') AS exclusion,
             REGEXP_SPLIT_TO_TABLE(extras, E',') AS extras,
             order_time,
	     ROW_NUMBER() OVER() AS unique_number
       FROM customer_orders
      );


ALTER TABLE customer_orders_customized
ALTER COLUMN extras TYPE INTEGER USING(extras::INTEGER),
ALTER COLUMN exclusion TYPE INTEGER USING(exclusion::INTEGER);
```

$~$

#### 4.) Generate an order item for each record in the customers_orders table in the format of one of the following:
- Meat Lovers
- Meat Lovers - Exclude Beef
- Meat Lovers - Extra Bacon
- Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers

```sql
WITH joined_tables AS (
SELECT COC.customer_id,
       PN.pizza_name,
       COC.unique_number,
       STRING_AGG(PT.topping_name, ', ') AS exclusion_name,
       STRING_AGG(PT2.topping_name, ', ') AS extras_name
FROM customer_orders_customized COC
INNER JOIN pizza_names PN
 ON COC.pizza_id = PN.pizza_id	
LEFT JOIN pizza_toppings PT
 ON COC.exclusion = PT.topping_id 
LEFT JOIN pizza_toppings PT2
 ON COC.extras = PT2.topping_id
GROUP BY COC.customer_id, PN.pizza_name, COC.unique_number
ORDER BY customer_id ASC
)

SELECT CASE WHEN exclusion_name IS NOT NULL
                 AND extras_name IS NOT NULL 
                THEN CONCAT(pizza_name, ' - Exclude ', exclusion_name, ' - Include ', extras_name)
            WHEN exclusion_name IS NOT NULL
                 AND extras_name IS NULL
                THEN CONCAT(pizza_name, ' - Exclude ', exclusion_name)
            WHEN exclusion_name IS NULL
                 AND extras_name IS NOT NULL
                THEN CONCAT(pizza_name, ' - Include ', extras_name)
            ELSE pizza_name END AS item
FROM joined_tables;
```
**Output:**
                               | item |
| ------------------------------------------------------------------- |
| Meatlovers |
| Meatlovers |
| Vegetarian |
| Meatlovers |
| Meatlovers |
| Vegetarian |
| Meatlovers - Exclude Cheese |
| Meatlovers - Exclude Cheese |
| Meatlovers - Exclude Cheese - Include Bacon, Chicken |
| Vegetarian - Exclude Cheese |
| Meatlovers - Include Bacon |
| Meatlovers |
| Meatlovers - Exclude BBQ Sauce, Mushrooms - Include Bacon, Cheese |
| Vegetarian - Include Bacon |

$~$

#### 5.) Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients
- For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"
```sql
WITH joined_tables AS (
SELECT COC.order_id,
	   COC.customer_id,
	   COC.unique_number,
	   PN.pizza_name,
	   CAST(REGEXP_SPLIT_TO_TABLE(PR.toppings, E',') AS INT) AS ingredients,
	   PT.topping_name AS exclusion_name,
	   PT2.topping_name AS extras_name,
	   ROW_NUMBER() OVER(PARTITION BY COC.unique_number) AS sub_number
FROM customer_orders_customized COC
INNER JOIN pizza_names PN
 ON COC.pizza_id = PN.pizza_id
INNER JOIN pizza_recipes PR
 ON COC.pizza_id = PR.pizza_id
LEFT JOIN pizza_toppings PT
 ON COC.exclusion = PT.topping_id
LEFT JOIN pizza_toppings PT2
 ON COC.extras = PT2.topping_id
ORDER BY order_id, customer_id, unique_number, sub_number, ingredients
),

     aggregated_table AS (
SELECT order_id,
       customer_id,
       unique_number,
       pizza_name,
       sub_number,
       STRING_AGG(CASE WHEN extras_name = topping_name THEN CONCAT('2x ', topping_name)
		       WHEN exclusion_name = topping_name THEN NULL
		       ELSE topping_name END, ', ') AS ingredients
FROM joined_tables JT
INNER JOIN pizza_toppings PT
ON JT.ingredients = PT.topping_id
GROUP BY order_id, customer_id, unique_number, pizza_name, sub_number
)

SELECT CONCAT(pizza_name, ': ', ingredients) AS pizza_ordered
FROM aggregated_table;
```
**Output:**
                                   | pizza_ordered |
|--------------------------------------------------------------------------------------|
| Meatlovers: Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami |
| Meatlovers: Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami |
| Meatlovers: Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami |
| Vegetarian: Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce | 
| Meatlovers: Bacon, BBQ Sauce, Beef, Chicken, Mushrooms, Pepperoni, Salami | 
| Meatlovers: Bacon, BBQ Sauce, Beef, Chicken, Mushrooms, Pepperoni, Salami | 
| Vegetarian: Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce | 
| Meatlovers: 2x Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami | 
| Vegetarian: Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce | 
| Vegetarian: Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce | 
| Meatlovers: Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami | 
| Meatlovers: Bacon, BBQ Sauce, Beef, Cheese, 2x Chicken, Mushrooms, Pepperoni, Salami |
| Meatlovers: 2x Bacon, BBQ Sauce, Beef, Chicken, Mushrooms, Pepperoni, Salami |
| Meatlovers: Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami | 
| Meatlovers: 2x Bacon, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami | 
| Meatlovers: Bacon, BBQ Sauce, Beef, 2x Cheese, Chicken, Pepperoni, Salami |

$~$

#### 6.) What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?
```sql
WITH joined_tables AS (
	SELECT PT.topping_name AS exclusion_name,
		   PT2.topping_name AS extras_name,
		   CAST(REGEXP_SPLIT_TO_TABLE(PR.toppings, E',') AS INT) AS ingredient
	FROM customer_orders_customized COC
	INNER JOIN pizza_recipes PR
	ON COC.pizza_id = PR.pizza_id
	LEFT JOIN pizza_toppings PT
	ON COC.exclusion = PT.topping_id
	LEFT JOIN pizza_toppings PT2
	ON COC.extras = PT2.topping_id)
 
SELECT topping_name AS ingredients,
       SUM(CASE WHEN exclusion_name = topping_name THEN 0
                WHEN extras_name = topping_name THEN 2
			    ELSE 1 END) AS used_count
FROM joined_tables JT
LEFT JOIN pizza_toppings PT
 ON JT.ingredient = PT.topping_id
GROUP BY topping_name
ORDER BY used_count DESC;
```
**Output:**
 ingredients  | used_count
| --- | --- |
| Bacon        |         15 |
| Mushrooms    |         15 |
| Chicken      |         13 | 
| Cheese       |         13 |
| Pepperoni    |         12 | 
| Salami       |         12 |
| Beef         |         12 | 
| BBQ Sauce    |         11 |
| Tomato Sauce |          4 |
| Onions       |          4 | 
| Tomatoes     |          4 |
| Peppers      |          4 |

### D. Pricing and Ratings

#### 1.) If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?
```sql
SELECT SUM(CASE WHEN pizza_id = 1 THEN 12 
                WHEN pizza_id = 2 THEN 10 
			    END) AS total_revenue
FROM customer_orders_temp
INNER JOIN runner_orders_temp USING(order_id)
WHERE cancellation IS NULL;
```
**Output:*

$~$

#### 2.) What if there was an additional $1 charge for any pizza extras?
Add cheese is $1 extra
```sql
SELECT SUM(CASE WHEN pizza_id = 1 AND LENGTH(extras) IS NULL THEN 12
                WHEN pizza_id = 1 AND LENGTH(extras) = 1 THEN 13
		WHEN pizza_id = 1 AND LENGTH(extras) = 4 THEN 14
		WHEN pizza_id = 2 AND LENGTH(extras) IS NULL THEN 10
		WHEN pizza_id = 2 AND LENGTH(extras) = 1 THEN 11
		WHEN pizza_id = 2 AND LENGTH(extras) = 4 THEN 12
		 END) AS total_revenue
FROM customer_orders_temp
INNER JOIN runner_orders_temp USING(order_id)
WHERE cancellation IS NULL;
```
**Output:**

$~$

#### 3.) The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.
```sql
CREATE TABLE runner_rating (
 runner_id INT,
 order_id INT,
 delivery_status VARCHAR(50),
 rating INT);
 
INSERT INTO runner_rating (runner_id, order_id, delivery_status, rating)
VALUES(1, 1, 'successful', 3),
      (1, 2, 'successful', 4),
      (1, 3, 'successful', 3),
      (2, 4, 'successful', 2),
      (3, 5, 'successful', 3),
      (3, 6, 'cancelled', NULL),
      (2, 7, 'successful', 5),
      (2, 8, 'successful', 5),
      (2, 9, 'cancelled', NULL),
      (1, 10, 'successful', 5); 

SELECT * FROM runner_rating;
```
**Output:**

$~$

#### 4.) Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?
- customer_id
- order_id
- runner_id
- rating
- order_time
- pickup_time
- Time between order and pickup
- Delivery duration
- Average speed
- Total number of pizzas
```sql
SELECT COT.customer_id,
       COT.order_id,
       ROT.runner_id,
	   RR.rating,
	   COT.order_time,
	   ROT.pickup_time,
	   ROT.pickup_time - COT.order_time AS time_between_order_and_pickup,
	   ROT.duration AS delivery_duration,
	   ROUND(AVG(ROT.duration)) AS average_duration,
	   COUNT(customer_id) AS total_number_of_pizzas
FROM customer_orders_temp COT
INNER JOIN runner_orders_temp ROT USING(order_id)
LEFT JOIN runner_rating RR USING(order_id)
WHERE RR.delivery_status = 'successful'
GROUP BY COT.customer_id,
	     COT.order_id,
	     ROT.runner_id,
	     RR.rating,
	     COT.order_time,
	     ROT.pickup_time,
	     time_between_order_and_pickup,
	     delivery_duration;
```
**Output:**

$~$

#### 5.) If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?
```sql
SELECT ROUND(SUM(revenue - runner_cut)) AS total_profit
FROM (
	SELECT order_id,
		runner_id,
		ROUND(distance * 0.30, 2) AS runner_cut,
		SUM(CASE WHEN pizza_id = 1 THEN 12 
				WHEN pizza_id = 2 THEN 10 
				END) AS revenue
	FROM customer_orders_temp
	INNER JOIN runner_orders_temp USING(order_id)
	WHERE distance IS NOT NULL
	GROUP BY order_id, runner_id, runner_cut);
```
**Output:**

$~$

### E. Bonus Questions
If Danny wants to expand his range of pizzas - how would this impact the existing data design? Write an INSERT statement to demonstrate what would happen if a new Supreme pizza with all the toppings was added to the Pizza Runner menu?
```sql
INSERT INTO pizza_names (pizza_id, pizza_name)
VALUES(3, 'Supreme');

INSERT INTO pizza_recipes (pizza_id, toppings)
VALUES(3, '1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12');
```








