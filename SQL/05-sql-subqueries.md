# 05. SQL Subqueries

> **Track:** SQL Interview Preparation  
> **Overall repository number:** 28  
> **SQL topic number:** 05  
> **Priority:** HIGH  
> **Target:** FAANG + top product companies | SDE + ML/AI Engineer  
> **Difficulty:** Easy → Medium → Hard, with Medium dominating

---

# 1. Overview

A **subquery** is a query nested inside another SQL statement.

Basic form:

```sql
SELECT ...
FROM ...
WHERE column OP (
    SELECT ...
);
```

Subqueries are used when one query needs the result of another query.

Core types/patterns for this topic:

```text
Scalar subquery
Multi-row subquery
Correlated subquery
EXISTS
NOT EXISTS
IN
NOT IN
```

The key question is not merely:

> "How do I write a subquery?"

It is:

> "What shape does the inner query return, and how should the outer query use that result?"

---

# 2. The Most Important Mental Model

Before writing a subquery, determine its result shape.

```text
Scalar subquery
→ one value

Multi-row subquery
→ multiple values, usually one column

Multi-column subquery
→ multiple rows/columns

Correlated subquery
→ depends on the current outer row

EXISTS
→ asks whether at least one matching row exists

NOT EXISTS
→ asks whether no matching row exists

IN
→ asks whether a value belongs to a returned set

NOT IN
→ asks whether a value is outside a returned set
```

The most important trap in this topic is:

```text
NOT IN + NULL
```

Because SQL uses three-valued logic.

---

# 3. Sample Schema

Use these tables throughout the topic.

## employees

| employee_id | name | department_id | salary | manager_id |
|---:|---|---:|---:|---:|
| 1 | Alice | 10 | 90000 | 3 |
| 2 | Bob | 10 | 75000 | 3 |
| 3 | Carol | 20 | 70000 | NULL |
| 4 | David | 30 | 65000 | 3 |
| 5 | Eva | NULL | 100000 | 3 |
| 6 | Frank | 20 | 60000 | NULL |
| 7 | Grace | 10 | 120000 | 3 |

## departments

| department_id | department_name |
|---:|---|
| 10 | Engineering |
| 20 | Sales |
| 30 | HR |
| 40 | Finance |

## projects

| project_id | project_name | department_id |
|---:|---|---:|
| 101 | API | 10 |
| 102 | Mobile | 10 |
| 103 | CRM | 20 |
| 104 | Payroll | 30 |

---

# 4. What Is a Subquery?

Example:

```sql
SELECT name
FROM employees
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
);
```

The inner query:

```sql
SELECT AVG(salary)
FROM employees
```

returns one value.

The outer query compares each employee's salary with that value.

Conceptually:

```text
inner query
→ average salary

outer query
→ employees whose salary > average
```

---

# 5. Subquery Locations

Subqueries can appear in several places.

Common forms include:

```sql
WHERE
HAVING
FROM
SELECT
```

Example in `WHERE`:

```sql
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
)
```

Example in `HAVING`:

```sql
HAVING COUNT(*) > (
    SELECT AVG(...)
)
```

Example in `FROM`:

```sql
FROM (
    SELECT ...
) t
```

Example in `SELECT`:

```sql
SELECT
    name,
    (SELECT ...) AS value
FROM employees;
```

The exact capabilities and restrictions vary by DBMS.

---

# 6. Scalar Subquery

A scalar subquery returns **at most one value** for the context in which it is used.

Example:

```sql
SELECT AVG(salary)
FROM employees;
```

This returns one aggregate value.

Use it as:

```sql
SELECT
    name,
    salary
FROM employees
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
);
```

---

# 7. Scalar Subquery Result Shape

Think:

```text
outer query
    |
    +-- compare salary
            |
            +-- scalar subquery
                    |
                    +-- one value
```

The inner query must produce a single scalar result for operators such as:

```text
=
>
<
>=
<=
<>
```

Example:

```sql
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
)
```

---

# 8. Scalar Subquery Returning Multiple Rows

This is invalid for a scalar context:

```sql
WHERE salary > (
    SELECT salary
    FROM employees
);
```

The inner query can return many salaries.

A scalar comparison:

```text
salary > [many rows]
```

is not valid in the ordinary scalar form.

If multiple values are intended, use:

```text
IN
ANY/SOME
ALL
```

or restructure the query.

`IN` is part of this topic; `ANY`/`ALL` are additional comparison-subquery operators worth recognizing but can be studied later.

---

# 9. Scalar Subquery Example: Above Average

```sql
SELECT
    name,
    salary
FROM employees
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
);
```

The inner query computes one average.

The outer query compares every employee against that average.

This is a canonical scalar-subquery pattern.

---

# 10. Scalar Subquery with HAVING

Question:

> Find departments whose average salary is above the overall average salary.

```sql
SELECT
    department_id,
    AVG(salary) AS avg_salary
FROM employees
GROUP BY department_id
HAVING AVG(salary) > (
    SELECT AVG(salary)
    FROM employees
);
```

The subquery returns one overall average.

The outer query computes one average per department.

Then `HAVING` compares:

```text
department average
>
overall average
```

---

# 11. Scalar Subquery with MIN/MAX

Question:

> Find employees with the maximum salary.

```sql
SELECT name, salary
FROM employees
WHERE salary = (
    SELECT MAX(salary)
    FROM employees
);
```

This is better than:

```sql
ORDER BY salary DESC
LIMIT 1
```

when the requirement is to return **all employees tied for the maximum salary**.

---

# 12. Multi-Row Subquery

A multi-row subquery returns multiple rows, commonly one column.

Example:

```sql
SELECT department_id
FROM employees
WHERE salary > 100000;
```

It may return:

```text
10
10
```

or multiple department IDs.

A multi-row result is appropriate for operators such as:

```sql
IN
NOT IN
EXISTS
NOT EXISTS
```

---

# 13. IN

`IN` checks whether a value matches one of the values returned by a subquery.

Example:

```sql
SELECT name
FROM employees
WHERE department_id IN (
    SELECT department_id
    FROM departments
    WHERE department_name IN ('Engineering', 'Sales')
);
```

Conceptually:

```text
inner query
→ allowed department IDs

outer query
→ keep employees whose department_id is in that set
```

---

# 14. IN with a Value List

`IN` can also use a literal list:

```sql
WHERE department_id IN (10, 20, 30)
```

Subquery form:

```sql
WHERE department_id IN (
    SELECT department_id
    FROM departments
    WHERE department_name IN ('Engineering', 'Sales', 'HR')
)
```

The subquery dynamically generates the set.

---

