# 02. SQL Filtering & Conditions

> **Track:** SQL Interview Preparation  
> **Overall repository number:** 25  
> **SQL topic number:** 02  
> **Target:** FAANG + top product companies | SDE + ML/AI Engineer  
> **Difficulty:** Easy → Medium → Hard, with Medium dominating

---

# 1. Overview

Filtering is one of the most frequently tested SQL skills.

The core idea is:

```text
FROM
  ↓
WHERE
  ↓
SELECT / later query stages
```

Use filtering to determine which rows participate in a query.

This topic covers:

- comparison operators
- `AND`
- `OR`
- `NOT`
- `IN`
- `NOT IN`
- `BETWEEN`
- `LIKE`
- wildcards
- `IS NULL`
- `IS NOT NULL`

The most important interview traps involve:

- operator precedence
- `NULL`
- `NOT IN`
- inclusive `BETWEEN`
- wildcard behavior
- combining conditions correctly

---

# 2. Sample Schema

Use this table for the examples.

### employees

| employee_id | name | department | salary | age | city | manager_id |
|---:|---|---|---:|---:|---|---:|
| 101 | Alice | Engineering | 90000 | 28 | Hyderabad | NULL |
| 102 | Bob | Engineering | 75000 | 24 | Bengaluru | 101 |
| 103 | Carol | Sales | 70000 | 31 | Mumbai | 105 |
| 104 | David | Sales | 65000 | 26 | Delhi | 105 |
| 105 | Eva | Sales | 100000 | 35 | Mumbai | NULL |
| 106 | Frank | HR | 60000 | 29 | Pune | NULL |
| 107 | Grace | Engineering | 120000 | 40 | Bengaluru | 101 |
| 108 | Henry | HR | 55000 | 23 | Hyderabad | 106 |

---

# 3. Comparison Operators

SQL comparison operators include:

```text
=
<>
!=
>
<
>=
<=
```

Examples:

```sql
SELECT *
FROM employees
WHERE salary = 90000;
```

```sql
SELECT *
FROM employees
WHERE salary > 80000;
```

```sql
SELECT *
FROM employees
WHERE age <= 30;
```

For portable SQL, `<>` is the standard SQL "not equal" operator. `!=` is supported by many popular DBMSs.

---

# 4. Equality

Use:

```sql
=
```

Example:

```sql
SELECT name
FROM employees
WHERE department = 'Engineering';
```

This returns employees whose department value equals `Engineering`.

String comparisons can be affected by:

- database collation
- character set
- case-sensitivity configuration

Do not assume that:

```text
'Engineering' = 'engineering'
```

has the same behavior in every DBMS.

---

# 5. Not Equal

Use:

```sql
<>
```

Example:

```sql
SELECT name
FROM employees
WHERE department <> 'HR';
```

Many systems also support:

```sql
WHERE department != 'HR'
```

For interview answers, know both, but prefer standard SQL terminology when discussing portability.

---

# 6. Greater Than / Less Than

Greater than:

```sql
WHERE salary > 80000
```

Less than:

```sql
WHERE salary < 80000
```

Greater than or equal:

```sql
WHERE salary >= 80000
```

Less than or equal:

```sql
WHERE salary <= 80000
```

---

# 7. Comparison with NULL

This is critical.

Do **not** write:

```sql
WHERE manager_id = NULL
```

or:

```sql
WHERE manager_id <> NULL
```

Use:

```sql
WHERE manager_id IS NULL
```

or:

```sql
WHERE manager_id IS NOT NULL
```

Why?

`NULL` represents missing/unknown/inapplicable information.

Normal equality/inequality comparisons involving NULL produce `UNKNOWN`, not the desired TRUE/FALSE test.

---

# 8. AND

`AND` requires both conditions to be true.

```sql
SELECT *
FROM employees
WHERE department = 'Engineering'
  AND salary > 80000;
```

A row must satisfy:

```text
department = Engineering
AND
salary > 80000
```

---

# 9. AND Truth Table

| A | B | A AND B |
|---|---|---|
| TRUE | TRUE | TRUE |
| TRUE | FALSE | FALSE |
| FALSE | TRUE | FALSE |
| FALSE | FALSE | FALSE |

With SQL's three-valued logic, `UNKNOWN` is also possible.

Important cases:

