# E-Commerce Database Sample Data (INSERT Statements)

This document contains realistic sample data for all 11 tables in the `e_commerce` database.

---

## 1. Raw SQL Data Insertion Script

```sql
USE e_commerce;

-- ============================================================
-- 1. INSERT INTO CUSTOMERS
-- ============================================================
INSERT INTO customers (customer_id, first_name, last_name, e_mail, phone_number, created_at) VALUES
(1, 'Aarav', 'Sharma', 'aarav.sharma@gmail.com', '9876543210', '2026-01-10 10:15:00'),
(2, 'Priya', 'Patel', 'priya.patel@yahoo.com', '9876543211', '2026-01-12 11:30:00'),
(3, 'Rohan', 'Verma', 'rohan.v@outlook.com', '9876543212', '2026-01-15 14:45:00'),
(4, 'Ananya', 'Gupta', 'ananya.g@gmail.com', '9876543213', '2026-01-18 09:20:00'),
(5, 'Vikram', 'Singh', 'vikram.s@gmail.com', '9876543214', '2026-01-20 16:10:00'),
(6, 'Neha', 'Reddy', 'neha.reddy@gmail.com', '9876543215', '2026-01-22 18:05:00'),
(7, 'Amit', 'Kumar', 'amit.kumar@yahoo.com', '9876543216', '2026-01-25 12:40:00'),
(8, 'Sneha', 'Joshi', 'sneha.j@gmail.com', '9876543217', '2026-01-28 15:50:00'),
(9, 'Rahul', 'Nair', 'rahul.nair@gmail.com', '9876543218', '2026-02-01 08:30:00'),
(10, 'Kavya', 'Rao', 'kavya.rao@outlook.com', '9876543219', '2026-02-05 13:15:00');


-- ============================================================
-- 2. INSERT INTO ADDRESS
-- ============================================================
INSERT INTO address (address_id, customer_id, address_line, city, state, pincode, country) VALUES
(1, 1, '101 MG Road, Sector 14', 'Bengaluru', 'Karnataka', '560001', 'INDIA'),
(2, 2, '202 Park Street, Flat 4B', 'Kolkata', 'West Bengal', '700016', 'INDIA'),
(3, 3, '303 Connaught Place', 'New Delhi', 'Delhi', '110001', 'INDIA'),
(4, 4, '404 Marine Drive', 'Mumbai', 'Maharashtra', '400020', 'INDIA'),
(5, 5, '505 Jubilee Hills', 'Hyderabad', 'Telangana', '500033', 'INDIA'),
(6, 6, '606 Anna Salai', 'Chennai', 'Tamil Nadu', '600002', 'INDIA'),
(7, 7, '707 FC Road, Shivajinagar', 'Pune', 'Maharashtra', '411005', 'INDIA'),
(8, 8, '808 SG Highway', 'Ahmedabad', 'Gujarat', '380015', 'INDIA'),
(9, 9, '909 Sector 17', 'Chandigarh', 'Punjab', '160017', 'INDIA'),
(10, 10, '1010 Civil Lines', 'Jaipur', 'Rajasthan', '302006', 'INDIA');


-- ============================================================
-- 3. INSERT INTO CATEGORIES
-- ============================================================
INSERT INTO categories (category_id, category_name, created_at) VALUES
(1, 'Electronics', '2026-01-01 00:00:00'),
(2, 'Fashion', '2026-01-01 00:00:00'),
(3, 'Home & Kitchen', '2026-01-01 00:00:00'),
(4, 'Books', '2026-01-01 00:00:00'),
(5, 'Sports & Fitness', '2026-01-01 00:00:00');


-- ============================================================
-- 4. INSERT INTO PRODUCTS
-- ============================================================
INSERT INTO products (product_id, category_id, product_name, price) VALUES
(1, 1, 'Wireless Earbuds', 2999.00),
(2, 1, 'Smartwatch Pro', 4999.00),
(3, 1, 'Gaming Mouse', 1499.00),
(4, 2, 'Men Denim Jacket', 2499.00),
(5, 2, 'Cotton T-Shirt', 699.00),
(6, 3, 'Stainless Kettle', 1899.00),
(7, 3, 'Non-Stick Pan', 1299.00),
(8, 4, 'SQL Mastery Guide', 599.00),
(9, 5, 'Yoga Mat Pro', 899.00),
(10, 5, 'Dumbbell Set 10kg', 1999.00);


-- ============================================================
-- 5. INSERT INTO INVENTORY
-- ============================================================
INSERT INTO inventory (inventory_id, product_id, quantity, updated_at) VALUES
(1, 1, 50, '2026-02-01 10:00:00'),
(2, 2, 30, '2026-02-01 10:00:00'),
(3, 3, 100, '2026-02-01 10:00:00'),
(4, 4, 45, '2026-02-01 10:00:00'),
(5, 5, 200, '2026-02-01 10:00:00'),
(6, 6, 60, '2026-02-01 10:00:00'),
(7, 7, 80, '2026-02-01 10:00:00'),
(8, 8, 150, '2026-02-01 10:00:00'),
(9, 9, 75, '2026-02-01 10:00:00'),
(10, 10, 40, '2026-02-01 10:00:00');


-- ============================================================
-- 6. INSERT INTO CART
-- ============================================================
INSERT INTO cart (cart_id, customer_id, created_at, updated_at) VALUES
(1, 1, '2026-02-10 11:00:00', '2026-02-10 11:05:00'),
(2, 2, '2026-02-10 12:00:00', '2026-02-10 12:10:00'),
(3, 3, '2026-02-11 09:30:00', '2026-02-11 09:40:00'),
(4, 4, '2026-02-11 14:15:00', '2026-02-11 14:20:00'),
(5, 5, '2026-02-12 16:45:00', '2026-02-12 16:50:00');


-- ============================================================
-- 7. INSERT INTO CART_ITEMS
-- ============================================================
INSERT INTO cart_items (cart_item_id, cart_id, product_id, quantity, added_at) VALUES
(1, 1, 1, 1, '2026-02-10 11:02:00'),
(2, 1, 3, 2, '2026-02-10 11:05:00'),
(3, 2, 5, 3, '2026-02-10 12:10:00'),
(4, 3, 8, 1, '2026-02-11 09:35:00'),
(5, 3, 9, 1, '2026-02-11 09:40:00'),
(6, 4, 2, 1, '2026-02-11 14:20:00'),
(7, 5, 6, 2, '2026-02-12 16:50:00');


-- ============================================================
-- 8. INSERT INTO ORDERS
-- ============================================================
INSERT INTO orders (order_id, customer_id, address_id, order_date, order_status, total_amount) VALUES
(1, 1, 1, '2026-02-01 10:00:00', 'Delivered', 7998.00),
(2, 2, 2, '2026-02-02 11:30:00', 'Delivered', 2499.00),
(3, 3, 3, '2026-02-03 14:15:00', 'Shipped', 3198.00),
(4, 4, 4, '2026-02-04 09:45:00', 'Processing', 1499.00),
(5, 5, 5, '2026-02-05 16:20:00', 'Pending...', 2898.00),
(6, 6, 6, '2026-02-06 18:00:00', 'Delivered', 1398.00),
(7, 7, 7, '2026-02-07 12:10:00', 'Shipped', 599.00),
(8, 8, 8, '2026-02-08 15:30:00', 'Cancelled', 4999.00);


-- ============================================================
-- 9. INSERT INTO ORDER_ITEMS
-- ============================================================
INSERT INTO order_items (order_item_id, order_id, product_id, quantity, price) VALUES
(1, 1, 1, 1, 2999.00),
(2, 1, 2, 1, 4999.00),
(3, 2, 4, 1, 2499.00),
(4, 3, 6, 1, 1899.00),
(5, 3, 7, 1, 1299.00),
(6, 4, 3, 1, 1499.00),
(7, 5, 9, 1, 899.00),
(8, 5, 10, 1, 1999.00),
(9, 6, 5, 2, 699.00),
(10, 7, 8, 1, 599.00),
(11, 8, 2, 1, 4999.00);


-- ============================================================
-- 10. INSERT INTO PAYMENTS
-- ============================================================
INSERT INTO payments (payment_id, order_id, payment_method, payment_status, amount_paid, payment_date, transaction_id) VALUES
(1, 1, 'Credit Card', 'Completed', 7998.00, '2026-02-01 10:05:00', 987654321001),
(2, 2, 'UPI', 'Completed', 2499.00, '2026-02-02 11:32:00', 987654321002),
(3, 3, 'Net Banking', 'Completed', 3198.00, '2026-02-03 14:18:00', 987654321003),
(4, 4, 'UPI', 'Completed', 1499.00, '2026-02-04 09:47:00', 987654321004),
(5, 5, 'Debit Card', 'Pending', 2898.00, '2026-02-05 16:22:00', 987654321005),
(6, 6, 'UPI', 'Completed', 1398.00, '2026-02-06 18:02:00', 987654321006),
(7, 7, 'Credit Card', 'Completed', 599.00, '2026-02-07 12:12:00', 987654321007),
(8, 8, 'Cash on Delivery', 'Failed', 4999.00, '2026-02-08 15:35:00', 987654321008);


-- ============================================================
-- 11. INSERT INTO SHIPMENTS
-- ============================================================
INSERT INTO shipments (shipment_id, order_id, shipment_status, delivery_date, address_id, phone_number, tracking_number) VALUES
(1, 1, 'Delivered', '2026-02-05 14:30:00', 1, '9876543210', 'TRK1000000001'),
(2, 2, 'Delivered', '2026-02-07 16:00:00', 2, '9876543211', 'TRK1000000002'),
(3, 3, 'In Transit', NULL, 3, '9876543212', 'TRK1000000003'),
(4, 4, 'Order Placed', NULL, 4, '9876543213', 'TRK1000000004'),
(5, 5, 'Order Placed', NULL, 5, '9876543214', 'TRK1000000005'),
(6, 6, 'Delivered', '2026-02-10 11:15:00', 6, '9876543215', 'TRK1000000006'),
(7, 7, 'In Transit', NULL, 7, '9876543216', 'TRK1000000007'),
(8, 8, 'Cancelled', NULL, 8, '9876543217', 'TRK1000000008');
```

