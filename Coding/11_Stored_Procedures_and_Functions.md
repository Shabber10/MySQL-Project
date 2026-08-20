# 11. Stored Procedures & User Defined Functions

Stored Procedures and Functions are reusable routines stored in the database server that encapsulate business operations, procedural logic, and calculations.

---

## 1. STORED PROCEDURES (IN & OUT PARAMETERS)

### Question 1.1: Customer Order Summary Procedure
**Scenario:** Write a stored procedure `GetCustomerOrderStats` that takes an input parameter `p_customer_id INT` and returns two output parameters: `p_total_orders INT` and `p_total_spent DECIMAL(10,2)`.

#### Expected Output
```text
Query OK, 0 rows affected (0.01 sec)
```
*Executing the procedure for customer 1:*
| @total_orders | @total_spent |
| :--- | :--- |
| 1 | 7998.00 |

#### SQL Query
```sql
DELIMITER //

CREATE PROCEDURE GetCustomerOrderStats(
    IN p_customer_id INT,
    OUT p_total_orders INT,
    OUT p_total_spent DECIMAL(10,2)
)
BEGIN
    SELECT 
        COUNT(order_id),
        COALESCE(SUM(total_amount), 0.00)
    INTO 
        p_total_orders,
        p_total_spent
    FROM orders
    WHERE customer_id = p_customer_id 
      AND order_status != 'Cancelled';
END //

DELIMITER ;

-- Calling the Procedure:
CALL GetCustomerOrderStats(1, @total_orders, @total_spent);
SELECT @total_orders, @total_spent;
```

---

### Question 1.2: Place Order & Deduct Inventory Stock Procedure
**Scenario:** Create a stored procedure `ProcessNewOrder` that accepts customer ID, address ID, product ID, and quantity, inserts into `orders` and `order_items`, and updates `inventory` within a transactional block.

#### Expected Output
```text
Query OK, 0 rows affected (0.02 sec)
```

#### SQL Query
```sql
DELIMITER //

CREATE PROCEDURE ProcessNewOrder(
    IN p_customer_id INT,
    IN p_address_id INT,
    IN p_product_id INT,
    IN p_quantity INT,
    OUT p_status_message VARCHAR(100)
)
BEGIN
    DECLARE v_product_price DECIMAL(10,2);
    DECLARE v_available_stock INT;
    DECLARE v_new_order_id INT;

    -- Check available stock and get price
    SELECT quantity INTO v_available_stock 
    FROM inventory 
    WHERE product_id = p_product_id;

    SELECT price INTO v_product_price 
    FROM products 
    WHERE product_id = p_product_id;

    IF v_available_stock >= p_quantity THEN
        START TRANSACTION;

        -- 1. Insert Order
        INSERT INTO orders (customer_id, address_id, order_date, order_status, total_amount)
        VALUES (p_customer_id, p_address_id, NOW(), 'Processing', v_product_price * p_quantity);
        
        SET v_new_order_id = LAST_INSERT_ID();

        -- 2. Insert Order Item
        INSERT INTO order_items (order_id, product_id, quantity, price)
        VALUES (v_new_order_id, p_product_id, p_quantity, v_product_price);

        -- 3. Deduct Stock
        UPDATE inventory 
        SET quantity = quantity - p_quantity,
            updated_at = NOW()
        WHERE product_id = p_product_id;

        COMMIT;
        SET p_status_message = 'ORDER_SUCCESSFUL';
    ELSE
        SET p_status_message = 'ERROR: INSUFFICIENT_STOCK';
    END IF;
END //

DELIMITER ;
```

---

## 2. USER DEFINED SCALAR FUNCTIONS (UDF)

### Question 2.1: Calculate Discounted Price Function
**Scenario:** Create a deterministic scalar function `CalculateDiscountedPrice` that accepts `p_price DECIMAL(10,2)` and `p_discount_pct DECIMAL(5,2)` and returns the net discounted price rounded to 2 decimal places.

#### Expected Output
```text
Query OK, 0 rows affected (0.01 sec)
```
*Testing the Function:*
| product_name | original_price | discounted_price_15pct |
| :--- | :--- | :--- |
| Wireless Earbuds | 2999.00 | 2549.15 |
| Smartwatch Pro | 4999.00 | 4249.15 |

#### SQL Query
```sql
DELIMITER //

CREATE FUNCTION CalculateDiscountedPrice(
    p_price DECIMAL(10,2),
    p_discount_pct DECIMAL(5,2)
)
RETURNS DECIMAL(10,2)
DETERMINISTIC
BEGIN
    DECLARE v_discounted_price DECIMAL(10,2);
    SET v_discounted_price = p_price - (p_price * (p_discount_pct / 100.0));
    RETURN ROUND(v_discounted_price, 2);
END //

DELIMITER ;

-- Test Query
SELECT 
    product_name, 
    price AS original_price, 
    CalculateDiscountedPrice(price, 15.00) AS discounted_price_15pct
FROM products 
LIMIT 2;
```

---

### Question 2.2: Customer Loyalty Tier Function
**Scenario:** Create a function `GetCustomerTier` that takes a customer's total spend and returns `'PLATINUM'` for >= 5000, `'GOLD'` for 2000-4999, and `'SILVER'` for < 2000.

#### Expected Output
```text
Query OK, 0 rows affected (0.01 sec)
```
*Testing the Function:*
| customer_name | total_spent | loyalty_tier |
| :--- | :--- | :--- |
| Aarav Sharma | 7998.00 | PLATINUM |
| Priya Patel | 2499.00 | GOLD |
| Neha Reddy | 1398.00 | SILVER |

#### SQL Query
```sql
DELIMITER //

CREATE FUNCTION GetCustomerTier(p_total_spent DECIMAL(10,2))
RETURNS VARCHAR(20)
DETERMINISTIC
BEGIN
    DECLARE v_tier VARCHAR(20);
    
    IF p_total_spent >= 5000.00 THEN
        SET v_tier = 'PLATINUM';
    ELSEIF p_total_spent >= 2000.00 THEN
        SET v_tier = 'GOLD';
    ELSE
        SET v_tier = 'SILVER';
    END IF;
    
    RETURN v_tier;
END //

DELIMITER ;

-- Test Query
SELECT 
    CONCAT(c.first_name, ' ', c.last_name) AS customer_name,
    SUM(o.total_amount) AS total_spent,
    GetCustomerTier(SUM(o.total_amount)) AS loyalty_tier
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
WHERE o.order_status != 'Cancelled'
GROUP BY c.customer_id, customer_name
LIMIT 3;
```
