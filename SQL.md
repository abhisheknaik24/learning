# SQL Interview Questions and Answers

> Examples use broadly familiar SQL. Syntax and behavior vary across SQL Server, PostgreSQL, MySQL, Oracle, and SQLite; check the target dialect, especially for pagination, identity columns, date functions, and procedural features.

## Contents

1. [Relational fundamentals](#1-relational-fundamentals)
2. [Queries and joins](#2-queries-and-joins)
3. [Aggregation and subqueries](#3-aggregation-and-subqueries)
4. [Schema design and constraints](#4-schema-design-and-constraints)
5. [Transactions and concurrency](#5-transactions-and-concurrency)
6. [Indexes and performance](#6-indexes-and-performance)
7. [Additional questions: syntax, types, and expressions](#7-additional-questions-51-100-sql-syntax-types-and-expressions)
8. [Additional questions: filtering and query composition](#8-additional-questions-101-150-filtering-ordering-and-query-composition)
9. [Additional questions: aggregates and window functions](#9-additional-questions-151-200-aggregates-ctes-and-window-functions)
10. [Additional questions: data modeling](#10-additional-questions-201-250-data-modeling-and-normalization)
11. [Additional questions: constraints and indexes](#11-additional-questions-251-300-constraints-indexes-and-objects)
12. [Additional questions: transactions and concurrency](#12-additional-questions-301-350-transactions-and-concurrency)
13. [Additional questions: data operations](#13-additional-questions-351-400-sql-programming-and-data-operations)
14. [Additional questions: performance](#14-additional-questions-401-450-query-plans-and-performance)
15. [Additional questions: security and advanced SQL](#15-additional-questions-451-500-security-reliability-and-advanced-sql)

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

## 7. Additional Questions (51-100): SQL Syntax, Types, and Expressions

### 51. What is the difference between DDL and DML?

DDL defines database objects, such as `CREATE TABLE`; DML reads or changes row data, such as `SELECT`, `INSERT`, `UPDATE`, and `DELETE`.

### 52. What is DCL?

Data Control Language includes permission statements such as `GRANT` and `REVOKE`; syntax and available privileges vary by database.

### 53. What is TCL?

Transaction Control Language includes commands such as `COMMIT`, `ROLLBACK`, and sometimes `SAVEPOINT`.

### 54. What is a SQL literal?

A literal is a constant written directly in a statement, such as `42`, `'hello'`, or a dialect-specific date literal.

### 55. What is a SQL identifier?

An identifier names an object, such as a table, column, schema, or index. Quoting syntax is dialect-specific; parameters generally cannot substitute identifiers.

### 56. What is a schema?

A schema is a logical namespace for database objects and can also be a unit for ownership and permissions, depending on the database.

### 57. What is a catalog?

A catalog is a database's metadata namespace or collection of schemas and object definitions; exact terminology differs by product.

### 58. What is a character set?

A character set defines representable characters and encoding; Unicode-capable encodings avoid loss for international text.

### 59. What is a collation?

A collation defines text comparison and sorting rules, including case, accents, and locale. It can affect equality and index behavior.

### 60. What is the difference between `CHAR` and `VARCHAR`?

`CHAR(n)` is fixed-width in standard usage; `VARCHAR(n)` stores variable-length text up to a limit. Padding and storage details vary by engine.

### 61. What is `NCHAR` or `NVARCHAR`?

These are Unicode-oriented character types in some databases, notably SQL Server. Other engines use Unicode-capable `CHAR`/`VARCHAR` encodings instead.

### 62. What is a decimal type?

`DECIMAL(p,s)` or `NUMERIC(p,s)` represents exact fixed-precision decimal values, suitable for many monetary calculations.

### 63. Why avoid floating point for currency?

Binary floating-point cannot exactly represent many decimal fractions; use an appropriate exact decimal representation and defined rounding rules.

### 64. What are precision and scale?

Precision is the maximum total digits; scale is the number of digits after the decimal point for a fixed-precision decimal.

### 65. What is integer overflow?

It occurs when an arithmetic result exceeds the type's representable range; error, wraparound, or coercion behavior depends on the engine and expression.

### 66. What is a date/time type?

It stores temporal values with varying precision and timezone semantics; select a type based on whether the value represents a date, local time, or instant.

### 67. What is a timestamp?

The term is dialect-dependent: it may mean a date-time value or a row-version token. Never assume it always denotes UTC time.

### 68. What is a time zone-aware timestamp?

It represents a temporal value with offset/zone semantics as defined by the database; storage normalization behavior varies by type and engine.

### 69. What is a binary type?

It stores bytes rather than text, used for binary data, hashes, or opaque values; large files are often better kept in object storage.

### 70. What is a Boolean type?

It represents true/false where supported; some databases represent Boolean-like values using numeric or character types.

### 71. What is type precedence?

It determines which type is chosen when expressions combine different types; implicit conversions can alter results or prevent index use.

### 72. What is an implicit conversion?

It is an automatic conversion between compatible types; it may lose precision or affect query plans, so use deliberate matching types.

### 73. What is an explicit cast?

`CAST(expression AS type)` requests conversion to a target type; invalid or lossy conversions may error or alter values.

### 74. What is `CONVERT`?

It is a dialect-specific conversion function, notably in SQL Server, sometimes supporting style formats beyond standard `CAST`.

### 75. What is `COALESCE`?

It returns the first non-`NULL` expression; arguments must be compatible types and may be evaluated according to database rules.

### 76. What is `NULLIF`?

It returns `NULL` when two expressions compare equal, otherwise returns the first expression; it can guard division by zero.

### 77. What is `CASE` expression evaluation?

It selects a result from matching conditions; result branches need compatible types. Do not rely on nonportable assumptions about evaluation short-circuiting.

### 78. What is `IIF`?

It is a conditional function available in some dialects; `CASE` is more portable SQL.

### 79. What is `COALESCE` versus `ISNULL`?

`COALESCE` is standard SQL with multiple arguments; `ISNULL` is dialect-specific and can differ in type inference and evaluation.

### 80. What is a concatenation operator?

It combines strings; syntax varies (`||`, `+`, or functions), and `NULL` concatenation behavior may vary.

### 81. What is `LIKE`?

It performs pattern matching using wildcards such as `%` for a sequence and `_` for one character; escape syntax varies.

### 82. What does `BETWEEN` mean?

It tests an inclusive range in standard usage; for timestamps, a half-open range is often safer than an inclusive end boundary.

### 83. What are comparison predicates?

They compare values with operators such as `=`, `<>`, `<`, `<=`, `>`, and `>=`, subject to type and `NULL` semantics.

### 84. What are logical operators?

`AND`, `OR`, and `NOT` combine predicates using SQL's three-valued logic, where a predicate may be `UNKNOWN`.

### 85. Why use parentheses in complex predicates?

They make intended precedence explicit; `AND` generally binds more tightly than `OR`.

### 86. What is a bitwise operator?

It operates on the bits of numeric values; operator syntax and supported types are database-specific.

### 87. What is modulo?

The modulo operation returns a remainder; syntax is commonly `%` or `MOD`, depending on the dialect.

### 88. What is integer division?

It divides integer operands with dialect-defined truncation or numeric conversion; cast operands when fractional output is required.

### 89. What does `ROUND` do?

It rounds a numeric value to a requested precision; midpoint and negative-precision behavior should be verified for the target engine.

### 90. What does `TRIM` do?

It removes selected leading/trailing characters from text; exact argument syntax depends on SQL dialect.

### 91. What does `SUBSTRING` do?

It returns a portion of text or binary data; indexing conventions and argument forms vary across dialects.

### 92. What does `LENGTH` or `LEN` count?

The function returns a length, but may count characters, bytes, or exclude trailing spaces depending on engine and function.

### 93. What does `UPPER` do?

It converts text to uppercase according to database collation and locale behavior.

### 94. What does `EXTRACT` do?

It retrieves a date/time component such as year or month in dialects that support the standard form.

### 95. What is a deterministic function?

It returns the same result for the same inputs under defined conditions; optimizer and index eligibility rules are engine-specific.

### 96. What is a nondeterministic function?

Its result can vary despite identical explicit inputs, for example current time or random values.

### 97. What is a scalar expression?

It produces one value per evaluation context, such as a column reference, literal, arithmetic expression, or scalar function.

### 98. What is a predicate?

A predicate is a condition that evaluates to true, false, or unknown in SQL contexts.

### 99. What is a parameterized query?

It binds values separately from SQL text, improving injection protection and often enabling plan reuse.

### 100. Why should parameter types match column types?

Mismatched types can trigger implicit conversions, incorrect comparisons, reduced index use, or precision problems.

## 8. Additional Questions (101-150): Filtering, Ordering, and Query Composition

### 101. What is `DISTINCT`?

It removes duplicate result rows based on the selected expressions; it may require sorting or hashing.

### 102. Is `DISTINCT` a substitute for a correct join?

No. It can hide row multiplication caused by an incorrect join instead of fixing the underlying logic.

### 103. What does `ORDER BY` support?

It orders results by expressions or selected aliases, with null ordering and collation details varying by dialect.

### 104. What is a stable sort?

A stable sort preserves prior relative order for tied keys, but SQL does not generally promise this unless all needed keys are specified.

### 105. How do you sort nulls first or last?

Use dialect-supported `NULLS FIRST/LAST` or an explicit `CASE` sort expression.

### 106. What is `OFFSET`?

It skips a specified number of result rows, usually after ordering; large skips may require processing many preceding rows.

### 107. What is `FETCH FIRST`?

It limits returned rows in a standard-style syntax supported by many, but not all, databases.

### 108. What is `TOP`?

It is SQL Server syntax for limiting returned rows; deterministic selection requires an appropriate `ORDER BY`.

### 109. What is `LIMIT`?

It limits output rows in engines such as PostgreSQL and MySQL; syntax and offset variants differ.

### 110. What is keyset pagination?

It selects rows after the last observed sort key, avoiding large offsets and often improving deep-page performance.

### 111. What makes keyset pagination deterministic?

The ordering must include a unique tie-breaker and the cursor must carry every ordered key value.

### 112. What is a search predicate?

It filters rows based on values, ranges, or patterns; matching types and suitable indexes often improve performance.

### 113. What is a prefix search?

It matches values beginning with a prefix, often indexable with `LIKE 'abc%'` under appropriate collation and plan conditions.

### 114. Why may a leading wildcard be slow?

`LIKE '%term'` often cannot seek a conventional B-tree index because the beginning of the value is unknown.

### 115. What is a full-text search?

It searches tokenized text using an engine-specific index and query language, supporting linguistic features beyond basic `LIKE`.

### 116. What is a range predicate?

It filters values between lower and upper bounds; half-open ranges avoid overlap for adjacent time intervals.

### 117. What is a half-open interval?

It includes the start and excludes the end (`>= start AND < end`), making adjacent time windows precise.

### 118. Why avoid `SELECT *`?

It fetches unnecessary columns, increases coupling, and can interfere with covering indexes and stable client mappings.

### 119. What is a column alias?

It gives an output expression a name, typically with `AS`; alias visibility in clauses depends on SQL rules and dialect.

### 120. What is a table alias?

It gives a table expression a short query-local name, improving clarity especially for joins.

### 121. What is a derived table?

It is a subquery in the `FROM` clause treated as a table expression and generally assigned an alias.

### 122. What is a lateral join?

It allows a `FROM` subquery to reference preceding sources; syntax includes `LATERAL` or `APPLY` depending on engine.

### 123. What is `CROSS APPLY`?

It is SQL Server syntax for evaluating a table-valued expression for each left-side row and retaining matching results.

### 124. What is `OUTER APPLY`?

It is SQL Server syntax that preserves left rows even if the applied expression returns no rows.

### 125. What is predicate pushdown?

It is optimizer movement of filters closer to data sources to reduce rows processed, subject to semantic constraints.

### 126. What is projection pushdown?

It limits columns read by lower plan operators to those needed by the query.

### 127. What is query composability?

A query expression can be embedded or combined with another while preserving its semantics, subject to ordering and limiting rules.

### 128. What is a semijoin?

It returns rows from one input that have a match in another, commonly expressed with `EXISTS`.

### 129. What is an antijoin?

It returns rows with no match, often expressed with `NOT EXISTS` or an outer join plus null test.

### 130. Why can `NOT IN` behave unexpectedly?

If the subquery contains `NULL`, SQL's unknown comparison can make the predicate reject all rows; `NOT EXISTS` often avoids this trap.

### 131. What is relational division?

It finds entities associated with every member of a required set, often expressed with nested `NOT EXISTS` or grouping/count logic.

### 132. What is a top-N-per-group query?

It selects the highest-ranked rows within each group, commonly using `ROW_NUMBER()` partitioned by the group key.

### 133. What is a deduplication query?

It identifies or removes duplicate logical rows using grouping or ranking, while preserving a deliberate survivor rule.

### 134. Why define a tie-breaker when deduplicating?

Without deterministic ordering, the chosen surviving row can vary between executions.

### 135. What is an anti-pattern in filtering dates?

Wrapping an indexed timestamp in a function can inhibit a seek; use a range with explicit boundaries where appropriate.

### 136. What is an optional filter?

It is a predicate applied only when a parameter is present; dynamic SQL or carefully designed predicates can implement it.

### 137. Why can `column = @p OR @p IS NULL` be slow?

It can complicate cardinality estimates and plan selection; evaluate alternatives using actual plans and data distributions.

### 138. What is dynamic SQL?

It is SQL text assembled at runtime; parameterize values and allowlist identifiers to avoid injection and plan instability.

### 139. What is query parameter sniffing?

Some engines compile a plan using initial parameter values, which may perform poorly for later values with different selectivity.

### 140. What is a row goal?

It is an optimizer objective to return a small number of rows quickly, potentially changing plan choices for limited queries.

### 141. What does `EXISTS` return?

It returns a Boolean-like truth result indicating whether the subquery contains at least one row.

### 142. Does `EXISTS` care which columns are selected?

Usually no; the existence of a row matters, so `SELECT 1` is a common convention.

### 143. What does `ANY` or `SOME` mean?

It compares a value against at least one value returned by a subquery; support and readability vary.

### 144. What does `ALL` mean in a comparison?

It requires the comparison to hold for every value returned by the subquery, with `NULL` semantics still applicable.

### 145. What is a quantified comparison?

It combines a comparison operator with `ANY`, `SOME`, or `ALL` over a subquery.

### 146. What is `INTERSECT`?

It returns rows common to both result sets, removing duplicates unless the dialect supports an `ALL` form.

### 147. What is `EXCEPT`?

It returns rows from the first result set absent from the second; some engines use `MINUS`.

### 148. What compatibility is required for set operations?

Inputs need the same number of columns with compatible types in corresponding positions.

### 149. How can you add a constant column to a query?

Select a literal with an alias, ensuring its type is appropriate for the combined result.

### 150. Why is query formatting important?

Consistent indentation and explicit joins make conditions, precedence, and review errors easier to see.

## 9. Additional Questions (151-200): Aggregates, CTEs, and Window Functions

### 151. What does `COUNT(*)` count?

It counts result rows, including rows whose columns contain `NULL`.

### 152. What does `COUNT(column)` count?

It counts rows where that expression is non-`NULL`.

### 153. What does `COUNT(DISTINCT column)` do?

It counts distinct non-`NULL` values in that expression; multi-column forms are dialect-specific.

### 154. How does `SUM` treat nulls?

It ignores null inputs; when no non-null input exists, result behavior is generally `NULL`, not zero.

### 155. How does `AVG` treat nulls?

It averages non-`NULL` values, so the denominator is the number of non-null inputs.

### 156. What is conditional aggregation?

It computes separate aggregates with conditional expressions such as `SUM(CASE WHEN condition THEN amount ELSE 0 END)`.

### 157. What is `FILTER` on an aggregate?

It restricts rows for one aggregate using syntax supported by some engines; `CASE` is a more portable alternative.

### 158. What is grouping by multiple columns?

It creates groups for each distinct combination of the listed values.

### 159. What is `ROLLUP`?

It produces hierarchical subtotals and a grand total for ordered grouping columns where supported.

### 160. What is `CUBE`?

It generates subtotals for combinations of grouping dimensions, potentially producing many rows.

### 161. What is `GROUPING SETS`?

It computes selected grouping combinations in one query, supported by several analytical databases.

### 162. How can subtotal rows be distinguished from null data?

Use grouping metadata functions such as `GROUPING()` where supported.

### 163. What is a window partition?

It divides rows into independent groups for a window calculation without collapsing them.

### 164. What does a window `ORDER BY` control?

It defines logical ordering within each partition for ranking and frame calculations.

### 165. What is a window frame?

It defines which rows around the current row participate in a window aggregate or value function.

### 166. What is `ROWS BETWEEN`?

It defines a frame by physical row offsets relative to the current row.

### 167. What is `RANGE BETWEEN`?

It defines a frame by ordering-value ranges; peer and datatype semantics differ from `ROWS`.

### 168. What is a running total?

It is a cumulative sum, commonly `SUM(value) OVER (PARTITION BY key ORDER BY date ROWS UNBOUNDED PRECEDING)`.

### 169. Why specify a window frame explicitly?

Default frames can include peers or differ across functions; an explicit frame documents intended cumulative or sliding behavior.

### 170. What is `LAG`?

It returns a value from a prior row in the window ordering without a self join.

### 171. What is `LEAD`?

It returns a value from a following row in the window ordering.

### 172. What are `FIRST_VALUE` and `LAST_VALUE`?

They return values from the first or last row of the window frame; frame boundaries matter, especially for `LAST_VALUE`.

### 173. What is `NTH_VALUE`?

It returns a value from the nth row in the current window frame where supported.

### 174. What is a percentile function?

It computes a percentile or quantile using engine-specific ordered-set or window syntax.

### 175. What is a moving average?

It averages a sliding frame, such as the current row and several preceding rows.

### 176. What is a cumulative distribution?

`CUME_DIST` calculates the relative position of a row including peers in an ordered partition.

### 177. What is `PERCENT_RANK`?

It reports a normalized rank position within a partition, with engine-defined edge behavior.

### 178. What is `NTILE`?

It divides ordered rows into a requested number of approximately equal buckets.

### 179. What is a recursive CTE?

It repeatedly evaluates an anchor query and recursive member to traverse hierarchical or iterative data.

### 180. What are the anchor and recursive members?

The anchor produces starting rows; the recursive member references prior CTE output to produce the next level.

### 181. What prevents infinite recursive CTE execution?

A correct termination predicate and, where available, recursion limits; malformed cycles can otherwise continue until engine limits.

### 182. How do you detect cycles in a hierarchy?

Track visited identifiers or use engine-specific cycle detection features and reject repeated path nodes.

### 183. What is a recursive traversal depth?

It is the number of recursive iterations or hierarchy levels reached; define and enforce a reasonable bound.

### 184. What is a materialized CTE?

It is a CTE whose intermediate result is physically stored or forced to be evaluated separately; syntax and optimizer behavior are dialect-specific.

### 185. Is a CTE always materialized?

No. Many optimizers inline or transform CTEs; verify behavior for the target database and query.

### 186. What is a CTE's scope?

It normally exists for one following statement only.

### 187. Can one CTE reference another?

Yes, later CTEs in a `WITH` clause can often reference earlier ones; recursive and forward-reference rules vary.

### 188. What is a windowed aggregate?

It applies an aggregate with `OVER`, returning a result per row instead of collapsing groups.

### 189. How do you select the latest row per customer?

Rank rows with `ROW_NUMBER()` partitioned by customer and ordered by timestamp plus a unique tie-breaker, then filter rank one.

### 190. How do you compare a row with its previous row?

Use `LAG` over a deterministic order and calculate the desired difference.

### 191. How do you find gaps in a sequence?

Compare each value to `LAG` or use interval/anti-join logic, selecting a method suited to the sequence definition.

### 192. How do you detect overlapping intervals?

Compare each interval's start and end against other intervals using the correct inclusive/exclusive boundary rule.

### 193. How do you compute a median?

Use a percentile-continuous/discrete function where available, or rank and select the central row or pair of rows.

### 194. How do you pivot rows into columns?

Use conditional aggregation or a dialect-specific `PIVOT`; dynamic pivot columns require carefully controlled dynamic SQL.

### 195. How do you unpivot columns into rows?

Use `UNPIVOT` where supported or `UNION ALL` over explicit column/value pairs.

### 196. What is grouping by an expression?

It groups by a computed value, such as a date bucket; compute consistently and consider indexed computed expressions where supported.

### 197. What is a grouping key cardinality?

It is the number of distinct groups produced; high cardinality can affect memory and aggregation strategy.

### 198. Why can `COUNT(DISTINCT)` be expensive?

It may need substantial sorting or hashing and memory, especially over large high-cardinality inputs.

### 199. What is approximate aggregation?

It estimates metrics such as distinct counts with bounded error using engine-specific algorithms.

### 200. When should aggregation happen in SQL?

Aggregate near the data source when it reduces transfer and the database can execute it efficiently.

## 10. Additional Questions (201-250): Data Modeling and Normalization

### 201. What is first normal form?

It requires each field to hold an appropriately atomic value for the intended relational model, rather than repeating groups in a cell.

### 202. What is second normal form?

It requires first normal form and that non-key attributes depend on the whole candidate key, not only part of a composite key.

### 203. What is third normal form?

It generally requires second normal form and no transitive dependency of non-key attributes on a key.

### 204. What is BCNF?

Boyce-Codd normal form requires every determinant to be a candidate key, a stronger condition than 3NF in some schemas.

### 205. What is an update anomaly?

It occurs when duplicated facts must be changed in multiple rows and inconsistent updates can result.

### 206. What is an insertion anomaly?

It occurs when a fact cannot be inserted without an unrelated fact because of table structure.

### 207. What is a deletion anomaly?

It occurs when deleting one fact unintentionally removes another fact stored in the same row.

### 208. What is denormalization?

It intentionally duplicates or precomputes data to meet performance or reporting needs, introducing consistency and maintenance costs.

### 209. What is a functional dependency?

It states that one attribute set uniquely determines another attribute set in the modeled data.

### 210. What is a natural key?

It is a key derived from meaningful domain attributes, such as a globally unique code.

### 211. What is a surrogate key?

It is an artificial identifier, such as a generated integer or UUID, without direct business meaning.

### 212. What is a composite primary key?

It identifies a row using multiple columns, common in junction tables and naturally scoped data.

### 213. What is a candidate key?

It is a minimal attribute set that uniquely identifies each row.

### 214. What is a partial key?

It identifies an entity only within the context of its owning entity, as in a weak-entity design.

### 215. What is an associative entity?

It models a many-to-many relationship as a table, especially when the relationship has its own attributes.

### 216. What is a junction table?

It is a linking table that stores foreign keys connecting entities in a many-to-many relationship.

### 217. What is a one-to-one relationship?

It associates at most one row on each side, typically enforced with a unique foreign key.

### 218. What is a one-to-many relationship?

One principal row can be referenced by multiple dependent rows through a foreign key.

### 219. What is a many-to-many relationship?

Rows on each side can relate to multiple rows on the other, usually through a junction table.

### 220. What is a self-referencing foreign key?

It references another row in the same table, commonly modeling parent-child hierarchies.

### 221. What is a weak entity?

It is an entity whose identity depends partly on an owning entity, often represented by a composite key.

### 222. What is an entity-relationship diagram?

It visually represents entities, attributes, and relationships used in data modeling.

### 223. What is a logical data model?

It defines entities and relationships independent of a particular database's physical implementation.

### 224. What is a physical data model?

It specifies concrete database objects, datatypes, indexes, partitions, and storage choices.

### 225. What is a slowly changing dimension?

It is a data-warehouse technique for managing changes to descriptive attributes over time.

### 226. What is SCD Type 1?

It overwrites the old dimension value and does not preserve history.

### 227. What is SCD Type 2?

It inserts a new versioned row and tracks validity periods to preserve history.

### 228. What is SCD Type 3?

It stores limited previous values in additional columns rather than full history.

### 229. What is a fact table?

It stores measurable business events or observations at a defined grain.

### 230. What is a dimension table?

It stores descriptive attributes used to categorize and filter fact data.

### 231. What is a star schema?

It organizes a central fact table with denormalized dimension tables for analytical querying.

### 232. What is a snowflake schema?

It normalizes dimension structures into related tables, reducing duplication but adding joins.

### 233. What is grain in a fact table?

It defines exactly what one row represents; measures and keys must follow that grain.

### 234. What is a degenerate dimension?

It is a dimensional identifier stored in a fact table without a separate dimension table.

### 235. What is a bridge table in analytics?

It resolves many-to-many relationships between dimensions or facts in an analytical schema.

### 236. What is a calendar dimension?

It stores date attributes such as fiscal periods and holidays, enabling consistent analytical filtering.

### 237. What is data integrity?

It is the accuracy and consistency of data, enforced through constraints, transactions, and application rules.

### 238. What is domain integrity?

It ensures a column's values conform to its allowed type, range, format, or domain.

### 239. What is entity integrity?

It ensures each row can be uniquely identified, commonly through a non-null primary key.

### 240. What is referential integrity?

It ensures references between related tables point to valid rows or are null where allowed.

### 241. What is a nullable foreign key?

It permits a row to have no related principal, representing an optional relationship.

### 242. What is a cascading delete?

It automatically deletes dependent rows when a principal row is deleted; use only when this matches retention rules.

### 243. What is a restrict/no-action delete rule?

It prevents or rejects deleting a referenced principal while dependent rows exist, subject to engine timing semantics.

### 244. What is a soft delete?

It marks a row as deleted rather than physically removing it, requiring consistent filtering and retention policies.

### 245. What is a temporal table?

It automatically preserves row versions over time in database features that support system-versioned history.

### 246. What is an audit table?

It records changes or actions for accountability, with access, retention, and tamper-resistance requirements.

### 247. What is a check constraint?

It restricts rows to values satisfying a predicate, subject to the database's treatment of unknown results.

### 248. What is an exclusion constraint?

It prevents specified conflicting values or ranges, available in some databases such as PostgreSQL.

### 249. What is a generated column?

It stores or computes a value from an expression, with virtual/stored behavior and index support varying by engine.

### 250. How do you choose a column datatype?

Choose the narrowest type that accurately represents valid domain values and supports required operations without premature truncation.

## 11. Additional Questions (251-300): Constraints, Indexes, and Objects

### 251. What is a unique index?

It enforces uniqueness using an index structure; `NULL` and filtered uniqueness semantics vary by database.

### 252. What is a filtered or partial index?

It indexes only rows satisfying a predicate, useful for selective subsets where supported.

### 253. What is a function-based index?

It indexes an expression such as `LOWER(email)`; syntax and determinism restrictions vary by engine.

### 254. What is an expression index?

It indexes the result of an expression so matching query expressions can use it.

### 255. What is a hash index?

It uses hashing for equality lookups; supported operators and persistence characteristics depend on the database.

### 256. What is a B-tree index?

It is a balanced tree structure supporting ordered lookups and range scans in many relational databases.

### 257. What is a bitmap index?

It uses bitmaps to represent value membership and can help analytical workloads with low-cardinality columns in supported systems.

### 258. What is a columnstore index?

It stores data column-wise, often improving compression and analytical scans at the cost of different write behavior.

### 259. What is a full-text index?

It indexes tokens and linguistic structures for text search rather than exact B-tree comparisons.

### 260. What is an index key?

It is the ordered column/expression portion used to locate entries and support ordering or seeks.

### 261. What are included columns?

They are non-key payload columns stored at index leaves in engines that support them, potentially covering queries.

### 262. What is index selectivity?

It describes how effectively an index key distinguishes rows; high selectivity often helps filters but is not the only optimizer factor.

### 263. What is index cardinality?

It is the number of distinct values or an estimate of them, used in selectivity and plan estimation.

### 264. What is a unique constraint versus unique index?

A unique constraint states a schema rule; databases often implement it with a unique index, but management and semantics differ.

### 265. Can a table have multiple indexes?

Yes, but each consumes space and adds write/maintenance work, so index sets should be workload-driven.

### 266. What is index fragmentation?

It describes suboptimal page organization in some storage engines; maintenance need depends on engine, workload, and measured impact.

### 267. What is an index rebuild?

It recreates an index, often compacting pages and updating structures; locking and online options vary by database.

### 268. What is an index reorganization?

It incrementally rearranges index pages in engines that offer the operation, typically with different locking costs than rebuild.

### 269. What is a covering query?

It is satisfied entirely from an index without fetching base-table rows, reducing lookups where the plan supports it.

### 270. What is a heap table?

It is a table without a clustered ordering structure in database terminology; exact physical behavior is engine-specific.

### 271. What is a clustered key?

It is the key used to organize a clustered table/index in engines that support clustered organization.

### 272. Why keep clustered keys narrow?

In engines that carry clustered keys in secondary indexes, wide keys can inflate every secondary index.

### 273. What is an index seek?

It navigates an index to a matching key range rather than scanning all index entries.

### 274. What is an index scan?

It reads many or all index entries; it may be optimal for large result sets or covering queries.

### 275. What is a key lookup?

It fetches missing columns from the base table after an index match, potentially expensive when repeated for many rows.

### 276. What is a bitmap scan?

It combines or applies bitmap index results to locate heap rows in systems that support the technique.

### 277. What is a database view?

It is a stored query definition presented as a virtual table.

### 278. What is an updatable view?

It permits modifications through the view when the database can map them unambiguously to base tables.

### 279. What is a materialized view?

It stores query results physically and must be refreshed or maintained according to engine rules.

### 280. What is an indexed view?

It is a view with a supporting physical index in engines that implement the feature, subject to strict definition rules.

### 281. What is a stored procedure?

It is a named database program that can accept parameters and perform SQL operations.

### 282. What is a user-defined function?

It is a database-defined function that returns a scalar or table result, with restrictions varying by engine.

### 283. What is a scalar function performance concern?

Per-row invocation or opaque function logic can inhibit optimization in some engines; newer optimizers may inline some functions.

### 284. What is a table-valued function?

It returns a rowset that can be queried as a table expression.

### 285. What is a trigger?

It runs automatically in response to configured database events; hidden side effects can complicate troubleshooting.

### 286. What is a DML trigger?

It responds to insert, update, or delete activity, often operating on sets of affected rows.

### 287. Why must triggers be set-based?

A single statement can modify many rows; trigger logic must handle the complete affected set, not assume one row.

### 288. What is a DDL trigger?

It responds to schema-definition events in databases that support such triggers.

### 289. What is a sequence?

It is a database object that generates numeric values independently of a particular table.

### 290. What is an identity or auto-increment column?

It generates values during inserts using engine-specific rules; gaps can occur and values should not be assumed gapless.

### 291. What is a sequence cache?

It reserves generated values for performance; crashes can leave gaps, which is normal for identifiers.

### 292. What is a synonym?

It is an alternate database name for an object in engines that support synonyms.

### 293. What is a temporary table?

It is a session- or transaction-scoped table, depending on the engine and creation form.

### 294. What is a table variable?

It is a dialect-specific table-like variable; optimizer statistics and transaction behavior differ from temporary tables.

### 295. What is a global temporary table?

It is a temporary table definition whose data visibility/lifetime may vary by session or transaction based on engine rules.

### 296. What is a database constraint name useful for?

It makes schema management, error diagnosis, and migration scripts clearer and more stable.

### 297. What is a check constraint's `UNKNOWN` behavior?

Many SQL systems accept a check when its predicate is true or unknown; pair with `NOT NULL` when null is prohibited.

### 298. Can a foreign key reference a non-unique column?

Usually the referenced columns must be a candidate key or otherwise unique according to the database's rules.

### 299. What is deferrable constraint checking?

Some databases allow constraints to be checked at transaction commit instead of after each statement.

### 300. What is schema migration?

It is a controlled change to database structure or data, usually versioned and deployed in a deliberate sequence.

## 12. Additional Questions (301-350): Transactions and Concurrency

### 301. What is autocommit?

It automatically commits each statement unless an explicit transaction is active; behavior and defaults depend on the client and engine.

### 302. What is an explicit transaction?

It groups multiple operations between transaction start and commit/rollback commands.

### 303. What is a savepoint?

It marks a point within a transaction to which partial work may be rolled back, where supported.

### 304. What does `COMMIT` do?

It makes the transaction's changes durable according to the database's configured guarantees.

### 305. What does `ROLLBACK` do?

It discards uncommitted transactional changes, subject to engine and operation semantics.

### 306. What is a nested transaction?

Some APIs expose nested transaction scopes, but many databases implement these as savepoints rather than independent durable transactions.

### 307. What is a distributed transaction?

It coordinates atomic work across multiple resource managers, often using a transaction coordinator and increased operational complexity.

### 308. What is a two-phase commit?

It is a protocol with prepare and commit phases for coordinating distributed atomicity, with blocking and recovery trade-offs.

### 309. What is a transaction log?

It records changes needed for recovery, durability, replication, or point-in-time restoration.

### 310. What is write-ahead logging?

It requires log records to be persisted before corresponding data pages, enabling recovery after failures.

### 311. What is a lock?

It is a concurrency-control mechanism that restricts conflicting access to data or resources.

### 312. What is a shared lock?

It generally allows concurrent readers while restricting conflicting writes; exact compatibility depends on the engine.

### 313. What is an exclusive lock?

It generally prevents other conflicting reads/writes while a transaction modifies data, subject to isolation implementation.

### 314. What is a lock granularity?

It is the size of the locked resource, such as row, page, or table.

### 315. What is lock escalation?

It replaces many fine-grained locks with a coarser lock to reduce lock-management overhead in some engines.

### 316. What is blocking?

It occurs when one operation waits for another to release a conflicting lock or resource.

### 317. What is a deadlock victim?

It is the transaction selected by the database to abort so the remaining transactions can proceed.

### 318. What is a deadlock graph?

It records participating processes and resources in a deadlock, helping identify conflicting access patterns.

### 319. How should applications retry deadlocks?

Retry only the aborted unit of work, with bounded backoff and idempotent effects; preserve transaction boundaries.

### 320. What is lock timeout?

It limits waiting for a lock before reporting failure; timeout behavior and configuration are database-specific.

### 321. What is MVCC?

Multi-Version Concurrency Control keeps row versions so readers can often access consistent snapshots without blocking writers.

### 322. What is snapshot isolation?

It lets a transaction read a consistent versioned snapshot; write conflicts and anomalies vary by implementation.

### 323. What is read committed snapshot?

It commonly gives each statement a consistent versioned view under a read-committed policy, in engines that support it.

### 324. What is serializable isolation?

It aims for outcomes equivalent to serial transaction execution, potentially using locks or conflict detection with reduced concurrency.

### 325. What is a lost update?

It occurs when concurrent writers overwrite each other's changes without detecting the conflict.

### 326. What is a write skew?

It is a concurrency anomaly where individually valid concurrent writes jointly violate a cross-row invariant.

### 327. What is a non-repeatable read?

A transaction reads a row, another transaction changes it, and a later read sees a different committed value.

### 328. What is a phantom?

Repeated range queries in a transaction see a different set of rows due to concurrent inserts or deletes.

### 329. What is a dirty write?

It is conflicting modification of data written by an uncommitted transaction; robust databases prevent this class of behavior.

### 330. What is a consistency invariant?

It is a business rule that must hold across data, enforced using constraints, transactions, or carefully coordinated application logic.

### 331. What is optimistic concurrency control?

It allows work to proceed without long-held locks and checks for conflicting changes at update time.

### 332. What is a version column?

It stores a value changed on updates and can be compared to detect stale writes.

### 333. What is compare-and-swap in SQL?

It updates a row only if its current version/value matches the caller's expected value, checking affected-row count for success.

### 334. What is a phantom-protection technique?

Serializable isolation or predicate/range locking can prevent phantoms, depending on database implementation.

### 335. What is a transaction boundary?

It defines which operations must commit or fail together to preserve a business invariant.

### 336. Why keep transactions short?

Short transactions reduce lock duration, contention, deadlock risk, and resource usage.

### 337. What is a long-running transaction risk?

It can retain locks or old row versions, grow logs, block maintenance, and increase failure recovery costs.

### 338. What is an idempotent operation?

Repeating it has the same intended final effect as performing it once, important for safe retries.

### 339. What is the transactional outbox pattern?

It stores a business change and outgoing message in one database transaction, then asynchronously publishes the message.

### 340. Why not publish to a message broker inside a database transaction?

The database and broker usually do not share one atomic transaction, so a failure can leave data and messages inconsistent.

### 341. What is a compensating transaction?

It performs a business-level reversal when distributed work cannot be rolled back atomically.

### 342. What is a transaction isolation anomaly?

It is an unexpected observation or outcome allowed by a concurrency policy, such as a non-repeatable read.

### 343. What is lock-free reading?

It is a broad description of reads that avoid traditional blocking locks, often using versions; exact guarantees are implementation-specific.

### 344. What is a dirty read hint?

It requests weak isolation in some dialects and can return uncommitted or inconsistent data; it is not a universal performance fix.

### 345. Why is `NOLOCK` risky?

In SQL Server it can permit dirty and inconsistent reads, including missing or duplicated rows; use a proper isolation strategy.

### 346. What is a phantom-safe range lock?

It protects a key range against inserts that would change a predicate result, depending on engine and isolation level.

### 347. What is lock compatibility?

It describes whether two lock modes can coexist on the same resource.

### 348. What is transaction promotion?

It escalates local transactional work to distributed coordination when multiple transactional resources are enlisted, depending on platform.

### 349. What is eventual consistency?

It allows replicas or derived views to converge over time rather than reflecting every write immediately.

### 350. How do you maintain invariants under concurrent inserts?

Enforce uniqueness or other invariants in the database and handle constraint conflicts, rather than relying only on pre-checks.

## 13. Additional Questions (351-400): SQL Programming and Data Operations

### 351. What is a batch?

It is a group of SQL statements submitted or compiled together; batch delimiters are client-specific.

### 352. What is a transaction script?

It is a sequence of statements with explicit success, rollback, and error-handling behavior.

### 353. What is a cursor?

It processes query rows one at a time; cursors can be useful for procedural tasks but set-based operations are often more efficient.

### 354. What is a set-based operation?

It expresses work over a collection of rows in one relational statement rather than looping one row at a time.

### 355. Why prefer set-based SQL?

It lets the optimizer choose execution strategies and often reduces round trips and procedural overhead.

### 356. When may a cursor be justified?

When work truly requires ordered per-row procedural interaction that cannot be expressed clearly or safely as set-based operations.

### 357. What is dynamic cursor behavior?

Cursor sensitivity, scrollability, and visibility of concurrent changes depend on cursor type and database.

### 358. What is a prepared statement?

It is a parsed/parameterized command that can be executed with varying values, often improving safety and reuse.

### 359. What is a batch separator?

It is a client-tool convention, such as SQL Server's `GO`, not necessarily a command understood by the database server.

### 360. What is an execution plan cache?

It stores compiled plans for potential reuse; invalidation and parameterization rules differ by engine.

### 361. What is a temporary staging table?

It holds intermediate data during ETL or multi-step processing, with scope and cleanup determined by its kind.

### 362. What is ETL?

Extract, Transform, Load moves and reshapes data from sources into a target store.

### 363. What is ELT?

Extract, Load, Transform loads source data first and transforms it within the target platform.

### 364. What is a bulk load?

It efficiently imports many rows using database-specific facilities, with logging and constraint behaviors to verify.

### 365. What is a bulk insert risk?

It may bypass or alter expected triggers, constraints, logging, or identity behavior depending on options and engine.

### 366. What is upsert?

It inserts a row or updates an existing row on a key conflict; syntax and concurrency guarantees differ by database.

### 367. What is `MERGE`?

It combines insert/update/delete actions based on matching source and target rows; correctness and concurrency edge cases are engine-specific.

### 368. What is an idempotent upsert?

It safely produces the same intended state when repeated, including when a previous response was lost.

### 369. How do you safely import CSV data?

Validate encoding, quoting, delimiters, null representations, datatypes, and row counts; stage and verify before publishing.

### 370. What is a staging table?

It temporarily stores incoming data for validation, deduplication, transformation, and controlled loading.

### 371. What is data reconciliation?

It compares source and target counts, totals, keys, or checksums to detect missing or changed records.

### 372. What is CDC?

Change Data Capture records data modifications for downstream processing using database-specific facilities.

### 373. What is change tracking?

It records which rows changed, often with less historical detail than CDC; exact semantics are engine-specific.

### 374. What is logical replication?

It replicates logical row changes or transactions between database instances, with platform-specific filtering and conflict behavior.

### 375. What is a database backup?

It is a recoverable copy of database data and, depending on method, logs and metadata.

### 376. What is point-in-time recovery?

It restores a backup and replays logs to recover to a selected time or transaction boundary.

### 377. What is a recovery point objective?

RPO is the acceptable amount of data loss measured in time after a failure.

### 378. What is a recovery time objective?

RTO is the target time to restore service after an outage.

### 379. What is a database restore test?

It verifies backups can actually be restored and that recovered data meets operational requirements.

### 380. What is a data retention policy?

It defines how long data is kept and how it is archived or securely deleted.

### 381. What is partitioning?

It divides a large table or index into manageable segments based on a partitioning key and engine support.

### 382. What is horizontal partitioning?

It splits rows into partitions, often by range, list, or hash.

### 383. What is vertical partitioning?

It separates columns into related tables or storage groups, often to isolate rarely accessed or large columns.

### 384. What is partition pruning?

It skips partitions that cannot satisfy query predicates, reducing scanned data.

### 385. What is partition-key selection?

It chooses a key aligned with query filtering, retention, and load patterns while avoiding skew.

### 386. What is sharding?

It distributes data across independent database nodes, adding routing, rebalancing, and cross-shard query complexity.

### 387. What is a shard key?

It determines which shard stores a row; a poor key can cause hotspots or costly distributed queries.

### 388. What is a hot partition?

It is a partition receiving disproportionate requests or writes, causing imbalance and contention.

### 389. What is a read replica?

It serves read traffic from replicated data, which may lag behind the primary.

### 390. What is replication lag?

It is the delay between a primary change and its visibility on a replica.

### 391. What is a consistency read concern?

It specifies freshness or consistency guarantees for reads in systems that expose such options.

### 392. What is a database failover?

It promotes or routes service to another node after a failure, with possible recovery time and data-loss characteristics.

### 393. What is a connection pool?

It reuses database connections to reduce setup cost; pool limits and lifetime should be configured for workload.

### 394. Why avoid opening one connection per row?

It incurs network and authentication overhead and can exhaust the connection pool.

### 395. What is a database session?

It is a client connection context that may retain settings, temporary objects, and transaction state.

### 396. What is a session setting risk?

Pooled connections can retain state if the client does not reset it correctly, affecting later operations.

### 397. What is a maintenance window?

It is a planned period for disruptive database work such as upgrades, index maintenance, or migrations.

### 398. What is a schema drift?

It is an unintended difference between the actual database schema and the declared or expected schema.

### 399. How do you deploy schema changes safely?

Use reviewed, versioned migrations and backward-compatible expand/contract steps for applications deployed concurrently.

### 400. What is expand-and-contract migration?

It adds compatible schema first, migrates application/data use, then removes obsolete schema in a later deployment.

## 14. Additional Questions (401-450): Query Plans and Performance

### 401. What is a cost-based optimizer?

It estimates alternative execution-plan costs using statistics and chooses a plan expected to be efficient.

### 402. What is cardinality estimation?

It predicts the number of rows produced by plan operators, influencing join, memory, and access choices.

### 403. What is a histogram statistic?

It summarizes value distribution for an indexed or analyzed column to estimate predicate selectivity.

### 404. What is a stale statistic?

It is a distribution summary that no longer reflects current data, potentially causing poor row estimates and plans.

### 405. How are statistics refreshed?

Use database-managed automatic statistics or explicit analyze/update commands appropriate to the engine.

### 406. What is parameter-sensitive planning?

Different parameter values need different plans because their matching row counts vary substantially.

### 407. What is plan recompilation?

It recompiles a statement to generate a fresh plan, potentially reflecting new statistics or parameters but adding compile cost.

### 408. What is a query hint?

It directs or constrains optimizer choices; hints can become harmful as data and engine versions change.

### 409. Should hints be a first optimization step?

No. First understand the plan, statistics, indexes, and query shape; hint only with measured justification.

### 410. What is a table scan?

It reads table pages broadly; it may be optimal for small tables or queries returning a large fraction of rows.

### 411. What is a nested-loop join?

It iterates rows from one input and probes another, often efficient when the outer input is small and inner access is indexed.

### 412. What is a hash join?

It builds a hash structure on one input and probes it with another, often useful for larger unsorted equijoins.

### 413. What is a merge join?

It combines sorted inputs by key and can be efficient when suitable ordering already exists.

### 414. What is a sort operator?

It orders rows and can consume CPU and memory or spill to temporary storage.

### 415. What is a hash aggregate?

It groups rows using a hash table, with memory needs related to group count and row size.

### 416. What is a stream aggregate?

It aggregates ordered rows, often using less memory when input is already grouped by key.

### 417. What is a spill to disk?

It occurs when an operator exceeds available memory and uses temporary storage, often degrading performance.

### 418. What is memory grant estimation?

It estimates memory needed by query operators such as sorts and hashes; too little can spill, too much can reduce concurrency.

### 419. What is logical I/O?

It measures page accesses through the database buffer cache, helping compare query work independent of physical reads.

### 420. What is physical I/O?

It reflects storage reads/writes beyond cached pages, though operating-system and database caches affect measurement.

### 421. What is CPU time versus elapsed time?

CPU time measures processor work; elapsed time includes waiting on I/O, locks, scheduling, and other delays.

### 422. What is wait statistics analysis?

It studies time spent waiting on resources to identify bottlenecks such as I/O, locks, or CPU scheduling.

### 423. What is a sargable expression?

It is a predicate form that can efficiently use an index search key rather than requiring per-row transformation.

### 424. How can implicit conversion hurt a plan?

It may force conversion on the indexed column, preventing an efficient seek or producing incorrect comparison semantics.

### 425. What is a non-sargable predicate?

It requires evaluating a function or expression per row in a way that prevents efficient index navigation.

### 426. What is a residual predicate?

It is a filter applied after an index access because it could not be fully represented as the seek condition.

### 427. What is selectivity estimation for `LIKE`?

It estimates the fraction matching a pattern; leading wildcards and skewed distributions make estimates difficult.

### 428. What is join cardinality explosion?

It is unexpected row multiplication from many-to-many matches or incorrect join keys.

### 429. How do you diagnose duplicate join rows?

Check uniqueness and grain on both sides, inspect join predicates, and aggregate/test keys before adding `DISTINCT`.

### 430. What is a covering index trade-off?

It can eliminate lookups but increases index size, cache pressure, and write cost.

### 431. What is index intersection?

It combines multiple index results to satisfy predicates, depending on optimizer capability and cost.

### 432. What is index skip scan?

It uses a composite index despite a missing leading predicate by iterating distinct leading values, where supported.

### 433. What is a partial index's benefit?

It reduces size and maintenance by indexing only rows relevant to a common selective predicate.

### 434. What is table statistics correlation?

It describes relationships between columns; missing correlation knowledge can produce poor estimates for combined predicates.

### 435. What is a correlation statistic?

It captures dependencies among columns in engines that support extended statistics.

### 436. What is query plan regression?

It occurs when a new plan performs worse after data, statistics, schema, parameters, or engine changes.

### 437. What is plan forcing?

It pins or guides a known plan in supported systems; monitor it because future data may make it suboptimal.

### 438. What is a query store?

It records query plans and runtime history in systems with that feature, helping identify regressions.

### 439. What is parameterization for plan reuse?

It allows similar statements with different values to share a compiled plan, but one plan may not suit all values.

### 440. What is batching database requests?

It sends multiple commands together to reduce round trips, while transaction and error semantics must be understood.

### 441. What is N+1 query behavior?

An application issues one query for a list and then another query per row; batching or joins can reduce round trips.

### 442. What is network round-trip cost?

It is latency and protocol overhead for each client-server exchange, often significant for chatty workloads.

### 443. What is result-set overfetching?

It retrieves more rows or columns than needed, consuming database, network, and application resources.

### 444. What is pagination's consistency issue?

Concurrent inserts or deletes can shift offset-based pages; stable keyset cursors or snapshot semantics may be required.

### 445. What is query timeout?

It limits how long a client or server waits for a query; timeout cancellation may not immediately stop all underlying work.

### 446. How do you optimize bulk updates?

Use set-based updates, suitable predicates/indexes, controlled batches, and transaction sizes that balance log and lock pressure.

### 447. What is batching large deletes?

It removes subsets iteratively to limit transaction log, lock duration, and replication impact; ensure progress and restartability.

### 448. Why may an index slow inserts?

Each insert must maintain relevant indexes, increasing I/O, logging, and possible page contention.

### 449. What is write amplification?

It is the extra physical work generated by logical writes across indexes, logs, replicas, and storage layers.

### 450. How do you validate a performance fix?

Compare representative plans and measurements under controlled data and load, and verify correctness and write-side trade-offs.

## 15. Additional Questions (451-500): Security, Reliability, and Advanced SQL

### 451. What is SQL injection?

It is attacker-controlled input changing SQL command structure; parameter binding is the primary defense for values.

### 452. Can a parameter represent a table name?

Usually no; identifiers must be selected from a strict allowlist and quoted using safe dialect-specific rules.

### 453. What is least privilege?

It grants each user or service account only the database permissions required for its role.

### 454. What is role-based access control?

It assigns permissions to roles and roles to principals, simplifying consistent privilege management.

### 455. What is row-level security?

It restricts visible or modifiable rows based on policies evaluated by the database, where supported.

### 456. What is column-level security?

It restricts access to selected columns through grants, views, masking, or policy mechanisms.

### 457. What is dynamic data masking?

It obscures returned values for selected principals in supported databases; it is not a substitute for access control or encryption.

### 458. What is encryption at rest?

It protects stored database files or backups using cryptographic mechanisms, with key management essential to security.

### 459. What is encryption in transit?

It protects network traffic, typically using TLS with certificate validation.

### 460. What is transparent data encryption?

It encrypts database files at rest transparently in systems that provide the feature; it does not necessarily protect query results or backups automatically.

### 461. What is column-level encryption?

It encrypts selected values, sometimes at the application layer, changing queryability and key-management requirements.

### 462. What is a secret rotation?

It replaces credentials or encryption keys on a planned schedule and updates dependent services safely.

### 463. What is an audit log?

It records access or change events for accountability and compliance, with protection against tampering and excessive sensitive data.

### 464. What is a database principal?

It is an identity recognized by the database, such as a user, role, or service account.

### 465. What is a schema owner?

It is a principal or role controlling objects in a schema; ownership affects permissions and security context.

### 466. What is `GRANT`?

It grants specified privileges to a principal or role.

### 467. What is `REVOKE`?

It removes a previously granted privilege, subject to database permission semantics.

### 468. What is `DENY`?

It explicitly denies a permission in systems that distinguish deny from revoke and grant.

### 469. What is a definer-rights procedure?

It executes with privileges of its owner/definer in supported systems, requiring careful review to avoid privilege escalation.

### 470. What is invoker-rights execution?

It executes using the caller's permissions in systems that support this security model.

### 471. What is data masking for nonproduction?

It replaces or transforms sensitive data in test environments while preserving useful structural characteristics.

### 472. What is tokenization?

It replaces sensitive values with tokens whose mapping is stored separately under controlled access.

### 473. What is a privacy deletion request?

It is a request to remove or anonymize personal data according to legal, retention, and referential requirements.

### 474. What is data minimization?

It limits collection, storage, and access to information necessary for a stated purpose.

### 475. What is a JSON column?

It stores JSON documents using database-specific text or native types, with varying validation and indexing support.

### 476. When should data be stored relationally versus JSON?

Use relational columns for stable, constrained, frequently queried attributes; use JSON for appropriate flexible structures while preserving validation and indexing needs.

### 477. What is a spatial datatype?

It represents geometric or geographic values and supports spatial operations in databases with spatial extensions.

### 478. What is a geospatial index?

It accelerates spatial predicates using specialized structures, with supported geometries and coordinate semantics depending on the engine.

### 479. What is an array column?

It stores multiple values in one column in databases that support arrays; use it only when query and integrity needs suit the model.

### 480. What is an XML datatype?

It stores or validates XML data in supported systems, often with specialized query and indexing features.

### 481. What is a database collation attack or risk?

Unexpected comparison rules can affect uniqueness, authorization matching, and sort behavior; define and test collations intentionally.

### 482. What is case-sensitive comparison?

It treats letter case as significant according to collation or explicit comparison settings.

### 483. What is Unicode normalization?

It represents equivalent text sequences in standardized forms; visually identical strings may compare differently without normalization.

### 484. What is a check digit?

It is a calculated digit used to detect some data-entry errors, not to provide security or prove authenticity.

### 485. What is a UUID/GUID?

It is a large identifier intended to be unique with extremely high probability; generation and index locality depend on variant.

### 486. What is a sequential UUID?

It is an identifier variant designed to improve insertion locality compared with randomly distributed UUIDs.

### 487. What is an idempotency key?

It is a client-supplied unique request key stored with operation outcome to prevent duplicate effects on retries.

### 488. What is a checksum?

It detects some data differences but is not necessarily cryptographically secure; use a cryptographic hash when adversarial tampering matters.

### 489. What is a window frame exclusion?

It excludes selected rows, such as the current row or ties, from a window frame where supported.

### 490. What is `LATERAL`?

It allows a `FROM` subquery to reference preceding `FROM` items in systems supporting lateral joins.

### 491. What is a `VALUES` table constructor?

It creates a small inline rowset of literal values, useful for test data or joins.

### 492. What is `RETURNING` or `OUTPUT`?

It returns affected-row values from data modifications in supported dialects, with syntax and trigger interactions varying.

### 493. What is a data-modifying CTE?

It combines modification statements with query expressions in databases supporting that syntax; atomicity and ordering rules are engine-specific.

### 494. What is `MERGE` concurrency risk?

Concurrent changes and engine-specific implementation details can produce unexpected conflicts; verify correctness, locking, and constraints for the exact version.

### 495. What is a generated identity gap?

It is a missing identifier value caused by rollback, caching, or failed inserts; identifiers are not generally gapless business sequences.

### 496. Why should financial sequence numbers not rely on identity values?

Identity generators can have gaps and are not designed to satisfy legal or business numbering guarantees.

### 497. What is temporal consistency?

It ensures related values represent compatible points in time, often requiring a transaction snapshot or explicit valid-time model.

### 498. What is bitemporal data?

It tracks both valid time in the real world and transaction/system time in the database.

### 499. What is a database portability concern?

SQL dialects differ in datatypes, functions, locking, identity generation, isolation, DDL, and optimizer behavior; test on the target engine.

### 500. How should you prepare for a SQL interview?

Practice explaining relational semantics, write queries with edge cases, read execution plans, and state database-specific assumptions explicitly.
