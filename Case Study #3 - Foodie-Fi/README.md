# Case Study #3 - Foodie-Fi
![3](https://github.com/Geomatric15/SQL-Weekly-Challenge/assets/167914482/3369daba-0759-4e8f-8ceb-ad30efc10ede)

$~$

# Introduction
Subscription based businesses are super popular and Danny realised that there was a large gap in the market - he wanted to create a new streaming service that only had food related content - something like Netflix but with only cooking shows!

Danny finds a few smart friends to launch his new startup Foodie-Fi in 2020 and started selling monthly and annual subscriptions, giving their customers unlimited on-demand access to exclusive food videos from around the world!

Danny created Foodie-Fi with a data driven mindset and wanted to ensure all future investment decisions and new features were decided using data. This case study focuses on using subscription style digital data to answer important business questions.

$~$

# Available Data
Danny has shared the data design for Foodie-Fi and also short descriptions on each of the database tables - our case study focuses on only 2 tables but there will be a challenge to create a new table for the Foodie-Fi team.

All datasets exist within the foodie_fi database schema - be sure to include this reference within your SQL scripts as you start exploring the data and answering the case study questions.

$~$

# Entity Relationship Diagram
![Untitled (1)](https://github.com/Geomatric15/SQL-Weekly-Challenge/assets/167914482/066f28e1-e288-4184-bbdc-71684cee0f31)

$~$

# Case Study Questions
> [!NOTE]
> These questions were answered using postgreSQL

$~$

## A. Customer Journey
Based off the 8 sample customers provided in the sample from the subscriptions table, write a brief description about each customer’s onboarding journey.

Try to keep it as short as possible - you may also want to run some sort of join to make your explanations a bit easier!

$~$

## B. Data Analysis Questions

#### 1.) How many customers has Foodie-Fi ever had?
```sql
SELECT COUNT(DISTINCT customer_id) AS customer_count
FROM subscriptions;
```
**Output:**
| customer_count |
| --- |
| 1000 |

$~$

#### 2.) What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value
```sql
SELECT EXTRACT(MONTH FROM start_date) AS month,
       COUNT(*) AS count
FROM subscriptions
GROUP BY month
ORDER BY month;
```
**Output:**
 month | count
| --- | --- |
|     1 |   236 |
|     2 |   195 |
|     3 |   245 |
|     4 |   217 |
|     5 |   214 |
|     6 |   204 |
|     7 |   221 |
|     8 |   235 |
|     9 |   225 |
|    10 |   230 |
|    11 |   208 |
|    12 |   220 |

$~$

#### 3.) What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name
```sql
SELECT P.plan_id,
       P.plan_name,
       COUNT(S.customer_id) AS number_of_events
FROM subscriptions S
LEFT JOIN plans P
 ON S.plan_id = P.plan_id
WHERE EXTRACT(YEAR FROM start_date) = 2021
GROUP BY P.plan_name, P.plan_id
ORDER BY P.plan_id;
```
**Output:**
 plan_id |   plan_name   | number_of_events
| --- | --- | --- |
|       1 | basic monthly |                8 |
|       2 | pro monthly   |               60 |
|       3 | pro annual    |               63 |
|       4 | churn         |               71 |

$~$

#### 4.) What is the customer count and percentage of customers who have churned rounded to 1 decimal place?
```sql
SELECT COUNT(customer_id) AS customer_count,
       ROUND(COUNT(customer_id) / (SELECT CAST(COUNT(DISTINCT customer_id) AS DECIMAL) 
		                               FROM subscriptions) * 100 , 1) AS percentage
FROM subscriptions
WHERE plan_id = 4;
```
**Output:**
 customer_count | percentage
| --- | --- |
|            307 |       30.7 |

$~$

#### 5.) How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?
```sql
SELECT COUNT(customer_id) AS customer_count,
       ROUND(COUNT(customer_id) / (SELECT CAST(COUNT(DISTINCT customer_id) AS DECIMAL) FROM subscriptions) * 100) AS percentage
FROM(
	SELECT customer_id,
		   plan_id,
		   DENSE_RANK() OVER(PARTITION BY customer_id ORDER BY plan_id) AS rank
	FROM subscriptions)
WHERE plan_id = 4 and rank = 2;
```
**Output:**
 customer_count | percentage
| --- | --- |
|             92 |          9 |

$~$

#### 6.) What is the number and percentage of customer plans after their initial free trial?
```sql
WITH joined_table AS (
SELECT S.customer_id,
       P.plan_name,
	   ROW_NUMBER() OVER(PARTITION BY customer_id ORDER BY start_date) AS placed
FROM subscriptions S
LEFT JOIN plans P
 ON S.plan_id = P.plan_id
WHERE S.plan_id != 0
)

SELECT plan_name,
       COUNT(plan_name) AS customer_count,
	   ROUND(COUNT(plan_name) / (SELECT CAST(COUNT(*) AS DECIMAL) FROM joined_table WHERE placed = 1), 2) AS percentage
FROM joined_table
WHERE placed = 1
GROUP BY plan_name 
ORDER BY plan_name;
```
**Output:**
   plan_name   | customer_count | percentage
| --- | --- | --- |
| basic monthly |            546 |       0.55 |
| churn         |             92 |       0.09 |
| pro annual    |             37 |       0.04 |
| pro monthly   |            325 |       0.33 |
 
$~$

#### 7.) What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?
```sql
WITH customers_count AS (
	SELECT plan_id,
		     COUNT(plan_id) AS customer_count
	FROM subscriptions
	WHERE start_date <= '2020-12-31'
	GROUP BY plan_id)

SELECT plan_id,
       customer_count,
	   ROUND(customer_count / (SELECT SUM(customer_count) FROM customers_count) * 100, 2) AS percentage
FROM customers_count;
```
**Output:**
 plan_id | customer_count | percentage
| --- | --- | --- |
|       0 |           1000 |      40.85 |
|       1 |            538 |      21.98 |
|       3 |            195 |       7.97 | 
|       2 |            479 |      19.57 |
|       4 |            236 |       9.64 |

$~$

#### 8.) How many customers have upgraded to an annual plan in 2020?
```sql
SELECT SUM(CASE WHEN plan_id = 3 THEN 1
                ELSE 0 END) AS customer_count 
FROM subscriptions
WHERE EXTRACT(YEAR FROM start_date) = 2020;
```
**Output:**
| customer_count |
| --- | 
| 195 | 

$~$

#### 9.) How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?
```sql
WITH updated_table AS (
SELECT customer_id, plan_id, start_date,
       CASE WHEN plan_id = 0 THEN start_date END AS started_date,
       CASE WHEN plan_id = 3 THEN start_date END AS upgraded_date
FROM subscriptions
ORDER BY customer_id)

SELECT ROUND(AVG(A.upgraded_date - B.started_date), 2) average_days
FROM updated_table A
LEFT JOIN updated_table B
 ON A.customer_id = B.customer_id
WHERE A.upgraded_date IS NOT NULL
  AND B.started_date IS NOT NULL;
```
**Output:**
| average_days |
| --- | 
| 104.62 | 

#### 10.) Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)
```sql
WITH updated_table AS (
SELECT customer_id, plan_id, start_date,
       CASE WHEN plan_id = 0 THEN start_date END AS started_date,
       CASE WHEN plan_id = 3 THEN start_date END AS upgraded_date
FROM subscriptions
ORDER BY customer_id),

     grouped_table AS (
SELECT WIDTH_BUCKET(A.upgraded_date - B.started_date, 0, 365, 12) AS bucket_group
FROM updated_table A
LEFT JOIN updated_table B
 ON A.customer_id = B.customer_id
WHERE A.upgraded_date IS NOT NULL
  AND B.started_date IS NOT NULL)
  
SELECT ((bucket_group - 1) * 30 || ' - ' || bucket_group * 30) AS date_group,
       COUNT(bucket_group) AS number_of_customers
FROM grouped_table
GROUP BY bucket_group
ORDER BY bucket_group;
```
**Output:**
date_group | number_of_customers
| --- | --- |
| 0 - 30     |                  49 |
| 30 - 60    |                  24 |
| 60 - 90    |                  35 |
| 90 - 120   |                  35 |
| 120 - 150  |                  43 |
| 150 - 180  |                  37 |
| 180 - 210  |                  24 |
| 210 - 240  |                   4 |
| 240 - 270  |                   4 |
| 270 - 300  |                   1 |
| 300 - 330  |                   1 |
| 330 - 360  |                   1 |
 
$~$

#### 11.) How many customers downgraded from a pro monthly to a basic monthly plan in 2020?
```sql
WITH updated_table AS (
SELECT customer_id,
       plan_id,
	     start_date,
       CASE WHEN plan_id = 2 THEN start_date END AS previous_plan,
	     CASE WHEN plan_id = 1 THEN start_date END AS current_plan
FROM subscriptions)

SELECT COUNT(A.customer_id) AS customer_count
FROM updated_table A
LEFT JOIN updated_table B
 ON A.customer_id = B.customer_id
WHERE A.previous_plan IS NOT NULL
  AND B.current_plan IS NOT NULL
  AND A.previous_plan < B.current_plan
  AND EXTRACT(YEAR FROM A.start_date) = 2020;
```
**Output:**
| customer_count | 
| --- |
| 0 |

$~$

## C. Challenge Payment Question
The Foodie-Fi team wants you to create a new payments table for the year 2020 that includes amounts paid by each customer in the subscriptions table with the following requirements:

+ monthly payments always occur on the same day of month as the original start_date of any monthly paid plan
+ upgrades from basic to monthly or pro plans are reduced by the current paid amount in that month and start immediately
+ upgrades from pro monthly to pro annual are paid at the end of the current billing period and also starts at the end of the month period
+ once a customer churns they will no longer make payments
```sql
CREATE TABLE payments (
 customer_id INT,
 plan_id INT,
 plan_name TEXT,
 payment_date DATE,
 amount NUMERIC,
 payment_order INT);
 

INSERT INTO payments
WITH RECURSIVE joined_table AS (
	SELECT A.customer_id,
		   A.plan_id,
		   B.plan_name,
		   A.start_date,
		   LEAD(A.start_date) OVER(PARTITION BY customer_id ORDER BY A.start_date, A.plan_id) AS next_date,
		   B.price AS amount
	FROM subscriptions A
	LEFT JOIN plans B
	 ON A.plan_id = B.plan_id
	WHERE A.plan_id NOT IN (0, 4)
	  AND A.start_date <= '2020-12-31'
),

fixed_table AS (
	SELECT customer_id,
		   plan_id,
		   plan_name,
		   start_date,
		   COALESCE(next_date, '2020-12-31') AS next_date,
		   amount
	FROM joined_table
),

recursive_table AS (
	SELECT customer_id,
		   plan_id,
		   plan_name,
		   start_date AS payment_date,
		   next_date,
		   amount
	FROM fixed_table
	UNION ALL
	SELECT customer_id,
		   plan_id,
		   plan_name,
		   DATE(payment_date + INTERVAL '1 month') AS payment_date,
		   next_date,
		   amount
	FROM recursive_table
	WHERE payment_date + INTERVAL '1 month' < next_date AND plan_id != 3
)	

SELECT customer_id,
       plan_id,
	   plan_name,
	   payment_date,
	   amount,
	   CAST(ROW_NUMBER() OVER(PARTITION BY customer_id ORDER BY payment_date, plan_id) AS INT) AS payment_order
FROM recursive_table
ORDER BY customer_id, plan_id;
```
**Partial Output:**
| customer_id | plan_id |   plan_name   | payment_date | amount | payment_order |
| ---------  | ------- | ------------- | ------------ | ------ | ------------- |
|           1 |       1 | basic monthly | 2020-08-08   |   9.90 |             1 | 
|           1 |       1 | basic monthly | 2020-09-08   |   9.90 |             2 |
|           1 |       1 | basic monthly | 2020-10-08   |   9.90 |             3 |
|           1 |       1 | basic monthly | 2020-11-08   |   9.90 |             4 |
|           1 |       1 | basic monthly | 2020-12-08   |   9.90 |             5 |
|           2 |       3 | pro annual    | 2020-09-27   |    199 |             1 |
|           3 |       1 | basic monthly | 2020-01-20   |   9.90 |             1 |
|           3 |       1 | basic monthly | 2020-02-20   |   9.90 |             2 |
|           3 |       1 | basic monthly | 2020-03-20   |   9.90 |             3 |
|           3 |       1 | basic monthly | 2020-04-20   |   9.90 |             4 |
|           3 |       1 | basic monthly | 2020-05-20   |   9.90 |             5 |
|           3 |       1 | basic monthly | 2020-06-20   |   9.90 |             6 |
|           3 |       1 | basic monthly | 2020-07-20   |   9.90 |             7 |
|           3 |       1 | basic monthly | 2020-08-20   |   9.90 |             8 |
|           3 |       1 | basic monthly | 2020-09-20   |   9.90 |             9 |
|           3 |       1 | basic monthly | 2020-10-20   |   9.90 |            10 |
|           3 |       1 | basic monthly | 2020-11-20   |   9.90 |            11 |
|           3 |       1 | basic monthly | 2020-12-20   |   9.90 |            12 |
|           4 |       1 | basic monthly | 2020-01-24   |   9.90 |             1 |
|           4 |       1 | basic monthly | 2020-02-24   |   9.90 |             2 |
