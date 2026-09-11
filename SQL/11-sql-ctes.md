# SQL Topic 11 — Common Table Expressions (CTEs)

> **Overall repository topic:** 35  
> **SQL topic:** 11  
> **Priority:** Very High  
> **Interview relevance:** Very High  
> **Difficulty:** Medium → Hard  
> **Prerequisite:** Window Functions, Aggregation, Joins, Subqueries

---

# 1. What Is a CTE?

A **Common Table Expression (CTE)** is a named temporary result set defined using `WITH` and referenced by the query that follows it.

Basic syntax:

```sql
WITH cte_name AS (
    SELECT ...
)
SELECT ...
FROM cte_name;
```

A CTE is primarily a **query-organization and composability mechanism**.

It lets you break a complicated SQL query into logical stages.

---

# 2. Basic Example

Suppose we want employees earning more than the company average.

Without a CTE:

```sql
SELECT *
FROM employees
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
);
```

With a CTE:

```sql
WITH company_avg AS (
    SELECT AVG(salary) AS avg_salary
    FROM employees
)
SELECT e.*
FROM employees e
CROSS JOIN company_avg a
WHERE e.salary > a.avg_salary;
```

The CTE gives the intermediate result a name.

---

# 3. `WITH`

The `WITH` clause appears before the main query.

```sql
WITH cte_name AS (
    SELECT ...
)
SELECT ...
FROM cte_name;
```

The CTE exists for the statement in which it is defined.

It is not normally a permanent table.

---

# 4. CTE Scope

A regular CTE has statement-level scope.

Example:

```sql
WITH x AS (
    SELECT *
    FROM employees
)
SELECT *
FROM x;
```

After this statement finishes, `x` is not a permanent database object.

You cannot generally do:

```sql
SELECT *
FROM x;
```

in a later independent statement.

---

# 5. CTE Is Not Automatically a Materialized Table

A CTE is a logical query expression.

Do not assume:

```text
CTE = temporary physical table
```

A database optimizer may inline it, materialize it, or otherwise transform the query depending on the DBMS and execution plan.

Therefore:

> A CTE is primarily a way to express a query, not a guaranteed storage mechanism.

---

# 6. Why Use CTEs?

CTEs are useful for:

- breaking complex queries into stages
- improving readability
- reusing an intermediate result
- ranking and then filtering
- aggregation followed by another calculation
- joining derived results
- recursive hierarchical queries
- organizing multi-step analytical SQL

---

# 7. CTE Structure

A query can be mentally viewed as:

```text
CTE 1
  ↓
CTE 2
  ↓
CTE 3
  ↓
Final SELECT
```

Each stage can build on earlier stages.

---

# 8. Multiple CTEs

You can define multiple CTEs after one `WITH`.

```sql
WITH
a AS (
    SELECT ...
),
b AS (
    SELECT ...
    FROM a
),
c AS (
    SELECT ...
    FROM b
)
SELECT ...
FROM c;
```

CTEs are separated by commas.

Do not write multiple `WITH` keywords:

```sql
-- Usually wrong
WITH a AS (...)
WITH b AS (...)
SELECT ...
```

Instead:

```sql
WITH
a AS (...),
b AS (...)
SELECT ...
```

---

# 9. Multiple CTE Example

Suppose:

1. calculate customer revenue
2. calculate average revenue
3. select customers above average

```sql
WITH customer_revenue AS (
    SELECT
        customer_id,
        SUM(amount) AS revenue
    FROM orders
    GROUP BY customer_id
),
average_revenue AS (
    SELECT
        AVG(revenue) AS avg_revenue
    FROM customer_revenue
)
SELECT
    cr.customer_id,
    cr.revenue
FROM customer_revenue cr
CROSS JOIN average_revenue ar
WHERE cr.revenue > ar.avg_revenue;
```

This is much easier to reason about than nesting everything into one expression.

---

# 10. CTE Dependency Order

A later CTE can reference an earlier CTE.

```sql
WITH
a AS (...),
b AS (
    SELECT *
    FROM a
),
c AS (
    SELECT *
    FROM b
)
SELECT *
FROM c;
```

Conceptually:

```text
a → b → c → final query
```

A regular non-recursive CTE generally cannot reference a later CTE that has not yet been defined.

---

# 11. CTE Names

Use descriptive names.

Good:

```sql
WITH customer_revenue AS (...)
```

Better than:

```sql
WITH x AS (...)
```

Especially in large interview queries.

Good names expose the logical purpose of each stage.

---

# 12. CTE vs Subquery

A subquery can be written inline:

```sql
SELECT *
FROM (
    SELECT
        customer_id,
        SUM(amount) AS revenue
    FROM orders
    GROUP BY customer_id
) x
WHERE revenue > 10000;
```

Equivalent CTE:

```sql
WITH customer_revenue AS (
    SELECT
        customer_id,
        SUM(amount) AS revenue
    FROM orders
    GROUP BY customer_id
)
SELECT *
FROM customer_revenue
WHERE revenue > 10000;
```

Both express a derived result.

---

# 13. CTE vs Subquery — Main Difference

The biggest practical difference is **organization and readability**, not automatically performance.

### Subquery

```sql
FROM (
    SELECT ...
) x
```

The intermediate query is embedded inside the main query.

### CTE

```sql
WITH x AS (
    SELECT ...
)
SELECT ...
FROM x;
```

The intermediate stage is named before the main query.

---

# 14. Does CTE Always Improve Performance?

No.

A CTE does not inherently make a query faster.

Depending on the DBMS and query:

- it may be inlined
- it may be materialized
- it may be optimized differently
- it may have no meaningful performance difference from an equivalent subquery

For performance questions:

> Inspect the execution plan rather than assuming a CTE is faster.

---

# 15. When CTEs Are Better Than Subqueries

Prefer a CTE when:

- the query has multiple logical stages
- an intermediate result has a meaningful name
- a window function must be filtered
- several calculations build on one another
- the same intermediate relation is referenced multiple times where the DBMS permits/benefits
- recursion is required

