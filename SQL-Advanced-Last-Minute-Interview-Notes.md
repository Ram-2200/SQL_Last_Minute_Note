# SQL Advanced Last-Minute Interview Notes

> **Fast revision for SQL interviews, coding assessments, production queries, analytics, and database fundamentals.**
>
> Learn the pattern → understand the execution → write the query → check edge cases → optimize it.

---

## 0. SQL Interview Mental Model

When you see a SQL problem, do not immediately start typing.

Think:

```text
What is the required output?
        ↓
What tables contain the required data?
        ↓
What is the relationship between tables?
        ↓
Do I need row filtering or group filtering?
        ↓
Do I need aggregation?
        ↓
Do I need ranking/window functions?
        ↓
Could duplicates appear?
        ↓
How should NULL behave?
        ↓
Can the query be optimized?
```

### Logical query processing order

A useful mental model is:

```text
FROM / JOIN
      ↓
WHERE
      ↓
GROUP BY
      ↓
HAVING
      ↓
WINDOW FUNCTIONS
      ↓
SELECT
      ↓
DISTINCT
      ↓
ORDER BY
      ↓
LIMIT / FETCH
```

The exact optimizer execution plan can differ. This is the **logical** order used to reason about SQL.

---

# 1. SQL vs MySQL

### SQL

SQL is a language used to define, query, manipulate, and control relational data.

### MySQL

MySQL is a relational database management system that implements SQL plus database-specific behavior and extensions.

Similar distinction:

```text
SQL       → language
MySQL     → database system
PostgreSQL → database system
Oracle    → database system
SQL Server → database system
```

Do not say:

> "SQL and MySQL are the same."

---

# 2. SQL Command Categories

| Category | Purpose | Examples |
|---|---|---|
| DDL | Define database objects | `CREATE`, `ALTER`, `DROP`, `TRUNCATE` |
| DML | Modify data | `INSERT`, `UPDATE`, `DELETE` |
| DQL | Query data | `SELECT` |
| DCL | Permissions | `GRANT`, `REVOKE` |
| TCL | Transactions | `COMMIT`, `ROLLBACK`, `SAVEPOINT` |

---

# 3. SELECT Fundamentals

```sql
SELECT name, salary
FROM employees
WHERE salary > 50000
ORDER BY salary DESC
LIMIT 10;
```

### Important

Avoid:

```sql
SELECT *
```

in production queries when you only need specific columns.

Reasons:

- Unnecessary data transfer
- More I/O
- More network traffic
- Less explicit dependencies
- Can interact poorly with covering/index strategies

---

# 4. DISTINCT

```sql
SELECT DISTINCT department
FROM employees;
```

`DISTINCT` removes duplicate result rows.

### Trap

`DISTINCT` applies to the complete selected row, not just one visually important column.

```sql
SELECT DISTINCT department, salary
FROM employees;
```

The combination `(department, salary)` must be unique in the result.

---

# 5. WHERE

Filters rows before aggregation.

```sql
SELECT *
FROM employees
WHERE salary > 50000;
```

Multiple conditions:

```sql
WHERE department = 'IT'
  AND salary > 50000
```

---

# 6. NULL

`NULL` means missing/unknown/not-applicable depending on context.

It is not:

```text
0
''
FALSE
```

### Wrong

```sql
WHERE manager_id = NULL
```

### Correct

```sql
WHERE manager_id IS NULL
```

### Not equal to NULL

```sql
WHERE manager_id IS NOT NULL
```

---

# 7. Three-Valued Logic

SQL conditions can evaluate to:

```text
TRUE
FALSE
UNKNOWN
```

Example:

```sql
NULL = 10
```

is not TRUE or FALSE in the ordinary sense; it evaluates to UNKNOWN.

This matters especially with:

```sql
WHERE
JOIN
NOT IN
CASE
```

---

# 8. COUNT() and NULL

Important distinction:

```sql
COUNT(*)
```

counts rows.

```sql
COUNT(column)
```

counts non-NULL values in that column.

Example:

```sql
SELECT
    COUNT(*) AS total_rows,
    COUNT(manager_id) AS rows_with_manager
FROM employees;
```

---

# 9. GROUP BY

Used to create groups for aggregation.

```sql
SELECT
    department,
    COUNT(*) AS employee_count
FROM employees
GROUP BY department;
```

Common aggregate functions:

```text
COUNT()
SUM()
AVG()
MIN()
MAX()
```

---

# 10. WHERE vs HAVING

### WHERE

Filters rows **before grouping**.