| A | B | A AND B |
|---|---|---|
| FALSE | UNKNOWN | FALSE |
| TRUE | UNKNOWN | UNKNOWN |

For `WHERE`, only TRUE rows pass the filter.

---

# 10. OR

`OR` requires at least one condition to be true.

```sql
SELECT *
FROM employees
WHERE department = 'Engineering'
   OR department = 'Sales';
```

This returns rows belonging to either department.

---

# 11. OR Truth Table

| A | B | A OR B |
|---|---|---|
| TRUE | TRUE | TRUE |
| TRUE | FALSE | TRUE |
| FALSE | TRUE | TRUE |
| FALSE | FALSE | FALSE |

With `UNKNOWN`:

| A | B | A OR B |
|---|---|---|
| TRUE | UNKNOWN | TRUE |
| FALSE | UNKNOWN | UNKNOWN |

---

# 12. NOT

`NOT` negates a condition.

```sql
SELECT *
FROM employees
WHERE NOT department = 'HR';
```

Equivalent conceptually to:

```sql
WHERE department <> 'HR'
```

However, with NULL values, `NOT` interacts with three-valued logic, so "not equal" and "not NULL" are not interchangeable concepts.

---

# 13. Operator Precedence

A major interview trap.

The common logical precedence is:

```text
NOT
AND
OR
```

Therefore:

```sql
WHERE A OR B AND C
```

means:

```sql
WHERE A OR (B AND C)
```

not:

```sql
WHERE (A OR B) AND C
```

Use parentheses when the intended grouping matters:

```sql
WHERE (A OR B)
  AND C
```

---

# 14. Example of AND/OR Precedence

Query:

```sql
SELECT *
FROM employees
WHERE department = 'Engineering'
   OR department = 'Sales'
  AND salary > 80000;
```

Interpretation:

```sql
WHERE department = 'Engineering'
   OR (department = 'Sales' AND salary > 80000);
```

If the intended condition is:

```text
Engineering OR Sales
```

and both must have salary > 80000:

```sql
SELECT *
FROM employees
WHERE (department = 'Engineering'
    OR department = 'Sales')
  AND salary > 80000;
```

---

# 15. IN

`IN` checks whether a value belongs to a specified set.

Instead of:

```sql
WHERE department = 'Engineering'
   OR department = 'Sales'
   OR department = 'HR'
```

write:

```sql
WHERE department IN ('Engineering', 'Sales', 'HR')
```

This is usually cleaner and easier to maintain.

---

# 16. IN with Numbers

```sql
SELECT *
FROM employees
WHERE employee_id IN (101, 104, 107);
```

This means:

```text
employee_id = 101
OR employee_id = 104
OR employee_id = 107
```

---

# 17. NOT IN

`NOT IN` excludes values in a specified set.

```sql
SELECT *
FROM employees
WHERE department NOT IN ('HR', 'Sales');
```

Conceptually:

```text
department != HR
AND
department != Sales
```

---

# 18. Critical NOT IN + NULL Trap

This is one of the highest-value SQL interview concepts.

Suppose a subquery returns:

```text
1
2
NULL
```

Then:

```sql
WHERE employee_id NOT IN (
    SELECT employee_id
    FROM some_table
)
```

can produce unexpected results because comparisons involving `NULL` become `UNKNOWN`.

For example:

```text
x NOT IN (1, 2, NULL)
```

is logically equivalent to:

```text
x <> 1
AND x <> 2
AND x <> NULL
```

The final comparison is `UNKNOWN`.

Therefore the overall result may be `UNKNOWN`, preventing the row from passing the `WHERE` clause.

### Safer Alternative

When expressing an anti-match against another table, `NOT EXISTS` is often safer:

```sql
SELECT e.*
FROM employees e
WHERE NOT EXISTS (
    SELECT 1
    FROM excluded_employees x
    WHERE x.employee_id = e.employee_id
);
```

`NOT EXISTS` does not have the same NULL trap as `NOT IN`.

---

# 19. IN and NULL

Consider:

```sql
WHERE department IN ('Engineering', NULL)
```

A row with:

```text
department = NULL
```

does not become a TRUE match merely because NULL appears in the `IN` list.

For NULL testing use:

```sql
WHERE department IS NULL
```

---

# 20. BETWEEN

