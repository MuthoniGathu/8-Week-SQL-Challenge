# Case Study #1 - Danny's Diner

1. What is the total amount each customer spent at the restaurant?

```
SELECT
    customer_id,
    SUM(price) as Total
FROM dannys_diner.sales as T1
INNER JOIN dannys_diner.menu as T2 
ON  T1.product_id = T2.product_id
GROUP BY customer_id
ORDER BY Total DESC
```
**OUTPUT:**

| customer_id | total |
| ----------- | ----- |
| A           | 76    |
| B           | 74    |
| C           | 36    |

---
2. How many days has each customer visited the restaurant?

```
SELECT
    customer_id,
    COUNT(DISTINCT order_date) as Days_Visited    
FROM dannys_diner.sales 
GROUP BY customer_id
ORDER BY Customer_id
```
**OUTPUT:**

| customer_id | days_visited |
| ----------- | ------------ |
| A           | 4            |
| B           | 6            |
| C           | 2            |

---
3. What was the first item from the menu purchased by each customer?

```
SELECT customer_id, product_name
FROM
        (SELECT
           customer_id,
           a.product_id as product_id,
           b.product_name as product_name, 
           a.order_date,
           RANK() OVER(PARTITION BY customer_id ORDER BY a.order_date) as numbered_order
         FROM dannys_diner.sales as a
         INNER JOIN dannys_diner.menu as b
         ON a.product_id = b.product_id 
         GROUP BY customer_id, a.product_id,b.product_name, a.order_date
         ORDER BY Customer_id) as T1
 WHERE numbered_order = 1
```
**OUTPUT:**

| customer_id | product_name |
| ----------- | ------------ |
| A           | sushi        |
| A           | curry        |
| B           | curry        |
| C           | ramen        |

---
4. What is the most purchased item on the menu and how many times was it purchased by all customers?

```
SELECT DISTINCT product_name,
       SUM(product_count) as Total
FROM
  (SELECT
        customer_id,
        COUNT(a.product_id) as product_count,
        a.product_id,
        b.product_name as product_name,
        ROW_NUMBER() OVER(PARTITION BY customer_id ORDER BY COUNT(a.product_id) DESC ) as ranked_order
  FROM dannys_diner.sales as a
  INNER JOIN dannys_diner.menu as b
  ON a.product_id = b.product_id 
  GROUP BY customer_id, a.product_id,b.product_name
  ORDER BY Customer_id) as T1 
WHERE ranked_order = 1
GROUP BY product_name
```
**OUTPUT:**

| product_name | total |
| ------------ | --- |
| ramen        | 8   |

---
5. Which item was the most popular for each customer?

```
 SELECT customer_id, product_name
FROM
  (SELECT
        customer_id,
        COUNT(a.product_id),
        a.product_id,
        b.product_name as product_name,
        RANK() OVER(PARTITION BY customer_id ORDER BY COUNT(a.product_id) DESC ) as ranked_order
  FROM dannys_diner.sales as a
  INNER JOIN dannys_diner.menu as b
  ON a.product_id = b.product_id 
  GROUP BY customer_id, a.product_id,b.product_name
  ORDER BY Customer_id) as T1
WHERE ranked_order = 1
```
**OUTPUT:**

| customer_id | product_name |
| ----------- | ------------ |
| A           | ramen        |
| B           | ramen        |
| B           | sushi        |
| B           | curry        |
| C           | ramen        |

---
6. Which item was purchased first by the customer after they became a member?

```
SELECT customer_id, product_name
FROM
  (SELECT
          a.customer_id as customer_id,
          b.join_date as join_date,
          a.order_date as order_date,
          (a.order_date::DATE- b.join_date::DATE) as difference,
          a.product_id,
          c.product_name,
          RANK() OVER(PARTITION BY a.customer_id ORDER BY (a.order_date::DATE- b.join_date::DATE)) as ranked
  FROM dannys_diner.sales as a
  INNER JOIN dannys_diner.members as b
  ON a.customer_id = b.customer_id
  INNER JOIN dannys_diner.menu c
  ON a.product_id = c.product_id
  GROUP BY a.customer_id, a.product_id, b.join_date, c.product_name,a.order_date
  HAVING (a.order_date::DATE- b.join_date::DATE) >= 0
  ORDER BY a.customer_id) as T1
 WHERE ranked = 1
```
**OUTPUT:**

