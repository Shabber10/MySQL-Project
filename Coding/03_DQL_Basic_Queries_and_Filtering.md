# 03. Data Query Language (DQL) & Filtering

Data Query Language (DQL) consists of `SELECT` statements used to retrieve and filter data from one or more tables.

---

## 1. SELECT, WHERE & COMPARISON / LOGICAL OPERATORS

### Question 1.1: Filter Products by Price Range & Category
**Scenario:** Retrieve all products from the `products` table that have a price between `₹1,000` and `₹3,000` (inclusive) and belong to category ID `1` or `2`.

#### Expected Output
| product_id | category_id | product_name | price |
| :--- | :--- | :--- | :--- |
| 1 | 1 | Wireless Earbuds | 2999.00 |
| 3 | 1 | Gaming Mouse | 1499.00 |
| 4 | 2 | Men Denim Jacket | 2499.00 |

#### SQL Query
```sql
SELECT product_id, category_id, product_name, price 
FROM products 
WHERE price BETWEEN 1000.00 AND 3000.00 
  AND category_id IN (1, 2)
ORDER BY product_id;
```

---

### Question 1.2: Filter Orders with LIKE Pattern & Status Conditions
**Scenario:** Find all customers whose email address ends with `'@gmail.com'` and whose first name starts with the letter `'A'`.

#### Expected Output
| customer_id | first_name | last_name | e_mail | phone_number |
| :--- | :--- | :--- | :--- | :--- |
| 1 | Aarav | Sharma | aarav.sharma@gmail.com | 9876543210 |
| 4 | Ananya | Gupta | ananya.g@gmail.com | 9876543213 |

#### SQL Query
```sql
SELECT customer_id, first_name, last_name, e_mail, phone_number 
FROM customers 
WHERE first_name LIKE 'A%' 
  AND e_mail LIKE '%@gmail.com'
ORDER BY customer_id;
```

---

## 2. ORDER BY, LIMIT, OFFSET & PAGINATION

### Question 2.1: Top 3 Most Expensive Products
**Scenario:** Retrieve the top 3 most expensive products from the `products` table, showing product name and price in descending order.

#### Expected Output
| product_name | price |
| :--- | :--- |
| Smartwatch Pro | 4999.00 |
| Wireless Earbuds | 2999.00 |
| Men Denim Jacket | 2499.00 |

#### SQL Query
```sql
SELECT product_name, price 
FROM products 
ORDER BY price DESC 
LIMIT 3;
```

---

### Question 2.2: Pagination (Page 2 of Orders)
**Scenario:** Fetch 3 orders for page 2 (skipping the first 3 records) ordered by `order_date` descending.

#### Expected Output
| order_id | customer_id | order_date | order_status | total_amount |
| :--- | :--- | :--- | :--- | :--- |
| 5 | 5 | 2026-02-05 16:20:00 | Pending... | 2898.00 |
| 4 | 4 | 2026-02-04 09:45:00 | Processing | 1499.00 |
| 3 | 3 | 2026-02-03 14:15:00 | Shipped | 3198.00 |

#### SQL Query
```sql
SELECT order_id, customer_id, order_date, order_status, total_amount 
FROM orders 
ORDER BY order_date DESC 
LIMIT 3 OFFSET 3;
```

---

## 3. DISTINCT & COLUMN ALIASING

### Question 3.1: Distinct Cities Served
**Scenario:** Find all unique cities where customers reside from the `address` table, sorted alphabetically.

#### Expected Output
| unique_city |
| :--- |
| Ahmedabad |
| Bengaluru |
| Chandigarh |
| Chennai |
| Hyderabad |
| Jaipur |
| Kolkata |
| Mumbai |
| New Delhi |
| Pune |

#### SQL Query
```sql
SELECT DISTINCT city AS unique_city 
FROM address 
ORDER BY unique_city ASC;
```

---

### Question 3.2: Distinct Payment Methods Used
**Scenario:** Find all distinct payment methods recorded in the `payments` table along with their payment status.

#### Expected Output
| payment_method | payment_status |
| :--- | :--- |
| Credit Card | Completed |
| UPI | Completed |
| Net Banking | Completed |
| Debit Card | Pending |

#### SQL Query
```sql
SELECT DISTINCT payment_method, payment_status 
FROM payments 
ORDER BY payment_method;
```