```sql
WHERE salary > 50000
```

### HAVING

Filters groups **after aggregation**.

```sql
HAVING COUNT(*) > 5
```

Example:

```sql
SELECT
    department,
    COUNT(*) AS employee_count
FROM employees
WHERE salary > 50000
GROUP BY department
HAVING COUNT(*) >= 5;
```

Mental model:

```text
WHERE
 ↓
rows

GROUP BY
 ↓
groups

HAVING
 ↓
groups
```

---

# 11. JOINs

## INNER JOIN

Returns matching rows.

```sql
SELECT e.name, d.name
FROM employees e
INNER JOIN departments d
    ON e.department_id = d.department_id;
```

---

## LEFT JOIN

Returns every row from the left table plus matching rows from the right.

```sql
SELECT e.name, d.name
FROM employees e
LEFT JOIN departments d
    ON e.department_id = d.department_id;
```

If there is no match:

```text
right-side columns → NULL
```

---

## RIGHT JOIN

Everything from the right table plus matching rows from the left.

In many codebases, rewriting it as a `LEFT JOIN` by swapping table order can make queries easier to read.

---

## FULL OUTER JOIN

Returns matched and unmatched rows from both sides.

Not supported with identical syntax by every database system; always check your database dialect.

---

## CROSS JOIN

Cartesian product.

```sql
SELECT *
FROM colors
CROSS JOIN sizes;
```

If:

```text
colors = 3 rows
sizes  = 4 rows
```

result:

```text
3 × 4 = 12 rows
```

---

## SELF JOIN

A table joined to itself.

Example employee-manager relationship:

```sql
SELECT
    e.name AS employee,
    m.name AS manager
FROM employees e
LEFT JOIN employees m
    ON e.manager_id = m.employee_id;
```

---

# 12. The LEFT JOIN + WHERE Trap

Consider:

```sql
SELECT e.name, d.name
FROM employees e
LEFT JOIN departments d
    ON e.department_id = d.department_id
WHERE d.name = 'IT';
```

The `WHERE` condition removes rows where `d.name` is NULL, so the result behaves like an inner join for that condition.

If you want to preserve unmatched left rows while restricting matches:

```sql
SELECT e.name, d.name
FROM employees e
LEFT JOIN departments d
    ON e.department_id = d.department_id
   AND d.name = 'IT';
```

This is a common interview question.

---

# 13. Duplicate Rows After JOIN

If a join unexpectedly multiplies rows, check the relationship.

Example:

```text
customers
    1
    ↓
orders
    many
```

Joining one customer to many orders naturally produces multiple rows per customer.

Before joining, ask:

```text
1:1?
1:N?
N:1?
N:N?
```

Many SQL bugs are actually **cardinality mistakes**.

---

# 14. Find Duplicates

```sql
SELECT
    email,
    COUNT(*) AS occurrences
FROM users
GROUP BY email
HAVING COUNT(*) > 1;
```

Find duplicate combinations:

```sql
SELECT
    first_name,
    last_name,
    birth_date,
    COUNT(*)
FROM customers
GROUP BY
    first_name,
    last_name,
    birth_date
HAVING COUNT(*) > 1;
```

---

# 15. Remove Duplicates from Results

First ask:

> Why are duplicates being generated?

Do not blindly add:

```sql
DISTINCT
```

`DISTINCT` can hide an incorrect join.

Better:

```text
Understand cardinality
        ↓
Fix JOIN
        ↓
Use aggregation if required
        ↓
Use DISTINCT only when duplicate result rows are legitimately unwanted
```

---

# 16. Subqueries

### Scalar subquery

Returns one value.

```sql
SELECT name, salary
FROM employees
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
);
```

### Multi-row subquery

```sql
SELECT *
FROM employees
WHERE department_id IN (
    SELECT department_id
    FROM departments
    WHERE location = 'Bangalore'
);
```

### Correlated subquery

References a value from the outer query.

```sql
SELECT e1.name, e1.salary
FROM employees e1
WHERE e1.salary > (
    SELECT AVG(e2.salary)
    FROM employees e2
    WHERE e2.department_id = e1.department_id
);
```

A correlated subquery can be expensive depending on the database and optimizer. Do not assume it literally executes once per outer row; inspect the execution plan.

---

# 17. EXISTS vs IN

### EXISTS

Checks whether at least one matching row exists.

```sql
SELECT c.customer_id
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
);
```

### IN

Compares against a set of values.

```sql
SELECT *
FROM employees
WHERE department_id IN (
    SELECT department_id
    FROM departments
);
```

