# SQL Topic 07 — CASE Expressions

> **Overall repository topic:** 30  
> **SQL topic number:** 7  
> **Difficulty focus:** Easy → Medium → Hard, with Medium dominant  
> **Goal:** Master `CASE WHEN`, categorization, conditional aggregation, conditional counting, and conditional sums.

---

# 1. What Is CASE?

`CASE` is SQL's conditional-expression mechanism.

It lets you return different values depending on whether conditions are satisfied.

Basic form:

```sql
CASE
    WHEN condition1 THEN result1
    WHEN condition2 THEN result2
    ELSE result_default
END
```

Think of it as:

```text
if condition1:
    result1
else if condition2:
    result2
else:
    result_default
```

Unlike procedural `IF` statements, `CASE` is an **expression** that produces a value.

Therefore it can appear in places such as:

- `SELECT`
- `ORDER BY`
- `GROUP BY`
- `WHERE` through an expression/subquery
- aggregate expressions
- window-function expressions
- `HAVING`

---

# 2. Why CASE Matters in SQL Interviews

`CASE` is heavily used for:

- categorizing rows,
- conditional counting,
- conditional sums,
- conditional averages,
- custom sorting,
- bucketing numeric values,
- replacing coded values with labels,
- conditional aggregation,
- business rules.

A large number of SQL interview problems reduce to:

```text
condition
    ↓
CASE
    ↓
aggregate
```

For example:

```sql
SUM(
    CASE
        WHEN status = 'completed' THEN amount
        ELSE 0
    END
)
```

This is one of the most important SQL patterns to master.

---

# 3. Searched CASE — CASE WHEN

The most flexible form is the searched `CASE`:

```sql
CASE
    WHEN condition THEN result
    WHEN condition THEN result
    ELSE result
END
```

Example:

```sql
SELECT
    employee_id,
    salary,
    CASE
        WHEN salary >= 100000 THEN 'High'
        WHEN salary >= 50000 THEN 'Medium'
        ELSE 'Low'
    END AS salary_band
FROM employees;
```

Possible result:

| employee_id | salary | salary_band |
|---:|---:|---|
| 1 | 120000 | High |
| 2 | 75000 | Medium |
| 3 | 40000 | Low |

---

# 4. CASE Is Evaluated Top to Bottom

This is critical.

Consider:

```sql
CASE
    WHEN salary >= 50000 THEN 'Medium or High'
    WHEN salary >= 100000 THEN 'High'
    ELSE 'Low'
END
```

A salary of `120000` satisfies both conditions.

But SQL reaches:

```sql
salary >= 50000
```

first.

Therefore it returns:

```text
Medium or High
```

and does not continue to the later condition.

So:

> Put more specific conditions before broader conditions.

Correct:

```sql
CASE
    WHEN salary >= 100000 THEN 'High'
    WHEN salary >= 50000 THEN 'Medium'
    ELSE 'Low'
END
```

---

# 5. ELSE

`ELSE` specifies the result when no `WHEN` condition matches.

```sql
CASE
    WHEN score >= 90 THEN 'A'
    WHEN score >= 75 THEN 'B'
    ELSE 'C'
END
```

Without `ELSE`, if no condition matches, the result is generally:

```text
NULL
```

Example:

```sql
CASE
    WHEN status = 'paid' THEN 'Completed'
END
```

For any status other than `'paid'`, the expression evaluates to `NULL`.

---

# 6. Always Think About the ELSE Case

Consider:

```sql
CASE
    WHEN status = 'completed' THEN 1
END
```

This produces:

```text
1     for completed
NULL  otherwise
```

That behavior can actually be useful for conditional counting:

```sql
COUNT(
    CASE
        WHEN status = 'completed' THEN 1
    END
)
```

Only the rows where the condition is true produce `1`; other rows produce `NULL`, and `COUNT(expression)` does not count NULL.

---

# 7. CASE Result Types

All branches of a `CASE` should produce compatible values.

Good:

```sql
CASE
    WHEN score >= 50 THEN 'Pass'
    ELSE 'Fail'
END
```

Potentially problematic:

```sql
CASE
    WHEN score >= 50 THEN 'Pass'
    ELSE 0
END
```

The exact behavior depends on the database's type-conversion rules.

Prefer consistent result types:

```sql
CASE
    WHEN score >= 50 THEN 'Pass'
    ELSE 'Fail'
END
```

or:

```sql
CASE
    WHEN score >= 50 THEN 1
    ELSE 0
END
```

---

# 8. Simple CASE

SQL also supports a simple `CASE` expression:

```sql
CASE expression
    WHEN value1 THEN result1
    WHEN value2 THEN result2
    ELSE result_default
END
```

Example:

```sql
SELECT
    employee_id,
    CASE department_id
        WHEN 10 THEN 'Engineering'
        WHEN 20 THEN 'Sales'
        WHEN 30 THEN 'HR'
        ELSE 'Other'
    END AS department_name
FROM employees;
```

This is useful for equality-based matching.

---

# 9. Searched CASE vs Simple CASE

| Feature | Searched CASE | Simple CASE |
|---|---|---|
| Syntax | `CASE WHEN condition` | `CASE expression WHEN value` |
| Complex conditions | Yes | No |
| Range conditions | Yes | No |
| Multiple logical conditions | Yes | Limited |
| Best for | General business rules | Equality mapping |

Example searched:

```sql
CASE
    WHEN salary >= 100000 THEN 'High'
    WHEN salary >= 50000 THEN 'Medium'
    ELSE 'Low'
END
```

Example simple:

```sql
CASE department_id
    WHEN 10 THEN 'Engineering'
    WHEN 20 THEN 'Sales'
    ELSE 'Other'
END
```

For interview problems, the searched form is usually more important.

---

# 10. Categorization

One of the most common uses of `CASE` is converting continuous values into categories.

Example:

```sql
SELECT
    customer_id,
    age,
    CASE
        WHEN age < 18 THEN 'Minor'
        WHEN age < 30 THEN 'Young Adult'
        WHEN age < 60 THEN 'Adult'
        ELSE 'Senior'
    END AS age_group
FROM customers;
```

The important detail is that the boundaries must be designed correctly.

Because conditions are checked top-to-bottom:

```text
age < 18
age < 30
age < 60
else
```

these represent:

```text
< 18
18–29
30–59
60+
```

---

# 11. Numeric Bucketing

Example:

```sql
SELECT
    order_id,
    amount,
    CASE
        WHEN amount < 1000 THEN 'Small'
        WHEN amount < 5000 THEN 'Medium'
        WHEN amount < 10000 THEN 'Large'
        ELSE 'Very Large'
    END AS order_size
FROM orders;
```

This is often called:

- categorization,
- bucketing,
- banding,
- segmentation.

---

# 12. Categorization with GROUP BY

Suppose you want the number of employees in each salary band.

You can group by the same `CASE` expression:

```sql
SELECT
    CASE
        WHEN salary < 50000 THEN 'Low'
        WHEN salary < 100000 THEN 'Medium'
        ELSE 'High'
    END AS salary_band,
    COUNT(*) AS employee_count
FROM employees
GROUP BY
    CASE
        WHEN salary < 50000 THEN 'Low'
        WHEN salary < 100000 THEN 'Medium'
        ELSE 'High'
    END;
```

Some database systems allow grouping by the alias:

```sql
GROUP BY salary_band
```

but dialect support varies.

For portable SQL, repeating the expression or using a CTE/subquery is safer.

---

# 13. CASE in a CTE

A clean alternative is:

```sql
WITH classified AS (
    SELECT
        employee_id,
        salary,
        CASE
            WHEN salary < 50000 THEN 'Low'
            WHEN salary < 100000 THEN 'Medium'
            ELSE 'High'
        END AS salary_band
    FROM employees
)
SELECT
    salary_band,
    COUNT(*) AS employee_count
FROM classified
GROUP BY salary_band;
```

This avoids repeating a long `CASE` expression.

---

# 14. Conditional Aggregation

## Definition

Conditional aggregation means applying an aggregate function only to rows satisfying a condition.

The standard SQL pattern is:

```sql
SUM(
    CASE
        WHEN condition THEN value
        ELSE 0
    END
)
```

or:

```sql
COUNT(
    CASE
        WHEN condition THEN 1
    END
)
```

This is extremely important for SQL interviews.

---

# 15. Conditional Counting

Suppose:

```text
orders
```

contains:

```text
order_id
status
```

Count completed orders:

```sql
SELECT
    COUNT(
        CASE
            WHEN status = 'completed' THEN 1
        END
    ) AS completed_orders
FROM orders;
```

Why does this work?

For completed rows:

```text
CASE → 1
```

For other rows:

```text
CASE → NULL
```

`COUNT(expression)` counts non-NULL values.

Therefore only completed rows are counted.

---

# 16. Conditional COUNT with ELSE

You could write:

```sql
COUNT(
    CASE
        WHEN status = 'completed' THEN 1
        ELSE 0
    END
)
```

But this is **wrong for conditional counting**.

Why?

`COUNT(expression)` counts both:

```text
1
0
```

because both are non-NULL.

Therefore it counts every row.

This is a classic interview trap.

Correct:

```sql
COUNT(
    CASE
        WHEN status = 'completed' THEN 1
    END
)
```

or:

```sql
SUM(
    CASE
        WHEN status = 'completed' THEN 1
        ELSE 0
    END
)
```

---

# 17. Conditional Counting with SUM

A very common pattern is:

```sql
SUM(
    CASE
        WHEN status = 'completed' THEN 1
        ELSE 0
    END
)
```

For each row:

```text
completed → 1
not completed → 0
```

Then `SUM` adds them.

If there are:

```text
1
0
1
0
1
```

the sum is:

```text
3
```

So:

```sql
SUM(CASE WHEN condition THEN 1 ELSE 0 END)
```

is a powerful general-purpose conditional-counting pattern.

---

# 18. COUNT vs SUM for Conditional Counting

Both can be correct:

### Pattern A

```sql
COUNT(
    CASE
        WHEN condition THEN 1
    END
)
```

### Pattern B

```sql
SUM(
    CASE
        WHEN condition THEN 1
        ELSE 0
    END
)
```

Mental model:

```text
COUNT(CASE ...)
→ count non-NULL matches

SUM(CASE ...)
→ add 1 for matches and 0 otherwise
```

The `SUM` form is often easier to extend to multiple conditions.

---

# 19. Conditional Sums

Suppose each order has:

```text
status
amount
```

Find total revenue from completed orders:

```sql
SELECT
    SUM(
        CASE
            WHEN status = 'completed' THEN amount
            ELSE 0
        END
    ) AS completed_revenue
FROM orders;
```

For each row:

```text
completed → amount
otherwise → 0
```

Then `SUM` adds the selected amounts.

---

# 20. Conditional Sums by Category

Example:

```sql
SELECT
    SUM(
        CASE
            WHEN category = 'Electronics' THEN amount
            ELSE 0
        END
    ) AS electronics_sales,

    SUM(
        CASE
            WHEN category = 'Clothing' THEN amount
            ELSE 0
        END
    ) AS clothing_sales
FROM orders;
```

