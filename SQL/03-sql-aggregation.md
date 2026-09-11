# 03. SQL Aggregation

> **Track:** SQL Interview Preparation  
> **Overall repository number:** 26  
> **SQL topic number:** 03  
> **Target:** FAANG + top product companies | SDE + ML/AI Engineer  
> **Difficulty:** Easy → Medium → Hard, with Medium dominating

---

# 1. Overview

Aggregation converts multiple rows into summarized results.

Core aggregate functions:

```text
COUNT
SUM
AVG
MIN
MAX
```

The two clauses that make aggregation useful for interview problems are:

```text
GROUP BY
HAVING
```

The central mental model is:

```text
Rows
  ↓
Filter rows with WHERE
  ↓
Form groups with GROUP BY
  ↓
Compute aggregates
  ↓
Filter groups with HAVING
  ↓
Produce selected output
  ↓
Sort with ORDER BY
  ↓
Restrict output with LIMIT
```

---

# 2. The Core Logical Query Order

A commonly taught conceptual order is:

```text
FROM
→ WHERE
→ GROUP BY
→ HAVING
→ SELECT
→ ORDER BY
→ LIMIT
```

This is **logical query processing order**, not the order in which SQL is written.

A query is normally written as:

```sql
SELECT ...
FROM ...
WHERE ...
GROUP BY ...
HAVING ...
ORDER BY ...
LIMIT ...;
```

The distinction is critical for understanding:

- aliases
- aggregate functions
- `WHERE` vs `HAVING`
- grouping
- window functions
- subqueries
- query execution

---

# 3. Written Order vs Logical Order

## Written SQL order

```sql
SELECT ...
FROM ...
WHERE ...
GROUP BY ...
HAVING ...
ORDER BY ...
LIMIT ...;
```

## Conceptual logical order

```text
1. FROM
2. WHERE
3. GROUP BY
4. HAVING
5. SELECT
6. ORDER BY
7. LIMIT
```

Think:

```text
FROM
→ WHERE
→ GROUP
→ FILTER GROUPS
→ SELECT
→ SORT
→ LIMIT
```

### Important nuance

This is a **logical processing model**, not a promise about the physical operations performed by the database engine.

A query optimizer is free to transform the physical execution plan while preserving the query's required semantics.

---

# 4. Sample Schema

Use:

### employees

| employee_id | name | department | salary | age |
|---:|---|---|---:|---:|
| 101 | Alice | Engineering | 90000 | 28 |
| 102 | Bob | Engineering | 75000 | 24 |
| 103 | Carol | Sales | 70000 | 31 |
| 104 | David | Sales | 65000 | 26 |
| 105 | Eva | Sales | 100000 | 35 |
| 106 | Frank | HR | 60000 | 29 |
| 107 | Grace | Engineering | 120000 | 40 |
| 108 | Henry | HR | 55000 | 23 |

---

# 5. COUNT

`COUNT` counts rows or non-NULL values depending on its argument.

---

## 5.1 COUNT(*)

```sql
SELECT COUNT(*)
FROM employees;
```

Counts rows.

For the sample table:

```text
8
```

---

## 5.2 COUNT(column)

```sql
SELECT COUNT(salary)
FROM employees;
```

Counts non-NULL values of `salary`.

If every salary is non-NULL:

```text
8
```

If one salary is NULL:

```text
COUNT(salary) = 7
```

---

## 5.3 COUNT(DISTINCT column)

```sql
SELECT COUNT(DISTINCT department)
FROM employees;
```

Counts distinct non-NULL department values.

For:

```text
Engineering
Sales
HR
```

the result is:

```text
3
```

---

# 6. COUNT and NULL

This distinction is extremely important.

```sql
COUNT(*)
```

counts rows.

```sql
COUNT(column)
```

counts non-NULL column values.

Example:

| id | bonus |
|---:|---:|
| 1 | 1000 |
| 2 | NULL |
| 3 | 2000 |

Then:

```sql
COUNT(*)
```

returns:

```text
3
```

while:

```sql
COUNT(bonus)
```

returns:

```text
2
```

---

# 7. SUM

`SUM` calculates the total of a numeric expression.

```sql
SELECT SUM(salary)
FROM employees;
```

For the sample data:

```text
90000 + 75000 + 70000 + 65000
+ 100000 + 60000 + 120000 + 55000
= 635000
```

Therefore:

```text
635000
```

---

# 8. SUM and NULL

`SUM(column)` generally ignores NULL values.

Example:

```text
100
200
NULL
```

Then:

```sql
SUM(x)
```

is:

```text
300
```

However, if the aggregate receives no non-NULL input values, the result is generally `NULL`, not zero.

Therefore:

```sql
COALESCE(SUM(amount), 0)
```

is a common pattern when the business requirement is to return zero.

---

# 9. AVG

`AVG` calculates the average of non-NULL numeric values.

Conceptually:

```text
AVG(x) = SUM(x) / COUNT(x)
```

where `COUNT(x)` counts non-NULL x values.

Example:

```text
100
200
300
```

Average:

```text
200
```

---

# 10. AVG and NULL

Consider:

```text
100
200
NULL
```

Then:

```sql
AVG(x)
```

uses:

```text
100, 200
```

not:

```text
100, 200, 0
```

Therefore:

```text
AVG(x) = 150
```

A NULL value is not automatically treated as zero.

---

# 11. MIN

`MIN` returns the minimum value.

```sql
SELECT MIN(salary)
FROM employees;
```

For the sample:

```text
55000
```

For dates:

```sql
SELECT MIN(order_date)
FROM orders;
```

can identify the earliest non-NULL date.

---

# 12. MAX

`MAX` returns the maximum value.

```sql
SELECT MAX(salary)
FROM employees;
```

For the sample:

```text
120000
```

For dates:

```sql
SELECT MAX(order_date)
FROM orders;
```

can identify the latest non-NULL date.

---

# 13. Aggregate Functions and NULL

For the common aggregates:

| Function | General NULL behavior |
|---|---|
| `COUNT(*)` | Counts rows |
| `COUNT(column)` | Ignores NULL column values |
| `SUM(column)` | Ignores NULL values |
| `AVG(column)` | Ignores NULL values |
| `MIN(column)` | Ignores NULL values |
| `MAX(column)` | Ignores NULL values |

Important:

```text
No qualifying/non-NULL values
```

can produce:

```text
NULL
```

