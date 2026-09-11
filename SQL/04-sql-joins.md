# 04. SQL Joins

> **Track:** SQL Interview Preparation  
> **Overall repository number:** 27  
> **SQL topic number:** 04  
> **Priority:** **CRITICAL**  
> **Target:** FAANG + top product companies | SDE + ML/AI Engineer  
> **Difficulty:** Easy → Medium → Hard, with Medium dominating

---

# 1. Overview

A **JOIN** combines rows from two or more tables according to a relationship or join condition.

The core join types:

```text
INNER JOIN
LEFT JOIN
RIGHT JOIN
FULL OUTER JOIN
CROSS JOIN
SELF JOIN
```

For interviews, knowing the syntax is not enough. You must be able to predict:

- which rows survive
- which rows disappear
- where `NULL` appears
- how many output rows are produced
- why duplicate-looking rows occur
- how one-to-one, one-to-many, and many-to-many relationships affect cardinality
- how filtering changes an outer join
- how to place conditions correctly in `ON` vs `WHERE`

---

# 2. The Central Mental Model

For every join, ask:

```text
1. What is the left table?
2. What is the right table?
3. What is the join condition?
4. Which rows can match?
5. Can one row match multiple rows?
6. What happens to unmatched rows?
7. Where will NULL appear?
8. Are filters applied before or after the join?
9. Can the join multiply rows?
```

The most important concept is:

> A join does not necessarily preserve one input row as one output row.

One input row can match:

```text
0 rows
1 row
2 rows
10 rows
...
```

and therefore produce a corresponding number of joined rows depending on the join type.

---

# 3. Sample Tables

Use these tables throughout the topic.

## employees

| employee_id | name | department_id | manager_id |
|---:|---|---:|---:|
| 1 | Alice | 10 | 3 |
| 2 | Bob | 10 | 3 |
| 3 | Carol | 20 | NULL |
| 4 | David | 30 | 3 |
| 5 | Eva | NULL | 3 |

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

# 4. What Is a Join?

Basic form:

```sql
SELECT ...
FROM table_a
JOIN table_b
    ON table_a.key = table_b.key;
```

The `ON` condition determines which row combinations match.

Example:

```sql
SELECT
    e.name,
    d.department_name
FROM employees e
JOIN departments d
    ON e.department_id = d.department_id;
```

This connects employees to their departments.

---

# 5. INNER JOIN

`INNER JOIN` returns only rows where the join condition matches.

Syntax:

```sql
SELECT ...
FROM A
INNER JOIN B
    ON A.key = B.key;
```

`JOIN` without a qualifier normally means:

```sql
INNER JOIN
```

in standard/common SQL usage.

---

# 6. INNER JOIN Example

```sql
SELECT
    e.name,
    d.department_name
FROM employees e
INNER JOIN departments d
    ON e.department_id = d.department_id;
```

Matching rows:

```text
Alice → Engineering
Bob   → Engineering
Carol → Sales
David → HR
```

Eva has:

```text
department_id = NULL
```

so she does not match a normal equality condition.

Finance has no employee, so it does not appear.

---

# 7. INNER JOIN Mental Model

Think:

```text
A
∩
B
```

More precisely, it returns row combinations satisfying the join predicate.

Unmatched rows from either side are discarded.

---

# 8. LEFT JOIN

`LEFT JOIN` returns:

1. every row from the left table
2. matching rows from the right table
3. `NULL` values for right-side columns when there is no match

Syntax:

```sql
SELECT ...
FROM A
LEFT JOIN B
    ON A.key = B.key;
```

---

# 9. LEFT JOIN Example

```sql
SELECT
    e.name,
    d.department_name
FROM employees e
LEFT JOIN departments d
    ON e.department_id = d.department_id;
```

Conceptually:

```text
Alice → Engineering
Bob   → Engineering
Carol → Sales
David → HR
Eva   → NULL
```

Eva remains because `employees` is the left table.

---

# 10. LEFT JOIN with Unmatched Right Rows

A left join does **not** preserve unmatched rows from the right table.

Finance has no employee.

Therefore:

```text
Finance
```

does not appear in the employee-driven result.

If you want all departments, make `departments` the left table:

```sql
SELECT
    d.department_name,
    e.name
FROM departments d
LEFT JOIN employees e
    ON d.department_id = e.department_id;
```

Now Finance appears with:

```text
Finance → NULL
```

---

# 11. RIGHT JOIN

`RIGHT JOIN` preserves every row from the right table.

Example:

```sql
SELECT
    e.name,
    d.department_name
FROM employees e
RIGHT JOIN departments d
    ON e.department_id = d.department_id;
```

Conceptually, every department appears.

Finance has no matching employee:

```text
NULL → Finance
```

---

# 12. RIGHT JOIN vs LEFT JOIN

A right join:

```sql
A RIGHT JOIN B
```

can usually be rewritten by reversing the table order:

```sql
B LEFT JOIN A
```

Therefore many developers prefer `LEFT JOIN` for readability and use fewer `RIGHT JOIN`s.

Example:

```sql
SELECT ...
FROM employees e
RIGHT JOIN departments d
    ON e.department_id = d.department_id;
```

Equivalent orientation:

```sql
SELECT ...
FROM departments d
LEFT JOIN employees e
    ON e.department_id = d.department_id;
```

---

# 13. FULL OUTER JOIN

`FULL OUTER JOIN` preserves:

- matching rows
- unmatched left rows
- unmatched right rows

Conceptually:

```text
LEFT JOIN
+
unmatched right rows
```

Syntax:

```sql
SELECT ...
FROM A
FULL OUTER JOIN B
    ON A.key = B.key;
```

---

# 14. FULL OUTER JOIN Example

Using employees and departments:

```sql
SELECT
    e.name,
    d.department_name
FROM employees e
FULL OUTER JOIN departments d
    ON e.department_id = d.department_id;
```

Conceptually:

```text
Alice → Engineering
Bob   → Engineering
Carol → Sales
David → HR
Eva   → NULL
NULL  → Finance
```

Both unmatched sides survive.

---

# 15. FULL OUTER JOIN Support

Not every DBMS supports `FULL OUTER JOIN` directly.

For example, MySQL historically does not provide native `FULL OUTER JOIN` syntax.

A conceptual emulation can be built using:

```text
LEFT JOIN
UNION
RIGHT JOIN
```

but the exact implementation and duplicate handling require care.

For interviews, know both:

```text
FULL OUTER JOIN
```

and the idea of constructing equivalent results when native support is absent.

---

# 16. CROSS JOIN

`CROSS JOIN` produces the Cartesian product.

Every row of A is paired with every row of B.

Syntax:

```sql
SELECT ...
FROM A
CROSS JOIN B;
```

If:

```text
A has m rows
B has n rows
```

then the result has:

```text
m × n
```

rows.

---

# 17. CROSS JOIN Example

If:

```text
employees = 5 rows
projects = 4 rows
```

then:

```sql
SELECT *
FROM employees
CROSS JOIN projects;
```

produces:

```text
5 × 4 = 20
```

row combinations.

There is no join condition.

---

# 18. CROSS JOIN Use Cases

Although often dangerous unintentionally, Cartesian products can be intentional.

Examples:

- generating all combinations
- creating a calendar × category matrix
- comparing every product with every scenario
- creating test combinations

Example:

```sql
SELECT
    s.size,
    c.color
FROM sizes s
CROSS JOIN colors c;
```

This generates every size-color combination.

---

