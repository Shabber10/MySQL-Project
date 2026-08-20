# 02. Data Manipulation Language (DML)

Data Manipulation Language (DML) statements manage data within database objects (inserting, modifying, deleting, and replacing rows).

---

## 1. INSERT STATEMENTS (Single & Multi-Row, INSERT INTO SELECT)

### Question 1.1: Multi-Row Bulk Insert into Categories
**Scenario:** Insert two new categories into the `categories` table: `'Beauty & Personal Care'` and `'Toys & Games'`.

#### Expected Output
```text
Query OK, 2 rows affected (0.01 sec)
Records: 2  Duplicates: 0  Warnings: 0
```

#### SQL Query
```sql
INSERT INTO categories (category_name, created_at) VALUES
('Beauty & Personal Care', NOW()),
('Toys & Games', NOW());
```

---

### Question 1.2: Insert into Cart from Wishlist / Staging
**Scenario:** Insert an item into the `cart_items` table for `cart_id = 1` adding `product_id = 4` with quantity `2`. If it already exists, update the quantity to 2 using `ON DUPLICATE KEY UPDATE`.

#### Expected Output
```text
Query OK, 1 row affected (0.01 sec)
```

#### SQL Query
```sql
INSERT INTO cart_items (cart_id, product_id, quantity, added_at)
VALUES (1, 4, 2, NOW())
ON DUPLICATE KEY UPDATE 
    quantity = VALUES(quantity),
    added_at = NOW();
```

---

## 2. UPDATE STATEMENTS (Simple & Correlated/Join Updates)

### Question 2.1: Simple Update on Order Status
**Scenario:** Update the status of order `order_id = 5` from `'Pending...'` to `'Processing'`.

#### Expected Output
```text
Query OK, 1 row affected (0.01 sec)
Rows matched: 1  Changed: 1  Warnings: 0
```

#### SQL Query
```sql
UPDATE orders 
SET order_status = 'Processing' 
WHERE order_id = 5;
```

---

### Question 2.2: Update with JOIN - Synchronize Inventory Stock
**Scenario:** Update the stock `quantity` in the `inventory` table by reducing it by `10` units for all products that belong to the `'Electronics'` category (`category_id = 1`).

#### Expected Output
```text
Query OK, 3 rows affected (0.01 sec)
Rows matched: 3  Changed: 3  Warnings: 0
```

#### SQL Query
```sql
UPDATE inventory inv
JOIN products p ON inv.product_id = p.product_id
JOIN categories c ON p.category_id = c.category_id
SET inv.quantity = inv.quantity - 10,
    inv.updated_at = NOW()
WHERE c.category_name = 'Electronics';
```

---

## 3. DELETE STATEMENTS (Conditional & Subquery/Join Delete)

### Question 3.1: Delete Inactive Abandoned Carts
**Scenario:** Delete all items from `cart_items` that belong to `cart_id = 5`.

#### Expected Output
```text
Query OK, 1 row affected (0.01 sec)
```

#### SQL Query
```sql
DELETE FROM cart_items 
WHERE cart_id = 5;
```

---

### Question 3.2: Delete Orders with Subquery / Specific Criteria
**Scenario:** Delete all payments where the corresponding order in `orders` table has `order_status = 'Cancelled'`.

#### Expected Output
```text
Query OK, 0 rows affected (0.01 sec)
```

#### SQL Query
```sql
DELETE FROM payments 
WHERE order_id IN (
    SELECT order_id 
    FROM orders 
    WHERE order_status = 'Cancelled'
);
```
