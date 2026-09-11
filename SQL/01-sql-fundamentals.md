# 01. SQL Fundamentals

> **Track:** SQL Interview Preparation  
> **Target:** FAANG + top product companies | SDE + ML/AI Engineer  
> **Level:** Beginner → Interview-ready foundation  
> **Scope:** Relational databases, tables, rows/columns, keys, NULL, data types, constraints, and core retrieval commands.

---

# 1. Why SQL Fundamentals Matter

SQL fundamentals are the base for every later SQL topic:

```text
SELECT
WHERE
JOIN
GROUP BY
HAVING
Subqueries
CTEs
Window Functions
Transactions
Indexes
Query Optimization
Analytical SQL
```

Before learning advanced SQL, you should be able to read a relational schema and write correct basic queries without hesitation.

---

# 2. Relational Databases

A **relational database** stores data in relations, commonly represented as tables.

A table consists of:

- rows
- columns

Example:

### Employee

| employee_id | name | department | salary |
|---:|---|---|---:|
| 1 | Alice | Engineering | 90000 |
| 2 | Bob | Sales | 70000 |
| 3 | Carol | Engineering | 95000 |

A row represents one record.

A column represents one attribute of the records.

---

# 3. Database Terminology

| Term | Meaning |
|---|---|
| Database | Organized collection of related data |
| Table | Relation containing rows and columns |
| Row | One record/tuple |
| Column | One attribute/field |
| Schema | Structure/definition of database objects |
| Primary Key | Attribute(s) uniquely identifying a row |
| Foreign Key | Attribute(s) referencing a key in another table |
| Constraint | Rule restricting valid data |
| NULL | Missing/unknown/not-applicable value |

---

# 4. Tables

A table represents a relation.

Example:

```sql
CREATE TABLE employees (
    employee_id INT,
    name VARCHAR(100),
    department VARCHAR(100),
    salary DECIMAL(10, 2)
);
```

Insert records:

```sql
INSERT INTO employees
(employee_id, name, department, salary)
VALUES
(1, 'Alice', 'Engineering', 90000),
(2, 'Bob', 'Sales', 70000),
(3, 'Carol', 'Engineering', 95000);
```

Query:

```sql
SELECT *
FROM employees;
```

---

# 5. Rows

A row is one complete record.

Example:

```text
(1, 'Alice', 'Engineering', 90000)
```

represents one employee.

SQL does not guarantee that rows are returned in insertion order unless an `ORDER BY` clause specifies the desired ordering.

Therefore:

```sql
SELECT *
FROM employees;
```

does **not** mean:

> Return employees in insertion order.

Use:

```sql
SELECT *
FROM employees
ORDER BY employee_id;
```

when ordering matters.

---

# 6. Columns

Columns define attributes.

Example:

```text
employee_id
name
department
salary
```

Selecting particular columns:

```sql
SELECT name, salary
FROM employees;
```

Avoid:

```sql
SELECT *
```

when you only need specific columns, especially in production queries.

---

# 7. Primary Key

A **primary key** uniquely identifies each row.

Properties:

1. unique
2. cannot be `NULL`
3. one primary-key constraint per table
4. can consist of multiple columns

Example:

```sql
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    name VARCHAR(100),
    salary DECIMAL(10, 2)
);
```

Now:

```text
employee_id = 1
```

uniquely identifies Alice.

---

# 8. Composite Primary Key

A primary key can contain multiple columns.

Example:

```sql
CREATE TABLE enrollments (
    student_id INT,
    course_id INT,
    enrolled_at DATE,
    PRIMARY KEY (student_id, course_id)
);
```

The pair:

```text
(student_id, course_id)
```

must be unique.

Individually:

```text
student_id
course_id
```

may repeat.

---

# 9. Foreign Key

A foreign key creates a referential relationship between tables.

Example:

```sql
CREATE TABLE departments (
    department_id INT PRIMARY KEY,
    department_name VARCHAR(100)
);

CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    name VARCHAR(100),
    department_id INT,
    FOREIGN KEY (department_id)
        REFERENCES departments(department_id)
);
```