# 19. SELF JOIN

A self join joins a table to itself.

It is useful when rows in the same table are related.

Example:

```text
employee
    employee_id
    name
    manager_id
```

The `manager_id` points back to another employee.

---

# 20. Self Join Example

```sql
SELECT
    e.name AS employee,
    m.name AS manager
FROM employees e
LEFT JOIN employees m
    ON e.manager_id = m.employee_id;
```

Here:

```text
employees e
```

represents the employee.

```text
employees m
```

represents the manager.

They are two aliases of the same table.

---

# 21. Why Aliases Are Essential in Self Joins

Without aliases:

```sql
FROM employees
JOIN employees
```

the columns become ambiguous.

Use:

```sql
employees e
employees m
```

so you can write:

```sql
e.name
m.name
```

and:

```sql
e.manager_id = m.employee_id
```

---

# 22. Join Comparison

| Join | Preserves unmatched left? | Preserves unmatched right? |
|---|---:|---:|
| `INNER JOIN` | No | No |
| `LEFT JOIN` | Yes | No |
| `RIGHT JOIN` | No | Yes |
| `FULL OUTER JOIN` | Yes | Yes |
| `CROSS JOIN` | Cartesian product | Cartesian product |
| `SELF JOIN` | Depends on join type | Depends on join type |

Important:

> `SELF JOIN` is not a separate matching semantics like inner/outer join. It means joining a table to itself. The actual join can be inner, left, etc.

---

# 23. Join Conditions

The join condition is usually written in `ON`.

Example:

```sql
ON e.department_id = d.department_id
```

This means rows are combined when:

```text
e.department_id
=
d.department_id
```

---

# 24. Equi Join

A join using equality:

```sql
ON A.key = B.key
```

is commonly called an **equi-join**.

Example:

```sql
SELECT *
FROM employees e
JOIN departments d
    ON e.department_id = d.department_id;
```

---

# 25. Non-Equi Join

A join condition does not have to use `=`.

Example:

```sql
SELECT
    e.name,
    g.grade
FROM employees e
JOIN salary_grades g
    ON e.salary BETWEEN g.min_salary AND g.max_salary;
```

This is a range/non-equality join.

The exact supported syntax depends on the schema and DBMS, but the conceptual point is:

```text
JOIN condition ≠ necessarily equality
```

---

# 26. Multiple Join Conditions

You can combine conditions.

```sql
SELECT *
FROM A
JOIN B
    ON A.id = B.id
   AND A.region = B.region;
```

Both conditions must be satisfied.

Equivalent conceptual predicate:

```text
(A.id = B.id)
AND
(A.region = B.region)
```

---

# 27. OR in Join Conditions

You can also use `OR`.

```sql
ON A.email = B.email
OR A.phone = B.phone
```

But be careful:

- it can produce multiple matches
- it can be expensive
- reasoning about duplicates becomes harder

Use the simplest correct join condition.

---

# 28. One-to-One Relationship

In a one-to-one relationship, each row in A maps to at most one row in B, and vice versa.

Example:

```text
person
person_id

passport
passport_id
person_id UNIQUE
```

A join:

```sql
SELECT *
FROM person p
JOIN passport pp
    ON p.person_id = pp.person_id;
```

can produce at most one passport row per person if the relationship is truly enforced as one-to-one.

---

# 29. One-to-Many Relationship

Very common.

Example:

```text
department
    1
    ↓
    many
employees
```

One department can have many employees.

```sql
SELECT *
FROM departments d
JOIN employees e
    ON d.department_id = e.department_id;
```

A department with three employees produces three joined rows.

---

# 30. One-to-Many Cardinality

Suppose:

```text
Engineering
    ↓
Alice
Bob
Grace
```

Then:

```sql
departments
JOIN employees
```

produces:

```text
Engineering | Alice
Engineering | Bob
Engineering | Grace
```

The department row is repeated.

This is not necessarily a duplicate error.

It represents three valid relationships.

---

# 31. Many-to-Many Relationship

Example:

```text
students
courses
```

A student can take many courses.

A course can have many students.

Use a junction table:

```text
student_courses
    student_id
    course_id
```

Schema:

```text
students
    ↓
student_courses
    ↓
courses
```

---

# 32. Many-to-Many Join

```sql
SELECT
    s.student_name,
    c.course_name
FROM students s
JOIN student_courses sc
    ON s.student_id = sc.student_id
JOIN courses c
    ON sc.course_id = c.course_id;
```

One student can appear many times:

```text
Alice → DBMS
Alice → OS
Alice → Algorithms
```

That is expected.

---

# 33. Duplicate Rows After Joins

One of the most important interview topics.

Suppose:

```text
A
id
1
```

and:

```text
B
a_id
1
1
1
```

Then:

```sql
SELECT *
FROM A
JOIN B
    ON A.id = B.a_id;
```

produces:

```text
1 | row1
1 | row2
1 | row3
```

One A row matched three B rows.

Therefore:

```text
1 input row
→ 3 output rows
```

---

# 34. Join Multiplication

Suppose one customer has:

```text
3 orders
```

and each order has:

```text
4 items
```

Joining customers → orders → items can produce multiple rows for that customer.

At the order-item level:

```text
3 orders × their item counts
```

The result is at a finer grain than the customer table.

Always identify the **grain** of your result.

---

# 35. Grain

"Grain" means:

> What does one output row represent?

Examples:

```text
one row = one employee
one row = one order
one row = one order item
one row = one customer
one row = one customer-order pair
```

Before writing a multi-table query, identify the intended grain.

This prevents many join bugs.

---

# 36. Duplicate-Looking Rows vs Actual Duplicates

Suppose:

```text
Alice | Engineering
Alice | Engineering
```

appears twice.

This may be because Alice has two matching related records.

The rows are not necessarily duplicates in the relational result.

Do not automatically solve the problem with:

```sql
DISTINCT
```

First determine why the multiplication occurred.

---

# 37. When DISTINCT Is Appropriate

Use:

```sql
SELECT DISTINCT ...
```

when the requirement genuinely asks for unique projected values.

Example:

> List departments that have at least one employee working on a project.

If joins produce multiple projects per department:

```sql
SELECT DISTINCT d.department_name
...
```

may be appropriate.

But `DISTINCT` should not be used as a blind fix for an incorrect join.

---

# 38. Join Filtering: The Critical Difference

Consider:

```sql
SELECT ...
FROM employees e
LEFT JOIN departments d
    ON e.department_id = d.department_id
WHERE d.department_name = 'Engineering';
```

This can eliminate rows where the department did not match because:

```text
d.department_name = NULL
```

and:

```text
NULL = 'Engineering'
```

is UNKNOWN.

Therefore the outer join can behave like an inner join for this condition.

---

# 39. Filter in ON vs WHERE

Compare:

### Version A

```sql
SELECT *
FROM employees e
LEFT JOIN departments d
    ON e.department_id = d.department_id
WHERE d.department_name = 'Engineering';
```

### Version B

```sql
SELECT *
FROM employees e
LEFT JOIN departments d
    ON e.department_id = d.department_id
   AND d.department_name = 'Engineering';
```

These are generally **not equivalent**.

---

# 40. Why ON and WHERE Differ for LEFT JOIN

### Version A

```sql
LEFT JOIN ...
WHERE d.department_name = 'Engineering'
```

First forms the left join.

Then `WHERE` removes rows whose right side does not satisfy the condition.

Unmatched rows have:

```text
d.department_name = NULL
```

