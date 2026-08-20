# 20 Important Theoretical & Conceptual Questions

This document covers **20 Additional Essential Theoretical, Architectural, and Systems Design Questions (Questions 11 to 30)** for the `e_commerce` database system.

---

## 11. Relational (RDBMS) vs. NoSQL Databases in E-Commerce
**Question:** Should an E-Commerce system use a Relational Database (like MySQL/PostgreSQL) or a NoSQL Database (like MongoDB/DynamoDB)?

### Answer:
Modern e-commerce architectures often use a **Polyglot Persistence** approach:
- **RDBMS (MySQL/PostgreSQL):** Ideal for Core Orders, Payments, Inventory, and User accounts where ACID compliance, relational integrity, and strict consistency are mandatory.
- **NoSQL (Document / Key-Value):** Ideal for Product Catalog variations (MongoDB for unstructured specs), User Session Caching (Redis), and Product Recommendation graphs (Neo4j).

---

## 12. Database Triggers vs. Application Logic
**Question:** Should business rules (like updating inventory or order status) be implemented as Database Triggers or in Application Code?

### Answer:
- **Database Triggers:**
  - *Pros:* Guaranteed enforcement regardless of which application service writes to the DB.
  - *Cons:* Hard to debug, invisible to developers inspecting code, difficult to unit test, can create hidden performance bottlenecks.
- **Application Logic:**
  - *Pros:* Easier to test, version control, debug, scale out microservices independently.
  - *Cons:* Must be reimplemented if multiple services touch the database directly.
- **Best Practice:** Prefer application services for complex business rules; use database triggers sparingly for low-level audit logging or strict integrity guards.

---

## 13. Stored Procedures: Pros and Cons
**Question:** What are the advantages and disadvantages of using Stored Procedures in an E-Commerce database?

### Answer:
- **Pros:**
  1. Reduced Network Traffic: Executes multiple SQL statements server-side in one call.
  2. Security: Can grant execution permission without giving raw table access.
  3. Performance: Execution plans can be cached by the database engine.
- **Cons:**
  1. Portability: Syntax is vendor-specific (MySQL PL/SQL vs T-SQL).
  2. CPU Load: Shifts computational burden to the DB server, which is harder to scale horizontally than web servers.

---

## 14. Preventing SQL Injection & Ensuring Database Security
**Question:** How do you protect an E-Commerce database against SQL Injection attacks and unauthorized data access?

### Answer:
1. **Parameterized Queries / Prepared Statements:** Never concatenate raw user inputs into SQL strings.
   ```python
   # SAFE
   cursor.execute("SELECT * FROM customers WHERE e_mail = %s", (email_input,))
   ```
2. **Principle of Least Privilege (RBAC):** Web applications should connect using a DB user with restricted rights (e.g., `SELECT`, `INSERT`, `UPDATE` only; no `DROP` or `ALTER`).
3. **Encryption at Rest & in Transit:** Use TLS/SSL for database connections and AES-256 for sensitive data at rest.

---

## 15. Choice of Data Types: `TIMESTAMP` vs `DATETIME` & `BIGINT`
**Question:** Why use `TIMESTAMP` for dates and `BIGINT` for transaction IDs?

### Answer:
- **`TIMESTAMP` vs `DATETIME`:**
  - `TIMESTAMP` converts values from local timezone to UTC for storage and back to local timezone for display. Ideal for global order tracking across different time zones.
  - `DATETIME` stores exact dates without timezone conversion.
- **`BIGINT` for Transaction IDs:**
  - Standard `INT` max value is ~2.14 billion. A high-volume store can exceed this limit. `BIGINT` supports up to $9 \times 10^{18}$ records.

---

## 16. Database Partitioning (Horizontal vs. Vertical)
**Question:** How can partitioning be used to scale an E-Commerce database containing millions of orders?

### Answer:
1. **Horizontal Partitioning (Sharding):** Splitting rows across multiple tables/disks based on a partition key (e.g., partitioning `orders` by `RANGE (YEAR(order_date))` or `HASH(customer_id)`).
2. **Vertical Partitioning:** Splitting columns of a table (e.g., moving large text blobs like product descriptions to a separate `product_details` table to keep the main `products` table lean for memory caching).