Do not claim that `EXISTS` is always faster than `IN` or vice versa.

Performance depends on:

- Data distribution
- Indexes
- Database engine
- Query shape
- Optimizer

---

# 18. NOT IN + NULL Trap

This is a classic interview question.

Suppose:

```sql
WHERE id NOT IN (
    SELECT employee_id
    FROM terminated_employees
)
```

If the subquery contains `NULL`, three-valued logic can cause unexpected results.

Safer pattern when appropriate:

```sql
WHERE NOT EXISTS (
    SELECT 1
    FROM terminated_employees t
    WHERE t.employee_id = e.employee_id
);
```

Always reason explicitly about NULL.

---

# 19. CTE

Common Table Expression:

```sql
WITH department_stats AS (
    SELECT
        department_id,
        AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department_id
)
SELECT *
FROM department_stats
WHERE avg_salary > 60000;
```

Advantages:

- Readability
- Breaking complex queries into steps
- Easier debugging
- Recursive queries

A CTE is not automatically faster than a subquery. Optimization behavior is database-specific.

---

# 20. Recursive CTE

Useful for hierarchical data.

Example:

```sql
WITH RECURSIVE employee_tree AS (
    SELECT
        employee_id,
        name,
        manager_id,
        0 AS level
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    SELECT
        e.employee_id,
        e.name,
        e.manager_id,
        t.level + 1
    FROM employees e
    JOIN employee_tree t
        ON e.manager_id = t.employee_id
)
SELECT *
FROM employee_tree;
```

Common use cases:

- Organization hierarchy
- Category trees
- Graph-like traversal
- Parent/child relationships

---

# 21. Window Functions

Window functions calculate across related rows **without collapsing the result into one row per group**.

MySQL documents window functions as calculations over rows related to the current row. citeturn0search5

General form:

```sql
function(...) OVER (
    PARTITION BY ...
    ORDER BY ...
)
```

---

# 22. ROW_NUMBER vs RANK vs DENSE_RANK

Suppose salaries:

```text
100
100
90
80
```

### ROW_NUMBER

```text
1
2
3
4
```

### RANK

```text
1
1
3
4
```

### DENSE_RANK

```text
1
1
2
3
```

---

# 23. Top N Per Group

One of the most important SQL patterns.

Example: top 3 salaries per department.

```sql
WITH ranked AS (
    SELECT
        employee_id,
        name,
        department_id,
        salary,
        DENSE_RANK() OVER (
            PARTITION BY department_id
            ORDER BY salary DESC
        ) AS rnk
    FROM employees
)
SELECT *
FROM ranked
WHERE rnk <= 3;
```

Use `ROW_NUMBER()` if you need exactly N rows per group.

Use `DENSE_RANK()` if ties should share a rank.

---

# 24. LAG and LEAD

Compare the current row with another row.

```sql
SELECT
    employee_id,
    salary,
    LAG(salary) OVER (
        ORDER BY employee_id
    ) AS previous_salary
FROM employees;
```

Useful for:

- Month-over-month changes
- Previous transaction
- Previous status
- Trend analysis

---

# 25. Running Total

```sql
SELECT
    order_date,
    amount,
    SUM(amount) OVER (
        ORDER BY order_date
    ) AS running_total
FROM orders;
```

Partitioned running total:

```sql
SUM(amount) OVER (
    PARTITION BY customer_id
    ORDER BY order_date
)
```

---

# 26. Moving Average

```sql
AVG(amount) OVER (
    ORDER BY order_date
    ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
)
```

This represents a 7-row moving average, not necessarily a 7-day average.

That distinction matters when dates have gaps.

---

# 27. GROUP BY vs Window Function

### GROUP BY

Collapses rows.

```sql
SELECT department_id, AVG(salary)
FROM employees
GROUP BY department_id;
```

Result:

```text
one row per department
```

### Window

Keeps rows.

```sql
SELECT
    employee_id,
    department_id,
    salary,
    AVG(salary) OVER (
        PARTITION BY department_id
    ) AS department_avg
FROM employees;
```

Result:

```text
one row per employee
+
department average
```

---

# 28. CASE

Conditional logic:

```sql
SELECT
    name,
    salary,
    CASE
        WHEN salary >= 100000 THEN 'High'
        WHEN salary >= 60000 THEN 'Medium'
        ELSE 'Low'
    END AS salary_band
FROM employees;
```

Conditional aggregation:

```sql
SELECT
    COUNT(CASE WHEN status = 'active' THEN 1 END) AS active_count
FROM users;
```

Portable alternative:

```sql
SUM(CASE WHEN status = 'active' THEN 1 ELSE 0 END)
```

---

# 29. COALESCE

Returns the first non-NULL expression.

```sql
SELECT
    name,
    COALESCE(phone, 'Not Available')
FROM customers;
```

Useful for:

- Default values
- NULL-safe calculations
- Display logic

---

# 30. NULLIF

Returns NULL when two expressions are equal.

Useful for preventing division-by-zero:

```sql
SELECT
    revenue / NULLIF(orders, 0)
FROM sales;
```

---

# 31. Date and Time

Common operations:

```sql
CURRENT_DATE
CURRENT_TIMESTAMP
```

But date functions vary between database systems.

Typical tasks:

```text
Extract year/month
Date difference
Date arithmetic
Monthly aggregation
Rolling periods
Latest record
```

Example pattern:

```sql
SELECT
    DATE(order_date) AS order_day,
    COUNT(*) AS orders
FROM orders
GROUP BY DATE(order_date);
```

For production queries, prefer sargable date predicates when possible.

Instead of wrapping an indexed timestamp column unnecessarily:

```sql
WHERE DATE(created_at) = '2026-09-25'
```

consider:

```sql
WHERE created_at >= '2026-09-25'
  AND created_at <  '2026-09-26'
```

This exact syntax is database-dependent, but the principle is important: avoid applying functions to indexed columns when it prevents efficient index use.

---

# 32. Keys

## Primary Key

Uniquely identifies a row.

Properties:

- Unique
- Not NULL
- One primary-key constraint per table

Example:

```sql
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    name VARCHAR(100)
);
```

---

## Foreign Key

Maintains referential relationships.

```sql
FOREIGN KEY (department_id)
REFERENCES departments(department_id)
```

---

## Candidate Key

A minimal set of attributes that can uniquely identify a row.

---

## Composite Key

Key consisting of multiple columns.

```sql
PRIMARY KEY (order_id, product_id)
```

---

# 33. UNIQUE vs PRIMARY KEY

| PRIMARY KEY | UNIQUE |
|---|---|
| Identifies row | Enforces uniqueness |
| Cannot be NULL in normal relational semantics | NULL behavior varies by DB |
| One primary-key constraint | Multiple unique constraints possible |
| Often referenced by foreign keys | Can also be referenced depending on DB |

Do not overgeneralize NULL behavior across database engines.

---

# 34. Normalization

Purpose:

- Reduce unnecessary redundancy
- Improve data integrity
- Reduce update anomalies

### 1NF

Atomic values; no repeating groups.

### 2NF

1NF + no partial dependency on part of a composite candidate key.

### 3NF

2NF + no problematic transitive dependency of non-key attributes on a key.

### BCNF

Every determinant is a candidate key.

Interview point:

> Normalization is not "always split everything." Database design balances integrity, maintainability, query patterns, and performance.

---

# 35. Transactions

A transaction groups operations into a unit of work.

```sql
START TRANSACTION;

UPDATE accounts
SET balance = balance - 100
WHERE id = 1;

UPDATE accounts
SET balance = balance + 100
WHERE id = 2;

COMMIT;
```

Rollback:

```sql
ROLLBACK;
```

---

# 36. ACID

### Atomicity

All required changes happen or the transaction is rolled back.

### Consistency

A committed transaction preserves the database's declared integrity rules and consistency guarantees.

### Isolation

Concurrent transactions are controlled so their interactions obey the database's isolation semantics.

### Durability

Once committed, changes are intended to survive failures according to the database's durability guarantees.

---

# 37. Isolation Levels

Common standard levels:

```text
READ UNCOMMITTED
READ COMMITTED
REPEATABLE READ
SERIALIZABLE
```

Potential phenomena:

```text
Dirty read
Non-repeatable read
Phantom read
```

The exact behavior and implementation differ between database engines.

Do not memorize a simplistic matrix without knowing the database you are interviewing for.

---

# 38. Dirty Read

Transaction A reads uncommitted changes made by Transaction B.

```text
B updates
 ↓
A reads
 ↓
B rolls back
 ↓
A had read data that never committed
```

---

# 39. Non-Repeatable Read

Transaction A reads a row.

Transaction B changes and commits it.

Transaction A reads again and gets a different committed value.

---

# 40. Phantom Read

Transaction A repeats a range query and sees a different set of rows because another transaction inserted/deleted matching rows.

---

