# 01. Data Definition Language (DDL)

Data Definition Language (DDL) commands define, modify, and manage the structure of database objects such as databases, tables, columns, indexes, and constraints.

---

## 1. CREATE DATABASE & CREATE TABLE

### Question 1.1: Create Product Reviews Table
**Scenario:** Create a new table named `product_reviews` to store customer reviews and ratings for products. It must include:
- `review_id` (Primary Key, Auto-increment)
- `product_id` (Foreign Key referencing `products(product_id)` on delete cascade)
- `customer_id` (Foreign Key referencing `customers(customer_id)`)
- `rating` (Integer between 1 and 5)
- `review_text` (Text)
- `created_at` (Timestamp default current timestamp)

#### Expected Output
```text
Query OK, 0 rows affected (0.03 sec)
```

#### SQL Query
```sql
CREATE TABLE IF NOT EXISTS product_reviews (
    review_id INT AUTO_INCREMENT PRIMARY KEY,
    product_id INT NOT NULL,
    customer_id INT NOT NULL,
    rating INT NOT NULL CHECK (rating BETWEEN 1 AND 5),
    review_text TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT fk_reviews_product FOREIGN KEY (product_id) 
        REFERENCES products(product_id) ON DELETE CASCADE,
    CONSTRAINT fk_reviews_customer FOREIGN KEY (customer_id) 
        REFERENCES customers(customer_id) ON DELETE CASCADE
);
```

---

### Question 1.2: Create Audit Log Table
**Scenario:** Create a system table `order_status_audit` to track whenever an order status changes. It must record `audit_id` (PK, Auto-increment), `order_id`, `old_status`, `new_status`, `changed_by`, and `changed_at`.

#### Expected Output
```text
Query OK, 0 rows affected (0.02 sec)
```

#### SQL Query
```sql
CREATE TABLE IF NOT EXISTS order_status_audit (
    audit_id INT AUTO_INCREMENT PRIMARY KEY,
    order_id INT NOT NULL,
    old_status VARCHAR(20) NOT NULL,
    new_status VARCHAR(20) NOT NULL,
    changed_by VARCHAR(50) DEFAULT 'SYSTEM',
    changed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT fk_audit_orders FOREIGN KEY (order_id)
        REFERENCES orders(order_id) ON DELETE CASCADE
);
```

---

## 2. ALTER TABLE (ADD, MODIFY, DROP COLUMN & RENAME)

### Question 2.1: Add Discount Column and Modify Column Length
**Scenario:** Modify the `products` table to add a new column `discount_percentage` (DECIMAL(5,2) with default 0.00) and expand the `product_name` column data type to `VARCHAR(100)`.

#### Expected Output
```text
Query OK, 0 rows affected (0.04 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

#### SQL Query
```sql
ALTER TABLE products 
    ADD COLUMN discount_percentage DECIMAL(5,2) DEFAULT 0.00 AFTER price,
    MODIFY COLUMN product_name VARCHAR(100) NOT NULL;
```

---

### Question 2.2: Add Foreign Key Constraint with Alter Table
**Scenario:** Add a new foreign key constraint `fk_orders_address_del` on the `orders` table linking `address_id` to `address(address_id)` with `ON UPDATE CASCADE`.

#### Expected Output
```text
Query OK, 0 rows affected (0.05 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

#### SQL Query
```sql
ALTER TABLE orders
    ADD CONSTRAINT fk_orders_address_custom 
    FOREIGN KEY (address_id) REFERENCES address(address_id)
    ON UPDATE CASCADE;
```

---

## 3. TRUNCATE & DROP STATEMENTS

### Question 3.1: Truncate Staging / Temporary Cart Table
**Scenario:** Empty all records from the `cart_items` table quickly while resetting the auto-increment counter, without deleting the table structure itself.

#### Expected Output
```text
Query OK, 0 rows affected (0.02 sec)
```

#### SQL Query
```sql
TRUNCATE TABLE cart_items;
```

---

### Question 3.2: Safely Drop Table if Exists
**Scenario:** Write a DDL statement to permanently drop the `order_status_audit` table only if it exists in the current database schema to prevent runtime script errors.

#### Expected Output
```text
Query OK, 0 rows affected (0.02 sec)
```

#### SQL Query
```sql
DROP TABLE IF EXISTS order_status_audit;
```