---

## 2. Visual Table Previews

### Customers Table
| customer_id | first_name | last_name | e_mail | phone_number | created_at |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | Aarav | Sharma | aarav.sharma@gmail.com | 9876543210 | 2026-01-10 10:15:00 |
| 2 | Priya | Patel | priya.patel@yahoo.com | 9876543211 | 2026-01-12 11:30:00 |
| 3 | Rohan | Verma | rohan.v@outlook.com | 9876543212 | 2026-01-15 14:45:00 |
| 4 | Ananya | Gupta | ananya.g@gmail.com | 9876543213 | 2026-01-18 09:20:00 |
| 5 | Vikram | Singh | vikram.s@gmail.com | 9876543214 | 2026-01-20 16:10:00 |

### Address Table
| address_id | customer_id | address_line | city | state | pincode | country |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | 1 | 101 MG Road, Sector 14 | Bengaluru | Karnataka | 560001 | INDIA |
| 2 | 2 | 202 Park Street, Flat 4B | Kolkata | West Bengal | 700016 | INDIA |
| 3 | 3 | 303 Connaught Place | New Delhi | Delhi | 110001 | INDIA |
| 4 | 4 | 404 Marine Drive | Mumbai | Maharashtra | 400020 | INDIA |
| 5 | 5 | 505 Jubilee Hills | Hyderabad | Telangana | 500033 | INDIA |