`BETWEEN` checks whether a value lies within an inclusive range.

```sql
WHERE salary BETWEEN 70000 AND 90000
```

means:

```sql
WHERE salary >= 70000
  AND salary <= 90000
```

Both endpoints are included.

---

# 21. BETWEEN Examples

```sql
SELECT name, salary
FROM employees
WHERE salary BETWEEN 70000 AND 90000;
```

This includes:

```text
70000
90000
```

as valid boundary values.

---

# 22. NOT BETWEEN

```sql
SELECT name, salary
FROM employees
WHERE salary NOT BETWEEN 70000 AND 90000;
```

Conceptually:

```sql
WHERE salary < 70000
   OR salary > 90000
```

For NULL values, the expression can evaluate to `UNKNOWN`, so a NULL salary will not pass the `WHERE` filter.

---

# 23. BETWEEN with Dates

`BETWEEN` can be used with date/time values, but care is required with timestamps.

For dates:

```sql
WHERE order_date BETWEEN DATE '2026-01-01'
                      AND DATE '2026-01-31'
```

The endpoints are included.

For timestamps, a half-open interval is often safer.

Instead of:

```sql
WHERE created_at BETWEEN '2026-01-01' AND '2026-01-31'
```

consider:

```sql
WHERE created_at >= '2026-01-01'
  AND created_at <  '2026-02-01'
```

This avoids accidentally excluding times later on January 31.

Exact date literal syntax varies by DBMS.

---

# 24. LIKE

`LIKE` performs pattern matching on character data.

Example:

```sql
SELECT *
FROM employees
WHERE name LIKE 'A%';
```

This finds names beginning with `A`.

---

# 25. LIKE Wildcard `%`

`%` represents zero or more characters.

Examples:

```sql
LIKE 'A%'
```

Starts with `A`.

```sql
LIKE '%a'
```

Ends with `a`.

```sql
LIKE '%ar%'
```

Contains `ar`.

```sql
LIKE '%'
```

Matches a string of any length, subject to NULL and DBMS pattern semantics.

---

# 26. LIKE Wildcard `_`

`_` represents one character.

Example:

```sql
WHERE name LIKE 'A___'
```

matches a four-character string beginning with `A`.

For example:

```text
Alex
```

matches.

But:

```text
Alice
```

does not match that exact pattern because it contains five characters.

---

# 27. `%` vs `_`

| Pattern | Meaning |
|---|---|
| `%` | zero or more characters |
| `_` | exactly one character |

Examples:

```sql
'A%'
```

```text
Alice
Adam
A
```

can match.

```sql
'A___'
```

requires:

```text
A + 3 characters
```

---

# 28. LIKE Examples

## Starts with A

```sql
WHERE name LIKE 'A%'
```

## Ends with a

```sql
WHERE name LIKE '%a'
```

## Contains "ar"

```sql
WHERE name LIKE '%ar%'
```

## Second character is `o`

```sql
WHERE name LIKE '_o%'
```

## Exactly three characters

```sql
WHERE name LIKE '___'
```

---

# 29. Escaping LIKE Wildcards

Sometimes `%` or `_` should be treated as literal characters.

Many SQL systems support:

```sql
LIKE '50\%%' ESCAPE '\'
```

Conceptually:

```text
50%
```

followed by any number of characters.

Another common form:

```sql
LIKE 'A\_B' ESCAPE '\'
```

matches the literal string:

```text
A_B
```

Exact escaping syntax can vary by DBMS.

---

# 30. Case Sensitivity of LIKE

Do not assume `LIKE` has identical case behavior in every database.

Behavior can depend on:

- DBMS
- collation
- column/database configuration

For example, case-insensitive matching may require DBMS-specific syntax or operators.

The portable concept to remember is:

```text
LIKE = pattern matching
```

not a universal case-sensitivity rule.

---

# 31. IS NULL

Use:

```sql
IS NULL
```

to test for NULL.

Example:

```sql
SELECT name
FROM employees
WHERE manager_id IS NULL;
```

This returns employees without a manager value.

---

# 32. IS NOT NULL

Use:

```sql
IS NOT NULL
```

to test for a non-NULL value.

```sql
SELECT name
FROM employees
WHERE manager_id IS NOT NULL;
```

---

# 33. NULL Truth Table