---

## 17. Database Sharding Strategies by Customer ID vs. Region
**Question:** What are the trade-offs between sharding an e-commerce database by `customer_id` versus by `region`?

### Answer:
- **Sharding by `customer_id`:**
  - *Pros:* All data for a single customer (cart, orders, address) resides on one shard. Eliminates cross-shard JOINs for customer actions.
  - *Cons:* Hotspotting if power-buyers or bulk buyers generate disproportionate traffic.
- **Sharding by `region` (e.g., North, South, East, West):**
  - *Pros:* Data resides close to physical users (low latency).
  - *Cons:* Cross-region reporting and inventory lookup require complex cross-shard queries.

---

## 18. Database Views vs. Materialized Views
**Question:** How do Database Views benefit e-commerce analytics and reporting?

### Answer:
- **Standard View (`CREATE VIEW`):** A saved virtual query. Computes data on-the-fly whenever queried. Keeps application code clean and provides column-level security.
- **Materialized View:** Stores the actual query result on disk and refreshes periodically. Essential for expensive aggregate reports (e.g., monthly category sales summary over millions of historical rows).

---

## 19. Disaster Recovery & Point-in-Time Recovery (PITR)
**Question:** How does an e-commerce platform prepare for database corruption or hardware failure?

### Answer:
1. **Full Backups:** Scheduled daily or weekly full database snapshots.
2. **Transaction Logs (Binary Logs / WAL):** Real-time recording of every SQL change.
3. **Point-in-Time Recovery (PITR):** Restores a full backup and replays binary logs up to the exact minute before a corruption occurred.
4. **Multi-Region Read Replicas:** Automatic failover to a replica in another availability zone.

---

## 20. OLTP vs. OLAP Database Architecture
**Question:** Why shouldn't heavy analytical reporting queries be run on the production `e_commerce` operational database?

### Answer:
- **OLTP (Online Transaction Processing):** Optimized for fast, frequent read/write single-row operations (checkout, inventory updates). Optimized using Row-oriented relational DBs.
- **OLAP (Online Analytical Processing):** Optimized for complex multi-million row aggregate queries (BigQuery, Snowflake, Redshift). Optimized using Columnar storage.
- **Solution:** Extract, Transform, Load (ETL / ELT) data from OLTP database into an OLAP Data Warehouse for reporting without degrading production checkout speeds.

---

## 21. Payment Gateway Webhooks & Transaction Idempotency
**Question:** How do you prevent double-charging or duplicate payment creation when payment webhooks retry failed network calls?

### Answer:
- **Idempotency Key / Transaction ID:**
  - Payments table includes `transaction_id BIGINT UNIQUE`.
  - When a payment webhook arrives, the handler attempts to insert or lookup `transaction_id`.
  - If the `transaction_id` already exists in the database, the server returns HTTP 200 OK without processing the charge a second time.

---

## 22. Foreign Key Deletion Strategies
**Question:** Explain `ON DELETE RESTRICT`, `ON DELETE CASCADE`, and `ON DELETE SET NULL` in an E-Commerce system.

### Answer:
- **`ON DELETE RESTRICT` (Default):** Prevents deletion of a parent row if child rows exist (e.g., prevents deleting a `category` if `products` still reference it). Recommended for safety.
- **`ON DELETE CASCADE`:** Automatically deletes child rows when parent is deleted (e.g., deleting a `cart` deletes its `cart_items`).
- **`ON DELETE SET NULL`:** Sets child foreign key to `NULL` when parent is deleted (e.g., deleting an address sets `orders.address_id` to `NULL` if permitted).

---

## 23. Handling Deadlocks in E-Commerce Transactions
**Question:** What causes database deadlocks during high-traffic sales, and how can they be mitigated?

### Answer:
- **Cause:** Transaction A locks Product 1 then requests Product 2. Simultaneously, Transaction B locks Product 2 then requests Product 1. Both wait indefinitely.
- **Mitigation:**
  1. Access tables and rows in a deterministic order across all application services (e.g., always lock product IDs in ascending numerical order).
  2. Keep transactions short and fast.
  3. Set a lock timeout (`innodb_lock_wait_timeout`) and implement retry loops in application code.

