# 13. Indexes & Constraints

- **Constraints**: Rules enforced on data columns to ensure entity integrity, domain integrity, and referential integrity (`PRIMARY KEY`, `FOREIGN KEY`, `UNIQUE`, `NOT NULL`, `CHECK`, `DEFAULT`).
- **Indexes**: Specialized B-Tree data structures that accelerate data retrieval speeds and search performance.

---

## 1. INTEGRITY CONSTRAINTS (CHECK, CASCADE, UNIQUE)

### Question 1.1: Add Check and Default Constraint
**Scenario:** Add a `CHECK` constraint to the `payments` table ensuring that `amount_paid` is strictly positive (`> 0.00`) and set a `DEFAULT` payment method of `'UPI'`.

#### Expected Output
```text
Query OK, 0 rows affected (0.03 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

#### SQL Query
```sql
ALTER TABLE payments
    ADD CONSTRAINT chk_payment_amount_positive CHECK (amount_paid > 0.00),
    ALTER COLUMN payment_method SET DEFAULT 'UPI';
```

---

### Question 1.2: Add Foreign Key with ON DELETE CASCADE
**Scenario:** Reconfigure foreign key constraint on `cart_items` pointing to `cart(cart_id)` so that if a customer's cart is deleted, all associated cart items are automatically cascaded and removed.

#### Expected Output
```text
Query OK, 0 rows affected (0.04 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

#### SQL Query
```sql
ALTER TABLE cart_items
    DROP FOREIGN KEY cart_items_ibfk_1;

ALTER TABLE cart_items
    ADD CONSTRAINT fk_cart_items_cart_cascade
    FOREIGN KEY (cart_id) REFERENCES cart(cart_id)
    ON DELETE CASCADE
    ON UPDATE CASCADE;
```

---

## 2. INDEXES & QUERY PERFORMANCE (EXPLAIN)

### Question 2.1: Create Composite Index for Multi-Column Filtering
**Scenario:** Customers and orders are frequently searched together by `customer_id` and `order_date`. Create a composite index `idx_orders_customer_date` on the `orders` table to speed up customer order history lookups.

#### Expected Output
```text
Query OK, 0 rows affected (0.03 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

#### SQL Query
```sql
CREATE INDEX idx_orders_customer_date 
ON orders (customer_id, order_date DESC);
```

---

### Question 2.2: Verify Index Usage with EXPLAIN Plan
**Scenario:** Write an `EXPLAIN` query analyzing performance for filtering orders by `customer_id = 1` and `order_date >= '2026-02-01'` to verify the optimizer utilizes the index.

#### Expected Output
| id | select_type | table | type | possible_keys | key | key_len | ref | rows | Extra |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | SIMPLE | orders | ref | idx_orders_customer_date | idx_orders_customer_date | 4 | const | 1 | Using index condition |

#### SQL Query
```sql
EXPLAIN SELECT order_id, order_date, total_amount 
FROM orders 
WHERE customer_id = 1 
  AND order_date >= '2026-02-01 00:00:00';
```
