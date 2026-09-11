# SQL Topic 10 — Window Functions

> **Overall repository topic:** 33  
> **SQL topic:** 10  
> **Priority:** EXTREMELY IMPORTANT  
> **Interview relevance:** Very High  
> **Difficulty:** Medium → Hard  
> **Primary use:** Ranking, row-to-row comparison, running metrics, moving metrics, per-group analytics without collapsing rows

---

# 1. What Are Window Functions?

A **window function** performs a calculation across a related set of rows while **preserving the individual rows** of the result.

This is the key difference between:

- `GROUP BY` → combines rows into groups and normally returns one row per group.
- Window functions → calculate across rows but keep the original row-level detail.

Example:

```sql
SELECT
    employee_id,
    department,
    salary,
    AVG(salary) OVER (PARTITION BY department) AS dept_avg_salary
FROM employees;
```

Every employee remains in the result, but each row also receives the average salary of its department.

---

# 2. The Core Mental Model

Think of a window function as:

> **"For this row, define a set of related rows, then calculate something over that set."**

The window is controlled primarily by:

```sql
OVER (
    PARTITION BY ...
    ORDER BY ...
    ROWS/RANGE ...
)
```

The three most important concepts are:

1. `OVER()`
2. `PARTITION BY`
3. `ORDER BY`

---

# 3. `OVER()`

`OVER()` tells SQL that a function should operate as a window function.

Example:

```sql
SELECT
    employee_id,
    salary,
    AVG(salary) OVER () AS company_avg_salary
FROM employees;
```

The average is calculated over the entire result set.

## Comparison with ordinary aggregate

### Aggregate

```sql
SELECT AVG(salary)
FROM employees;
```

Returns one value.

### Window aggregate

```sql
SELECT
    employee_id,
    salary,
    AVG(salary) OVER () AS avg_salary
FROM employees;
```

Returns every employee plus the overall average.

---

# 4. `PARTITION BY`

`PARTITION BY` divides the result into independent windows.

Example:

```sql
SELECT
    employee_id,
    department,
    salary,
    AVG(salary) OVER (
        PARTITION BY department
    ) AS department_avg
FROM employees;
```

Each department gets its own average.

It does **not** collapse the rows.

If departments are:

```text
Engineering
Engineering
Sales
Sales
HR
```

the window calculation is performed independently for:

```text
Engineering
Sales
HR
```

---

# 5. `ORDER BY` Inside `OVER()`

Window `ORDER BY` defines the ordering used by the window calculation.

Example:

```sql
SELECT
    employee_id,
    salary,
    ROW_NUMBER() OVER (
        ORDER BY salary DESC
    ) AS salary_position
FROM employees;
```

This assigns positions based on salary.

Important:

```sql
ORDER BY salary DESC
```

inside `OVER()` is not necessarily the same as the final result ordering.

You may separately write:

```sql
ORDER BY salary_position;
```

at the end of the query.

---

# 6. Window `ORDER BY` vs Final `ORDER BY`

These are conceptually different.

```sql
SELECT
    employee_id,
    salary,
    ROW_NUMBER() OVER (
        ORDER BY salary DESC
    ) AS rn
FROM employees
ORDER BY employee_id;
```

The row numbers are assigned by salary, but the final output is displayed by employee ID.

Therefore:

- Window `ORDER BY` → determines calculation order.
- Final `ORDER BY` → determines output order.

---

# 7. Basic Window Syntax

General form:

```sql
function(...) OVER (
    PARTITION BY column1, column2
    ORDER BY column3
)
```

Example:

```sql
SELECT
    employee_id,
    department,
    salary,
    RANK() OVER (
        PARTITION BY department
        ORDER BY salary DESC
    ) AS salary_rank
FROM employees;
```

---

# 8. Window Functions vs `GROUP BY`

Suppose:

```text
employees
--------------------------------
id | department | salary
1  | IT         | 80000
2  | IT         | 90000
3  | HR         | 70000
4  | HR         | 75000
```

## `GROUP BY`

```sql
SELECT
    department,
    AVG(salary)
FROM employees
GROUP BY department;
```

Result:

```text
IT   85000
HR   72500
```

Rows are collapsed.

## Window function

```sql
SELECT
    id,
    department,
    salary,
    AVG(salary) OVER (
        PARTITION BY department
    ) AS dept_avg
FROM employees;
```

Result conceptually:

```text
1 | IT | 80000 | 85000
2 | IT | 90000 | 85000
3 | HR | 70000 | 72500
4 | HR | 75000 | 72500
```

The original rows remain.

---

# 9. The Most Important Window Categories

## Ranking

- `ROW_NUMBER()`
- `RANK()`
- `DENSE_RANK()`

## Navigation

- `LAG()`
- `LEAD()`
- `FIRST_VALUE()`
- `LAST_VALUE()`

## Aggregation

- Running sum
- Running average
- Moving average
- Cumulative count
- Partitioned aggregation

---

# 10. `ROW_NUMBER()`

`ROW_NUMBER()` assigns a unique sequential number to each row in the window.

```sql
ROW_NUMBER() OVER (
    ORDER BY salary DESC
)
```

Example:

```text
salary   row_number
100000   1
90000    2
90000    3
80000    4
```

Even tied values receive different row numbers.

---

# 11. `ROW_NUMBER()` With Partitions

```sql
SELECT
    employee_id,
    department,
    salary,
    ROW_NUMBER() OVER (
        PARTITION BY department
        ORDER BY salary DESC
    ) AS rn
FROM employees;
```

Each department starts numbering from 1.

Example:

```text
department | salary | rn
-----------+--------+---
IT         | 100000 | 1
IT         | 90000  | 2
HR         | 90000  | 1
HR         | 70000  | 2
```

---

# 12. When to Use `ROW_NUMBER()`

Use it when you need:

- exactly one unique position per row
- top 1 row per group
- deduplication
- latest record per entity
- deterministic selection of one row after adding a tie-breaker

Classic pattern:

```sql
WITH ranked AS (
    SELECT
        *,
        ROW_NUMBER() OVER (
            PARTITION BY customer_id
            ORDER BY order_date DESC, order_id DESC
        ) AS rn
    FROM orders
)
SELECT *
FROM ranked
WHERE rn = 1;
```

This returns the latest order per customer.

---

# 13. Why a Tie-Breaker Matters

Consider:

```sql
ROW_NUMBER() OVER (
    PARTITION BY customer_id
    ORDER BY order_date DESC
)
```

If two rows have exactly the same `order_date`, their relative ordering may not be deterministic.

Prefer:

```sql
ROW_NUMBER() OVER (
    PARTITION BY customer_id
    ORDER BY order_date DESC, order_id DESC
)
```

The second column breaks ties.

---

# 14. `RANK()`

`RANK()` gives equal values the same rank.

Example:

```text
salary   rank
100000   1
90000    2
90000    2
80000    4
```

There is a gap after the tie.

Syntax:

```sql
RANK() OVER (
    ORDER BY salary DESC
)
```

---

# 15. `DENSE_RANK()`

`DENSE_RANK()` also gives ties the same rank, but does not leave gaps.

Example:

```text
salary   dense_rank
100000   1
90000    2
90000    2
80000    3
```

---

# 16. `ROW_NUMBER()` vs `RANK()` vs `DENSE_RANK()`

| Salary | `ROW_NUMBER()` | `RANK()` | `DENSE_RANK()` |
|---:|---:|---:|---:|
| 100 | 1 | 1 | 1 |
| 90 | 2 | 2 | 2 |
| 90 | 3 | 2 | 2 |
| 80 | 4 | 4 | 3 |
| 70 | 5 | 5 | 4 |