Result:

| electronics_sales | clothing_sales |
|---:|---:|
| 250000 | 175000 |

This technique lets one query produce multiple conditional metrics.

---

# 21. Multiple Conditional Metrics in One Query

This is a very common interview pattern:

```sql
SELECT
    COUNT(*) AS total_orders,

    SUM(
        CASE
            WHEN status = 'completed' THEN 1
            ELSE 0
        END
    ) AS completed_orders,

    SUM(
        CASE
            WHEN status = 'cancelled' THEN 1
            ELSE 0
        END
    ) AS cancelled_orders,

    SUM(
        CASE
            WHEN status = 'completed' THEN amount
            ELSE 0
        END
    ) AS completed_revenue

FROM orders;
```

One table scan can conceptually produce:

```text
total orders
completed orders
cancelled orders
completed revenue
```

The actual optimizer/execution plan depends on the database.

---

# 22. Conditional Aggregation by Group

Suppose you want completed and cancelled orders for every customer:

```sql
SELECT
    customer_id,

    SUM(
        CASE
            WHEN status = 'completed' THEN 1
            ELSE 0
        END
    ) AS completed_orders,

    SUM(
        CASE
            WHEN status = 'cancelled' THEN 1
            ELSE 0
        END
    ) AS cancelled_orders

FROM orders
GROUP BY customer_id;
```

Result:

| customer_id | completed_orders | cancelled_orders |
|---:|---:|---:|
| 101 | 4 | 1 |
| 102 | 2 | 3 |
| 103 | 5 | 0 |

The `GROUP BY` determines the groups, while each `CASE` determines which rows contribute to each metric.

---

# 23. Conditional Aggregation vs WHERE

This distinction is essential.

Suppose you write:

```sql
SELECT COUNT(*)
FROM orders
WHERE status = 'completed';
```

This returns:

```text
number of completed orders
```

But:

```sql
SELECT
    COUNT(*) AS total_orders,
    SUM(
        CASE
            WHEN status = 'completed' THEN 1
            ELSE 0
        END
    ) AS completed_orders
FROM orders;
```

returns both:

```text
total orders
completed orders
```

A `WHERE` clause removes rows from the query.

A conditional aggregate keeps the rows in the query but controls which rows contribute to a particular metric.

---

# 24. Why WHERE Cannot Replace Multiple Conditional Metrics Easily

Suppose you want:

```text
total orders
completed orders
cancelled orders
pending orders
```

A single:

```sql
WHERE status = ...
```

cannot simultaneously keep all statuses for the total while filtering different statuses for separate metrics.

Conditional aggregation solves this:

```sql
SELECT
    COUNT(*) AS total,

    SUM(CASE WHEN status = 'completed' THEN 1 ELSE 0 END) AS completed,

    SUM(CASE WHEN status = 'cancelled' THEN 1 ELSE 0 END) AS cancelled,

    SUM(CASE WHEN status = 'pending' THEN 1 ELSE 0 END) AS pending

FROM orders;
```

---

# 25. Conditional Aggregation with Multiple Conditions

`CASE` can contain complex conditions:

```sql
SUM(
    CASE
        WHEN status = 'completed'
         AND amount >= 10000
        THEN amount
        ELSE 0
    END
) AS high_value_completed_revenue
```

You can combine:

- `AND`
- `OR`
- `IN`
- comparisons
- date conditions
- NULL checks

Example:

```sql
SUM(
    CASE
        WHEN status IN ('completed', 'shipped')
        THEN 1
        ELSE 0
    END
)
```

---

# 26. Conditional Average

The same technique works with `AVG`.

Example:

```sql
AVG(
    CASE
        WHEN status = 'completed' THEN amount
    END
)
```

Rows not satisfying the condition produce `NULL`.

`AVG` ignores NULL values.

Therefore this computes the average amount among completed orders.

Do **not** write:

```sql
AVG(
    CASE
        WHEN status = 'completed' THEN amount
        ELSE 0
    END
)
```

unless zero is genuinely intended to be part of the average.

Otherwise the non-completed rows will incorrectly lower the average.

---

# 27. Conditional MIN and MAX

The same principle can apply:

```sql
MAX(
    CASE
        WHEN status = 'completed' THEN amount
    END
)
```

This returns the maximum completed-order amount.

Similarly:

```sql
MIN(
    CASE
        WHEN status = 'completed' THEN amount
    END
)
```

returns the minimum completed-order amount.

---

# 28. Conditional Aggregation with DISTINCT

You may need a conditional count of unique users:

```sql
COUNT(
    DISTINCT CASE
        WHEN status = 'completed' THEN customer_id
    END
)
```

This means:

1. Keep `customer_id` for completed rows.
2. Produce `NULL` for other rows.
3. Apply `DISTINCT`.
4. Count the remaining non-NULL customer IDs.

This is a high-value interview pattern.

---

# 29. Example — Unique Active Customers

```sql
SELECT
    COUNT(
        DISTINCT CASE
            WHEN status = 'active' THEN customer_id
        END
    ) AS active_customers
FROM customers;
```

This counts unique customer IDs whose status is active.

---

# 30. Conditional Aggregation with Dates

Example:

```sql
SELECT
    SUM(
        CASE
            WHEN order_date >= DATE '2026-01-01'
             AND order_date < DATE '2026-02-01'
            THEN amount
            ELSE 0
        END
    ) AS january_revenue
FROM orders;
```

Half-open date intervals are often safer for timestamps:

```text
>= start
< next boundary
```