# 15. IN and Equality

Conceptually:

```sql
x IN (a, b, c)
```

is similar to:

```sql
x = a
OR x = b
OR x = c
```

For a subquery:

```sql
x IN (SELECT y FROM B)
```

means:

```text
Does x match at least one returned y?
```

NULL semantics must still be considered.

---

# 16. IN with Duplicate Values

Suppose the subquery returns:

```text
10
10
20
20
20
```

For membership testing:

```sql
x IN (subquery)
```

the duplicates generally do not change whether x is a member.

Membership is the important question.

However, if you rewrite the logic using a join, duplicate matches can affect output row counts.

This is a key difference between:

```text
membership
```

and:

```text
join cardinality
```

---

# 17. IN vs JOIN

These can sometimes express similar requirements.

Example:

```sql
SELECT e.*
FROM employees e
WHERE e.department_id IN (
    SELECT d.department_id
    FROM departments d
    WHERE d.department_name = 'Engineering'
);
```

Equivalent inner-join style:

```sql
SELECT e.*
FROM employees e
JOIN departments d
    ON e.department_id = d.department_id
WHERE d.department_name = 'Engineering';
```

But they are not universally interchangeable without considering:

- duplicate rows
- required output
- null behavior
- existence semantics
- optimizer behavior

---

# 18. EXISTS

`EXISTS` asks:

> Does the subquery return at least one row?

Example:

```sql
SELECT e.name
FROM employees e
WHERE EXISTS (
    SELECT 1
    FROM projects p
    WHERE p.department_id = e.department_id
);
```

For each employee, the subquery checks whether their department has at least one project.

The actual value selected by the subquery is irrelevant to `EXISTS`.

Therefore:

```sql
SELECT 1
```

is conventional.

---

# 19. EXISTS Does Not Need SELECT 1

These are logically equivalent for `EXISTS`:

```sql
SELECT 1
FROM projects p
WHERE ...
```

and:

```sql
SELECT *
FROM projects p
WHERE ...
```

and:

```sql
SELECT p.project_id
FROM projects p
WHERE ...
```

`EXISTS` only cares whether at least one row exists.

`SELECT 1` communicates that intent clearly.

---

# 20. EXISTS and Correlation

The common form is correlated:

```sql
SELECT e.name
FROM employees e
WHERE EXISTS (
    SELECT 1
    FROM projects p
    WHERE p.department_id = e.department_id
);
```

The inner query refers to:

```sql
e.department_id
```

from the outer query.

Therefore the inner query's result depends on the current outer employee row.

---

# 21. Correlated Subquery

A correlated subquery references columns from the outer query.

Example:

```sql
SELECT
    e.name,
    e.salary
FROM employees e
WHERE e.salary > (
    SELECT AVG(e2.salary)
    FROM employees e2
    WHERE e2.department_id = e.department_id
);
```

For each employee, the subquery calculates the average salary of that employee's department.

Then the outer salary is compared against the department average.

---

# 22. Correlated Subquery Mental Model

Conceptually:

```text
for each outer employee e:

    run/consider subquery using e.department_id

    calculate department average

    compare e.salary with that average
```

This is a useful logical model.

However, do not assume the DBMS literally executes the inner query from scratch once per outer row. The optimizer may transform a correlated query into a more efficient plan.

---

# 23. Correlated vs Non-Correlated

### Non-correlated

```sql
SELECT AVG(salary)
FROM employees;
```

It does not refer to the outer query.

### Correlated

```sql
SELECT AVG(e2.salary)
FROM employees e2
WHERE e2.department_id = e.department_id;
```

It refers to:

```text
e.department_id
```

from the outer query.

---

# 24. Correlated Subquery: Above Department Average

```sql
SELECT
    e.name,
    e.department_id,
    e.salary
FROM employees e
WHERE e.salary > (
    SELECT AVG(e2.salary)
    FROM employees e2
    WHERE e2.department_id = e.department_id
);
```

This returns employees whose salary is above the average for their own department.

This is one of the most important correlated-subquery patterns.

---

# 25. Correlated EXISTS

Example:

> Find employees whose department has at least one project.

```sql
SELECT e.name
FROM employees e
WHERE EXISTS (
    SELECT 1
    FROM projects p
    WHERE p.department_id = e.department_id
);
```

Correlation:

```sql
p.department_id = e.department_id
```

The outer row determines which department the inner query checks.

---

# 26. NOT EXISTS

`NOT EXISTS` asks:

> Does the subquery return zero rows?

Example:

```sql
SELECT e.name
FROM employees e
WHERE NOT EXISTS (
    SELECT 1
    FROM projects p
    WHERE p.department_id = e.department_id
);
```

Meaning:

```text
Return employees whose department has no project.
```

---

# 27. NOT EXISTS as an Anti-Join

This is a standard anti-existence pattern:

```sql
SELECT e.*
FROM employees e
WHERE NOT EXISTS (
    SELECT 1
    FROM departments d
    WHERE d.department_id = e.department_id
);
```

Meaning:

```text
employees with no matching department
```

Equivalent-style pattern:

```sql
SELECT e.*
FROM employees e
LEFT JOIN departments d
    ON d.department_id = e.department_id
WHERE d.department_id IS NULL;
```

The two approaches can have different physical execution plans but express the same logical anti-match requirement under appropriate assumptions.

---

# 28. EXISTS vs IN

For a basic membership problem:

```sql
WHERE e.department_id IN (
    SELECT d.department_id
    FROM departments d
)
```

and:

```sql
WHERE EXISTS (
    SELECT 1
    FROM departments d
    WHERE d.department_id = e.department_id
)
```

can express similar matching logic.

But they differ conceptually:

```text
IN
→ membership in a returned set

EXISTS
→ existence of at least one matching row
```

The choice should reflect the requirement.

---

# 29. EXISTS and Duplicate Rows

Suppose the inner query finds:

```text
5 matching rows
```

For:

```sql
WHERE EXISTS (...)
```

the outer row is still simply accepted once.

`EXISTS` does not multiply the outer result.

This is a major distinction from a regular join.

---

# 30. JOIN vs EXISTS

Suppose:

```text
one department
→ 5 projects
```

A join:

```sql
employees
JOIN projects
```

can produce five rows for an employee.

But:

```sql
WHERE EXISTS (...)
```

still produces one outer employee row.

Therefore:

```text
JOIN
→ combines matching rows

EXISTS
→ tests whether a match exists
```

This distinction is extremely important.

---

# 31. NOT EXISTS vs LEFT JOIN

Both can express:

```text
A rows with no B match
```

### NOT EXISTS