for `SUM`, `AVG`, `MIN`, and `MAX`.

`COUNT` returns zero when there are no rows/non-NULL values to count, according to the form used.

---

# 14. GROUP BY

`GROUP BY` divides rows into groups based on one or more expressions.

Example:

```sql
SELECT department, COUNT(*)
FROM employees
GROUP BY department;
```

Result:

| department | count |
|---|---:|
| Engineering | 3 |
| Sales | 3 |
| HR | 2 |

The query changes the result from:

```text
8 employee rows
```

to:

```text
3 department groups
```

---

# 15. GROUP BY One Column

```sql
SELECT department, COUNT(*) AS employee_count
FROM employees
GROUP BY department;
```

Each distinct department becomes a group.

Conceptually:

```text
Engineering → Alice, Bob, Grace
Sales       → Carol, David, Eva
HR          → Frank, Henry
```

---

# 16. GROUP BY Multiple Columns

You can group using multiple columns.

```sql
SELECT department, age, COUNT(*) AS employee_count
FROM employees
GROUP BY department, age;
```

The group key is the combination:

```text
(department, age)
```

not each column independently.

Example:

```text
Engineering, 28
Engineering, 24
Engineering, 40
...
```

---

# 17. GROUP BY as a Partition

A useful mental model:

```text
GROUP BY columns
```

creates equivalence classes of rows based on the grouping key.

For:

```sql
GROUP BY department
```

all rows with the same department belong to the same group.

For:

```sql
GROUP BY department, city
```

the pair:

```text
(department, city)
```

defines the group.

---

# 18. Aggregate Without GROUP BY

You can aggregate the entire filtered result into one group.

```sql
SELECT COUNT(*)
FROM employees;
```

Conceptually:

```text
all qualifying rows
→ one group
→ COUNT
```

Similarly:

```sql
SELECT AVG(salary), MIN(salary), MAX(salary)
FROM employees;
```

returns one result row.

---

# 19. GROUP BY + SUM

```sql
SELECT department, SUM(salary) AS total_salary
FROM employees
GROUP BY department;
```

Sample results:

```text
Engineering → 285000
Sales       → 235000
HR          → 115000
```

---

# 20. GROUP BY + AVG

```sql
SELECT department, AVG(salary) AS avg_salary
FROM employees
GROUP BY department;
```

Sample:

```text
Engineering → 95000
Sales       → 78333.33...
HR          → 57500
```

The exact displayed precision depends on the DBMS/data type.

---

# 21. GROUP BY + MIN/MAX

```sql
SELECT
    department,
    MIN(salary) AS minimum_salary,
    MAX(salary) AS maximum_salary
FROM employees
GROUP BY department;
```

This produces one row per department.

---

# 22. Multiple Aggregates

A single query can calculate multiple metrics.

```sql
SELECT
    department,
    COUNT(*) AS employee_count,
    SUM(salary) AS total_salary,
    AVG(salary) AS average_salary,
    MIN(salary) AS minimum_salary,
    MAX(salary) AS maximum_salary
FROM employees
GROUP BY department;
```

This is a common interview pattern.

---

# 23. HAVING

`HAVING` filters groups after grouping/aggregation.

Example:

```sql
SELECT department, COUNT(*) AS employee_count
FROM employees
GROUP BY department
HAVING COUNT(*) >= 3;
```

Sample result:

```text
Engineering → 3
Sales       → 3
```

HR has only two employees and is removed.

---

# 24. WHERE vs HAVING

This distinction must be automatic.

### WHERE

Filters individual rows:

```sql
WHERE salary > 70000
```

### HAVING

Filters groups:

```sql
HAVING AVG(salary) > 70000
```

Think:

```text
WHERE
→ row-level filtering

GROUP BY
→ create groups

HAVING
→ group-level filtering
```

---

# 25. Example: WHERE Before GROUP BY

Question:

> For each department, count employees earning more than 70000.

```sql
SELECT department, COUNT(*) AS employee_count
FROM employees
WHERE salary > 70000
GROUP BY department;
```

Processing:

```text
all employees
    ↓
remove salary <= 70000
    ↓
group remaining rows by department
    ↓
COUNT each group
```

---

# 26. Example: HAVING After GROUP BY

Question:

> Find departments whose average salary is greater than 70000.

```sql
SELECT department, AVG(salary) AS avg_salary
FROM employees
GROUP BY department
HAVING AVG(salary) > 70000;
```

Processing:

```text
employees
    ↓
group by department
    ↓
calculate AVG per group
    ↓
keep groups with AVG > 70000
```

You cannot replace this directly with:

```sql
WHERE AVG(salary) > 70000
```

because the aggregate is a group-level calculation.

---

# 27. WHERE + GROUP BY + HAVING

These clauses are often combined.

Question:

> Among employees earning above 60000, find departments having at least 2 such employees.

```sql
SELECT department, COUNT(*) AS employee_count
FROM employees
WHERE salary > 60000
GROUP BY department
HAVING COUNT(*) >= 2;
```

Logical flow:

```text
FROM
  ↓
WHERE salary > 60000
  ↓
GROUP BY department
  ↓
HAVING COUNT(*) >= 2
  ↓
SELECT department, COUNT(*)
```

---

# 28. HAVING Without GROUP BY

`HAVING` can be used with an aggregate query without an explicit `GROUP BY`.

Example:

```sql
SELECT COUNT(*) AS employee_count
FROM employees
HAVING COUNT(*) >= 5;
```

Conceptually, the entire result is treated as one group.

If there are at least five rows, the query returns the aggregate row.

---

# 29. GROUP BY and SELECT

A classic SQL rule:

When grouping, a selected expression generally needs to be:

1. a grouping expression, or
2. an aggregate expression,

subject to the SQL standard and DBMS-specific functional-dependency extensions.

Correct:

```sql
SELECT department, COUNT(*)
FROM employees
GROUP BY department;
```

Incorrect in standard grouping semantics:

```sql
SELECT department, name, COUNT(*)
FROM employees
GROUP BY department;
```

Why?

There can be many names inside one department group.

Which `name` should be returned?

SQL cannot arbitrarily choose one under standard grouping rules.

---

# 30. The "Non-Aggregated Column" Trap

Suppose:

```text
department | name
-----------|------
Engineering | Alice
Engineering | Bob
Engineering | Grace
```

Query:

```sql
SELECT department, name, COUNT(*)
FROM employees
GROUP BY department;
```

There is no single well-defined `name` for the Engineering group.

Therefore the query is invalid under standard grouping semantics.

Correct approaches depend on the requirement:

```sql
SELECT department, COUNT(*)
FROM employees
GROUP BY department;
```

or use an aggregate:

```sql
MIN(name)
```

or use a ranking/window-function solution if you actually need a particular employee.

---

# 31. GROUP BY Expression

You can group by expressions.

Example:

```sql
SELECT
    salary / 10000 AS salary_band,
    COUNT(*)
FROM employees
GROUP BY salary / 10000;
```

Exact numeric behavior can depend on data types and DBMS rules.

Another example:

```sql
SELECT
    EXTRACT(YEAR FROM hire_date) AS hire_year,
    COUNT(*)
FROM employees
GROUP BY EXTRACT(YEAR FROM hire_date);
```

Date syntax varies by DBMS.

---

# 32. GROUP BY Alias

Whether a SELECT alias can be referenced directly in `GROUP BY` varies across SQL systems.

For maximum conceptual portability:

```sql
SELECT salary / 10000 AS salary_band,
       COUNT(*)
FROM employees
GROUP BY salary / 10000;
```

Some DBMSs allow:

```sql
GROUP BY salary_band
```

but do not assume that syntax is universally portable.

---

# 33. HAVING and Aliases

Similarly, alias visibility in `HAVING` varies by DBMS.

Portable/common interview form:

```sql
SELECT department,
       AVG(salary) AS avg_salary
FROM employees
GROUP BY department
HAVING AVG(salary) > 70000;
```

Do not rely on:

```sql
HAVING avg_salary > 70000
```

unless the target DBMS supports it.

---

# 34. ORDER BY Aggregates

You can sort by an aggregate.

```sql
SELECT
    department,
    AVG(salary) AS avg_salary
FROM employees
GROUP BY department
ORDER BY avg_salary DESC;
```

This is a very common pattern.

---

# 35. GROUP BY + ORDER BY + LIMIT

Question:

> Return the two departments with the highest average salary.

```sql
SELECT
    department,
    AVG(salary) AS avg_salary
FROM employees
GROUP BY department
ORDER BY avg_salary DESC
LIMIT 2;
```

Conceptually:

```text
group
→ calculate average
→ sort groups
→ take top 2
```

---

# 36. HAVING + ORDER BY

Question:

> Show departments with at least 2 employees, ordered by employee count descending.

```sql
SELECT
    department,
    COUNT(*) AS employee_count
FROM employees
GROUP BY department
HAVING COUNT(*) >= 2
ORDER BY employee_count DESC;
```

---

# 37. Filtering Before vs After Aggregation

These two queries answer different questions.

### Query A

```sql
SELECT department, AVG(salary)
FROM employees
WHERE salary > 70000
GROUP BY department;
```

Meaning:

```text
Average salary of employees whose salary > 70000
```

### Query B

```sql
SELECT department, AVG(salary)
FROM employees
GROUP BY department
HAVING AVG(salary) > 70000;
```

Meaning:

```text
Average salary of all employees in departments whose average > 70000
```

They are **not equivalent**.

This distinction is heavily tested.

---

# 38. Example Showing the Difference

Sales:

```text
70000
65000
100000
```

Query A:

```sql
WHERE salary > 70000
```

keeps:

```text
100000
```

Average:

```text
100000
```

Query B:

```sql
HAVING AVG(salary) > 70000
```

first calculates:

```text
(70000 + 65000 + 100000) / 3
= 78333.33...
```

The entire Sales group remains because:

```text
78333.33 > 70000
```

The selected rows and aggregate therefore differ.

---

# 39. COUNT(*) vs COUNT(column) in GROUP BY

Suppose:

| department | manager_id |
|---|---:|
| A | NULL |
| A | 1 |
| A | 2 |
| B | NULL |
| B | NULL |

Query:

```sql
SELECT
    department,
    COUNT(*) AS rows_count,
    COUNT(manager_id) AS manager_count
FROM employees
GROUP BY department;
```

Results:

```text
A → rows_count = 3, manager_count = 2
B → rows_count = 2, manager_count = 0
```

This pattern is common in interviews.

---

# 40. Counting Conditional Rows

A common pattern is conditional aggregation.

Example:

```sql
SELECT
    department,
    SUM(CASE WHEN salary >= 80000 THEN 1 ELSE 0 END) AS high_paid
FROM employees
GROUP BY department;
```

This is a bridge to advanced analytical SQL.

Another widely supported pattern in some DBMSs is:

```sql
COUNT(CASE WHEN salary >= 80000 THEN 1 END)
```

The exact behavior follows the fact that `COUNT(expression)` ignores NULL.

---

# 41. Counting Distinct Values Per Group

Question:

> How many distinct cities are represented in each department?

```sql
SELECT
    department,
    COUNT(DISTINCT city) AS city_count
FROM employees
GROUP BY department;
```

This pattern is frequently tested.

---

# 42. SUM vs COUNT

These answer different questions.

```sql
COUNT(*)
```

asks:

> How many rows?

```sql
SUM(salary)
```

asks:

> What is the total salary?

Example:

```sql
SELECT
    department,
    COUNT(*) AS employees,
    SUM(salary) AS payroll
FROM employees
GROUP BY department;
```

---

# 43. AVG vs SUM / COUNT

Conceptually:

```text
AVG(x)
=
SUM(x) / COUNT(x)
```

for the non-NULL values.

Therefore, if NULLs exist:

```sql
AVG(x)
```

does not generally equal:

```sql
SUM(x) / COUNT(*)
```

because `COUNT(*)` includes rows where x is NULL.

The corresponding conceptual relationship is:

```sql
SUM(x) / COUNT(x)
```

with appropriate numeric/type handling.

---

# 44. MIN/MAX Are Not Row Retrieval

A common mistake:

```sql
SELECT department, MAX(salary)
FROM employees
GROUP BY department;
```

returns the maximum salary value per department.

It does **not** automatically return the employee who earned that salary.

If you need:

```text
employee name + maximum salary
```

you generally need:

- a subquery
- a join
- a window function
- or another ranking technique

depending on the problem.

---

# 45. Finding Departments with More Than N Employees

General pattern:

```sql
SELECT department, COUNT(*) AS employee_count
FROM employees
GROUP BY department
HAVING COUNT(*) > N;
```