SQL has three logical values:

```text
TRUE
FALSE
UNKNOWN
```

Important:

```text
NULL = NULL → UNKNOWN
NULL <> NULL → UNKNOWN
NULL > 10   → UNKNOWN
NULL < 10   → UNKNOWN
```

Therefore:

```sql
WHERE column = NULL
```

does not identify NULL rows.

Use:

```sql
WHERE column IS NULL
```

---

# 34. Combining NULL with Conditions

Example:

```sql
SELECT *
FROM employees
WHERE manager_id IS NULL
  AND salary > 80000;
```

This explicitly handles NULL.

Avoid trying to use ordinary equality for NULL.

---

# 35. Three-Valued Logic

The core values are:

```text
TRUE
FALSE
UNKNOWN
```

For `WHERE`, only:

```text
TRUE
```

passes the filter.

## AND

```text
TRUE AND UNKNOWN   = UNKNOWN
FALSE AND UNKNOWN  = FALSE
```

## OR

```text
TRUE OR UNKNOWN    = TRUE
FALSE OR UNKNOWN   = UNKNOWN
```

## NOT

```text
NOT TRUE     = FALSE
NOT FALSE    = TRUE
NOT UNKNOWN  = UNKNOWN
```

These rules explain many SQL NULL traps.

---

# 36. De Morgan's Laws

Important for `AND`, `OR`, and `NOT`.

```text
NOT (A AND B)
=
(NOT A) OR (NOT B)
```

and:

```text
NOT (A OR B)
=
(NOT A) AND (NOT B)
```

Example:

```sql
WHERE NOT (
    department = 'HR'
    OR salary < 60000
)
```

can conceptually become:

```sql
WHERE department <> 'HR'
  AND salary >= 60000
```

With NULL values, SQL's three-valued logic means you must be careful when claiming exact equivalence in the presence of UNKNOWN.

---

# 37. Combining IN with NOT

```sql
WHERE department NOT IN ('HR', 'Sales')
```

is usually preferable to writing many `AND` comparisons.

Similarly:

```sql
WHERE department IN ('Engineering', 'Sales')
```

is preferable to repeated `OR` equality tests.

---

# 38. Comparison + AND

Example:

> Find Engineering employees with salary at least 80000.

```sql
SELECT name, salary
FROM employees
WHERE department = 'Engineering'
  AND salary >= 80000;
```

---

# 39. Comparison + OR

Example:

> Find employees in Engineering or HR.

```sql
SELECT name, department
FROM employees
WHERE department = 'Engineering'
   OR department = 'HR';
```

Prefer:

```sql
WHERE department IN ('Engineering', 'HR')
```

for simple membership conditions.

---

# 40. NOT + Condition

Example:

> Find employees who are not in HR.

```sql
SELECT name
FROM employees
WHERE NOT department = 'HR';
```

or:

```sql
SELECT name
FROM employees
WHERE department <> 'HR';
```

If NULL departments are possible, neither condition should be interpreted as "return every row except HR including NULL" without analyzing SQL's three-valued logic.

---

# 41. BETWEEN + AND

Do not confuse the `AND` inside `BETWEEN` with the separate logical `AND`.

```sql
salary BETWEEN 70000 AND 90000
```

is a range predicate.

It is conceptually:

```sql
salary >= 70000
AND salary <= 90000
```

---

# 42. LIKE + OR

Example:

```sql
SELECT name
FROM employees
WHERE name LIKE 'A%'
   OR name LIKE 'B%';
```

This finds names beginning with A or B.

---

# 43. LIKE + AND

```sql
SELECT name
FROM employees
WHERE name LIKE 'A%'
  AND salary > 80000;
```

Both conditions must be true.

---

# 44. Complex Filtering Example

Question:

> Find Engineering employees earning between 70000 and 100000, located in Bengaluru or Hyderabad, with a non-NULL manager.

```sql
SELECT name, salary, city
FROM employees
WHERE department = 'Engineering'
  AND salary BETWEEN 70000 AND 100000
  AND city IN ('Bengaluru', 'Hyderabad')
  AND manager_id IS NOT NULL;
```

Breakdown:

```text
Engineering
    AND
salary range
    AND
city membership
    AND
manager exists
```

---

# 45. Another Complex Example

Question:

> Find employees whose names begin with A or G and whose salary is greater than 80000.

```sql
SELECT name, salary
FROM employees
WHERE (name LIKE 'A%' OR name LIKE 'G%')
  AND salary > 80000;
```

Parentheses make the intended logic explicit.

---

# 46. WHERE Filters Rows

`WHERE` is applied to rows before aggregation.

Example:

```sql
SELECT *
FROM employees
WHERE salary > 80000;
```

Only rows satisfying the predicate participate in later query processing.

This distinction becomes important when learning:

```text
WHERE vs HAVING
```

---

# 47. WHERE vs HAVING Preview

`WHERE`:

```text
filters rows
```

`HAVING`:

```text
filters groups
```

Example:

```sql
SELECT department, COUNT(*)
FROM employees
WHERE salary > 60000
GROUP BY department
HAVING COUNT(*) >= 2;
```

Conceptually:

```text
WHERE → remove individual rows
GROUP BY → create groups
HAVING → remove groups
```

Full aggregation will be covered in a later topic.

---

# 48. Filtering NULL Correctly

Suppose:

```text
manager_id
----------
NULL
101
105
NULL
```

Correct:

```sql
WHERE manager_id IS NULL
```

returns:

```text
NULL
NULL
```

Correct:

```sql
WHERE manager_id IS NOT NULL
```

returns:

```text
101
105
```

---

# 49. Important NULL + NOT Example

Consider:

```sql
WHERE NOT manager_id = 101
```

This does **not** mean:

```text
manager_id != 101 OR manager_id IS NULL
```

For a NULL manager ID:

```text
manager_id = 101
→ UNKNOWN

NOT UNKNOWN
→ UNKNOWN
```

The row therefore does not pass the `WHERE`.

If you want NULL values included:

```sql
WHERE manager_id <> 101
   OR manager_id IS NULL
```

---

# 50. NOT IN vs NOT EXISTS

This is a high-priority interview distinction.

Potentially dangerous:

```sql
WHERE id NOT IN (
    SELECT id
    FROM other_table
)
```

if the subquery can return NULL.

Safer anti-join logic:

```sql
WHERE NOT EXISTS (
    SELECT 1
    FROM other_table o
    WHERE o.id = t.id
)
```

The exact performance depends on the DBMS, indexes, data distribution, and query plan. The key conceptual distinction here is NULL semantics.

---

# 51. SARGability Preview

Some filtering predicates can interact differently with indexes.

A common example:

```sql
WHERE salary >= 80000
```

can be index-friendly if an appropriate index exists.

A transformed expression such as:

```sql
WHERE function(column) = ...
```

may prevent or complicate efficient index use depending on the DBMS and available indexes.

Index and query-optimization behavior is DBMS-specific and will be covered later.

---

# 52. Common Interview Patterns

## Pattern 1 — Exact match

```sql
WHERE department = 'Engineering'
```

## Pattern 2 — Range

```sql
WHERE salary >= 70000
  AND salary <= 90000
```

or:

```sql
WHERE salary BETWEEN 70000 AND 90000
```

## Pattern 3 — Membership

```sql
WHERE department IN ('Engineering', 'Sales')
```

## Pattern 4 — Exclusion

```sql
WHERE department NOT IN ('HR', 'Sales')
```

Remember the NULL issue for subqueries.

## Pattern 5 — Prefix

```sql
WHERE name LIKE 'A%'
```

## Pattern 6 — Contains

```sql
WHERE name LIKE '%an%'
```

## Pattern 7 — Missing value

```sql
WHERE manager_id IS NULL
```

## Pattern 8 — Present value

```sql
WHERE manager_id IS NOT NULL
```

---

# 53. Common Mistakes

## Mistake 1

```sql
WHERE column = NULL
```

Correct:

```sql
WHERE column IS NULL
```

---

## Mistake 2

Assuming:

```sql
NOT IN
```

is always equivalent to repeated `<>` conditions when NULL values are possible.

Analyze NULL semantics.

---

## Mistake 3

Forgetting that `BETWEEN` is inclusive.

```sql
BETWEEN 10 AND 20
```

includes:

```text
10
20
```

---

## Mistake 4

Misreading precedence.

```sql
A OR B AND C
```

means:

```text
A OR (B AND C)
```