and are removed.

### Version B

```sql
LEFT JOIN ...
ON ...
AND d.department_name = 'Engineering'
```

The condition controls which right-side rows match.

The left row can still survive with NULL right-side values.

This distinction is essential.

---

# 41. Concrete Example

Employees:

```text
Alice → Engineering
Bob   → Engineering
Carol → Sales
Eva   → NULL
```

Query:

```sql
SELECT e.name, d.department_name
FROM employees e
LEFT JOIN departments d
    ON e.department_id = d.department_id
WHERE d.department_name = 'Engineering';
```

Result:

```text
Alice → Engineering
Bob   → Engineering
```

Carol and Eva are removed.

---

# 42. Same Requirement in ON

```sql
SELECT e.name, d.department_name
FROM employees e
LEFT JOIN departments d
    ON e.department_id = d.department_id
   AND d.department_name = 'Engineering';
```

Result conceptually:

```text
Alice → Engineering
Bob   → Engineering
Carol → NULL
David → NULL
Eva   → NULL
```

All employees survive because employees is the left table.

Only Engineering departments are eligible to match.

---

# 43. Filtering the Left Table

If the filter is on the preserved left table:

```sql
SELECT ...
FROM employees e
LEFT JOIN departments d
    ON e.department_id = d.department_id
WHERE e.name LIKE 'A%';
```

the employee filter removes left-side rows.

That is usually straightforward.

The major outer-join trap is filtering columns from the nullable side in `WHERE`.

---

# 44. Filtering Before vs After Joins

You can conceptually filter source rows before joining:

```sql
SELECT ...
FROM (
    SELECT *
    FROM employees
    WHERE salary > 80000
) e
JOIN departments d
    ON e.department_id = d.department_id;
```

or write:

```sql
SELECT ...
FROM employees e
JOIN departments d
    ON e.department_id = d.department_id
WHERE e.salary > 80000;
```

For an inner join, the optimizer may transform equivalent predicates, but semantics and null-preservation must be considered carefully for outer joins.

---

# 45. INNER JOIN Filtering

For an inner join, a predicate on either table can often be placed in `WHERE` or incorporated into the `ON` condition without changing the final matching rows.

Example:

```sql
FROM employees e
JOIN departments d
    ON e.department_id = d.department_id
WHERE d.department_name = 'Engineering'
```

versus:

```sql
FROM employees e
JOIN departments d
    ON e.department_id = d.department_id
   AND d.department_name = 'Engineering'
```

For this inner-join case, they are logically equivalent.

This equivalence should **not** be blindly transferred to outer joins.

---

# 46. LEFT JOIN Filtering Rule

A useful rule:

```text
LEFT JOIN
+
condition on right table in ON
→ controls matching

LEFT JOIN
+
condition on right table in WHERE
→ can eliminate NULL-extended rows
```

Therefore:

```text
ON → match condition
WHERE → final row filter
```

with special importance for outer joins.

---

# 47. NULL in Join Conditions

Consider:

```sql
ON e.department_id = d.department_id
```

If:

```text
e.department_id = NULL
```

then the comparison is:

```text
NULL = d.department_id
```

which is UNKNOWN.

It does not become TRUE simply because the other value is also NULL.

Therefore ordinary equality joins do not match NULL to NULL.

---

# 48. NULL-Safe Matching

If the requirement is:

> Treat two NULL keys as matching

then ordinary:

```sql
A.key = B.key
```

is not sufficient.

Some DBMSs provide NULL-safe comparison operators, while portable SQL can express the desired logic explicitly, for example:

```sql
ON A.key = B.key
OR (A.key IS NULL AND B.key IS NULL)
```

Whether a specialized operator exists depends on the DBMS.

---

# 49. NULLs in LEFT JOIN

If the right side has no match:

```text
right_table.column
=
NULL
```

in the output row.

This is called NULL-extension.

Example:

```text
employees
LEFT JOIN departments
```

for an employee without a department:

```text
employee columns → actual values
department columns → NULL
```

---

# 50. Finding Unmatched Rows with LEFT JOIN

Classic anti-join pattern:

> Find employees who do not have a matching department.

```sql
SELECT e.*
FROM employees e
LEFT JOIN departments d
    ON e.department_id = d.department_id
WHERE d.department_id IS NULL;
```

This is a very important interview pattern.

---

# 51. Why IS NULL Is Required

Do not write:

```sql
WHERE d.department_id = NULL
```

because:

```text
NULL = NULL
```

is not TRUE.

Use:

```sql
WHERE d.department_id IS NULL
```

---

# 52. LEFT JOIN as an Anti-Join

Pattern:

```sql
SELECT A.*
FROM A
LEFT JOIN B
    ON A.key = B.key
WHERE B.key IS NULL;
```

Meaning:

```text
rows in A with no matching row in B
```

This is commonly called an anti-join pattern.

Another way to express the same logical requirement is often:

```sql
WHERE NOT EXISTS (...)
```

which will be covered with subqueries.

---

# 53. Finding Matched Rows

An inner join naturally returns matched rows:

```sql
SELECT e.*
FROM employees e
JOIN departments d
    ON e.department_id = d.department_id;
```

---

# 54. Finding All Departments Including Empty Ones

Start from departments:

```sql
SELECT
    d.department_name,
    COUNT(e.employee_id) AS employee_count
FROM departments d
LEFT JOIN employees e
    ON e.department_id = d.department_id
GROUP BY d.department_name;
```

Finance appears with:

```text
employee_count = 0
```

This is a classic join + aggregation pattern.

---

# 55. Why COUNT(*) Can Be Wrong Here

Consider:

```sql
SELECT
    d.department_name,
    COUNT(*) AS employee_count
FROM departments d
LEFT JOIN employees e
    ON e.department_id = d.department_id
GROUP BY d.department_name;
```

For a department with no employees, the left join still creates a NULL-extended joined row.

Therefore:

```text
COUNT(*) = 1
```

may occur.

Instead use:

```sql
COUNT(e.employee_id)
```

because NULL employee IDs are ignored.

Result:

```text
Finance → 0
```

---

# 56. One-to-Many + Aggregation

Suppose:

```text
department
    ↓
employees
```

Question:

> Count employees per department, including departments with zero employees.

Correct pattern:

```sql
SELECT
    d.department_name,
    COUNT(e.employee_id) AS employee_count
FROM departments d
LEFT JOIN employees e
    ON e.department_id = d.department_id
GROUP BY d.department_name;
```

This is a highly reusable interview pattern.

---

# 57. Many-to-Many + Aggregation

Suppose:

```text
students
student_courses
courses
```

Question:

> Count courses per student.

```sql
SELECT
    s.student_id,
    COUNT(sc.course_id) AS course_count
FROM students s
LEFT JOIN student_courses sc
    ON s.student_id = sc.student_id
GROUP BY s.student_id;
```

If you also join courses:

```sql
SELECT
    s.student_id,
    COUNT(c.course_id) AS course_count
FROM students s
LEFT JOIN student_courses sc
    ON s.student_id = sc.student_id
LEFT JOIN courses c
    ON sc.course_id = c.course_id
GROUP BY s.student_id;
```

The relationship's cardinality must be understood before aggregating.

---

# 58. Many-to-Many Duplicate Trap

Suppose a student has:

```text
3 enrollments
```

and each course has several related records.

A subsequent join can multiply rows.

For example:

```text
student
→ enrollment
→ course
→ course_instructor
```