For `N = 2`:

```sql
HAVING COUNT(*) > 2
```

This is one of the most common aggregation templates.

---

# 46. Finding Departments with Minimum Average

```sql
SELECT department, AVG(salary) AS avg_salary
FROM employees
GROUP BY department
HAVING AVG(salary) >= 80000;
```

The aggregate condition belongs in `HAVING`.

---

# 47. Finding Total per Group

```sql
SELECT department, SUM(salary) AS total_salary
FROM employees
GROUP BY department;
```

General pattern:

```text
GROUP BY dimension
+
SUM(measure)
```

This pattern appears throughout business and analytical SQL.

---

# 48. Finding Min/Max per Group

```sql
SELECT
    department,
    MIN(salary) AS min_salary,
    MAX(salary) AS max_salary
FROM employees
GROUP BY department;
```

---

# 49. Aggregation with WHERE

Question:

> Calculate the average salary of Engineering employees.

```sql
SELECT AVG(salary) AS avg_salary
FROM employees
WHERE department = 'Engineering';
```

No `GROUP BY` is necessary because only one filtered population is being summarized.

---

# 50. Aggregation with GROUP BY

Question:

> Calculate the average salary for every department.

```sql
SELECT
    department,
    AVG(salary) AS avg_salary
FROM employees
GROUP BY department;
```

Difference:

```text
one overall population
vs
one population per department
```

---

# 51. Aggregation with HAVING

Question:

> Return departments whose average salary exceeds 80000.

```sql
SELECT
    department,
    AVG(salary) AS avg_salary
FROM employees
GROUP BY department
HAVING AVG(salary) > 80000;
```

---

# 52. Aggregation with ORDER BY

Question:

> List departments from highest total payroll to lowest.

```sql
SELECT
    department,
    SUM(salary) AS total_salary
FROM employees
GROUP BY department
ORDER BY total_salary DESC;
```

---

# 53. Full Aggregation Pattern

A very important template:

```sql
SELECT
    group_column,
    AGGREGATE_FUNCTION(measure) AS metric
FROM table_name
WHERE row_condition
GROUP BY group_column
HAVING group_condition
ORDER BY metric DESC
LIMIT N;
```

Example:

```sql
SELECT
    department,
    AVG(salary) AS avg_salary
FROM employees
WHERE age >= 25
GROUP BY department
HAVING AVG(salary) >= 70000
ORDER BY avg_salary DESC
LIMIT 3;
```

---

# 54. Exact Logical Processing Walkthrough

Query:

```sql
SELECT
    department,
    AVG(salary) AS avg_salary
FROM employees
WHERE age >= 25
GROUP BY department
HAVING AVG(salary) >= 70000
ORDER BY avg_salary DESC
LIMIT 2;
```

## Step 1 — FROM

Source:

```text
employees
```

## Step 2 — WHERE

Keep:

```text
age >= 25
```

## Step 3 — GROUP BY

Create:

```text
Engineering
Sales
HR
```

from the remaining rows.

## Step 4 — HAVING

Calculate:

```text
AVG(salary)
```

for each group.

Remove groups whose average is below:

```text
70000
```

## Step 5 — SELECT

Produce:

```text
department
avg_salary
```

## Step 6 — ORDER BY

Sort:

```text
avg_salary DESC
```

## Step 7 — LIMIT

Keep:

```text
2
```

rows.

---

# 55. Why SELECT Comes Later Conceptually

Consider:

```sql
SELECT salary * 12 AS annual_salary
FROM employees
WHERE annual_salary > 1000000;
```

In standard SQL semantics, the SELECT alias generally is not available to `WHERE` because `WHERE` logically happens before `SELECT`.

Instead write:

```sql
WHERE salary * 12 > 1000000
```

or use a subquery/CTE:

```sql
SELECT *
FROM (
    SELECT name, salary * 12 AS annual_salary
    FROM employees
) t
WHERE annual_salary > 1000000;
```

Some DBMSs provide special alias behavior, but do not assume SELECT aliases are available everywhere in WHERE.

---

# 56. Why HAVING Can Use Aggregates

Example:

```sql
HAVING COUNT(*) >= 3
```

The group exists before the `HAVING` condition is evaluated in the logical model.

Therefore the aggregate:

```sql
COUNT(*)
```

is meaningful.

By contrast:

```sql
WHERE COUNT(*) >= 3
```

attempts to use a group-level aggregate during row filtering and is not the correct construction.

---

# 57. WHERE vs HAVING Decision Rule

Ask:

> Am I filtering individual rows or an aggregate/group result?

If individual rows:

```sql
WHERE
```

If groups/aggregate values:

```sql
HAVING
```

Examples:

```text
salary > 80000
→ WHERE

department = 'Engineering'
→ WHERE

COUNT(*) >= 5
→ HAVING

AVG(salary) > 70000
→ HAVING

SUM(revenue) > 1000000
→ HAVING
```

---

# 58. WHERE + HAVING Together

Use both when the problem contains two different levels of filtering.

Example:

> Among orders from 2026, find customers with at least 5 orders.

Conceptually:

```sql
SELECT customer_id, COUNT(*) AS order_count
FROM orders
WHERE order_date >= DATE '2026-01-01'
  AND order_date < DATE '2027-01-01'
GROUP BY customer_id
HAVING COUNT(*) >= 5;
```

Meaning:

```text
WHERE
→ restrict source rows to 2026

GROUP BY
→ make one group per customer

HAVING
→ keep customers with at least 5 orders
```

---

# 59. HAVING Without Aggregate in Predicate

Some SQL systems allow HAVING conditions that refer only to grouping columns, but this is not the main use case.

For interview problems:

```text
row-level condition
→ WHERE

group-level aggregate condition
→ HAVING
```

is the safest mental model.

---

# 60. GROUP BY and NULL

NULL values can form a group.

Example:

```sql
SELECT department, COUNT(*)
FROM employees
GROUP BY department;
```

If some employees have:

```text
department = NULL
```

those NULL department rows belong to a NULL grouping category.

This differs from ordinary equality testing with NULL.

---

# 61. GROUP BY NULL Concept

Although:

```text
NULL = NULL
```

is UNKNOWN under ordinary SQL comparison semantics, grouping treats NULL values as belonging to the same grouping category for the purpose of forming groups.

This distinction is important:

```text
comparison semantics
≠
grouping semantics
```

