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