can produce multiple rows per enrollment.

If the question asks:

> Number of courses

you may need:

```sql
COUNT(DISTINCT c.course_id)
```

rather than:

```sql
COUNT(*)
```

depending on the intended grain.

---

# 59. Join Cardinality

Let:

```text
A row matches k rows in B
```

Then that A row can contribute:

```text
k
```

joined output rows for an inner/left match portion.

If:

```text
k = 0
```

then:

- `INNER JOIN` loses the A row
- `LEFT JOIN` retains it with NULL right columns

This is the fundamental cardinality behavior.

---

# 60. Primary Key / Foreign Key and Joins

Typical relationship:

```text
departments.department_id
    ↓
employees.department_id
```

where:

```text
departments.department_id
```

is a primary/unique key and:

```text
employees.department_id
```

is a foreign key.

Then each employee matches at most one department row if the referenced key is unique.

But one department can match many employees.

Therefore:

```text
department → employee
```

is:

```text
one-to-many
```

---

# 61. Join Direction and Result Grain

Consider:

```sql
FROM departments d
LEFT JOIN employees e
    ON d.department_id = e.department_id
```

The natural result grain becomes approximately:

```text
one row per department-employee match
```

not:

```text
one row per department
```

If you want one row per department, aggregation may be required:

```sql
GROUP BY d.department_id
```

---

# 62. Joining Three Tables

Example:

```sql
SELECT
    e.name,
    d.department_name,
    p.project_name
FROM employees e
JOIN departments d
    ON e.department_id = d.department_id
JOIN projects p
    ON d.department_id = p.department_id;
```

A department with:

```text
3 employees
2 projects
```

can produce:

```text
3 × 2 = 6
```

employee-project combinations.

This is not necessarily an error.

It represents the join's resulting grain.

---

# 63. The Cartesian Multiplication Trap

Suppose:

```text
Customer A
    2 orders

Customer A
    3 support tickets
```

If you independently join both child tables:

```sql
customers
LEFT JOIN orders
LEFT JOIN tickets
```

you can get:

```text
2 × 3 = 6
```

rows for Customer A.

If you then:

```sql
COUNT(orders.id)
```

you may incorrectly count:

```text
6
```

instead of:

```text
2
```

This is a major SQL interview trap.

Possible solutions include:

- pre-aggregate each child table
- use `COUNT(DISTINCT ...)`
- use separate correlated/subqueries/CTEs
- restructure the query around the desired grain

---

# 64. Pre-Aggregation Pattern

Instead of joining raw child rows:

```sql
SELECT
    c.customer_id,
    o.order_count,
    t.ticket_count
FROM customers c
LEFT JOIN (
    SELECT customer_id, COUNT(*) AS order_count
    FROM orders
    GROUP BY customer_id
) o
    ON c.customer_id = o.customer_id
LEFT JOIN (
    SELECT customer_id, COUNT(*) AS ticket_count
    FROM tickets
    GROUP BY customer_id
) t
    ON c.customer_id = t.customer_id;
```

Each derived table is first reduced to:

```text
one row per customer
```

This prevents cross-multiplication between orders and tickets.

CTEs can make the same idea easier to read.

---

# 65. Join Conditions and Business Keys

Do not assume a join should use only one column.

If uniqueness is defined by:

```text
(company_id, employee_id)
```

then joining only on:

```sql
employee_id
```

may create incorrect matches.

Use the full relationship:

```sql
ON a.company_id = b.company_id
AND a.employee_id = b.employee_id
```

---

# 66. Composite Join Keys

Example:

```sql
SELECT *
FROM orders o
JOIN shipments s
    ON o.order_id = s.order_id
   AND o.customer_id = s.customer_id;
```

A composite relationship requires all necessary key components.

Missing a key component can cause:

```text
too many matches
→ duplicate rows
→ incorrect aggregates
```

---

# 67. Join on Non-Key Columns

You can join on non-key columns:

```sql
ON e.email = u.email
```

But if `email` is not unique in either table, one row can match multiple rows.

This is a common source of accidental multiplication.

Before joining, ask:

```text
Is this join column unique?
```

---

# 68. Foreign Keys Do Not Automatically Perform Joins

A foreign key expresses a database relationship/constraint.

SQL still requires an explicit join:

```sql
JOIN departments d
    ON e.department_id = d.department_id
```

The database does not automatically combine tables just because a foreign key exists.

---

# 69. Natural Join

Some SQL systems support:

```sql
NATURAL JOIN
```

It automatically joins using same-named columns.

For interview preparation, understand why explicit joins are generally safer:

```sql
ON A.id = B.id
```

is clear.

`NATURAL JOIN` can change behavior when schemas gain additional same-named columns.

It is not one of the core joins required for this topic.

---

# 70. USING Clause

Some SQL dialects allow:

```sql
SELECT *
FROM employees e
JOIN departments d
USING (department_id);
```

`USING` is shorthand when both tables contain the same join-column name.

It is dialect-supported/common SQL syntax, but:

```sql
ON e.department_id = d.department_id
```

is the most universal mental model.

---

# 71. Filtering Before a LEFT JOIN

Suppose the requirement is:

> Return all departments and count only employees with salary > 80000.

A safe pattern is:

```sql
SELECT
    d.department_name,
    COUNT(e.employee_id) AS high_paid_count
FROM departments d
LEFT JOIN employees e
    ON d.department_id = e.department_id
   AND e.salary > 80000
GROUP BY d.department_name;
```

The salary condition is in `ON` because departments must remain even when no high-paid employee matches.

---

# 72. The Wrong Outer-Join Filter

This query:

```sql
SELECT
    d.department_name,
    COUNT(e.employee_id)
FROM departments d
LEFT JOIN employees e
    ON d.department_id = e.department_id
WHERE e.salary > 80000
GROUP BY d.department_name;
```

removes departments whose employees do not satisfy the condition.

Departments with no matching employees have:

```text
e.salary = NULL
```

and fail the `WHERE`.

Thus the intended "include every department" semantics are lost.

---

# 73. Inner Join + WHERE

For inner joins, a condition can often be written in either place:

```sql
FROM A
JOIN B
    ON A.id = B.id
WHERE B.status = 'ACTIVE'
```

or:

```sql
FROM A
JOIN B
    ON A.id = B.id
   AND B.status = 'ACTIVE'
```

The result is generally equivalent for an inner join.

Still, keep the join relationship in `ON` and row filters in `WHERE` when that makes the query clearer.

---

# 74. Outer Join Rule to Memorize

For:

```sql
A
LEFT JOIN
B
```

remember:

```text
A = preserved side
B = nullable side
```

A condition on B in:

```sql
ON
```

controls matching.

A condition on B in:

```sql
WHERE
```

can remove the NULL-extended rows.

---

# 75. LEFT JOIN vs INNER JOIN

Requirement:

> Return employees even if they have no department.

Use:

```sql
LEFT JOIN
```

Requirement:

> Return only employees with a matching department.

Use:

```sql
INNER JOIN
```

---

# 76. RIGHT JOIN vs LEFT JOIN

Requirement:

> Return every department, even if it has no employee.

Either:

```sql
departments d
LEFT JOIN employees e
```

or:

```sql
employees e
RIGHT JOIN departments d
```

Most teams prefer the first because the preserved table is visually on the left.

---

# 77. FULL OUTER JOIN Use Case

Requirement:

> Compare two datasets and show matches plus records missing from either side.

Use:

```sql
FULL OUTER JOIN
```

