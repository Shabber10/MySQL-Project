# 12. Database Triggers

A **Trigger** is a specialized stored program executed automatically in response to specified database events (`INSERT`, `UPDATE`, or `DELETE`) on a particular table.

---

## 1. BEFORE INSERT & BEFORE UPDATE TRIGGERS

### Question 1.1: Standardize Email to Lowercase (BEFORE INSERT)
**Scenario:** Create a `BEFORE INSERT` trigger named `trg_customers_before_insert` on the `customers` table to automatically trim leading/trailing whitespace and convert the email address to lowercase.

#### Expected Output
```text
Query OK, 0 rows affected (0.02 sec)
```

#### SQL Query
```sql
DELIMITER //

CREATE TRIGGER trg_customers_before_insert
BEFORE INSERT ON customers
FOR EACH ROW
BEGIN
    SET NEW.e_mail = LOWER(TRIM(NEW.e_mail));
    SET NEW.first_name = TRIM(NEW.first_name);
    SET NEW.last_name = TRIM(NEW.last_name);
END //

DELIMITER ;
```

---

### Question 1.2: Validate Minimum Product Price (BEFORE UPDATE)
**Scenario:** Create a `BEFORE UPDATE` trigger on `products` named `trg_products_before_update` that prevents setting any product's price to a negative value or zero by throwing a custom SQLSTATE exception.

#### Expected Output
```text
Query OK, 0 rows affected (0.02 sec)
```

#### SQL Query
```sql
DELIMITER //

CREATE TRIGGER trg_products_before_update
BEFORE UPDATE ON products
FOR EACH ROW
BEGIN
    IF NEW.price <= 0 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Validation Error: Product price must be strictly greater than 0.00';
    END IF;
END //

DELIMITER ;
```

---

## 2. AFTER INSERT & AFTER UPDATE TRIGGERS

### Question 2.1: Automatic Inventory Stock Deduction (AFTER INSERT on Order Items)
**Scenario:** Create an `AFTER INSERT` trigger `trg_deduct_inventory_after_order` on `order_items` that automatically subtracts the ordered quantity from the `inventory` table for that product.

#### Expected Output
```text
Query OK, 0 rows affected (0.02 sec)
```

#### SQL Query
```sql
DELIMITER //

CREATE TRIGGER trg_deduct_inventory_after_order
AFTER INSERT ON order_items
FOR EACH ROW
BEGIN
    UPDATE inventory 
    SET quantity = quantity - NEW.quantity,
        updated_at = NOW()
    WHERE product_id = NEW.product_id;
END //

DELIMITER ;
```

---

### Question 2.2: Order Status Change Audit Logger (AFTER UPDATE on Orders)
**Scenario:** Create an `AFTER UPDATE` trigger `trg_audit_order_status_change` on `orders` that records old status, new status, order ID, and timestamp into `order_status_audit` table whenever `order_status` changes.

#### Expected Output
```text
Query OK, 0 rows affected (0.02 sec)
```

#### SQL Query
```sql
-- Ensure audit table exists
CREATE TABLE IF NOT EXISTS order_status_audit (
    audit_id INT AUTO_INCREMENT PRIMARY KEY,
    order_id INT NOT NULL,
    old_status VARCHAR(20) NOT NULL,
    new_status VARCHAR(20) NOT NULL,
    changed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

DELIMITER //

CREATE TRIGGER trg_audit_order_status_change
AFTER UPDATE ON orders
FOR EACH ROW
BEGIN
    IF OLD.order_status <> NEW.order_status THEN
        INSERT INTO order_status_audit (order_id, old_status, new_status, changed_at)
        VALUES (NEW.order_id, OLD.order_status, NEW.order_status, NOW());
    END IF;
END //

DELIMITER ;
```