---

# 62. DISTINCT vs GROUP BY

These can sometimes produce similar results:

```sql
SELECT DISTINCT department
FROM employees;
```

and:

```sql
SELECT department
FROM employees
GROUP BY department;
```

Both can produce one row per distinct department.

But `GROUP BY` is designed for aggregation:

```sql
SELECT department, COUNT(*)
FROM employees
GROUP BY department;
```

Use the construct that best communicates the intent.

---

# 63. GROUP BY vs Window Functions

`GROUP BY` generally collapses multiple input rows into fewer output rows.

Example:

```sql
SELECT department, AVG(salary)
FROM employees
GROUP BY department;
```

One row per department.

A window function can calculate a department average while retaining employee rows:

```sql
SELECT
    name,
    department,
    salary,
    AVG(salary) OVER (PARTITION BY department) AS dept_avg
FROM employees;
```

Window functions are a later topic, but understanding this distinction prevents many mistakes.

---

# 64. Common Aggregation Interview Patterns

## Pattern 1 — Count per group

```sql
SELECT group_col, COUNT(*)
FROM table
GROUP BY group_col;
```

## Pattern 2 — Sum per group

```sql
SELECT group_col, SUM(value)
FROM table
GROUP BY group_col;
```

## Pattern 3 — Average per group

```sql
SELECT group_col, AVG(value)
FROM table
GROUP BY group_col;
```

## Pattern 4 — Filter groups

```sql
GROUP BY group_col
HAVING COUNT(*) > N;
```

## Pattern 5 — Top groups

```sql
GROUP BY group_col
ORDER BY COUNT(*) DESC
LIMIT N;
```

## Pattern 6 — Distinct count

```sql
COUNT(DISTINCT value)
```

## Pattern 7 — Conditional count

```sql
SUM(CASE WHEN condition THEN 1 ELSE 0 END)
```

---

# 65. Common Mistakes

## Mistake 1

Using:

```sql
WHERE COUNT(*) > 5
```

Use:

```sql
HAVING COUNT(*) > 5
```

---

## Mistake 2

Using:

```sql
COUNT(column)
```

when you need all rows.

Use:

```sql
COUNT(*)
```

---

## Mistake 3

Assuming NULL contributes to `AVG`.

It does not count as zero.

---

## Mistake 4

Assuming:

```sql
SUM(column)
```

returns zero when there are no qualifying rows.

It can return NULL.

Use:

```sql
COALESCE(SUM(column), 0)
```

when zero is the desired business result.

---

## Mistake 5

Selecting a non-grouped, non-aggregated column.

Example:

```sql
SELECT department, name, COUNT(*)
FROM employees
GROUP BY department;
```

This is not valid under standard grouping semantics.

---

## Mistake 6

Confusing:

```sql
WHERE salary > 70000
```

with:

```sql
HAVING AVG(salary) > 70000
```

They answer different questions.

---

## Mistake 7

Thinking the written query order is the logical processing order.

Written:

```text
SELECT
FROM
WHERE
GROUP BY
HAVING
ORDER BY
LIMIT
```

Conceptual:

```text
FROM
WHERE
GROUP BY
HAVING
SELECT
ORDER BY
LIMIT
```

---

## Mistake 8

Thinking the logical processing order is the physical execution plan.

The optimizer may reorder/transform physical operations while preserving semantics.

---

# 66. GATE / CS Theory

## 66.1 Aggregate Functions

Know the conceptual purpose:

```text
COUNT → cardinality/count
SUM   → total
AVG   → average
MIN   → minimum
MAX   → maximum
```

---

## 66.2 Grouping

`GROUP BY` partitions rows according to grouping expressions.

Each group can produce aggregate values.

---

## 66.3 HAVING

`HAVING` filters groups after grouping/aggregation in the logical query model.

---

## 66.4 WHERE

`WHERE` filters rows before grouping.

---

## 66.5 Logical Processing Order

Core interview/GATE model:

```text
FROM
→ WHERE
→ GROUP BY
→ HAVING
→ SELECT
→ ORDER BY
→ LIMIT
```

Note:

```text
LIMIT
```

is common SQL syntax but is not part of the SQL standard's universal clause vocabulary; pagination syntax varies by DBMS.

---

## 66.6 COUNT and NULL

```text
COUNT(*)      → counts rows
COUNT(x)      → counts non-NULL x
```

---

## 66.7 AVG and NULL

Conceptually:

```text
AVG(x)
=
SUM(non-NULL x)
/
COUNT(non-NULL x)
```

---

## 66.8 GROUP BY and NULL

NULL grouping values are treated as one grouping category.

This does not mean:

```text
NULL = NULL
```

becomes TRUE for ordinary comparisons.

---

# 67. GATE-Style Solved Questions

> These are **GATE-style practice questions**, not claims about exact official GATE wording or years.

## Question 1 — COUNT and NULL

Table:

| id | value |
|---:|---:|
| 1 | 10 |
| 2 | NULL |
| 3 | 20 |
| 4 | NULL |

Evaluate:

```sql
SELECT COUNT(*), COUNT(value)
FROM T;
```

### Solution

Total rows:

```text
COUNT(*) = 4
```

Non-NULL values:

```text
10, 20
```

Therefore:

```text
COUNT(value) = 2
```

Answer:

```text
(4, 2)
```

---

# 68. Question 2 — WHERE vs HAVING

Consider:

```sql
SELECT department, AVG(salary)
FROM employees
WHERE salary > 70000
GROUP BY department
HAVING AVG(salary) > 80000;
```

What does `WHERE` do and what does `HAVING` do?

### Solution

`WHERE` removes rows:

```text
salary <= 70000
```

before grouping.

Then:

```text
GROUP BY department
```

creates department groups from the remaining rows.

Then:

```text
HAVING AVG(salary) > 80000
```

removes groups whose average of the remaining rows is not greater than 80000.

Therefore:

```text
WHERE  → row-level filter
HAVING → group-level filter
```

---

# 69. Question 3 — GROUP BY

Table:

```text
department
-----------
A
A
B
B
B
C
```

Evaluate:

```sql
SELECT department, COUNT(*)
FROM T
GROUP BY department;
```

### Solution

Groups:

```text
A → 2
B → 3
C → 1
```

Therefore the result contains one row per distinct department.

---

# 70. Question 4 — AVG with NULL

Values:

```text
10
20
NULL
30
```