Typical examples:

- old vs new records
- source system vs target system
- two inventories
- reconciliation
- data-quality comparison

---

# 78. SELF JOIN Use Cases

Common patterns:

### Employee-manager hierarchy

```sql
employees e
LEFT JOIN employees m
```

### Find pairs

```sql
employees a
JOIN employees b
```

with a condition such as:

```sql
a.department_id = b.department_id
AND a.employee_id < b.employee_id
```

The `<` prevents pairing each pair twice and avoids self-pairing.

---

# 79. Finding Employee Pairs

Question:

> Find unique pairs of employees in the same department.

```sql
SELECT
    e1.name AS employee_1,
    e2.name AS employee_2
FROM employees e1
JOIN employees e2
    ON e1.department_id = e2.department_id
   AND e1.employee_id < e2.employee_id;
```

Why:

```sql
e1.employee_id < e2.employee_id
```

?

Without it:

```text
Alice, Bob
Bob, Alice
```

would both appear.

It also prevents:

```text
Alice, Alice
```

---

# 80. Self Join and NULL

Manager lookup:

```sql
SELECT
    e.name,
    m.name AS manager
FROM employees e
LEFT JOIN employees m
    ON e.manager_id = m.employee_id;
```

The top-level manager:

```text
manager_id = NULL
```

remains because this is a `LEFT JOIN`.

The manager column becomes:

```text
NULL
```

---

# 81. JOIN + GROUP BY

Question:

> Count employees in every department.

```sql
SELECT
    d.department_name,
    COUNT(e.employee_id) AS employee_count
FROM departments d
LEFT JOIN employees e
    ON d.department_id = e.department_id
GROUP BY d.department_name;
```

Important:

```text
LEFT JOIN
→ preserves departments
COUNT(e.employee_id)
→ counts actual employees
```

---

# 82. JOIN + HAVING

Question:

> Find departments with at least three employees.

```sql
SELECT
    d.department_name,
    COUNT(e.employee_id) AS employee_count
FROM departments d
LEFT JOIN employees e
    ON d.department_id = e.department_id
GROUP BY d.department_name
HAVING COUNT(e.employee_id) >= 3;
```

The `HAVING` condition is evaluated at group level.

---

# 83. JOIN + DISTINCT

Question:

> List departments that have at least one project.

```sql
SELECT DISTINCT d.department_name
FROM departments d
JOIN projects p
    ON d.department_id = p.department_id;
```

A department with multiple projects produces multiple joined rows, so `DISTINCT` reduces the final projected result to unique departments.

Another solution is often `EXISTS`, which avoids generating all matches when only existence matters.

---

# 84. JOIN + ORDER BY

```sql
SELECT
    d.department_name,
    COUNT(e.employee_id) AS employee_count
FROM departments d
LEFT JOIN employees e
    ON d.department_id = e.department_id
GROUP BY d.department_name
ORDER BY employee_count DESC;
```

This returns departments ordered by employee count.

---

# 85. JOIN + LIMIT

Top department by employee count:

```sql
SELECT
    d.department_name,
    COUNT(e.employee_id) AS employee_count
FROM departments d
LEFT JOIN employees e
    ON d.department_id = e.department_id
GROUP BY d.department_name
ORDER BY employee_count DESC
LIMIT 1;
```

If ties matter, `LIMIT 1` may be insufficient.

Later ranking/window techniques can return all tied winners.

---

# 86. Join Order

For inner joins, relational algebra permits many equivalent reorderings.

For outer joins, order affects which rows are preserved.

Therefore:

```text
Do not casually reorder LEFT/RIGHT/FULL joins.
```

Always identify the preserved side.

---

# 87. Associativity Caveat

Inner joins have useful algebraic properties that let optimizers reorder them under appropriate conditions.

Outer joins are more constrained because null-preservation matters.

Interview rule:

```text
INNER JOIN → generally flexible
OUTER JOIN → preserve side matters
```

---

# 88. Join Predicate vs Filter Predicate

A useful organization:

### Join predicate

Defines how rows relate:

```sql
ON e.department_id = d.department_id
```

### Filter predicate

Defines which rows/groups you want:

```sql
WHERE e.salary > 80000
```

But for outer joins, moving predicates between `ON` and `WHERE` can change semantics.

Therefore classify the predicate and consider null-preservation.

---

# 89. Accidental CROSS JOIN

This is dangerous:

```sql
SELECT *
FROM employees e, departments d;
```

Without an appropriate relationship predicate, this can create a Cartesian product.

If there are:

```text
1000 employees
50 departments
```

the result can contain:

```text
50,000
```

row combinations.

Prefer explicit:

```sql
JOIN ... ON ...
```

or explicit:

```sql
CROSS JOIN
```

when a Cartesian product is intentional.

---

# 90. Legacy Comma Join

Old-style syntax:

```sql
SELECT *
FROM employees e, departments d
WHERE e.department_id = d.department_id;
```

is conceptually an inner join.

Modern explicit form:

```sql
SELECT *
FROM employees e
JOIN departments d
    ON e.department_id = d.department_id;
```

is clearer and makes outer joins possible without mixing join logic with filtering.

---

# 91. NULL and WHERE After LEFT JOIN

Suppose:

```sql
SELECT *
FROM A
LEFT JOIN B
    ON A.id = B.a_id
WHERE B.status = 'ACTIVE';
```

For unmatched B rows:

```text
B.status = NULL
```

Then:

```text
NULL = 'ACTIVE'
→ UNKNOWN
```

`WHERE` keeps only TRUE conditions.

Therefore unmatched rows are removed.

This is why the query may effectively behave like an inner join for the filtered B rows.

---

# 92. WHERE Uses Three-Valued Logic

SQL conditions can evaluate to:

```text
TRUE
FALSE
UNKNOWN
```

`WHERE` retains only:

```text
TRUE
```

Therefore:

```sql
WHERE B.status = 'ACTIVE'
```

does not retain:

```text
B.status = NULL
```

This connects joins directly to SQL NULL semantics.

---

# 93. Join + NULL-Safe Requirement

If a data-cleaning problem requires matching nullable attributes, explicitly decide:

```text
Should NULL match NULL?
```

If yes, use DBMS-specific NULL-safe equality or explicit logic.

If no, ordinary:

```sql
A.x = B.x
```

is appropriate.

Never assume NULL behaves like an ordinary value.

---

# 94. Referential Integrity and Missing Matches

Even if a foreign key is declared, real systems can contain:

- nullable foreign keys
- historical records
- soft-deleted parents
- imperfect imported data
- different datasets being reconciled

Therefore a join query must still be reasoned about in terms of matching and unmatched rows rather than assuming every foreign key is non-NULL and matched.

---

# 95. Join Cardinality Checklist

Before joining:

```text
A key uniqueness:
    unique / non-unique?

B key uniqueness:
    unique / non-unique?

Relationship:
    1:1
    1:N
    N:1
    N:M?

Expected output grain:
    what does one row represent?

Potential multiplication:
    yes / no?
```

This should become automatic.

---

# 96. GATE / CS Theory

## 96.1 Inner Join

Returns tuples satisfying the join predicate.

Unmatched tuples are removed.

---

## 96.2 Left Outer Join

Preserves all tuples from the left relation.

For unmatched right-side tuples, right attributes are NULL-extended.

---

## 96.3 Right Outer Join

Preserves all tuples from the right relation.

Unmatched left-side attributes are NULL-extended.

---

## 96.4 Full Outer Join

