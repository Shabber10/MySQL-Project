# Top 10 Theory & Architectural Questions

This document covers the **Top 10 Essential Theoretical, Architectural, and Conceptual Questions** for the `e_commerce` database system.

---

## 1. Database Normalization (1NF, 2NF, 3NF)
**Question:** Explain how normalization principles (1NF, 2NF, 3NF) are applied in this E-Commerce database schema.

### Answer:
The `e_commerce` schema achieves **Third Normal Form (3NF)**:

1. **First Normal Form (1NF):**
   - All columns hold atomic values (e.g., `first_name` and `last_name` are separated; single phone numbers).
   - Each row has a unique Primary Key (e.g., `customer_id`, `product_id`).
   - No repeating groups or comma-separated lists of values (e.g., items in an order are stored as separate rows in `order_items`, not as a comma-separated list in `orders`).

2. **Second Normal Form (2NF):**
   - Satisfies 1NF.
   - All non-key attributes fully depend on the entire Primary Key.
   - For example, in `order_items`, attributes like `price` and `quantity` depend on `order_item_id`, not partially on `order_id` or `product_id`.

3. **Third Normal Form (3NF):**
   - Satisfies 2NF.
   - No transitive dependencies (non-key columns depending on other non-key columns).
   - Address details (`city`, `state`, `pincode`) are in the `address` table linked via `customer_id`, rather than duplicating full addresses inside the `customers` or `orders` table.

---

## 2. Primary Key vs. Foreign Key Relationships & Entity Cardinality
**Question:** Describe the entity relationships and cardinality among key tables in this database.

### Answer:
The database defines clear cardinalities:

- **`customers` to `address` (1-to-Many / 1:N):** One customer can save multiple delivery addresses over time, but each address belongs to one customer.
- **`customers` to `cart` (1-to-1 / 1:1):** Enforced by `customer_id INT UNIQUE` in `cart`. A customer has at most one active cart.
- **`customers` to `orders` (1-to-Many / 1:N):** A customer can place multiple historical orders.
- **`categories` to `products` (1-to-Many / 1:N):** One category contains multiple products.
- **`products` to `inventory` (1-to-1 / 1:1):** Enforced by `product_id INT UNIQUE` in `inventory`. Every product maps to exactly one inventory stock record.
- **`orders` to `order_items` (1-to-Many / 1:N):** One master order contains one or many ordered line items.
- **`orders` to `payments` (1-to-1 or 1-to-Many / 1:N):** An order has associated payment records tracking transaction status.
- **`orders` to `shipments` (1-to-1 / 1:1):** An order links to tracking and shipping logistics via `order_id`.

---

## 3. Why Separate `orders` and `order_items` (Header vs. Detail Pattern)?
**Question:** Why are `orders` and `order_items` separated into two tables instead of putting all information in one table?

### Answer:
This separation follows the **Header-Detail Database Pattern**:

1. **Elimination of Data Redundancy:** Storing customer ID, address ID, order date, and total status inside every purchased product row would duplicate order header metadata dozens of times per purchase.
2. **Support for Variable Line Items:** An order can contain 1 item or 50 items. Relational databases require fixed column structures; storing items in `order_items` allows dynamic, multi-item orders without creating empty unused columns (`item1`, `item2`, `item3`).
3. **Data Integrity & Aggregation:** Header attributes like `total_amount` and `order_status` apply to the entire transaction, while `quantity` and `price` apply to individual products.

---

## 4. Maintaining ACID Properties During Order Checkout
**Question:** How are ACID (Atomicity, Consistency, Isolation, Durability) properties enforced during an e-commerce checkout transaction?

### Answer:
When a customer clicks "Place Order", multiple operations must occur atomically inside a database transaction (`START TRANSACTION ... COMMIT`):

1. **Atomicity (All-or-Nothing):** If creating the `orders` row succeeds, inserting into `order_items` succeeds, but inventory deduction fails (e.g., out of stock), the entire transaction is rolled back (`ROLLBACK`). No partial order is created.
2. **Consistency:** Database constraints (`FOREIGN KEY`, `CHECK (quantity > 0)`, `NOT NULL`) are strictly preserved. Stock cannot drop below zero if guarded by constraints/triggers.
3. **Isolation:** Concurrent checkouts by different users are isolated using transaction isolation levels (e.g., `REPEATABLE READ`). Locks prevent one transaction from reading uncommitted stock changes of another.
4. **Durability:** Once `COMMIT` executes, changes are written to disk / write-ahead logs (WAL), surviving power outages or crash recoveries.

---

## 5. Database Indexing Strategy for E-Commerce Optimization
**Question:** Which columns in this schema require indexing to maximize query performance, and why?

### Answer:
Indexes speed up search (`WHERE`), joins (`JOIN`), and ordering (`ORDER BY`):