Here:

```text
employees.department_id
        ↓
departments.department_id
```

The foreign key helps maintain **referential integrity**.

---

# 10. Primary Key vs Foreign Key

| Property | Primary Key | Foreign Key |
|---|---|---|
| Identifies row in its table | Yes | No |
| Must be unique | Yes | Not necessarily |
| Allows NULL | No | Can, depending on definition/business rule |
| References another table | No | Usually yes |
| Number per table | One constraint | Multiple possible |

---

# 11. NULL

`NULL` does not mean:

```text
0
```

or:

```text
''
```

or:

```text
FALSE
```

It represents an absent, unknown, or inapplicable value.

Example:

| employee_id | name | manager_id |
|---:|---|---:|
| 1 | Alice | NULL |

Alice may not have a manager.

---

# 12. NULL and Comparisons

This is one of the most important SQL fundamentals.

Incorrect:

```sql
WHERE manager_id = NULL
```

Correct:

```sql
WHERE manager_id IS NULL
```

For non-NULL:

```sql
WHERE manager_id IS NOT NULL
```

---

# 13. Three-Valued Logic

SQL uses:

```text
TRUE
FALSE
UNKNOWN
```

instead of only TRUE/FALSE.

For example:

```sql
salary = NULL
```

does not evaluate to `TRUE`.

It evaluates to `UNKNOWN`.

Therefore:

```sql
WHERE salary = NULL
```

does not return rows with `NULL` salaries.

Use:

```sql
WHERE salary IS NULL
```

---

# 14. NULL and Arithmetic

Suppose:

```text
salary = 50000
bonus = NULL
```

Then:

```sql
SELECT salary + bonus
```

normally produces:

```text
NULL
```

because an unknown value participates in the expression.

Use `COALESCE` later when you need a replacement:

```sql
SELECT salary + COALESCE(bonus, 0)
FROM employees;
```

---

# 15. NULL and Aggregation

Important behavior:

Most aggregate functions ignore `NULL` values.

For:

```text
salary
------
100
200
NULL
```

then:

```sql
SELECT AVG(salary)
```

uses:

```text
100, 200
```

not the `NULL`.

`COUNT(*)` counts rows.

```sql
SELECT COUNT(*)
FROM employees;
```

`COUNT(column)` counts non-NULL values.

```sql
SELECT COUNT(manager_id)
FROM employees;
```

This distinction becomes important in analytical SQL.

---

# 16. SQL Data Types

Common categories:

### Integer

```sql
INT
BIGINT
SMALLINT
```

### Decimal

```sql
DECIMAL(p, s)
NUMERIC(p, s)
```

Useful for exact numeric values such as money.

### Floating Point

Depending on the DBMS:

```sql
FLOAT
REAL
DOUBLE
```

### Character

```sql
CHAR(n)
VARCHAR(n)
TEXT
```

### Date/Time

Common types include:

```sql
DATE
TIME
TIMESTAMP
```

Exact type names and behavior can vary by DBMS.

---

# 17. CHAR vs VARCHAR

## CHAR

Fixed-length character data.

Example:

```sql
CHAR(10)
```

Conceptually stores fixed-width values.

## VARCHAR

Variable-length character data.

Example:

```sql
VARCHAR(100)
```

Usually preferable when strings have varying lengths.

Do not memorize implementation details as universal rules; storage and optimization behavior can depend on the database system.

---

# 18. DECIMAL vs FLOAT

For exact decimal values such as financial amounts:

```sql
DECIMAL
```

is generally preferable.

Floating-point values can have representation/rounding effects.

Example:

```sql
price DECIMAL(10, 2)
```

means precision/scale constraints are specified by the database system.

---

# 19. Constraints

Constraints enforce rules on table data.

Important constraints:

```text
PRIMARY KEY
FOREIGN KEY
NOT NULL
UNIQUE
CHECK
DEFAULT
```

Example:

```sql
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    email VARCHAR(255) UNIQUE,
    name VARCHAR(100) NOT NULL,
    salary DECIMAL(10, 2) CHECK (salary >= 0),
    country VARCHAR(50) DEFAULT 'India'
);
```

---

# 20. NOT NULL

Ensures a column cannot contain `NULL`.

```sql
name VARCHAR(100) NOT NULL
```

This means every inserted row must provide a valid non-NULL value for `name`, unless the database/schema provides another mechanism that satisfies the constraint.

---

# 21. UNIQUE

Ensures uniqueness according to the database's constraint semantics.

Example:

```sql
email VARCHAR(255) UNIQUE
```

A duplicate email cannot be inserted.

### Important

`UNIQUE` and `PRIMARY KEY` are not identical.

A table can have:

```text
one primary key
multiple unique constraints
```

NULL behavior for a `UNIQUE` constraint can vary across database systems, so check the target DBMS when the distinction matters.

---

# 22. CHECK

Restricts values according to a condition.

Example:

```sql
salary DECIMAL(10, 2)
CHECK (salary >= 0)
```

Another:

```sql
age INT CHECK (age >= 18)
```

The exact enforcement of `CHECK` constraints depends on the database system/version.

---

# 23. DEFAULT

Provides a default value when a value is omitted.

```sql
country VARCHAR(50) DEFAULT 'India'
```

Example:

```sql
INSERT INTO employees (employee_id, name)
VALUES (10, 'David');
```

If supported by the schema and omitted column semantics, `country` receives its default.

---

# 24. SELECT

`SELECT` retrieves data.

```sql
SELECT name, salary
FROM employees;
```

Select all columns:

```sql
SELECT *
FROM employees;
```

Select expressions:

```sql
SELECT name, salary * 12 AS annual_salary
FROM employees;
```

---

# 25. FROM

`FROM` identifies the source table or relation.

```sql
SELECT name
FROM employees;
```

Conceptually:

```text
FROM
→ choose source
```

More complex SQL can have:

```sql
FROM employees
JOIN departments ...
```

---

# 26. WHERE

`WHERE` filters rows.

```sql
SELECT *
FROM employees
WHERE salary > 80000;
```

Multiple conditions:

```sql
SELECT *
FROM employees
WHERE department = 'Engineering'
  AND salary > 80000;
```

Using `OR`:

```sql
SELECT *
FROM employees
WHERE department = 'Engineering'
   OR department = 'Sales';
```

---

# 27. Comparison Operators

Common operators:

```text
=
<>
!=
>
<
>=
<=
```

Example:

```sql
SELECT *
FROM employees
WHERE salary >= 80000;
```

For portable SQL, prefer `<>` for "not equal"; `!=` is also supported by many systems.

---

# 28. Logical Operators

### AND

Both conditions must be true.

```sql
WHERE salary > 50000
  AND department = 'Engineering'
```

### OR

At least one condition is true.

```sql
WHERE department = 'Engineering'
   OR department = 'Sales'
```

### NOT

Negates a condition.

```sql
WHERE NOT department = 'Sales'
```

---

# 29. Operator Precedence

A common logical precedence is:

```text
NOT
AND
OR
```

Therefore:

```sql
WHERE A OR B AND C
```

is generally interpreted as:

```sql
WHERE A OR (B AND C)
```

Use parentheses when the intended logic is not immediately obvious:

```sql
WHERE (A OR B) AND C
```

This is safer and easier to read.

---

# 30. DISTINCT

Removes duplicate rows from the selected result.

```sql
SELECT DISTINCT department
FROM employees;
```

For multiple columns:

```sql
SELECT DISTINCT department, job_title
FROM employees;
```

Distinctness applies to the **combination of selected expressions**.

---

# 31. ORDER BY

Sorts the result.

Ascending:

```sql
SELECT *
FROM employees
ORDER BY salary ASC;
```

Descending:

```sql
SELECT *
FROM employees
ORDER BY salary DESC;
```

`ASC` is commonly the default.

---

# 32. Multiple ORDER BY Columns

```sql
SELECT *
FROM employees
ORDER BY department ASC, salary DESC;
```

