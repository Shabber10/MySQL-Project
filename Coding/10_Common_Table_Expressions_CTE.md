# 10. Common Table Expressions (CTEs)

A Common Table Expression (CTE) is a named temporary result set defined within the execution scope of a single `SELECT`, `INSERT`, `UPDATE`, or `DELETE` statement using the `WITH` clause.

---

## 1. STANDARD NON-RECURSIVE CTE

### Question 1.1: Identify High-Spending Customers via CTE
**Scenario:** Using a CTE named `CustomerSpending`, calculate total spending per customer and filter out customers whose spending exceeds the average customer spend.

#### Expected Output
| customer_id | customer_name | total_spent |
| :--- | :--- | :--- |
| 1 | Aarav Sharma | 7998.00 |
| 8 | Sneha Joshi | 4999.00 |
| 3 | Rohan Verma | 3198.00 |

#### SQL Query
```sql
WITH CustomerSpending AS (
    SELECT 
        c.customer_id,
        CONCAT(c.first_name, ' ', c.last_name) AS customer_name,
        SUM(o.total_amount) AS total_spent
    FROM customers c
    JOIN orders o ON c.customer_id = o.customer_id
    GROUP BY c.customer_id, customer_name
)
SELECT customer_id, customer_name, total_spent
FROM CustomerSpending
WHERE total_spent > (SELECT AVG(total_spent) FROM CustomerSpending)
ORDER BY total_spent DESC;
```

---

### Question 1.2: Stock Replenishment Priority CTE
**Scenario:** Using a CTE `StockAnalysis`, categorize inventory into `'Critical Low'` (quantity < 40), `'Moderate'` (40-80), and `'Healthy'` (> 80), and select products needing attention.

#### Expected Output
| product_id | product_name | quantity | stock_status |
| :--- | :--- | :--- | :--- |
| 2 | Smartwatch Pro | 30 | Critical Low |
| 10 | Dumbbell Set 10kg | 40 | Moderate |
| 4 | Men Denim Jacket | 45 | Moderate |
| 1 | Wireless Earbuds | 50 | Moderate |
| 6 | Stainless Kettle | 60 | Moderate |
| 9 | Yoga Mat Pro | 75 | Moderate |
| 7 | Non-Stick Pan | 80 | Moderate |

#### SQL Query
```sql
WITH StockAnalysis AS (
    SELECT 
        p.product_id,
        p.product_name,
        i.quantity,
        CASE 
            WHEN i.quantity < 40 THEN 'Critical Low'
            WHEN i.quantity BETWEEN 40 AND 80 THEN 'Moderate'
            ELSE 'Healthy'
        END AS stock_status
    FROM products p
    JOIN inventory i ON p.product_id = i.product_id
)
SELECT product_id, product_name, quantity, stock_status
FROM StockAnalysis
WHERE stock_status IN ('Critical Low', 'Moderate')
ORDER BY quantity ASC;
```

---

## 2. MULTIPLE / CHAINED CTES

### Question 2.1: Category Performance and Revenue Contribution Percentage
**Scenario:** Use two CTEs (`CategoryTotals` and `GrandTotal`) to compute each category's revenue and its percentage share of total company sales.

#### Expected Output
| category_name | category_revenue | total_company_revenue | revenue_percentage |
| :--- | :--- | :--- | :--- |
| Electronics | 14496.00 | 25088.00 | 57.78% |
| Home & Kitchen | 3198.00 | 25088.00 | 12.75% |
| Fashion | 3197.00 | 25088.00 | 12.74% |
| Sports & Fitness | 2898.00 | 25088.00 | 11.55% |
| Books | 599.00 | 25088.00 | 2.39% |

#### SQL Query
```sql
WITH CategoryTotals AS (
    SELECT 
        c.category_name,
        SUM(oi.quantity * oi.price) AS category_revenue
    FROM categories c
    JOIN products p ON c.category_id = p.category_id
    JOIN order_items oi ON p.product_id = oi.product_id
    GROUP BY c.category_name
),
GrandTotal AS (
    SELECT SUM(category_revenue) AS total_revenue 
    FROM CategoryTotals
)
SELECT 
    ct.category_name,
    ct.category_revenue,
    gt.total_revenue AS total_company_revenue,
    CONCAT(ROUND((ct.category_revenue / gt.total_revenue) * 100, 2), '%') AS revenue_percentage
FROM CategoryTotals ct
CROSS JOIN GrandTotal gt
ORDER BY ct.category_revenue DESC;
```

---