| customer_id | product_name |
| ----------- | ------------ |
| A           | curry        |
| B           | sushi        |

---
7. Which item was purchased just before the customer became a member?

```
SELECT customer_id, product_name
FROM
  (SELECT
          a.customer_id as customer_id,
          b.join_date as join_date,
          a.order_date as order_date,
          (a.order_date::DATE- b.join_date::DATE) as difference,
          a.product_id,
          c.product_name,
          RANK() OVER(PARTITION BY a.customer_id ORDER BY (a.order_date::DATE- b.join_date::DATE) DESC) as ranked
  FROM dannys_diner.sales as a
  INNER JOIN dannys_diner.members as b
  ON a.customer_id = b.customer_id
  INNER JOIN dannys_diner.menu c
  ON a.product_id = c.product_id
  GROUP BY a.customer_id, a.product_id, b.join_date, c.product_name,a.order_date
  HAVING (a.order_date::DATE- b.join_date::DATE) < 0
  ORDER BY a.customer_id) as T1
 WHERE ranked = 1
```
**OUTPUT:**

| customer_id | product_name |
| ----------- | ------------ |
| A           | sushi        |
| A           | curry        |
| B           | sushi        |

---
8. What is the total items and amount spent for each member before they became a member?

```
SELECT DISTINCT customer_id, Total_Items, Total_Price
FROM
  (SELECT
          a.customer_id as customer_id,
          a.product_id,
          c.price,
          (a.order_date::DATE- b.join_date::DATE) as difference,         
           SUM(c.price) OVER(PARTITION BY a.customer_id) as Total_Price,
           COUNT(a.product_id) OVER(PARTITION BY a.customer_id) as Total_Items
  FROM dannys_diner.sales as a
  INNER JOIN dannys_diner.members as b
  ON a.customer_id = b.customer_id
  INNER JOIN dannys_diner.menu c
  ON a.product_id = c.product_id
  GROUP BY a.customer_id,b.join_date,a.product_id,c.price, a.order_date,c.price
  HAVING (a.order_date::DATE- b.join_date::DATE) < 0
  ORDER BY a.customer_id) as T1 
  ORDER BY customer_id
```
**OUTPUT:**

| customer_id | total_items | total_price |
| ----------- | ----------- | ----------- |
| A           | 2           | 25          |
| B           | 3           | 40          |

---

9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

```
 SELECT DISTINCT customer_id,
     SUM(Points*Total_Price) OVER(PARTITION BY customer_id) as Total_Points
FROM
    (SELECT
        customer_id,
        COUNT( a.product_id),
        a.product_id,
        b.product_name,
        price,
        SUM( price),
        CASE a.product_id
            WHEN 1 THEN 20
         ELSE 10
         END as Points, 
         price * COUNT( a.product_id) as Total_Price
    FROM dannys_diner.sales as a
    INNER JOIN dannys_diner.menu as b 
    ON  a.product_id = b.product_id
    GROUP BY customer_id,a.product_id,b.product_name,price
    ORDER BY customer_id) as T1
ORDER BY customer_id
```
**OUTPUT:**

| customer_id | total_points |
| ----------- | ------------ |
| A           | 860          |
| B           | 940          |
| C           | 360          |

---

10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

``` 
SELECT DISTINCT customer_id,
     SUM(Points*Total_Price) OVER(PARTITION BY customer_id) as Total_Points
FROM
    (SELECT
            a.customer_id,
            a.order_date as order_date,
            COUNT( a.product_id),
            a.product_id,
            b.product_name,
            price,
            SUM( price),
         CASE
              WHEN a.product_id = 1 THEN 20
              WHEN a.customer_id IN (SELECT customer_id FROM dannys_diner.members) AND(a.order_date >= c.join_date AND a.order_date < (c.join_date+7)) THEN 20
      	      ELSE 10
        END AS points, 
             price * COUNT( a.product_id)  as Total_Price
        FROM dannys_diner.sales as a
        INNER JOIN dannys_diner.menu as b 
        ON  a.product_id = b.product_id
        INNER JOIN dannys_diner.members as c
        ON a.customer_id = c.customer_id
        WHERE a.order_date <'2021-01-31'
        GROUP BY a.customer_id,a.order_date,c.join_date,a.product_id,b.product_name,price
        ORDER BY a.customer_id) as T1
```
**OUTPUT:**

| customer_id | total_points |
| ----------- | ------------ |
| A           | 1370         |
| B           | 820          |

---