Interpretation:

1. sort by department ascending
2. within each department, sort by salary descending

---

# 33. Deterministic Ordering

If ties exist:

```sql
ORDER BY salary DESC
```

does not fully specify the order of equal salaries.

Add a tie-breaker:

```sql
ORDER BY salary DESC, employee_id ASC;
```

This is important for interview queries involving:

- top N
- pagination
- ranking
- deterministic output

---

# 34. LIMIT

`LIMIT` restricts the number of returned rows in systems that support it.

```sql
SELECT *
FROM employees
ORDER BY salary DESC
LIMIT 5;
```

This returns at most five rows.

### DBMS Note

Pagination syntax differs across database systems. For example, some systems use `TOP` or `FETCH FIRST` instead of `LIMIT`.

---

# 35. OFFSET

`OFFSET` skips rows before returning results.

```sql
SELECT *
FROM employees
ORDER BY employee_id
LIMIT 10 OFFSET 20;
```

Conceptually:

```text
skip 20
return next 10
```

Always pair pagination with a meaningful `ORDER BY`.

---

# 36. LIMIT + OFFSET Pagination

Page size:

```text
10
```

Page 1:

```sql
LIMIT 10 OFFSET 0
```

Page 2:

```sql
LIMIT 10 OFFSET 10
```

Page 3:

```sql
LIMIT 10 OFFSET 20
```

Formula:

```text
OFFSET = (page_number - 1) × page_size
```

---

# 37. Why OFFSET Pagination Can Become Expensive

For large offsets, the database may still need to process/skip many rows.

For large-scale systems, **keyset/seek pagination** can often be more efficient.

Example:

```sql
SELECT *
FROM employees
WHERE employee_id > 1000
ORDER BY employee_id
LIMIT 10;
```

This requires an appropriate ordering/index strategy.

Keyset pagination will be covered in more advanced SQL topics.

---

# 38. SQL Query Order vs Logical Processing Order

The written order is commonly:

```sql
SELECT
FROM
WHERE
ORDER BY
LIMIT
```

But SQL's **logical query processing** is conceptually closer to:

```text
FROM
WHERE
GROUP BY
HAVING
SELECT
DISTINCT
ORDER BY
LIMIT/OFFSET
```

The exact logical model has additional nuance, but this ordering is essential for understanding later topics.

---

# 39. Why This Matters

Consider:

```sql
SELECT department, COUNT(*)
FROM employees
WHERE salary > 50000
GROUP BY department
HAVING COUNT(*) >= 5
ORDER BY COUNT(*) DESC;
```

Conceptually:

1. choose `employees`
2. filter salaries
3. form groups
4. filter groups
5. produce selected output
6. sort output

This distinction becomes critical when learning:

- aliases
- aggregation
- `GROUP BY`
- `HAVING`
- window functions

---

# 40. Aliases

Column alias:

```sql
SELECT
    salary * 12 AS annual_salary
FROM employees;
```

Table alias:

```sql
SELECT e.name
FROM employees AS e;
```

Aliases improve readability and become essential with joins.

---

# 41. Basic Query Patterns

## Select everything

```sql
SELECT *
FROM employees;
```

## Select columns

```sql
SELECT name, department
FROM employees;
```

## Filter

```sql
SELECT name
FROM employees
WHERE department = 'Engineering';
```

## Sort

```sql
SELECT name, salary
FROM employees
ORDER BY salary DESC;
```

## Top N

```sql
SELECT name, salary
FROM employees
ORDER BY salary DESC
LIMIT 5;
```

## Distinct values

```sql
SELECT DISTINCT department
FROM employees;
```

## Pagination

```sql
SELECT *
FROM employees
ORDER BY employee_id
LIMIT 20 OFFSET 40;
```

---

# 42. Important SQL Predicates

Although later topics will cover these in depth, recognize them now.

## BETWEEN

```sql
WHERE salary BETWEEN 50000 AND 80000
```

`BETWEEN` is generally inclusive of both endpoints.

Equivalent conceptually:

```sql
salary >= 50000
AND salary <= 80000
```

## IN

```sql
WHERE department IN ('Engineering', 'Sales')
```

Useful for membership tests.

## LIKE

```sql
WHERE name LIKE 'A%'
```

Common wildcards:

```text
%  → zero or more characters
_  → one character
```

## IS NULL

```sql
WHERE manager_id IS NULL
```

## IS NOT NULL

```sql
WHERE manager_id IS NOT NULL
```

---

# 43. SQL Comments

Single-line:

```sql
-- Get employees with high salaries
SELECT *
FROM employees
WHERE salary > 100000;
```

Multi-line syntax commonly supported:

```sql
/*
  Employee query
*/
SELECT *
FROM employees;
```

Exact comment support is broadly standard but can vary in edge cases by DBMS.

---

# 44. INSERT, UPDATE, DELETE

These are core SQL commands even though this topic focuses primarily on retrieval.

## INSERT

```sql
INSERT INTO employees
(employee_id, name, department, salary)
VALUES
(4, 'Daniel', 'Engineering', 88000);
```

## UPDATE

```sql
UPDATE employees
SET salary = 95000
WHERE employee_id = 4;
```

## DELETE

```sql
DELETE FROM employees
WHERE employee_id = 4;
```

### Critical Rule

Never casually run:

```sql
UPDATE employees
SET salary = 95000;
```

or:

```sql
DELETE FROM employees;
```

without understanding the absence of a `WHERE` clause.

---

# 45. DELETE vs DROP vs TRUNCATE

| Command | Basic effect |
|---|---|
| `DELETE` | Removes rows |
| `TRUNCATE` | Removes all rows using DBMS-specific table-truncation semantics |
| `DROP` | Removes the table/object itself |

These commands have important transaction, logging, identity, trigger, and rollback differences across DBMSs.

For interviews, know the conceptual distinction and then learn the exact behavior of the target DBMS.

---

# 46. Schema Example

Use the following conceptual schema for practice:

### departments

| department_id | department_name |
|---:|---|
| 1 | Engineering |
| 2 | Sales |
| 3 | HR |

### employees

| employee_id | name | department_id | salary | manager_id |
|---:|---|---:|---:|---:|
| 101 | Alice | 1 | 90000 | NULL |
| 102 | Bob | 1 | 75000 | 101 |
| 103 | Carol | 2 | 70000 | 105 |
| 104 | David | 2 | 65000 | 105 |
| 105 | Eva | 2 | 100000 | NULL |
| 106 | Frank | 3 | 60000 | NULL |

This schema will be reused for examples.

---

# 47. Interview Query Examples

## 47.1 Find all employees

```sql
SELECT *
FROM employees;
```

## 47.2 Find names and salaries

```sql
SELECT name, salary
FROM employees;
```

## 47.3 Employees earning above 80000

```sql
SELECT name, salary
FROM employees
WHERE salary > 80000;
```

## 47.4 Engineering employees

```sql
SELECT name
FROM employees
WHERE department_id = 1;
```

## 47.5 Highest salaries first

```sql
SELECT name, salary
FROM employees
ORDER BY salary DESC;
```

## 47.6 Highest-paid employee

```sql
SELECT name, salary
FROM employees
ORDER BY salary DESC
LIMIT 1;
```

## 47.7 Distinct department IDs

```sql
SELECT DISTINCT department_id
FROM employees;
```

## 47.8 Employees without managers

```sql
SELECT name
FROM employees
WHERE manager_id IS NULL;
```

## 47.9 Employees with salary between 70000 and 90000

```sql
SELECT name, salary
FROM employees
WHERE salary BETWEEN 70000 AND 90000;
```

## 47.10 Employees from Engineering or Sales

```sql
SELECT name, department_id
FROM employees
WHERE department_id IN (1, 2);
```

---

# 48. SQL NULL Interview Traps

### Trap 1

```sql
WHERE x = NULL
```

Wrong for testing NULL.

Use:

```sql
WHERE x IS NULL
```