For a tiny one-off expression, an inline subquery can be simpler.

---

# 16. CTE + Window Functions

This is one of the highest-value interview combinations.

Problem:

> Find the highest-paid employee in every department.

First calculate row numbers:

```sql
WITH ranked AS (
    SELECT
        employee_id,
        department,
        salary,
        ROW_NUMBER() OVER (
            PARTITION BY department
            ORDER BY salary DESC, employee_id
        ) AS rn
    FROM employees
)
SELECT
    employee_id,
    department,
    salary
FROM ranked
WHERE rn = 1;
```

The CTE creates a stage where the window result becomes an ordinary column that can be filtered.

---

# 17. CTE + `RANK()`

Find all employees tied for the highest salary in each department:

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
WHERE rnk = 1;
```

Use `RANK()` when all tied top rows should remain.

---

# 18. CTE + `DENSE_RANK()`

Find the second-highest distinct salary per department:

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

---

# 19. CTE + `LAG()`

Find month-over-month revenue changes:

```sql
WITH revenue_with_previous AS (
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
FROM revenue_with_previous;
```

The CTE makes the navigation result reusable.

---

# 20. CTE + Running Total

```sql
WITH monthly AS (
    SELECT
        month,
        SUM(amount) AS revenue
    FROM sales
    GROUP BY month
)
SELECT
    month,
    revenue,
    SUM(revenue) OVER (
        ORDER BY month
        ROWS BETWEEN UNBOUNDED PRECEDING
             AND CURRENT ROW
    ) AS cumulative_revenue
FROM monthly;
```

This demonstrates an important sequence:

```text
raw rows
  ↓
GROUP BY
  ↓
window calculation
  ↓
final result
```

---

# 21. CTE + Aggregation

CTEs are excellent when an aggregate result becomes the input to another aggregate/calculation.

Example:

```sql
WITH customer_revenue AS (
    SELECT
        customer_id,
        SUM(amount) AS revenue
    FROM orders
    GROUP BY customer_id
)
SELECT
    AVG(revenue) AS average_customer_revenue
FROM customer_revenue;
```

The first stage aggregates orders by customer.

The second stage aggregates the customer-level result.

---

# 22. Multi-Level Aggregation

Example:

> Find the average order value per customer, then find the average of those customer averages.

```sql
WITH customer_avg AS (
    SELECT
        customer_id,
        AVG(amount) AS avg_order_value
    FROM orders
    GROUP BY customer_id
)
SELECT
    AVG(avg_order_value)
FROM customer_avg;
```

This is not necessarily the same as:

```sql
SELECT AVG(amount)
FROM orders;
```

because customers may have different numbers of orders.

This distinction is important.

---

# 23. CTE + Aggregation + Window Function

A common analytical pattern:

```sql
WITH customer_revenue AS (
    SELECT
        customer_id,
        SUM(amount) AS revenue
    FROM orders
    GROUP BY customer_id
)
SELECT
    customer_id,
    revenue,
    RANK() OVER (
        ORDER BY revenue DESC
    ) AS revenue_rank
FROM customer_revenue;
```

Stages:

```text
orders
  ↓
customer aggregation
  ↓
customer ranking
```

---

# 24. CTE + Joins

Example:

```sql
WITH customer_revenue AS (
    SELECT
        customer_id,
        SUM(amount) AS revenue
    FROM orders
    GROUP BY customer_id
)
SELECT
    c.customer_id,
    c.name,
    COALESCE(cr.revenue, 0) AS revenue
FROM customers c
LEFT JOIN customer_revenue cr
    ON cr.customer_id = c.customer_id;
```

The CTE pre-aggregates orders before joining them to customers.

This can also prevent unwanted row multiplication.

---

# 25. Why Pre-Aggregation Matters

Suppose:

```text
customers
orders
payments
```

If you directly join:

```text
customers
    ↓
orders
    ↓
payments
```

one order can match multiple payments and one customer can have multiple orders.

This can multiply rows.

Instead:

```text
orders → aggregate by customer
payments → aggregate by customer
                    ↓
                 join
```

Example:

```sql
WITH order_totals AS (
    SELECT
        customer_id,
        SUM(amount) AS order_total
    FROM orders
    GROUP BY customer_id
),
payment_totals AS (
    SELECT
        customer_id,
        SUM(amount) AS payment_total
    FROM payments
    GROUP BY customer_id
)
SELECT
    c.customer_id,
    COALESCE(o.order_total, 0) AS order_total,
    COALESCE(p.payment_total, 0) AS payment_total
FROM customers c
LEFT JOIN order_totals o
    ON o.customer_id = c.customer_id
LEFT JOIN payment_totals p
    ON p.customer_id = c.customer_id;
```

---

# 26. Multiple CTEs + Joins

A highly reusable interview pattern:

```sql
WITH
sales AS (
    SELECT
        customer_id,
        SUM(amount) AS total_sales
    FROM orders
    GROUP BY customer_id
),
refunds AS (
    SELECT
        customer_id,
        SUM(amount) AS total_refunds
    FROM refunds
    GROUP BY customer_id
)
SELECT
    c.customer_id,
    COALESCE(s.total_sales, 0) AS total_sales,
    COALESCE(r.total_refunds, 0) AS total_refunds,
    COALESCE(s.total_sales, 0)
      - COALESCE(r.total_refunds, 0) AS net_revenue
FROM customers c
LEFT JOIN sales s
    ON s.customer_id = c.customer_id
LEFT JOIN refunds r
    ON r.customer_id = c.customer_id;
```

This is much safer than joining raw orders and raw refunds first.

---

# 27. CTEs as a Query Pipeline

A complex analytical query can often be designed as:

```text
CTE 1: clean/filter
       ↓
CTE 2: aggregate
       ↓
CTE 3: window/rank
       ↓
CTE 4: join/enrich
       ↓
final filter/output
```

This is a useful interview strategy.

Do not try to write the entire query at once.

---

# 28. Recursive CTEs

A **recursive CTE** allows a CTE to reference itself.

It is useful for hierarchical or graph-like data.

Typical examples:

- organizational hierarchies
- category trees
- folder structures
- bill of materials
- parent-child relationships
- generating sequences
- graph traversal in SQL dialects that support it

---

# 29. Recursive CTE Structure

General conceptual form:

```sql
WITH RECURSIVE cte AS (
    -- anchor member
    SELECT ...

    UNION ALL

    -- recursive member
    SELECT ...
    FROM cte
    ...
)
SELECT *
FROM cte;
```

There are two key components:

1. **Anchor member**
2. **Recursive member**

---

# 30. Anchor Member

The anchor query produces the initial rows.

Example:

```sql
SELECT
    employee_id,
    manager_id,
    name,
    0 AS level
FROM employees
WHERE manager_id IS NULL
```

These are the roots.

---

# 31. Recursive Member

The recursive query uses rows already produced by the CTE to find the next level.

```sql
SELECT
    e.employee_id,
    e.manager_id,
    e.name,
    h.level + 1
FROM employees e
JOIN org_chart h
    ON e.manager_id = h.employee_id
```

The recursive CTE continues until the recursive member produces no new rows or a DBMS-specific recursion limit is reached.

---

# 32. Recursive Organizational Hierarchy

Suppose:

```text
Alice
 ├── Bob
 │    └── David
 └── Carol
```

Table:

```text
employee | manager
---------+--------
Alice    | NULL
Bob      | Alice
Carol    | Alice
David    | Bob
```

Recursive query:

```sql
WITH RECURSIVE org AS (
    SELECT
        employee,
        manager,
        0 AS level
    FROM employees
    WHERE manager IS NULL

    UNION ALL

    SELECT
        e.employee,
        e.manager,
        o.level + 1
    FROM employees e
    JOIN org o
        ON e.manager = o.employee
)
SELECT *
FROM org;
```

Conceptually:

```text
Alice  0
Bob    1
Carol  1
David  2
```

---

# 33. Recursive CTE Iteration Model

Think of recursion as:

```text
Anchor
  ↓
Level 0
  ↓
Recursive step
  ↓
Level 1
  ↓
Recursive step
  ↓
Level 2
  ↓
...
  ↓
No more rows
```

This is similar to breadth-by-level traversal for many hierarchical queries.

Exact duplicate/cycle behavior depends on the query and DBMS.

---

# 34. Recursive CTE for Number Generation

A conceptual sequence generator:

```sql
WITH RECURSIVE numbers AS (
    SELECT 1 AS n

    UNION ALL

    SELECT n + 1
    FROM numbers
    WHERE n < 10
)
SELECT n
FROM numbers;
```

Produces:

```text
1
2
3
...
10
```

The stopping condition is essential.

---

# 35. Recursive CTE Must Have a Termination Condition

Bad:

```sql
WITH RECURSIVE numbers AS (
    SELECT 1
    UNION ALL
    SELECT n + 1
    FROM numbers
)
SELECT *
FROM numbers;
```

There is no logical stopping condition.

This can lead to unbounded recursion until the DBMS stops it through a recursion limit/error.

Correct:

```sql
WHERE n < 10
```

---

# 36. `UNION ALL` in Recursive CTEs

Recursive CTEs commonly use:

```sql
anchor
UNION ALL
recursive_member
```

because each recursive iteration adds rows.

Some DBMSs support or permit alternatives, but the exact recursive syntax and restrictions are dialect-specific.

For interviews, recognize the standard conceptual structure.

---

# 37. Recursive CTE Cycle Problem

Hierarchical data can contain cycles:

```text
A → B → C → A
```

A naive recursive traversal can repeatedly revisit the same nodes.

Therefore recursive graph/hierarchy queries may need:

- cycle detection
- visited-node tracking
- recursion depth limits
- DBMS-specific cycle features

Do not assume every parent-child table is acyclic.

---

# 38. Recursive CTE for Hierarchy Depth

A useful pattern:

```sql
WITH RECURSIVE hierarchy AS (
    SELECT
        employee_id,
        manager_id,
        0 AS depth
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    SELECT
        e.employee_id,
        e.manager_id,
        h.depth + 1
    FROM employees e
    JOIN hierarchy h
        ON e.manager_id = h.employee_id
)
SELECT *
FROM hierarchy;
```

The `depth` column tracks hierarchy level.

---

# 39. Recursive CTE With Path

You can carry a path through recursion.

Conceptually:

```sql
WITH RECURSIVE hierarchy AS (
    SELECT
        employee_id,
        manager_id,
        CAST(employee_id AS VARCHAR(...)) AS path
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    SELECT
        e.employee_id,
        e.manager_id,
        CONCAT(h.path, '>', e.employee_id)
    FROM employees e
    JOIN hierarchy h
        ON e.manager_id = h.employee_id
)
SELECT *
FROM hierarchy;
```

Exact string type and concatenation syntax varies by DBMS.

Path tracking can help with:

- displaying hierarchy
- cycle detection
- ancestor tracing

---

# 40. CTE + Recursive Hierarchy + Aggregation

You can first expand a hierarchy and then aggregate.

For example:

```sql
WITH RECURSIVE hierarchy AS (
    SELECT
        employee_id,
        manager_id,
        employee_id AS root_id
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    SELECT
        e.employee_id,
        e.manager_id,
        h.root_id
    FROM employees e
    JOIN hierarchy h
        ON e.manager_id = h.employee_id
)
SELECT
    root_id,
    COUNT(*) AS employees_in_tree
FROM hierarchy
GROUP BY root_id;
```

The exact desired semantics depend on whether the root itself should be counted.

---

# 41. CTE + Window Functions: Common Pipeline

A very common interview design:

```sql
WITH base AS (
    SELECT ...
),
ranked AS (
    SELECT
        ...,
        ROW_NUMBER() OVER (...) AS rn
    FROM base
)
SELECT ...
FROM ranked
WHERE rn = 1;
```

Why this is powerful:

```text
Base data
  ↓
Window calculation
  ↓
Filtering
```

---

# 42. CTE + Aggregation: Common Pipeline

```sql
WITH totals AS (
    SELECT
        group_id,
        SUM(value) AS total
    FROM table_name
    GROUP BY group_id
)
SELECT *
FROM totals
WHERE total > 10000;
```

This separates:

```text
aggregation
```

from:

```text
group-level filtering
```

---

# 43. CTE + Join: Common Pipeline

```sql
WITH totals AS (
    SELECT
        customer_id,
        SUM(amount) AS total
    FROM orders
    GROUP BY customer_id
)
SELECT
    c.customer_id,
    c.name,
    t.total
FROM customers c
LEFT JOIN totals t
    ON t.customer_id = c.customer_id;
```

This is especially useful when the aggregated CTE has one row per entity.

---

# 44. CTE + Multiple Joins

```sql
WITH
orders_by_customer AS (
    SELECT
        customer_id,
        SUM(amount) AS orders_total
    FROM orders
    GROUP BY customer_id
),
tickets_by_customer AS (
    SELECT
        customer_id,
        COUNT(*) AS ticket_count
    FROM support_tickets
    GROUP BY customer_id
)
SELECT
    c.customer_id,
    c.name,
    COALESCE(o.orders_total, 0) AS orders_total,
    COALESCE(t.ticket_count, 0) AS ticket_count
FROM customers c
LEFT JOIN orders_by_customer o
    ON o.customer_id = c.customer_id
LEFT JOIN tickets_by_customer t
    ON t.customer_id = c.customer_id;
```

The important principle:

> Pre-aggregate independent one-to-many sources before joining them.

---

# 45. CTE + `HAVING`

You can use `HAVING` inside a CTE:

```sql
WITH active_customers AS (
    SELECT
        customer_id,
        COUNT(*) AS order_count
    FROM orders
    GROUP BY customer_id
    HAVING COUNT(*) >= 5
)
SELECT *
FROM active_customers;
```

This makes the CTE itself a filtered aggregate relation.

---

# 46. CTE + `CASE`

```sql
WITH classified AS (
    SELECT
        employee_id,
        salary,
        CASE
            WHEN salary >= 100000 THEN 'HIGH'
            WHEN salary >= 50000 THEN 'MEDIUM'
            ELSE 'LOW'
        END AS salary_band
    FROM employees
)
SELECT
    salary_band,
    COUNT(*) AS employees
FROM classified
GROUP BY salary_band;
```

Pipeline:

```text
row classification
   ↓
aggregation
```

---

# 47. CTE + `DISTINCT`

```sql
WITH unique_customers AS (
    SELECT DISTINCT
        customer_id
    FROM orders
)
SELECT COUNT(*)
FROM unique_customers;
```

Although there may be other ways to write this, a CTE can make a multi-stage transformation explicit.

---

# 48. CTE + Set Operations

CTEs can feed set operations:

```sql
WITH a AS (
    SELECT customer_id
    FROM orders
),
b AS (
    SELECT customer_id
    FROM returns
)
SELECT customer_id
FROM a
EXCEPT
SELECT customer_id
FROM b;
```

This combines the earlier set-operation concepts with CTE organization.

---

# 49. CTE + Subquery

A CTE can itself contain subqueries:

```sql
WITH customer_data AS (
    SELECT
        customer_id,
        (
            SELECT COUNT(*)
            FROM orders o
            WHERE o.customer_id = c.customer_id
        ) AS order_count
    FROM customers c
)
SELECT *
FROM customer_data;
```

The goal should still be readability rather than adding layers unnecessarily.

---

# 50. Nested CTE Design

For difficult interview questions, name stages based on what they represent.

Example:

```sql
WITH
filtered_orders AS (...),
customer_totals AS (...),
ranked_customers AS (...),
top_customers AS (...)
SELECT ...
FROM top_customers;
```

This is easier to debug than:

```sql
WITH a AS (...),
b AS (...),
c AS (...),
d AS (...)
SELECT ...
FROM d;
```

---

# 51. CTE Debugging Strategy

If a complex query fails, temporarily inspect each stage.

Start with:

```sql
WITH filtered_orders AS (
    ...
)
SELECT *
FROM filtered_orders;
```

Then add:

```sql
WITH filtered_orders AS (...),
customer_totals AS (...)
SELECT *
FROM customer_totals;
```

Then add the next stage.

This turns one complicated problem into smaller testable steps.

---

# 52. CTE Materialization — Interview Concept

Some DBMSs provide explicit controls or optimizer behavior around CTE materialization.

Do not assume:

```text
CTE → always materialized
```

or:

```text
CTE → always inlined
```

The exact behavior is DBMS-specific.

For performance questions, state:

> CTE optimization/materialization behavior depends on the database system and execution plan.

---

# 53. Recursive CTE vs Normal CTE

| Feature | Normal CTE | Recursive CTE |
|---|---|---|
| Self-reference | No | Yes |
| `WITH` | Yes | Yes |
| Multiple stages | Yes | Yes |
| Hierarchies | Possible but awkward | Natural |
| Sequence generation | Limited | Common use |
| Graph traversal | Not naturally recursive | Possible |
| Termination condition | Not applicable | Essential |
| Cycle risk | No recursive cycle | Yes |

---

# 54. CTE vs Temporary Table

A CTE:

- normally exists for one statement
- is defined inside the query
- is primarily a query-expression construct

A temporary table:

- is a database object with temporary lifetime
- can often be referenced by multiple statements during its lifetime
- can generally have indexes/statistics depending on DBMS
- has storage/materialization semantics

Do not treat the two as interchangeable.

---

# 55. CTE vs View

A view is a persistent database object.

A normal CTE is statement-scoped.

Conceptually:

```text
CTE  → temporary named query expression
View → persistent named query definition
```

A materialized view is different again because it stores materialized results.

---

# 56. CTE vs Subquery vs Temp Table vs View

| Feature | CTE | Subquery | Temp Table | View |
|---|---|---|---|---|
| Named | Yes | Sometimes alias only | Yes | Yes |
| Statement-scoped | Usually | Yes | No | No |
| Persistent object | No | No | Temporary object | Yes |
| Recursive | Yes, if supported | Not directly | Can participate in recursion logic | Depends on DBMS/features |
| Multiple statements | No | No | Yes | Yes |
| Main purpose | Query organization | Inline derivation | Intermediate stored data | Reusable query object |

---

# 57. Common Mistake: Thinking CTE Automatically Improves Performance

Wrong assumption:

```text
CTE = faster
```

Correct:

```text
CTE = clearer query structure
```

Performance depends on the optimizer, DBMS, data, indexes, and execution plan.

---

# 58. Common Mistake: Overusing CTEs

Do not create ten CTEs for a query that needs two.

Bad style:

```text
CTE 1 → just renames one column
CTE 2 → just selects all columns
CTE 3 → just filters one condition
...
```

Use CTEs when they make the logical stages clearer.

---

# 59. Common Mistake: Filtering a Window Function Too Early

Wrong:

```sql
SELECT
    *,
    ROW_NUMBER() OVER (...) AS rn
FROM employees
WHERE rn = 1;
```

Correct:

```sql
WITH ranked AS (
    SELECT
        *,
        ROW_NUMBER() OVER (...) AS rn
    FROM employees
)
SELECT *
FROM ranked
WHERE rn = 1;
```

---

# 60. Common Mistake: Aggregating After a Multiplying Join

Suppose orders and payments are both one-to-many.

This can be dangerous:

```sql
SELECT
    customer_id,
    SUM(order_amount),
    SUM(payment_amount)
FROM orders
JOIN payments USING (customer_id)
GROUP BY customer_id;
```

The join may multiply order/payment rows.

Safer:

```sql
WITH order_totals AS (...),
payment_totals AS (...)
SELECT ...
FROM order_totals
JOIN payment_totals ...;
```

---

# 61. Common Mistake: Recursive CTE Without Progress

A recursive member must make meaningful progress toward termination.

Bad logic can repeatedly produce the same row.

Always verify:

```text
anchor
→ recursive transition
→ termination
```

---

# 62. Common Mistake: Ignoring Cycles

For recursive hierarchy/graph queries, test whether the data can contain:

```text
A → B → C → A
```

If cycles are possible, incorporate appropriate cycle protection.

---

# 63. Common Mistake: Assuming Recursive Syntax Is Identical Everywhere

Recursive CTE support and syntax vary between database systems.

Examples of differences can include:

- whether `RECURSIVE` is required
- recursion limits
- cycle-detection syntax
- restrictions on recursive members
- data-type requirements

For interviews, identify the assumed SQL dialect when exact syntax matters.

---

# 64. Query Construction Strategy

For a difficult SQL interview problem:

### Step 1 — Identify the grain

Ask:

> What should one final row represent?

Examples:

```text
one row per customer
one row per department
one row per customer-month
one row per employee
```

### Step 2 — Build that stage

Use a CTE if useful.

### Step 3 — Aggregate

If needed:

```sql
GROUP BY
```

### Step 4 — Add window calculations

For:

- ranking
- previous/next
- cumulative values

### Step 5 — Join enrichment data

### Step 6 — Filter the final result

This prevents many grain-related errors.

---

# 65. Advanced Pattern: Filter → Aggregate → Rank

Problem:

> Find the top 5 customers by paid revenue.

```sql
WITH paid_orders AS (
    SELECT
        customer_id,
        amount
    FROM orders
    WHERE status = 'paid'
),
customer_revenue AS (
    SELECT
        customer_id,
        SUM(amount) AS revenue
    FROM paid_orders
    GROUP BY customer_id
),
ranked AS (
    SELECT
        customer_id,
        revenue,
        ROW_NUMBER() OVER (
            ORDER BY revenue DESC, customer_id
        ) AS rn
    FROM customer_revenue
)
SELECT *
FROM ranked
WHERE rn <= 5;
```

Pipeline:

```text
filter
  ↓
aggregate
  ↓
rank
  ↓
filter rank
```

---

# 66. Advanced Pattern: Aggregate → Window → Join

```sql
WITH customer_revenue AS (
    SELECT
        customer_id,
        SUM(amount) AS revenue
    FROM orders
    GROUP BY customer_id
),
ranked AS (
    SELECT
        customer_id,
        revenue,
        RANK() OVER (
            ORDER BY revenue DESC
        ) AS revenue_rank
    FROM customer_revenue
)
SELECT
    r.customer_id,
    c.name,
    r.revenue,
    r.revenue_rank
FROM ranked r
JOIN customers c
    ON c.customer_id = r.customer_id;
```

---

# 67. Advanced Pattern: Multiple Independent Aggregates

```sql
WITH
sales AS (
    SELECT
        customer_id,
        SUM(amount) AS sales
    FROM orders
    GROUP BY customer_id
),
returns AS (
    SELECT
        customer_id,
        SUM(amount) AS returns
    FROM returns
    GROUP BY customer_id
),
tickets AS (
    SELECT
        customer_id,
        COUNT(*) AS tickets
    FROM support_tickets
    GROUP BY customer_id
)
SELECT
    c.customer_id,
    COALESCE(s.sales, 0) AS sales,
    COALESCE(r.returns, 0) AS returns,
    COALESCE(t.tickets, 0) AS tickets
FROM customers c
LEFT JOIN sales s
    ON s.customer_id = c.customer_id
LEFT JOIN returns r
    ON r.customer_id = c.customer_id
LEFT JOIN tickets t
    ON t.customer_id = c.customer_id;
```

This pattern is highly reusable in analytics interviews.

---

# 68. Advanced Pattern: CTE + Window + Conditional Aggregation

```sql
WITH customer_month AS (
    SELECT
        customer_id,
        month,
        SUM(
            CASE
                WHEN status = 'paid' THEN amount
                ELSE 0
            END
        ) AS paid_revenue
    FROM orders
    GROUP BY customer_id, month
)
SELECT
    customer_id,
    month,
    paid_revenue,
    SUM(paid_revenue) OVER (
        PARTITION BY customer_id
        ORDER BY month
        ROWS BETWEEN UNBOUNDED PRECEDING
             AND CURRENT ROW
    ) AS cumulative_paid_revenue
FROM customer_month;
```

---

# 69. Advanced Pattern: CTE + `LAG()` + Filter

Find users whose current purchase is greater than their previous purchase:

```sql
WITH purchases AS (
    SELECT
        user_id,
        purchase_date,
        amount,
        LAG(amount) OVER (
            PARTITION BY user_id
            ORDER BY purchase_date
        ) AS previous_amount
    FROM purchases_table
)
SELECT *
FROM purchases
WHERE previous_amount IS NOT NULL
  AND amount > previous_amount;
```

---

# 70. Advanced Pattern: CTE + Multiple Windows

```sql
WITH metrics AS (
    SELECT
        customer_id,
        order_date,
        amount,

        LAG(amount) OVER (
            PARTITION BY customer_id
            ORDER BY order_date
        ) AS previous_amount,

        SUM(amount) OVER (
            PARTITION BY customer_id
            ORDER BY order_date
            ROWS BETWEEN UNBOUNDED PRECEDING
                 AND CURRENT ROW
        ) AS running_amount,

        AVG(amount) OVER (
            PARTITION BY customer_id
        ) AS customer_average
    FROM orders
)
SELECT *
FROM metrics;
```

One stage produces several analytical features.

---

# 71. Advanced Pattern: Recursive Hierarchy + Final Join

```sql
WITH RECURSIVE hierarchy AS (
    SELECT
        employee_id,
        manager_id,
        employee_id AS root_manager,
        0 AS depth
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    SELECT
        e.employee_id,
        e.manager_id,
        h.root_manager,
        h.depth + 1
    FROM employees e
    JOIN hierarchy h
        ON e.manager_id = h.employee_id
)
SELECT
    h.employee_id,
    e.name,
    h.root_manager,
    h.depth
FROM hierarchy h
JOIN employees e
    ON e.employee_id = h.employee_id;
```

---

# 72. Recursive CTE: Anchor vs Recursive Member

For an exam question, identify:

```text
Anchor member
    ↓
initial rows

UNION ALL

Recursive member
    ↓
rows derived from previous iteration
```

The recursive member references the CTE itself.

---

# 73. Recursive CTE and Depth

A depth column is useful:

```sql
0 AS depth
```

then:

```sql
depth + 1
```

at every recursion.

It can also provide a safety mechanism:

```sql
WHERE depth < 20
```

if a maximum traversal depth is appropriate.

---

# 74. GATE/CS Theory — Core Facts

1. `WITH` introduces a CTE.
2. A normal CTE is scoped to its statement.
3. A CTE is not inherently a physical temporary table.
4. Multiple CTEs are separated by commas.
5. Later CTEs can normally reference earlier CTEs.
6. CTEs can contain aggregation.
7. CTEs can contain joins.
8. CTEs can contain window functions.
9. A CTE can be joined by the final query.
10. A recursive CTE can reference itself.
11. Recursive CTEs have an anchor member.
12. Recursive CTEs have a recursive member.
13. Recursive queries require termination/progress.
14. Cycles can cause problems in recursive hierarchy traversal.
15. CTE vs subquery is primarily a query-organization distinction; performance is DBMS/plan dependent.
16. A CTE can turn a window-function result into a relation that can be filtered by an outer query.
17. Pre-aggregating independent one-to-many sources can prevent join multiplication.
18. CTE materialization behavior is database-specific.
19. A CTE is not the same thing as a view.
20. A CTE is not the same thing as a temporary table.

---

# 75. GATE-Style Solved Question 1 — Basic CTE

Consider:

```sql
WITH x AS (
    SELECT
        department,
        AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department
)
SELECT *
FROM x
WHERE avg_salary > 50000;
```

What does `x` represent?

## Solution

The CTE first groups employees by department.

Therefore `x` contains approximately:

```text
one row per department
```

with:

```text
department
avg_salary
```

The final query filters those department-level rows.

### Answer

`x` is a derived relation containing department-level average salaries.

---

# 76. GATE-Style Solved Question 2 — Multiple CTEs

Given:

```sql
WITH a AS (
    SELECT 10 AS x
),
b AS (
    SELECT x + 5 AS y
    FROM a
)
SELECT y
FROM b;
```

What is returned?

## Solution

First:

```text
a.x = 10
```

Then:

```text
b.y = 10 + 5 = 15
```

### Answer

```text
15
```

---

# 77. GATE-Style Solved Question 3 — CTE + Window

Given:

```sql
WITH x AS (
    SELECT
        employee_id,
        salary,
        ROW_NUMBER() OVER (
            ORDER BY salary DESC
        ) AS rn
    FROM employees
)
SELECT employee_id
FROM x
WHERE rn = 1;
```

What does this query find?

## Solution

The CTE assigns a unique row number based on descending salary.

The outer query selects:

```text
rn = 1
```

Therefore it returns one highest-salary employee.

If multiple employees have the same maximum salary, only one is selected unless the ranking logic is changed.

### Answer

One highest-paid employee.

---

# 78. GATE-Style Solved Question 4 — Aggregation Through CTE

Given:

```sql
WITH x AS (
    SELECT
        customer_id,
        SUM(amount) AS total
    FROM orders
    GROUP BY customer_id
)
SELECT AVG(total)
FROM x;
```

What is being averaged?

## Solution

The first stage calculates:

```text
total revenue per customer
```

The second stage calculates:

```text
average of customer totals
```

It is **not necessarily** the same as:

```sql
SELECT AVG(amount)
FROM orders;
```

because customers can have different numbers of orders.

### Answer

The average of customer-level totals.

---

# 79. GATE-Style Solved Question 5 — Recursive CTE

Conceptually:

```sql
WITH RECURSIVE nums AS (
    SELECT 1 AS n

    UNION ALL

    SELECT n + 1
    FROM nums
    WHERE n < 5
)
SELECT n
FROM nums;
```

What values are generated?

## Solution

Anchor:

```text
1
```

Recursive iterations:

```text
2
3
4
5
```

At `n = 5`, the condition:

```sql
n < 5
```

is false.

### Answer

```text
1, 2, 3, 4, 5
```

---

# 80. GATE-Style Solved Question 6 — CTE + Join

Consider:

```sql
WITH totals AS (
    SELECT
        customer_id,
        SUM(amount) AS total
    FROM orders
    GROUP BY customer_id
)
SELECT
    c.customer_id,
    t.total
FROM customers c
LEFT JOIN totals t
    ON c.customer_id = t.customer_id;
```

Why is the CTE useful?

## Solution

The CTE produces at most one aggregate row per customer.

The final `LEFT JOIN` attaches that summary to the customer table.

Customers with no orders can remain in the result, with `t.total` as `NULL`.

### Answer

The CTE pre-aggregates orders at customer grain before joining to customers.

---

# 81. Advanced GATE-Style Question — Join Multiplication

Suppose a customer has:

```text
3 orders
4 payments
```

A direct join on customer ID can potentially produce:

```text
3 × 4 = 12
```

joined rows.

If order and payment amounts are summed after that join, totals may be inflated.

## Correct strategy

```sql
WITH order_totals AS (...),
payment_totals AS (...)
SELECT ...
FROM order_totals o
JOIN payment_totals p
    ON o.customer_id = p.customer_id;
```

### Key principle

Aggregate independent one-to-many sources before joining them.

---

# 82. Advanced GATE-Style Question — Recursive Structure

A recursive CTE contains:

```text
A = anchor
B = recursive member
```

If:

```text
A produces {1}
B transforms n → n+1 while n < 4
```

the iterations are:

```text
1
2
3
4
```

The recursive member does not run indefinitely because the predicate eventually becomes false.

### Key principle

A recursive CTE requires a valid stopping condition or another mechanism that guarantees termination.

---

# 83. Interview Practice — Easy

1. Write a CTE that returns employees with salary above 100000.
2. Use a CTE to calculate total revenue per customer.
3. Use a CTE to count orders per customer.
4. Use a CTE to classify employees into salary bands.
5. Use two CTEs and join their results.
6. Use a CTE to filter an aggregated result.

---

# 84. Interview Practice — Medium

7. Find the top 3 employees per department using a CTE and `ROW_NUMBER()`.
8. Find all employees tied for the highest salary per department.
9. Find the second-highest distinct salary per department.
10. Calculate month-over-month revenue using `LAG()` inside a CTE.
11. Calculate cumulative revenue after monthly aggregation.
12. Find customers whose revenue is above the average customer revenue.
13. Calculate each customer's percentage of total revenue.
14. Pre-aggregate orders and refunds separately and join them.
15. Find the latest transaction per customer.
16. Find customers with at least five orders and rank them by revenue.

---

# 85. Interview Practice — Hard

17. Build an organizational hierarchy using a recursive CTE.
18. Return every employee together with their hierarchy depth.
19. Return all descendants of a given manager.
20. Return the chain of managers for a given employee.
21. Detect or prevent cycles in a recursive hierarchy.
22. Generate a date/number sequence using recursion where appropriate.
23. Build a customer-month dataset, then calculate running revenue.
24. Aggregate two independent event streams and combine them without double counting.
25. Find the first month each customer crosses a cumulative revenue threshold.
26. Combine recursive hierarchy expansion, aggregation, and ranking.
27. Find top K entities per group after multiple filtering and aggregation stages.

---

# 86. Problem-Solving Template

When a problem looks complicated, ask:

```text
What should the final row represent?
```

Then design the CTE pipeline.

Example:

```text
Raw transactions
      ↓
filtered_transactions
      ↓
customer_totals
      ↓
ranked_customers
      ↓
top_customers
      ↓
join customer metadata
      ↓
final result
```

This is often easier than attempting one enormous nested query.

---

# 87. CTE Decision Tree

## Do I need a named intermediate result?

```text
YES → consider CTE
NO  → inline subquery may be enough
```

## Do I need to filter a window result?

```text
YES → CTE/subquery is usually useful
```

## Do I need multiple logical stages?

```text
YES → CTEs are usually clearer
```

## Do I need self-reference?

```text
YES → recursive CTE
```

## Do I need the intermediate data across multiple statements?

```text
YES → consider a temporary table instead
```

## Do I need a persistent reusable query object?

```text
YES → consider a view
```

---

# 88. CTE Master Templates

## Basic

```sql
WITH name AS (
    SELECT ...
)
SELECT ...
FROM name;
```

## Multiple CTEs

```sql
WITH
first_stage AS (
    ...
),
second_stage AS (
    SELECT ...
    FROM first_stage
)
SELECT ...
FROM second_stage;
```

## Window + filter

```sql
WITH ranked AS (
    SELECT
        *,
        ROW_NUMBER() OVER (
            PARTITION BY group_col
            ORDER BY metric DESC
        ) AS rn
    FROM table_name
)
SELECT *
FROM ranked
WHERE rn <= 3;
```

## Aggregate then analyze

```sql
WITH totals AS (
    SELECT
        group_col,
        SUM(value) AS total
    FROM table_name
    GROUP BY group_col
)
SELECT
    *,
    RANK() OVER (
        ORDER BY total DESC
    ) AS rnk
FROM totals;
```

## Aggregate then join

```sql
WITH totals AS (
    SELECT
        entity_id,
        SUM(value) AS total
    FROM fact_table
    GROUP BY entity_id
)
SELECT
    d.*,
    t.total
FROM dimension_table d
LEFT JOIN totals t
    ON t.entity_id = d.entity_id;
```

## Recursive

```sql
WITH RECURSIVE hierarchy AS (
    -- anchor
    SELECT ...

    UNION ALL

    -- recursive member
    SELECT ...
    FROM hierarchy h
    JOIN ...
)
SELECT *
FROM hierarchy;
```

---

# 89. CTE Mastery Checklist

## `WITH`

- [ ] Define a basic CTE.
- [ ] Understand statement-level scope.
- [ ] Explain that a CTE is not automatically materialized.
- [ ] Name CTEs descriptively.

## Multiple CTEs

- [ ] Define multiple CTEs.
- [ ] Reference earlier CTEs.
- [ ] Build a multi-stage query pipeline.
- [ ] Debug one CTE at a time.

## CTE vs Subquery

- [ ] Explain structural differences.
- [ ] Convert a subquery to a CTE.
- [ ] Explain why CTE does not automatically mean faster.
- [ ] Know when a subquery is simpler.

## Recursive CTE

- [ ] Explain anchor member.
- [ ] Explain recursive member.
- [ ] Write a sequence generator.
- [ ] Traverse a hierarchy.
- [ ] Track depth.
- [ ] Understand cycle risk.
- [ ] Ensure termination.
- [ ] Recognize DBMS-specific recursion syntax.

## CTE + Window

- [ ] Top K per group.
- [ ] Latest row per entity.
- [ ] Ranking after aggregation.
- [ ] `LAG()` comparison.
- [ ] Running totals.

## CTE + Aggregation

- [ ] Aggregate once, then aggregate again.
- [ ] Filter aggregate results.
- [ ] Calculate group-level statistics.
- [ ] Avoid confusing weighted and unweighted averages.

## CTE + Joins

- [ ] Pre-aggregate before joins.
- [ ] Prevent one-to-many multiplication.
- [ ] Join multiple independent aggregate CTEs.
- [ ] Preserve unmatched entities with `LEFT JOIN`.

---

# 90. Final Revision Sheet

### Basic CTE

```sql
WITH x AS (
    SELECT ...
)
SELECT ...
FROM x;
```

### Multiple CTEs

```sql
WITH
a AS (...),
b AS (
    SELECT ...
    FROM a
)
SELECT ...
FROM b;
```

### Window filtering

```sql
WITH ranked AS (
    SELECT
        *,
        ROW_NUMBER() OVER (...) AS rn
    FROM t
)
SELECT *
FROM ranked
WHERE rn = 1;
```

### Aggregate → window

```sql
WITH totals AS (
    SELECT
        group_id,
        SUM(value) AS total
    FROM t
    GROUP BY group_id
)
SELECT
    *,
    RANK() OVER (ORDER BY total DESC) AS rnk
FROM totals;
```

### Aggregate → join

```sql
WITH totals AS (
    SELECT
        entity_id,
        SUM(value) AS total
    FROM fact
    GROUP BY entity_id
)
SELECT ...
FROM dimension d
LEFT JOIN totals t
    ON d.entity_id = t.entity_id;
```

### Recursive

```sql
WITH RECURSIVE tree AS (
    SELECT ...

    UNION ALL

    SELECT ...
    FROM tree
    JOIN ...
)
SELECT *
FROM tree;
```

---

# 91. Final Interview Rules

Memorize these:

1. **`WITH` defines a CTE.**
2. **A normal CTE is scoped to the statement.**
3. **A CTE is a named query expression, not automatically a stored table.**
4. **Multiple CTEs are separated with commas.**
5. **Later CTEs can build on earlier CTEs.**
6. **CTEs are excellent for multi-stage analytical queries.**
7. **CTE vs subquery is primarily about query structure/readability; performance is optimizer-dependent.**
8. **Use a CTE to calculate a window value and filter it in an outer query.**
9. **Use CTEs to separate aggregation stages.**
10. **Pre-aggregate independent one-to-many sources before joining them when necessary.**
11. **Recursive CTEs contain an anchor member and recursive member.**
12. **Recursive CTEs require termination/progress.**
13. **Recursive hierarchy queries can encounter cycles.**
14. **CTE materialization behavior is DBMS-specific.**
15. **A CTE is not the same as a temporary table.**
16. **A CTE is not the same as a view.**
17. **Always identify the grain of each CTE.**
18. **Name CTEs according to what their rows represent.**
19. **Do not add CTE layers without a reason.**
20. **For complex SQL, think in stages rather than writing one giant query.**

---

# 92. One-Page Mental Model

```text
                         CTE
                          │
                     WITH name AS
                          │
          ┌───────────────┼────────────────┐
          │               │                │
       Normal          Multiple         Recursive
          │               │                │
      one stage       pipeline       anchor + recursion
          │               │                │
          └───────────────┼────────────────┘
                          │
              ┌───────────┼───────────┐
              │           │           │
          Aggregation   Windows      Joins
              │           │           │
           GROUP BY   RANK/LAG/    pre-aggregate
                       SUM OVER       safely
              │           │           │
              └───────────┼───────────┘
                          │
                    Final SELECT
```

Core recognition:

```text
Complex query              → CTE pipeline
Window result needs filter  → CTE + window
Aggregate then rank         → CTE + aggregation + window
Independent one-to-many     → pre-aggregate CTEs + join
Hierarchy/tree             → recursive CTE
Sequence generation        → recursive CTE
Readable multi-step SQL    → multiple CTEs
```

**Master CTEs together with Window Functions, Aggregation, Joins, and Subqueries. These combinations account for a large fraction of advanced SQL interview problems.**
