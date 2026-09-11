# SQL Topic 06 — Set Operations

> **Overall repository topic:** 29  
> **SQL topic number:** 6  
> **Difficulty focus:** Easy → Medium → Hard, with Medium dominant  
> **Goal:** Master `UNION`, `UNION ALL`, `INTERSECT`, and `EXCEPT`, especially the difference between `UNION` and `UNION ALL`.

---

## 1. What Are Set Operations?

Set operations combine the results of two or more `SELECT` queries **vertically**.

Instead of joining columns side-by-side like a `JOIN`, set operations place rows from one result below rows from another result.

Conceptually:

```text
Query A
-------
A1
A2
A3

Query B
-------
B1
B2

Set operation
-------------
A1
A2
A3
B1
B2
```

The major SQL set operations are:

| Operation | Meaning | Removes duplicates? |
|---|---|---:|
| `UNION` | Rows in A or B | Yes |
| `UNION ALL` | Rows in A and B | No |
| `INTERSECT` | Rows common to A and B | Yes |
| `EXCEPT` | Rows in A but not B | Yes |

The exact support for some operators varies by database system, but these are standard relational/set concepts.

---

# 2. The Most Important Rule: Compatible Result Shapes

For a set operation, the participating queries must return compatible columns.

Typical requirements:

1. Same number of columns.
2. Corresponding columns must have compatible data types.
3. Columns are matched **by position**, not by column name.

Example:

```sql
SELECT employee_id, name
FROM employees

UNION

SELECT student_id, student_name
FROM students;
```

This is conceptually valid if the corresponding data types are compatible.

The names do not need to match.

The output column names normally come from the first `SELECT`.

---

## 2.1 Same Number of Columns

This is invalid:

```sql
SELECT id, name
FROM employees

UNION

SELECT id
FROM students;
```

The first query returns 2 columns and the second returns 1.

---

## 2.2 Compatible Data Types

This is generally valid when the database can reconcile the types:

```sql
SELECT employee_id
FROM employees

UNION

SELECT customer_id
FROM customers;
```

Both are identifiers and can commonly be represented using compatible numeric types.

But blindly combining unrelated types is not good SQL practice.

---

# 3. UNION

## Definition

`UNION` combines the results of two queries and **removes duplicate rows**.

```sql
SELECT column1, column2
FROM table1

UNION

SELECT column1, column2
FROM table2;
```

Think:

```text
UNION = combine + DISTINCT
```

---

## 3.1 Example

Table A:

| id |
|---:|
| 1 |
| 2 |
| 3 |

Table B:

| id |
|---:|
| 3 |
| 4 |
| 5 |

Query:

```sql
SELECT id
FROM A

UNION

SELECT id
FROM B;
```

Result:

| id |
|---:|
| 1 |
| 2 |
| 3 |
| 4 |
| 5 |

The duplicate `3` appears only once.

---

## 3.2 UNION Is Not "Append Everything"

A common mistake is to think:

```sql
UNION
```

simply appends rows.

It does not.

It combines results and eliminates duplicate rows.

If duplicate preservation is required, use:

```sql
UNION ALL
```

---

# 4. UNION ALL

## Definition

`UNION ALL` combines query results **without removing duplicates**.

```sql
SELECT column1, column2
FROM table1

UNION ALL

SELECT column1, column2
FROM table2;
```

Think:

```text
UNION ALL = append all rows
```

---

## 4.1 Example

Table A:

| id |
|---:|
| 1 |
| 2 |
| 3 |

Table B:

| id |
|---:|
| 3 |
| 4 |
| 5 |

Query:

```sql
SELECT id
FROM A

UNION ALL

SELECT id
FROM B;
```

Result:

| id |
|---:|
| 1 |
| 2 |
| 3 |
| 3 |
| 4 |
| 5 |

The duplicate `3` remains.

---

# 5. UNION vs UNION ALL — Master This Properly

This is one of the most important interview distinctions in this topic.

| Property | `UNION` | `UNION ALL` |
|---|---|---|
| Combines results | Yes | Yes |
| Removes duplicate rows | Yes | No |
| Preserves duplicate rows | No | Yes |
| Usually requires duplicate-elimination work | Yes | No |
| Usually faster | No | Yes |
| Use when unique combined results are required | Yes | No |
| Use when every row must be retained | No | Yes |