rather than relying on an inclusive end timestamp.

Exact date-literal syntax varies by DBMS.

---

# 31. CASE for Custom Ordering

`CASE` is useful in `ORDER BY`.

Suppose:

```text
priority:
High
Medium
Low
```

Alphabetical ordering is not the desired order.

Use:

```sql
SELECT *
FROM tickets
ORDER BY
    CASE priority
        WHEN 'High' THEN 1
        WHEN 'Medium' THEN 2
        WHEN 'Low' THEN 3
        ELSE 4
    END;
```

Result:

```text
High
Medium
Low
Other
```

---

# 32. CASE for NULL Categorization

Example:

```sql
SELECT
    customer_id,
    CASE
        WHEN phone_number IS NULL THEN 'Missing'
        ELSE 'Available'
    END AS phone_status
FROM customers;
```

Remember:

```sql
phone_number = NULL
```

is not the correct NULL test.

Use:

```sql
phone_number IS NULL
```

---

# 33. CASE and Three-Valued Logic

SQL conditions can evaluate to:

```text
TRUE
FALSE
UNKNOWN
```

For example:

```sql
salary > 50000
```

when `salary` is `NULL` evaluates to `UNKNOWN`.

A `WHEN` branch is taken only when its condition is true.

Therefore:

```sql
CASE
    WHEN salary > 50000 THEN 'High'
    ELSE 'Other'
END
```

places a NULL salary into:

```text
Other
```

unless another branch explicitly handles NULL.

---

# 34. Explicit NULL Handling

If NULL deserves its own category:

```sql
CASE
    WHEN salary IS NULL THEN 'Unknown'
    WHEN salary >= 100000 THEN 'High'
    WHEN salary >= 50000 THEN 'Medium'
    ELSE 'Low'
END
```

This is usually clearer than relying on the `ELSE` branch to absorb NULLs.

---

# 35. CASE and Boolean Conditions

Some database systems have boolean data types; others do not.

Portable conditional aggregation is:

```sql
SUM(
    CASE
        WHEN condition THEN 1
        ELSE 0
    END
)
```

Do not assume that:

```sql
SUM(condition)
```

works identically across database systems.

---

# 36. Conditional Counting: Three Important Patterns

## Pattern 1 — COUNT(CASE)

```sql
COUNT(
    CASE
        WHEN condition THEN 1
    END
)
```

Use when counting matching rows.

---

## Pattern 2 — SUM(CASE)

```sql
SUM(
    CASE
        WHEN condition THEN 1
        ELSE 0
    END
)
```

Use for conditional counts and when you want explicit 1/0 behavior.

---

## Pattern 3 — COUNT(DISTINCT CASE)

```sql
COUNT(
    DISTINCT CASE
        WHEN condition THEN customer_id
    END
)
```

Use for unique entities satisfying a condition.

---

# 37. Conditional Sum: Core Pattern

Memorize:

```sql
SUM(
    CASE
        WHEN condition THEN amount
        ELSE 0
    END
)
```

Example:

```sql
SELECT
    SUM(
        CASE
            WHEN payment_status = 'paid'
            THEN amount
            ELSE 0
        END
    ) AS paid_amount
FROM payments;
```

---

# 38. CASE + GROUP BY + Conditional Aggregation

A very common interview question:

> For each department, find the number of employees, number earning at least 100000, and total salary of employees earning at least 100000.

Solution:

```sql
SELECT
    department_id,

    COUNT(*) AS employee_count,

    SUM(
        CASE
            WHEN salary >= 100000 THEN 1
            ELSE 0
        END
    ) AS high_earners,

    SUM(
        CASE
            WHEN salary >= 100000 THEN salary
            ELSE 0
        END
    ) AS high_earner_salary

FROM employees
GROUP BY department_id;
```

This pattern is highly interview-relevant.

---

# 39. CASE + HAVING

You can aggregate conditionally and then filter groups.

Example:

> Find departments with at least 5 completed orders.

```sql
SELECT
    department_id,
    SUM(
        CASE
            WHEN status = 'completed' THEN 1
            ELSE 0
        END
    ) AS completed_orders
FROM orders
GROUP BY department_id
HAVING
    SUM(
        CASE
            WHEN status = 'completed' THEN 1
            ELSE 0
        END
    ) >= 5;
```

Conceptually:

```text
rows
 ↓
GROUP BY
 ↓
conditional aggregation
 ↓
HAVING
 ↓
selected groups
```

---

# 40. CASE + Window Functions

`CASE` can also be used inside a window expression.

Example:

```sql
SELECT
    employee_id,
    department_id,
    salary,

    SUM(
        CASE
            WHEN salary >= 100000 THEN 1
            ELSE 0
        END
    ) OVER (
        PARTITION BY department_id
    ) AS high_earners_in_department

FROM employees;
```

This gives every employee the count of high earners in their department.

Window functions are covered more deeply in later SQL topics.

---

# 41. CASE in UPDATE

`CASE` can also generate conditional update values.

Example:

```sql
UPDATE employees
SET bonus =
    CASE
        WHEN performance_score >= 90 THEN 10000
        WHEN performance_score >= 75 THEN 5000
        ELSE 0
    END;
```

Use caution with updates: this changes stored data.

---

# 42. CASE in SELECT vs WHERE

`CASE` is often used to **produce** a value:

```sql
SELECT
    CASE
        WHEN salary >= 100000 THEN 'High'
        ELSE 'Other'
    END AS salary_category
FROM employees;
```

`WHERE` is normally used to **filter** rows:

```sql
SELECT *
FROM employees
WHERE salary >= 100000;
```

