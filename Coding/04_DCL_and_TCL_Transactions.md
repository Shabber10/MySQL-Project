# 04. Data Control Language (DCL) & Transaction Control Language (TCL)

- **DCL (Data Control Language)**: Controls user permissions and security (`GRANT`, `REVOKE`).
- **TCL (Transaction Control Language)**: Manages database transactions to ensure ACID compliance (`COMMIT`, `ROLLBACK`, `SAVEPOINT`).

---

## 1. DATA CONTROL LANGUAGE (GRANT & REVOKE)

### Question 1.1: Grant Read-Only Permissions to Analyst
**Scenario:** Create a read-only role or user `'analyst_user'@'localhost'` with password `'SecurePass123!'` and grant `SELECT` privilege on only `customers`, `orders`, and `products` tables.

#### Expected Output
```text
Query OK, 0 rows affected (0.01 sec)
```

#### SQL Query
```sql
-- Create User
CREATE USER IF NOT EXISTS 'analyst_user'@'localhost' IDENTIFIED BY 'SecurePass123!';

-- Grant specific SELECT permissions
GRANT SELECT ON e_commerce.customers TO 'analyst_user'@'localhost';
GRANT SELECT ON e_commerce.orders TO 'analyst_user'@'localhost';
GRANT SELECT ON e_commerce.products TO 'analyst_user'@'localhost';

-- Apply Privileges
FLUSH PRIVILEGES;
```

---

### Question 1.2: Revoke Insert & Update Privileges
**Scenario:** Revoke `INSERT` and `UPDATE` permissions on the `orders` table from user `'inventory_manager'@'localhost'`.

#### Expected Output
```text
Query OK, 0 rows affected (0.01 sec)
```

#### SQL Query
```sql
REVOKE INSERT, UPDATE ON e_commerce.orders 
FROM 'inventory_manager'@'localhost';

FLUSH PRIVILEGES;
```

---

## 2. TRANSACTION CONTROL LANGUAGE (COMMIT, ROLLBACK & SAVEPOINT)

### Question 2.1: Atomic Order Placement Transaction
**Scenario:** Write a transaction that inserts a new order for `customer_id = 1` and deducts `1` unit from product `1` in `inventory`. Commit if successful.

#### Expected Output
```text
Query OK, 0 rows affected (0.00 sec)
Query OK, 1 row affected (0.01 sec)
Query OK, 1 row affected (0.01 sec)
Query OK, 0 rows affected (0.01 sec)
```

#### SQL Query
```sql
START TRANSACTION;

-- 1. Insert new order
INSERT INTO orders (customer_id, address_id, order_date, order_status, total_amount)
VALUES (1, 1, NOW(), 'Processing', 2999.00);

-- 2. Deduct inventory stock
UPDATE inventory 
SET quantity = quantity - 1, 
    updated_at = NOW() 
WHERE product_id = 1;

-- Commit transaction
COMMIT;
```

---

### Question 2.2: Transaction with SAVEPOINT and Partial Rollback
**Scenario:** Start a transaction, insert an order item, create a `SAVEPOINT sp1`, attempt to add an invalid item, and rollback to `sp1` before committing.

#### Expected Output
```text
Query OK, 0 rows affected (0.00 sec)
Query OK, 1 row affected (0.01 sec)
Query OK, 0 rows affected (0.00 sec)
Query OK, 0 rows affected (0.00 sec)
Query OK, 0 rows affected (0.01 sec)
```

#### SQL Query
```sql
START TRANSACTION;

-- Step 1: Insert valid order item
INSERT INTO order_items (order_id, product_id, quantity, price)
VALUES (1, 3, 1, 1499.00);

-- Step 2: Set Savepoint
SAVEPOINT item_added;

-- Step 3: Insert item with incorrect data (simulated error scenario)
INSERT INTO order_items (order_id, product_id, quantity, price)
VALUES (1, 999, 1, 500.00);

-- Step 4: Rollback only the faulty step to the savepoint
ROLLBACK TO SAVEPOINT item_added;

-- Step 5: Safely commit the valid portion
COMMIT;
```
