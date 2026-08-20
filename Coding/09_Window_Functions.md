# 09. Window Functions

Window functions perform calculations across a set of table rows that are related to the current row, without collapsing the individual rows into a single summary output.

---

## 1. RANKING FUNCTIONS (ROW_NUMBER, RANK, DENSE_RANK, NTILE)

### Question 1.1: Rank Products Within Each Category by Price
**Scenario:** Rank products in descending order of price within their respective categories using `ROW_NUMBER()`, `RANK()`, and `DENSE_RANK()` side by side to compare behavior.

#### Expected Output
| category_id | product_name | price | row_num | rnk | dense_rnk |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | Smartwatch Pro | 4999.00 | 1 | 1 | 1 |
| 1 | Wireless Earbuds | 2999.00 | 2 | 2 | 2 |
| 1 | Gaming Mouse | 1499.00 | 3 | 3 | 3 |
| 2 | Men Denim Jacket | 2499.00 | 1 | 1 | 1 |
| 2 | Cotton T-Shirt | 699.00 | 2 | 2 | 2 |
| 3 | Stainless Kettle | 1899.00 | 1 | 1 | 1 |
| 3 | Non-Stick Pan | 1299.00 | 2 | 2 | 2 |
| 4 | SQL Mastery Guide | 599.00 | 1 | 1 | 1 |
| 5 | Dumbbell Set 10kg | 1999.00 | 1 | 1 | 1 |
| 5 | Yoga Mat Pro | 899.00 | 2 | 2 | 2 |

#### SQL Query
```sql
SELECT 
    category_id,
    product_name,
    price,
    ROW_NUMBER() OVER (PARTITION BY category_id ORDER BY price DESC) AS row_num,
    RANK() OVER (PARTITION BY category_id ORDER BY price DESC) AS rnk,
    DENSE_RANK() OVER (PARTITION BY category_id ORDER BY price DESC) AS dense_rnk
FROM products
ORDER BY category_id, price DESC;
```

---

### Question 1.2: Customer Lifetime Value Segmentation (NTILE)
**Scenario:** Divide all customers who have placed orders into `3` spending tiers (High, Medium, Low) using `NTILE(3)` based on their total order spend.

#### Expected Output
| customer_id | customer_name | total_spend | tier_group |
| :--- | :--- | :--- | :--- |
| 1 | Aarav Sharma | 7998.00 | 1 |
| 8 | Sneha Joshi | 4999.00 | 1 |
| 3 | Rohan Verma | 3198.00 | 1 |
| 5 | Vikram Singh | 2898.00 | 2 |
| 2 | Priya Patel | 2499.00 | 2 |
| 4 | Ananya Gupta | 1499.00 | 2 |
| 6 | Neha Reddy | 1398.00 | 3 |
| 7 | Amit Kumar | 599.00 | 3 |

#### SQL Query
```sql
SELECT 
    c.customer_id,
    CONCAT(c.first_name, ' ', c.last_name) AS customer_name,
    SUM(o.total_amount) AS total_spend,
    NTILE(3) OVER (ORDER BY SUM(o.total_amount) DESC) AS tier_group
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, customer_name
ORDER BY total_spend DESC;
```

---

## 2. VALUE & OFFSET FUNCTIONS (LEAD & LAG)

### Question 2.1: Order-over-Order Value Growth Comparison (LAG)
**Scenario:** For each order sorted chronologically by `order_date`, show the current order amount, previous order amount using `LAG()`, and the difference in value.

#### Expected Output
| order_id | order_date | current_order_amount | previous_order_amount | amount_difference |
| :--- | :--- | :--- | :--- | :--- |
| 1 | 2026-02-01 10:00:00 | 7998.00 | NULL | NULL |
| 2 | 2026-02-02 11:30:00 | 2499.00 | 7998.00 | -5499.00 |
| 3 | 2026-02-03 14:15:00 | 3198.00 | 2499.00 | 699.00 |
| 4 | 2026-02-04 09:45:00 | 1499.00 | 3198.00 | -1699.00 |
| 5 | 2026-02-05 16:20:00 | 2898.00 | 1499.00 | 1399.00 |
| 6 | 2026-02-06 18:00:00 | 1398.00 | 2898.00 | -1500.00 |
| 7 | 2026-02-07 12:10:00 | 599.00 | 1398.00 | -799.00 |
| 8 | 2026-02-08 15:30:00 | 4999.00 | 599.00 | 4400.00 |

