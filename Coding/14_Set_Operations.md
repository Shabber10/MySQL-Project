# 14. Set Operations

Set Operations combine the results of two or more `SELECT` queries into a single unified result set (`UNION`, `UNION ALL`, `INTERSECT`, `EXCEPT` / `MINUS`).

---

## 1. UNION & UNION ALL

### Question 1.1: Combine All Active Customer and Delivery Phone Contacts (UNION)
**Scenario:** Extract a consolidated list of distinct phone numbers from both the `customers` table and the `shipments` table to construct a customer SMS alert list.

#### Expected Output
| contact_phone | contact_source |
| :--- | :--- |
| 9876543210 | Customer Profile |
| 9876543211 | Customer Profile |
| 9876543212 | Customer Profile |
| 9876543213 | Customer Profile |
| 9876543214 | Customer Profile |
| 9876543215 | Customer Profile |
| 9876543216 | Customer Profile |
| 9876543217 | Customer Profile |
| 9876543218 | Customer Profile |
| 9876543219 | Customer Profile |

#### SQL Query
```sql
SELECT phone_number AS contact_phone, 'Customer Profile' AS contact_source 
FROM customers

UNION

SELECT phone_number AS contact_phone, 'Delivery Contact' AS contact_source 
FROM shipments
ORDER BY contact_phone;
```

---

### Question 1.2: Audit Combined Financial Inflows and Outstanding Amounts (UNION ALL)
**Scenario:** Display a combined transaction flow report showing all completed payments as positive revenue and pending order balances as pending receivables using `UNION ALL`.

#### Expected Output
| reference_id | transaction_type | amount | status |
| :--- | :--- | :--- | :--- |
| 1 | Payment Received | 7998.00 | Completed |
| 2 | Payment Received | 2499.00 | Completed |
| 3 | Payment Received | 3198.00 | Completed |
| 4 | Payment Received | 1499.00 | Completed |
| 5 | Payment Received | 2898.00 | Pending |
| 5 | Pending Receivable | 2898.00 | Order Pending |

#### SQL Query
```sql
SELECT 
    order_id AS reference_id,
    'Payment Received' AS transaction_type,
    amount_paid AS amount,
    payment_status AS status
FROM payments

UNION ALL

SELECT 
    order_id AS reference_id,
    'Pending Receivable' AS transaction_type,
    total_amount AS amount,
    'Order Pending' AS status
FROM orders
WHERE order_status = 'Pending...';
```

---

## 2. INTERSECT & EXCEPT / MINUS

### Question 2.1: Products in Carts AND in Placed Orders (INTERSECT)
**Scenario:** Find the common list of product IDs that have been added to a shopping cart AND have also been ordered by customers using the `INTERSECT` operator (or equivalent `IN` / `INNER JOIN`).

#### Expected Output
| product_id | product_name |
| :--- | :--- |
| 1 | Wireless Earbuds |
| 2 | Smartwatch Pro |
| 3 | Gaming Mouse |
| 5 | Cotton T-Shirt |
| 6 | Stainless Kettle |
| 8 | SQL Mastery Guide |
| 9 | Yoga Mat Pro |

#### SQL Query
```sql
-- MySQL 8.0.31+ Native INTERSECT syntax:
SELECT product_id FROM cart_items
INTERSECT
SELECT product_id FROM order_items;

-- Universal / Subquery equivalent:
SELECT DISTINCT p.product_id, p.product_name 
FROM products p
WHERE p.product_id IN (SELECT product_id FROM cart_items)
  AND p.product_id IN (SELECT product_id FROM order_items)
ORDER BY p.product_id;
```

---

### Question 2.2: Customers Registered BUT Never Checked Out (EXCEPT / MINUS)
**Scenario:** Find customer IDs present in the `customers` table EXCEPT those present in the `orders` table.

#### Expected Output
| customer_id |
| :--- |
| 9 |
| 10 |

#### SQL Query
```sql
-- MySQL 8.0.31+ Native EXCEPT syntax:
SELECT customer_id FROM customers
EXCEPT
SELECT customer_id FROM orders;

-- Universal / NOT IN equivalent:
SELECT customer_id 
FROM customers 
WHERE customer_id NOT IN (SELECT DISTINCT customer_id FROM orders)
ORDER BY customer_id;
```
