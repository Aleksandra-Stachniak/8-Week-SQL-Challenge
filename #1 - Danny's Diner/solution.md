## Case Study Questions

1. What is the total amount each customer spent at the restaurant?

2. How many days has each customer visited the restaurant?
3. What was the first item from the menu purchased by each customer?
4. What is the most purchased item on the menu and how many times was it purchased by all customers?
5. Which item was the most popular for each customer?
6. Which item was purchased first by the customer after they became a member?
7. Which item was purchased just before the customer became a member?
8. What is the total items and amount spent for each member before they became a member?
9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

<br>

**BONUS questions** :

i. Join all the things

ii. Rank all the things

---


## Solutions:


### 1. What is the total amount each customer spent at the restaurant?

```sql
SELECT 
    s.customer_id,
    SUM(m.price) AS total_sales
FROM dannys_diner.sales s
JOIN dannys_diner.menu m ON s.product_id = m.product_id
GROUP BY s.customer_id
ORDER BY s.customer_id;
```

#### Result set:

| customer_id | total_sales |
| ----------- | ----------- |
| A           | 76          |
| B           | 74          |
| C           | 36          |

---

### 2. How many days has each customer visited the restaurant?

```sql
SELECT 
    customer_id,
    COUNT(DISTINCT order_date) AS visit_count
FROM dannys_diner.sales
GROUP BY customer_id
ORDER BY customer_id;
```

#### Result set:

| customer_id | visit_count |
| ----------- | ----------- |
| A           | 4           |
| B           | 6           |
| C           | 2           |

---

### 3. What was the first item from the menu purchased by each customer?

```sql
WITH ranked_orders AS (
  SELECT 
      s.customer_id, 
      m.product_name, 
      DENSE_RANK() OVER(PARTITION BY s.customer_id ORDER BY s.order_date) AS rank
  FROM dannys_diner.sales s 
  JOIN dannys_diner.menu m ON s.product_id = m.product_id
)
SELECT customer_id, product_name
FROM ranked_orders
WHERE rank = 1
GROUP BY customer_id, product_name;
```

#### Result set:

| customer_id | product_name |
| ----------- | ------------ |
| A           | curry        |
| A           | sushi        |
| B           | curry        |
| C           | ramen        |

---

### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

```sql
SELECT 
    m.product_name AS most_purchased_item,
    COUNT(s.product_id) AS order_count
FROM dannys_diner.sales s
JOIN dannys_diner.menu m ON s.product_id = m.product_id
GROUP BY m.product_name
ORDER BY order_count DESC
LIMIT 1;
```

#### Result set:

| most_purchased_item | order_count |
| ------------------- | ----------- |
| ramen               | 8           |

---

### 5. Which item was the most popular for each customer?

```sql
WITH ranked_orders AS (
  SELECT 
      s.customer_id,
      m.product_name,
      COUNT(m.product_name) AS order_count,
      RANK() OVER (
          PARTITION BY s.customer_id 
          ORDER BY COUNT(m.product_name) DESC
      ) AS rank
  FROM dannys_diner.sales s
  JOIN dannys_diner.menu m ON s.product_id = m.product_id
  GROUP BY s.customer_id, m.product_name
)
SELECT 
    customer_id,
    product_name,
    order_count
FROM ranked_orders
WHERE rank = 1
ORDER BY customer_id;
```

#### Result set:

| customer_id | product_name | order_count |
| ----------- | ------------ | ----------- |
| A           | ramen        | 3           |
| B           | sushi        | 2           |
| B           | curry        | 2           |
| B           | ramen        | 2           |
| C           | ramen        | 3           |

---

### 6. Which item was purchased first by the customer after they became a member?

```sql
WITH diner_info AS (
  SELECT 
      s.customer_id,
      m.product_name,
      s.order_date,
      mem.join_date,
      DENSE_RANK() OVER (
          PARTITION BY s.customer_id 
          ORDER BY s.order_date
      ) AS rank
  FROM dannys_diner.sales s
  INNER JOIN dannys_diner.menu m ON s.product_id = m.product_id
  INNER JOIN dannys_diner.members mem ON s.customer_id = mem.customer_id
  WHERE s.order_date >= mem.join_date
)
SELECT 
    customer_id,
    product_name,
    order_date,
    join_date
FROM diner_info
WHERE rank = 1
ORDER BY customer_id;
```

