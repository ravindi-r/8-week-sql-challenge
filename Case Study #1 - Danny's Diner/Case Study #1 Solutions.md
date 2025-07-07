## Case Study #1 - Danny's Diner Solutions

[Database Schema](https://github.com/ravindi-r/8-week-sql-challenge/tree/d01fd4dd428dba27a737d0fc98ba875b900360e6/Case%20Study%20%231%20-%20Danny's%20Diner#schema)
<hr>

1. What is the total amount each customer spent at the restaurant?

```
SELECT
    customer_id,
    SUM(price)
FROM
    sales s
    JOIN menu m ON (s.product_id = m.product_id)
GROUP BY customer_id
ORDER BY customer_id;
```

| customer_id | sum |
| --- | --- |
| A | 76 |
| B | 74 |
| C | 36 |

<hr>

2. How many days has each customer visited the restaurant?

```
SELECT
    customer_id,
    COUNT (DISTINCT order_date)
FROM sales
GROUP BY customer_id;
```

| customer_id |	count |
| --- | --- |
| A | 4 |
| B | 6 |
| C | 2 |
<hr>

3. What was the first item from the menu purchased by each customer?

```
with min_order_date AS (
    SELECT
        customer_id,
        MIN(order_date) AS min_date
    FROM sales
    GROUP BY customer_id
)
SELECT
    DISTINCT s.customer_id, product_name
FROM
    sales s
    JOIN min_order_date md ON (s.customer_id = md.customer_id AND s.order_date = md.min_date)
    JOIN menu m ON (s.product_id = m.product_id)
ORDER BY s.customer_id;
```

| customer_id |	product_name |
| --- | --- |
| A	| curry |
| A	| sushi |
| B	| curry |
| C	| ramen |

<hr>

4. What is the most purchased item on the menu and how many times was it purchased by all customers?

```
WITH ranked_orders AS (
    SELECT
        product_id,
        COUNT(order_date) AS order_count,
        RANK() OVER (ORDER BY COUNT(order_date) DESC) AS rnk
    FROM sales
    GROUP BY product_id
)
SELECT product_name, order_count
FROM
    menu m
    JOIN ranked_orders r ON (m.product_id = r.product_id)
WHERE
    rnk = 1;
```

| product_name | order_count |
| --- | --- |
| ramen	| 8 |

<hr>

5. Which item was the most popular for each customer?

```
WITH ranked_orders AS (
    SELECT
        customer_id, 
        product_id,
        RANK() OVER (PARTITION BY customer_id ORDER BY COUNT(order_date) DESC) AS rnk
    FROM sales
    GROUP BY customer_id, product_id
)
SELECT
    customer_id, product_name
FROM
    ranked_orders r
    JOIN menu m ON (r.product_id = m.product_id)
WHERE rnk = 1
ORDER BY customer_id;
```

| customer_id |	product_name |
| --- | --- |
| A | ramen |
| B | sushi |
| B | curry |
| B | ramen |
| C | ramen |

<hr>

6. Which item was purchased first by the customer after they became a member?

```
WITH member_orders AS (
    SELECT
        s.customer_id, product_id, 
        RANK() OVER (PARTITION BY s.customer_id ORDER BY order_date) AS rnk
    FROM
        sales s
        JOIN members m ON (s.customer_id = m.customer_id)
    WHERE order_date > join_date
)
SELECT
    customer_id, product_name
FROM
    member_orders mo 
    JOIN menu m ON( mo.product_id = m.product_id)
WHERE rnk = 1
ORDER BY customer_id;
```

| customer_id |	product_name |
| --- | --- |
| A | ramen |
| B | sushi |

<hr>

7. Which item was purchased just before the customer became a member?

```
WITH member_orders AS (
    SELECT
        s.customer_id, product_id, 
        RANK() OVER (PARTITION BY s.customer_id ORDER BY order_date DESC) AS rnk
    FROM
        sales s
        JOIN members m ON (s.customer_id = m.customer_id)
    WHERE order_date < join_date
)
SELECT
    customer_id, product_name
FROM
    member_orders mo 
    JOIN menu m ON( mo.product_id = m.product_id)
WHERE rnk = 1
ORDER BY customer_id;
```

| customer_id |	product_name |
| --- | --- |
| A | sushi |
| A | curry |
| B | sushi |

<hr>

8. What is the total items and amount spent for each member before they became a member?

```
SELECT
    s.customer_id, COUNT(s.product_id), SUM(price)
FROM
    sales s 
    JOIN menu m ON (s.product_id = m.product_id)
    JOIN members mem ON (s.customer_id = mem.customer_id)
WHERE order_date < join_date
GROUP BY s.customer_id
ORDER BY s.customer_id;
```

| customer_id |	count |	sum |
| --- | --- | --- |
| A | 2 | 25 |
| B | 3 | 40 |

<hr>

9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

```
SELECT
    customer_id,
    SUM(CASE WHEN 
        product_name = 'sushi' THEN 20*price
        ELSE 10*price END) AS points
FROM
    sales s
    JOIN menu m ON (s.product_id = m.product_id)
GROUP BY customer_id
ORDER BY customer_id;
```

| customer_id |	points |
| --- | --- |
| A | 860 |
| B | 940 |
| C | 360 |

<hr>

10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

```
SELECT
    s.customer_id,
    SUM(
        CASE 
            WHEN product_name = 'sushi' THEN 20*price
            WHEN order_date - join_date BETWEEN 0 AND 6 THEN 20*price
        ELSE 10*price END) AS points
FROM
    sales s
    JOIN menu m ON (s.product_id = m.product_id)
    JOIN members mem ON (s.customer_id = mem.customer_id)
WHERE 
    order_date <= '2021-01-31' 
    AND order_date >= join_date
GROUP BY s.customer_id
ORDER BY s.customer_id;
```

| customer_id |	points |
| --- | --- |
| A | 1020 |
| B | 320 |


### Bonus Questions
#### Join All The Things
The following questions are related creating basic data tables that Danny and his team can use to quickly derive insights without needing to join the underlying tables using SQL.

Recreate the following table output using the available data:

TODO: Add image



#### Rank All The Things
Danny also requires further information about the ranking of customer products, but he purposely does not need the ranking for non-member purchases so he expects null ranking values for the records when customers are not yet part of the loyalty program.