Preserves tuples from both relations.

Unmatched attributes on either side are NULL-extended.

---

## 96.5 Cross Join

Produces Cartesian product.

For relations with cardinalities:

```text
|R| = m
|S| = n
```

the Cartesian product has:

```text
m × n
```

tuples.

---

## 96.6 Self Join

A relation is joined with itself using aliases.

It is useful for hierarchical and pairwise relationships.

---

## 96.7 One-to-Many

One row in the parent can match many rows in the child.

A join can therefore repeat parent attributes across multiple output rows.

---

## 96.8 Many-to-Many

Usually represented using an associative/junction relation.

Example:

```text
Student
   ↓
Enrollment
   ↓
Course
```

---

## 96.9 Join Cardinality

If a row matches multiple rows on the other side, multiple output rows can result.

Therefore joins can increase result cardinality.

---

## 96.10 Outer Join + WHERE

A predicate on the nullable side in `WHERE` can eliminate NULL-extended rows.

This can change an intended outer join into an effectively inner-filtered result.

---

# 97. GATE-Style Solved Questions

> These are **GATE-style practice questions**, not claims about exact official GATE wording or years.

## Question 1 — INNER vs LEFT JOIN

Tables:

```text
A
id
1
2
3

B
id
1
3
4
```

Consider:

```sql
SELECT A.id, B.id
FROM A
LEFT JOIN B
    ON A.id = B.id;
```

What rows are produced?

### Solution

All A rows survive:

```text
A.id = 1 → B.id = 1
A.id = 2 → B.id = NULL
A.id = 3 → B.id = 3
```

Result:

```text
1 | 1
2 | NULL
3 | 3
```

`B.id = 4` does not appear because B is the right/non-preserved side.

---

# 98. Question 2 — CROSS JOIN Cardinality

Relation A has:

```text
5 rows
```

Relation B has:

```text
7 rows
```

How many rows does:

```sql
SELECT *
FROM A
CROSS JOIN B;
```

produce?

### Solution

Cartesian product:

```text
5 × 7 = 35
```

Answer:

```text
35 rows
```

---

# 99. Question 3 — One-to-Many

Table A contains one row with:

```text
id = 10
```

Table B contains four rows with:

```text
a_id = 10
```

How many joined rows can this A row produce under:

```sql
A
JOIN
B
ON A.id = B.a_id
```

?

### Solution

The A row matches four B rows.

Therefore it contributes:

```text
4 output rows
```

This is the core one-to-many multiplication behavior.

---

# 100. Question 4 — LEFT JOIN + WHERE

Tables:

```text
A
id
1
2

B
a_id | status
1    | ACTIVE
```

Query:

```sql
SELECT A.id, B.status
FROM A
LEFT JOIN B
    ON A.id = B.a_id
WHERE B.status = 'ACTIVE';
```

What is the result?

### Solution

The left join initially produces:

```text
1 | ACTIVE
2 | NULL
```

Then:

```sql
WHERE B.status = 'ACTIVE'
```

removes:

```text
2 | NULL
```

because:

```text
NULL = 'ACTIVE'
→ UNKNOWN
```

Final result:

```text
1 | ACTIVE
```

The unmatched A row is eliminated.

---

# 101. Question 5 — ON vs WHERE

Using the same tables, compare:

```sql
SELECT A.id, B.status
FROM A
LEFT JOIN B
    ON A.id = B.a_id
   AND B.status = 'ACTIVE';
```

### Solution

The left row with:

```text
A.id = 2
```

still survives.

Result:

```text
1 | ACTIVE
2 | NULL
```

The condition in `ON` controls which B rows match; it does not remove the preserved A row.

---

# 102. Question 6 — Duplicate Multiplication

A customer has:

```text
2 orders
```

and:

```text
3 support tickets
```

A query joins:

```text
customer
→ orders
→ support_tickets
```

with both relationships using `customer_id`.

How many customer-level joined combinations can result for that customer?

### Solution

The two independent child sets can combine:

```text
2 × 3 = 6
```

Therefore the customer can produce:

```text
6 joined rows
```

This can cause incorrect counts if aggregates are applied without controlling the grain.

---

# 103. Interview Practice — Easy

## Q1

Return employees and their departments.

```sql
SELECT e.name, d.department_name
FROM employees e
JOIN departments d
    ON e.department_id = d.department_id;
```

---

## Q2

Return all employees, including those without departments.

```sql
SELECT e.name, d.department_name
FROM employees e
LEFT JOIN departments d
    ON e.department_id = d.department_id;
```

---

## Q3

Return all departments, including those without employees.

```sql
SELECT d.department_name, e.name
FROM departments d
LEFT JOIN employees e
    ON d.department_id = e.department_id;
```

---

## Q4

Find employees without a matching department.

```sql
SELECT e.*
FROM employees e
LEFT JOIN departments d
    ON e.department_id = d.department_id
WHERE d.department_id IS NULL;
```

---

# 104. Interview Practice — Medium

## Q5

Count employees per department, including empty departments.

```sql
SELECT
    d.department_name,
    COUNT(e.employee_id) AS employee_count
FROM departments d
LEFT JOIN employees e
    ON d.department_id = e.department_id
GROUP BY d.department_name;
```

---

## Q6

Find departments with at least three employees.

```sql
SELECT
    d.department_name,
    COUNT(e.employee_id) AS employee_count
FROM departments d
LEFT JOIN employees e
    ON d.department_id = e.department_id
GROUP BY d.department_name
HAVING COUNT(e.employee_id) >= 3;
```

---

## Q7

Show each employee and their manager.

```sql
SELECT
    e.name AS employee,
    m.name AS manager
FROM employees e
LEFT JOIN employees m
    ON e.manager_id = m.employee_id;
```

---

## Q8

Find unique employee pairs in the same department.

```sql
SELECT
    e1.name AS employee_1,
    e2.name AS employee_2
FROM employees e1
JOIN employees e2
    ON e1.department_id = e2.department_id
   AND e1.employee_id < e2.employee_id;
```

---

# 105. Interview Practice — Harder

## Q9

Suppose a customer can have many orders and many payments. Return one row per customer with the number of orders and number of payments without cross-multiplying the two child tables.

A robust approach is to pre-aggregate:

```sql
WITH order_counts AS (
    SELECT customer_id, COUNT(*) AS order_count
    FROM orders
    GROUP BY customer_id
),
payment_counts AS (
    SELECT customer_id, COUNT(*) AS payment_count
    FROM payments
    GROUP BY customer_id
)
SELECT
    c.customer_id,
    COALESCE(o.order_count, 0) AS order_count,
    COALESCE(p.payment_count, 0) AS payment_count
FROM customers c
LEFT JOIN order_counts o
    ON c.customer_id = o.customer_id
LEFT JOIN payment_counts p
    ON c.customer_id = p.customer_id;
```

Key idea:

```text
aggregate each one-to-many relationship first
→ one row per customer
→ join the already-aggregated results
```

---

# 106. Harder Pattern — All Parents, Filtered Children

Requirement:

> Show every department and the number of employees earning more than 80000.

Correct:

```sql
SELECT
    d.department_name,
    COUNT(e.employee_id) AS high_paid_count
FROM departments d
LEFT JOIN employees e
    ON d.department_id = e.department_id
   AND e.salary > 80000
GROUP BY d.department_name;
```

This preserves departments with:

```text
0 qualifying employees
```

---

# 107. Harder Pattern — Matching vs Non-Matching