#### SQL Query
```sql
SELECT 
    order_id,
    order_date,
    total_amount AS current_order_amount,
    LAG(total_amount, 1) OVER (ORDER BY order_date ASC) AS previous_order_amount,
    total_amount - LAG(total_amount, 1) OVER (ORDER BY order_date ASC) AS amount_difference
FROM orders
ORDER BY order_date ASC;
```

---

### Question 2.2: Identify Next Delivery Date (LEAD)
**Scenario:** List all shipments ordered by `delivery_date` showing current shipment ID, current delivery date, and next planned delivery date using `LEAD()`.

#### Expected Output
| shipment_id | order_id | delivery_date | next_delivery_date |
| :--- | :--- | :--- | :--- |
| 1 | 1 | 2026-02-03 15:00:00 | 2026-02-04 17:30:00 |
| 2 | 2 | 2026-02-04 17:30:00 | 2026-02-07 16:00:00 |
| 6 | 6 | 2026-02-07 16:00:00 | NULL |

#### SQL Query
```sql
SELECT 
    shipment_id,
    order_id,
    delivery_date,
    LEAD(delivery_date, 1) OVER (ORDER BY delivery_date ASC) AS next_delivery_date
FROM shipments
WHERE delivery_date IS NOT NULL
ORDER BY delivery_date ASC;
```

---

## 3. AGGREGATE WINDOW FUNCTIONS (RUNNING TOTAL & MOVING AVERAGE)

### Question 3.1: Running Cumulative Revenue Over Time
**Scenario:** Calculate the cumulative running total of revenue day-by-day as orders are placed.

#### Expected Output
| order_id | order_date | total_amount | cumulative_revenue |
| :--- | :--- | :--- | :--- |
| 1 | 2026-02-01 10:00:00 | 7998.00 | 7998.00 |
| 2 | 2026-02-02 11:30:00 | 2499.00 | 10497.00 |
| 3 | 2026-02-03 14:15:00 | 3198.00 | 13695.00 |
| 4 | 2026-02-04 09:45:00 | 1499.00 | 15194.00 |
| 5 | 2026-02-05 16:20:00 | 2898.00 | 18092.00 |
| 6 | 2026-02-06 18:00:00 | 1398.00 | 19490.00 |
| 7 | 2026-02-07 12:10:00 | 599.00 | 20089.00 |
| 8 | 2026-02-08 15:30:00 | 4999.00 | 25088.00 |

#### SQL Query
```sql
SELECT 
    order_id,
    order_date,
    total_amount,
    SUM(total_amount) OVER (
        ORDER BY order_date ASC 
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS cumulative_revenue
FROM orders
ORDER BY order_date ASC;
```

---

### Question 3.2: 3-Order Moving Average of Total Amount
**Scenario:** Calculate a 3-order moving average (current order plus preceding 2 orders) of order values.

#### Expected Output
| order_id | order_date | total_amount | moving_avg_3_orders |
| :--- | :--- | :--- | :--- |
| 1 | 2026-02-01 10:00:00 | 7998.00 | 7998.00 |
| 2 | 2026-02-02 11:30:00 | 2499.00 | 5248.50 |
| 3 | 2026-02-03 14:15:00 | 3198.00 | 4565.00 |
| 4 | 2026-02-04 09:45:00 | 1499.00 | 2398.67 |
| 5 | 2026-02-05 16:20:00 | 2898.00 | 2531.67 |
| 6 | 2026-02-06 18:00:00 | 1398.00 | 1931.67 |
| 7 | 2026-02-07 12:10:00 | 599.00 | 1631.67 |
| 8 | 2026-02-08 15:30:00 | 4999.00 | 2332.00 |

#### SQL Query
```sql
SELECT 
    order_id,
    order_date,
    total_amount,
    ROUND(AVG(total_amount) OVER (
        ORDER BY order_date ASC 
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ), 2) AS moving_avg_3_orders
FROM orders
ORDER BY order_date ASC;
```
