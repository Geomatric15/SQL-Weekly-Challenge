# Case Study #1 -  Danny's Diner
![1](https://github.com/Geomatric15/SQL-Weekly-Challenge/assets/167914482/8cde3b2d-1329-4cb3-b752-008979151f6f)

## Introduction
Danny seriously loves Japanese food so in the beginning of 2021, he decides to embark upon a risky venture and opens up a cute little restaurant that sells his 3 favourite foods: sushi, curry and ramen.

Danny’s Diner is in need of your assistance to help the restaurant stay afloat - the restaurant has captured some very basic data from their few months of operation but have no idea how to use their data to help them run the business.

## Problem Statement
Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers.

He plans on using these insights to help him decide whether he should expand the existing customer loyalty program - additionally he needs help to generate some basic datasets so his team can easily inspect the data without needing to use SQL.

Danny has provided you with a sample of his overall customer data due to privacy issues - but he hopes that these examples are enough for you to write fully functioning SQL queries to help him answer his questions!

Danny has shared with you 3 key datasets for this case study:
+ **sales**
+ **menu**
+ **members**

You can inspect the entity relationship diagram and example data below.

## Entity Relationshop Diagram
![Danny's Diner](https://github.com/Geomatric15/SQL-Weekly-Challenge/assets/167914482/0ac9dc03-b062-4cf5-a602-e1444e36650e)

## Case Study Questions
> [!NOTE]
> These questions were answered using postgreSQL

#### 1.) What is the total amount each customer spent at the restaurant?
```sql
SELECT sales.customer_id AS customer,
       SUM(menu.price) AS total_spent
FROM sales
INNER JOIN menu USING(product_id)
GROUP BY sales.customer_id;
```
**Output:**
| customer | total_spent |
| -------- | ----------- |
| B        |          74 |
| C        |          36 |
| A        |          76 |

#### 2.) How many days has each customer visited the restaurant?
```sql
SELECT customer_id AS customer,
       COUNT(DISTINCT order_date) AS total_visit
FROM sales
GROUP BY customer_id;
```
**Output:**
| customer | total_visit |
| -------- | ----------- |
| A        |           4 |
| B        |           6 |
| C        |           2 |

#### 3.) What was the first item from the menu purchased by each customer?
```sql
SELECT sales.customer_id AS customer,
       MIN(menu.product_name) AS product,
       MIN(sales.order_date) AS first_order_date
FROM sales
INNER JOIN menu USING(product_id)
GROUP BY customer_id;
```
**Output:**
| customer | product | first_order_date | 
| -------- | ------- | ---------------- |
| B        | curry   | 2021-01-01       |      
| C        | ramen   | 2021-01-01       |     
| A        | curry   | 2021-01-01       |      

#### 4.) What is the most purchased item on the menu and how many times was it purchased by all customers?
```sql
SELECT sales.customer_id AS customer,
       menu.product_name AS item,
       COUNT(menu.product_name) AS no_of_order
FROM sales
INNER JOIN menu USING(product_id)
WHERE sales.product_id IN (SELECT product
                           FROM (
                                 SELECT product_id AS product,
                                        COUNT(product_id) AS count
                                 FROM sales
                                 GROUP BY product_id
                                 ORDER BY count DESC
                                 LIMIT 1
                                ) AS top_product  
                           )
GROUP BY sales.customer_id, menu.product_name;
```
**Output:**
| customer | item | no_of_order |
| --- | --- | --- |
| B | ramen | 2 |
| A | ramen | 3 |
| C | ramen | 3 |

#### 5.) Which item was the most popular for each customer?
```sql
SELECT customer,
       item
FROM (
      SELECT sales.customer_id AS customer,
             menu.product_name AS item,
             DENSE_RANK() OVER(PARTITION BY sales.customer_id ORDER BY COUNT(sales.product_id) DESC) AS rank
      FROM sales
      INNER JOIN menu USING(product_id)
      GROUP BY sales.customer_id, menu.product_name
      )
WHERE rank = 1;
```
**Output:**
| customer | item |
| --- | --- |
| A | ramen |
| B | sushi |
| B | curry |
| B | ramen |
| C | ramen |

#### 6.) Which item was purchased first by the customer after they became a member?
```sql
SELECT sales.customer_id AS customer,
       MIN(menu.product_name) AS item,
       MIN(order_date) AS order_date,
	   members.join_date AS join_date
FROM sales
INNER JOIN menu USING(product_id)
LEFT JOIN members USING(customer_id)
WHERE members.join_date <= sales.order_date
GROUP BY sales.customer_id, members.join_date;
```
**Output:**
| customer | item | order_date | join_date |
| --- | --- | --- | --- |
| A | curry | 2021-01-07 | 2021-01-07 |
| B | ramen | 2021-01-11 | 2021-01-09 |

#### 7.) Which item was purchased just before the customer became a member?
```sql
SELECT sales.customer_id AS customer,
       MAX(menu.product_name) AS item,
       MAX(order_date) AS order_date,
	   members.join_date AS join_date
FROM sales
INNER JOIN menu USING(product_id)
LEFT JOIN members USING(customer_id)
WHERE members.join_date > sales.order_date
GROUP BY sales.customer_id, members.join_date;
```
**Output:**
| customer | item | order_date | join_date |
| --- | --- | --- | --- |
| A | sushi | 2021-01-01 | 2021-01-07 |
| B | sushi | 2021-01-04 | 2021-01-09 |

#### 8.) What is the total items and amount spent for each member before they became a member?
```sql
SELECT sales.customer_id AS customer,
       SUM(menu.price) AS total_spent
FROM sales
INNER JOIN menu USING(product_id)
LEFT JOIN members USING(customer_id)
WHERE members.join_date > sales.order_date
GROUP BY sales.customer_id;
```
**Output:**
| customer | total_spent |
| --- | --- |
| B | 40 |
| A | 25 |

#### 9.) If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
```sql
SELECT customer,
       SUM(price * 10 * multiplier) AS total_points
FROM (
      SELECT sales.customer_id AS customer,
              menu.price AS price,
              CASE WHEN menu.product_name = 'sushi' THEN 2 ELSE 1 END AS multiplier
       FROM sales
       INNER JOIN menu USING(product_id) 
      )
GROUP BY customer;
```
**Output:**
| customer | total_points |
| --- | --- |
| B | 940 |
| C | 360 |
| A | 860 |

#### 10.) In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
```sql
SELECT members.customer_id AS customer,
       SUM(CASE WHEN menu.product_name = 'sushi' 
		    OR sales.order_date BETWEEN members.join_date 
                                          AND members.join_date + integer'6' 
		  THEN menu.price * 10 * 2 
		  ELSE menu.price * 10 END) AS points
FROM sales
INNER JOIN menu USING(product_id)
INNER JOIN members USING(customer_id)
WHERE DATE_PART('month', order_date) = 1
GROUP BY members.customer_id;
```
**Output:**
| customer | points |
| --- | --- |
| B | 820 |
| A | 1370 | 

## Bonus Questions

#### A.) Join All The Things
```sql
SELECT sales.customer_id AS customer,
       sales.order_date AS order_date,
       menu.product_name AS prodoct_name,
       menu.price AS price,
       CASE WHEN members.join_date <= sales.order_date THEN 'Y' ELSE 'N' END AS member
FROM sales
INNER JOIN menu USING(product_id)
LEFT JOIN members USING(customer_id)
ORDER BY customer, order_date;
```
**Output:**
 customer | order_date | prodoct_name | price | member
| --- | --- | --- | --- | --- |
 A        | 2021-01-01 | sushi        |    10 | N
 A        | 2021-01-01 | curry        |    15 | N
 A        | 2021-01-07 | curry        |    15 | Y
 A        | 2021-01-10 | ramen        |    12 | Y
 A        | 2021-01-11 | ramen        |    12 | Y
 A        | 2021-01-11 | ramen        |    12 | Y
 B        | 2021-01-01 | curry        |    15 | N
 B        | 2021-01-02 | curry        |    15 | N
 B        | 2021-01-04 | sushi        |    10 | N
 B        | 2021-01-11 | sushi        |    10 | Y
 B        | 2021-01-16 | ramen        |    12 | Y
 B        | 2021-02-01 | ramen        |    12 | Y
 C        | 2021-01-01 | ramen        |    12 | N
 C        | 2021-01-01 | ramen        |    12 | N
 C        | 2021-01-07 | ramen        |    12 | N

#### B.) Rank All The Things
```sql
WITH temporary_table AS (
	SELECT sales.customer_id AS customer,
	       sales.order_date AS order_date,
               menu.product_name AS prodoct_name,
               menu.price AS price,
               CASE WHEN members.join_date <= sales.order_date THEN 'Y' ELSE 'N' END AS member
	FROM sales
	INNER JOIN menu USING(product_id)
	LEFT JOIN members USING(customer_id)
	ORDER BY sales.customer_id, sales.order_date
)

SELECT *,
      CASE WHEN member = 'N' THEN NULL
	   ELSE RANK() OVER(PARTITION BY customer, member ORDER BY order_date) END AS rank
FROM temporary_table;
```
**Output:**
customer | order_date | prodoct_name | price | member | rank
| --- | --- | --- | --- | --- | --- |
 A        | 2021-01-01 | sushi        |    10 | N      |
 A        | 2021-01-01 | curry        |    15 | N      |
 A        | 2021-01-07 | curry        |    15 | Y      |    1
 A        | 2021-01-10 | ramen        |    12 | Y      |    2
 A        | 2021-01-11 | ramen        |    12 | Y      |    3
 A        | 2021-01-11 | ramen        |    12 | Y      |    3
 B        | 2021-01-01 | curry        |    15 | N      |
 B        | 2021-01-02 | curry        |    15 | N      |
 B        | 2021-01-04 | sushi        |    10 | N      |
 B        | 2021-01-11 | sushi        |    10 | Y      |    1
 B        | 2021-01-16 | ramen        |    12 | Y      |    2
 B        | 2021-02-01 | ramen        |    12 | Y      |    3
 C        | 2021-01-01 | ramen        |    12 | N      |
 C        | 2021-01-01 | ramen        |    12 | N      |
 C        | 2021-01-07 | ramen        |    12 | N      |