---

## 24. Scaling Read Operations with Read Replicas
**Question:** How do Read Replicas scale an e-commerce platform during major promotional events?

### Answer:
- E-Commerce traffic is typically **80-90% Reads** (browsing products, searching categories) and **10-20% Writes** (placing orders, adding to cart).
- **Primary Database (Master):** Handles all write transactions (`INSERT`, `UPDATE`, `DELETE`).
- **Read Replicas (Slaves):** Asynchronously replicate data from Primary and handle all read-only traffic (`SELECT`).

---

## 25. Database Schema Version Control & Migration Tools
**Question:** How do modern engineering teams safely apply schema changes (like adding columns) to a live production database?

### Answer:
- Use automated migration tools like **Flyway**, **Liquibase**, or **Prisma Migrations**.
- Schema changes are written as versioned SQL scripts (`V1__init.sql`, `V2__add_discount_code.sql`).
- Migrations run automatically during CI/CD deployment pipelines, tracking applied versions in a `schema_migrations` table.

---

## 26. Optimistic Locking vs. Pessimistic Locking
**Question:** Compare Optimistic Locking and Pessimistic Locking for inventory management.

### Answer:
- **Pessimistic Locking:** Assumes conflict will occur. Uses explicit database row locks (`SELECT ... FOR UPDATE`).
  - *Best for:* High-contention items with low stock remaining.
- **Optimistic Locking:** Assumes conflicts are rare. Adds a `version` column to the table.
  ```sql
  UPDATE inventory 
  SET quantity = quantity - 1, version = version + 1 
  WHERE product_id = 1 AND version = 5;
  ```
  - *Best for:* Low-contention, high-throughput web applications.

---

## 27. Audit Logging & Change Data Capture (CDC)
**Question:** How do you track administrative changes (e.g., price modifications or order status overrides) for compliance?

### Answer:
- **Audit Tables:** Maintain separate shadow tables (e.g., `products_audit`) updated via triggers recording `old_value`, `new_value`, `changed_by`, and `changed_at`.
- **Change Data Capture (CDC):** Tools like **Debezium** stream database transaction logs to Kafka/Cloud Storage for real-time security auditing without affecting DB performance.

---

## 28. Automated Garbage Collection for Abandoned Carts
**Question:** How should an e-commerce system clean up millions of abandoned carts without blocking active users?

### Answer:
- Running a large `DELETE FROM cart WHERE updated_at < ...` on production locks tables and causes latency spikes.
- **Solution:** Batch deletion jobs executing off-peak:
  ```sql
  DELETE FROM cart 
  WHERE updated_at < DATE_SUB(NOW(), INTERVAL 30 DAY) 
  LIMIT 1000;
  ```
  Run in a loop with small pauses (`SLEEP(1)`).

---

## 29. Compliance Standards: GDPR & PCI-DSS in Database Design
**Question:** What database guidelines must be followed for customer privacy and credit card security?

### Answer:
1. **PCI-DSS Compliance:** Never store raw Credit Card numbers, CVV codes, or PINs in the database. Outsource payment processing to gateways (Stripe, Razorpay) and store only tokenized `transaction_id` references.
2. **GDPR Compliance:** Support "Right to be Forgotten" by anonymizing customer PII (`first_name = 'DELETED'`, `e_mail = 'anon_123@deleted.com'`) while retaining anonymized order financial values for tax auditing.

---

## 30. Transforming Relational E-Commerce Schema to Star Schema
**Question:** How do you transform this transactional schema into a Star Schema for Data Analytics?

### Answer:
In Data Warehousing:
- **Fact Table:** `fact_sales` (contains numerical metrics: `order_id`, `quantity`, `price`, `discount_amount`, `total_amount`, foreign keys to dimension tables).
- **Dimension Tables:** 
  - `dim_customers` (customer demographic attributes)
  - `dim_products` (product names, category hierarchies)
  - `dim_date` (year, quarter, month, day, day of week)
  - `dim_location` (city, state, country, pincode)
- *Benefit:* Enables super-fast aggregation and OLAP cube slicing/dicing.