### Example

A:

```text
10
20
30
```

B:

```text
20
30
40
```

`UNION`:

```text
10
20
30
40
```

`UNION ALL`:

```text
10
20
30
20
30
40
```

---

## 5.1 Performance Difference

Suppose:

- Query A returns 10 million rows.
- Query B returns 10 million rows.

With:

```sql
UNION
```

the database must produce the combined result while eliminating duplicates.

This may require operations such as:

- sorting,
- hashing,
- duplicate elimination,
- additional memory,
- additional temporary work.

With:

```sql
UNION ALL
```

the database can generally append/concatenate the rows without global duplicate elimination.

Therefore:

> If duplicate removal is not required, prefer `UNION ALL`.

Do not choose `UNION` merely because it "looks safer."

---

# 6. Duplicate Means Duplicate Entire Row

`UNION` removes duplicate **result rows**, not duplicate values in an individual column.

Example:

```sql
SELECT department_id, employee_name
FROM employees
```

If the result is:

| department_id | employee_name |
|---:|---|
| 10 | Ravi |
| 10 | Ravi |
| 10 | Arun |

then `UNION` can eliminate the identical `(10, Ravi)` row.

But:

| department_id | employee_name |
|---:|---|
| 10 | Ravi |
| 20 | Ravi |

contains two different rows.

The repeated `employee_name` does not make them duplicates.

---

# 7. INTERSECT

## Definition

`INTERSECT` returns rows that appear in **both** query results.

```sql
SELECT column1
FROM A

INTERSECT

SELECT column1
FROM B;
```

Conceptually:

```text
A ∩ B
```

---

## 7.1 Example

A:

```text
1
2
3
4
```

B:

```text
3
4
5
6
```

Result:

```text
3
4
```

---

## 7.2 Duplicates

`INTERSECT` is normally duplicate-eliminating.

If:

A:

```text
1
1
2
3
```

B:

```text
1
1
1
3
```

the ordinary set-style result is:

```text
1
3
```

Do not confuse this with bag/multiset operations.

---

# 8. EXCEPT

## Definition

`EXCEPT` returns rows that occur in the first query but **not** in the second query.

```sql
SELECT column1
FROM A

EXCEPT

SELECT column1
FROM B;
```

Conceptually:

```text
A - B
```

---

## 8.1 Example

A:

```text
1
2
3
4
```

B:

```text
3
4
5
6
```

Result:

```text
1
2
```

The order matters.

```sql
A EXCEPT B
```

is not equivalent to:

```sql
B EXCEPT A
```

---

# 9. Set Operation Mental Model

Given:

```text
A = {1, 2, 3, 4}
B = {3, 4, 5, 6}
```

| Operation | Result |
|---|---|
| `A UNION B` | `{1,2,3,4,5,6}` |
| `A UNION ALL B` | `{1,2,3,4,3,4,5,6}` |
| `A INTERSECT B` | `{3,4}` |
| `A EXCEPT B` | `{1,2}` |
| `B EXCEPT A` | `{5,6}` |

Memorize:

```text
UNION       → A OR B
INTERSECT   → A AND B
EXCEPT      → A NOT IN B
```

---

# 10. Set Operations vs JOINs

This distinction is important.

## Set Operation

Combines result sets vertically:

```text
A
rows

+

B
rows
```

Example:

```sql
SELECT name FROM employees
UNION
SELECT name FROM contractors;
```

Result has one column and rows from both sources.

---

## JOIN

Combines related rows horizontally:

```text
A columns + B columns
```

Example:

```sql
SELECT e.name, d.department_name
FROM employees e
JOIN departments d
  ON e.department_id = d.department_id;
```

Result contains columns from both tables.

### Mental model

```text
UNION → vertical combination
JOIN  → horizontal combination
```

---

# 11. UNION and DISTINCT

A useful equivalence is:

```sql
SELECT x
FROM A

UNION

SELECT x
FROM B;
```

is conceptually similar to:

```sql
SELECT DISTINCT x
FROM (
    SELECT x FROM A
    UNION ALL
    SELECT x FROM B
) t;
```

The second formulation explicitly shows the idea:

```text
UNION ALL
    ↓
combine everything
    ↓
DISTINCT
    ↓
remove duplicates
```

This is a mental model, not a requirement that the database execute exactly those steps.

---

# 12. ORDER BY with Set Operations

A common mistake is placing independent `ORDER BY` clauses in each query.

Usually, when ordering the final combined result, place `ORDER BY` at the end:

```sql
SELECT id
FROM A

UNION ALL

SELECT id
FROM B

ORDER BY id;
```

The final `ORDER BY` orders the complete result.

---

## 12.1 Why This Matters

The set operation produces one result set.

If you want to order that final result, sort the final result:

```text
Query A
   +
Query B
   ↓
UNION ALL
   ↓
combined result
   ↓
ORDER BY
```

---

# 13. LIMIT / FETCH with Set Operations

If limiting the final result, be careful about scope.

Example:

```sql
(
    SELECT id
    FROM A
)
UNION ALL
(
    SELECT id
    FROM B
)
ORDER BY id
LIMIT 10;
```

Exact syntax varies across database systems.

If a `LIMIT` belongs to an individual branch rather than the final result, use parentheses where supported/required.

---

# 14. Column Names in Set Operations

Consider:

```sql
SELECT employee_id AS id, name AS person
FROM employees

UNION ALL

SELECT customer_id AS customer_id, customer_name AS customer
FROM customers;
```

The final result's column names generally come from the first query:

```text
id
person
```

The aliases in the second query do not normally rename the final output columns.

This is a common interview trap.

---

# 15. Column Matching Is Positional

This is critical.

Suppose:

```sql
SELECT employee_id, department_id
FROM employees

UNION ALL

SELECT department_id, employee_id
FROM departments;
```

The database does not match:

```text
employee_id ↔ employee_id
department_id ↔ department_id
```

by name.

It combines:

```text
first column ↔ first column
second column ↔ second column
```

Therefore, the semantic meaning of corresponding positions must be correct.

---

# 16. NULL and Set Operations

`NULL` requires special attention.

In duplicate elimination and set comparison, SQL engines treat rows according to set-operation semantics rather than ordinary `WHERE column = NULL` logic.

Do not reason about:

```sql
NULL = NULL
```

as though it were ordinary boolean equality.

For set operations, focus on the operator's row-set semantics.

Example:

```sql
SELECT NULL
UNION
SELECT NULL;
```

The duplicate result is eliminated under ordinary `UNION` semantics, leaving one row.

---

# 17. Duplicates and UNION ALL: Important Counting Pattern

Suppose:

```text
A has 100 rows
B has 80 rows
```

Then:

```sql
A UNION ALL B
```

returns:

```text
180 rows
```

assuming both queries execute successfully and all rows are included.

But:

```sql
A UNION B
```

returns:

```text
180 - number_of_duplicate_rows_removed
```

Therefore, you cannot determine the `UNION` row count merely by adding the input row counts.

---

# 18. Set Operations and Bag Semantics

SQL tables and query results commonly behave like **bags/multisets**, meaning duplicates can exist.

This explains why SQL provides both:

```sql
UNION
```

and:

```sql
UNION ALL
```

Think:

```text
UNION
    → set-like result
    → duplicate elimination

UNION ALL
    → bag/multiset-style combination
    → duplicates preserved
```

This distinction is important in database theory and GATE questions.

---

# 19. Common Interview Pattern: Combine Two Populations

Suppose you have:

```text
employees
contractors
```

and want all people appearing in either table.

If each person should appear only once:

```sql
SELECT email
FROM employees

UNION

SELECT email
FROM contractors;
```

If every source record must be retained:

```sql
SELECT email
FROM employees

UNION ALL

SELECT email
FROM contractors;
```

The correct choice depends on the requirement.

---

# 20. Common Interview Pattern: Common IDs

Find IDs present in both tables:

```sql
SELECT customer_id
FROM current_customers

INTERSECT

SELECT customer_id
FROM premium_customers;
```

Conceptually:

```text
current ∩ premium
```

An alternative using `JOIN` may be possible, but the set-operation formulation directly expresses the requirement.

---

# 21. Common Interview Pattern: IDs Missing from Another Set

Find IDs in A but not B:

```sql
SELECT user_id
FROM registered_users

EXCEPT

SELECT user_id
FROM banned_users;
```

Conceptually:

```text
registered - banned
```

In MySQL, note that `EXCEPT` support depends on the MySQL version; older versions require an alternative such as `NOT EXISTS` or an anti-join.

---

# 22. EXCEPT vs NOT EXISTS

These can express similar logic.

Set operation:

```sql
SELECT user_id
FROM A

EXCEPT

SELECT user_id
FROM B;
```

Alternative:

```sql
SELECT a.user_id
FROM A a
WHERE NOT EXISTS (
    SELECT 1
    FROM B b
    WHERE b.user_id = a.user_id
);
```

But do not assume they are always interchangeable in every detail.

Consider:

- duplicate behavior,
- NULL semantics,
- extra columns,
- optimizer behavior,
- database dialect.

---

# 23. INTERSECT vs EXISTS

Similarly:

```sql
SELECT user_id
FROM A

INTERSECT

SELECT user_id
FROM B;
```

can often be expressed as:

```sql
SELECT a.user_id
FROM A a
WHERE EXISTS (
    SELECT 1
    FROM B b
    WHERE b.user_id = a.user_id
);
```

Again, duplicate semantics can differ unless explicitly controlled.

---

# 24. Set Operations vs DISTINCT

These are different concepts.

```sql
SELECT DISTINCT department_id
FROM employees;
```

removes duplicates **inside one query result**.

```sql
SELECT department_id
FROM employees

UNION

SELECT department_id
FROM contractors;
```

combines two result sets and removes duplicate rows across the combined result.

---

# 25. Operator Precedence and Parentheses

When multiple set operators are combined, use parentheses when you need to make the intended grouping explicit.

Example:

```sql
(
    SELECT id FROM A
    UNION
    SELECT id FROM B
)
EXCEPT
(
    SELECT id FROM C
);
```

This makes the intended operation clear:

```text
(A ∪ B) − C
```

Do not rely on memory of dialect-specific precedence rules when parentheses can remove ambiguity.

---

# 26. Practical Examples

## Example 1 — All employees and contractors

```sql
SELECT email
FROM employees

UNION

SELECT email
FROM contractors;
```

Use `UNION` when an email should appear only once.

---

## Example 2 — Every record from both sources

```sql
SELECT email
FROM employees

UNION ALL

SELECT email
FROM contractors;
```

Use `UNION ALL` when duplicate source records matter.

---

## Example 3 — Users in both systems

```sql
SELECT user_id
FROM app_users

INTERSECT

SELECT user_id
FROM website_users;
```

---

## Example 4 — Users only in the app

```sql
SELECT user_id
FROM app_users

EXCEPT

SELECT user_id
FROM website_users;
```

---

# 27. A Very Important Business Example

Suppose an organization stores customers from two systems:

```text
crm_customers
marketing_customers
```

The same customer can occur in both.

Requirement:

> "Give me the unique customers reachable by either system."

Use:

```sql
SELECT customer_id
FROM crm_customers

UNION

SELECT customer_id
FROM marketing_customers;
```

Requirement:

> "Give me every source record because I need to count source activity."

Use:

```sql
SELECT customer_id
FROM crm_customers

UNION ALL

SELECT customer_id
FROM marketing_customers;
```

This is the central practical distinction.

---

# 28. The Classic Interview Trap

Question:

> Which is generally faster, `UNION` or `UNION ALL`?

Answer:

> `UNION ALL` is generally faster because it does not need to eliminate duplicates.

But the more important answer is:

> Use `UNION` when duplicate removal is required; otherwise use `UNION ALL`.

Do not optimize by changing semantics.

---

# 29. Another Classic Trap

Question:

```sql
SELECT 1
UNION
SELECT 1
UNION ALL
SELECT 1;
```

Do not casually answer without evaluating the operations and their grouping according to the SQL dialect.

When a question mixes operators, use parentheses if the intended grouping is known:

```sql
(
    SELECT 1
    UNION
    SELECT 1
)
UNION ALL
SELECT 1;
```

First:

```text
1 UNION 1 → 1
```

Then:

```text
1 UNION ALL 1 → 1, 1
```

Result:

```text
1
1
```

