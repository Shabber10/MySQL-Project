# 06. All Types of Joins

Joins combine columns from one or more tables based on related columns. This guide covers all primary join types: **INNER JOIN**, **LEFT (OUTER) JOIN**, **RIGHT (OUTER) JOIN**, **FULL (OUTER) JOIN**, **CROSS JOIN**, and **SELF JOIN**.

---

## 1. INNER JOIN

### Question 1.1: Order Line Items with Customer and Product Details
**Scenario:** Retrieve all completed order items with customer name, order ID, product name, quantity, unit price, and total line price.

#### Expected Output
| order_id | customer_name | product_name | quantity | unit_price | line_total |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | Aarav Sharma | Wireless Earbuds | 1 | 2999.00 | 2999.00 |
| 1 | Aarav Sharma | Smartwatch Pro | 1 | 4999.00 | 4999.00 |
| 2 | Priya Patel | Men Denim Jacket | 1 | 2499.00 | 2499.00 |
| 3 | Rohan Verma | Stainless Kettle | 1 | 1899.00 | 1899.00 |
| 3 | Rohan Verma | Non-Stick Pan | 1 | 1299.00 | 1299.00 |
| 4 | Ananya Gupta | Gaming Mouse | 1 | 1499.00 | 1499.00 |
| 5 | Vikram Singh | Yoga Mat Pro | 1 | 899.00 | 899.00 |
| 5 | Vikram Singh | Dumbbell Set 10kg | 1 | 1999.00 | 1999.00 |
| 6 | Neha Reddy | Cotton T-Shirt | 2 | 699.00 | 1398.00 |
| 7 | Amit Kumar | SQL Mastery Guide | 1 | 599.00 | 599.00 |
| 8 | Sneha Joshi | Smartwatch Pro | 1 | 4999.00 | 4999.00 |

#### SQL Query
```sql
SELECT 
    o.order_id,
    CONCAT(c.first_name, ' ', c.last_name) AS customer_name,
    p.product_name,
    oi.quantity,
    oi.price AS unit_price,
    (oi.quantity * oi.price) AS line_total
FROM orders o
INNER JOIN customers c ON o.customer_id = c.customer_id
INNER JOIN order_items oi ON o.order_id = oi.order_id
INNER JOIN products p ON oi.product_id = p.product_id
ORDER BY o.order_id, p.product_name;
```

---

### Question 1.2: Products with Category and Current Stock
**Scenario:** List all products along with their category name and available inventory quantity.

#### Expected Output
| product_id | product_name | category_name | price | quantity |
| :--- | :--- | :--- | :--- | :--- |
| 1 | Wireless Earbuds | Electronics | 2999.00 | 50 |
| 2 | Smartwatch Pro | Electronics | 4999.00 | 30 |
| 3 | Gaming Mouse | Electronics | 1499.00 | 100 |
| 4 | Men Denim Jacket | Fashion | 2499.00 | 45 |
| 5 | Cotton T-Shirt | Fashion | 699.00 | 200 |
| 6 | Stainless Kettle | Home & Kitchen | 1899.00 | 60 |
| 7 | Non-Stick Pan | Home & Kitchen | 1299.00 | 80 |
| 8 | SQL Mastery Guide | Books | 599.00 | 150 |
| 9 | Yoga Mat Pro | Sports & Fitness | 899.00 | 75 |
| 10 | Dumbbell Set 10kg | Sports & Fitness | 1999.00 | 40 |

#### SQL Query
```sql
SELECT 
    p.product_id,
    p.product_name,
    c.category_name,
    p.price,
    i.quantity
FROM products p
INNER JOIN categories c ON p.category_id = c.category_id
INNER JOIN inventory i ON p.product_id = i.product_id
ORDER BY p.product_id;
```

---

## 2. LEFT (OUTER) JOIN

### Question 2.1: Find All Customers and Their Orders (Including Customers with No Orders)
**Scenario:** Retrieve all customers from the database, displaying their name, email, and order ID if they have placed any orders. Customers with zero orders must appear with `NULL` order information.

