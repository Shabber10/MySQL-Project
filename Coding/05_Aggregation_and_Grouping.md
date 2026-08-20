# 05. Aggregation & Grouping

Aggregation queries compute summary statistics over sets of rows using aggregate functions (`COUNT`, `SUM`, `AVG`, `MIN`, `MAX`) combined with `GROUP BY`, `HAVING`, and `WITH ROLLUP`.

---

## 1. AGGREGATE FUNCTIONS (COUNT, SUM, AVG, MIN, MAX)

### Question 1.1: Overall Sales and Order Metrics
**Scenario:** Calculate the total number of orders placed, total revenue generated, average order value, minimum order value, and maximum order value across all non-cancelled orders.

#### Expected Output
| total_orders | total_revenue | avg_order_value | min_order_value | max_order_value |
| :--- | :--- | :--- | :--- | :--- |
| 7 | 19090.00 | 2727.14 | 599.00 | 7998.00 |

#### SQL Query
```sql
SELECT 
    COUNT(order_id) AS total_orders,
    SUM(total_amount) AS total_revenue,
    ROUND(AVG(total_amount), 2) AS avg_order_value,
    MIN(total_amount) AS min_order_value,
    MAX(total_amount) AS max_order_value
FROM orders
WHERE order_status != 'Cancelled';
```

---

### Question 1.2: Inventory Stock Summary per Product
**Scenario:** Find the total inventory units in stock, average units per product, lowest in-stock quantity, and highest in-stock quantity.

#### Expected Output
| total_stock_units | avg_stock_per_product | min_stock | max_stock |
| :--- | :--- | :--- | :--- |
| 830 | 83.00 | 30 | 200 |

#### SQL Query
```sql
SELECT 
    SUM(quantity) AS total_stock_units,
    ROUND(AVG(quantity), 2) AS avg_stock_per_product,
    MIN(quantity) AS min_stock,
    MAX(quantity) AS max_stock
FROM inventory;
```

---

## 2. GROUP BY & HAVING CLAUSE

### Question 2.1: Category-Wise Product Count & Average Price
**Scenario:** Group products by category name, showing category name, total products in each category, and average product price. Only include categories having an average price greater than `₹1,000`.

#### Expected Output
| category_name | total_products | avg_price |
| :--- | :--- | :--- |
| Electronics | 3 | 3165.67 |
| Fashion | 2 | 1599.00 |
| Home & Kitchen | 2 | 1599.00 |
| Sports & Fitness | 2 | 1449.00 |

#### SQL Query
```sql
SELECT 
    c.category_name,
    COUNT(p.product_id) AS total_products,
    ROUND(AVG(p.price), 2) AS avg_price
FROM categories c
JOIN products p ON c.category_id = p.category_id
GROUP BY c.category_name
HAVING AVG(p.price) > 1000.00
ORDER BY avg_price DESC;
```

---

### Question 2.2: Customers with High Order Volume
**Scenario:** Find customers who have placed at least 1 order and spent a cumulative total of more than `₹2,500`. Display customer name, total orders, and total amount spent.

#### Expected Output
| customer_id | customer_name | orders_count | total_spent |
| :--- | :--- | :--- | :--- |
| 1 | Aarav Sharma | 1 | 7998.00 |
| 3 | Rohan Verma | 1 | 3198.00 |
| 5 | Vikram Singh | 1 | 2898.00 |

#### SQL Query
```sql
SELECT 
    c.customer_id,
    CONCAT(c.first_name, ' ', c.last_name) AS customer_name,
    COUNT(o.order_id) AS orders_count,
    SUM(o.total_amount) AS total_spent
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
WHERE o.order_status != 'Cancelled'
GROUP BY c.customer_id, customer_name
HAVING SUM(o.total_amount) > 2500.00
ORDER BY total_spent DESC;
```

---

## 3. MULTI-LEVEL AGGREGATION & WITH ROLLUP

### Question 3.1: Revenue by Payment Method & Status with Super-Totals
**Scenario:** Calculate the total amount paid grouped by `payment_method` and `payment_status` using `WITH ROLLUP` to display sub-totals and grand total.

#### Expected Output
| payment_method | payment_status | total_paid |
| :--- | :--- | :--- |
| Credit Card | Completed | 7998.00 |
| Credit Card | NULL | 7998.00 |
| Debit Card | Pending | 2898.00 |
| Debit Card | NULL | 2898.00 |
| Net Banking | Completed | 3198.00 |
| Net Banking | NULL | 3198.00 |
| UPI | Completed | 3998.00 |
| UPI | NULL | 3998.00 |
| NULL | NULL | 18092.00 |

#### SQL Query
```sql
SELECT 
    IFNULL(payment_method, 'ALL METHODS') AS payment_method,
    IFNULL(payment_status, 'SUBTOTAL') AS payment_status,
    SUM(amount_paid) AS total_paid
FROM payments
GROUP BY payment_method, payment_status WITH ROLLUP;
```

---

### Question 3.2: Order Status Distribution by State
**Scenario:** Count the number of orders and total revenue per state and order status.

#### Expected Output
| state | order_status | total_orders | total_value |
| :--- | :--- | :--- | :--- |
| Delhi | Shipped | 1 | 3198.00 |
| Gujarat | Cancelled | 1 | 4999.00 |
| Karnataka | Delivered | 1 | 7998.00 |
| Maharashtra | Processing | 1 | 1499.00 |
| Punjab | Shipped | 1 | 599.00 |
| Tamil Nadu | Delivered | 1 | 1398.00 |
| Telangana | Pending... | 1 | 2898.00 |
| West Bengal | Delivered | 1 | 2499.00 |

#### SQL Query
```sql
SELECT 
    a.state,
    o.order_status,
    COUNT(o.order_id) AS total_orders,
    SUM(o.total_amount) AS total_value
FROM orders o
JOIN address a ON o.address_id = a.address_id
GROUP BY a.state, o.order_status
ORDER BY a.state;
```
