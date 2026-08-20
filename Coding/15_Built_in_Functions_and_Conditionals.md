# 15. Built-in Functions & Conditional Expressions

SQL provides a rich set of built-in scalar functions for string processing, date/time arithmetic, numeric manipulation, and conditional branching logic (`CASE`, `IF`, `IFNULL`, `COALESCE`).

---

## 1. STRING & DATE/TIME FUNCTIONS

### Question 1.1: Customer Name Formatting & Email Domain Extraction
**Scenario:** Format customer full name in uppercase (`CONCAT`, `UPPER`), determine the character length of the first name, and extract only the email domain (e.g. `'gmail.com'`, `'yahoo.com'`) using `SUBSTRING_INDEX()`.

#### Expected Output
| customer_id | formatted_name | name_length | email_domain |
| :--- | :--- | :--- | :--- |
| 1 | AARAV SHARMA | 5 | gmail.com |
| 2 | PRIYA PATEL | 5 | yahoo.com |
| 3 | ROHAN VERMA | 5 | outlook.com |
| 4 | ANANYA GUPTA | 6 | gmail.com |
| 5 | VIKRAM SINGH | 6 | gmail.com |

#### SQL Query
```sql
SELECT 
    customer_id,
    UPPER(CONCAT(first_name, ' ', last_name)) AS formatted_name,
    LENGTH(first_name) AS name_length,
    SUBSTRING_INDEX(e_mail, '@', -1) AS email_domain
FROM customers
LIMIT 5;
```

---

### Question 1.2: Order Delivery Lead Time & Date Formatting
**Scenario:** Calculate the exact delivery duration in days and hours (`TIMESTAMPDIFF` or `DATEDIFF`) between `order_date` and `delivery_date`, formatting the `order_date` as `'DD-Mon-YYYY HH:MI AM/PM'`.

#### Expected Output
| order_id | formatted_order_date | delivery_lead_days |
| :--- | :--- | :--- |
| 1 | 01-Feb-2026 10:00 AM | 2.21 days |
| 2 | 02-Feb-2026 11:30 AM | 2.25 days |
| 6 | 06-Feb-2026 06:00 PM | 0.92 days |

#### SQL Query
```sql
SELECT 
    o.order_id,
    DATE_FORMAT(o.order_date, '%d-%b-%Y %h:%i %p') AS formatted_order_date,
    CONCAT(ROUND(TIMESTAMPDIFF(MINUTE, o.order_date, s.delivery_date) / 1440.0, 2), ' days') AS delivery_lead_days
FROM orders o
JOIN shipments s ON o.order_id = s.order_id
WHERE s.delivery_date IS NOT NULL
ORDER BY o.order_id;
```

---

## 2. CONDITIONAL EXPRESSIONS (CASE WHEN, IF, IFNULL, COALESCE)

### Question 2.1: Order Urgency and Size Classification (CASE WHEN)
**Scenario:** Categorize orders based on total amount:
- `'Bulk / High Value'` for total_amount >= 5000
- `'Standard Order'` for total_amount BETWEEN 2000 AND 4999.99
- `'Small Basket'` for total_amount < 2000

#### Expected Output
| order_id | customer_id | total_amount | order_classification |
| :--- | :--- | :--- | :--- |
| 1 | 1 | 7998.00 | Bulk / High Value |
| 2 | 2 | 2499.00 | Standard Order |
| 3 | 3 | 3198.00 | Standard Order |
| 4 | 4 | 1499.00 | Small Basket |
| 5 | 5 | 2898.00 | Standard Order |
| 6 | 6 | 1398.00 | Small Basket |
| 7 | 7 | 599.00 | Small Basket |
| 8 | 8 | 4999.00 | Standard Order |

#### SQL Query
```sql
SELECT 
    order_id,
    customer_id,
    total_amount,
    CASE 
        WHEN total_amount >= 5000.00 THEN 'Bulk / High Value'
        WHEN total_amount >= 2000.00 THEN 'Standard Order'
        ELSE 'Small Basket'
    END AS order_classification
FROM orders
ORDER BY order_id;
```

---

### Question 2.2: Tracking Status Fallback with COALESCE & IFNULL
**Scenario:** Display shipment tracking status, using `COALESCE` to show `'Pending Allocation'` if tracking number is null, and `IF` to flag whether delivery is complete or in progress.

#### Expected Output
| shipment_id | order_id | tracking_display | is_delivered_flag |
| :--- | :--- | :--- | :--- |
| 1 | 1 | TRK10001 | DELIVERED |
| 2 | 2 | TRK10002 | DELIVERED |
| 3 | 3 | TRK10003 | IN TRANSIT |
| 4 | 4 | TRK10004 | IN TRANSIT |
| 5 | 5 | TRK10005 | IN TRANSIT |
| 6 | 6 | TRK10006 | DELIVERED |
| 7 | 7 | TRK10007 | IN TRANSIT |

#### SQL Query
```sql
SELECT 
    shipment_id,
    order_id,
    COALESCE(tracking_number, 'Pending Allocation') AS tracking_display,
    IF(shipment_status = 'Delivered', 'DELIVERED', 'IN TRANSIT') AS is_delivered_flag
FROM shipments
ORDER BY shipment_id;
```