Do not use `CASE` merely because a `WHERE` clause is simpler.

Use the construct that matches the requirement.

---

# 43. Categorization Boundaries

This is a common source of bugs.

Suppose the intended categories are:

```text
0–49
50–79
80–100
```

A clean implementation is:

```sql
CASE
    WHEN score < 50 THEN 'Low'
    WHEN score < 80 THEN 'Medium'
    WHEN score <= 100 THEN 'High'
    ELSE 'Invalid'
END
```

Because conditions are evaluated in order, the ranges become:

```text
score < 50
50 <= score < 80
80 <= score <= 100
```

Always test boundary values:

```text
49
50
79
80
100
101
NULL
```

---

# 44. Overlapping Conditions

Suppose:

```sql
CASE
    WHEN amount > 1000 THEN 'Large'
    WHEN amount > 500 THEN 'Medium'
    ELSE 'Small'
END
```

This is correct because the largest threshold appears first.

But:

```sql
CASE
    WHEN amount > 500 THEN 'Medium'
    WHEN amount > 1000 THEN 'Large'
    ELSE 'Small'
END
```

classifies `1500` as:

```text
Medium
```

because the first condition already matched.

---

# 45. Exhaustiveness

Ask:

> What happens to every possible row?

A robust classification considers:

- normal values,
- boundary values,
- NULL,
- unexpected values,
- negative values where applicable,
- values outside the expected range.

Use an explicit `ELSE` when an unmatched category should be represented.

Example:

```sql
CASE
    WHEN score >= 90 THEN 'A'
    WHEN score >= 80 THEN 'B'
    WHEN score >= 70 THEN 'C'
    WHEN score >= 60 THEN 'D'
    ELSE 'F'
END
```

---

# 46. Common Conditional Aggregation Mistake

Wrong:

```sql
COUNT(
    CASE
        WHEN status = 'completed' THEN 1
        ELSE 0
    END
)
```

Why wrong?

Because:

```text
COUNT(1) → counts
COUNT(0) → counts
```

Both are non-NULL.

Correct:

```sql
COUNT(
    CASE
        WHEN status = 'completed' THEN 1
    END
)
```

or:

```sql
SUM(
    CASE
        WHEN status = 'completed' THEN 1
        ELSE 0
    END
)
```

---

# 47. Common Conditional SUM Mistake

Wrong when you want completed revenue:

```sql
SUM(
    CASE
        WHEN status = 'completed' THEN amount
    END
)
```

This can be valid because NULL values are ignored by `SUM`, but it can produce `NULL` when no rows match.

If you want a numeric zero when no rows match:

```sql
SUM(
    CASE
        WHEN status = 'completed' THEN amount
        ELSE 0
    END
)
```

For complete control over an empty input/no-match result, `COALESCE` may also be appropriate:

```sql
COALESCE(
    SUM(
        CASE
            WHEN status = 'completed' THEN amount
            ELSE 0
        END
    ),
    0
)
```

---

# 48. CASE and COALESCE

These solve different problems.

`CASE`:

```sql
CASE
    WHEN condition THEN value
    ELSE other_value
END
```

`COALESCE`:

```sql
COALESCE(value, fallback)
```

Use `CASE` for conditional logic.

Use `COALESCE` for NULL fallback.

They can be combined:

```sql
COALESCE(
    SUM(
        CASE
            WHEN status = 'completed' THEN amount
            ELSE 0
        END
    ),
    0
)
```

---

# 49. CASE and DISTINCT

Suppose:

```text
customer_id
status
```

You need the number of unique customers who completed an order:

```sql
SELECT
    COUNT(
        DISTINCT CASE
            WHEN status = 'completed'
            THEN customer_id
        END
    ) AS completed_customers
FROM orders;
```

This is a high-value pattern.

---

# 50. Conditional Aggregation with Multiple Groups

Example:

```sql
SELECT
    region,

    SUM(
        CASE
            WHEN status = 'completed' THEN amount
            ELSE 0
        END
    ) AS completed_revenue,

    SUM(
        CASE
            WHEN status = 'refunded' THEN amount
            ELSE 0
        END
    ) AS refunded_amount

FROM orders
GROUP BY region;
```

This produces one row per region with separate metrics.

---

# 51. CASE for Pivot-Like Results

Conditional aggregation can transform rows into columns.

Suppose:

```text
status
completed
pending
cancelled
```

Instead of:

```text
status       count
completed    100
pending       20
cancelled     10
```

you can produce:

```text
completed_count | pending_count | cancelled_count
```

using:

```sql
SELECT
    SUM(CASE WHEN status = 'completed' THEN 1 ELSE 0 END)
        AS completed_count,

    SUM(CASE WHEN status = 'pending' THEN 1 ELSE 0 END)
        AS pending_count,

    SUM(CASE WHEN status = 'cancelled' THEN 1 ELSE 0 END)
        AS cancelled_count
FROM orders;
```

This is often called a **manual pivot** or **conditional pivot**.

---

# 52. Conditional Aggregation vs PIVOT

Some database systems provide a `PIVOT` feature.

But conditional aggregation is highly portable:

```sql
SUM(CASE WHEN ... THEN ... ELSE ... END)
```

Therefore it is an important interview technique even when the database has dedicated pivot functionality.

---

# 53. Conditional Counting with Different Predicates

Example:

```sql
SELECT
    SUM(CASE WHEN age < 18 THEN 1 ELSE 0 END) AS minors,

    SUM(CASE WHEN age >= 18 AND age < 60 THEN 1 ELSE 0 END)
        AS adults,

    SUM(CASE WHEN age >= 60 THEN 1 ELSE 0 END)
        AS seniors
FROM customers;
```

