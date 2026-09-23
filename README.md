# SQL_Last_Minute_Note
SQL-Last-Minute-Revision/

# SQL Last-Minute Revision 🚀

<p align="center">
  <b>A practical and visual SQL revision guide for interviews, coding rounds, and quick refresh before technical discussions.</b>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/SQL-Revision-blue?style=for-the-badge&logo=mysql&logoColor=white" alt="SQL Revision">
  <img src="https://img.shields.io/badge/Interview-Preparation-orange?style=for-the-badge" alt="Interview Preparation">
  <img src="https://img.shields.io/badge/Beginner-Friendly-green?style=for-the-badge" alt="Beginner Friendly">
</p>

---

## 📌 About This Repository

SQL is easy to learn but surprisingly easy to forget.

This repository is designed as a **last-minute SQL revision handbook** covering the concepts and query patterns that frequently appear in technical interviews and SQL-based assessments.

Instead of going through lengthy tutorials before an interview, you can use this repository to quickly revise:

* 📖 Core SQL concepts
* 🔎 Filtering and sorting
* 📊 Aggregate functions
* 🗂️ `GROUP BY` and `HAVING`
* 🔗 SQL Joins
* 🧩 Subqueries and CTEs
* 🪟 Window Functions
* 🔐 Keys and Constraints
* 🧱 Normalization
* ⚡ Indexes and query performance
* 🔄 Transactions and ACID
* 🎯 Frequently asked interview queries
* ⚠️ Common SQL mistakes and tricky concepts

> **Learn the concept → Understand the pattern → Write the query → Practice the variation.**

---

# 📚 Table of Contents

