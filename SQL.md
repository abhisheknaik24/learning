# SQL Interview Questions and Answers

> Examples use broadly familiar SQL. Syntax and behavior vary across SQL Server, PostgreSQL, MySQL, Oracle, and SQLite; check the target dialect, especially for pagination, identity columns, date functions, and procedural features.

## Contents

1. [Relational fundamentals](#1-relational-fundamentals)
2. [Queries and joins](#2-queries-and-joins)
3. [Aggregation and subqueries](#3-aggregation-and-subqueries)
4. [Schema design and constraints](#4-schema-design-and-constraints)
5. [Transactions and concurrency](#5-transactions-and-concurrency)
6. [Indexes and performance](#6-indexes-and-performance)

## 1. Relational fundamentals

### 1. What is SQL?

SQL (Structured Query Language) is a language for defining, querying, and manipulating data in relational database systems. Its common categories include data definition (DDL), data manipulation (DML), access control (DCL), and transaction control (TCL). Exact syntax and features vary by database.

### 2. What is a relational database?

A relational database organizes data into relations, commonly represented as tables of rows and columns, and uses keys and constraints to describe relationships and enforce integrity. Relational systems also provide query operations, transactions, and often indexes and views.

### 3. What is a primary key?

A primary key uniquely identifies each row in a table. It must be unique and non-null under standard relational semantics. A table has one primary-key constraint, which may consist of one or multiple columns. Choose keys that are stable and appropriately sized for the use case.

### 4. What is a foreign key?

A foreign key constrains values in one table to reference a candidate/primary key in another table (or, where supported, the same table). It enforces referential integrity. Delete/update actions such as cascade or restrict should be chosen intentionally to avoid accidental data loss.

### 5. What are candidate, alternate, and composite keys?

A candidate key is a minimal set of columns that uniquely identifies a row. A primary key is the candidate key selected as the table's primary identifier; other candidate keys are alternate keys and can be enforced with unique constraints. A composite key uses more than one column.

### 6. What is `NULL`?

`NULL` represents missing or unknown information; it is not equal to zero, an empty string, or another `NULL`. Comparisons with `NULL` typically evaluate to `UNKNOWN`, so use `IS NULL` or `IS NOT NULL`. SQL's three-valued logic affects `WHERE`, joins, and constraints.

### 7. What is the difference between `WHERE` and `HAVING`?

`WHERE` filters input rows before grouping and aggregation. `HAVING` filters groups after aggregation. A predicate that does not depend on an aggregate usually belongs in `WHERE`, which can reduce rows earlier.

### 8. What is the difference between `DELETE`, `TRUNCATE`, and `DROP`?

`DELETE` removes selected rows and can usually use a `WHERE` clause. `TRUNCATE` removes all rows using a database-specific operation, with different logging, identity, trigger, and transactional behavior by dialect. `DROP` removes the table object itself. Permissions and rollback behavior vary by database; never assume these commands are interchangeable.

## 2. Queries and joins

### 9. What does `SELECT` do?

`SELECT` retrieves expressions from one or more sources. Prefer an explicit column list over `SELECT *` in stable application queries, because it documents required data and reduces accidental coupling to schema growth.

### 10. What is an `INNER JOIN`?

An inner join returns rows with matching join conditions on both sides:

```sql
SELECT c.CustomerId, o.OrderId
FROM Customers AS c
INNER JOIN Orders AS o ON o.CustomerId = c.CustomerId;
```

Rows without a match are omitted.

### 11. What is a `LEFT JOIN`?

A left outer join returns every row from the left input and matching rows from the right; unmatched right-side columns are `NULL`:

```sql
SELECT c.CustomerId, o.OrderId
FROM Customers AS c
LEFT JOIN Orders AS o ON o.CustomerId = c.CustomerId;
```

Predicates on the right table placed in `WHERE` can remove unmatched rows and effectively turn the result into an inner join. Put match restrictions in `ON` when unmatched left rows must remain.

### 12. What are `RIGHT JOIN` and `FULL OUTER JOIN`?

A right join preserves every row from the right input and matches from the left. A full outer join preserves unmatched rows from both sides, filling missing columns with `NULL`. Not all database engines support every join type; equivalent rewrites may be needed.

### 13. What is a cross join?

A cross join returns the Cartesian product of two inputs: every left row paired with every right row. Its row count is the product of the input row counts, so use it deliberately.

### 14. What is a self join?

A self join joins a table to itself, usually with aliases, to compare rows or represent hierarchies such as employees and managers. The join condition must distinguish the roles of the two table instances.

### 15. What is the difference between `UNION` and `UNION ALL`?

`UNION` combines compatible result sets and removes duplicate rows. `UNION ALL` keeps duplicates and commonly avoids deduplication work. Each query must return compatible numbers and types of columns in the same order.

### 16. What is the logical order of query processing?

A useful conceptual order is `FROM`/`JOIN`, `WHERE`, `GROUP BY`, `HAVING`, `SELECT`, `DISTINCT`, `ORDER BY`, then row limiting (`OFFSET`/`FETCH`, `LIMIT`, or dialect equivalent). The optimizer may execute a different physical plan while preserving required semantics. This explains why a `SELECT` alias may not be visible in `WHERE` in many dialects.

### 17. Why should query results use `ORDER BY`?

Without `ORDER BY`, row order is not guaranteed, even if repeated executions appear ordered. Add a deterministic ordering, including a unique tie-breaker when pagination must be stable.

### 18. How does pagination work?

Common forms include `LIMIT`/`OFFSET` or `OFFSET ... FETCH`, depending on dialect. Large offsets may be expensive and concurrent changes can shift pages. Keyset pagination uses the last-seen ordering key to fetch the next page and can perform better for deep navigation, provided ordering keys are stable and indexed appropriately.

## 3. Aggregation and subqueries

### 19. What does `GROUP BY` do?

`GROUP BY` partitions rows into groups with equal grouping expressions so aggregate functions can calculate one result per group. In standard SQL, selected non-aggregate expressions generally need to be included in the grouping expressions.

### 20. What are aggregate functions?

Common aggregates include `COUNT`, `SUM`, `AVG`, `MIN`, and `MAX`. Most aggregates ignore `NULL` values, while `COUNT(*)` counts rows and `COUNT(column)` counts non-null values. Empty-input behavior varies by aggregate and dialect.

### 21. What is a subquery?

A subquery is a query nested inside another query. It can appear in `FROM`, `WHERE`, `SELECT`, or other contexts. The optimizer may transform it; do not assume a subquery always executes once per outer row.

### 22. What is a correlated subquery?

A correlated subquery references columns from the outer query and is logically evaluated in relation to each outer row. Optimizers can often decorrelate it, but performance depends on the query and indexes. Compare its plan with a join or window-function alternative.

### 23. What is the difference between `EXISTS` and `IN`?

`EXISTS` tests whether a subquery returns any row. `IN` tests membership in a set of values. They can often express equivalent logic, but `NULL` behavior—especially `NOT IN` when the subquery contains `NULL`—can surprise. `NOT EXISTS` is often safer for anti-join logic.

### 24. What is a common table expression (CTE)?

A CTE is a named query expression scoped to a statement, introduced with `WITH`. It can improve readability and support recursive queries in many systems. A CTE is not necessarily materialized; execution and optimization behavior are database-specific.

### 25. What is a window function?

A window function computes a value across related rows while preserving individual rows, using an `OVER` clause with optional partitioning and ordering. Examples include `ROW_NUMBER`, `RANK`, and running totals. Window functions are useful for ranking and analytics without collapsing rows as `GROUP BY` does.

```sql
SELECT EmployeeId, DepartmentId, Salary,
       ROW_NUMBER() OVER (
           PARTITION BY DepartmentId
           ORDER BY Salary DESC, EmployeeId
       ) AS position
FROM Employees;
```

### 26. What is the difference between `ROW_NUMBER`, `RANK`, and `DENSE_RANK`?

`ROW_NUMBER` assigns a unique sequential number, including for ties. `RANK` gives tied rows the same rank and leaves gaps after ties. `DENSE_RANK` gives tied rows the same rank without gaps. The window `ORDER BY` defines which values tie.

### 27. What is `CASE`?

`CASE` is a conditional expression that returns a value based on conditions. It can be used in projections, sorting, and aggregation. Ensure result branches have compatible types and define a fallback when appropriate.

### 28. How do you find duplicate values?

Group by the candidate duplicate columns and filter groups with a count greater than one:

```sql
SELECT Email, COUNT(*) AS occurrences
FROM Users
GROUP BY Email
HAVING COUNT(*) > 1;
```

Consider whether `NULL` values and collation/case sensitivity count as duplicates under the business rule.

## 4. Schema design and constraints

### 29. What is normalization?

Normalization organizes relational data to reduce update anomalies and unnecessary duplication. First, second, and third normal forms describe progressively stronger structural properties. Normalization supports integrity; deliberate denormalization may be useful for measured read performance or reporting, with consistency costs handled explicitly.

### 30. What is a unique constraint?

A unique constraint enforces uniqueness for a column or column combination. Treatment of `NULL` values differs by database. A unique index may implement the constraint, but database terminology and features differ. Prefer database enforcement over application-only “check then insert” logic, which can race.

### 31. What are `CHECK`, `NOT NULL`, and `DEFAULT` constraints?

`NOT NULL` disallows missing values in a column. `CHECK` constrains values using a predicate. `DEFAULT` supplies a value when an insert omits a column (not necessarily when an explicit `NULL` is supplied). Constraint expression support and enforcement details vary by database/version.

### 32. What is a view?

A view is a named query presented like a table. A standard view stores a query definition, not necessarily its results. A materialized or indexed view stores results or indexes them according to database-specific rules. Views can simplify access but are not automatically a security boundary.

### 33. What is a stored procedure?

A stored procedure is a named database-side program that can accept parameters and perform queries or updates. It can centralize logic and reduce round trips, but may increase coupling to a database dialect and make deployment/versioning more complex. Parameterize calls and grant least privilege.

### 34. What is a trigger?

A trigger is database logic invoked by defined data or schema events. Triggers can enforce or record behavior close to the data, but hidden side effects complicate debugging, performance, and transaction reasoning. Use them intentionally and document their effects.

### 35. What is a surrogate key versus a natural key?

A surrogate key is an artificial identifier, such as a generated integer or UUID. A natural key comes from meaningful domain data, such as a regulated code. Surrogate keys can remain stable when business attributes change; natural keys can express real uniqueness. Many systems use a surrogate primary key plus a unique constraint for the natural key.

## 5. Transactions and concurrency

### 36. What are ACID properties?

- **Atomicity:** A transaction commits all its operations or none.
- **Consistency:** Constraints and invariants hold before and after the transaction.
- **Isolation:** Concurrent transactions interact according to the isolation level.
- **Durability:** Committed changes persist according to the system's guarantees.

Actual guarantees depend on database configuration, storage, and failure model.

### 37. What is a transaction?

A transaction groups database operations into a logical unit that can commit or roll back. Transactions protect consistency across related writes, but long-running transactions can hold locks, increase contention, and interfere with maintenance. Keep them as short as practical.

### 38. What are isolation levels?

Isolation levels define which effects of concurrent transactions may be visible. Common names include Read Uncommitted, Read Committed, Repeatable Read, and Serializable; Snapshot/MVCC options are database-specific. The same level name may have implementation differences across engines.

### 39. What are dirty reads, non-repeatable reads, and phantom reads?

- A **dirty read** observes data written by a transaction that has not committed.
- A **non-repeatable read** sees a row change between reads in one transaction.
- A **phantom read** sees rows appear or disappear when repeating a range query.

Isolation level and database implementation determine whether these anomalies can occur.

### 40. What is a deadlock?

A deadlock occurs when transactions wait on each other's resources in a cycle. The database typically detects a victim and aborts it. Reduce risk by accessing resources in a consistent order, keeping transactions short, indexing appropriately, and retrying selected deadlock victims safely.

### 41. What is optimistic versus pessimistic concurrency?

Optimistic concurrency detects conflicts at write time, often with a version token. Pessimistic concurrency locks resources to restrict conflicting operations before they occur. Optimistic approaches work well when conflicts are uncommon; pessimistic locking may be appropriate for specific contention-sensitive operations but can reduce throughput.

## 6. Indexes and performance

### 42. What is an index?

An index is a data structure that helps the database locate rows without scanning the entire table. It can accelerate reads but consumes storage and adds work to inserts, updates, and deletes. Index choice should follow actual query patterns and execution plans.

### 43. What is a clustered index?

A clustered index defines or strongly influences the physical organization of table rows in systems that support that concept. A table generally has at most one clustered organization. The exact meaning differs by engine; PostgreSQL's ordinary indexes, for example, do not continuously maintain table order in the same way as SQL Server clustered indexes.

### 44. What is a composite index?

A composite index contains multiple columns. Column order matters: an index on `(A, B)` is generally useful for predicates beginning with `A`, and may not efficiently support a predicate only on `B`. Choose key order using selectivity, filtering, joining, sorting, and workload patterns; verify with a plan.

### 45. What is a covering index?

A covering index contains all columns needed by a query, either as key columns or included/non-key columns where the engine supports them. It can avoid additional table lookups, but wider indexes require more space and write maintenance. Do not include every output column without measurement.

### 46. What is an execution plan?

An execution plan describes how the database optimizer intends to execute a query, including scans, seeks, joins, sorts, and estimates. An actual plan may include measured row counts and runtime information. Use plans to investigate bottlenecks, stale estimates, missing indexes, and unexpected row expansion.

### 47. Why might an index not be used?

The optimizer may estimate a scan is cheaper, the predicate may not match the index's leading columns, implicit conversions or expressions may prevent a seek, statistics may be stale, or the result may be a large fraction of the table. Index usage depends on data distribution, query shape, and engine; inspect the plan instead of assuming.

### 48. What is a sargable predicate?

A sargable predicate can use an index efficiently for search arguments. Applying a function or conversion to an indexed column can prevent efficient index use in some engines, e.g. `WHERE YEAR(CreatedAt) = 2025`. A range predicate such as `CreatedAt >= ... AND CreatedAt < ...` may be more index-friendly, subject to datatype and timezone semantics.

### 49. What is SQL injection and how is it prevented?

SQL injection occurs when untrusted input changes the meaning of a dynamically constructed SQL command. Use parameterized queries or prepared statements so input is treated as data. Identifiers and sort directions generally cannot be parameterized as values; select them from strict allowlists. Escaping alone is fragile and is not a substitute for parameterization.

### 50. How do you investigate a slow query?

Reproduce with representative data, capture the actual execution plan and timings, inspect row estimates versus actual rows, I/O, sorts, scans, joins, and parameter values, then test targeted changes. Check indexes, statistics, query predicates, result size, and blocking. Avoid adding indexes or hints blindly; retest reads and write costs.