### Matching rows

```sql
SELECT A.*
FROM A
INNER JOIN B
    ON A.id = B.id;
```

### Non-matching A rows

```sql
SELECT A.*
FROM A
LEFT JOIN B
    ON A.id = B.id
WHERE B.id IS NULL;
```

These two patterns cover a large class of interview questions.

---

# 108. Harder Pattern — Compare Two Datasets

Conceptual requirement:

> Show records present in either dataset, including records that exist in only one side.

Use:

```sql
FULL OUTER JOIN
```

Example:

```sql
SELECT
    a.id AS a_id,
    b.id AS b_id
FROM A a
FULL OUTER JOIN B b
    ON a.id = b.id;
```

Unmatched rows on either side remain.

---

# 109. Harder Pattern — Same Table Pairing

Requirement:

> Find employees who work in the same department but are different people.

```sql
SELECT
    e1.name,
    e2.name
FROM employees e1
JOIN employees e2
    ON e1.department_id = e2.department_id
   AND e1.employee_id <> e2.employee_id;
```

This produces each pair twice:

```text
Alice, Bob
Bob, Alice
```

If only one ordering is wanted:

```sql
AND e1.employee_id < e2.employee_id
```

---

# 110. Join Problem-Solving Framework

For any join problem:

## Step 1 — Identify tables

```text
Which entities are involved?
```

## Step 2 — Identify relationship

```text
PK/FK?
1:1?
1:N?
N:M?
```

## Step 3 — Identify required output

```text
What does one output row represent?
```

## Step 4 — Choose join type

```text
Only matches?
→ INNER

Keep all left rows?
→ LEFT

Keep all right rows?
→ RIGHT

Keep both sides?
→ FULL

All combinations?
→ CROSS

Same table?
→ SELF JOIN
```

## Step 5 — Write join predicate

```sql
ON ...
```

## Step 6 — Check multiplication

```text
Can one row match many?
```

## Step 7 — Place filters carefully

```text
ON or WHERE?
```

especially for outer joins.

## Step 8 — Check NULL behavior

```text
Which side can become NULL?
```

## Step 9 — Aggregate only at the intended grain

```text
COUNT?
SUM?
AVG?
DISTINCT?
```

---

# 111. Join Decision Tree

```text
Do you need to combine tables?
        |
        YES
        |
        +-- Same table?
        |      |
        |      YES → SELF JOIN
        |
        +-- Every combination?
        |      |
        |      YES → CROSS JOIN
        |
        +-- Only matching rows?
        |      |
        |      YES → INNER JOIN
        |
        +-- Keep all rows from left?
        |      |
        |      YES → LEFT JOIN
        |
        +-- Keep all rows from right?
        |      |
        |      YES → RIGHT JOIN
        |
        +-- Keep unmatched rows from both?
               |
               YES → FULL OUTER JOIN
```

---

# 112. Join Type Cheat Sheet

```text
INNER JOIN
→ matched rows only

LEFT JOIN
→ all left + matching right

RIGHT JOIN
→ matching left + all right

FULL OUTER JOIN
→ all left + all right

CROSS JOIN
→ every left/right combination

SELF JOIN
→ table joined to itself
```

---

# 113. Cardinality Cheat Sheet

```text
1 : 1
→ one row can match at most one row

1 : N
→ one row can match many rows

N : 1
→ many rows can match one row

N : M
→ usually use a junction table
→ joins can multiply rows substantially
```

---

# 114. NULL Cheat Sheet

```text
A.key = B.key

NULL = value
→ UNKNOWN

NULL = NULL
→ UNKNOWN

LEFT JOIN unmatched right side
→ right columns become NULL

WHERE condition
→ keeps only TRUE

WHERE right_column = ...
→ can eliminate NULL-extended rows
```

---

# 115. ON vs WHERE Cheat Sheet

For:

```sql
A LEFT JOIN B
```

remember:

```text
ON
→ determines matching

WHERE
→ filters resulting rows
```

Therefore:

```sql
LEFT JOIN B
ON A.id = B.a_id
AND B.status = 'ACTIVE'
```

can preserve unmatched A rows.

But:

```sql
LEFT JOIN B
ON A.id = B.a_id
WHERE B.status = 'ACTIVE'
```

can remove them.

---

# 116. Duplicate/Multiplication Cheat Sheet

Never assume:

```text
1 input row = 1 output row
```

Instead:

```text
one A row
    ↓
0 matching B rows
1 matching B row
many matching B rows
```

For outer joins:

```text
0 matches
→ preserved row + NULLs
```

For inner joins:

```text
0 matches
→ row disappears
```

---

# 117. Grain Cheat Sheet

Before joining, write mentally:

```text
Current grain:
one row = ?

After join:
one row = ?

After aggregation:
one row = ?
```

Example:

```text
customers
→ one row per customer

JOIN orders
→ one row per customer-order

JOIN order_items
→ one row per order-item

GROUP BY customer
→ one row per customer
```

This single habit prevents many SQL errors.

---

# 118. Performance Awareness

Joins can be expensive because the database must identify matching row combinations.

Important factors:

- indexes
- table sizes
- join cardinality
- uniqueness of keys
- selectivity
- statistics
- join algorithm
- memory
- physical execution plan

Common physical join strategies include:

```text
Nested Loop Join
Hash Join
Merge Join
```

The optimizer chooses based on the DBMS, schema, statistics, indexes, and query.

---

# 119. Indexes and Join Columns

If:

```sql
A.key = B.key
```

is frequently used, indexes on appropriate join/filter columns can help.

A common relational design is:

```text
parent primary key
→ indexed/unique

child foreign key
→ often indexed for join performance
```

But index usefulness depends on the query and data distribution.

Do not memorize:

```text
"Every join requires an index."
```

That is false.

---

# 120. Join Algorithm Awareness

## Nested Loop

Conceptually:

```text
for each row in A
    find matching rows in B
```

Can be efficient when one side is small or an appropriate index exists.

## Hash Join

Conceptually:

```text
build hash structure for one input
→ probe using the other input
```

Often useful for equality joins.

## Merge Join

Conceptually:

```text
sort both inputs
→ scan them together
```

Can be effective when inputs are suitably ordered/sorted.

These are physical execution strategies, not different SQL join semantics.

---

# 121. GATE-Level Conceptual Distinction

Do not confuse:

```text
JOIN TYPE
```

with:

```text
JOIN ALGORITHM
```

### Join type

Defines relational semantics:

```text
INNER
LEFT
RIGHT
FULL
CROSS
```

### Join algorithm

Defines physical execution:

```text
Nested Loop
Hash Join
Merge Join
```

The same logical inner join can be executed using different physical algorithms.

---

# 122. Common Join Mistakes

## Mistake 1 — Wrong preserved side

```sql
employees
LEFT JOIN departments
```

preserves employees, not departments.

---

## Mistake 2 — Using INNER JOIN when unmatched rows must survive

If every department must appear:

```text
departments LEFT JOIN employees
```

not an inner join.

---

## Mistake 3 — Filtering the nullable side in WHERE

```sql
LEFT JOIN B
WHERE B.status = 'X'
```

can remove unmatched A rows.

---

## Mistake 4 — Joining on incomplete keys

Using only part of a composite key can create incorrect matches.

---

## Mistake 5 — Ignoring many-to-many multiplication

Joins can produce far more rows than either input table.

---

## Mistake 6 — Blind DISTINCT

`DISTINCT` may hide a faulty join rather than fix it.