```sql
SELECT a.*
FROM A a
WHERE NOT EXISTS (
    SELECT 1
    FROM B b
    WHERE b.key = a.key
);
```

### LEFT JOIN

```sql
SELECT a.*
FROM A a
LEFT JOIN B b
    ON b.key = a.key
WHERE b.key IS NULL;
```

The `NOT EXISTS` version is often especially clear when the question is explicitly about existence.

---

# 32. NOT IN

`NOT IN` checks whether a value is not equal to any value in a returned set.

Example:

```sql
SELECT name
FROM employees
WHERE department_id NOT IN (
    SELECT department_id
    FROM departments
);
```

This looks straightforward.

But:

```text
NOT IN + NULL
```

is a critical SQL trap.

---

# 33. The NOT IN + NULL Trap

Suppose the subquery returns:

```text
10
20
NULL
```

Consider:

```sql
WHERE x NOT IN (10, 20, NULL)
```

Conceptually this becomes:

```text
x <> 10
AND
x <> 20
AND
x <> NULL
```

The last comparison is:

```text
x <> NULL
→ UNKNOWN
```

Therefore the whole predicate can become:

```text
UNKNOWN
```

rather than TRUE.

Since `WHERE` retains only TRUE, rows can unexpectedly disappear.

---

# 34. Example of NOT IN Failure

Suppose:

```text
employees.department_id

10
20
30
```

Subquery:

```text
10
20
NULL
```

Query:

```sql
SELECT *
FROM employees
WHERE department_id NOT IN (
    SELECT department_id
    FROM departments;
);
```

The NULL in the subquery can make the predicate UNKNOWN for candidate values.

Therefore `NOT IN` can return no rows or fewer rows than expected.

This is one of the most important SQL interview traps.

---

# 35. Safe NOT IN Pattern

If you know the subquery column cannot be NULL:

```sql
WHERE x NOT IN (
    SELECT y
    FROM B
)
```

can be safe with respect to this particular NULL trap.

Otherwise, explicitly remove NULLs:

```sql
WHERE x NOT IN (
    SELECT y
    FROM B
    WHERE y IS NOT NULL
)
```

But for anti-existence logic, `NOT EXISTS` is often safer and clearer.

---

# 36. NOT EXISTS vs NOT IN

Suppose:

```sql
A.key
```

must not exist in:

```sql
B.key
```

Prefer:

```sql
SELECT A.*
FROM A
WHERE NOT EXISTS (
    SELECT 1
    FROM B
    WHERE B.key = A.key
);
```

This avoids the classic subquery-NULL problem associated with `NOT IN`.

---

# 37. Why NOT EXISTS Is NULL-Safer

`NOT EXISTS` asks:

```text
Does a matching row exist?
```

If no matching row exists:

```text
NOT EXISTS = TRUE
```

A NULL elsewhere in the inner table does not automatically poison the existence predicate.

By contrast, `NOT IN` uses comparisons against the returned set, so a NULL element can introduce UNKNOWN.

---

# 38. IN + NULL

`IN` also has NULL behavior, but the common catastrophic trap is especially important with `NOT IN`.

Example:

```sql
x IN (10, 20, NULL)
```

If:

```text
x = 10
```

the result is TRUE.

If:

```text
x = 30
```

the comparisons include:

```text
30 = 10 → FALSE
30 = 20 → FALSE
30 = NULL → UNKNOWN
```

Therefore:

```text
FALSE OR FALSE OR UNKNOWN
→ UNKNOWN
```

So the row is not selected by `WHERE`.

---

# 39. NOT IN Truth Table

For:

```text
x NOT IN (a, b, NULL)
```

consider:

```text
x = a
```

Then:

```text
x NOT IN ...
→ FALSE
```

For a value matching neither non-NULL value:

```text
x <> a
AND x <> b
AND x <> NULL
```

becomes:

```text
TRUE
AND TRUE
AND UNKNOWN
→ UNKNOWN
```

Therefore:

```text
WHERE
```

does not retain it.

---

# 40. NOT EXISTS Equivalent

Instead of:

```sql
WHERE e.department_id NOT IN (
    SELECT d.department_id
    FROM departments d
)
```

use:

```sql
WHERE NOT EXISTS (
    SELECT 1
    FROM departments d
    WHERE d.department_id = e.department_id
)
```

The second form directly expresses:

```text
no matching department exists
```

---

# 41. Scalar vs Multi-Row Operator Choice

A useful decision table:

| Subquery result | Typical operator |
|---|---|
| One value | `=`, `>`, `<`, `>=`, `<=` |
| Multiple values | `IN`, `NOT IN` |
| Need existence | `EXISTS`, `NOT EXISTS` |
| Depends on outer row | Correlated subquery |

Examples:

```sql
salary > (SELECT AVG(salary) ...)
```

```sql
department_id IN (SELECT department_id ...)
```

```sql
WHERE EXISTS (SELECT 1 ...)
```

```sql
WHERE NOT EXISTS (SELECT 1 ...)
```

---

# 42. Subquery Result Shape

Before writing:

```sql
WHERE x = (subquery)
```

ask:

```text
Does subquery return exactly one value?
```

Before writing:

```sql
WHERE x IN (subquery)
```

ask:

```text
Does subquery return one compatible column?
```

Before writing:

```sql
WHERE EXISTS (subquery)
```

ask:

```text
Does the subquery represent the matching condition?
```

This simple check prevents many syntax and logic errors.

---

# 43. Multi-Column Subqueries

SQL can compare row values in systems that support row constructors.

Example:

```sql
WHERE (department_id, salary) IN (
    SELECT department_id, MAX(salary)
    FROM employees
    GROUP BY department_id
)
```

This asks for rows whose pair:

```text
(department_id, salary)
```

matches a returned pair.

Exact row-constructor support and behavior can vary by DBMS.

For general interview preparation, understand the concept, but use simpler joins/window functions when they make the intended logic clearer.

---

# 44. Subquery in FROM

A subquery in `FROM` is a derived table.

Example:

```sql
SELECT *
FROM (
    SELECT
        department_id,
        AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department_id
) t
WHERE t.avg_salary > 70000;
```

The inner query creates an intermediate result.

The outer query filters it.

---

# 45. Derived Table and Grain

Inner query:

```sql
GROUP BY department_id
```

creates:

```text
one row per department
```

Therefore the derived table:

```text
t
```

has the grain:

```text
one row per department
```

The outer query then works on that grain.

Always track grain through nested queries.

---

# 46. Subquery in SELECT

A scalar subquery can appear in the SELECT list.

Example:

```sql
SELECT
    e.name,
    (
        SELECT AVG(e2.salary)
        FROM employees e2
    ) AS overall_avg_salary
FROM employees e;
```

Each output row receives the same scalar value.

A correlated version could vary by outer row.

---

# 47. Correlated Scalar Subquery

Example:

```sql
SELECT
    e.name,
    e.salary,
    (
        SELECT AVG(e2.salary)
        FROM employees e2
        WHERE e2.department_id = e.department_id
    ) AS department_avg
FROM employees e;
```

Each employee row receives its department's average.

This preserves employee-level rows while calculating a related aggregate.

A window function is often more natural for this problem, but the correlated subquery is important to understand.

---

# 48. Subquery in HAVING

Example:

```sql
SELECT
    department_id,
    AVG(salary) AS avg_salary
FROM employees
GROUP BY department_id
HAVING AVG(salary) > (
    SELECT AVG(salary)
    FROM employees
);
```

The scalar subquery supplies the comparison value.

---

# 49. Nested Subqueries

Subqueries can contain subqueries.

Example:

```sql
SELECT name
FROM employees
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
    WHERE department_id IN (
        SELECT department_id
        FROM departments
        WHERE department_name IN ('Engineering', 'Sales')
    )
);
```

Nested subqueries can be correct, but excessive nesting can reduce readability.

Prefer:

```text
clear CTEs
joins
derived tables
```

when they communicate the logic better.

---

# 50. Correlated Subquery and Aggregation

Question:

> Find employees earning more than their department average.

```sql
SELECT
    e.name,
    e.salary
FROM employees e
WHERE e.salary > (
    SELECT AVG(e2.salary)
    FROM employees e2
    WHERE e2.department_id = e.department_id
);
```

Logical steps for each employee:

```text
1. Identify employee's department.
2. Find employees in that department.
3. Calculate department average.
4. Compare current employee salary.
5. Keep employee if salary is greater.
```

---

# 51. EXISTS and SELECT List

This:

```sql
WHERE EXISTS (
    SELECT 1
    FROM projects p
    WHERE p.department_id = e.department_id
)
```

does not mean:

> Select the number 1.

It means:

> Return TRUE if this inner query has at least one row.

The projection is ignored for existence semantics.

---

# 52. EXISTS Stops Being a Counting Operation

If five rows match:

```sql
EXISTS (...)
```

is still just:

```text
TRUE
```

It does not become:

```text
5
```

If you need the count, use:

```sql
COUNT(*)
```

or aggregation.

---

# 53. EXISTS and NULL

Suppose the inner table contains:

```text
NULL
```

`EXISTS` can still return TRUE if that row satisfies the inner `WHERE` condition.

Existence is about rows, not whether selected columns contain non-NULL values.

Example:

```sql
WHERE EXISTS (
    SELECT 1
    FROM B
    WHERE B.some_column IS NULL
)
```

can legitimately be TRUE.

---

# 54. NOT EXISTS and NULL

Similarly:

```sql
WHERE NOT EXISTS (
    SELECT 1
    FROM B
    WHERE B.key = A.key
)
```

depends on whether a matching row satisfying:

```sql
B.key = A.key
```

exists.

A NULL `B.key` does not match an ordinary equality comparison to a non-NULL `A.key`.

This does not cause the same global poisoning effect as `NOT IN`.

---

# 55. Correlation Must Be Intentional

Compare:

```sql
SELECT e.name
FROM employees e
WHERE EXISTS (
    SELECT 1
    FROM projects p
);
```

This is not correlated.

If at least one project exists anywhere, the condition is true for every employee.

A correlated version:

```sql
WHERE EXISTS (
    SELECT 1
    FROM projects p
    WHERE p.department_id = e.department_id
)
```

checks the employee's department.

Missing the correlation condition can completely change the result.

---

# 56. Common Correlation Bug

Intended:

> Employees whose department has a project.

Wrong:

```sql
SELECT e.name
FROM employees e
WHERE EXISTS (
    SELECT 1
    FROM projects p
);
```

Meaning:

> There exists at least one project anywhere.

Correct:

```sql
SELECT e.name
FROM employees e
WHERE EXISTS (
    SELECT 1
    FROM projects p
    WHERE p.department_id = e.department_id
);
```

Meaning:

> A project exists for this employee's department.

---

# 57. IN Subquery and NULL Outer Value

Suppose:

```text
e.department_id = NULL
```

and:

```sql
WHERE e.department_id IN (
    SELECT department_id
    FROM departments
)
```

The membership comparison is not TRUE because NULL does not equal an ordinary value.

Therefore the employee is not selected.

If the business meaning treats NULL specially, handle it explicitly.

---

# 58. NOT IN Outer NULL

Suppose:

```text
e.department_id = NULL
```

Then:

```sql
e.department_id NOT IN (...)
```

also does not evaluate to TRUE.

Therefore the row is not selected.

Again:

```text
NULL is not a normal value.
```

---

# 59. IN vs EXISTS with Duplicates

Suppose a department appears five times in the inner query.

For:

```sql
WHERE e.department_id IN (
    SELECT department_id
    FROM B
)
```

the outer row is either:

```text
selected
```

or:

```text
not selected
```

not repeated five times.

For a join:

```sql
FROM employees e
JOIN B
    ON e.department_id = B.department_id
```

the employee row can appear five times.

This distinction is essential.

---

# 60. EXISTS vs JOIN for Existence

Requirement:

> Employees belonging to departments with projects.

Use:

```sql
SELECT e.*
FROM employees e
WHERE EXISTS (
    SELECT 1
    FROM projects p
    WHERE p.department_id = e.department_id
);
```

A join can also work:

```sql
SELECT DISTINCT e.*
FROM employees e
JOIN projects p
    ON p.department_id = e.department_id;
```

But the join creates matching row combinations and may require `DISTINCT`.

`EXISTS` directly represents the existence requirement.

---

# 61. NOT EXISTS vs LEFT JOIN for Anti-Matching

Requirement:

> Employees whose department has no project.

Using `NOT EXISTS`:

```sql
SELECT e.*
FROM employees e
WHERE NOT EXISTS (
    SELECT 1
    FROM projects p
    WHERE p.department_id = e.department_id
);
```

Using `LEFT JOIN`:

```sql
SELECT DISTINCT e.*
FROM employees e
LEFT JOIN projects p
    ON p.department_id = e.department_id
WHERE p.project_id IS NULL;
```

The first naturally expresses the existence condition and avoids project-driven row multiplication.

---

# 62. Subquery vs Aggregation

Some problems can be solved with either.

Question:

> Employees whose salary is above overall average.

Subquery:

```sql
SELECT e.*
FROM employees e
WHERE e.salary > (
    SELECT AVG(salary)
    FROM employees
);
```

A window-function solution can also solve related problems while retaining row-level detail, but window functions are a separate topic.

Choose the approach that best matches the required output and readability.

---

# 63. Subquery vs JOIN

Question:

> Employees in Engineering.

Subquery:

```sql
SELECT e.*
FROM employees e
WHERE e.department_id IN (
    SELECT d.department_id
    FROM departments d
    WHERE d.department_name = 'Engineering'
);
```

Join:

```sql
SELECT e.*
FROM employees e
JOIN departments d
    ON d.department_id = e.department_id
WHERE d.department_name = 'Engineering';
```

Both can be valid.

Consider:

```text
result grain
duplicates
NULL semantics
existence vs data retrieval
readability
```

---

# 64. Choosing EXISTS

Prefer the mental model:

```text
"Does at least one matching row exist?"
```

Then:

```sql
EXISTS (...)
```

Examples:

```text
customers with at least one order
employees with at least one project
products with at least one review
departments with at least one employee
```

---

# 65. Choosing NOT EXISTS

Use:

```text
"Does no matching row exist?"
```

Examples:

```text
customers with no orders
employees with no project
products with no review
departments with no employees
```

Pattern:

```sql
WHERE NOT EXISTS (
    SELECT 1
    FROM child
    WHERE child.parent_id = parent.parent_id
)
```

---

# 66. Choosing IN

Use:

```text
"Is this value among the values returned by the subquery?"
```

Example:

```sql
WHERE department_id IN (
    SELECT department_id
    FROM departments
    WHERE department_name LIKE 'Eng%'
)
```

---

# 67. Choosing NOT IN

Use only when:

```text
"Is this value outside the returned set?"
```

and NULL behavior has been explicitly considered.

Safer form when appropriate:

```sql
WHERE x NOT IN (
    SELECT y
    FROM B
    WHERE y IS NOT NULL
)
```

For anti-existence requirements, strongly consider:

```sql
NOT EXISTS
```

instead.

---

# 68. Scalar Subquery Pattern Library

## Above overall average

```sql
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
)
```

## Equal to maximum

```sql
WHERE salary = (
    SELECT MAX(salary)
    FROM employees
)
```

## Below minimum threshold

```sql
WHERE value < (
    SELECT MIN(value)
    FROM table_name
)
```

---

# 69. Multi-Row IN Pattern Library

## Rows belonging to selected groups

```sql
WHERE group_id IN (
    SELECT group_id
    FROM groups
    WHERE condition
)
```

## Products sold in selected stores

```sql
WHERE store_id IN (
    SELECT store_id
    FROM stores
    WHERE region = 'WEST'
)
```

---

# 70. EXISTS Pattern Library

## At least one child

```sql
WHERE EXISTS (
    SELECT 1
    FROM child c
    WHERE c.parent_id = p.parent_id
)
```

## At least one qualifying child

```sql
WHERE EXISTS (
    SELECT 1
    FROM child c
    WHERE c.parent_id = p.parent_id
      AND c.status = 'ACTIVE'
)
```

---

# 71. NOT EXISTS Pattern Library

## No child

```sql
WHERE NOT EXISTS (
    SELECT 1
    FROM child c
    WHERE c.parent_id = p.parent_id
)
```

## No qualifying child

```sql
WHERE NOT EXISTS (
    SELECT 1
    FROM child c
    WHERE c.parent_id = p.parent_id
      AND c.status = 'ACTIVE'
)
```

---

# 72. Correlated Aggregate Pattern

General form:

```sql
SELECT a.*
FROM A a
WHERE a.value > (
    SELECT AVG(b.value)
    FROM B b
    WHERE b.group_id = a.group_id
);
```

Meaning:

```text
Compare each A row
against an aggregate of its own group.
```

This is a high-value interview pattern.

---

# 73. Correlated EXISTS Pattern

General form:

```sql
SELECT p.*
FROM parent p
WHERE EXISTS (
    SELECT 1
    FROM child c
    WHERE c.parent_id = p.parent_id
      AND c.condition = ...
);
```

Meaning:

```text
Return parent rows having at least one qualifying child.
```

---

# 74. Correlated NOT EXISTS Pattern

General form:

```sql
SELECT p.*
FROM parent p
WHERE NOT EXISTS (
    SELECT 1
    FROM child c
    WHERE c.parent_id = p.parent_id
      AND c.condition = ...
);
```

Meaning:

```text
Return parent rows having no qualifying child.
```

---

# 75. Subquery Evaluation: Logical vs Physical

A correlated subquery is often explained conceptually as:

```text
for each outer row
    evaluate inner query
```

This is useful for understanding semantics.

But the database optimizer may transform it into:

```text
join
semi-join
anti-join
aggregation
other optimized plan
```

Therefore:

```text
conceptual correlation
≠ guaranteed repeated physical execution
```

This is the same distinction as logical SQL processing versus physical execution.

---

# 76. Semi-Join Mental Model

`EXISTS` can be understood conceptually as a **semi-join**:

```text
Return A rows for which B has at least one match.
```

Unlike a normal join:

```text
matching B rows do not multiply A's output.
```

This is why `EXISTS` is natural for existence questions.

---

# 77. Anti-Join Mental Model

`NOT EXISTS` can be understood as an anti-join:

```text
Return A rows for which B has no matching row.
```

This connects:

```text
NOT EXISTS
```

with:

```text
LEFT JOIN ... IS NULL
```

as two common ways to express anti-matching.

---

# 78. Subquery and NULL Summary

```text
Scalar subquery
→ NULL can be returned as its single value

IN
→ NULL in the result can produce UNKNOWN for non-matching values

NOT IN
→ NULL in the subquery result can cause major unexpected filtering

EXISTS
→ asks whether a row exists
→ NULL values do not inherently prevent existence

NOT EXISTS
→ asks whether no matching row exists
→ generally safer than NOT IN for anti-match logic
```

---

# 79. NOT IN Safety Checklist

Before using:

```sql
NOT IN (subquery)
```

ask:

```text
Can the subquery return NULL?
```

If yes:

```text
Do not blindly use NOT IN.
```

Options:

```sql
WHERE x NOT IN (
    SELECT y
    FROM B
    WHERE y IS NOT NULL
)
```

or often better:

```sql
WHERE NOT EXISTS (
    SELECT 1
    FROM B
    WHERE B.y = A.x
)
```

---

# 80. Common Mistakes

## Mistake 1 — Scalar subquery returns many rows

Wrong:

```sql
WHERE salary > (
    SELECT salary
    FROM employees
)
```

unless the inner query is guaranteed to return one row.

---

## Mistake 2 — Using WHERE for existence

For:

```text
Does a related row exist?
```

consider:

```sql
EXISTS
```

rather than creating unnecessary row multiplication with a join.

---

## Mistake 3 — Using NOT IN with nullable subquery values

Danger:

```sql
NOT IN (SELECT nullable_column ...)
```

A NULL can make the predicate UNKNOWN.

---

## Mistake 4 — Forgetting correlation

Wrong:

```sql
WHERE EXISTS (
    SELECT 1
    FROM projects
)
```

when you actually need:

```text
a project for this department
```

Correct:

```sql
WHERE EXISTS (
    SELECT 1
    FROM projects p
    WHERE p.department_id = e.department_id
)
```

---

## Mistake 5 — Assuming EXISTS counts matches

It does not.

```text
1 match → TRUE
100 matches → TRUE
```

---

## Mistake 6 — Assuming IN duplicates outer rows

`IN` is a membership test.

A join can multiply rows; `IN` does not do so merely because the subquery has duplicates.

---

## Mistake 7 — Treating NULL like a normal value

Remember:

```text
NULL = NULL → UNKNOWN
```

---

## Mistake 8 — Blindly rewriting NOT EXISTS as NOT IN

These are not safely interchangeable when NULLs are possible.

---

# 81. GATE / CS Theory

## 81.1 Scalar Subquery

A scalar subquery supplies one value to a scalar expression/context.

---

## 81.2 Multi-Row Subquery

Returns multiple rows and is commonly used with:

```text
IN
NOT IN
EXISTS
NOT EXISTS
```

---

## 81.3 Correlated Subquery

References one or more columns from the outer query.

Its result can depend on the current outer row.

---

## 81.4 EXISTS

True when the subquery produces at least one row.

The selected values are irrelevant to the existence test.

---

## 81.5 NOT EXISTS

True when the subquery produces no rows.

---

## 81.6 IN

Tests membership against the values returned by a subquery.

---

## 81.7 NOT IN

Tests non-membership but is sensitive to NULL values in the subquery result.

---

## 81.8 Three-Valued Logic

SQL predicates can evaluate to:

```text
TRUE
FALSE
UNKNOWN
```

`WHERE` retains only TRUE.

This explains:

```text
NOT IN + NULL
```

behavior.

---

# 82. GATE-Style Solved Questions

> These are **GATE-style practice questions**, not claims about exact official GATE wording or years.

## Question 1 — Scalar Subquery

Given:

```text
salary:
100
200
300
```

What does this query return?

```sql
SELECT *
FROM employees
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
);
```

### Solution

Inner query:

```text
AVG = (100 + 200 + 300) / 3
    = 200
```

Outer condition:

```text
salary > 200
```

Only:

```text
300
```

is selected.

---

# 83. Question 2 — EXISTS

Suppose:

```text
A:
id
1
2
3

B:
a_id
1
1
3
```

Evaluate:

```sql
SELECT A.id
FROM A
WHERE EXISTS (
    SELECT 1
    FROM B
    WHERE B.a_id = A.id
);
```

### Solution

For:

```text
A.id = 1
```

B has matches.

For:

```text
A.id = 2
```

B has no match.

For:

```text
A.id = 3
```

B has a match.

Result:

```text
1
3
```

Even though `A.id = 1` has two B matches, it appears once.

---

# 84. Question 3 — NOT EXISTS

Using the same data:

```sql
SELECT A.id
FROM A
WHERE NOT EXISTS (
    SELECT 1
    FROM B
    WHERE B.a_id = A.id
);
```

### Solution

Only:

```text
A.id = 2
```

has no matching B row.

Answer:

```text
2
```

---

# 85. Question 4 — NOT IN + NULL

Suppose:

```text
B.x:
10
20
NULL
```

Evaluate conceptually:

```sql
SELECT *
FROM A
WHERE A.x NOT IN (
    SELECT B.x
    FROM B
);
```

for:

```text
A.x = 30
```

### Solution

Equivalent comparison idea:

```text
30 <> 10 → TRUE
30 <> 20 → TRUE
30 <> NULL → UNKNOWN
```

Therefore:

```text
TRUE AND TRUE AND UNKNOWN
→ UNKNOWN
```

`WHERE` keeps only TRUE.

Therefore the row is not selected.

---

# 86. Question 5 — Correlated Subquery

Table:

```text
department | salary
-----------|-------
A          | 100
A          | 200
B          | 300
B          | 100
```

Query:

```sql
SELECT *
FROM employees e
WHERE salary > (
    SELECT AVG(e2.salary)
    FROM employees e2
    WHERE e2.department = e.department
);
```

Which rows qualify?

### Solution

Department A:

```text
AVG = 150
```

Rows above 150:

```text
A | 200
```

Department B:

```text
AVG = 200
```

Rows above 200:

```text
B | 300
```

Answer:

```text
A | 200
B | 300
```

---

# 87. Question 6 — IN vs JOIN Multiplication

Suppose:

```text
A:
id
1

B:
a_id
1
1
1
```

Compare:

```sql
SELECT *
FROM A
WHERE id IN (
    SELECT a_id
    FROM B
);
```

with:

```sql
SELECT A.*
FROM A
JOIN B
    ON A.id = B.a_id;
```

### Solution

The `IN` query asks:

```text
Does 1 belong to the returned set?
```

Yes.

Therefore A contributes:

```text
1 row
```

The join combines A with all three B matches:

```text
3 rows
```

Therefore:

```text
IN
→ membership
→ no row multiplication

JOIN
→ row combination
→ can multiply rows
```

---

# 88. Interview Practice — Easy

## Q1

Find employees earning above the overall average salary.

```sql
SELECT name, salary
FROM employees
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
);
```

---

## Q2

Find employees with the maximum salary.

```sql
SELECT name, salary
FROM employees
WHERE salary = (
    SELECT MAX(salary)
    FROM employees
);
```

---

## Q3

Find employees belonging to Engineering.

```sql
SELECT e.*
FROM employees e
WHERE e.department_id IN (
    SELECT d.department_id
    FROM departments d
    WHERE d.department_name = 'Engineering'
);
```

---

# 89. Interview Practice — Medium

## Q4

Find employees whose department has at least one project.

```sql
SELECT e.*
FROM employees e
WHERE EXISTS (
    SELECT 1
    FROM projects p
    WHERE p.department_id = e.department_id
);
```

---

## Q5

Find employees whose department has no projects.