### Categories Table
| category_id | category_name | created_at |
| :--- | :--- | :--- |
| 1 | Electronics | 2026-01-01 00:00:00 |
| 2 | Fashion | 2026-01-01 00:00:00 |
| 3 | Home & Kitchen | 2026-01-01 00:00:00 |
| 4 | Books | 2026-01-01 00:00:00 |
| 5 | Sports & Fitness | 2026-01-01 00:00:00 |

### Products Table
| product_id | category_id | product_name | price |
| :--- | :--- | :--- | :--- |
| 1 | 1 | Wireless Earbuds | 2999.00 |
| 2 | 1 | Smartwatch Pro | 4999.00 |
| 3 | 1 | Gaming Mouse | 1499.00 |
| 4 | 2 | Men Denim Jacket | 2499.00 |
| 5 | 2 | Cotton T-Shirt | 699.00 |
| 6 | 3 | Stainless Kettle | 1899.00 |
| 7 | 3 | Non-Stick Pan | 1299.00 |
| 8 | 4 | SQL Mastery Guide | 599.00 |
| 9 | 5 | Yoga Mat Pro | 899.00 |
| 10 | 5 | Dumbbell Set 10kg | 1999.00 |

### Inventory Table
| inventory_id | product_id | quantity | updated_at |
| :--- | :--- | :--- | :--- |
| 1 | 1 | 50 | 2026-02-01 10:00:00 |
| 2 | 2 | 30 | 2026-02-01 10:00:00 |
| 3 | 3 | 100 | 2026-02-01 10:00:00 |
| 4 | 4 | 45 | 2026-02-01 10:00:00 |
| 5 | 5 | 200 | 2026-02-01 10:00:00 |