Memorize:

- `ROW_NUMBER()` → no ties
- `RANK()` → ties + gaps
- `DENSE_RANK()` → ties + no gaps

---

# 17. Ranking Decision Rule

### Need exactly one row?

Use:

```sql
ROW_NUMBER()
```

### Need competition ranking?

Use:

```sql
RANK()
```

### Need ranking of distinct values without gaps?

Use:

```sql
DENSE_RANK()
```

---

# 18. Top N Per Group

One of the most important SQL interview patterns.

Problem:

> Find the top 3 highest-paid employees in every department.

Solution:

```sql
WITH ranked AS (
    SELECT
        employee_id,
        department,
        salary,
        ROW_NUMBER() OVER (
            PARTITION BY department
            ORDER BY salary DESC
        ) AS rn
    FROM employees
)
SELECT *
FROM ranked
WHERE rn <= 3;
```

If ties should all be included:

```sql
WITH ranked AS (
    SELECT
        employee_id,
        department,
        salary,
        RANK() OVER (
            PARTITION BY department
            ORDER BY salary DESC
        ) AS rnk
    FROM employees
)
SELECT *
FROM ranked
WHERE rnk <= 3;
```

---

# 19. Top N and Tie Semantics

Suppose salaries are:

```text
100
90
90
80
70
```

For top 3:

### `ROW_NUMBER() <= 3`

Returns:

```text
100
90
90
```

### `RANK() <= 3`

Returns:

```text
100
90
90
80
```

because `80` has rank 4? Wait carefully:

```text
100 -> 1
90  -> 2
90  -> 2
80  -> 4
```

So `RANK() <= 3` returns only:

```text
100
90
90
```

If the cutoff value itself is tied at rank 3, `RANK()` includes all rows sharing that rank.

---

# 20. `DENSE_RANK()` for Top Distinct Values

Suppose:

```text
100
90
90
80
80
70
```

Distinct salary levels:

```text
100 -> 1
90  -> 2
80  -> 3
70  -> 4
```

For top 3 salary levels:

```sql
DENSE_RANK() OVER (
    ORDER BY salary DESC
)
```

then:

```sql
WHERE dense_rank <= 3
```

returns all employees earning:

```text
100
90
80
```

---

# 21. `LAG()`

`LAG()` accesses a previous row in the window.

Syntax:

```sql
LAG(column, offset, default)
OVER (
    ORDER BY ...
)
```

Basic:

```sql
LAG(sales)
OVER (
    ORDER BY sale_date
)
```

Conceptually:

```text
date       sales   previous_sales
Jan 1      100     NULL
Jan 2      120     100
Jan 3      90      120
```

---

# 22. Previous Value With `LAG()`

```sql
SELECT
    sale_date,
    sales,
    LAG(sales) OVER (
        ORDER BY sale_date
    ) AS previous_sales
FROM sales;
```

This is one of the most common interview patterns.

---

# 23. `LAG()` With an Offset

Previous 2 rows:

```sql
LAG(sales, 2)
OVER (
    ORDER BY sale_date
)
```

Example:

```text
day   sales   lag_2
1     100     NULL
2     120     NULL
3     140     100
4     160     120
```

---

# 24. `LAG()` With a Default

```sql
LAG(sales, 1, 0)
OVER (
    ORDER BY sale_date
)
```

For the first row, `0` is returned instead of `NULL`.

---

# 25. Comparing Current Row With Previous Row

```sql
SELECT
    sale_date,
    sales,
    LAG(sales) OVER (
        ORDER BY sale_date
    ) AS previous_sales,
    sales -
    LAG(sales) OVER (
        ORDER BY sale_date
    ) AS change
FROM sales;
```

For readability and portability, calculate `LAG()` in a CTE when reusing it:

```sql
WITH x AS (
    SELECT
        sale_date,
        sales,
        LAG(sales) OVER (
            ORDER BY sale_date
        ) AS previous_sales
    FROM sales
)
SELECT
    sale_date,
    sales,
    previous_sales,
    sales - previous_sales AS change
FROM x;
```

---

# 26. Percentage Change

```sql
WITH x AS (
    SELECT
        sale_date,
        sales,
        LAG(sales) OVER (
            ORDER BY sale_date
        ) AS previous_sales
    FROM sales
)
SELECT
    sale_date,
    sales,
    previous_sales,
    100.0 * (sales - previous_sales)
        / NULLIF(previous_sales, 0) AS pct_change
FROM x;
```

Important:

```sql
NULLIF(previous_sales, 0)
```

prevents division by zero.

---

# 27. `LEAD()`

`LEAD()` accesses a future row.

```sql
LEAD(sales)
OVER (
    ORDER BY sale_date
)
```

Conceptually:

```text
date       sales   next_sales
Jan 1      100     120
Jan 2      120     90
Jan 3      90      NULL
```

---

# 28. `LAG()` vs `LEAD()`

| Function | Direction |
|---|---|
| `LAG()` | previous row |
| `LEAD()` | next row |

Typical use:

### `LAG()`

- month-over-month growth
- previous transaction
- previous status
- change from previous event

### `LEAD()`

- next transaction
- next event
- time until next event
- detecting gaps

---

# 29. Partitioned `LAG()` / `LEAD()`

For each customer:

```sql
SELECT
    customer_id,
    order_date,
    amount,
    LAG(amount) OVER (
        PARTITION BY customer_id
        ORDER BY order_date
    ) AS previous_amount
FROM orders;
```

The previous row is previous **within the customer**, not previous globally.

This is critical.

---

# 30. `FIRST_VALUE()`

Returns the first value in the window according to the window ordering/frame.

Example:

```sql
FIRST_VALUE(salary) OVER (
    PARTITION BY department
    ORDER BY salary DESC
)
```

This can return the highest salary in each department.

---

# 31. `LAST_VALUE()` — Important Trap

`LAST_VALUE()` is more subtle.

Consider:

```sql
LAST_VALUE(salary) OVER (
    PARTITION BY department
    ORDER BY salary
)
```

Many users expect the final salary of the department.

But the default window frame in many SQL systems with an ordered window does **not necessarily mean the entire partition**.

The current row can be the end of the default frame.

Therefore `LAST_VALUE()` often appears to return the current row's value.

---

# 32. Correct Full-Partition `LAST_VALUE()`

Use an explicit frame when you mean the last value across the full ordered partition:

```sql
LAST_VALUE(salary) OVER (
    PARTITION BY department
    ORDER BY salary
    ROWS BETWEEN UNBOUNDED PRECEDING
             AND UNBOUNDED FOLLOWING
)
```

The exact default-frame behavior varies by DBMS and frame type, so explicit framing is safer when `LAST_VALUE()` semantics matter.

---

# 33. `FIRST_VALUE()` vs `MIN()`

These are not identical concepts.

```sql
MIN(salary) OVER (
    PARTITION BY department
)
```

returns the minimum numeric salary.

But:

```sql
FIRST_VALUE(employee_name) OVER (
    PARTITION BY department
    ORDER BY salary
)
```

returns the employee associated with the first row according to the specified ordering.

Window ordering can therefore define which row's value is selected.

---

# 34. Window Frames

A window can be narrowed to a specific frame.

General form:

```sql
ROWS BETWEEN start AND end
```

Common boundaries:

```text
UNBOUNDED PRECEDING
n PRECEDING
CURRENT ROW
n FOLLOWING
UNBOUNDED FOLLOWING
```