```sql
SELECT e.*
FROM employees e
WHERE NOT EXISTS (
    SELECT 1
    FROM projects p
    WHERE p.department_id = e.department_id
);
```

---

## Q6

Find employees earning above their department average.

```sql
SELECT e.*
FROM employees e
WHERE e.salary > (
    SELECT AVG(e2.salary)
    FROM employees e2
    WHERE e2.department_id = e.department_id
);
```

---

## Q7

Find departments whose average salary is above the company-wide average.

```sql
SELECT
    department_id,
    AVG(salary) AS avg_salary
FROM employees
GROUP BY department_id
HAVING AVG(salary) > (
    SELECT AVG(salary)
    FROM employees
);
```

---

# 90. Interview Practice — Harder

## Q8

Find customers who have placed at least one order but have never placed a cancelled order.

```sql
SELECT c.*
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
)
AND NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
      AND o.status = 'CANCELLED'
);
```

Mental model:

```text
must have qualifying existence
AND
must have no disqualifying existence
```

---

# 91. Harder Pattern — At Least One and No Matching Row

A very reusable pattern:

```sql
WHERE EXISTS (
    SELECT 1
    FROM child
    WHERE child.parent_id = parent.id
      AND child.condition = ...
)
AND NOT EXISTS (
    SELECT 1
    FROM child
    WHERE child.parent_id = parent.id
      AND child.bad_condition = ...
)
```

This appears frequently in advanced SQL interviews.

---

# 92. Harder Pattern — Above Group Average

General:

```sql
SELECT a.*
FROM A a
WHERE a.value > (
    SELECT AVG(b.value)
    FROM A b
    WHERE b.group_id = a.group_id
);
```

Recognize immediately:

```text
"above their group average"
→ correlated aggregate subquery
```

A later window-function topic will provide another important solution.

---

# 93. Harder Pattern — No Related Records

English:

> Customers with no orders.

Immediate pattern:

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

# 94. Harder Pattern — Related Records Exist

English:

> Customers with at least one order.

```sql
SELECT c.*
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
);
```

---

# 95. Harder Pattern — Maximum with Ties

English:

> Return all employees earning the maximum salary.

```sql
SELECT e.*
FROM employees e
WHERE e.salary = (
    SELECT MAX(salary)
    FROM employees
);
```

This returns all rows tied at the maximum.

Contrast:

```sql
ORDER BY salary DESC
LIMIT 1
```

which normally returns only one row.

---

# 96. Subquery Selection Decision Tree

```text
What does the inner query produce?

        ONE VALUE
           |
           ├─ Compare with =, >, <, >=, <=
           |
           └─ Example:
              salary > (SELECT AVG(...))

        MANY VALUES
           |
           ├─ Membership?
           |      → IN / NOT IN
           |
           └─ Existence?
                  → EXISTS / NOT EXISTS

        DEPENDS ON OUTER ROW
           |
           → CORRELATED SUBQUERY
```

---

# 97. EXISTS Decision Rule

Ask:

> "Do I care about the matching rows themselves, or only whether at least one exists?"

If only existence:

```sql
EXISTS
```

If no matching row:

```sql
NOT EXISTS
```

If you need actual columns from the matching rows:

```text
JOIN
```

may be more appropriate.

---

# 98. IN Decision Rule

Ask:

> "Do I have a value that needs to belong to a set produced by another query?"

Use:

```sql
IN
```

Example:

```text
employee.department_id
IN
departments selected by some condition
```

---

# 99. NOT IN Decision Rule

Ask:

> "Do I need non-membership?"

Then consider:

```sql
NOT IN
```

but immediately ask:

```text
Can the subquery return NULL?
```

If yes, strongly consider:

```sql
NOT EXISTS
```

---

# 100. Scalar Subquery Decision Rule

Ask:

> "Do I need one computed value to compare against every outer row?"

Typical patterns:

```text
above average
below average
equal to maximum
equal to minimum
greater than total threshold
```

Use a scalar subquery.

---

# 101. Correlated Subquery Decision Rule

Ask:

> "Does the inner query need a value from the current outer row?"

If yes:

```text
correlated subquery
```

Typical wording:

```text
above their department average
customers with an order in their region
employees with a project in their department
products with a review above their own average
```

---

# 102. Subquery vs Join Summary

| Requirement | Natural pattern |
|---|---|
| Compare against one aggregate value | Scalar subquery |
| Membership in another result | `IN` |
| At least one related row | `EXISTS` |
| No related row | `NOT EXISTS` |
| Need columns from both tables | `JOIN` |
| Per-row related aggregate | Correlated subquery or later window function |
| Need all rows from one side | Outer join |

This is a decision guide, not an absolute rule. SQL problems often have multiple valid formulations.

---

# 103. Performance Awareness

Do not assume:

```text
subquery = slow
join = fast
```

or:

```text
EXISTS = always faster
IN = always slower
```

Modern optimizers can transform many equivalent queries.

Performance depends on:

- indexes
- statistics
- cardinality
- selectivity
- DBMS
- query structure
- available join/semijoin strategies
- data distribution

Use execution plans for performance analysis.

---

# 104. Correlated Subquery Performance

Conceptually:

```text
outer rows
× inner lookup
```

can suggest expensive repeated work.

But the optimizer may decorrelate or transform the query.

Therefore evaluate the actual plan rather than relying on a simplistic rule such as:

```text
"correlated subquery always runs once per outer row."
```

That is a useful conceptual model, not a guaranteed physical execution strategy.

---

# 105. Index Awareness

Correlated existence checks can benefit from indexes on the correlated lookup column.

Example:

```sql
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
)
```

An index on:

```text
orders.customer_id
```

may make matching more efficient.

The actual benefit depends on the optimizer and data distribution.

---

# 106. Subqueries and Query Readability

Use subqueries when they make the logical requirement obvious.

Good:

```sql
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
)
```

This clearly communicates:

```text
above company average
```

Do not create deeply nested subqueries merely because SQL allows them.

For complex multi-step logic, consider:

```text
CTEs
derived tables
joins
window functions
```

as appropriate.

---

# 107. Common Interview Traps

### Trap 1

```sql
x = (SELECT many_rows ...)
```

Scalar context cannot accept multiple rows.

### Trap 2

```sql
NOT IN (subquery)
```

with possible NULL.

### Trap 3

Forgetting correlation:

```sql
EXISTS (SELECT 1 FROM projects)
```

instead of:

```sql
EXISTS (
    SELECT 1
    FROM projects p
    WHERE p.department_id = e.department_id
)
```

### Trap 4

Using a JOIN when only existence matters and then needing `DISTINCT`.

### Trap 5

