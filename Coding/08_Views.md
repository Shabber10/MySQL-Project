# 08. SQL Views

A **View** is a virtual table based on the result-set of an SQL statement. Views simplify complex queries, encapsulate business logic, enforce column-level security, and allow reusable modular access.

---

## 1. SIMPLE VIEWS

### Question 1.1: Customer Contact Directory View
**Scenario:** Create a simple view `vw_customer_contacts` that exposes customer ID, full name, masked phone number (last 4 digits visible), and email for marketing communications.

#### Expected Output
```text
Query OK, 0 rows affected (0.02 sec)
```
*Querying the View:*
| customer_id | full_name | masked_phone | e_mail |
| :--- | :--- | :--- | :--- |
| 1 | Aarav Sharma | ******3210 | aarav.sharma@gmail.com |
| 2 | Priya Patel | ******3211 | priya.patel@yahoo.com |
| 3 | Rohan Verma | ******3212 | rohan.v@outlook.com |

#### SQL Query
```sql
-- Create View
CREATE OR REPLACE VIEW vw_customer_contacts AS
SELECT 
    customer_id,
    CONCAT(first_name, ' ', last_name) AS full_name,
    CONCAT('******', RIGHT(phone_number, 4)) AS masked_phone,
    e_mail
FROM customers;

-- Query the View
SELECT * FROM vw_customer_contacts LIMIT 3;
```

---

### Question 1.2: Active Available Inventory View
**Scenario:** Create a view `vw_available_stock` showing product ID, name, category ID, and current quantity where stock is greater than 50 units.

#### Expected Output
```text
Query OK, 0 rows affected (0.02 sec)
```
*Querying the View:*
| product_id | product_name | quantity |
| :--- | :--- | :--- |
| 3 | Gaming Mouse | 100 |
| 5 | Cotton T-Shirt | 200 |
| 6 | Stainless Kettle | 60 |
| 7 | Non-Stick Pan | 80 |
| 8 | SQL Mastery Guide | 150 |
| 9 | Yoga Mat Pro | 75 |

#### SQL Query
```sql
-- Create View
CREATE OR REPLACE VIEW vw_available_stock AS
SELECT 
    p.product_id,
    p.product_name,
    i.quantity
FROM products p
JOIN inventory i ON p.product_id = i.product_id
WHERE i.quantity > 50;

-- Query View
SELECT * FROM vw_available_stock;
```

---

## 2. COMPLEX & JOINED VIEWS

### Question 2.1: Comprehensive Order Fulfillment Summary View
**Scenario:** Create a complex joined view `vw_order_fulfillment_summary` that joins `orders`, `customers`, `address`, `payments`, and `shipments` to provide a complete order overview.

#### Expected Output
```text
Query OK, 0 rows affected (0.02 sec)
```
*Querying the View:*
| order_id | customer_name | city | total_amount | payment_status | shipment_status | tracking_number |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | Aarav Sharma | Bengaluru | 7998.00 | Completed | Delivered | TRK10001 |
| 2 | Priya Patel | Kolkata | 2499.00 | Completed | Delivered | TRK10002 |
| 3 | Rohan Verma | New Delhi | 3198.00 | Completed | In Transit | TRK10003 |

#### SQL Query
```sql
CREATE OR REPLACE VIEW vw_order_fulfillment_summary AS
SELECT 
    o.order_id,
    CONCAT(c.first_name, ' ', c.last_name) AS customer_name,
    a.city,
    a.state,
    o.order_date,
    o.order_status,
    o.total_amount,
    p.payment_method,
    p.payment_status,
    s.shipment_status,
    s.tracking_number
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
JOIN address a ON o.address_id = a.address_id
LEFT JOIN payments p ON o.order_id = p.order_id
LEFT JOIN shipments s ON o.order_id = s.order_id;

-- Sample Query
SELECT order_id, customer_name, city, total_amount, payment_status, shipment_status, tracking_number 
FROM vw_order_fulfillment_summary 
LIMIT 3;
```

---

### Question 2.2: Category Revenue Performance View
**Scenario:** Create a view `vw_category_sales_metrics` calculating total units sold and total revenue generated for each category.

#### Expected Output
```text
Query OK, 0 rows affected (0.02 sec)
```
*Querying the View:*
| category_name | units_sold | total_revenue |
| :--- | :--- | :--- |
| Electronics | 4 | 14496.00 |
| Home & Kitchen | 2 | 3198.00 |
| Fashion | 3 | 3197.00 |
| Sports & Fitness | 2 | 2898.00 |
| Books | 1 | 599.00 |

#### SQL Query
```sql
CREATE OR REPLACE VIEW vw_category_sales_metrics AS
SELECT 
    c.category_id,
    c.category_name,
    COALESCE(SUM(oi.quantity), 0) AS units_sold,
    COALESCE(SUM(oi.quantity * oi.price), 0.00) AS total_revenue
FROM categories c
LEFT JOIN products p ON c.category_id = p.category_id
LEFT JOIN order_items oi ON p.product_id = oi.product_id
GROUP BY c.category_id, c.category_name
ORDER BY total_revenue DESC;

-- Sample Query
SELECT category_name, units_sold, total_revenue FROM vw_category_sales_metrics;
```

---

## 3. UPDATABLE VIEWS & WITH CHECK OPTION

### Question 3.1: Updatable View with WITH CHECK OPTION
**Scenario:** Create an updatable view `vw_electronics_products` that only includes products belonging to `category_id = 1` and uses `WITH CHECK OPTION` to prevent inserting or updating products with a different category ID through the view.

#### Expected Output
```text
Query OK, 0 rows affected (0.02 sec)
```

#### SQL Query
```sql
CREATE OR REPLACE VIEW vw_electronics_products AS
SELECT 
    product_id,
    category_id,
    product_name,
    price
FROM products
WHERE category_id = 1
WITH CHECK OPTION;
```

---

### Question 3.2: Alter and Drop View Safely
**Scenario:** Demonstrate how to alter the definition of `vw_customer_contacts` to include state, and then drop it safely.

#### Expected Output
```text
Query OK, 0 rows affected (0.02 sec)
Query OK, 0 rows affected (0.01 sec)
```

#### SQL Query
```sql
-- Alter / Replace View Definition
ALTER VIEW vw_customer_contacts AS
SELECT 
    c.customer_id,
    CONCAT(c.first_name, ' ', c.last_name) AS full_name,
    c.e_mail,
    a.state
FROM customers c
LEFT JOIN address a ON c.customer_id = a.customer_id;

-- Drop View
DROP VIEW IF EXISTS vw_customer_contacts;
```