### Question 2.2: Customer Cart vs Actual Purchase Comparison
**Scenario:** Use chained CTEs (`CartSummary` and `OrderSummary`) to compare what each customer has left in their shopping cart versus what they have actually purchased in completed orders.

#### Expected Output
| customer_id | customer_name | cart_items_count | orders_placed_count |
| :--- | :--- | :--- | :--- |
| 1 | Aarav Sharma | 2 | 1 |
| 2 | Priya Patel | 1 | 1 |
| 3 | Rohan Verma | 2 | 1 |
| 4 | Ananya Gupta | 1 | 1 |
| 5 | Vikram Singh | 1 | 1 |

#### SQL Query
```sql
WITH CartSummary AS (
    SELECT 
        c.customer_id,
        COUNT(ci.cart_item_id) AS cart_items_count
    FROM customers c
    JOIN cart crt ON c.customer_id = crt.customer_id
    JOIN cart_items ci ON crt.cart_id = ci.cart_id
    GROUP BY c.customer_id
),
OrderSummary AS (
    SELECT 
        customer_id,
        COUNT(order_id) AS orders_placed_count
    FROM orders
    GROUP BY customer_id
)
SELECT 
    c.customer_id,
    CONCAT(c.first_name, ' ', c.last_name) AS customer_name,
    COALESCE(cs.cart_items_count, 0) AS cart_items_count,
    COALESCE(os.orders_placed_count, 0) AS orders_placed_count
FROM customers c
INNER JOIN CartSummary cs ON c.customer_id = cs.customer_id
INNER JOIN OrderSummary os ON c.customer_id = os.customer_id;
```

---

## 3. RECURSIVE CTES

### Question 3.1: Generate Consecutive Date Series for Daily Sales Reporting
**Scenario:** Using a Recursive CTE, generate a continuous daily date calendar from `'2026-02-01'` to `'2026-02-08'` and left join with `orders` to display daily revenue (showing 0.00 on days with no orders).

#### Expected Output
| calendar_date | total_orders | daily_revenue |
| :--- | :--- | :--- |
| 2026-02-01 | 1 | 7998.00 |
| 2026-02-02 | 1 | 2499.00 |
| 2026-02-03 | 1 | 3198.00 |
| 2026-02-04 | 1 | 1499.00 |
| 2026-02-05 | 1 | 2898.00 |
| 2026-02-06 | 1 | 1398.00 |
| 2026-02-07 | 1 | 599.00 |
| 2026-02-08 | 1 | 4999.00 |

#### SQL Query
```sql
WITH RECURSIVE DateSeries AS (
    SELECT DATE('2026-02-01') AS calendar_date
    UNION ALL
    SELECT DATE_ADD(calendar_date, INTERVAL 1 DAY)
    FROM DateSeries
    WHERE calendar_date < '2026-02-08'
)
SELECT 
    ds.calendar_date,
    COUNT(o.order_id) AS total_orders,
    COALESCE(SUM(o.total_amount), 0.00) AS daily_revenue
FROM DateSeries ds
LEFT JOIN orders o ON DATE(o.order_date) = ds.calendar_date
GROUP BY ds.calendar_date
ORDER BY ds.calendar_date ASC;
```

---

### Question 3.2: Number Sequence Generation for EMI / Installment Calculation
**Scenario:** Use a Recursive CTE to generate a 6-month EMI installment schedule for an order worth `₹6,000` with 0% interest.

#### Expected Output
| installment_no | due_date | installment_amount | remaining_balance |
| :--- | :--- | :--- | :--- |
| 1 | 2026-03-01 | 1000.00 | 5000.00 |
| 2 | 2026-04-01 | 1000.00 | 4000.00 |
| 3 | 2026-05-01 | 1000.00 | 3000.00 |
| 4 | 2026-06-01 | 1000.00 | 2000.00 |
| 5 | 2026-07-01 | 1000.00 | 1000.00 |
| 6 | 2026-08-01 | 1000.00 | 0.00 |

#### SQL Query
```sql
WITH RECURSIVE EmiSchedule AS (
    SELECT 
        1 AS installment_no,
        DATE('2026-03-01') AS due_date,
        1000.00 AS installment_amount,
        5000.00 AS remaining_balance
    UNION ALL
    SELECT 
        installment_no + 1,
        DATE_ADD(due_date, INTERVAL 1 MONTH),
        1000.00,
        remaining_balance - 1000.00
    FROM EmiSchedule
    WHERE installment_no < 6
)
SELECT * FROM EmiSchedule;
```