Assuming `EXISTS` counts rows.

### Trap 6

Assuming `IN` duplicates outer rows.

### Trap 7

Assuming NULL behaves like a normal value.

### Trap 8

Assuming a correlated subquery physically executes from scratch once per outer row.

---

# 108. Mastery Checklist

## Scalar Subqueries

- [ ] Understand one-value result shape
- [ ] Use `=`, `>`, `<`, `>=`, `<=`
- [ ] Above overall average
- [ ] Equal to maximum
- [ ] Equal to minimum
- [ ] Understand multi-row error

## Multi-Row Subqueries

- [ ] Understand multiple returned values
- [ ] Use `IN`
- [ ] Use `NOT IN`
- [ ] Understand NULL behavior
- [ ] Understand duplicate values

## Correlated Subqueries

- [ ] Identify outer references
- [ ] Above group average
- [ ] Correlated `EXISTS`
- [ ] Correlated `NOT EXISTS`
- [ ] Understand logical per-row model
- [ ] Distinguish logical from physical execution

## EXISTS

- [ ] Understand existence semantics
- [ ] Use `SELECT 1`
- [ ] Understand duplicates do not multiply outer rows
- [ ] Use for "at least one"
- [ ] Recognize semi-join concept

## NOT EXISTS

- [ ] Use for "none"
- [ ] Recognize anti-join concept
- [ ] Compare with `LEFT JOIN ... IS NULL`
- [ ] Prefer for nullable anti-match situations

## IN / NOT IN

- [ ] Membership
- [ ] Non-membership
- [ ] `NULL` in subquery
- [ ] `NULL` in outer value
- [ ] `NOT IN` trap
- [ ] Know when `NOT EXISTS` is safer

## Interview Reasoning

- [ ] Identify result shape before choosing operator
- [ ] Track NULL semantics
- [ ] Track output grain
- [ ] Distinguish existence from joining
- [ ] Recognize duplicate multiplication
- [ ] Choose the clearest formulation

---

# 109. Final Revision Sheet

```text
SUBQUERY
→ query inside another query

SCALAR SUBQUERY
→ one value

    salary > (
        SELECT AVG(salary)
        FROM employees
    )

MULTI-ROW SUBQUERY
→ multiple values

    x IN (
        SELECT y
        FROM B
    )

CORRELATED SUBQUERY
→ references outer query

    SELECT ...
    FROM A a
    WHERE a.x > (
        SELECT AVG(b.x)
        FROM B b
        WHERE b.group_id = a.group_id
    )

EXISTS
→ at least one matching row

    WHERE EXISTS (
        SELECT 1
        FROM B
        WHERE B.key = A.key
    )

NOT EXISTS
→ no matching row

    WHERE NOT EXISTS (
        SELECT 1
        FROM B
        WHERE B.key = A.key
    )

IN
→ value belongs to returned set

    WHERE x IN (
        SELECT y
        FROM B
    )

NOT IN
→ value does not belong to returned set

    WHERE x NOT IN (
        SELECT y
        FROM B
    )

CRITICAL NOT IN RULE

    NOT IN + NULL
    → can produce UNKNOWN
    → WHERE keeps only TRUE
    → rows can disappear unexpectedly

SAFER ANTI-MATCH

    NOT EXISTS (
        SELECT 1
        FROM B
        WHERE B.key = A.key
    )

EXISTS vs JOIN

    EXISTS
    → asks whether a match exists
    → outer row remains once

    JOIN
    → combines matching rows
    → can multiply output rows

IN vs JOIN

    IN
    → membership
    → does not multiply outer rows merely because
      the subquery contains duplicates

    JOIN
    → matching rows combine
    → can multiply rows

RESULT SHAPE

    ONE VALUE
        → scalar comparison

    MANY VALUES
        → IN / NOT IN

    EXISTENCE
        → EXISTS / NOT EXISTS

    OUTER-ROW DEPENDENCY
        → correlated subquery

NULL

    NULL = NULL
        → UNKNOWN

    NULL IN (...)
        → not TRUE

    NULL NOT IN (...)
        → not TRUE

LOGICAL MODEL

    correlated subquery
    → conceptually evaluated with the current outer row

PHYSICAL REALITY

    optimizer may transform it
    → join
    → semi-join
    → anti-join
    → aggregation
    → other equivalent plan

MOST IMPORTANT INTERVIEW PATTERNS

    Above overall average
        → scalar subquery

    Above department average
        → correlated scalar subquery

    Has at least one order
        → EXISTS

    Has no orders
        → NOT EXISTS

    Belongs to selected groups
        → IN

    Does not belong to selected groups
        → NOT IN, but check NULL
        → often NOT EXISTS is safer

    Maximum with ties
        → = (SELECT MAX(...))
```

---

# 110. Mastery Standard

You have mastered SQL Subqueries when you can:

1. Identify whether a subquery returns one value or multiple rows.
2. Choose scalar comparison vs `IN`/`EXISTS`.
3. Write an above-average scalar subquery.
4. Write an above-group-average correlated subquery.
5. Explain exactly what makes a subquery correlated.
6. Use `EXISTS` for "at least one".
7. Use `NOT EXISTS` for "none".
8. Explain why `EXISTS` does not multiply outer rows.
9. Explain why a normal join can multiply outer rows.
10. Use `IN` for membership.
11. Explain `IN` vs `EXISTS`.
12. Explain `NOT IN` vs `NOT EXISTS`.
13. Identify the `NOT IN + NULL` trap immediately.
14. Explain SQL's TRUE/FALSE/UNKNOWN behavior.
15. Find unmatched records with `NOT EXISTS`.
16. Use scalar subqueries with `HAVING`.
17. Track the grain of nested/derived results.
18. Recognize missing correlation as a major logic bug.
19. Distinguish conceptual correlated execution from physical execution.
20. Choose the clearest formulation among subquery, join, and existence patterns.

The target translation process is:

```text
English requirement
        ↓
What does the inner query represent?
        ↓
ONE VALUE?
    → scalar comparison

SET OF VALUES?
    → IN / NOT IN

DOES A MATCH EXIST?
    → EXISTS

DOES NO MATCH EXIST?
    → NOT EXISTS

DEPENDS ON CURRENT OUTER ROW?
    → correlated subquery

        ↓
Check NULL behavior
        ↓
Check duplicate/multiplication behavior
        ↓
Check output grain
        ↓
Write query
```

The single most important warning to retain:

```text
NOT IN
+
possible NULL
=
STOP AND REASON ABOUT THREE-VALUED LOGIC

For anti-existence problems:

NOT EXISTS
is usually the safer mental default.
```