#### Expected Output
| customer_id | customer_name | e_mail | order_id | order_status |
| :--- | :--- | :--- | :--- | :--- |
| 1 | Aarav Sharma | aarav.sharma@gmail.com | 1 | Delivered |
| 2 | Priya Patel | priya.patel@yahoo.com | 2 | Delivered |
| 3 | Rohan Verma | rohan.v@outlook.com | 3 | Shipped |
| 4 | Ananya Gupta | ananya.g@gmail.com | 4 | Processing |
| 5 | Vikram Singh | vikram.s@gmail.com | 5 | Pending... |
| 6 | Neha Reddy | neha.reddy@gmail.com | 6 | Delivered |
| 7 | Amit Kumar | amit.kumar@yahoo.com | 7 | Shipped |
| 8 | Sneha Joshi | sneha.j@gmail.com | 8 | Cancelled |
| 9 | Rahul Nair | rahul.nair@gmail.com | NULL | NULL |
| 10 | Kavya Rao | kavya.rao@outlook.com | NULL | NULL |

#### SQL Query
```sql
SELECT 
    c.customer_id,
    CONCAT(c.first_name, ' ', c.last_name) AS customer_name,
    c.e_mail,
    o.order_id,
    o.order_status
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
ORDER BY c.customer_id;
```

---

### Question 2.2: Identify Products Never Ordered
**Scenario:** Find products that have never been purchased in any order using a `LEFT JOIN` and checking for `NULL` order items.

#### Expected Output
| product_id | product_name | price |
| :--- | :--- | :--- |
*(All 10 products have test order items in sample data, returning 0 rows if all purchased; query shows robust anti-join pattern)*

#### SQL Query
```sql
SELECT 
    p.product_id,
    p.product_name,
    p.price
FROM products p
LEFT JOIN order_items oi ON p.product_id = oi.product_id
WHERE oi.order_item_id IS NULL;
```

---

## 3. RIGHT (OUTER) JOIN

### Question 3.1: All Categories and Their Associated Products
**Scenario:** Display all product categories and their matching products using a `RIGHT JOIN`, ensuring all categories appear even if they have no products assigned.

#### Expected Output
| category_name | product_name | price |
| :--- | :--- | :--- |
| Books | SQL Mastery Guide | 599.00 |
| Electronics | Wireless Earbuds | 2999.00 |
| Electronics | Smartwatch Pro | 4999.00 |
| Electronics | Gaming Mouse | 1499.00 |
| Fashion | Men Denim Jacket | 2499.00 |
| Fashion | Cotton T-Shirt | 699.00 |
| Home & Kitchen | Stainless Kettle | 1899.00 |
| Home & Kitchen | Non-Stick Pan | 1299.00 |
| Sports & Fitness | Yoga Mat Pro | 899.00 |
| Sports & Fitness | Dumbbell Set 10kg | 1999.00 |

#### SQL Query
```sql
SELECT 
    c.category_name,
    p.product_name,
    p.price
FROM products p
RIGHT JOIN categories c ON p.category_id = c.category_id
ORDER BY c.category_name, p.product_name;
```

---

### Question 3.2: Right Join - Addresses and Delivered Shipments
**Scenario:** List all addresses alongside any shipment tracking number routed to that address.

#### Expected Output
| address_id | city | state | tracking_number | shipment_status |
| :--- | :--- | :--- | :--- | :--- |
| 1 | Bengaluru | Karnataka | TRK10001 | Delivered |
| 2 | Kolkata | West Bengal | TRK10002 | Delivered |
| 3 | New Delhi | Delhi | TRK10003 | In Transit |
| 4 | Mumbai | Maharashtra | TRK10004 | Dispatched |
| 5 | Hyderabad | Telangana | TRK10005 | Order Placed |
| 6 | Chennai | Tamil Nadu | TRK10006 | Delivered |
| 7 | Pune | Maharashtra | TRK10007 | In Transit |
| 8 | Ahmedabad | Gujarat | NULL | NULL |
| 9 | Chandigarh | Punjab | NULL | NULL |
| 10 | Jaipur | Rajasthan | NULL | NULL |