1. [SQL Basics](#-sql-basics)
2. [Filtering & Sorting](#-filtering--sorting)
3. [Aggregate Functions](#-aggregate-functions)
4. [GROUP BY & HAVING](#-group-by--having)
5. [Joins](#-joins)
6. [Subqueries](#-subqueries)
7. [CTEs](#-common-table-expressions-ctes)
8. [Window Functions](#-window-functions)
9. [Keys & Constraints](#-keys--constraints)
10. [Normalization](#-normalization)
11. [Indexes](#-indexes)
12. [Transactions & ACID](#-transactions--acid)
13. [Common Interview Queries](#-common-sql-interview-queries)
14. [Tricky SQL Questions](#-tricky-sql-concepts)
15. [Quick Revision Checklist](#-quick-revision-checklist)

---

# 🟢 SQL Basics

Before solving complex queries, understand the basic building blocks of SQL.

### Topics

* What is SQL?
* SQL vs MySQL
* Database vs Table
* Rows and Columns
* `SELECT`
* `DISTINCT`
* `FROM`
* `WHERE`
* `ORDER BY`
* `LIMIT`
* SQL comments
* SQL data types

### Example

```sql
SELECT name, salary
FROM employees
WHERE salary > 50000
ORDER BY salary DESC;
```

---

# 🔎 Filtering & Sorting

Learn how to retrieve exactly the data you need.

### Important Operators

```text
=       Equal
<>      Not equal
>       Greater than
<       Less than
>=      Greater than or equal
<=      Less than or equal
BETWEEN Range
IN      Match multiple values
LIKE    Pattern matching
IS NULL NULL checking
```

### Example

```sql
SELECT *
FROM employees
WHERE department IN ('IT', 'HR')
  AND salary BETWEEN 40000 AND 80000;
```

---

# 📊 Aggregate Functions

Aggregate functions perform calculations across multiple rows.

| Function  | Purpose           |
| --------- | ----------------- |
| `COUNT()` | Count rows        |
| `SUM()`   | Calculate total   |
| `AVG()`   | Calculate average |
| `MIN()`   | Find minimum      |
| `MAX()`   | Find maximum      |

### Example

```sql
SELECT
    department,
    COUNT(*) AS employee_count,
    AVG(salary) AS average_salary,
    MAX(salary) AS highest_salary
FROM employees
GROUP BY department;
```

---

# 🗂️ GROUP BY & HAVING

This is one of the most important areas for SQL interviews.

### Remember

```text
WHERE
  ↓
Filters individual rows

GROUP BY
  ↓
Creates groups

HAVING
  ↓
Filters groups
```

### Example

```sql
SELECT
    department,
    COUNT(*) AS employee_count
FROM employees
GROUP BY department
HAVING COUNT(*) > 5;
```

### Interview Trap

❌ Incorrect:

```sql
WHERE COUNT(*) > 5
```

✅ Correct:

```sql
HAVING COUNT(*) > 5
```

---

# 🔗 Joins

Joins combine data from multiple tables.

### Types of Joins

```text
INNER JOIN
LEFT JOIN
RIGHT JOIN
FULL OUTER JOIN
CROSS JOIN
SELF JOIN
```

### Visual Overview

```text
INNER JOIN
     A ∩ B

LEFT JOIN
     A + matching B

RIGHT JOIN
     B + matching A

FULL JOIN
     A ∪ B
```

### Example

```sql
SELECT
    e.name,
    d.department_name
FROM employees e
INNER JOIN departments d
    ON e.department_id = d.department_id;
```

### Quick Memory

> **INNER JOIN → Matching rows**

> **LEFT JOIN → Everything from the left table + matches**

> **RIGHT JOIN → Everything from the right table + matches**

> **FULL JOIN → Everything from both tables**

---

# 🧩 Subqueries

A subquery is a query inside another query.

### Example

Find employees earning more than the average salary:

```sql
SELECT name, salary
FROM employees
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
);
```

### Common Types

* Scalar subquery
* Single-row subquery
* Multi-row subquery
* Correlated subquery
* Nested subquery

---

# 🧱 Common Table Expressions (CTEs)

CTEs allow you to create a temporary named result set that can be referenced by the main query.

```sql
WITH department_salary AS (
    SELECT
        department,
        AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department
)
SELECT *
FROM department_salary
WHERE avg_salary > 60000;
```

### Why use CTEs?

* Improve readability
* Break complex queries into steps
* Make debugging easier
* Useful for recursive queries

---

# 🪟 Window Functions

Window functions are one of the most important topics for modern SQL interviews.

Unlike `GROUP BY`, window functions **do not collapse rows**.

### Important Functions

```sql
ROW_NUMBER()
RANK()
DENSE_RANK()
NTILE()
LEAD()
LAG()
FIRST_VALUE()
LAST_VALUE()
SUM() OVER()
AVG() OVER()
COUNT() OVER()
```

### Example

Find salary ranking within each department:

```sql
SELECT
    name,
    department,
    salary,
    DENSE_RANK() OVER (
        PARTITION BY department
        ORDER BY salary DESC
    ) AS salary_rank
FROM employees;
```

### RANK vs DENSE_RANK

| Salary | RANK | DENSE_RANK |
| -----: | ---: | ---------: |
| 100000 |    1 |          1 |
|  90000 |    2 |          2 |
|  90000 |    2 |          2 |
|  80000 |    4 |          3 |

> `RANK()` leaves gaps after ties.
> `DENSE_RANK()` does not.

---

# 🔐 Keys & Constraints

### Important Keys

* Primary Key
* Foreign Key
* Candidate Key
* Composite Key
* Alternate Key
* Unique Key

### Common Constraints

```sql
PRIMARY KEY
FOREIGN KEY
UNIQUE
NOT NULL
CHECK
DEFAULT
```

Example:

```sql
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(150) UNIQUE,
    salary DECIMAL(10,2) CHECK (salary > 0),
    department_id INT,
    FOREIGN KEY (department_id)
        REFERENCES departments(department_id)
);
```

---

# 🧱 Normalization

Normalization organizes data to reduce redundancy and improve data integrity.

### Important Normal Forms

```text
1NF
 ↓
2NF
 ↓
3NF
 ↓
BCNF
```

### Quick Understanding

**1NF**

* Atomic values
* No repeating groups

**2NF**

* Must be in 1NF
* No partial dependency on a composite key

**3NF**

* Must be in 2NF
* No transitive dependency

---

# ⚡ Indexes

Indexes help databases find rows more efficiently.

Think of an index like the **index of a book**:

```text
Without Index
Database → Scan many rows → Find data

With Index
Database → Index → Locate rows → Fetch data
```

### Important Topics

* Clustered Index
* Non-clustered Index
* Composite Index
* Unique Index
* Index Selectivity
* When indexes help
* When indexes can hurt performance

> Indexes can improve read performance, but they also require storage and can add overhead to `INSERT`, `UPDATE`, and `DELETE` operations.

---

# 🔄 Transactions & ACID

A transaction is a logical unit of database work.

### ACID

```text
A → Atomicity
C → Consistency
I → Isolation
D → Durability
```

### Common Commands

```sql
BEGIN;
COMMIT;
ROLLBACK;
SAVEPOINT;
```

### Example

```sql
BEGIN;

UPDATE accounts
SET balance = balance - 1000
WHERE account_id = 101;

UPDATE accounts
SET balance = balance + 1000
WHERE account_id = 102;

COMMIT;
```

If something goes wrong:

```sql
ROLLBACK;
```

---

# 🎯 Common SQL Interview Queries

These patterns are worth practicing repeatedly.

### 1. Second Highest Salary

```sql
SELECT MAX(salary)
FROM employees
WHERE salary < (
    SELECT MAX(salary)
    FROM employees
);
```

### 2. Duplicate Records

```sql
SELECT email, COUNT(*)
FROM employees
GROUP BY email
HAVING COUNT(*) > 1;
```

### 3. Highest Salary in Each Department

```sql
SELECT department, MAX(salary)
FROM employees
GROUP BY department;
```

### 4. Top 3 Salaries Per Department

```sql
SELECT *
FROM (
    SELECT
        name,
        department,
        salary,
        DENSE_RANK() OVER (
            PARTITION BY department
            ORDER BY salary DESC
        ) AS rnk
    FROM employees
) t
WHERE rnk <= 3;
```

### 5. Employees Earning More Than Their Manager

```sql
SELECT
    e.name AS employee,
    e.salary AS employee_salary,
    m.name AS manager,
    m.salary AS manager_salary
FROM employees e
JOIN employees m
    ON e.manager_id = m.employee_id
WHERE e.salary > m.salary;
```

---

# ⚠️ Tricky SQL Concepts

These are common areas where candidates make mistakes:

* `NULL` vs `0`
* `COUNT(*)` vs `COUNT(column)`
* `WHERE` vs `HAVING`
* `UNION` vs `UNION ALL`
* `DELETE` vs `TRUNCATE` vs `DROP`
* `RANK()` vs `DENSE_RANK()`
* `INNER JOIN` vs `LEFT JOIN`
* `IN` vs `EXISTS`
* `NOT IN` with `NULL`
* `COUNT(DISTINCT column)`
* Duplicate rows after joins
* Filtering before vs after aggregation
* Window functions vs `GROUP BY`

---

# 🧠 SQL Query Execution Order

One of the most useful things to remember before an interview:

```text
FROM
  ↓
JOIN
  ↓
WHERE
  ↓
GROUP BY
  ↓
HAVING
  ↓
SELECT
  ↓
DISTINCT
  ↓
ORDER BY
  ↓
LIMIT
```

Remember:

> **The order you write SQL is not necessarily the order in which the database logically processes it.**

---

# ⚡ Quick Revision Checklist

### Fundamentals

* [ ] SELECT
* [ ] DISTINCT
* [ ] WHERE
* [ ] ORDER BY
* [ ] LIMIT
* [ ] NULL
* [ ] LIKE
* [ ] IN
* [ ] BETWEEN

### Aggregation

* [ ] COUNT
* [ ] SUM
* [ ] AVG
* [ ] MIN
* [ ] MAX
* [ ] GROUP BY
* [ ] HAVING

### Joins

* [ ] INNER JOIN
* [ ] LEFT JOIN
* [ ] RIGHT JOIN
* [ ] FULL JOIN
* [ ] CROSS JOIN
* [ ] SELF JOIN

### Advanced SQL

* [ ] Subqueries
* [ ] CTEs
* [ ] CASE
* [ ] UNION
* [ ] EXISTS
* [ ] Window Functions
* [ ] ROW_NUMBER
* [ ] RANK
* [ ] DENSE_RANK
* [ ] LEAD
* [ ] LAG

### Database Concepts

* [ ] Primary Key
* [ ] Foreign Key
* [ ] Constraints
* [ ] Normalization
* [ ] Indexes
* [ ] Transactions
* [ ] ACID
* [ ] Views

---

# 🎤 Before Your SQL Interview

Focus on these first:

```text
1. Joins
2. GROUP BY + HAVING
3. Subqueries
4. CTEs
5. Window Functions
6. Aggregate Functions
7. NULL handling
8. Common interview queries
9. DELETE vs TRUNCATE vs DROP
10. Primary Key / Foreign Key / Indexes
```

Then solve problems without looking at the answer.

The goal isn't to memorize 100 queries.

The goal is to recognize the **query pattern** and adapt it to a new problem.

---

# 🤝 Contributing

Found an error or want to add a useful SQL interview problem?

Contributions are welcome.

1. Fork the repository
2. Create a new branch
3. Add or improve the content
4. Commit your changes
5. Open a Pull Request

---

# ⭐ Support

If you find this repository useful for your SQL preparation, consider giving it a ⭐.

Happy querying! 🚀

```sql
SELECT 'Keep Learning!' AS message;
```