### Trap 2

```sql
COUNT(*)
```

counts rows.

```sql
COUNT(x)
```

counts non-NULL `x` values.

### Trap 3

```sql
NULL = NULL
```

does not evaluate to `TRUE`.

NULL represents unknown/missing information, so ordinary equality is not the correct NULL test.

---

# 49. SQL Logical Thinking

When writing a query, translate the English requirement into operations.

Example:

> Find the three highest-paid Engineering employees.

Break it down:

```text
source
→ employees

filter
→ department = Engineering

sort
→ salary DESC

limit
→ 3
```

Query:

```sql
SELECT name, salary
FROM employees
WHERE department_id = 1
ORDER BY salary DESC
LIMIT 3;
```

This decomposition method becomes increasingly important as queries become complex.

---

# 50. GATE / CS Theory

## 50.1 Relation

In relational-model terminology, a relation can be viewed as a set of tuples over attributes.

Practical SQL tables have additional implementation behavior and may allow duplicate rows unless constrained otherwise.

## 50.2 Tuple

A tuple corresponds conceptually to a row.

## 50.3 Attribute

An attribute corresponds conceptually to a column.

## 50.4 Domain

A domain describes the set/type of permissible values associated with an attribute.

## 50.5 Candidate Key

A candidate key is a minimal set of attributes that uniquely identifies tuples.

A relation can have multiple candidate keys.

## 50.6 Primary Key

One candidate key is selected as the primary key.

## 50.7 Foreign Key

A foreign key references a candidate/primary key in another relation according to relational constraint rules.

## 50.8 Entity Integrity

Primary-key values cannot be NULL.

## 50.9 Referential Integrity

Foreign-key relationships should not contain invalid references according to the constraint's configured semantics.

---

# 51. DDL, DML, DQL and Related Categories

Common interview classification:

### DDL — Data Definition Language

```text
CREATE
ALTER
DROP
TRUNCATE
```

### DML — Data Manipulation Language

```text
INSERT
UPDATE
DELETE
```

### DQL — Data Query Language

Often used informally for:

```text
SELECT
```

### DCL

```text
GRANT
REVOKE
```

### TCL

Common examples:

```text
COMMIT
ROLLBACK
SAVEPOINT
```

Exact categorization can vary by textbook/system; focus on command purpose.

---

# 52. Common Mistakes

## Mistake 1: Forgetting WHERE

```sql
UPDATE employees
SET salary = salary * 1.1;
```

This updates every row.

## Mistake 2: Using `= NULL`

Use:

```sql
IS NULL
```

## Mistake 3: Assuming row order

Without:

```sql
ORDER BY
```

do not rely on result order.

## Mistake 4: Using SELECT *

When only a few columns are needed, select those columns explicitly.

## Mistake 5: Confusing WHERE and later GROUP BY/HAVING behavior

`WHERE` filters rows before grouping; `HAVING` is used to filter groups.

## Mistake 6: Ignoring ties

For deterministic ordering:

```sql
ORDER BY salary DESC, employee_id ASC
```

## Mistake 7: Assuming all SQL syntax is universal

Examples:

```text
LIMIT
TOP
FETCH FIRST
```

may differ by DBMS.

---

# 53. Mini Practice Set

Use the `employees` table.

## Q1

Find all employees earning more than `75000`.

### Answer

```sql
SELECT name, salary
FROM employees
WHERE salary > 75000;
```

---

## Q2

Return employees ordered from highest salary to lowest salary.

### Answer

```sql
SELECT name, salary
FROM employees
ORDER BY salary DESC;
```

---

## Q3

Return the three highest-paid employees.

### Answer

```sql
SELECT name, salary
FROM employees
ORDER BY salary DESC
LIMIT 3;
```

---

## Q4

Return all distinct department IDs.

### Answer

```sql
SELECT DISTINCT department_id
FROM employees;
```

---

## Q5

Return employees who do not have a manager.

### Answer

```sql
SELECT name
FROM employees
WHERE manager_id IS NULL;
```

---

## Q6