For interview/GATE questions involving multiple set operators, carefully determine the specified precedence or use explicit parentheses.

---

# 30. GATE / Database Theory Concepts

## 30.1 Set vs Bag

Relational algebra traditionally reasons about sets:

```text
no duplicate tuples
```

SQL commonly allows duplicates:

```text
duplicate rows can exist
```

Therefore:

```text
UNION      → duplicate-eliminating
UNION ALL  → duplicate-preserving
```

This distinction is frequently tested conceptually.

---

## 30.2 Closure

Set operations combine compatible relations/results into another relation/result.

For union-like operations, the participating relations need compatible schemas.

This is often called **union compatibility**.

---

## 30.3 Union Compatibility

Two relations are union-compatible when their attributes correspond appropriately in:

- number,
- domains/types,
- positional correspondence.

Modern SQL expresses compatibility through compatible query result columns.

---

# 31. GATE-Style Solved Questions

> These are **GATE-style practice questions**, not claims of official GATE PYQs.

## Question 1 — UNION vs UNION ALL

Given:

```text
A = {1, 2, 3}
B = {3, 4, 5}
```

What are the number of rows produced by:

```text
A UNION B
```

and:

```text
A UNION ALL B
```

### Solution

`UNION` removes duplicate `3`.

```text
A UNION B = {1,2,3,4,5}
```

Rows:

```text
5
```

`UNION ALL` preserves both occurrences of `3`.

```text
A UNION ALL B = {1,2,3,3,4,5}
```

Rows:

```text
6
```

### Answer

```text
UNION     → 5
UNION ALL → 6
```

---

## Question 2 — Column Compatibility

Consider:

```sql
SELECT employee_id, salary
FROM employees

UNION

SELECT customer_id
FROM customers;
```

Will this query generally be valid?

### Solution

No.

First query:

```text
2 columns
```

Second query:

```text
1 column
```

Set operations require compatible result shapes.

### Answer

**Invalid because the number of columns differs.**

---

## Question 3 — Positional Matching

Consider:

```sql
SELECT employee_id, department_id
FROM employees

UNION ALL

SELECT department_id, employee_id
FROM departments;
```

How are columns matched?

### Solution

Set operations match corresponding columns by **position**:

```text
first ↔ first
second ↔ second
```

They are not matched by alias or column name.

### Answer

The first column of the first query is combined with the first column of the second query.

---

## Question 4 — INTERSECT

Given:

```text
A = {1, 2, 2, 3, 4}
B = {2, 2, 4, 5}
```

What does ordinary `INTERSECT` return?

### Solution

The values occurring in both:

```text
2
4
```

Ordinary `INTERSECT` eliminates duplicates.

### Answer

```text
{2, 4}
```

---

## Question 5 — EXCEPT Direction

Given:

```text
A = {1,2,3,4}
B = {3,4,5}
```

Find:

```text
A EXCEPT B
```

### Solution

Keep rows from A that are absent from B:

```text
1, 2
```

Therefore:

```text
A EXCEPT B = {1,2}
```

But:

```text
B EXCEPT A = {5}
```

### Answer

```text
{1,2}
```

---

## Question 6 — Row Count

Query A produces 1,000 rows.

Query B produces 500 rows.

There are 100 duplicate rows across the two results.

What is the row count of:

```sql
A UNION ALL B
```

and, assuming the 100 overlapping rows are duplicate result rows:

```sql
A UNION B
```

### Solution

`UNION ALL` preserves everything:

```text
1000 + 500 = 1500
```

`UNION` removes the 100 duplicate rows:

```text
1500 - 100 = 1400
```

### Answer

```text
UNION ALL → 1500
UNION     → 1400
```

---

# 32. Interview Practice Questions

Try these without looking at the answers.

### Q1
What is the difference between `UNION` and `UNION ALL`?

### Q2
Why is `UNION ALL` generally faster?

### Q3
Can two queries in a set operation have different column names?

### Q4
How are columns matched in a set operation?

### Q5
What does `INTERSECT` return?

### Q6
What does `EXCEPT` return?

### Q7
Is `A EXCEPT B` the same as `B EXCEPT A`?

### Q8
What is the difference between a `JOIN` and a `UNION`?

### Q9
Where should a final `ORDER BY` normally be placed?