Example:

```sql
SUM(amount) OVER (
    ORDER BY order_date
    ROWS BETWEEN UNBOUNDED PRECEDING
             AND CURRENT ROW
)
```

This produces a running sum.

---

# 35. `ROWS` vs `RANGE`

This distinction is important for interviews.

### `ROWS`

Works with physical rows.

```sql
ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
```

means the current row plus the previous two rows.

### `RANGE`

Works with the ordering value and peer rows.

Its behavior with duplicate ordering values can therefore differ from `ROWS`.

For precise row-count windows, prefer `ROWS`.

---

# 36. Running Sum

A running sum accumulates values from the beginning through the current row.

```sql
SELECT
    order_date,
    amount,
    SUM(amount) OVER (
        ORDER BY order_date
        ROWS BETWEEN UNBOUNDED PRECEDING
                 AND CURRENT ROW
    ) AS running_total
FROM orders;
```

Example:

```text
amount   running_total
100      100
200      300
50       350
150      500
```

---

# 37. Running Sum Per Customer

```sql
SELECT
    customer_id,
    order_date,
    amount,
    SUM(amount) OVER (
        PARTITION BY customer_id
        ORDER BY order_date
        ROWS BETWEEN UNBOUNDED PRECEDING
                 AND CURRENT ROW
    ) AS customer_running_total
FROM orders;
```

Each customer gets an independent cumulative total.

---

# 38. Running Average

```sql
AVG(amount) OVER (
    ORDER BY order_date
    ROWS BETWEEN UNBOUNDED PRECEDING
             AND CURRENT ROW
)
```

Example:

```text
amount   running_avg
100      100
200      150
50       116.67
```

---

# 39. Running Count

Count rows cumulatively:

```sql
COUNT(*) OVER (
    ORDER BY order_date
    ROWS BETWEEN UNBOUNDED PRECEDING
             AND CURRENT ROW
) AS cumulative_count
```

Per customer:

```sql
COUNT(*) OVER (
    PARTITION BY customer_id
    ORDER BY order_date
    ROWS BETWEEN UNBOUNDED PRECEDING
             AND CURRENT ROW
)
```

---

# 40. Moving Average

A moving average uses a fixed-width window around the current row.

Example: 3-row moving average.

```sql
AVG(amount) OVER (
    ORDER BY order_date
    ROWS BETWEEN 2 PRECEDING
             AND CURRENT ROW
) AS moving_avg
```

For:

```text
100
200
300
400
```

the averages are:

```text
100
150
200
300
```

---

# 41. Moving Average vs Running Average

## Running average

Window grows:

```text
1 row
2 rows
3 rows
4 rows
...
```

Example:

```sql
ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
```

## Moving average

Window has a fixed maximum width:

```text
current + previous N rows
```

Example:

```sql
ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
```

---

# 42. Partitioned Aggregation

Window aggregates can calculate group statistics without `GROUP BY`.

Example:

```sql
SELECT
    employee_id,
    department,
    salary,
    SUM(salary) OVER (
        PARTITION BY department
    ) AS department_total
FROM employees;
```

Every employee sees the total salary of their department.

---

# 43. Partitioned Average

```sql
AVG(salary) OVER (
    PARTITION BY department
) AS department_average
```

Useful for:

- employee vs department average
- product vs category average
- customer vs customer-segment average
- transaction vs account average

---

# 44. Difference From Group Average

```sql
SELECT
    employee_id,
    department,
    salary,
    salary - AVG(salary) OVER (
        PARTITION BY department
    ) AS difference_from_dept_avg
FROM employees;
```

This keeps row-level detail.

---

# 45. Salary Above Department Average

A window function cannot generally be referenced directly in the same `WHERE` clause where it is defined.

Use a CTE/subquery:

```sql
WITH x AS (
    SELECT
        employee_id,
        department,
        salary,
        AVG(salary) OVER (
            PARTITION BY department
        ) AS dept_avg
    FROM employees
)
SELECT *
FROM x
WHERE salary > dept_avg;
```

---

# 46. Why CTEs Are Common With Window Functions

A common pattern is:

```text
1. Calculate window value
2. Put it in a CTE/subquery
3. Filter or reuse the result outside
```

Example:

```sql
WITH ranked AS (
    SELECT
        *,
        ROW_NUMBER() OVER (
            PARTITION BY department
            ORDER BY salary DESC
        ) AS rn
    FROM employees
)
SELECT *
FROM ranked
WHERE rn <= 3;
```

This is one of the most important SQL interview templates.

---

# 47. Window Function Execution Order

A simplified logical model is:

```text
FROM
JOIN
WHERE
GROUP BY
HAVING
SELECT
WINDOW FUNCTIONS
DISTINCT
ORDER BY
LIMIT/OFFSET
```

Exact optimizer execution can differ physically.

The important practical consequence:

You generally cannot write:

```sql
WHERE ROW_NUMBER() OVER (...) <= 3
```

Instead:

```sql
WITH ranked AS (...)
SELECT *
FROM ranked
WHERE rn <= 3;
```

---

# 48. Multiple Window Functions

You can calculate several window metrics in one query:

```sql
SELECT
    employee_id,
    department,
    salary,

    ROW_NUMBER() OVER (
        PARTITION BY department
        ORDER BY salary DESC
    ) AS rn,

    RANK() OVER (
        PARTITION BY department
        ORDER BY salary DESC
    ) AS rnk,

    AVG(salary) OVER (
        PARTITION BY department
    ) AS dept_avg,

    LAG(salary) OVER (
        PARTITION BY department
        ORDER BY salary DESC
    ) AS previous_salary
FROM employees;
```

---

# 49. Common Interview Pattern: Latest Row Per Group

Problem:

> Find the latest order for every customer.

```sql
WITH ranked AS (
    SELECT
        o.*,
        ROW_NUMBER() OVER (
            PARTITION BY customer_id
            ORDER BY order_date DESC, order_id DESC
        ) AS rn
    FROM orders o
)
SELECT *
FROM ranked
WHERE rn = 1;
```

This is usually cleaner than complicated correlated subqueries.

---

# 50. Common Interview Pattern: Second Highest Salary Per Department

```sql
WITH ranked AS (
    SELECT
        employee_id,
        department,
        salary,
        DENSE_RANK() OVER (
            PARTITION BY department
            ORDER BY salary DESC
        ) AS rnk
    FROM employees
)
SELECT *
FROM ranked
WHERE rnk = 2;
```

Use `DENSE_RANK()` when "second highest" means the second distinct salary.

---

# 51. Common Interview Pattern: Previous Transaction

```sql
SELECT
    customer_id,
    transaction_date,
    amount,
    LAG(transaction_date) OVER (
        PARTITION BY customer_id
        ORDER BY transaction_date
    ) AS previous_transaction_date
FROM transactions;
```

---

# 52. Days Since Previous Transaction

Syntax for date arithmetic varies by DBMS.

Conceptually:

```sql
WITH x AS (
    SELECT
        customer_id,
        transaction_date,
        LAG(transaction_date) OVER (
            PARTITION BY customer_id
            ORDER BY transaction_date
        ) AS previous_date
    FROM transactions
)
SELECT
    customer_id,
    transaction_date,
    previous_date,
    transaction_date - previous_date AS gap
FROM x;
```

Use the date-difference function appropriate to your DBMS when exact syntax matters.

---

# 53. Common Interview Pattern: Detect Change From Previous Status