Return employees whose salary is between `65000` and `90000`.

### Answer

```sql
SELECT name, salary
FROM employees
WHERE salary BETWEEN 65000 AND 90000;
```

---

## Q7

Return employees belonging to departments 1 or 2.

### Answer

```sql
SELECT name, department_id
FROM employees
WHERE department_id IN (1, 2);
```

---

# 54. GATE-Style Solved Questions

> These are **GATE-style practice questions**, not claims about exact official GATE wording or years.

## Question 1 — NULL

Consider:

```text
A
---
10
20
NULL
```

What does:

```sql
SELECT COUNT(A)
FROM T;
```

return?

### Solution

`COUNT(A)` counts non-NULL values.

There are:

```text
10
20
```

Therefore:

```text
Answer = 2
```

---

## Question 2 — COUNT

Table:

```text
employee_id | manager_id
------------|-----------
1           | NULL
2           | 1
3           | 1
4           | NULL
```

Evaluate:

```sql
SELECT COUNT(*), COUNT(manager_id)
FROM employees;
```

### Solution

`COUNT(*)` counts all four rows:

```text
4
```

`COUNT(manager_id)` counts non-NULL manager IDs:

```text
1, 1
```

Therefore:

```text
COUNT(*)       = 4
COUNT(manager_id) = 2
```

---

## Question 3 — DISTINCT

Table:

```text
department_id
-------------
1
1
2
2
2
3
```

Evaluate:

```sql
SELECT COUNT(DISTINCT department_id)
FROM employees;
```

Distinct values:

```text
1, 2, 3
```

Answer:

```text
3
```

---

## Question 4 — ORDER BY + LIMIT

Table:

```text
salary
------
50000
80000
70000
90000
```

Query:

```sql
SELECT salary
FROM employees
ORDER BY salary DESC
LIMIT 2;
```

Sorted result:

```text
90000
80000
70000
50000
```

First two:

```text
90000
80000
```

Answer:

```text
[90000, 80000]
```

---

## Question 5 — NULL Comparison

Consider:

```sql
SELECT *
FROM employees
WHERE manager_id = NULL;
```

Will this correctly return rows where `manager_id` is NULL?

### Solution

No.

The correct predicate is:

```sql
WHERE manager_id IS NULL
```

Reason:

```text
NULL represents unknown/missing information.
```

Ordinary equality is not the correct NULL test.

---

## Question 6 — Logical Precedence

Consider:

```sql
WHERE salary > 80000
   OR department_id = 1
   AND employee_id = 10
```

Assuming standard logical precedence, this is interpreted as:

```sql
WHERE salary > 80000
   OR (department_id = 1 AND employee_id = 10)
```

not:

```sql
WHERE (salary > 80000 OR department_id = 1)
  AND employee_id = 10
```

When the intended logic is the second form, parentheses are required.

---

# 55. Interview Questions You Must Be Able to Answer

1. What is a relational database?
2. What is a table?
3. What is a row/tuple?
4. What is a column/attribute?
5. What is a schema?
6. What is a primary key?
7. What is a candidate key?
8. What is a composite primary key?
9. What is a foreign key?
10. What is referential integrity?
11. What is NULL?
12. Why is `= NULL` incorrect?
13. Difference between `COUNT(*)` and `COUNT(column)`?
14. What does `DISTINCT` do?
15. What does `ORDER BY` do?
16. Why should ordering be explicit?
17. What does `LIMIT` do?
18. What does `OFFSET` do?
19. What is pagination?
20. Why can large OFFSET pagination be inefficient?
21. What is `NOT NULL`?
22. What is `UNIQUE`?
23. What is `CHECK`?
24. What is `DEFAULT`?
25. Difference between `DELETE`, `TRUNCATE`, and `DROP`?
26. What are DDL commands?
27. What are DML commands?
28. What is DQL?
29. What is three-valued logic?
30. What is the logical processing order of a SQL query?

---

# 56. Mastery Checklist

## Relational Concepts