#### SQL Query
```sql
SELECT 
    a.address_id,
    a.city,
    a.state,
    s.tracking_number,
    s.shipment_status
FROM shipments s
RIGHT JOIN address a ON s.address_id = a.address_id
ORDER BY a.address_id;
```

---

## 4. FULL OUTER JOIN (Simulated via UNION in MySQL)

### Question 4.1: Complete Customer Cart and Order Activity
**Scenario:** Combine all customers and cart records with all orders such that customers without orders and orders without active carts are both preserved.

#### Expected Output
| customer_id | customer_name | cart_id | order_id | total_amount |
| :--- | :--- | :--- | :--- | :--- |
| 1 | Aarav Sharma | 1 | 1 | 7998.00 |
| 2 | Priya Patel | 2 | 2 | 2499.00 |
| 3 | Rohan Verma | 3 | 3 | 3198.00 |
| 4 | Ananya Gupta | 4 | 4 | 1499.00 |
| 5 | Vikram Singh | 5 | 5 | 2898.00 |
| 6 | Neha Reddy | NULL | 6 | 1398.00 |
| 7 | Amit Kumar | NULL | 7 | 599.00 |
| 8 | Sneha Joshi | NULL | 8 | 4999.00 |
| 9 | Rahul Nair | NULL | NULL | NULL |
| 10 | Kavya Rao | NULL | NULL | NULL |

#### SQL Query
```sql
SELECT 
    c.customer_id,
    CONCAT(c.first_name, ' ', c.last_name) AS customer_name,
    crt.cart_id,
    o.order_id,
    o.total_amount
FROM customers c
LEFT JOIN cart crt ON c.customer_id = crt.customer_id
LEFT JOIN orders o ON c.customer_id = o.customer_id

UNION

SELECT 
    c.customer_id,
    CONCAT(c.first_name, ' ', c.last_name) AS customer_name,
    crt.cart_id,
    o.order_id,
    o.total_amount
FROM orders o
RIGHT JOIN customers c ON o.customer_id = c.customer_id
RIGHT JOIN cart crt ON c.customer_id = crt.customer_id;
```

---

### Question 4.2: Full Matrix of Products and Cart Items
**Scenario:** Perform a full outer join simulation between `products` and `cart_items` to see which products are in carts and which products are not.

#### Expected Output
| product_id | product_name | cart_id | cart_quantity |
| :--- | :--- | :--- | :--- |
| 1 | Wireless Earbuds | 1 | 1 |
| 2 | Smartwatch Pro | 4 | 1 |
| 3 | Gaming Mouse | 1 | 2 |
| 4 | Men Denim Jacket | NULL | NULL |
| 5 | Cotton T-Shirt | 2 | 3 |
| 6 | Stainless Kettle | 5 | 2 |
| 7 | Non-Stick Pan | NULL | NULL |
| 8 | SQL Mastery Guide | 3 | 1 |
| 9 | Yoga Mat Pro | 3 | 1 |
| 10 | Dumbbell Set 10kg | NULL | NULL |

#### SQL Query
```sql
SELECT 
    p.product_id,
    p.product_name,
    ci.cart_id,
    ci.quantity AS cart_quantity
FROM products p
LEFT JOIN cart_items ci ON p.product_id = ci.product_id

UNION

SELECT 
    p.product_id,
    p.product_name,
    ci.cart_id,
    ci.quantity AS cart_quantity
FROM products p
RIGHT JOIN cart_items ci ON p.product_id = ci.product_id
ORDER BY product_id;
```

---

## 5. CROSS JOIN (Cartesian Product)