#### Result set:

| customer_id | product_name | order_date               | join_date                |
| ----------- | ------------ | ------------------------ | ------------------------ |
| A           | curry        | 2021-01-07               | 2021-01-07               |
| B           | sushi        | 2021-01-11               | 2021-01-09               |

---

### 7. Which item was purchased just before the customer became a member?

```sql
WITH diner_info AS (
  SELECT 
      s.customer_id,
      m.product_name,
      s.order_date,
      mem.join_date,
      DENSE_RANK() OVER (
          PARTITION BY s.customer_id 
          ORDER BY s.order_date DESC
      ) AS rank
  FROM dannys_diner.sales s
  INNER JOIN dannys_diner.menu m ON s.product_id = m.product_id
  INNER JOIN dannys_diner.members mem ON s.customer_id = mem.customer_id
  WHERE s.order_date < mem.join_date
)
SELECT 
    customer_id,
    product_name,
    order_date,
    join_date
FROM diner_info
WHERE rank = 1
ORDER BY customer_id;
```

#### Result set:

| customer_id | product_name | order_date               | join_date                |
| ----------- | ------------ | ------------------------ | ------------------------ |
| A           | sushi        | 2021-01-01               | 2021-01-07               |
| A           | curry        | 2021-01-01               | 2021-01-07               |
| B           | sushi        | 2021-01-04               | 2021-01-09               |

---

### 8. What is the total items and amount spent for each member before they became a member?

```sql
SELECT 
    s.customer_id,
    COUNT(s.product_id) AS total_items,
    SUM(m.price) AS amount_spent
FROM dannys_diner.sales s
INNER JOIN dannys_diner.menu m ON s.product_id = m.product_id
INNER JOIN dannys_diner.members mem ON s.customer_id = mem.customer_id
WHERE s.order_date < mem.join_date
GROUP BY s.customer_id
ORDER BY s.customer_id;
```

#### Result set:

| customer_id | total_items | amount_spent |
| ----------- | ----------- | ------------ |
| A           | 2           | 25           |
| B           | 3           | 40           |

---

### 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

```sql
SELECT 
    s.customer_id,
    SUM(
        CASE 
            WHEN m.product_name = 'sushi' THEN m.price * 20
            ELSE m.price * 10
        END
    ) AS customer_points
FROM dannys_diner.sales s
INNER JOIN dannys_diner.menu m ON s.product_id = m.product_id
GROUP BY s.customer_id
ORDER BY s.customer_id;
```

#### Result set:

| customer_id | customer_points |
| ----------- | --------------- |
| A           | 860             |
| B           | 940             |
| C           | 360             |

---

### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January

```sql
WITH cte AS (
  SELECT 
      customer_id,
      join_date,
      join_date + INTERVAL '6 days' AS program_last_date
  FROM dannys_diner.members
)
SELECT 
    s.customer_id,
    SUM(
        CASE 
            WHEN s.order_date BETWEEN cte.join_date AND cte.program_last_date 
                THEN m.price * 20
            WHEN s.order_date > cte.program_last_date AND m.product_name = 'sushi' 
                THEN m.price * 20
            ELSE m.price * 10
        END
    ) AS customer_points
FROM dannys_diner.sales s
JOIN dannys_diner.menu m ON s.product_id = m.product_id
JOIN cte ON s.customer_id = cte.customer_id
WHERE s.order_date BETWEEN cte.join_date AND '2021-01-31'
GROUP BY s.customer_id
ORDER BY s.customer_id;
```

#### Result set:

| customer_id | customer_points |
| ----------- | --------------- |
| A           | 1020            |
| B           | 320             |

---

### **Bonus Questions**

#### i. Join All The Things

Create basic data tables that Danny and his team can use to quickly derive insights without needing to join the underlying tables using SQL. Fill Member column as 'N' if the purchase was made before becoming a member and 'Y' if the after is amde after joining the membership.