```sql
WITH x AS (
    SELECT
        user_id,
        event_time,
        status,
        LAG(status) OVER (
            PARTITION BY user_id
            ORDER BY event_time
        ) AS previous_status
    FROM events
)
SELECT *
FROM x
WHERE previous_status IS NOT NULL
  AND status <> previous_status;
```

This pattern is useful for event streams and state transitions.

---

# 54. Common Interview Pattern: Running Total

```sql
SELECT
    order_date,
    amount,
    SUM(amount) OVER (
        ORDER BY order_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_total
FROM orders;
```

---

# 55. Common Interview Pattern: Moving Average

```sql
SELECT
    order_date,
    amount,
    AVG(amount) OVER (
        ORDER BY order_date
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS seven_row_average
FROM orders;
```

Important:

This is a **7-row** average, not necessarily a **7-calendar-day** average.

If dates are missing, these are not equivalent.

---

# 56. Calendar-Time vs Row-Based Windows

Suppose sales exist only on:

```text
Monday
Thursday
Friday
```

Then:

```sql
ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
```

means three records.

It does not mean three calendar days.

This distinction is frequently tested.

---

# 57. Ranking by Multiple Columns

You can order by multiple columns:

```sql
ROW_NUMBER() OVER (
    PARTITION BY department
    ORDER BY salary DESC, employee_id ASC
)
```

The first column is the primary ordering criterion.

The second breaks ties.

---

# 58. Ranking With NULLs

Ordering of `NULL` values differs by DBMS and by `ASC`/`DESC`.

If exact placement matters, make it explicit where supported:

```sql
ORDER BY salary DESC NULLS LAST
```

or use a `CASE` expression for portable control.

Do not assume all DBMSs treat NULL ordering identically.

---

# 59. Window Functions and NULLs

Window aggregates generally follow the behavior of the underlying aggregate.

For example:

```sql
AVG(value) OVER (...)
```

normally ignores `NULL` values.

But:

```sql
COUNT(*)
```

counts rows, while:

```sql
COUNT(value)
```

counts non-NULL values.

The same distinction remains important inside windows.

---

# 60. `COUNT(*) OVER()` for Total Row Count

A useful pattern:

```sql
SELECT
    employee_id,
    department,
    COUNT(*) OVER () AS total_rows
FROM employees;
```

Every row receives the total number of rows.

Partitioned:

```sql
COUNT(*) OVER (
    PARTITION BY department
) AS department_employee_count
```

---

# 61. Percentage of Partition Total

```sql
SELECT
    employee_id,
    department,
    salary,
    100.0 * salary
        / NULLIF(
            SUM(salary) OVER (PARTITION BY department),
            0
        ) AS pct_of_department_payroll
FROM employees;
```

This is a high-value analytical pattern.

---

# 62. Window Functions After Aggregation

Window functions can operate on the rows produced after grouping.

Example:

```sql
SELECT
    department,
    SUM(salary) AS total_salary,
    RANK() OVER (
        ORDER BY SUM(salary) DESC
    ) AS department_rank
FROM employees
GROUP BY department;
```

Conceptually:

1. Group employees by department.
2. Calculate total salary.
3. Rank the resulting department rows.

This combination is extremely common.

---

# 63. Ranking Aggregated Results

Example:

> Rank products by total revenue.

```sql
SELECT
    product_id,
    SUM(amount) AS revenue,
    RANK() OVER (
        ORDER BY SUM(amount) DESC
    ) AS revenue_rank
FROM sales
GROUP BY product_id;
```

The window function is ranking the grouped result.

---

# 64. Named Windows

Some SQL dialects support the `WINDOW` clause:

```sql
SELECT
    employee_id,
    salary,
    ROW_NUMBER() OVER w AS rn,
    RANK() OVER w AS rnk
FROM employees
WINDOW w AS (
    PARTITION BY department
    ORDER BY salary DESC
);
```

This can reduce repeated window definitions.

Support varies by DBMS.

---

# 65. `OVER()` Without `PARTITION BY` or `ORDER BY`

```sql
SUM(amount) OVER ()
```

means the entire result set is the window.

Example:

```sql
SELECT
    order_id,
    amount,
    SUM(amount) OVER () AS total_sales
FROM orders;
```

---

# 66. `PARTITION BY` Without `ORDER BY`

```sql
SUM(amount) OVER (
    PARTITION BY customer_id
)
```

means:

> Calculate the aggregate independently for each customer.

There is no row sequence involved.

This is ideal for:

- group totals
- group averages
- group counts
- percentage-of-group calculations

---

# 67. `ORDER BY` Without `PARTITION BY`

```sql
SUM(amount) OVER (
    ORDER BY order_date
)
```

means:

> Use one global window, ordered by date.

This is typical for running calculations.

---

# 68. The Three Basic Forms

Memorize these:

### Entire result

```sql
SUM(x) OVER ()
```

### Per group

```sql
SUM(x) OVER (
    PARTITION BY group_col
)
```

### Ordered/running calculation

```sql
SUM(x) OVER (
    ORDER BY date_col
)
```

---

# 69. Window Function Decision Tree

## Need ranking?

Use:

```text
ROW_NUMBER / RANK / DENSE_RANK
```

## Need previous row?

```text
LAG
```

## Need next row?

```text
LEAD
```

## Need first/last value?

```text
FIRST_VALUE / LAST_VALUE
```

## Need group metric while retaining rows?

```text
SUM/AVG/COUNT/... OVER (PARTITION BY ...)
```

## Need cumulative metric?

```text
SUM/AVG/COUNT OVER (ORDER BY ... ROWS ...)
```

## Need fixed-width recent-row metric?

```text
ROWS BETWEEN N PRECEDING AND CURRENT ROW
```

---

# 70. High-Value Syntax Templates

## Ranking

```sql
ROW_NUMBER() OVER (
    PARTITION BY group_col
    ORDER BY metric DESC
)
```

## Previous row

```sql
LAG(value) OVER (
    PARTITION BY group_col
    ORDER BY time_col
)
```

## Next row

```sql
LEAD(value) OVER (
    PARTITION BY group_col
    ORDER BY time_col
)
```

## Group aggregate

```sql
SUM(value) OVER (
    PARTITION BY group_col
)
```

## Running sum

```sql
SUM(value) OVER (
    ORDER BY time_col
    ROWS BETWEEN UNBOUNDED PRECEDING
             AND CURRENT ROW
)
```

## Moving average

```sql
AVG(value) OVER (
    ORDER BY time_col
    ROWS BETWEEN 6 PRECEDING
             AND CURRENT ROW
)
```

---

# 71. Important Trap: Window Ordering Is Not Final Ordering

This:

```sql
ROW_NUMBER() OVER (
    ORDER BY salary DESC
)
```

does not guarantee the displayed result is sorted by salary.

If output ordering matters:

```sql
ORDER BY salary DESC;
```

must be specified separately.

---

# 72. Important Trap: `ROW_NUMBER()` Is Not a Tie-Aware Rank

If two employees earn the same salary:

```text
ROW_NUMBER -> 1, 2
RANK       -> 1, 1
DENSE_RANK -> 1, 1
```

Choose the function based on the business meaning.

---

# 73. Important Trap: `RANK()` Has Gaps

For:

```text
100
90
90
80
```

`RANK()` is:

```text
1
2
2
4
```

Not:

```text
1
2
2
3
```

That latter behavior is `DENSE_RANK()`.

---

# 74. Important Trap: `LAST_VALUE()`

Do not casually write:

```sql
LAST_VALUE(x) OVER (
    PARTITION BY group_col
    ORDER BY date_col
)
```

if you mean the final value of the entire partition.

Use an explicit full frame when required:

```sql
ROWS BETWEEN UNBOUNDED PRECEDING
         AND UNBOUNDED FOLLOWING
```

---

# 75. Important Trap: `ROWS N PRECEDING`

```sql
ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
```

means up to 7 rows.

It does not automatically mean:

```text
7 calendar days
```

---

# 76. Important Trap: Filtering Window Results

Wrong:

```sql
SELECT
    *,
    ROW_NUMBER() OVER (...) AS rn
FROM employees
WHERE rn = 1;
```

`rn` is not available to `WHERE` at that query level.

Correct:

```sql
WITH x AS (
    SELECT
        *,
        ROW_NUMBER() OVER (...) AS rn
    FROM employees
)
SELECT *
FROM x
WHERE rn = 1;
```

---

# 77. Important Trap: Determinism

For top-row problems, this:

```sql
ROW_NUMBER() OVER (
    PARTITION BY customer_id
    ORDER BY order_date DESC
)
```

may be insufficient if multiple rows share the same date.

Prefer a unique tie-breaker:

```sql
ORDER BY order_date DESC, order_id DESC
```

---

# 78. Important Trap: Missing Dates

A 7-row moving average:

```sql
ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
```

does not require seven consecutive dates.

If dates are missing, you may need a calendar table/date series first.

---

# 79. Important Trap: Duplicate Timestamps

If the ordering column is not unique:

```sql
ORDER BY event_time
```

multiple rows may be peers.

For deterministic row sequencing, use a unique secondary key:

```sql
ORDER BY event_time, event_id
```

---

# 80. Important Trap: Window Functions Do Not Automatically Remove Duplicates

Window functions preserve rows.

If a join creates 10 rows, a window function generally operates over those 10 rows.

Therefore:

> Fix unintended row multiplication before applying analytical windows.

---

# 81. Window Functions With Joins

Example:

```sql
SELECT
    o.order_id,
    o.customer_id,
    o.amount,
    SUM(o.amount) OVER (
        PARTITION BY o.customer_id
    ) AS customer_total
FROM orders o
JOIN customers c
    ON c.customer_id = o.customer_id;
```

Be careful if the join is one-to-many and duplicates order rows.

---

# 82. Window Functions With `CASE`

Example:

```sql
SUM(
    CASE
        WHEN status = 'paid' THEN amount
        ELSE 0
    END
) OVER (
    PARTITION BY customer_id
) AS paid_total
```

This creates a conditional partition aggregate.

---

# 83. Conditional Running Total

```sql
SUM(
    CASE
        WHEN status = 'paid' THEN amount
        ELSE 0
    END
) OVER (
    PARTITION BY customer_id
    ORDER BY order_date
    ROWS BETWEEN UNBOUNDED PRECEDING
             AND CURRENT ROW
) AS running_paid_total
```

This combines:

- `CASE`
- `SUM`
- `PARTITION BY`
- `ORDER BY`
- explicit frame

It is a useful advanced interview pattern.

---

# 84. `LAG()` for Growth Detection

Example:

```sql
WITH x AS (
    SELECT
        product_id,
        month,
        revenue,
        LAG(revenue) OVER (
            PARTITION BY product_id
            ORDER BY month
        ) AS previous_revenue
    FROM monthly_revenue
)
SELECT
    product_id,
    month,
    revenue,
    previous_revenue,
    revenue - previous_revenue AS change
FROM x;
```

---

# 85. `LEAD()` for Next Event

```sql
SELECT
    user_id,
    event_time,
    event_type,
    LEAD(event_time) OVER (
        PARTITION BY user_id
        ORDER BY event_time
    ) AS next_event_time
FROM events;
```

Then calculate the time gap according to the DBMS.

---

# 86. Finding the Maximum Row Per Group

A robust pattern:

```sql
WITH ranked AS (
    SELECT
        *,
        ROW_NUMBER() OVER (
            PARTITION BY department
            ORDER BY salary DESC, employee_id
        ) AS rn
    FROM employees
)
SELECT *
FROM ranked
WHERE rn = 1;
```

This returns one deterministic row per department.

---

# 87. Finding All Rows Tied for Maximum

Use:

```sql
WITH ranked AS (
    SELECT
        *,
        RANK() OVER (
            PARTITION BY department
            ORDER BY salary DESC
        ) AS rnk
    FROM employees
)
SELECT *
FROM ranked
WHERE rnk = 1;
```

All employees tied for the highest salary are retained.

---

# 88. Finding the Second Distinct Value

Use:

```sql
DENSE_RANK() OVER (
    PARTITION BY department
    ORDER BY salary DESC
)
```

then:

```sql
WHERE dense_rank = 2
```

This is safer than simply using `ROW_NUMBER()` when duplicate values exist.

---

# 89. Running Maximum

Although not one of the core requested functions, it follows the same window-aggregation pattern:

```sql
MAX(value) OVER (
    ORDER BY date_col
    ROWS BETWEEN UNBOUNDED PRECEDING
             AND CURRENT ROW
) AS running_max
```

Likewise:

```sql
MIN(value) OVER (...)
```

can produce a running minimum.

---

# 90. Partitioned Maximum

```sql
MAX(salary) OVER (
    PARTITION BY department
) AS department_max
```

Then:

```sql
salary = department_max
```

can identify maximum-salary rows.

For all ties, this approach can be preferable to ranking depending on the problem.

---

# 91. Running Distinct Counts

Be careful.

A simple:

```sql
COUNT(DISTINCT user_id) OVER (...)
```

is not supported by every DBMS.

Even where supported, distinct cumulative metrics can have dialect-specific constraints.

For interview questions, first check the DBMS assumptions and whether a simpler deduplication + cumulative strategy is required.

---

# 92. Window Functions and Pagination

A classic alternative to `LIMIT/OFFSET` is row numbering.

```sql
WITH x AS (
    SELECT
        *,
        ROW_NUMBER() OVER (
            ORDER BY employee_id
        ) AS rn
    FROM employees
)
SELECT *
FROM x
WHERE rn BETWEEN 101 AND 120;
```

For modern production systems, keyset/seek pagination can be preferable for large changing datasets, but row numbering is useful conceptually and in interview questions.

---

# 93. Window Functions and Percentiles

Some SQL dialects provide percentile/distribution window functions, such as:

```text
PERCENT_RANK()
CUME_DIST()
NTILE()
```

These are beyond the core scope here but are useful extensions once the fundamental window model is mastered.

---

# 94. Performance Considerations

Window functions may require sorting or partitioning work.

Potential performance factors:

- size of the input
- `PARTITION BY` cardinality
- window `ORDER BY`
- number of different windows
- joins before the window
- indexes and physical execution plan
- amount of data that must be sorted

Do not assume an index automatically eliminates all window-function cost.

Always inspect the execution plan for production performance problems.

---

# 95. Reducing Repeated Work

If several window functions share the same partition/order definition:

```sql
PARTITION BY department
ORDER BY salary DESC
```

some databases can optimize common work.

Still, query readability should remain a priority.

Named windows can also reduce duplication where supported.

---

# 96. Window Function Complexity

A simple conceptual complexity model:

If `n` rows must be processed and the window requires ordering:

```text
Sorting: O(n log n)
Window scan: O(n)
```

So a common overall cost is approximately:

```text
O(n log n)
```

But actual complexity depends on:

- indexes
- partition structure
- database optimizer
- existing physical order
- number of windows
- joins and aggregation before the window