What is:

```sql
AVG(value)
```

### Solution

NULL is ignored.

Therefore:

```text
AVG = (10 + 20 + 30) / 3
    = 20
```

Answer:

```text
20
```

It is not:

```text
15
```

because NULL is not automatically treated as zero.

---

# 71. Question 5 — HAVING

Table:

```text
department | salary
-----------|-------
A          | 100
A          | 200
B          | 50
B          | 60
C          | 300
```

Evaluate:

```sql
SELECT department, SUM(salary) AS total
FROM T
GROUP BY department
HAVING SUM(salary) >= 300;
```

### Solution

Group totals:

```text
A → 300
B → 110
C → 300
```

Keep totals >= 300:

```text
A
C
```

Answer:

```text
A → 300
C → 300
```

---

# 72. Question 6 — Logical Order

Consider:

```sql
SELECT department, COUNT(*) AS c
FROM employees
WHERE salary > 70000
GROUP BY department
HAVING COUNT(*) >= 2
ORDER BY c DESC
LIMIT 2;
```

State the conceptual processing order.

### Solution

```text
FROM
→ WHERE
→ GROUP BY
→ HAVING
→ SELECT
→ ORDER BY
→ LIMIT
```

Meaning:

```text
1. Read employees
2. Keep salary > 70000
3. Group remaining rows by department
4. Keep groups with at least 2 rows
5. Produce department and count
6. Sort by count descending
7. Return first 2 groups
```

---

# 73. Interview Practice

## Easy

### Q1
Count all employees.

```sql
SELECT COUNT(*)
FROM employees;
```

### Q2
Find total payroll.

```sql
SELECT SUM(salary)
FROM employees;
```

### Q3
Find average salary.

```sql
SELECT AVG(salary)
FROM employees;
```

### Q4
Find minimum salary.

```sql
SELECT MIN(salary)
FROM employees;
```

### Q5
Find maximum salary.

```sql
SELECT MAX(salary)
FROM employees;
```

---

# 74. Medium Practice

## Q6

Count employees per department.

```sql
SELECT department, COUNT(*) AS employee_count
FROM employees
GROUP BY department;
```

## Q7

Find average salary per department.

```sql
SELECT department, AVG(salary) AS avg_salary
FROM employees
GROUP BY department;
```

## Q8

Find departments with at least three employees.

```sql
SELECT department, COUNT(*) AS employee_count
FROM employees
GROUP BY department
HAVING COUNT(*) >= 3;
```

## Q9

Find departments whose total salary exceeds 200000.

```sql
SELECT department, SUM(salary) AS total_salary
FROM employees
GROUP BY department
HAVING SUM(salary) > 200000;
```

## Q10

Return the department with the highest average salary.

```sql
SELECT department, AVG(salary) AS avg_salary
FROM employees
GROUP BY department
ORDER BY avg_salary DESC
LIMIT 1;
```

---

# 75. Harder Aggregation Practice

## Q11

Among employees aged at least 25, find departments having at least two employees and average salary above 70000.

```sql
SELECT
    department,
    COUNT(*) AS employee_count,
    AVG(salary) AS avg_salary
FROM employees
WHERE age >= 25
GROUP BY department
HAVING COUNT(*) >= 2
   AND AVG(salary) > 70000;
```

---

## Q12

Find the number of distinct cities represented in each department.

```sql
SELECT
    department,
    COUNT(DISTINCT city) AS city_count
FROM employees
GROUP BY department;
```

---

## Q13

Find total salary and maximum salary for each department, ordered by total salary descending.

```sql
SELECT
    department,
    SUM(salary) AS total_salary,
    MAX(salary) AS max_salary
FROM employees
GROUP BY department
ORDER BY total_salary DESC;
```

---

# 76. Aggregation Decision Tree

When you see:

> "How many?"

Use:

```sql
COUNT(*)
```

or:

```sql
COUNT(column)
```

depending on NULL semantics.

When you see:

> "Total"

Use:

```sql
SUM(...)
```

When you see:

> "Average"

Use:

```sql
AVG(...)
```

When you see:

> "Minimum"

Use:

```sql
MIN(...)
```

When you see:

> "Maximum"

Use:

```sql
MAX(...)
```

When you see:

> "For each department/category/customer"

Use:

```sql
GROUP BY
```

When you see:

> "Groups having..."

Use:

```sql
HAVING
```

---

# 77. Aggregation Translation Patterns

English:

> Number of employees in each department

```sql
GROUP BY department
COUNT(*)
```

English:

> Total sales per customer

```sql
GROUP BY customer_id
SUM(amount)
```

English:

> Average order value per customer

```sql
GROUP BY customer_id
AVG(amount)
```

English:

> Departments with more than 5 employees

```sql
GROUP BY department
HAVING COUNT(*) > 5
```

English:

> Customers whose total spending exceeds 10000

```sql
GROUP BY customer_id
HAVING SUM(amount) > 10000
```

---

# 78. Aggregation + Sorting Pattern

Question:

> Find the three departments with the highest total payroll.

```sql
SELECT
    department,
    SUM(salary) AS total_payroll
FROM employees
GROUP BY department
ORDER BY total_payroll DESC
LIMIT 3;
```

Pattern:

```text
GROUP BY
→ aggregate
→ ORDER BY aggregate DESC
→ LIMIT N
```

This pattern appears constantly in SQL interviews.

---

# 79. Aggregation + Filtering Pattern

Question:

> Find departments whose payroll exceeds 200000.

```sql
SELECT
    department,
    SUM(salary) AS total_payroll
FROM employees
GROUP BY department
HAVING SUM(salary) > 200000;
```

Do not use:

```sql
WHERE SUM(salary) > 200000
```

because `SUM(salary)` is a group-level calculation.

---

# 80. Aggregation + Row Filtering Pattern

Question:

> Find the average salary of employees earning at least 70000, by department.

```sql
SELECT
    department,
    AVG(salary) AS avg_salary
FROM employees
WHERE salary >= 70000
GROUP BY department;
```

The filtering changes the population included in the average.

---

# 81. Aggregation + Row and Group Filtering

Question:

> Among employees earning at least 70000, find departments with at least two employees.

```sql
SELECT
    department,
    COUNT(*) AS employee_count
FROM employees
WHERE salary >= 70000
GROUP BY department
HAVING COUNT(*) >= 2;
```

This is the canonical:

```text
WHERE
→ GROUP BY
→ HAVING
```

pattern.

---

# 82. SQL Query Clause Mental Model

Memorize:

```text
FROM
    choose source rows

WHERE
    filter individual rows

GROUP BY
    form groups

HAVING
    filter groups

SELECT
    produce output expressions

ORDER BY
    sort final result

LIMIT
    restrict returned rows
```

And distinguish it from the **written syntax order**:

```text
SELECT
FROM
WHERE
GROUP BY
HAVING
ORDER BY
LIMIT
```

---

# 83. Logical vs Physical Execution

This distinction matters in interviews.

### Logical processing order

Describes the semantics used to reason about the query.

```text
FROM
→ WHERE
→ GROUP BY
→ HAVING
→ SELECT
→ ORDER BY
→ LIMIT
```

### Physical execution

The database optimizer may choose a different execution plan.

It may:

- use indexes
- push filters down
- reorder joins
- aggregate early
- eliminate unnecessary work
- choose hash aggregation
- choose sort-based aggregation
- parallelize operations

The optimizer must preserve the query's observable semantics.

Therefore:

```text
logical order ≠ physical execution order
```

---

# 84. Why This Distinction Matters

Suppose:

```sql
SELECT department, COUNT(*)
FROM employees
WHERE salary > 70000
GROUP BY department;
```

Logically:

```text
WHERE
→ then GROUP BY
```

But the database may execute a physical plan involving:

```text
index scan
→ filter
→ hash aggregate
```

or another equivalent plan.

Do not confuse SQL's conceptual processing order with the database engine's exact physical execution sequence.

---

# 85. Aggregate Function Edge Cases

## Empty input

Conceptually:

```sql
SELECT COUNT(*)
FROM employees
WHERE 1 = 0;
```

returns:

```text
0
```

But:

```sql
SELECT SUM(salary)
FROM employees
WHERE 1 = 0;
```

generally returns:

```text
NULL
```

Likewise, `AVG`, `MIN`, and `MAX` generally return NULL when there is no applicable value.

If the requirement is zero:

```sql
SELECT COALESCE(SUM(salary), 0)
FROM employees
WHERE 1 = 0;
```

---

# 86. DISTINCT Aggregation

You can combine `DISTINCT` with aggregates.

```sql
SELECT COUNT(DISTINCT department)
FROM employees;
```

Or:

```sql
SELECT SUM(DISTINCT salary)
FROM employees;
```

The latter sums only distinct salary values.

Use `DISTINCT` only when duplicates should genuinely be eliminated.

---

# 87. Aggregate on Expressions

Aggregates can operate on expressions.

```sql
SELECT SUM(salary * 12)
FROM employees;
```

Or:

```sql
SELECT AVG(salary * 1.10)
FROM employees;
```

Be aware that numeric type, precision, and overflow behavior depend on the DBMS/data type.

---

# 88. Conditional Aggregation

One of the most useful advanced patterns:

```sql
SELECT
    department,
    SUM(CASE WHEN salary >= 100000 THEN 1 ELSE 0 END) AS high_earners
FROM employees
GROUP BY department;
```

This produces one row per department and counts qualifying employees.

General form:

```sql
SUM(
    CASE
        WHEN condition THEN 1
        ELSE 0
    END
)
```

This becomes extremely important in analytical SQL.

---

# 89. Multiple Conditional Metrics

```sql
SELECT
    department,
    SUM(CASE WHEN salary >= 100000 THEN 1 ELSE 0 END) AS high_earners,
    SUM(CASE WHEN salary < 70000 THEN 1 ELSE 0 END) AS low_earners
FROM employees
GROUP BY department;
```

This lets one grouped query produce multiple conditional statistics.

---

# 90. Aggregation and Joins Preview

Aggregation is often combined with joins.

Example:

```sql
SELECT
    d.department_name,
    COUNT(e.employee_id) AS employee_count
FROM departments d
LEFT JOIN employees e
    ON e.department_id = d.department_id
GROUP BY d.department_name;
```

The choice of:

```text
INNER JOIN
LEFT JOIN
```

can change which groups appear.

Joins are covered separately.

---

# 91. Why COUNT(e.employee_id) Can Matter with LEFT JOIN

Consider:

```sql
LEFT JOIN employees e
```

A department with no employee can still produce a joined row where:

```text
e.employee_id = NULL
```

Then:

```sql
COUNT(e.employee_id)
```

returns:

```text
0
```

for that department.

By contrast:

```sql
COUNT(*)
```

counts the joined row and may return:

```text
1
```

This is an important aggregation + join interview trap.

---

# 92. Common "At Least N" Pattern

Requirement:

> Customers with at least 3 orders.

```sql
SELECT customer_id, COUNT(*) AS order_count
FROM orders
GROUP BY customer_id
HAVING COUNT(*) >= 3;
```

This exact structure appears repeatedly in interview problems.

---

# 93. Common "Total Greater Than X" Pattern

Requirement:

> Customers whose total spending exceeds 5000.

```sql
SELECT
    customer_id,
    SUM(amount) AS total_spending
FROM orders
GROUP BY customer_id
HAVING SUM(amount) > 5000;
```

---

# 94. Common "Average Above X" Pattern

Requirement:

> Products with average rating above 4.

```sql
SELECT
    product_id,
    AVG(rating) AS avg_rating
FROM reviews
GROUP BY product_id
HAVING AVG(rating) > 4;
```

---

# 95. Common "Top Group" Pattern

Requirement:

> Department with the highest number of employees.

```sql
SELECT
    department,
    COUNT(*) AS employee_count
FROM employees
GROUP BY department
ORDER BY employee_count DESC
LIMIT 1;
```

If ties must all be returned, this problem requires a different technique, such as a ranking/window function or comparison against the maximum count.

---

# 96. Common "Group with Minimum/Maximum Aggregate" Trap

Question:

> Find the employee with the highest salary in each department.

This is **not** solved completely by:

```sql
SELECT department, MAX(salary)
FROM employees
GROUP BY department;
```

That only returns:

```text
department
maximum salary
```

It does not identify the employee.

Later topics will use:

```text
ROW_NUMBER()
RANK()
DENSE_RANK()
subqueries
CTEs
joins
```

to solve such problems.

---

# 97. Performance Awareness

Aggregation can be expensive on large datasets.

The database may need to:

```text
scan rows
→ group rows
→ maintain aggregate state
```

