# 20 Advanced Technical & SQL Coding Questions

This document contains **20 Additional Technical & SQL Coding Questions (Questions 11 to 30)** for the `e_commerce` database project, covering Window Functions, CTEs, Data Integrity Checks, Subqueries, Market Basket Analysis, and Analytical Reports.

---

## 11. Category Product Ranking using DENSE_RANK()
**Question:** Rank products by price within their respective categories in descending order.

```sql
SELECT 
    c.category_name,
    p.product_name,
    p.price,
    DENSE_RANK() OVER (PARTITION BY p.category_id ORDER BY p.price DESC) AS price_rank
FROM products p
JOIN categories c ON p.category_id = c.category_id;
```

---

## 12. Customer Lifetime Value (CLV) Ranking
**Question:** Calculate total revenue generated from each customer and rank customers by lifetime spend using `RANK()`.

```sql
SELECT 
    cust.customer_id,
    CONCAT(cust.first_name, ' ', cust.last_name) AS customer_name,
    COUNT(o.order_id) AS total_orders,
    SUM(o.total_amount) AS lifetime_value,
    RANK() OVER (ORDER BY SUM(o.total_amount) DESC) AS customer_rank
FROM customers cust
JOIN orders o ON cust.customer_id = o.customer_id
WHERE o.order_status != 'Cancelled'
GROUP BY cust.customer_id, customer_name;
```

---

## 13. Stored Procedure: Stock Deduction on Order Placement
**Question:** Write a SQL Stored Procedure `DeductStock` that reduces product stock in `inventory` when an order line item is placed.

```sql
DELIMITER //

CREATE PROCEDURE DeductStock(
    IN p_product_id INT,
    IN p_quantity_purchased INT
)
BEGIN
    DECLARE current_stock INT;
    
    -- Check available stock
    SELECT quantity INTO current_stock 
    FROM inventory 
    WHERE product_id = p_product_id;
    
    IF current_stock >= p_quantity_purchased THEN
        UPDATE inventory 
        SET quantity = quantity - p_quantity_purchased,
            updated_at = CURRENT_TIMESTAMP
        WHERE product_id = p_product_id;
    ELSE
        SIGNAL SQLSTATE '45000' 
        SET MESSAGE_TEXT = 'Insufficient inventory stock to complete transaction!';
    END IF;
END //

DELIMITER ;
```

---

## 14. Delivery Lead Time Analysis
**Question:** Calculate the delivery lead time (in days) between order placement date and delivery date for completed shipments.

```sql
SELECT 
    o.order_id,
    c.first_name,
    c.last_name,
    o.order_date,
    s.delivery_date,
    DATEDIFF(s.delivery_date, o.order_date) AS delivery_lead_time_days
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
JOIN shipments s ON o.order_id = s.order_id
WHERE s.shipment_status = 'Delivered' AND s.delivery_date IS NOT NULL
ORDER BY delivery_lead_time_days ASC;
```

---

## 15. State-Wise Customer Distribution & Sales
**Question:** Group sales volume and revenue by customer state.

```sql
SELECT 
    a.state,
    COUNT(DISTINCT a.customer_id) AS customer_count,
    COUNT(DISTINCT o.order_id) AS total_orders,
    SUM(o.total_amount) AS total_state_revenue
FROM address a
JOIN orders o ON a.address_id = o.address_id
WHERE o.order_status != 'Cancelled'
GROUP BY a.state
ORDER BY total_state_revenue DESC;
```

---

## 16. Price Change Audit (Catalog vs Purchase Price Difference)
**Question:** Identify order line items where the price stored in `order_items` differs from the current catalog `products.price`.

```sql
SELECT 
    oi.order_id,
    p.product_id,
    p.product_name,
    oi.price AS purchased_price,
    p.price AS current_catalog_price,
    (p.price - oi.price) AS price_difference
FROM order_items oi
JOIN products p ON oi.product_id = p.product_id
WHERE oi.price != p.price;
```

---

## 17. Stale / Inactive Shopping Carts (Older than 7 Days)
**Question:** Retrieve cart items from shopping carts that haven't been updated for more than 7 days.