For interviews, understand the sorting requirement rather than memorizing one universal complexity.

---

# 97. Interview Pattern Recognition

When you see:

> "highest/lowest row per group"

Think:

```text
ROW_NUMBER / RANK
```

When you see:

> "top K per category"

Think:

```text
PARTITION BY + ranking + outer filter
```

When you see:

> "previous value"

Think:

```text
LAG
```

When you see:

> "next value"

Think:

```text
LEAD
```

When you see:

> "change from previous"

Think:

```text
LAG + arithmetic
```

When you see:

> "running total"

Think:

```text
SUM OVER ORDER BY
```

When you see:

> "moving average"

Think:

```text
AVG OVER ORDER BY ROWS BETWEEN
```

When you see:

> "average for group while keeping every row"

Think:

```text
AVG OVER PARTITION BY
```

---

# 98. Interview Question: Top 3 Salaries Per Department

```sql
WITH ranked AS (
    SELECT
        employee_id,
        department,
        salary,
        DENSE_RANK() OVER (
            PARTITION BY department
            ORDER BY salary DESC
        ) AS rnk
    FROM employees
)
SELECT
    employee_id,
    department,
    salary
FROM ranked
WHERE rnk <= 3;
```

Why `DENSE_RANK()`?

Because the question is interpreted as the top three **distinct salary levels**.

If exactly three rows are required, use `ROW_NUMBER()` instead.

---

# 99. Interview Question: Latest Order Per Customer

```sql
WITH ranked AS (
    SELECT
        order_id,
        customer_id,
        order_date,
        amount,
        ROW_NUMBER() OVER (
            PARTITION BY customer_id
            ORDER BY order_date DESC, order_id DESC
        ) AS rn
    FROM orders
)
SELECT
    order_id,
    customer_id,
    order_date,
    amount
FROM ranked
WHERE rn = 1;
```

---

# 100. Interview Question: Month-over-Month Change

```sql
WITH x AS (
    SELECT
        month,
        revenue,
        LAG(revenue) OVER (
            ORDER BY month
        ) AS previous_revenue
    FROM monthly_revenue
)
SELECT
    month,
    revenue,
    previous_revenue,
    revenue - previous_revenue AS revenue_change
FROM x;
```

---

# 101. Interview Question: Running Revenue

```sql
SELECT
    month,
    revenue,
    SUM(revenue) OVER (
        ORDER BY month
        ROWS BETWEEN UNBOUNDED PRECEDING
                 AND CURRENT ROW
    ) AS cumulative_revenue
FROM monthly_revenue;
```

---

# 102. Interview Question: Department Average

```sql
SELECT
    employee_id,
    department,
    salary,
    AVG(salary) OVER (
        PARTITION BY department
    ) AS department_average
FROM employees;
```

---

# 103. Interview Question: Employee's Percentage of Department Payroll

```sql
SELECT
    employee_id,
    department,
    salary,
    100.0 * salary /
        NULLIF(
            SUM(salary) OVER (
                PARTITION BY department
            ),
            0
        ) AS payroll_percentage
FROM employees;
```

---

# 104. Interview Question: 7-Row Moving Average

```sql
SELECT
    event_date,
    value,
    AVG(value) OVER (
        ORDER BY event_date
        ROWS BETWEEN 6 PRECEDING
                 AND CURRENT ROW
    ) AS moving_average
FROM metrics;
```

Remember:

> This is a 7-row window, not necessarily seven calendar days.

---

# 105. Interview Question: Identify Status Changes

```sql
WITH x AS (
    SELECT
        user_id,
        event_time,
        status,
        LAG(status) OVER (
            PARTITION BY user_id
            ORDER BY event_time
        ) AS previous_status
    FROM user_status_events
)
SELECT
    user_id,
    event_time,
    status,
    previous_status
FROM x
WHERE previous_status IS NOT NULL
  AND status <> previous_status;
```

---

# 106. Interview Question: First Order Per Customer

```sql
WITH x AS (
    SELECT
        *,
        ROW_NUMBER() OVER (
            PARTITION BY customer_id
            ORDER BY order_date, order_id
        ) AS rn
    FROM orders
)
SELECT *
FROM x
WHERE rn = 1;
```

---

# 107. Interview Question: Last Order Per Customer

```sql
WITH x AS (
    SELECT
        *,
        ROW_NUMBER() OVER (
            PARTITION BY customer_id
            ORDER BY order_date DESC, order_id DESC
        ) AS rn
    FROM orders
)
SELECT *
FROM x
WHERE rn = 1;
```

---

# 108. Interview Question: Compare Each Employee to Department Maximum

```sql
SELECT
    employee_id,
    department,
    salary,
    MAX(salary) OVER (
        PARTITION BY department
    ) AS department_max
FROM employees;
```

Then:

```sql
WITH x AS (
    SELECT
        *,
        MAX(salary) OVER (
            PARTITION BY department
        ) AS department_max
    FROM employees
)
SELECT *
FROM x
WHERE salary = department_max;
```

---

# 109. Interview Question: Running Count Per Customer

```sql
SELECT
    customer_id,
    order_date,
    COUNT(*) OVER (
        PARTITION BY customer_id
        ORDER BY order_date, order_id
        ROWS BETWEEN UNBOUNDED PRECEDING
                 AND CURRENT ROW
    ) AS order_number
FROM orders;
```

This effectively gives each customer's sequential order number.

---

# 110. Interview Question: Running Paid Revenue

```sql
SELECT
    customer_id,
    order_date,
    amount,
    SUM(
        CASE
            WHEN status = 'paid' THEN amount
            ELSE 0
        END
    ) OVER (
        PARTITION BY customer_id
        ORDER BY order_date, order_id
        ROWS BETWEEN UNBOUNDED PRECEDING
                 AND CURRENT ROW
    ) AS running_paid_revenue
FROM orders;
```

---

# 111. GATE/CS Theory — Important Facts

1. Window functions preserve row-level results.
2. `GROUP BY` generally collapses rows.
3. `PARTITION BY` creates independent windows.
4. Window `ORDER BY` defines calculation ordering.
5. Final `ORDER BY` controls displayed ordering.
6. `ROW_NUMBER()` assigns unique sequence numbers.
7. `RANK()` gives ties equal ranks and leaves gaps.
8. `DENSE_RANK()` gives ties equal ranks without gaps.
9. `LAG()` accesses a previous row.
10. `LEAD()` accesses a subsequent row.
11. `FIRST_VALUE()` depends on window ordering/frame.
12. `LAST_VALUE()` can surprise you without an explicit frame.
13. `SUM() OVER (ORDER BY ...)` can produce a running sum.
14. `AVG() OVER (ORDER BY ...)` can produce running or moving averages depending on the frame.
15. `ROWS BETWEEN 6 PRECEDING AND CURRENT ROW` means up to seven physical rows.
16. Window results usually need an outer query/CTE before filtering.
17. `COUNT(*) OVER ()` gives total row count on every result row.
18. `COUNT(*) OVER (PARTITION BY g)` gives group row count while preserving rows.
19. Window functions can operate over grouped results.
20. Duplicate rows caused by joins can affect window calculations.

---

# 112. GATE-Style Solved Question 1 — Ranking

Consider salaries:

```text
salary
------
100
90
90
80
```

Compute:

```sql
ROW_NUMBER() OVER (ORDER BY salary DESC)
RANK()       OVER (ORDER BY salary DESC)
DENSE_RANK() OVER (ORDER BY salary DESC)
```