### Question 5.1: Generate Category and Discount Matrix
**Scenario:** Cross join all product categories with a fixed list of discount promotional tiers (10%, 20%, 30%) for marketing strategy planning.

#### Expected Output
| category_name | discount_tier |
| :--- | :--- |
| Electronics | Tier 1 (10%) |
| Electronics | Tier 2 (20%) |
| Electronics | Tier 3 (30%) |
| Fashion | Tier 1 (10%) |
| Fashion | Tier 2 (20%) |
| Fashion | Tier 3 (30%) |
| Home & Kitchen | Tier 1 (10%) |
| Home & Kitchen | Tier 2 (20%) |
| Home & Kitchen | Tier 3 (30%) |
| Books | Tier 1 (10%) |
| Books | Tier 2 (20%) |
| Books | Tier 3 (30%) |
| Sports & Fitness | Tier 1 (10%) |
| Sports & Fitness | Tier 2 (20%) |
| Sports & Fitness | Tier 3 (30%) |

#### SQL Query
```sql
SELECT 
    c.category_name,
    promo.tier_name AS discount_tier
FROM categories c
CROSS JOIN (
    SELECT 'Tier 1 (10%)' AS tier_name
    UNION ALL SELECT 'Tier 2 (20%)'
    UNION ALL SELECT 'Tier 3 (30%)'
) promo
ORDER BY c.category_id, promo.tier_name;
```

---

### Question 5.2: Potential Bundles (Product Pairs within Electronics)
**Scenario:** Cross join products within the `'Electronics'` category to display all possible bundle combinations (where product A != product B).

#### Expected Output
| bundle_item_1 | bundle_item_2 | combined_price |
| :--- | :--- | :--- |
| Wireless Earbuds | Smartwatch Pro | 7998.00 |
| Wireless Earbuds | Gaming Mouse | 4498.00 |
| Smartwatch Pro | Gaming Mouse | 6498.00 |

#### SQL Query
```sql
SELECT 
    p1.product_name AS bundle_item_1,
    p2.product_name AS bundle_item_2,
    (p1.price + p2.price) AS combined_price
FROM products p1
CROSS JOIN products p2
WHERE p1.category_id = 1 
  AND p2.category_id = 1 
  AND p1.product_id < p2.product_id;
```

---

## 6. SELF JOIN

### Question 6.1: Find Products in Same Category with Similar Price
**Scenario:** Self-join the `products` table to find distinct pairs of products that share the same category and have a price difference of less than `₹1,000`.

#### Expected Output
| category_id | product_1 | price_1 | product_2 | price_2 | price_difference |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | Wireless Earbuds | 2999.00 | Gaming Mouse | 1499.00 | 1500.00 |
| 3 | Stainless Kettle | 1899.00 | Non-Stick Pan | 1299.00 | 600.00 |
| 5 | Yoga Mat Pro | 899.00 | Dumbbell Set 10kg | 1999.00 | 1100.00 |

#### SQL Query
```sql
SELECT 
    p1.category_id,
    p1.product_name AS product_1,
    p1.price AS price_1,
    p2.product_name AS product_2,
    p2.price AS price_2,
    ABS(p1.price - p2.price) AS price_difference
FROM products p1
JOIN products p2 
    ON p1.category_id = p2.category_id 
    AND p1.product_id < p2.product_id
WHERE ABS(p1.price - p2.price) <= 1500.00
ORDER BY p1.category_id;
```

---

### Question 6.2: Customers Located in the Same State
**Scenario:** Self-join the `address` table to find pairs of distinct customers who reside in the same state.

#### Expected Output
| state | customer_1_id | customer_2_id |
| :--- | :--- | :--- |
| Maharashtra | 4 | 7 |

#### SQL Query
```sql
SELECT 
    a1.state,
    a1.customer_id AS customer_1_id,
    a2.customer_id AS customer_2_id
FROM address a1
JOIN address a2 
    ON a1.state = a2.state 
    AND a1.customer_id < a2.customer_id
ORDER BY a1.state;
```
