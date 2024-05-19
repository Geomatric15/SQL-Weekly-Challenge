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
SELECT COUNT(DISTINCT customer_id) 
FROM subscriptions;
```
$~$

#### 2.) What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value
```sql
SELECT EXTRACT(MONTH FROM start_date) AS month,
       COUNT(*) AS count
FROM subscriptions
GROUP BY month
ORDER BY month;
```
$~$

#### 3.) What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name
```sql
SELECT plan_id,
       COUNT(plan_id) AS distribution,
       EXTRACT(YEAR FROM start_date) AS year,
	     EXTRACT(MONTH FROM start_date) AS month,
	     EXTRACT(DAY FROM start_date) AS day
FROM subscriptions
WHERE EXTRACT(YEAR FROM start_date) > 2020
GROUP BY plan_id, year, month, day
ORDER BY month, day 
```
$~$

#### 4.) What is the customer count and percentage of customers who have churned rounded to 1 decimal place?
```sql
SELECT COUNT(customer_id) AS customer_count,
       ROUND(COUNT(customer_id) / (SELECT CAST(COUNT(DISTINCT customer_id) AS DECIMAL) 
		                               FROM subscriptions), 1) AS percentage
FROM subscriptions
WHERE plan_id = 4;
```

$~$

#### 5.) How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?
```sql
SELECT COUNT(customer_id) AS customer_count,
       ROUND(COUNT(customer_id) / (SELECT CAST(COUNT(DISTINCT customer_id) AS DECIMAL) FROM subscriptions)) AS percentage
FROM(
	SELECT customer_id,
		   plan_id,
		   DENSE_RANK() OVER(PARTITION BY customer_id ORDER BY plan_id) AS rank
	FROM subscriptions)
WHERE plan_id = 4 and rank = 2;
```

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
$~$

#### 8.) How many customers have upgraded to an annual plan in 2020?
```sql
SELECT SUM(CASE WHEN plan_id = 3 THEN 1
                ELSE 0 END) AS customer_count 
FROM subscriptions
WHERE EXTRACT(YEAR FROM start_date) = 2020;
```

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
