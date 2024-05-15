# Case Study #1 -  Danny's Diner
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

1.) What is the total amount each customer spent at the restaurant?
```sql
SELECT s.customer_id AS customer
       SUM(menu.price) AS total_spent
FROM sales
INNER JOIN menu USING(product_id)
GROUP BY sales.customer_id;
```
2.) How many days has each customer visited the restaurant?
```sql
SELECT customer_id AS customer
       COUNT(DISTINCT order_date) AS total_visit
FROM sales
GROUP BY customer_id;
```

3.) What was the first item from the menu purchased by each customer?
```sql
SELECT sales.customer_id AS customer
       MIN(menu.product_name) AS product
       MIN(sales.order_date) AS first_order_date
FROM sales
INNER JOIN menu USING(product_id)
GROUP BY customer_id;
```

4.) What is the most purchased item on the menu and how many times was it purchased by all customers?
```sql
SELECT sales.customer_id AS customer
       menu.product_name AS item
       COUNT(menu.product_name) AS no_of_order
FROM sales
INNER JOIN menu USING(product_id)
WHERE sales.product_id IN (SELECT product
                           FROM (
                                 SELECT product_id AS product
                                        COUNT(product_id) AS count
                                 FROM sales
                                 GROUP BY product_id
                                 ORDER BY count DESC
                                 LIMIT 1
                                ) AS top_product  
                           )
GROUP BY sales.customer_id, menu.product_name;
```

5.) Which item was the most popular for each customer?
```sql
SELECT customer,
       item
FROM (
      SELECT sales.customer_id AS customer
             menu.product_name AS item
             DENSE_RANK() OVER(PARTITION BY sales.customer_id ORDER BY COUNT(sales.product_id) DESC) AS rank
      FROM sales
      INNER JOIN menu USING(product_id)
      GROUP BY sales.customer_id, menu.product_name
      )
WHERE rank = 1;
```

6.) Which item was purchased first by the customer after they became a member?
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

7.) Which item was purchased just before the customer became a member?
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

8.) What is the total items and amount spent for each member before they became a member?
```sql
SELECT sales.customer_id AS customer,
       SUM(menu.price) AS total_spent
FROM sales
INNER JOIN menu USING(product_id)
LEFT JOIN members USING(customer_id)
WHERE members.join_date > sales.order_date
GROUP BY sales.customer_id;
```

9.) If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
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

10.) In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
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

