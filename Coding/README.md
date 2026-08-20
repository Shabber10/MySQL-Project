# SQL Concept-Wise Coding Questions & Solutions

This directory contains a complete, topic-by-topic SQL reference guide with practical business questions, expected outputs, and executable SQL queries based on the **E-Commerce Database Schema**.

---

## 📚 Topic-Wise Table of Contents

| # | Category | Topic / Concept | Markdown File | Questions Included |
|---|---|---|---|---|
| **01** | **DDL** | Data Definition Language | [01_DDL_Data_Definition_Language.md](./01_DDL_Data_Definition_Language.md) | `CREATE TABLE`, `ALTER TABLE`, `TRUNCATE`, `DROP` |
| **02** | **DML** | Data Manipulation Language | [02_DML_Data_Manipulation_Language.md](./02_DML_Data_Manipulation_Language.md) | `INSERT` (Bulk/Upsert), `UPDATE` (Join update), `DELETE` |
| **03** | **DQL** | Data Query Language & Filtering | [03_DQL_Basic_Queries_and_Filtering.md](./03_DQL_Basic_Queries_and_Filtering.md) | `SELECT`, `WHERE`, `LIKE`, `ORDER BY`, `LIMIT`, `OFFSET`, `DISTINCT` |
| **04** | **DCL & TCL** | Permissions & Transactions | [04_DCL_and_TCL_Transactions.md](./04_DCL_and_TCL_Transactions.md) | `GRANT`, `REVOKE`, `START TRANSACTION`, `COMMIT`, `ROLLBACK`, `SAVEPOINT` |
| **05** | **Aggregation** | Aggregation & Grouping | [05_Aggregation_and_Grouping.md](./05_Aggregation_and_Grouping.md) | `COUNT`, `SUM`, `AVG`, `MIN`, `MAX`, `GROUP BY`, `HAVING`, `WITH ROLLUP` |
| **06** | **Joins** | All Types of Joins | [06_All_Types_of_Joins.md](./06_All_Types_of_Joins.md) | `INNER JOIN`, `LEFT JOIN`, `RIGHT JOIN`, `FULL OUTER JOIN`, `CROSS JOIN`, `SELF JOIN` |
| **07** | **Subqueries** | Subqueries & Nested Queries | [07_Subqueries_and_Nested_Queries.md](./07_Subqueries_and_Nested_Queries.md) | Scalar Subqueries, `IN`, `ALL`, `ANY`, Correlated Subqueries, `EXISTS`, `NOT EXISTS` |
| **08** | **Views** | SQL Views | [08_Views.md](./08_Views.md) | Simple Views, Complex Joined Views, `WITH CHECK OPTION`, `ALTER`/`DROP VIEW` |
| **09** | **Window Functions** | Analytical & Window Functions | [09_Window_Functions.md](./09_Window_Functions.md) | `ROW_NUMBER`, `RANK`, `DENSE_RANK`, `NTILE`, `LEAD`, `LAG`, Cumulative Running Sums |
| **10** | **CTEs** | Common Table Expressions | [10_Common_Table_Expressions_CTE.md](./10_Common_Table_Expressions_CTE.md) | Standard CTEs, Chained/Multiple CTEs, Recursive CTEs |
| **11** | **Routines** | Stored Procedures & Functions | [11_Stored_Procedures_and_Functions.md](./11_Stored_Procedures_and_Functions.md) | `PROCEDURE` with IN/OUT, Transactional logic, Deterministic Scalar `FUNCTION` |
| **12** | **Triggers** | Database Triggers | [12_Triggers.md](./12_Triggers.md) | `BEFORE INSERT`, `BEFORE UPDATE` validation, `AFTER INSERT` sync, `AFTER UPDATE` audit log |
| **13** | **Indexes & Rules** | Indexes & Constraints | [13_Indexes_and_Constraints.md](./13_Indexes_and_Constraints.md) | `CHECK`, `ON DELETE CASCADE`, Composite Indexes, `EXPLAIN` execution plans |
| **14** | **Set Operations** | Set Operations | [14_Set_Operations.md](./14_Set_Operations.md) | `UNION`, `UNION ALL`, `INTERSECT`, `EXCEPT` / `MINUS` |
| **15** | **Functions** | Built-in Functions & Conditionals | [15_Built_in_Functions_and_Conditionals.md](./15_Built_in_Functions_and_Conditionals.md) | String Functions, Date/Time arithmetic, `CASE WHEN`, `COALESCE`, `IFNULL` |

---

## 🚀 Structure per Concept
Every topic follows a consistent, high-yield format:
1. **Scenario & Business Requirement:** Real-world problem using the e-commerce schema.
2. **Expected Output:** Tabular markdown or server response preview.
3. **SQL Answer Query:** Clean, executable, production-grade SQL code.