Use parentheses for clarity.

---

## Mistake 5

Assuming `%` means exactly one character.

It means:

```text
zero or more characters
```

---

## Mistake 6

Assuming `_` means zero or more characters.

It means:

```text
exactly one character
```

---

## Mistake 7

Assuming `LIKE` has identical case behavior everywhere.

Case behavior can depend on DBMS/collation.

---

## Mistake 8

Using `BETWEEN` carelessly with timestamps.

For a whole-day/month time range, a half-open interval is often clearer:

```sql
timestamp >= start
AND timestamp < next_boundary
```

---

# 54. GATE / CS Theory

## 54.1 Boolean Logic

Know:

```text
AND
OR
NOT
```

and their truth tables.

---

## 54.2 Three-Valued Logic

SQL extends classical Boolean logic with:

```text
UNKNOWN
```

because NULL represents missing/unknown information.

---

## 54.3 WHERE and UNKNOWN

A `WHERE` clause retains rows for which the predicate evaluates to:

```text
TRUE
```

Rows evaluating to:

```text
FALSE
UNKNOWN
```

do not pass the filter.

---

## 54.4 NULL Comparisons

```text
NULL = value   → UNKNOWN
NULL <> value  → UNKNOWN
NULL = NULL    → UNKNOWN
```

Therefore:

```sql
IS NULL
```

is required for NULL testing.

---

## 54.5 BETWEEN

Conceptually:

```text
x BETWEEN a AND b
```

means:

```text
x >= a AND x <= b
```

with inclusive boundaries.

---

## 54.6 IN

Conceptually:

```text
x IN (a, b, c)
```

corresponds to:

```text
x = a OR x = b OR x = c
```

subject to SQL's NULL/three-valued logic.

---

## 54.7 NOT IN

Conceptually:

```text
x NOT IN (a, b, c)
```

corresponds to:

```text
x <> a AND x <> b AND x <> c
```

Again, NULL can make the overall predicate UNKNOWN.

---

## 54.8 LIKE

The important standard pattern symbols are:

```text
%
_
```

where:

```text
% → zero or more characters
_ → one character
```

---

# 55. GATE-Style Solved Questions

> These are **GATE-style practice questions**, not claims about exact official GATE wording or years.

## Question 1 — NULL Filtering

Consider:

```text
T
---
A
10
20
NULL
30
```

Evaluate:

```sql
SELECT COUNT(*)
FROM T
WHERE A <> NULL;
```

### Solution

For the non-NULL values:

```text
10 <> NULL → UNKNOWN
20 <> NULL → UNKNOWN
30 <> NULL → UNKNOWN
```

For NULL:

```text
NULL <> NULL → UNKNOWN
```

No row satisfies the predicate with TRUE.

Therefore:

```text
Answer = 0
```

Correct NULL test:

```sql
WHERE A IS NOT NULL
```

---

# 56. Question 2 — BETWEEN

Suppose:

```text
A = [5, 10, 15, 20, 25]
```

How many values satisfy:

```sql
A BETWEEN 10 AND 20
```

### Solution

`BETWEEN` is inclusive.

Matching values:

```text
10
15
20
```

Answer:

```text
3
```

---

# 57. Question 3 — Operator Precedence

Consider:

```sql
WHERE A = 1
   OR B = 2
  AND C = 3
```

Which grouping is correct?

### Solution

`AND` has higher precedence than `OR`.

Therefore:

```sql
WHERE A = 1
   OR (B = 2 AND C = 3)
```

Answer:

```text
A OR (B AND C)
```

---

# 58. Question 4 — LIKE

Which strings match:

```sql
LIKE 'A__%'
```

### Solution

Break the pattern down:

```text
A
_
_
%
```

Therefore:

```text
A
+ exactly two characters
+ zero or more additional characters
```

Examples that match:

```text
Alice
Adam
Aaron
```

assuming ordinary character semantics.

A string such as:

```text
A
```

does not match because two `_` characters require two additional characters.

---

# 59. Question 5 — NOT IN and NULL

Suppose a subquery returns:

```text
1
2
NULL
```

Consider:

```sql
SELECT *
FROM employees
WHERE employee_id NOT IN (
    SELECT employee_id
    FROM excluded
);
```

Why can this fail to return expected rows?

### Solution

Conceptually:

```text
x NOT IN (1, 2, NULL)
```

becomes:

```text
x <> 1
AND x <> 2
AND x <> NULL
```

The final comparison is:

```text
UNKNOWN
```

Thus the complete predicate can become UNKNOWN.

A common safer formulation is:

```sql
WHERE NOT EXISTS (
    SELECT 1
    FROM excluded x
    WHERE x.employee_id = employees.employee_id
)
```

---

# 60. Question 6 — AND / OR

Given:

```text
salary = 90000
department = 'Engineering'
age = 25
```

evaluate:

```sql
WHERE department = 'Engineering'
   OR salary > 100000
  AND age < 30
```

### Solution

Because `AND` has higher precedence:

```text
Engineering
OR
(90000 > 100000 AND 25 < 30)
```

becomes:

```text
TRUE OR (FALSE AND TRUE)
```

which is:

```text
TRUE OR FALSE
```

Therefore:

```text
TRUE
```

The row passes.

---

# 61. Interview Practice Questions

You should be able to write each query without reference.

## Q1

Find employees with salary greater than `80000`.

```sql
SELECT *
FROM employees
WHERE salary > 80000;
```

## Q2

Find employees whose department is Engineering or Sales.

```sql
SELECT *
FROM employees
WHERE department IN ('Engineering', 'Sales');
```

## Q3

Find employees outside Engineering and Sales.

```sql
SELECT *
FROM employees
WHERE department NOT IN ('Engineering', 'Sales');
```

Remember that NULL departments require separate consideration.

## Q4

Find employees aged between 25 and 35 inclusive.

```sql
SELECT *
FROM employees
WHERE age BETWEEN 25 AND 35;
```

## Q5

Find names beginning with `A`.

```sql
SELECT name
FROM employees
WHERE name LIKE 'A%';
```

## Q6

Find names containing `an`.

```sql
SELECT name
FROM employees
WHERE name LIKE '%an%';
```

## Q7

Find employees without managers.

```sql
SELECT *
FROM employees
WHERE manager_id IS NULL;
```

## Q8

Find employees who have managers.

```sql
SELECT *
FROM employees
WHERE manager_id IS NOT NULL;
```

---

# 62. Pattern Recognition

| Requirement | Pattern |
|---|---|
| Exact value | `=` |
| Not equal | `<>` |
| Greater/less | `>`, `<`, `>=`, `<=` |
| Multiple allowed values | `IN` |
| Exclude values | `NOT IN` |
| Inclusive numeric range | `BETWEEN` |
| Starts with text | `LIKE 'x%'` |
| Ends with text | `LIKE '%x'` |
| Contains text | `LIKE '%x%'` |
| One-character wildcard | `_` |
| Any-number-of-character wildcard | `%` |
| Missing value | `IS NULL` |
| Existing value | `IS NOT NULL` |
| Multiple required conditions | `AND` |
| Alternative conditions | `OR` |
| Negation | `NOT` |

---

# 63. Query Construction Method

When given an English requirement:

### Step 1 — Identify the column

Example:

```text
salary
```

### Step 2 — Identify the comparison

```text
greater than
```

becomes:

```sql
>
```

### Step 3 — Identify multiple conditions

```text
Engineering AND salary > 80000
```

### Step 4 — Identify set membership

```text
Engineering or Sales
```

becomes:

```sql
IN ('Engineering', 'Sales')
```

### Step 5 — Identify ranges

```text
70000 through 90000
```

becomes:

```sql
BETWEEN 70000 AND 90000
```

### Step 6 — Identify patterns

```text
starts with A
```

becomes:

```sql
LIKE 'A%'
```

### Step 7 — Identify missing values

```text
no manager
```

becomes:

```sql
IS NULL
```

This translation process is more useful than memorizing individual queries.

---

# 64. Advanced Filtering Preview

Later SQL topics will extend filtering into:

```text
CASE
COALESCE
NULLIF
Subqueries
EXISTS
NOT EXISTS
CTEs
JOIN conditions
HAVING
Window functions
Conditional aggregation
Date filtering
String filtering
Regular expressions
```

Do not mix these into basic filtering until the fundamentals are automatic.

---

# 65. SQL Dialect Awareness

SQL is standardized, but individual DBMSs differ.

Potential differences include:

- pagination syntax
- date literals
- string concatenation
- case-insensitive matching
- regular expressions
- NULL-related operators
- functions
- collation behavior

For interviews, first understand the **SQL concept**, then adapt syntax to the specified DBMS.

---

# 66. Mastery Checklist

## Comparison Operators

- [ ] `=`
- [ ] `<>`
- [ ] `!=`
- [ ] `>`
- [ ] `<`
- [ ] `>=`
- [ ] `<=`

## Logical Operators

- [ ] Understand `AND`
- [ ] Understand `OR`
- [ ] Understand `NOT`
- [ ] Know precedence
- [ ] Use parentheses correctly
- [ ] Understand three-valued logic

## Membership

- [ ] `IN`
- [ ] `NOT IN`
- [ ] Understand `NOT IN` + NULL
- [ ] Know when `NOT EXISTS` is safer

## Ranges

- [ ] `BETWEEN`
- [ ] Know endpoints are inclusive
- [ ] Understand `NOT BETWEEN`
- [ ] Know timestamp range pitfalls

## Pattern Matching

- [ ] `LIKE`
- [ ] `%`
- [ ] `_`
- [ ] Prefix matching
- [ ] Suffix matching
- [ ] Contains matching
- [ ] Literal wildcard escaping
- [ ] Understand DBMS/collation case behavior

## NULL

- [ ] Understand NULL
- [ ] `IS NULL`
- [ ] `IS NOT NULL`
- [ ] Understand NULL comparisons
- [ ] Understand UNKNOWN
- [ ] Understand NULL with `NOT`
- [ ] Understand NULL with `NOT IN`

## Interview Readiness

- [ ] Can translate English requirements into predicates
- [ ] Can combine multiple conditions
- [ ] Can spot precedence errors
- [ ] Can spot NULL errors
- [ ] Can write pattern-matching queries
- [ ] Can explain `NOT IN` vs `NOT EXISTS`
- [ ] Can explain every predicate used

---

# 67. Final Revision Sheet

```text
COMPARISON
    =       equal
    <>      not equal
    !=      not equal in many DBMSs
    >       greater
    <       less
    >=      greater/equal
    <=      less/equal

LOGIC
    NOT
    AND
    OR

PRECEDENCE
    NOT
      ↓
    AND
      ↓
    OR

MEMBERSHIP
    x IN (...)
    x NOT IN (...)

RANGE
    x BETWEEN a AND b
    → x >= a AND x <= b
    → endpoints included

LIKE
    % → zero or more characters
    _ → exactly one character

PATTERNS
    'A%'    → starts with A
    '%A'    → ends with A
    '%A%'   → contains A
    '_A%'   → second character is A

NULL
    x = NULL          ❌
    x <> NULL         ❌
    x IS NULL         ✓
    x IS NOT NULL     ✓

THREE-VALUED LOGIC
    TRUE
    FALSE
    UNKNOWN

WHERE
    only TRUE passes

NOT IN TRAP
    NOT IN + NULL
    → UNKNOWN can eliminate expected rows

ANTI-MATCH
    Prefer NOT EXISTS when NULL semantics make NOT IN unsafe

BETWEEN + TIMESTAMP
    Prefer:
    timestamp >= start
    AND timestamp < next_boundary
    when representing a whole time interval
```

---

# 68. Mastery Standard

You have mastered SQL Filtering & Conditions when you can:

1. Write all comparison predicates from memory.
2. Combine conditions with `AND`, `OR`, and `NOT` correctly.
3. Explain operator precedence.
4. Use `IN` and `NOT IN` appropriately.
5. Explain why `NOT IN` can fail with NULL.
6. Use `BETWEEN` correctly and know that its boundaries are inclusive.
7. Write `LIKE` patterns without confusing `%` and `_`.
8. Correctly test NULL with `IS NULL` and `IS NOT NULL`.
9. Explain SQL's three-valued logic.
10. Translate natural-language requirements into precise SQL predicates.
11. Recognize when parentheses are necessary.
12. Explain the difference between row filtering and group filtering at a conceptual level.

The target is not simply:

```text
"I know the WHERE clause."
```

The target is:

```text
"I can translate a requirement into a
correct SQL predicate, including edge cases
involving NULL, ranges, pattern matching,
and Boolean logic."
```