- [ ] Understand relational databases
- [ ] Understand tables
- [ ] Understand rows/tuples
- [ ] Understand columns/attributes
- [ ] Understand schemas
- [ ] Understand domains
- [ ] Understand candidate keys
- [ ] Understand primary keys
- [ ] Understand composite keys
- [ ] Understand foreign keys
- [ ] Understand referential integrity

## NULL

- [ ] Understand what NULL represents
- [ ] Know `IS NULL`
- [ ] Know `IS NOT NULL`
- [ ] Understand three-valued logic
- [ ] Know `COUNT(*)` vs `COUNT(column)`
- [ ] Understand NULL propagation in expressions

## Constraints

- [ ] PRIMARY KEY
- [ ] FOREIGN KEY
- [ ] NOT NULL
- [ ] UNIQUE
- [ ] CHECK
- [ ] DEFAULT

## Basic Commands

- [ ] SELECT
- [ ] FROM
- [ ] WHERE
- [ ] DISTINCT
- [ ] ORDER BY
- [ ] LIMIT
- [ ] OFFSET

## Query Construction

- [ ] Select required columns
- [ ] Filter correctly
- [ ] Sort deterministically
- [ ] Apply top-N logic
- [ ] Understand pagination
- [ ] Use aliases
- [ ] Understand logical query processing

## Interview Readiness

- [ ] Can write basic queries without reference material
- [ ] Can explain NULL correctly
- [ ] Can explain primary vs foreign keys
- [ ] Can explain constraints
- [ ] Can solve basic filtering/sorting questions
- [ ] Can identify DBMS-specific syntax differences
- [ ] Can explain every clause used in a query

---

# 57. Final Revision Sheet

```text
RELATIONAL DATABASE
    ↓
TABLE
    ↓
ROWS + COLUMNS

PRIMARY KEY
    → uniquely identifies row
    → not NULL

FOREIGN KEY
    → references another table's key
    → maintains referential relationships

NULL
    → missing/unknown/inapplicable
    → use IS NULL
    → use IS NOT NULL
    → not equal to 0 or ''

CONSTRAINTS
    → PRIMARY KEY
    → FOREIGN KEY
    → NOT NULL
    → UNIQUE
    → CHECK
    → DEFAULT

BASIC RETRIEVAL
    SELECT
    FROM
    WHERE
    DISTINCT
    ORDER BY
    LIMIT
    OFFSET

COMMON QUERY FLOW
    FROM
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
    LIMIT / OFFSET

KEY NULL TRAP
    x = NULL        ❌
    x IS NULL       ✓

COUNT
    COUNT(*)        → rows
    COUNT(x)        → non-NULL x values

ORDER
    No ORDER BY     → do not assume ordering
    ORDER BY x      → explicit ordering

TOP N
    ORDER BY value DESC
    LIMIT N

PAGINATION
    LIMIT page_size
    OFFSET (page - 1) * page_size
```

---

# 58. What Comes Next

This topic establishes the SQL foundation.

The next SQL topics should build toward:

```text
01. SQL Fundamentals
        ↓
02. Filtering & Operators
        ↓
03. Aggregation & GROUP BY
        ↓
04. HAVING
        ↓
05. Joins
        ↓
06. Subqueries
        ↓
07. CTEs
        ↓
08. Set Operations
        ↓
09. CASE Expressions
        ↓
10. String Functions
        ↓
11. Date & Time
        ↓
12. Window Functions
        ↓
13. Advanced Window Patterns
        ↓
14. Conditional Aggregation
        ↓
15. NULL / Three-Valued Logic
        ↓
16. Constraints & Data Modeling
        ↓
17. Transactions
        ↓
18. Indexes
        ↓
19. Query Optimization
        ↓
20. Views
        ↓
21. Stored Procedures / Triggers
        ↓
22. Advanced SQL Patterns
        ↓
23. Analytical SQL
        ↓
24. SQL Interview Patterns
```

The exact numbering of subsequent files should follow the master SQL roadmap you provide; this file is **SQL Topic 01**, corresponding to **item 24 in the overall DSA + SQL repository sequence**.