# 41. Deadlock

Two or more transactions wait on resources held by each other.

Example:

```text
Transaction A:
locks Row 1
waits for Row 2

Transaction B:
locks Row 2
waits for Row 1
```

Database systems typically detect deadlocks and abort one transaction.

Prevention strategies:

- Consistent lock ordering
- Short transactions
- Appropriate indexes
- Avoid unnecessary locking
- Retry aborted transactions where appropriate

---

# 42. Indexes

An index is a data structure that helps the database locate rows efficiently.

MySQL documentation notes that indexes can speed filtering and joins and that composite indexes can be used through their leftmost prefixes. citeturn0search4

Example:

```sql
CREATE INDEX idx_employee_department
ON employees(department_id);
```

---

# 43. Composite Index

```sql
CREATE INDEX idx_orders_customer_date
ON orders(customer_id, order_date);
```

Conceptually useful for queries beginning with:

```text
customer_id
customer_id + order_date
```

The exact optimizer behavior is engine-specific.

---

# 44. Leftmost Prefix

For:

```sql
INDEX(a, b, c)
```

the index can generally support lookups using:

```text
a
a,b
a,b,c
```

But a query only filtering on `b` does not get the same direct leftmost-prefix benefit.

MySQL documents this leftmost-prefix behavior explicitly. citeturn0search4

---

# 45. When NOT to Add an Index

Indexes have costs.

They can increase:

- Storage
- INSERT cost
- UPDATE cost
- DELETE cost
- Maintenance work

Do not create an index for every column.

Consider:

```text
Query frequency
Selectivity
Data distribution
Write volume
Column cardinality
Composite access patterns
Execution plan
```

---

# 46. EXPLAIN

Use the optimizer's execution plan.

```sql
EXPLAIN
SELECT *
FROM employees
WHERE department_id = 10;
```

For MySQL:

```sql
EXPLAIN ANALYZE
SELECT ...
```

when supported by the target version.

Look for:

```text
Access type
Possible indexes
Chosen index
Rows examined/estimated
Join order
Filtering
Sorts
Temporary operations
```

Never claim an index improved a query merely because you created it.

Measure.

---

# 47. Query Optimization Checklist

When a query is slow:

```text
1. Understand the query.
2. Check EXPLAIN.
3. Identify expensive operation.
4. Check indexes.
5. Check JOIN cardinality.
6. Reduce unnecessary rows early.
7. Avoid SELECT *.
8. Avoid unnecessary DISTINCT.
9. Check sort/group operations.
10. Check functions on indexed columns.
11. Check data volume and statistics.
12. Test alternatives.
13. Measure before/after.
```

---

# 48. SARGability

A predicate is generally more index-friendly when the database can use the indexed column directly rather than computing a function over every row.

Less index-friendly pattern:

```sql
WHERE YEAR(created_at) = 2026
```

Potentially better:

```sql
WHERE created_at >= '2026-01-01'
  AND created_at <  '2027-01-01'
```

Exact date syntax and optimizer behavior depend on the DBMS.

---

# 49. UNION vs UNION ALL

### UNION

Combines results and removes duplicates.

```sql
SELECT city FROM customers
UNION
SELECT city FROM suppliers;
```

### UNION ALL

Combines results without duplicate elimination.

```sql
SELECT city FROM customers
UNION ALL
SELECT city FROM suppliers;
```

If duplicates are valid and not required to be removed, `UNION ALL` avoids the duplicate-elimination work.

---

# 50. DELETE vs TRUNCATE vs DROP

### DELETE

```sql
DELETE FROM employees
WHERE department_id = 10;
```

Removes rows and can use a `WHERE` condition.

### TRUNCATE

```sql
TRUNCATE TABLE employees;
```

Removes all rows using database-specific DDL/transaction semantics.

### DROP

```sql
DROP TABLE employees;
```

Removes the table object itself.

Do not memorize one universal rollback rule; behavior depends on the DBMS and transaction context.

---

# 51. UPDATE Safely

Before:

```sql
UPDATE employees
SET salary = salary * 1.10
WHERE department_id = 10;
```

Verify the target set first:

```sql
SELECT *
FROM employees
WHERE department_id = 10;
```

For important changes:

```sql
START TRANSACTION;

UPDATE ...;

SELECT ...;

COMMIT;
```

or:

```sql
ROLLBACK;
```

depending on the operation and environment.

---

# 52. Common Interview Queries

## Q1. Second highest salary

Simple version:

```sql
SELECT MAX(salary)
FROM employees
WHERE salary < (
    SELECT MAX(salary)
    FROM employees
);
```

This handles distinct salary values, but returns NULL if no second distinct salary exists.

Window approach:

```sql
SELECT salary
FROM (
    SELECT
        salary,
        DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk
    FROM employees
) x
WHERE rnk = 2;
```

---

## Q2. Nth highest salary

```sql
SELECT salary
FROM (
    SELECT
        salary,
        DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk
    FROM employees
) x
WHERE rnk = 5;
```

Clarify whether "Nth highest" means:

```text
Nth distinct salary
```

or:

```text
Nth row after sorting
```

Those are different questions.

---

## Q3. Highest salary in each department

```sql
SELECT *
FROM (
    SELECT
        e.*,
        DENSE_RANK() OVER (
            PARTITION BY department_id
            ORDER BY salary DESC
        ) AS rnk
    FROM employees e
) x
WHERE rnk = 1;
```

---

## Q4. Employees earning more than department average

```sql
SELECT *
FROM (
    SELECT
        e.*,
        AVG(salary) OVER (
            PARTITION BY department_id
        ) AS dept_avg
    FROM employees e
) x
WHERE salary > dept_avg;
```

---

## Q5. Find duplicate emails

```sql
SELECT
    email,
    COUNT(*) AS cnt
FROM users
GROUP BY email
HAVING COUNT(*) > 1;
```

---

## Q6. Find employees without a department

```sql
SELECT e.*
FROM employees e
LEFT JOIN departments d
    ON e.department_id = d.department_id
WHERE d.department_id IS NULL;
```

---

## Q7. Customers with no orders

```sql
SELECT c.*
FROM customers c
LEFT JOIN orders o
    ON o.customer_id = c.customer_id
WHERE o.customer_id IS NULL;
```

Alternative:

```sql
SELECT c.*
FROM customers c
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
);
```

---

## Q8. Latest order for each customer

```sql
SELECT *
FROM (
    SELECT
        o.*,
        ROW_NUMBER() OVER (
            PARTITION BY customer_id
            ORDER BY order_date DESC, order_id DESC
        ) AS rn
    FROM orders o
) x
WHERE rn = 1;
```

Include a deterministic tie-breaker such as `order_id` when required.

---

## Q9. Top 3 products by sales

```sql
SELECT
    product_id,
    SUM(amount) AS total_sales
FROM sales
GROUP BY product_id
ORDER BY total_sales DESC
LIMIT 3;
```

If the requirement means top 3 including ties, use a ranking strategy instead.

---

## Q10. Employees hired in the last 30 days

The exact expression is DB-specific.

Concept:

```sql
WHERE hire_date >= CURRENT_DATE - INTERVAL 30 DAY
```

Always check the target database's date syntax.

---

# 53. More Advanced Query Patterns

## Running total

```sql
SELECT
    order_date,
    amount,
    SUM(amount) OVER (
        ORDER BY order_date, order_id
    ) AS running_total
FROM orders;
```

---

## Previous row

```sql
SELECT
    order_id,
    order_date,
    amount,
    LAG(amount) OVER (
        ORDER BY order_date, order_id
    ) AS previous_amount
FROM orders;
```

---

## Difference from previous row

```sql
SELECT
    order_id,
    amount,
    amount -
    LAG(amount) OVER (
        ORDER BY order_date, order_id
    ) AS difference
FROM orders;
```

---

## Percentage of department total

```sql
SELECT
    employee_id,
    department_id,
    salary,
    salary * 100.0 /
        SUM(salary) OVER (
            PARTITION BY department_id
        ) AS pct_of_department_salary
FROM employees;
```

---

# 54. SQL Injection

Bad:

```text
Application input
      ↓
String concatenation
      ↓
SQL query
```

Example of dangerous construction:

```text
"SELECT * FROM users WHERE name = '" + userInput + "'"
```

Use parameterized queries / prepared statements.

Concept:

```text
SQL structure
      +
parameters
```

rather than concatenating untrusted values into SQL text.

---

# 55. Transactions and Application Design

Do not hold transactions open unnecessarily.

Long transactions can:

- Hold locks
- Increase contention
- Delay cleanup
- Increase resource usage
- Increase deadlock opportunities

Prefer:

```text
Begin
 ↓
Perform required work
 ↓
Validate
 ↓
Commit
```

Keep transaction scope appropriate to the business operation.

---

# 56. Most Asked SQL Interview Questions

### Fundamentals