---

## Mistake 7 — COUNT(*), when counting matched children

With a left join, use:

```sql
COUNT(child.id)
```

when you need the number of actual child matches.

---

## Mistake 8 — Matching NULL with `=`

Do not expect:

```sql
A.key = B.key
```

to match two NULLs.

---

## Mistake 9 — Self-join pair duplication

Without:

```sql
a.id < b.id
```

you may get both:

```text
A,B
B,A
```

---

## Mistake 10 — Accidental Cartesian product

A missing join condition can create enormous row counts.

---

# 123. Interview Questions You Must Be Able to Answer

### Conceptual

1. Difference between `INNER JOIN` and `LEFT JOIN`?
2. When would you use `FULL OUTER JOIN`?
3. What does `CROSS JOIN` produce?
4. What is a self join?
5. What is a one-to-many relationship?
6. How is many-to-many represented?
7. Why can joins create duplicate-looking rows?
8. What is result grain?
9. Why does `COUNT(*)` differ from `COUNT(child.id)` after a left join?
10. What happens to unmatched rows in an outer join?

### SQL reasoning

11. Why can `WHERE B.col = ...` turn a left join into an effectively inner-filtered result?
12. What is the difference between a condition in `ON` and `WHERE`?
13. Why does `NULL = NULL` not produce TRUE?
14. How do you find rows in A with no matching row in B?
15. How do you find all parents including those with zero children?
16. How do you avoid many-to-many count inflation?
17. How do you find unique pairs using a self join?
18. How do you join using a composite key?
19. How do you return all records from both datasets?
20. How do you count distinct related entities after a multiplying join?

---

# 124. Mastery Checklist

## Join Types

- [ ] INNER JOIN
- [ ] LEFT JOIN
- [ ] RIGHT JOIN
- [ ] FULL OUTER JOIN
- [ ] CROSS JOIN
- [ ] SELF JOIN

## Relationships

- [ ] One-to-one
- [ ] One-to-many
- [ ] Many-to-one
- [ ] Many-to-many
- [ ] Junction tables

## Join Conditions

- [ ] Equality join
- [ ] Non-equality/range join
- [ ] Multiple join conditions
- [ ] Composite keys
- [ ] Non-key joins

## Cardinality

- [ ] Predict output row count
- [ ] Identify row multiplication
- [ ] Identify result grain
- [ ] Recognize accidental Cartesian products
- [ ] Recognize many-to-many multiplication

## NULL

- [ ] NULL in equality joins
- [ ] NULL-extended outer-join rows
- [ ] `IS NULL`
- [ ] NULL-safe matching concept
- [ ] Three-valued logic

## Filtering

- [ ] `ON` vs `WHERE`
- [ ] Filtering before/after joins
- [ ] Outer-join filtering trap
- [ ] Preserve parent rows with child filters
- [ ] Inner join predicate equivalence

## Aggregation

- [ ] JOIN + GROUP BY
- [ ] JOIN + HAVING
- [ ] `COUNT(*)` vs `COUNT(child.id)`
- [ ] `COUNT(DISTINCT ...)`
- [ ] Pre-aggregation
- [ ] Avoid cross-multiplication

## Self Join

- [ ] Employee-manager
- [ ] Same-group pairs
- [ ] Avoid duplicate pair order
- [ ] Avoid self-pairs

---

# 125. Final Revision Sheet

```text
JOIN
→ combines row combinations according to a predicate

INNER JOIN
→ only matching combinations

LEFT JOIN
→ all left rows
→ matching right rows
→ NULL for unmatched right

RIGHT JOIN
→ all right rows
→ matching left rows
→ NULL for unmatched left

FULL OUTER JOIN
→ all rows from both sides
→ NULL where unmatched

CROSS JOIN
→ Cartesian product
→ |A| × |B| rows

SELF JOIN
→ table joined to itself
→ use aliases

RELATIONSHIPS

1 : 1
→ at most one match each side

1 : N
→ one parent can produce many output rows

N : M
→ use junction table
→ potentially large multiplication

JOIN CONDITION

ON A.key = B.key

can be:
    equality
    inequality
    range
    multiple predicates
    composite-key condition

NULL

NULL = value
→ UNKNOWN

NULL = NULL
→ UNKNOWN

LEFT JOIN unmatched B
→ B columns become NULL

WHERE
→ keeps TRUE only

CARDINALITY

one A row
    ↓
0 B matches
    → INNER: disappears
    → LEFT: survives with NULL B

1 B match
    → 1 output row

many B matches
    → many output rows

GRAIN

Always ask:
"What does one output row represent?"

ON vs WHERE

LEFT JOIN B
ON condition
→ controls matching

LEFT JOIN B
WHERE B.condition
→ filters final rows
→ can remove NULL-extended rows

ANTI-JOIN

A rows with no B match:

SELECT A.*
FROM A
LEFT JOIN B
    ON A.key = B.key
WHERE B.key IS NULL;

ALL PARENTS INCLUDING ZERO CHILDREN

SELECT p.id, COUNT(c.id)
FROM parent p
LEFT JOIN child c
    ON p.id = c.parent_id
GROUP BY p.id;

IMPORTANT

COUNT(*)
→ counts joined rows

COUNT(child.id)
→ counts non-NULL child IDs

MANY-TO-MANY TRAP

A
JOIN B
JOIN C

can create:

B matches × C matches

Do not aggregate blindly.

Prefer:
    pre-aggregation
    COUNT(DISTINCT ...)
    separate CTEs/subqueries

SELF-JOIN PAIRS

e1.department_id = e2.department_id
AND e1.employee_id < e2.employee_id

→ unique unordered pairs

JOIN TYPE ≠ JOIN ALGORITHM

Logical:
    INNER / LEFT / RIGHT / FULL / CROSS

Physical:
    Nested Loop
    Hash Join
    Merge Join
```

---

# 126. Mastery Standard

You have mastered SQL Joins when you can:

1. Choose the correct join type from an English requirement.
2. Explain exactly which rows survive each join.
3. Predict where NULLs appear.
4. Explain one-to-one, one-to-many, and many-to-many relationships.
5. Predict row multiplication before executing the query.
6. Identify the grain of a joined result.
7. Explain why a join can create repeated-looking rows without being incorrect.
8. Use `ON` correctly for join relationships.
9. Explain the `ON` vs `WHERE` distinction for outer joins.
10. Find unmatched rows with a left anti-join.
11. Count children correctly after a `LEFT JOIN`.
12. Use self joins for hierarchies and pair generation.
13. Use composite join keys correctly.
14. Recognize accidental Cartesian products.
15. Avoid count inflation from multiple one-to-many joins.
16. Use `COUNT(DISTINCT ...)` when the intended entity grain requires it.
17. Explain why NULL does not equal NULL under ordinary SQL equality.
18. Distinguish logical join type from physical join algorithm.
19. Reason about join order when outer joins are involved.
20. Solve multi-table interview problems without guessing the output grain.

The target mental process is:

```text
English requirement
        ↓
Identify entities
        ↓
Identify relationship
        ↓
Identify preserved side
        ↓
Choose JOIN type
        ↓
Write exact ON condition
        ↓
Predict cardinality
        ↓
Identify output grain
        ↓
Place filters carefully
        ↓
Handle NULLs
        ↓
Aggregate only after validating grain
```

If this becomes automatic, most medium-level SQL join questions become a matter of translating the relationship and required grain into SQL rather than trial-and-error.