### Cart Table
| cart_id | customer_id | created_at | updated_at |
| :--- | :--- | :--- | :--- |
| 1 | 1 | 2026-02-10 11:00:00 | 2026-02-10 11:05:00 |
| 2 | 2 | 2026-02-10 12:00:00 | 2026-02-10 12:10:00 |
| 3 | 3 | 2026-02-11 09:30:00 | 2026-02-11 09:40:00 |

### Cart Items Table
| cart_item_id | cart_id | product_id | quantity | added_at |
| :--- | :--- | :--- | :--- | :--- |
| 1 | 1 | 1 | 1 | 2026-02-10 11:02:00 |
| 2 | 1 | 3 | 2 | 2026-02-10 11:05:00 |
| 3 | 2 | 5 | 3 | 2026-02-10 12:10:00 |

### Orders Table
| order_id | customer_id | address_id | order_date | order_status | total_amount |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | 1 | 1 | 2026-02-01 10:00:00 | Delivered | 7998.00 |
| 2 | 2 | 2 | 2026-02-02 11:30:00 | Delivered | 2499.00 |
| 3 | 3 | 3 | 2026-02-03 14:15:00 | Shipped | 3198.00 |
| 4 | 4 | 4 | 2026-02-04 09:45:00 | Processing | 1499.00 |
| 5 | 5 | 5 | 2026-02-05 16:20:00 | Pending... | 2898.00 |

### Order Items Table
| order_item_id | order_id | product_id | quantity | price |
| :--- | :--- | :--- | :--- | :--- |
| 1 | 1 | 1 | 1 | 2999.00 |
| 2 | 1 | 2 | 1 | 4999.00 |
| 3 | 2 | 4 | 1 | 2499.00 |
| 4 | 3 | 6 | 1 | 1899.00 |
| 5 | 3 | 7 | 1 | 1299.00 |

### Payments Table
| payment_id | order_id | payment_method | payment_status | amount_paid | payment_date | transaction_id |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | 1 | Credit Card | Completed | 7998.00 | 2026-02-01 10:05:00 | 987654321001 |
| 2 | 2 | UPI | Completed | 2499.00 | 2026-02-02 11:32:00 | 987654321002 |
| 3 | 3 | Net Banking | Completed | 3198.00 | 2026-02-03 14:18:00 | 987654321003 |

### Shipments Table
| shipment_id | order_id | shipment_status | delivery_date | address_id | phone_number | tracking_number |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | 1 | Delivered | 2026-02-05 14:30:00 | 1 | 9876543210 | TRK1000000001 |
| 2 | 2 | Delivered | 2026-02-07 16:00:00 | 2 | 9876543211 | TRK1000000002 |
| 3 | 3 | In Transit | *NULL* | 3 | 9876543212 | TRK1000000003 |
