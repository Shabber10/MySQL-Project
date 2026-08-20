# Top 10 Technical & Coding Questions (SQL Queries)

This document contains the **Top 10 Frequently Asked Technical & SQL Coding Questions** based on the `e_commerce` database schema, complete with clear problem statements, explanations, and fully executable SQL queries.

---

## 1. Top 10 Selling Products by Revenue
**Question:** Write a query to list the top selling products based on total sales revenue generated. Show product ID, product name, total quantity sold, and total revenue.

```sql
SELECT 
    p.product_id,
    p.product_name,
    c.category_name,
    SUM(oi.quantity) AS total_units_sold,
    SUM(oi.quantity * oi.price) AS total_revenue
FROM order_items oi
JOIN products p ON oi.product_id = p.product_id
JOIN categories c ON p.category_id = c.category_id
JOIN orders o ON oi.order_id = o.order_id
WHERE o.order_status != 'Cancelled'
GROUP BY p.product_id, p.product_name, c.category_name
ORDER BY total_revenue DESC
LIMIT 10;
```

---

## 2. Category-Wise Sales Summary
**Question:** Calculate total revenue, total orders, and average item price for each product category.

```sql
SELECT 
    c.category_id,
    c.category_name,
    COUNT(DISTINCT oi.order_id) AS total_orders,
    SUM(oi.quantity) AS total_items_sold,
    SUM(oi.quantity * oi.price) AS total_category_revenue,
    AVG(oi.price) AS avg_item_price
FROM categories c
JOIN products p ON c.category_id = p.category_id
JOIN order_items oi ON p.product_id = oi.product_id
JOIN orders o ON oi.order_id = o.order_id
WHERE o.order_status != 'Cancelled'
GROUP BY c.category_id, c.category_name
ORDER BY total_category_revenue DESC;
```

---

## 3. High-Value Customer Spending Analysis
**Question:** Find all customers who have spent more than ₹3,000 in total across all their completed orders. Display customer ID, full name, email, order count, and total spent.

```sql
SELECT 
    cust.customer_id,
    CONCAT(cust.first_name, ' ', cust.last_name) AS full_name,
    cust.e_mail,
    COUNT(o.order_id) AS total_orders_placed,
    SUM(o.total_amount) AS total_amount_spent
FROM customers cust
JOIN orders o ON cust.customer_id = o.customer_id
WHERE o.order_status IN ('Delivered', 'Shipped', 'Processing')
GROUP BY cust.customer_id, cust.first_name, cust.last_name, cust.e_mail
HAVING total_amount_spent > 3000.00
ORDER BY total_amount_spent DESC;
```

---

## 4. Abandoned Cart Analysis
**Question:** Identify customers who currently have active items in their shopping cart but have not placed any orders yet.

```sql
SELECT 
    c.customer_id,
    CONCAT(c.first_name, ' ', c.last_name) AS customer_name,
    c.e_mail,
    c.phone_number,
    crt.cart_id,
    COUNT(ci.cart_item_id) AS items_in_cart,
    SUM(ci.quantity) AS total_cart_quantity
FROM customers c
JOIN cart crt ON c.customer_id = crt.customer_id
JOIN cart_items ci ON crt.cart_id = ci.cart_id
LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE o.order_id IS NULL
GROUP BY c.customer_id, customer_name, c.e_mail, c.phone_number, crt.cart_id;
```

---

## 5. Low Inventory Stock Alert Report
**Question:** Retrieve all products whose current inventory level is less than 50 units, along with their category name and current stock status.

```sql
SELECT 
    p.product_id,
    p.product_name,
    cat.category_name,
    p.price,
    i.quantity AS current_stock,
    CASE 
        WHEN i.quantity = 0 THEN 'OUT OF STOCK'
        WHEN i.quantity < 30 THEN 'CRITICAL LOW'
        WHEN i.quantity < 50 THEN 'LOW STOCK'
        ELSE 'IN STOCK'
    END AS stock_status
FROM products p
JOIN categories cat ON p.category_id = cat.category_id
JOIN inventory i ON p.product_id = i.product_id
WHERE i.quantity < 50
ORDER BY i.quantity ASC;
```

---

## 6. Complete Order & Tracking Status View
**Question:** Write a query to display complete order details including Customer Name, Delivery City, Order Date, Total Amount, Payment Method, Payment Status, and Shipment Tracking Number.

```sql
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
JOIN payments p ON o.order_id = p.order_id
JOIN shipments s ON o.order_id = s.order_id
ORDER BY o.order_date DESC;
```

---

## 7. Most Preferred Payment Methods Breakdown
**Question:** Find the distribution of orders and total transaction volume across different payment methods.

```sql
SELECT 
    p.payment_method,
    COUNT(p.payment_id) AS total_transactions,
    SUM(CASE WHEN p.payment_status = 'Completed' THEN 1 ELSE 0 END) AS successful_transactions,
    SUM(CASE WHEN p.payment_status = 'Failed' THEN 1 ELSE 0 END) AS failed_transactions,
    SUM(p.amount_paid) AS total_volume_processed
FROM payments p
GROUP BY p.payment_method
ORDER BY total_transactions DESC;
```

---

## 8. Customer Order Frequency & Multi-Category Buying Pattern
**Question:** List customers who have purchased items across more than one product category in their order history.

```sql
SELECT 
    cust.customer_id,
    CONCAT(cust.first_name, ' ', cust.last_name) AS customer_name,
    COUNT(DISTINCT cat.category_id) AS unique_categories_purchased,
    GROUP_CONCAT(DISTINCT cat.category_name SEPARATOR ', ') AS categories_list
FROM customers cust
JOIN orders o ON cust.customer_id = o.customer_id
JOIN order_items oi ON o.order_id = oi.order_id
JOIN products p ON oi.product_id = p.product_id
JOIN categories cat ON p.category_id = cat.category_id
WHERE o.order_status != 'Cancelled'
GROUP BY cust.customer_id, customer_name
HAVING unique_categories_purchased > 1
ORDER BY unique_categories_purchased DESC;
```

---

## 9. Products Never Purchased (Unsold Inventory Analysis)
**Question:** Find all products in the database that have never been ordered by any customer.

```sql
SELECT 
    p.product_id,
    p.product_name,
    cat.category_name,
    p.price,
    inv.quantity AS available_stock
FROM products p
JOIN categories cat ON p.category_id = cat.category_id
LEFT JOIN inventory inv ON p.product_id = inv.product_id
LEFT JOIN order_items oi ON p.product_id = oi.product_id
WHERE oi.order_item_id IS NULL;
```

---

## 10. Monthly Sales & Revenue Growth Trend
**Question:** Calculate monthly total revenue, order count, and average order value (AOV).

```sql
SELECT 
    DATE_FORMAT(o.order_date, '%Y-%m') AS sales_month,
    COUNT(DISTINCT o.order_id) AS total_orders,
    SUM(o.total_amount) AS monthly_revenue,
    ROUND(AVG(o.total_amount), 2) AS average_order_value
FROM orders o
WHERE o.order_status != 'Cancelled'
GROUP BY DATE_FORMAT(o.order_date, '%Y-%m')
ORDER BY sales_month DESC;
```