This is both:

- categorization,
- conditional aggregation.

---

# 54. Categorization vs Conditional Aggregation

They are related but not identical.

### Categorization

Creates a label for each row:

```sql
CASE
    WHEN salary < 50000 THEN 'Low'
    ELSE 'High'
END
```

### Conditional aggregation

Creates a summary metric:

```sql
SUM(
    CASE
        WHEN salary < 50000 THEN 1
        ELSE 0
    END
)
```

Mental model:

```text
CASE alone
→ classify rows

CASE + aggregate
→ summarize selected rows
```

---

# 55. A Reusable Conditional Aggregation Template

```sql
SELECT
    group_column,

    COUNT(*) AS total_rows,

    SUM(
        CASE
            WHEN condition_1 THEN 1
            ELSE 0
        END
    ) AS metric_1,

    SUM(
        CASE
            WHEN condition_2 THEN 1
            ELSE 0
        END
    ) AS metric_2,

    SUM(
        CASE
            WHEN condition_3 THEN value_column
            ELSE 0
        END
    ) AS metric_3

FROM table_name
GROUP BY group_column;
```

Memorize this structure.

---

# 56. GATE / Database Theory

## 56.1 CASE Is an Expression

`CASE` returns a value.

Therefore:

```sql
SELECT CASE ... END
```

is valid.

This is conceptually different from a procedural control-flow statement.

---

## 56.2 CASE and NULL

If no `WHEN` condition is true and no `ELSE` is supplied:

```text
CASE result → NULL
```

This matters greatly when the `CASE` is passed to:

```text
COUNT
AVG
SUM
MIN
MAX
```

because aggregate functions have different NULL behavior.

---

## 56.3 COUNT(NULL)

`COUNT(expression)` counts non-NULL expression results.

Therefore:

```sql
COUNT(
    CASE
        WHEN condition THEN 1
    END
)
```

is a conditional count.

---

## 56.4 SUM with 1 and 0

```sql
SUM(
    CASE
        WHEN condition THEN 1
        ELSE 0
    END
)
```

converts a boolean-style condition into numeric indicators:

```text
true  → 1
false → 0
```

Then aggregation counts the matches.

---

# 57. GATE-Style Solved Questions

> These are **GATE-style practice questions**, not claims of official GATE PYQs.

## Question 1 — CASE Ordering

Given:

```sql
CASE
    WHEN salary >= 50000 THEN 'M'
    WHEN salary >= 100000 THEN 'H'
    ELSE 'L'
END
```

What category is assigned to salary `120000`?

### Solution

The first condition:

```sql
salary >= 50000
```

is already true.

`CASE` stops at the first matching `WHEN`.

### Answer

```text
M
```

---

## Question 2 — Conditional COUNT

Consider:

```sql
SELECT
    COUNT(
        CASE
            WHEN status = 'completed' THEN 1
        END
    )
FROM orders;
```

If there are 100 orders and 35 are completed, what is returned?

### Solution

Completed rows produce:

```text
1
```

Other rows produce:

```text
NULL
```

`COUNT(expression)` ignores NULL.

### Answer

```text
35
```

---

## Question 3 — COUNT with ELSE 0

Consider:

```sql
SELECT
    COUNT(
        CASE
            WHEN status = 'completed' THEN 1
            ELSE 0
        END
    )
FROM orders;
```

There are 100 orders, 35 completed.

What does this return?

### Solution

Every row produces either:

```text
1
```

or:

```text
0
```

Both are non-NULL.

Therefore `COUNT` counts all 100 rows.

### Answer

```text
100
```

This is a classic trap.

---

## Question 4 — Conditional SUM

Suppose amounts are:

| status | amount |
|---|---:|
| completed | 100 |
| pending | 50 |
| completed | 200 |
| cancelled | 75 |

Evaluate:

```sql
SUM(
    CASE
        WHEN status = 'completed' THEN amount
        ELSE 0
    END
)
```

### Solution

The CASE values are:

```text
100
0
200
0
```

Sum:

```text
300
```

### Answer

```text
300
```

---

## Question 5 — Categorization

Evaluate:

```sql
CASE
    WHEN score < 50 THEN 'Low'
    WHEN score < 80 THEN 'Medium'
    ELSE 'High'
END
```

for:

```text
score = 79
```

### Solution

First:

```text
79 < 50 → FALSE
```

Second:

```text
79 < 80 → TRUE
```

Therefore:

```text
Medium
```

### Answer

```text
Medium
```

---

## Question 6 — Conditional DISTINCT Count

Suppose orders contain:

| customer_id | status |
|---:|---|
| 1 | completed |
| 1 | completed |
| 2 | completed |
| 3 | pending |
| 3 | completed |

Evaluate:

```sql
COUNT(
    DISTINCT CASE
        WHEN status = 'completed'
        THEN customer_id
    END
)
```

### Solution

Completed customer IDs:

```text
1
1
2
3
```

After `DISTINCT`:

```text
1
2
3
```

Count:

```text
3
```

### Answer

```text
3
```

---

# 58. Interview Practice

Try solving these without looking at the solution patterns.

### Q1
Classify salaries into:

```text
< 50000       → Low
50000–99999   → Medium
>= 100000     → High
```

### Q2
Count completed orders.

### Q3
Count completed and cancelled orders in the same query.

### Q4
Calculate completed-order revenue.

### Q5
Calculate the average amount of completed orders.

### Q6
Count unique customers who completed at least one order.

### Q7
For every department, count employees earning at least 100000.