## Solution

| Salary | Row Number | Rank | Dense Rank |
|---:|---:|---:|---:|
| 100 | 1 | 1 | 1 |
| 90 | 2 | 2 | 2 |
| 90 | 3 | 2 | 2 |
| 80 | 4 | 4 | 3 |

### Answer

```text
ROW_NUMBER: 1,2,3,4
RANK:       1,2,2,4
DENSE_RANK: 1,2,2,3
```

---

# 113. GATE-Style Solved Question 2 — Partitioning

Table:

```text
id | dept | salary
---+------+-------
1  | A    | 10
2  | A    | 20
3  | B    | 30
4  | B    | 50
```

What does:

```sql
AVG(salary) OVER (PARTITION BY dept)
```

produce?

## Solution

Department A:

```text
(10 + 20) / 2 = 15
```

Department B:

```text
(30 + 50) / 2 = 40
```

Result:

```text
id | dept | salary | avg
---+------+--------+----
1  | A    | 10     | 15
2  | A    | 20     | 15
3  | B    | 30     | 40
4  | B    | 50     | 40
```

### Key point

`PARTITION BY` does not remove the original rows.

---

# 114. GATE-Style Solved Question 3 — Running Sum

Given:

```text
day | value
----+------
1   | 10
2   | 20
3   | 5
4   | 15
```

Evaluate:

```sql
SUM(value) OVER (
    ORDER BY day
    ROWS BETWEEN UNBOUNDED PRECEDING
             AND CURRENT ROW
)
```

## Solution

```text
day  value  running_sum
1    10     10
2    20     30
3    5      35
4    15     50
```

### Answer

```text
10, 30, 35, 50
```

---

# 115. GATE-Style Solved Question 4 — `LAG()`

Given:

```text
day | value
----+------
1   | 100
2   | 120
3   | 90
```

Evaluate:

```sql
LAG(value) OVER (ORDER BY day)
```

## Solution

The first row has no previous row:

```text
day  value  lag
1    100    NULL
2    120    100
3    90     120
```

### Answer

```text
NULL, 100, 120
```

---

# 116. GATE-Style Solved Question 5 — Moving Average

Given:

```text
day | value
----+------
1   | 10
2   | 20
3   | 30
4   | 40
```

Evaluate:

```sql
AVG(value) OVER (
    ORDER BY day
    ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
)
```

## Solution

Row 1:

```text
10 / 1 = 10
```

Row 2:

```text
(10 + 20) / 2 = 15
```

Row 3:

```text
(10 + 20 + 30) / 3 = 20
```

Row 4:

```text
(20 + 30 + 40) / 3 = 30
```

### Answer

```text
10, 15, 20, 30
```

---

# 117. GATE-Style Solved Question 6 — Top Row Per Group

Given:

```text
employee | dept | salary
---------+------+-------
A        | X    | 100
B        | X    | 100
C        | X    | 90
D        | Y    | 80
E        | Y    | 70
```

If the requirement is:

> Return **exactly one** highest-paid employee from each department.

Use:

```sql
ROW_NUMBER() OVER (
    PARTITION BY dept
    ORDER BY salary DESC, employee
)
```

Then:

```sql
WHERE rn = 1
```

Result:

```text
A | X | 100
D | Y | 80
```

The tie-breaker ensures deterministic selection.

If instead the requirement is:

> Return **all** highest-paid employees.

Use:

```sql
RANK() OVER (
    PARTITION BY dept
    ORDER BY salary DESC
)
```

and:

```sql
WHERE rnk = 1
```

---

# 118. Advanced GATE-Style Question — Ranking After Grouping

Suppose:

```text
department | salary
-----------+-------
A          | 10
A          | 20
B          | 30
B          | 40
C          | 5
```

Query:

```sql
SELECT
    department,
    SUM(salary) AS total_salary,
    RANK() OVER (
        ORDER BY SUM(salary) DESC
    ) AS rnk
FROM employees
GROUP BY department;
```

First aggregate:

```text
A -> 30
B -> 70
C -> 5
```

Then rank:

```text
B -> 1
A -> 2
C -> 3
```

The window function ranks the grouped result.

---

# 119. Advanced GATE-Style Question — Partitioned Running Total

Given:

```text
customer | day | amount
---------+-----+-------
A        | 1   | 10
A        | 2   | 20
B        | 1   | 100
B        | 2   | 50
```

Query:

```sql
SUM(amount) OVER (
    PARTITION BY customer
    ORDER BY day
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
)
```

Result:

```text
A day 1 -> 10
A day 2 -> 30
B day 1 -> 100
B day 2 -> 150
```

The partition resets the cumulative calculation for each customer.

---

# 120. Interview Practice Set

## Easy

1. Add total employee count to every employee row.
2. Add department employee count.
3. Add department average salary.
4. Number employees by salary.
5. Find the previous transaction using `LAG()`.
6. Find the next transaction using `LEAD()`.

## Medium

7. Find the top 3 employees per department.
8. Find the latest order per customer.
9. Find the second-highest salary per department.
10. Calculate month-over-month revenue change.
11. Calculate a running total per customer.
12. Calculate a 7-row moving average.
13. Find employees earning above department average.
14. Calculate each product's percentage of category revenue.
15. Detect status changes for each user.

## Hard

16. Find the longest gap between consecutive user events.
17. Find the first purchase after a user's previous purchase.
18. Find users whose current transaction is greater than every previous transaction.
19. Find the first date on which cumulative revenue exceeds a threshold.
20. Calculate rolling metrics while handling missing dates.
21. Find consecutive-event streaks using `LAG()` and grouping.
22. Find top K products per category including ties.
23. Combine grouped aggregates with multiple window rankings.
24. Calculate cohort-level running metrics.
25. Build a query that compares current, previous, next, group average, and cumulative total for each entity.

---

# 121. Common Mistakes Checklist

Before submitting a window-function query, check:

- [ ] Did I need `GROUP BY`, or do I need to preserve rows?
- [ ] Is the partition correct?
- [ ] Is the window ordering correct?
- [ ] Is the final output ordering also specified if needed?
- [ ] Do I need a deterministic tie-breaker?
- [ ] Should ties share a rank?
- [ ] Should ranks have gaps?
- [ ] Am I using `ROW_NUMBER`, `RANK`, or `DENSE_RANK` correctly?
- [ ] Is `LAG` looking in the correct direction?
- [ ] Is `LEAD` looking in the correct direction?
- [ ] Do I need an explicit frame?
- [ ] Am I using `ROWS` when I mean physical rows?
- [ ] Is `LAST_VALUE()` behaving as intended?
- [ ] Is my moving window row-based or time-based?
- [ ] Can I filter the window result at this query level?
- [ ] Do I need a CTE/subquery?
- [ ] Could a previous join have multiplied rows?
- [ ] Are NULLs handled correctly?
- [ ] Are duplicate timestamps deterministic?

---

# 122. Master Comparison Table