1. What is SQL?
2. SQL vs MySQL?
3. What are DDL, DML, DCL and TCL?
4. Primary key vs foreign key?
5. Primary key vs unique key?
6. What is a composite key?
7. What is NULL?
8. What is normalization?
9. What are constraints?
10. What is referential integrity?

### Querying

11. WHERE vs HAVING?
12. GROUP BY?
13. DISTINCT?
14. UNION vs UNION ALL?
15. DELETE vs TRUNCATE vs DROP?
16. IN vs EXISTS?
17. JOIN vs subquery?
18. Correlated subquery?
19. CTE?
20. Recursive CTE?

### Joins

21. INNER JOIN?
22. LEFT JOIN?
23. RIGHT JOIN?
24. FULL OUTER JOIN?
25. CROSS JOIN?
26. SELF JOIN?
27. Why do joins create duplicates?
28. How do you find unmatched rows?
29. LEFT JOIN + WHERE trap?
30. How do you identify join cardinality?

### Window Functions

31. What is a window function?
32. ROW_NUMBER vs RANK?
33. RANK vs DENSE_RANK?
34. PARTITION BY?
35. LAG vs LEAD?
36. Running total?
37. Top N per group?
38. Latest row per group?
39. Moving average?
40. Window vs GROUP BY?

### Performance

41. What is an index?
42. Clustered vs non-clustered index?
43. Composite index?
44. Leftmost-prefix rule?
45. What is selectivity?
46. What is a covering index?
47. What is EXPLAIN?
48. What is a full table scan?
49. What is SARGability?
50. Why can an index make writes slower?

### Transactions

51. What is ACID?
52. What is a transaction?
53. COMMIT vs ROLLBACK?
54. Isolation levels?
55. Dirty read?
56. Non-repeatable read?
57. Phantom read?
58. Deadlock?
59. Lock?
60. Optimistic vs pessimistic concurrency?

### Real-world scenarios

61. Query suddenly became slow. What do you check?
62. Duplicate rows after a JOIN. Why?
63. Database CPU is high. What do you investigate?
64. How do you find the latest record per customer?
65. How do you find customers without orders?
66. How do you find duplicate records?
67. How do you delete duplicates safely?
68. How do you find the second-highest salary?
69. How do you find top N per department?
70. How do you optimize a query without changing the result?

---

# 57. Scenario Answer: Slow Query

A strong answer:

```text
1. Reproduce the query.
2. Check execution plan.
3. Check rows examined.
4. Check indexes.
5. Check JOIN cardinality.
6. Check filters and selectivity.
7. Check sorting/grouping.
8. Check functions on indexed columns.
9. Check table statistics/data growth.
10. Test an alternative.
11. Measure before/after.
```

Use:

```sql
EXPLAIN ...
```

and, where supported:

```sql
EXPLAIN ANALYZE ...
```

Do not say:

> "I will add an index."

Say:

> "I will inspect the execution plan first and determine whether the bottleneck is access path, join strategy, sorting, aggregation, cardinality, or something outside the query."

---

# 58. Scenario Answer: Duplicate Rows

Strong answer:

```text
1. Check the JOIN condition.
2. Identify primary/unique keys.
3. Determine relationship cardinality.
4. Check whether one-to-many is expected.
5. Determine whether aggregation is required.
6. Fix the query logic.
7. Use DISTINCT only if duplicate result rows are legitimately intended to collapse.
```

---

# 59. Scenario Answer: Delete Duplicate Records

First identify duplicates:

```sql
SELECT
    email,
    COUNT(*)
FROM users
GROUP BY email
HAVING COUNT(*) > 1;
```

Then decide which row should survive.

Example pattern:

```sql
WITH ranked AS (
    SELECT
        user_id,
        ROW_NUMBER() OVER (
            PARTITION BY email
            ORDER BY user_id
        ) AS rn
    FROM users
)
DELETE FROM users
WHERE user_id IN (
    SELECT user_id
    FROM ranked
    WHERE rn > 1
);
```

**Important:** `DELETE` syntax involving CTEs varies by database. Test the equivalent syntax for your DBMS before production use.

Always:

```text
SELECT candidates
    ↓
Review
    ↓
Transaction
    ↓
Delete
    ↓
Verify
```

---

# 60. Scenario Answer: Database CPU High

Do not assume one query is responsible.

Investigate:

```text
Active queries
Execution plans
Rows scanned
Missing/ineffective indexes
Large sorts
Large joins
Lock contention
Connection volume
Recent deployments
Data growth
```

MySQL example tools include:

```sql
SHOW PROCESSLIST;
EXPLAIN ...;
```

and Performance Schema depending on the environment.

---

# 61. Query Quality Checklist

Before submitting a query:

```text
[ ] Correct tables?
[ ] Correct JOIN condition?
[ ] Correct cardinality?
[ ] NULL handled?
[ ] Duplicate rows expected?
[ ] WHERE vs HAVING correct?
[ ] GROUP BY correct?
[ ] Window function required?
[ ] Tie behavior defined?
[ ] Deterministic ordering?
[ ] Date boundary correct?
[ ] Index-friendly predicate?
[ ] EXPLAIN checked if performance matters?
```

---

# 62. SQL Interview Traps

### Trap 1

`COUNT(*)` ≠ `COUNT(column)`

---

### Trap 2

`NULL = NULL` is not TRUE.

Use:

```sql
IS NULL
```

---

### Trap 3

`NOT IN` + NULL can produce surprising results.

Consider:

```sql
NOT EXISTS
```

---

### Trap 4

`WHERE` filters rows.

`HAVING` filters groups.

---

### Trap 5

`GROUP BY` collapses rows.

Window functions generally preserve the row-level result.

---

### Trap 6

`RANK()` can produce gaps.

`DENSE_RANK()` does not.

---

### Trap 7

A `LEFT JOIN` can effectively behave like an inner join if right-table predicates are placed incorrectly in `WHERE`.

---

### Trap 8

`DISTINCT` is not a universal duplicate fix.

It can hide a bad join.

---

### Trap 9

An index does not guarantee a faster query.

The optimizer may choose another access path.

---

### Trap 10

A query that works on 10,000 rows may fail operationally at 100 million rows.

Always think about:

```text
data volume
cardinality
indexes
execution plan
memory
I/O
```

---

# 63. Fast Last-Minute Cheat Sheet

## Filtering

```sql
WHERE
IN
BETWEEN
LIKE
IS NULL
IS NOT NULL
```

## Aggregation

```sql
GROUP BY
HAVING
COUNT
SUM
AVG
MIN
MAX
```

## Joins

```sql
INNER
LEFT
RIGHT
FULL
CROSS
SELF
```

## Advanced

```sql
CTE
RECURSIVE CTE
EXISTS
NOT EXISTS
CASE
COALESCE
NULLIF
WINDOW FUNCTIONS
```

## Window

```sql
ROW_NUMBER
RANK
DENSE_RANK
LAG
LEAD
SUM() OVER
AVG() OVER
```

## Performance

```sql
EXPLAIN
EXPLAIN ANALYZE
INDEX
COMPOSITE INDEX
COVERING INDEX
SARGABILITY
CARDINALITY
SELECTIVITY
```

## Transactions

```sql
START TRANSACTION
COMMIT
ROLLBACK
SAVEPOINT
```

## Must-know concepts

```text
NULL
3-valued logic
JOIN cardinality
ACID
Isolation
Locks
Deadlocks
Indexes
Execution plans
Normalization
SQL injection
```

---

# 64. Reference Links

## Official Documentation

- MySQL Reference Manual: https://dev.mysql.com/doc/refman/9.7/en/
- MySQL JOIN documentation: https://dev.mysql.com/doc/refman/8.4/en/join.html
- MySQL Window Functions: https://dev.mysql.com/doc/refman/9.7/en/window-functions.html
- MySQL Indexes: https://dev.mysql.com/doc/refman/9.7/en/mysql-indexes.html
- PostgreSQL Documentation: https://www.postgresql.org/docs/
- SQLite Documentation: https://www.sqlite.org/docs.html

Use the documentation for the **specific database engine** used by the interview or project. SQL syntax and behavior are not identical across vendors.

---

# 65. Final Interview Strategy

When asked to solve a SQL problem:

```text
1. Clarify the output.
2. Identify tables.
3. Identify keys and relationships.
4. Decide JOIN type.
5. Filter rows.
6. Aggregate if needed.
7. Use window functions when row-level context must be preserved.
8. Handle NULL.
9. Handle duplicates.
10. Define tie behavior.
11. Test edge cases.
12. Explain complexity/performance.
```

For optimization questions:

```text
Don't guess.
Don't blindly add indexes.
Don't blindly add DISTINCT.

EXPLAIN
    ↓
Understand
    ↓
Change
    ↓
Measure
    ↓
Verify
```

> **SQL skill is not memorizing syntax. It is translating a business question into correct relational logic and then making that logic efficient enough for real data.**