### Q8
For every department, calculate total salary and high-earner salary.

### Q9
Order tickets as High → Medium → Low.

### Q10
Create columns for completed, pending, and cancelled order counts.

---

# 59. Interview Solutions

## Q1 — Salary Classification

```sql
CASE
    WHEN salary < 50000 THEN 'Low'
    WHEN salary < 100000 THEN 'Medium'
    ELSE 'High'
END
```

---

## Q2 — Completed Orders

```sql
SUM(
    CASE
        WHEN status = 'completed' THEN 1
        ELSE 0
    END
)
```

---

## Q3 — Completed and Cancelled

```sql
SELECT
    SUM(CASE WHEN status = 'completed' THEN 1 ELSE 0 END)
        AS completed_orders,

    SUM(CASE WHEN status = 'cancelled' THEN 1 ELSE 0 END)
        AS cancelled_orders
FROM orders;
```

---

## Q4 — Completed Revenue

```sql
SUM(
    CASE
        WHEN status = 'completed' THEN amount
        ELSE 0
    END
)
```

---

## Q5 — Average Completed Amount

```sql
AVG(
    CASE
        WHEN status = 'completed' THEN amount
    END
)
```

Do not use `ELSE 0` unless non-completed orders should contribute zero to the average.

---

## Q6 — Unique Completed Customers

```sql
COUNT(
    DISTINCT CASE
        WHEN status = 'completed' THEN customer_id
    END
)
```

---

## Q7 — High Earners per Department

```sql
SELECT
    department_id,
    SUM(
        CASE
            WHEN salary >= 100000 THEN 1
            ELSE 0
        END
    ) AS high_earners
FROM employees
GROUP BY department_id;
```

---

## Q8 — Total and High-Earner Salary

```sql
SELECT
    department_id,
    SUM(salary) AS total_salary,

    SUM(
        CASE
            WHEN salary >= 100000 THEN salary
            ELSE 0
        END
    ) AS high_earner_salary

FROM employees
GROUP BY department_id;
```

---

## Q9 — Custom Priority

```sql
ORDER BY
    CASE priority
        WHEN 'High' THEN 1
        WHEN 'Medium' THEN 2
        WHEN 'Low' THEN 3
        ELSE 4
    END;
```

---

## Q10 — Status Columns

```sql
SELECT
    SUM(CASE WHEN status = 'completed' THEN 1 ELSE 0 END)
        AS completed_count,

    SUM(CASE WHEN status = 'pending' THEN 1 ELSE 0 END)
        AS pending_count,

    SUM(CASE WHEN status = 'cancelled' THEN 1 ELSE 0 END)
        AS cancelled_count

FROM orders;
```

---

# 60. Common Interview Traps

## Trap 1 — Wrong CASE order

Wrong:

```sql
CASE
    WHEN score >= 50 THEN 'Pass'
    WHEN score >= 90 THEN 'Excellent'
END
```

Correct:

```sql
CASE
    WHEN score >= 90 THEN 'Excellent'
    WHEN score >= 50 THEN 'Pass'
    ELSE 'Fail'
END
```

---

## Trap 2 — COUNT with ELSE 0

Wrong:

```sql
COUNT(CASE WHEN condition THEN 1 ELSE 0 END)
```

Correct:

```sql
COUNT(CASE WHEN condition THEN 1 END)
```

or:

```sql
SUM(CASE WHEN condition THEN 1 ELSE 0 END)
```

---

## Trap 3 — ELSE 0 inside AVG

Wrong when non-matching rows should be excluded:

```sql
AVG(
    CASE
        WHEN status = 'completed' THEN amount
        ELSE 0
    END
)
```

Correct:

```sql
AVG(
    CASE
        WHEN status = 'completed' THEN amount
    END
)
```

---

## Trap 4 — Using `=` with NULL

Wrong:

```sql
CASE
    WHEN phone = NULL THEN 'Missing'
END
```

Correct:

```sql
CASE
    WHEN phone IS NULL THEN 'Missing'
END
```

---

## Trap 5 — Forgetting ELSE

Without `ELSE`, unmatched rows generally become `NULL`.

That may be correct or incorrect depending on the aggregate.

---

## Trap 6 — Overlapping ranges

Always inspect the order:

```text
specific → general
```

when ranges overlap.

---

## Trap 7 — Confusing categorization and filtering

```sql
CASE
    WHEN ... THEN ...
END
```

creates a value.

```sql
WHERE ...
```

removes rows.

---

# 61. Pattern Recognition

When the question says:

### "Classify", "categorize", "bucket", "band"

Think:

```sql
CASE WHEN ...
```

---

### "Count rows satisfying condition"

Think:

```sql
SUM(CASE WHEN condition THEN 1 ELSE 0 END)
```

or:

```sql
COUNT(CASE WHEN condition THEN 1 END)
```

---

### "Count unique entities satisfying condition"

Think:

```sql
COUNT(
    DISTINCT CASE
        WHEN condition THEN id
    END
)
```

---

### "Sum values satisfying condition"

Think:

```sql
SUM(
    CASE
        WHEN condition THEN value
        ELSE 0
    END
)
```

---

### "Average values satisfying condition"

Think:

```sql
AVG(
    CASE
        WHEN condition THEN value
    END
)
```

---

### "Multiple conditional metrics"

Think:

```sql
SELECT
    SUM(CASE WHEN condition1 THEN ... END),
    SUM(CASE WHEN condition2 THEN ... END),
    SUM(CASE WHEN condition3 THEN ... END)
FROM ...
```

---

### "Custom sort order"

Think:

```sql
ORDER BY CASE ...
```

---

# 62. Decision Tree

```text
Need conditional logic?
        |
        v
Need to produce a label/value per row?
        |
       YES
        |
        v
     CASE WHEN
        |
        +---- Categorize/bucket → CASE
        |
        +---- Custom ordering   → CASE in ORDER BY
        |
        +---- Conditional value → CASE in SELECT

Need a summary metric?
        |
       YES
        |
        v
   CASE + aggregate
        |
        +---- Count rows
        |       → SUM(CASE WHEN ... THEN 1 ELSE 0 END)
        |
        +---- Count unique IDs
        |       → COUNT(DISTINCT CASE WHEN ... THEN id END)
        |
        +---- Sum values
        |       → SUM(CASE WHEN ... THEN value ELSE 0 END)
        |
        +---- Average values
        |       → AVG(CASE WHEN ... THEN value END)
```

---

# 63. High-Value Templates

## Categorization

```sql
CASE
    WHEN condition_1 THEN 'Category 1'
    WHEN condition_2 THEN 'Category 2'
    ELSE 'Other'
END
```

## Conditional Count

```sql
SUM(
    CASE
        WHEN condition THEN 1
        ELSE 0
    END
)
```

## Conditional COUNT

```sql
COUNT(
    CASE
        WHEN condition THEN 1
    END
)
```

## Conditional Distinct Count

```sql
COUNT(
    DISTINCT CASE
        WHEN condition THEN id
    END
)
```

## Conditional Sum

```sql
SUM(
    CASE
        WHEN condition THEN amount
        ELSE 0
    END
)
```

## Conditional Average

```sql
AVG(
    CASE
        WHEN condition THEN amount
    END
)
```

## Conditional Maximum

```sql
MAX(
    CASE
        WHEN condition THEN amount
    END
)
```

---

# 64. Mastery Checklist

You should be able to:

- [ ] Explain `CASE`.
- [ ] Write searched `CASE WHEN`.
- [ ] Write simple `CASE`.
- [ ] Explain the difference between searched and simple CASE.
- [ ] Explain top-to-bottom evaluation.
- [ ] Correctly order overlapping conditions.
- [ ] Use `ELSE`.
- [ ] Explain what happens when `ELSE` is omitted.
- [ ] Categorize numeric values.
- [ ] Create ranges/buckets.
- [ ] Handle boundary values correctly.
- [ ] Handle NULL explicitly.
- [ ] Use CASE in `SELECT`.
- [ ] Use CASE in `ORDER BY`.
- [ ] Use CASE with `GROUP BY`.
- [ ] Use CASE with aggregate functions.
- [ ] Write conditional counts.
- [ ] Write conditional sums.
- [ ] Write conditional averages.
- [ ] Count distinct values conditionally.
- [ ] Produce multiple conditional metrics in one query.
- [ ] Explain why `COUNT(CASE ... ELSE 0 END)` is wrong for conditional counting.
- [ ] Explain why `AVG(CASE ... ELSE 0 END)` can be wrong.
- [ ] Distinguish `WHERE` filtering from conditional aggregation.
- [ ] Recognize manual pivot patterns.
- [ ] Solve GATE-style CASE questions.
- [ ] Recognize CASE patterns quickly in interview problems.

---

# 65. Final Revision Sheet

## CASE

```sql
CASE
    WHEN condition THEN result
    ELSE default_result
END
```

## Evaluation

```text
Top → bottom
First TRUE WHEN wins
No match + no ELSE → NULL
```

## Categorization

```sql
CASE
    WHEN value < 10 THEN 'Low'
    WHEN value < 20 THEN 'Medium'
    ELSE 'High'
END
```

## Conditional count

Best general pattern:

```sql
SUM(
    CASE
        WHEN condition THEN 1
        ELSE 0
    END
)
```

Alternative:

```sql
COUNT(
    CASE
        WHEN condition THEN 1
    END
)
```

## Critical trap

```sql
COUNT(CASE WHEN condition THEN 1 ELSE 0 END)
```

counts **all rows**, because `0` is not NULL.

## Conditional sum

```sql
SUM(
    CASE
        WHEN condition THEN amount
        ELSE 0
    END
)
```

## Conditional average

```sql
AVG(
    CASE
        WHEN condition THEN amount
    END
)
```

Do not use `ELSE 0` unless zero should be included in the average.

## Conditional distinct count

```sql
COUNT(
    DISTINCT CASE
        WHEN condition THEN id
    END
)
```

## Core mental model

```text
CASE
→ classify/select a value

CASE + aggregate
→ conditionally summarize rows
```

---

# 66. Final Takeaway

The most important skill in this topic is recognizing the pattern:

```text
IF a row satisfies condition
        ↓
CASE
        ↓
produce a value
        ↓
aggregate if necessary
```

For interviews, memorize these four patterns:

```sql
-- Categorization
CASE
    WHEN condition THEN category
    ELSE other_category
END
```

```sql
-- Conditional count
SUM(
    CASE
        WHEN condition THEN 1
        ELSE 0
    END
)
```

```sql
-- Conditional sum
SUM(
    CASE
        WHEN condition THEN amount
        ELSE 0
    END
)
```

```sql
-- Conditional distinct count
COUNT(
    DISTINCT CASE
        WHEN condition THEN id
    END
)
```

And remember the two major traps:

```text
COUNT + ELSE 0
→ counts every row

AVG + ELSE 0
→ can incorrectly include non-matching rows as zero
```

If you can recognize and write these patterns without hesitation, you have the core of `CASE` and conditional aggregation required for SQL interviews.