### Q10
What happens to duplicate rows in `UNION`?

### Q11
What happens to duplicate rows in `UNION ALL`?

### Q12
Why can changing `UNION ALL` to `UNION` be a correctness bug?

---

# 33. Interview Answers

## Q1

`UNION` removes duplicate result rows; `UNION ALL` preserves them.

## Q2

`UNION ALL` generally avoids the additional work needed for duplicate elimination.

## Q3

Yes, column names do not need to match. Corresponding columns must be compatible, and final names generally come from the first query.

## Q4

By position.

## Q5

Rows common to both result sets.

## Q6

Rows in the first result set but not the second.

## Q7

No. `EXCEPT` is directional.

## Q8

`UNION` combines rows vertically; `JOIN` combines related columns horizontally.

## Q9

Normally at the end of the complete set expression when ordering the final combined result.

## Q10

Duplicates are removed.

## Q11

Duplicates are preserved.

## Q12

Because duplicate rows may be meaningful. Removing them changes the query's semantics.

---

# 34. Common Mistakes

## Mistake 1 — Thinking UNION ALL removes duplicates

Wrong:

```text
UNION ALL → unique rows
```

Correct:

```text
UNION ALL → all rows
```

---

## Mistake 2 — Assuming UNION and UNION ALL have identical performance

They can have materially different execution costs because `UNION` requires duplicate elimination.

---

## Mistake 3 — Matching columns by name

Wrong mental model:

```text
employee_id ↔ employee_id
```

Correct:

```text
position 1 ↔ position 1
position 2 ↔ position 2
```

---

## Mistake 4 — Confusing UNION with JOIN

```text
UNION → vertical
JOIN  → horizontal
```

---

## Mistake 5 — Forgetting duplicate semantics

Changing:

```sql
UNION ALL
```

to:

```sql
UNION
```

can change the answer.

---

## Mistake 6 — Reversing EXCEPT

```text
A EXCEPT B ≠ B EXCEPT A
```

---

## Mistake 7 — Ignoring column compatibility

Set operations do not require identical table schemas, but the selected result columns must be compatible.

---

## Mistake 8 — Assuming set-operation syntax is identical across every DBMS

`UNION` and `UNION ALL` are broadly supported.

`INTERSECT` and `EXCEPT` support/syntax can vary by database/version.

Always know the dialect being tested.

---

# 35. Pattern Recognition

When you see:

### "All records from A and B"

Ask:

```text
Do duplicates matter?
```

If no:

```sql
UNION
```

If yes:

```sql
UNION ALL
```

---

### "Present in both"

Think:

```sql
INTERSECT
```

or an equivalent `JOIN`/`EXISTS` formulation.

---

### "Present in A but not B"

Think:

```sql
EXCEPT
```

or:

```sql
NOT EXISTS
```

---

### "Combine rows from two queries"

Think:

```sql
UNION / UNION ALL
```

not `JOIN`.

---

# 36. Decision Tree

```text
Need to combine two result sets?
            |
            v
       Same shape?
        /       \
      No         Yes
      |           |
   Invalid        v
             What do you need?
             /      |       \
            /       |        \
         A or B   Both       A only
           |        |           |
           v        v           v
        UNION    INTERSECT    EXCEPT
           |
     Are duplicates
       required?
       /       \
     No         Yes
     |           |
   UNION      UNION ALL
```

---

# 37. SQL Dialect Note

The core concepts are portable, but exact support differs.

Examples:

- PostgreSQL supports `UNION`, `UNION ALL`, `INTERSECT`, and `EXCEPT`.
- SQL Server supports these set operators with dialect-specific details.
- Oracle supports set operations, with `MINUS` traditionally serving the role of `EXCEPT`.
- MySQL supports `UNION`/`UNION ALL`; support for `INTERSECT` and `EXCEPT` depends on the MySQL version.

For interview preparation, learn the standard concepts first, then the syntax of the database used by the company or course.

---

# 38. Complexity / Execution Intuition

Do not assign a universal Big-O complexity to SQL set operations without knowing:

- input sizes,
- indexes,
- execution plan,
- sorting/hashing strategy,
- memory,
- database engine.

A useful interview-level intuition is:

```text
UNION ALL
→ concatenate/append
→ no duplicate elimination

UNION
→ combine
→ eliminate duplicates
→ potentially sort/hash
```

Therefore, `UNION ALL` is generally cheaper.

---

# 39. Quick Comparison Sheet

| Operation | Mathematical idea | Duplicate handling | Direction |
|---|---|---|---|
| `UNION` | A ∪ B | Removes | Symmetric |
| `UNION ALL` | Bag concatenation | Preserves | Symmetric in row inclusion |
| `INTERSECT` | A ∩ B | Removes | Symmetric |
| `EXCEPT` | A − B | Removes | Directional |

---

# 40. Must-Memorize Facts

```text
UNION
→ combine + remove duplicates

UNION ALL
→ combine + preserve duplicates

INTERSECT
→ common rows

EXCEPT
→ rows in first query but not second

Set operations
→ combine rows vertically

JOIN
→ combine columns horizontally

Columns
→ matched by position

Output column names
→ generally taken from first SELECT

UNION ALL
→ generally faster than UNION

EXCEPT
→ order matters
```

---

# 41. Mastery Checklist

You should be able to answer all of these immediately:

- [ ] Define a SQL set operation.
- [ ] Explain `UNION`.
- [ ] Explain `UNION ALL`.
- [ ] Explain `INTERSECT`.
- [ ] Explain `EXCEPT`.
- [ ] State the exact difference between `UNION` and `UNION ALL`.
- [ ] Explain why `UNION ALL` is generally faster.
- [ ] Determine output rows with duplicates.
- [ ] Explain union compatibility.
- [ ] Explain positional column matching.
- [ ] Explain output column naming.
- [ ] Distinguish set operations from joins.
- [ ] Use `INTERSECT` for common rows.
- [ ] Use `EXCEPT` for A-but-not-B.
- [ ] Understand why `EXCEPT` is directional.
- [ ] Place final `ORDER BY` correctly.
- [ ] Handle multiple set operators with explicit parentheses when needed.
- [ ] Recognize dialect differences.
- [ ] Convert simple `EXCEPT` logic to `NOT EXISTS`.
- [ ] Convert simple `INTERSECT` logic to `EXISTS`.
- [ ] Explain duplicate semantics in interview terms.

---

# 42. Final Revision Sheet

## One-line definitions

```text
UNION       = combine two results and remove duplicates
UNION ALL   = combine two results and keep duplicates
INTERSECT   = rows present in both results
EXCEPT      = rows present in first result but absent from second
```

## Most important comparison

```text
A = {1,2,3}
B = {3,4}

A UNION B
→ {1,2,3,4}

A UNION ALL B
→ {1,2,3,3,4}
```

## Compatibility

```text
Same number of columns
+
Compatible corresponding data types
+
Columns matched by position
```

## Mental model

```text
UNION       → OR
INTERSECT   → AND
EXCEPT      → A but not B
```

## Interview rule

> If duplicates must be preserved, use `UNION ALL`. If duplicates must be removed, use `UNION`.

## Performance rule

> If duplicate elimination is unnecessary, `UNION ALL` is generally preferable.

---

# 43. LeetCode / Interview Relevance

Set operations appear less frequently than joins, aggregation, subqueries, and window functions in many SQL interview sets, but they are useful for:

- combining populations,
- comparing datasets,
- finding common entities,
- finding entities exclusive to one dataset,
- multi-source reporting,
- data reconciliation,
- duplicate-aware pipelines.

The most important interview skill is not memorizing syntax. It is translating the requirement:

```text
either → UNION
either, duplicates matter → UNION ALL
both → INTERSECT
A but not B → EXCEPT / NOT EXISTS
```

---

# 44. Final Takeaway

The central distinction of this topic is:

```text
UNION
    ↓
duplicates removed

UNION ALL
    ↓
duplicates retained
```

Do not use `UNION` and `UNION ALL` interchangeably.

If the business requirement says **"all rows"**, `UNION ALL` is usually the correct starting point.

If the requirement says **"unique rows"**, `UNION` is appropriate.

For:

```text
common rows
```

think:

```sql
INTERSECT
```

For:

```text
A but not B
```

think:

```sql
EXCEPT
```

And always remember:

```text
Set operations combine rows.
JOINs combine columns.
```