1. **Foreign Key Columns (Automatic/Explicit B-Tree Indexes):**
   - `address(customer_id)`, `products(category_id)`, `orders(customer_id)`, `order_items(order_id, product_id)`, `payments(order_id)`.
   - *Reason:* Accelerates `JOIN` execution when retrieving order history or product catalog queries.
2. **Frequently Searched / Filtered Columns:**
   - `customers(e_mail)` — Already indexed via `UNIQUE` for fast login lookup.
   - `products(product_name)` — Already indexed via `UNIQUE` for product searches.
   - `orders(order_date, order_status)` — Composite index for dashboard date filtering and order tracking filters.
   - `shipments(tracking_number)` — `UNIQUE` index for instant shipment lookup.

---

## 6. Managing Historical Price Changes (Price Snapshot Pattern)
**Question:** Why does `order_items` have a `price` column when `products` already has a `price` column?

### Answer:
This implements the **Point-in-Time Price Snapshot Pattern**:

- Product catalog prices change over time due to discounts, inflation, or sales promotions (e.g., a product priced at ₹2,999 today might drop to ₹1,999 during a holiday sale next month).
- If `order_items` did NOT store the price at purchase time and instead joined with `products.price` dynamically, past orders would retroactively change their financial totals, corrupting historical invoices and accounting reports.
- **Rule:** Catalog price lives in `products.price`; transactional price lives in `order_items.price`.

---

## 7. Preventing Concurrency Issues & Race Conditions in Inventory
**Question:** What happens if two customers buy the last remaining unit of a product at the exact same millisecond? How do we prevent overbooking?

### Answer:
This is a classic **Race Condition (Lost Update Problem)**.

### Solutions:
1. **Pessimistic Locking (`FOR UPDATE`):**
   ```sql
   SELECT quantity FROM inventory WHERE product_id = 1 FOR UPDATE;
   ```
   Locks the row until the transaction finishes, forcing the second user to wait.
2. **Database CHECK Constraint / Atomic Conditional Update:**
   ```sql
   UPDATE inventory 
   SET quantity = quantity - 1 
   WHERE product_id = 1 AND quantity >= 1;
   ```
   If `quantity < 1`, affected rows count will be 0, alerting the application layer that stock was depleted.

---

## 8. Hard Delete vs. Soft Delete in E-Commerce Systems
**Question:** Why should you avoid `DELETE FROM customers` or `DELETE FROM products` (Hard Delete) in a production database?

### Answer:
- **Hard Delete (`DELETE FROM`):** Permanently removes data from disk.
  - *Consequences:* Breaks foreign key referential integrity with historical orders or requires cascading deletes, wiping out financial audit trails.
- **Soft Delete (`is_active` or `deleted_at` flag):**
  - Add a flag: `is_deleted TINYINT DEFAULT 0` or `deleted_at TIMESTAMP NULL`.
  - *Benefits:* Preserves historical orders, analytics integrity, and tax/legal compliance while hiding deleted products or inactive accounts from the customer UI.

---

## 9. Data Type Selection: `DECIMAL(10,2)` vs. `FLOAT` / `DOUBLE`
**Question:** Why is `DECIMAL(10,2)` strictly used for prices and order amounts instead of `FLOAT` or `DOUBLE`?

### Answer:
- **`FLOAT` and `DOUBLE` are IEEE 754 floating-point types:** They store binary approximations of real numbers, which causes imprecise rounding errors (e.g., `0.1 + 0.2 = 0.30000000000000004`).
- In financial applications, accumulated rounding errors result in missing or extra pennies across thousands of transactions.
- **`DECIMAL(10,2)` is an Exact Numeric Data Type:** It stores numbers as exact fixed-point decimals, guaranteeing 100% precision up to 10 digits with exactly 2 decimal places (e.g., ₹99999998.99).

---

## 10. Integrity Constraints: Role of `UNIQUE`, `CHECK`, and `DEFAULT`
**Question:** How do constraints defined in this schema safeguard business logic at the database layer?

### Answer:
1. **`UNIQUE` Constraints:**
   - `customers.e_mail` — Prevents duplicate account registrations with the same email.
   - `cart.customer_id` — Ensures a customer cannot spawn multiple active carts.
   - `inventory.product_id` — Ensures 1-to-1 stock record mapping per catalog item.
   - `shipments.tracking_number` & `payments.transaction_id` — Prevents collision of logistics tracking codes and bank transaction IDs.
2. **`CHECK` Constraints:**
   - `CHECK(quantity > 0)` in `cart_items` and `order_items` — Prevents negative or zero quantity items.
3. **`DEFAULT` Constraints:**
   - `country DEFAULT 'INDIA'` in `address`.
   - `order_status DEFAULT 'Pending...'` in `orders`.
   - `created_at DEFAULT CURRENT_TIMESTAMP` for audit logging.