```sql
SELECT 
    c.cart_id,
    cust.customer_id,
    CONCAT(cust.first_name, ' ', cust.last_name) AS customer_name,
    cust.e_mail,
    c.updated_at AS last_cart_activity
FROM cart c
JOIN customers cust ON c.customer_id = cust.customer_id
WHERE c.updated_at < DATE_SUB(CURRENT_TIMESTAMP, INTERVAL 7 DAY);
```

---

## 18. Products Added to Cart but Never Ordered
**Question:** Find products that are currently in a customer's cart but have never been ordered by that same customer.

```sql
SELECT DISTINCT
    ci.cart_id,
    c.customer_id,
    p.product_name,
    ci.added_at
FROM cart_items ci
JOIN cart c ON ci.cart_id = c.cart_id
JOIN products p ON ci.product_id = p.product_id
LEFT JOIN orders o ON c.customer_id = o.customer_id
LEFT JOIN order_items oi ON o.order_id = oi.order_id AND ci.product_id = oi.product_id
WHERE oi.order_item_id IS NULL;
```

---

## 19. Cumulative Daily Revenue Running Total
**Question:** Compute a running total of daily revenue using Window Function `SUM() OVER ()`.

```sql
SELECT 
    DATE(order_date) AS sale_date,
    SUM(total_amount) AS daily_revenue,
    SUM(SUM(total_amount)) OVER (ORDER BY DATE(order_date)) AS cumulative_running_revenue
FROM orders
WHERE order_status != 'Cancelled'
GROUP BY DATE(order_date)
ORDER BY sale_date ASC;
```

---

## 20. Customer Onboarding to First Order Duration
**Question:** Calculate the time gap between account creation (`created_at`) and the first placed order date per customer.

```sql
SELECT 
    c.customer_id,
    CONCAT(c.first_name, ' ', c.last_name) AS customer_name,
    c.created_at AS registered_at,
    MIN(o.order_date) AS first_order_at,
    DATEDIFF(MIN(o.order_date), c.created_at) AS days_to_first_purchase
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, customer_name, c.created_at
ORDER BY days_to_first_purchase ASC;
```

---

## 21. Most Expensive Product per Category (using CTE)
**Question:** Use a Common Table Expression (CTE) to find the single highest priced product in each category.

```sql
WITH RankedProducts AS (
    SELECT 
        p.product_id,
        p.product_name,
        p.price,
        c.category_name,
        ROW_NUMBER() OVER (PARTITION BY p.category_id ORDER BY p.price DESC) AS rn
    FROM products p
    JOIN categories c ON p.category_id = c.category_id
)
SELECT 
    category_name,
    product_name,
    price
FROM RankedProducts
WHERE rn = 1;
```

---

## 22. Category Percentage Contribution to Total Revenue
**Question:** Calculate the percentage contribution of each category's sales to total company revenue.

```sql
SELECT 
    cat.category_name,
    SUM(oi.quantity * oi.price) AS category_revenue,
    ROUND(
        (SUM(oi.quantity * oi.price) / (SELECT SUM(total_amount) FROM orders WHERE order_status != 'Cancelled')) * 100, 
        2
    ) AS revenue_percentage_share
FROM categories cat
JOIN products p ON cat.category_id = p.category_id
JOIN order_items oi ON p.product_id = oi.product_id
JOIN orders o ON oi.order_id = o.order_id
WHERE o.order_status != 'Cancelled'
GROUP BY cat.category_name
ORDER BY category_revenue DESC;
```

---

## 23. Orders Containing Multiple Categories
**Question:** Find orders that contain products belonging to 2 or more distinct product categories.

```sql
SELECT 
    o.order_id,
    o.customer_id,
    COUNT(DISTINCT p.category_id) AS distinct_categories_in_order
FROM orders o
JOIN order_items oi ON o.order_id = oi.order_id
JOIN products p ON oi.product_id = p.product_id
GROUP BY o.order_id, o.customer_id
HAVING distinct_categories_in_order >= 2;
```

---

## 24. Customers with Multiple Saved Addresses
**Question:** Find all customers who have registered more than 1 address in the database.

```sql
SELECT 
    c.customer_id,
    CONCAT(c.first_name, ' ', c.last_name) AS customer_name,
    COUNT(a.address_id) AS total_addresses_saved
FROM customers c
JOIN address a ON c.customer_id = a.customer_id
GROUP BY c.customer_id, customer_name
HAVING total_addresses_saved > 1;
```