```sql
SELECT   
    s.customer_id,
    s.order_date,
    m.product_name,
    m.price,
    CASE 
        WHEN mem.join_date IS NOT NULL AND s.order_date >= mem.join_date 
            THEN 'Y' 
        ELSE 'N' 
    END AS member
FROM dannys_diner.sales s
LEFT JOIN dannys_diner.members mem ON s.customer_id = mem.customer_id
INNER JOIN dannys_diner.menu m ON s.product_id = m.product_id
ORDER BY s.customer_id, s.order_date;
```

#### Result set:

| customer_id | order_date | product_name | price | member |
| ----------- | ---------- | ------------ | ----- | ------ |
| A           | 2021-01-01 | sushi        | 10    | N      |
| A           | 2021-01-01 | curry        | 15    | N      |
| A           | 2021-01-07 | curry        | 15    | Y      |
| A           | 2021-01-10 | ramen        | 12    | Y      |
| A           | 2021-01-11 | ramen        | 12    | Y      |
| A           | 2021-01-11 | ramen        | 12    | Y      |
| B           | 2021-01-01 | curry        | 15    | N      |
| B           | 2021-01-02 | curry        | 15    | N      |
| B           | 2021-01-04 | sushi        | 10    | N      |
| B           | 2021-01-11 | sushi        | 10    | Y      |
| B           | 2021-01-16 | ramen        | 12    | Y      |
| B           | 2021-02-01 | ramen        | 12    | Y      |
| C           | 2021-01-01 | ramen        | 12    | N      |
| C           | 2021-01-01 | ramen        | 12    | N      |
| C           | 2021-01-07 | ramen        | 12    | N      |

---

#### ii. Rank All The Things

Danny also requires further information about the ranking of customer products, but he purposely does not need the ranking for non-member purchases so he expects null ranking values for the records when customers are not yet part of the loyalty program.

```sql
WITH cte AS (
 	SELECT    
        s.customer_id,
        s.order_date,
        m.product_name,
        m.price,
        CASE 
            WHEN mem.join_date IS NOT NULL AND s.order_date >= mem.join_date 
                THEN 'Y' 
            ELSE 'N' 
        END AS member
    FROM dannys_diner.sales s
    LEFT JOIN dannys_diner.members mem ON s.customer_id = mem.customer_id
    INNER JOIN dannys_diner.menu m ON s.product_id = m.product_id
)
SELECT *,
       CASE 
           WHEN member = 'N' THEN NULL 
           ELSE DENSE_RANK() OVER (PARTITION BY customer_id, member ORDER BY order_date) 
       END AS ranking
FROM cte
ORDER BY customer_id, order_date;
```

#### Result set:

| customer_id | order_date | product_name | price | member | ranking |
| ----------- | ---------- | ------------ | ----- | ------ | ------- |
| A           | 2021-01-01 | sushi        | 10    | N      | NULL    |
| A           | 2021-01-01 | curry        | 15    | N      | NULL    |
| A           | 2021-01-07 | curry        | 15    | Y      | 1       |
| A           | 2021-01-10 | ramen        | 12    | Y      | 2       |
| A           | 2021-01-11 | ramen        | 12    | Y      | 3       |
| A           | 2021-01-11 | ramen        | 12    | Y      | 3       |
| B           | 2021-01-01 | curry        | 15    | N      | NULL    |
| B           | 2021-01-02 | curry        | 15    | N      | NULL    |
| B           | 2021-01-04 | sushi        | 10    | N      | NULL    |
| B           | 2021-01-11 | sushi        | 10    | Y      | 1       |
| B           | 2021-01-16 | ramen        | 12    | Y      | 2       |
| B           | 2021-02-01 | ramen        | 12    | Y      | 3       |
| C           | 2021-01-01 | ramen        | 12    | N      | NULL    |
| C           | 2021-01-01 | ramen        | 12    | N      | NULL    |
| C           | 2021-01-07 | ramen        | 12    | N      | NULL    |

---
