# 07. Subqueries & Nested Queries

A subquery is a query nested inside another SQL statement (e.g., `SELECT`, `INSERT`, `UPDATE`, `DELETE`, or within a `FROM` / `WHERE` / `HAVING` clause).

---

## 1. SINGLE-ROW & SCALAR SUBQUERIES

### Question 1.1: Products Priced Higher than Average
**Scenario:** Find all products whose price is strictly greater than the overall average price of all products in the catalog.

#### Expected Output
| product_id | product_name | price |
| :--- | :--- | :--- |
| 1 | Wireless Earbuds | 2999.00 |
| 2 | Smartwatch Pro | 4999.00 |
| 4 | Men Denim Jacket | 2499.00 |
| 6 | Stainless Kettle | 1899.00 |
| 10 | Dumbbell Set 10kg | 1999.00 |

#### SQL Query
```sql
SELECT product_id, product_name, price 
FROM products 
WHERE price > (
    SELECT AVG(price) 
    FROM products
)
ORDER BY price DESC;
```

---

### Question 1.2: Customer Who Placed the Highest Single Order
**Scenario:** Find the details of the customer who placed the single order with the highest total amount.

#### Expected Output
| customer_id | first_name | last_name | e_mail | total_amount |
| :--- | :--- | :--- | :--- | :--- |
| 1 | Aarav | Sharma | aarav.sharma@gmail.com | 7998.00 |

#### SQL Query
```sql
SELECT 
    c.customer_id, 
    c.first_name, 
    c.last_name, 
    c.e_mail,
    o.total_amount
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
WHERE o.total_amount = (
    SELECT MAX(total_amount) 
    FROM orders
);
```

---

## 2. MULTI-ROW SUBQUERIES (IN, ANY / SOME, ALL)

### Question 2.1: Products Ordered by Multiple Customers (IN Operator)
**Scenario:** Find all products that have been included in orders shipped to either `'Karnataka'` or `'Delhi'` using the `IN` operator with a nested subquery.

#### Expected Output
| product_id | product_name | price |
| :--- | :--- | :--- |
| 1 | Wireless Earbuds | 2999.00 |
| 2 | Smartwatch Pro | 4999.00 |
| 6 | Stainless Kettle | 1899.00 |
| 7 | Non-Stick Pan | 1299.00 |

#### SQL Query
```sql
SELECT DISTINCT p.product_id, p.product_name, p.price
FROM products p
WHERE p.product_id IN (
    SELECT oi.product_id
    FROM order_items oi
    WHERE oi.order_id IN (
        SELECT o.order_id
        FROM orders o
        JOIN address a ON o.address_id = a.address_id
        WHERE a.state IN ('Karnataka', 'Delhi')
    )
)
ORDER BY p.product_id;
```

---

### Question 2.2: Products Cheaper Than ALL Books (ALL Operator)
**Scenario:** Find all products in any category whose price is lower than the price of `ALL` products in category `'Electronics'`.

#### Expected Output
| product_id | product_name | price | category_id |
| :--- | :--- | :--- | :--- |
| 5 | Cotton T-Shirt | 699.00 | 2 |
| 7 | Non-Stick Pan | 1299.00 | 3 |
| 8 | SQL Mastery Guide | 599.00 | 4 |
| 9 | Yoga Mat Pro | 899.00 | 5 |

#### SQL Query
```sql
SELECT product_id, product_name, price, category_id
FROM products
WHERE price < ALL (
    SELECT price 
    FROM products 
    WHERE category_id = 1
)
ORDER BY price;
```

---

## 3. CORRELATED SUBQUERIES & EXISTS / NOT EXISTS

### Question 3.1: Products Priced Higher Than Their Category Average (Correlated)
**Scenario:** Find all products that are priced above the average price of products within their own respective category.

#### Expected Output
| product_id | product_name | category_id | price |
| :--- | :--- | :--- | :--- |
| 2 | Smartwatch Pro | 1 | 4999.00 |
| 4 | Men Denim Jacket | 2 | 2499.00 |
| 6 | Stainless Kettle | 3 | 1899.00 |
| 10 | Dumbbell Set 10kg | 5 | 1999.00 |

#### SQL Query
```sql
SELECT 
    p1.product_id, 
    p1.product_name, 
    p1.category_id, 
    p1.price
FROM products p1
WHERE p1.price > (
    SELECT AVG(p2.price) 
    FROM products p2 
    WHERE p2.category_id = p1.category_id
)
ORDER BY p1.category_id;
```

---

### Question 3.2: Customers Who Have Never Placed an Order (NOT EXISTS)
**Scenario:** Retrieve all customers who do not have any registered orders using the `NOT EXISTS` operator.

#### Expected Output
| customer_id | first_name | last_name | e_mail |
| :--- | :--- | :--- | :--- |
| 9 | Rahul | Nair | rahul.nair@gmail.com |
| 10 | Kavya | Rao | kavya.rao@outlook.com |

#### SQL Query
```sql
SELECT 
    c.customer_id, 
    c.first_name, 
    c.last_name, 
    c.e_mail
FROM customers c
WHERE NOT EXISTS (
    SELECT 1 
    FROM orders o 
    WHERE o.customer_id = c.customer_id
)
ORDER BY c.customer_id;
```