Possible physical strategies include:

```text
hash aggregation
sort/group aggregation
index-assisted strategies
parallel aggregation
```

The actual strategy is DBMS- and data-dependent.

Indexes can sometimes reduce work, but an index does not automatically make every aggregation query fast.

Use `EXPLAIN` / execution plans when performance matters.

---

# 98. Aggregation Checklist for Interviews

Before submitting an aggregation query, ask:

1. What is the source table?
2. Which rows should be filtered?
3. What defines a group?
4. Which aggregate is required?
5. Does NULL matter?
6. Do I need `COUNT(*)` or `COUNT(column)`?
7. Is the condition row-level or group-level?
8. Should it be `WHERE` or `HAVING`?
9. Do I need `DISTINCT`?
10. Do I need sorting?
11. Do I need top N?
12. Are ties important?
13. Am I accidentally selecting a non-grouped column?
14. Is the result supposed to contain one row or one row per group?

---

# 99. Mastery Checklist

## Aggregate Functions

- [ ] `COUNT(*)`
- [ ] `COUNT(column)`
- [ ] `COUNT(DISTINCT column)`
- [ ] `SUM`
- [ ] `AVG`
- [ ] `MIN`
- [ ] `MAX`
- [ ] Understand NULL behavior
- [ ] Understand empty-input behavior
- [ ] Understand `COALESCE` with aggregates

## GROUP BY

- [ ] Group by one column
- [ ] Group by multiple columns
- [ ] Group by expressions
- [ ] Understand group keys
- [ ] Understand grouping with NULL
- [ ] Know grouped SELECT rules

## HAVING

- [ ] Filter groups
- [ ] Use aggregate expressions in HAVING
- [ ] Distinguish HAVING from WHERE
- [ ] Use WHERE + GROUP BY + HAVING together

## Logical Processing

- [ ] FROM
- [ ] WHERE
- [ ] GROUP BY
- [ ] HAVING
- [ ] SELECT
- [ ] ORDER BY
- [ ] LIMIT
- [ ] Understand written vs logical order
- [ ] Understand logical vs physical execution

## Interview Patterns

- [ ] Count per group
- [ ] Sum per group
- [ ] Average per group
- [ ] Min/max per group
- [ ] At least N
- [ ] Total greater than X
- [ ] Average greater than X
- [ ] Top N groups
- [ ] Distinct count
- [ ] Conditional aggregation

---

# 100. Final Revision Sheet

```text
AGGREGATION
    COUNT → count
    SUM   → total
    AVG   → average
    MIN   → minimum
    MAX   → maximum

COUNT
    COUNT(*)       → rows
    COUNT(x)       → non-NULL x
    COUNT(DISTINCT x)
                   → distinct non-NULL x values

NULL
    COUNT(*)       → includes rows
    COUNT(x)       → ignores NULL
    SUM/AVG/MIN/MAX
                   → generally ignore NULL
    empty SUM/AVG/MIN/MAX
                   → generally NULL

GROUP BY
    rows
      ↓
    groups
      ↓
    aggregate per group

HAVING
    filters groups

WHERE
    filters rows

CORE LOGICAL ORDER

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
    ORDER BY
      ↓
    LIMIT

WRITTEN ORDER

    SELECT
    FROM
    WHERE
    GROUP BY
    HAVING
    ORDER BY
    LIMIT

KEY DISTINCTION

    WHERE
    → before grouping
    → row-level

    HAVING
    → after grouping
    → group-level

COMMON TEMPLATE

    SELECT group_col,
           AGG(value) AS metric
    FROM table
    WHERE row_condition
    GROUP BY group_col
    HAVING AGG(value) > threshold
    ORDER BY metric DESC
    LIMIT N;

COMMON PATTERNS

    COUNT PER GROUP
        GROUP BY x
        COUNT(*)

    TOTAL PER GROUP
        GROUP BY x
        SUM(value)

    AVERAGE PER GROUP
        GROUP BY x
        AVG(value)

    GROUPS WITH >= N
        GROUP BY x
        HAVING COUNT(*) >= N

    TOP N GROUPS
        GROUP BY x
        ORDER BY metric DESC
        LIMIT N

    DISTINCT COUNT
        COUNT(DISTINCT x)

    CONDITIONAL COUNT
        SUM(CASE WHEN condition THEN 1 ELSE 0 END)

MOST IMPORTANT INTERVIEW TRAP

    WHERE AVG(x) > ...
        ❌

    GROUP BY ...
    HAVING AVG(x) > ...
        ✓

SECOND MAJOR TRAP

    SELECT group_col, non_grouped_col, COUNT(*)
    GROUP BY group_col

    → invalid under standard grouping semantics
      unless the selected expression is otherwise
      determined/allowed by the DBMS's grouping rules.

THIRD MAJOR TRAP

    WHERE x > 70000
        ≠
    HAVING AVG(x) > 70000

    The first changes which rows enter the aggregate.
    The second filters groups after their aggregate is calculated.

FINAL DISTINCTION

    Logical processing order
        ≠
    Written SQL order
        ≠
    Physical execution plan
```

---

# 101. Mastery Standard

You have mastered SQL Aggregation when you can:

1. Choose the correct aggregate function immediately.
2. Explain `COUNT(*)` vs `COUNT(column)`.
3. Explain how NULL affects every core aggregate.
4. Group by one or multiple columns.
5. Use `HAVING` without confusing it with `WHERE`.
6. Combine `WHERE → GROUP BY → HAVING`.
7. Use `COUNT(DISTINCT ...)`.
8. Build top-N grouped queries.
9. Use conditional aggregation.
10. Identify invalid non-grouped SELECT expressions.
11. Explain why `WHERE AVG(...)` is incorrect.
12. Explain why `WHERE` and `HAVING` can produce different averages.
13. State the conceptual order:

```text
FROM
→ WHERE
→ GROUP BY
→ HAVING
→ SELECT
→ ORDER BY
→ LIMIT
```

14. Distinguish logical query processing from the database optimizer's physical execution plan.
15. Solve aggregation problems involving NULLs without guessing.

The target is:

```text
English requirement
    ↓
Identify row filter
    ↓
Identify grouping key
    ↓
Identify aggregate
    ↓
Identify group filter
    ↓
Identify ordering
    ↓
Identify top-N requirement
    ↓
Write query
    ↓
Check NULL + grouping edge cases
```