| Requirement | Best Tool |
|---|---|
| Unique sequential row number | `ROW_NUMBER()` |
| Rank with gaps | `RANK()` |
| Rank without gaps | `DENSE_RANK()` |
| Previous row | `LAG()` |
| Next row | `LEAD()` |
| First ordered value | `FIRST_VALUE()` |
| Last ordered value | `LAST_VALUE()` + explicit frame when necessary |
| Group total, preserve rows | `SUM() OVER (PARTITION BY ...)` |
| Group average, preserve rows | `AVG() OVER (PARTITION BY ...)` |
| Group count, preserve rows | `COUNT() OVER (PARTITION BY ...)` |
| Running sum | `SUM() OVER (ORDER BY ... ROWS ...)` |
| Running average | `AVG() OVER (ORDER BY ... ROWS ...)` |
| Moving average | `AVG() OVER (ORDER BY ... ROWS BETWEEN N PRECEDING AND CURRENT ROW)` |
| Cumulative count | `COUNT() OVER (ORDER BY ... ROWS ...)` |
| Previous-to-current comparison | `LAG()` |
| Current-to-next comparison | `LEAD()` |
| Top 1 per group | `ROW_NUMBER()` + `rn = 1` |
| All tied top rows | `RANK()` + `rnk = 1` |
| Top distinct values | `DENSE_RANK()` |

---

# 123. Final Revision Sheet

## Core syntax

```sql
function(...) OVER (
    PARTITION BY ...
    ORDER BY ...
    ROWS BETWEEN ...
)
```

## Ranking

```sql
ROW_NUMBER()
RANK()
DENSE_RANK()
```

Remember:

```text
ROW_NUMBER -> unique
RANK       -> ties + gaps
DENSE_RANK -> ties + no gaps
```

## Navigation

```sql
LAG()
LEAD()
FIRST_VALUE()
LAST_VALUE()
```

Remember:

```text
LAG  -> previous
LEAD -> next
```

## Partitioned aggregate

```sql
SUM(x) OVER (PARTITION BY g)
AVG(x) OVER (PARTITION BY g)
COUNT(*) OVER (PARTITION BY g)
```

## Running metric

```sql
SUM(x) OVER (
    ORDER BY t
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
)
```

## Moving metric

```sql
AVG(x) OVER (
    ORDER BY t
    ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
)
```

## Top K per group

```sql
WITH x AS (
    SELECT
        *,
        ROW_NUMBER() OVER (
            PARTITION BY group_col
            ORDER BY metric DESC
        ) AS rn
    FROM table_name
)
SELECT *
FROM x
WHERE rn <= K;
```

## Latest row per entity

```sql
WITH x AS (
    SELECT
        *,
        ROW_NUMBER() OVER (
            PARTITION BY entity_id
            ORDER BY timestamp_col DESC, unique_id DESC
        ) AS rn
    FROM table_name
)
SELECT *
FROM x
WHERE rn = 1;
```

## Previous-row comparison

```sql
WITH x AS (
    SELECT
        *,
        LAG(value) OVER (
            PARTITION BY entity_id
            ORDER BY timestamp_col
        ) AS previous_value
    FROM table_name
)
SELECT
    *,
    value - previous_value AS change
FROM x;
```

---

# 124. Mastery Checklist

You should be able to explain and implement all of the following without reference material:

### Core

- [ ] Explain what a window function does.
- [ ] Explain why it differs from `GROUP BY`.
- [ ] Explain `OVER()`.
- [ ] Explain `PARTITION BY`.
- [ ] Explain window `ORDER BY`.
- [ ] Distinguish window ordering from final ordering.

### Ranking

- [ ] `ROW_NUMBER()`
- [ ] `RANK()`
- [ ] `DENSE_RANK()`
- [ ] Tie handling
- [ ] Deterministic tie-breakers
- [ ] Top K per group
- [ ] Latest row per group
- [ ] Highest row with ties

### Navigation

- [ ] `LAG()`
- [ ] `LEAD()`
- [ ] Offsets
- [ ] Default values
- [ ] Partitioned navigation
- [ ] Previous-row comparisons
- [ ] Next-row comparisons
- [ ] Status-change detection

### Value functions

- [ ] `FIRST_VALUE()`
- [ ] `LAST_VALUE()`
- [ ] Explicit window frames
- [ ] `LAST_VALUE()` trap

### Aggregation

- [ ] Partitioned sum
- [ ] Partitioned average
- [ ] Partitioned count
- [ ] Running sum
- [ ] Running average
- [ ] Moving average
- [ ] Cumulative count
- [ ] Running min/max

### Frames

- [ ] `ROWS`
- [ ] `RANGE`
- [ ] `UNBOUNDED PRECEDING`
- [ ] `CURRENT ROW`
- [ ] `N PRECEDING`
- [ ] `N FOLLOWING`
- [ ] `UNBOUNDED FOLLOWING`

### Interview

- [ ] Top K per group
- [ ] Latest row per entity
- [ ] Second highest distinct value
- [ ] Previous transaction
- [ ] Month-over-month change
- [ ] Running total
- [ ] Moving average
- [ ] Above-group-average rows
- [ ] Percentage of group total
- [ ] Status changes
- [ ] Event gaps
- [ ] Streak patterns

---

# 125. Final Interview Rules

Memorize these rules:

1. **Window functions preserve rows.**
2. **`PARTITION BY` resets the window for each group.**
3. **Window `ORDER BY` controls calculation order.**
4. **Final `ORDER BY` controls displayed order.**
5. **`ROW_NUMBER()` gives unique positions.**
6. **`RANK()` gives ties the same rank and leaves gaps.**
7. **`DENSE_RANK()` gives ties the same rank without gaps.**
8. **`LAG()` means previous.**
9. **`LEAD()` means next.**
10. **Running metrics normally need `ORDER BY`.**
11. **Moving metrics need an explicit bounded frame.**
12. **`ROWS` counts physical rows, not calendar time.**
13. **Use a CTE/subquery when filtering a calculated window result.**
14. **Use tie-breakers when deterministic row selection matters.**
15. **Be especially careful with `LAST_VALUE()` and window frames.**
16. **A join that multiplies rows can change window results.**
17. **Window functions can rank aggregated results.**
18. **`COUNT(*) OVER ()` gives the total result-row count on every row.**
19. **`SUM(x) OVER (PARTITION BY g)` gives a group total without collapsing rows.**
20. **Choose the window function based on the exact meaning of "top", "rank", "previous", "next", "running", and "moving".**

---

# 126. Final One-Page Mental Model

```text
WINDOW FUNCTIONS
│
├── OVER()
│   └── defines the window
│
├── PARTITION BY
│   └── splits rows into independent groups
│
├── ORDER BY
│   └── defines sequence within the window
│
├── RANKING
│   ├── ROW_NUMBER()
│   ├── RANK()
│   └── DENSE_RANK()
│
├── NAVIGATION
│   ├── LAG()
│   ├── LEAD()
│   ├── FIRST_VALUE()
│   └── LAST_VALUE()
│
├── AGGREGATION
│   ├── SUM() OVER(...)
│   ├── AVG() OVER(...)
│   ├── COUNT() OVER(...)
│   ├── MIN() OVER(...)
│   └── MAX() OVER(...)
│
└── FRAMES
    ├── UNBOUNDED PRECEDING
    ├── N PRECEDING
    ├── CURRENT ROW
    ├── N FOLLOWING
    └── UNBOUNDED FOLLOWING
```

Core recognition:

```text
Top K per group        -> PARTITION + RANKING
Previous row           -> LAG
Next row               -> LEAD
Group statistic        -> PARTITION BY
Running statistic      -> ORDER BY + UNBOUNDED PRECEDING
Moving statistic       -> ORDER BY + bounded ROWS frame
Latest row per group   -> ROW_NUMBER + DESC + tie-breaker
All tied maximum rows  -> RANK + rank = 1
Second distinct value  -> DENSE_RANK + rank = 2
```

**Master this topic before moving on. Window functions are one of the highest-value SQL skills for FAANG/product-company interviews and analytical SQL.**