---

## 25. Data Integrity Audit: Delivered Orders with Pending Payment
**Question:** Detect any anomalous order records where shipment status is 'Delivered' but payment status is NOT 'Completed'.

```sql
SELECT 
    o.order_id,
    o.customer_id,
    o.total_amount,
    p.payment_status,
    s.shipment_status
FROM orders o
JOIN payments p ON o.order_id = p.order_id
JOIN shipments s ON o.order_id = s.order_id
WHERE s.shipment_status = 'Delivered' AND p.payment_status != 'Completed';
```

---

## 26. Cleanup Script: Delete Carts Older than 30 Days with No Items
**Question:** Write a DELETE query to remove empty carts (`cart_items` count = 0) updated over 30 days ago.

```sql
DELETE FROM cart
WHERE cart_id IN (
    SELECT c.cart_id
    FROM (SELECT * FROM cart) c
    LEFT JOIN cart_items ci ON c.cart_id = ci.cart_id
    WHERE ci.cart_item_id IS NULL
      AND c.updated_at < DATE_SUB(CURRENT_TIMESTAMP, INTERVAL 30 DAY)
);
```

---

## 27. Automated Trigger: Update Order Status on Delivery
**Question:** Create a trigger on `shipments` that updates the `orders.order_status` to 'Delivered' whenever a shipment record status changes to 'Delivered'.

```sql
DELIMITER //

CREATE TRIGGER trg_after_shipment_update
AFTER UPDATE ON shipments
FOR EACH ROW
BEGIN
    IF NEW.shipment_status = 'Delivered' THEN
        UPDATE orders 
        SET order_status = 'Delivered' 
        WHERE order_id = NEW.order_id;
    END IF;
END //

DELIMITER ;
```

---

## 28. Co-Purchased Products (Market Basket Analysis)
**Question:** Find pairs of products that are most frequently ordered together in the same order.

```sql
SELECT 
    p1.product_name AS product_1,
    p2.product_name AS product_2,
    COUNT(*) AS times_bought_together
FROM order_items oi1
JOIN order_items oi2 ON oi1.order_id = oi2.order_id AND oi1.product_id < oi2.product_id
JOIN products p1 ON oi1.product_id = p1.product_id
JOIN products p2 ON oi2.product_id = p2.product_id
GROUP BY p1.product_name, p2.product_name
ORDER BY times_bought_together DESC
LIMIT 5;
```

---

## 29. Month-Over-Month (MoM) Growth Rate using LAG()
**Question:** Calculate month-over-month percentage revenue growth using `LAG()`.

```sql
WITH MonthlySales AS (
    SELECT 
        DATE_FORMAT(order_date, '%Y-%m') AS sale_month,
        SUM(total_amount) AS revenue
    FROM orders
    WHERE order_status != 'Cancelled'
    GROUP BY DATE_FORMAT(order_date, '%Y-%m')
)
SELECT 
    sale_month,
    revenue,
    LAG(revenue, 1) OVER (ORDER BY sale_month) AS previous_month_revenue,
    ROUND(
        ((revenue - LAG(revenue, 1) OVER (ORDER BY sale_month)) / LAG(revenue, 1) OVER (ORDER BY sale_month)) * 100, 
        2
    ) AS mom_growth_percentage
FROM MonthlySales;
```

---

## 30. Analytical View: Comprehensive Customer Summary
**Question:** Create a SQL View `v_customer_analytics` that summarizes customer order statistics, lifetime spend, last order date, and default city.

```sql
CREATE VIEW v_customer_analytics AS
SELECT 
    c.customer_id,
    CONCAT(c.first_name, ' ', c.last_name) AS customer_name,
    c.e_mail,
    c.phone_number,
    COUNT(DISTINCT o.order_id) AS total_orders,
    COALESCE(SUM(o.total_amount), 0.00) AS total_lifetime_spent,
    MAX(o.order_date) AS last_order_date
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id AND o.order_status != 'Cancelled'
GROUP BY c.customer_id, customer_name, c.e_mail, c.phone_number;
```
