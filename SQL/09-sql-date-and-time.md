# SQL Topic 09 — Date & Time

> **Overall repository topic:** 32  
> **SQL topic number:** 9  
> **Difficulty focus:** Easy → Medium → Hard, with Medium dominant  
> **Goal:** Master date extraction, date arithmetic, date differences, month/year/week/day operations, date filtering, rolling-date logic, and time intervals.

---

# 1. Why Date & Time Is Critical

Date/time questions appear constantly in SQL interviews.

Typical requirements include:

- orders in the last 7 days,
- users active this month,
- sales by month,
- revenue by year,
- users who signed up before a given date,
- days between two events,
- time between login and logout,
- monthly retention,
- rolling 7-day metrics,
- records from the previous week,
- finding the latest event,
- filtering timestamps correctly.

The difficult part is often not the function itself.

The difficult part is choosing the correct:

- date boundary,
- interval,
- comparison,
- granularity,
- timezone assumption,
- inclusive/exclusive boundary.

---

# 2. DATE vs TIMESTAMP

A `DATE` generally represents a calendar date:

```text
2026-09-11
```

A timestamp represents a date plus time:

```text
2026-09-11 16:30:45
```

Some databases distinguish:

```text
DATE
DATETIME
TIMESTAMP
TIME
```

and may have timezone-aware timestamp types.

The exact type names vary by DBMS.

---

# 3. The Most Important Date/Time Mental Model

Think in terms of:

```text
calendar date
    +
time of day
    +
interval
    +
boundary
```

For example:

```text
"orders today"
```

is not merely:

```sql
date = today
```

If `order_time` is a timestamp, a robust pattern is often:

```sql
order_time >= start_of_today
AND order_time < start_of_tomorrow
```

This avoids accidentally excluding records later in the day.

---

# 4. Date Extraction

Date extraction means retrieving a component such as:

```text
year
month
day
hour
minute
second
week
weekday
```

A standard SQL-style form is:

```sql
EXTRACT(YEAR FROM order_date)
```

Examples:

```sql
EXTRACT(YEAR FROM order_date)
EXTRACT(MONTH FROM order_date)
EXTRACT(DAY FROM order_date)
```

Exact support varies by database.

---

# 5. YEAR Extraction

Example:

```sql
SELECT
    EXTRACT(YEAR FROM order_date) AS order_year
FROM orders;
```

For:

```text
2026-09-11
```

the result is:

```text
2026
```

---

# 6. MONTH Extraction

```sql
SELECT
    EXTRACT(MONTH FROM order_date) AS order_month
FROM orders;
```

For:

```text
2026-09-11
```

result:

```text
9
```

---

# 7. DAY Extraction

```sql
SELECT
    EXTRACT(DAY FROM order_date) AS day_of_month
FROM orders;
```

For:

```text
2026-09-11
```

result:

```text
11
```

Important:

```text
DAY
```

can mean different things depending on the function/dialect.

It may refer to:

```text
day of month
```

while another function may return:

```text
day of week
```

Always identify the intended meaning.

---

# 8. Hour / Minute / Second

For timestamp values, a standard conceptual form is:

```sql
EXTRACT(HOUR FROM created_at)
EXTRACT(MINUTE FROM created_at)
EXTRACT(SECOND FROM created_at)
```

Example timestamp:

```text
2026-09-11 16:35:42
```

produces conceptually:

```text
hour   → 16
minute → 35
second → 42
```

---

# 9. MONTH/YEAR Extraction Together

Example:

```sql
SELECT
    EXTRACT(YEAR FROM order_date) AS year,
    EXTRACT(MONTH FROM order_date) AS month,
    COUNT(*) AS orders
FROM orders
GROUP BY
    EXTRACT(YEAR FROM order_date),
    EXTRACT(MONTH FROM order_date);
```

This creates one group per year-month combination.

---

# 10. Do Not Group Only by Month When Years Matter

Suppose data contains:

```text
2025-01
2026-01
```

If you group only by:

```sql
EXTRACT(MONTH FROM order_date)
```

both January values are combined.

This is usually wrong for year-over-year analysis.

Instead group by:

```text
year + month
```

or use a month-start/date-truncation representation.

---

# 11. Date Arithmetic

Date arithmetic means adding or subtracting:

```text
days
weeks
months
years
hours
minutes
seconds
```

Conceptually:

```text
date + interval
date - interval
```

Examples:

```sql
date_column + INTERVAL '7 days'
```

or database-specific equivalents.

Syntax varies substantially between DBMSs.

---

# 12. Add Days

Conceptually:

```sql
order_date + INTERVAL '7 days'
```

means:

```text
7 days after order_date
```

Example:

```text
2026-09-11
+
7 days
=
2026-09-18
```

---

# 13. Subtract Days

```sql
order_date - INTERVAL '7 days'
```

means:

```text
7 days before order_date
```

Example:

```text
2026-09-11
-
7 days
=
2026-09-04
```

---

# 14. Add Months

Conceptually:

```sql
order_date + INTERVAL '1 month'
```

Example:

```text
2026-01-15
→
2026-02-15
```

Month arithmetic becomes interesting around month-end.

For example:

```text
January 31 + 1 month
```

does not have a universal intuitive answer because February may not have a 31st.

Database systems handle such cases according to their date-arithmetic rules.

For interviews, know that:

> Month arithmetic is not equivalent to adding a fixed number of days.

---

# 15. Add Years

Conceptually:

```sql
birth_date + INTERVAL '1 year'
```

Again, leap-year edge cases matter.

---

# 16. Date Differences

A date difference asks:

```text
How much time separates A and B?
```

Possible units:

```text
days
hours
minutes
seconds
months
years
```

The exact function differs by DBMS.

Examples of dialect-specific approaches include:

```text
DATEDIFF
TIMESTAMPDIFF
date subtraction
EXTRACT(epoch ...)
```

Do not memorize one syntax as universally portable.

---

# 17. Days Between Two Dates

Conceptually:

```text
later_date - earlier_date
```

may produce an interval or number of days depending on the database.

Example:

```text
2026-09-20
-
2026-09-11
=
9 days
```

Always determine what unit the database's difference function returns.

---

# 18. Date Difference Direction

Order matters.

Conceptually:

```text
B - A
```

is the negative of:

```text
A - B
```

Example:

```text
2026-09-20 - 2026-09-11 = 9
```

but:

```text
2026-09-11 - 2026-09-20 = -9
```

This is a common interview trap.

---

# 19. Difference Between DATE and TIMESTAMP

Suppose:

```text
start = 2026-09-11 10:00:00
end   = 2026-09-11 18:30:00
```

The elapsed time is:

```text
8.5 hours
```

but the calendar dates are the same.

Therefore:

```text
calendar difference
```

and:

```text
elapsed-time difference
```

are different concepts.

Know which one the question asks for.

---

# 20. Month Difference Is Not Always Day Difference / 30

Do not assume:

```text
months = days / 30
```

because calendar months have different lengths.

Likewise:

```text
1 year = 365 days
```

is not always true because leap years exist.

For date calculations involving months/years, use calendar-aware date arithmetic.

---

# 21. Week Extraction

A database may provide a week component:

```sql
EXTRACT(WEEK FROM order_date)
```

but week numbering is database- and standard-dependent.

Potential differences include:

- week starts on Sunday vs Monday,
- ISO week numbering,
- week 1 definition,
- year boundary behavior.

This matters around:

```text
late December
early January
```

---

# 22. ISO Week Trap

A date near New Year can belong to:

```text
ISO week 1 of the next ISO year
```

even though the calendar date is still in December.

Therefore, if a problem asks for weekly reporting around year boundaries, understand whether it means:

```text
calendar year + week
```

or:

```text
ISO week-year + week
```

Do not assume they are identical.

---

# 23. Day of Week

Some systems provide:

```text
day of week
```

through functions such as:

```text
EXTRACT
DAYOFWEEK
DATE_PART
```

The exact numbering can vary.

For example, one system may use:

```text
Sunday = 0
```

while another may use:

```text
Monday = 1
```

Therefore:

> Never memorize day-of-week numeric values without associating them with the DBMS.

---

# 24. Finding Weekends

A conceptual approach is:

```text
extract day of week
→ identify Saturday/Sunday
```

But exact syntax depends on the DBMS.

For portable interview reasoning, explain the logic first and then implement using the target dialect.

---

# 25. Date Filtering

This is one of the most important sections.

Suppose:

```text
created_at
```

is a timestamp.

To find records on a particular calendar date:

```text
2026-09-11
```

a robust pattern is:

```sql
created_at >= '2026-09-11'
AND created_at <  '2026-09-12'
```

This captures:

```text
2026-09-11 00:00:00
through
2026-09-11 23:59:59.999...
```

without needing to guess the final fractional second.

---

# 26. Why Half-Open Intervals Are Powerful

Use:

```text
>= start
< end
```

rather than:

```text
>= start
<= 23:59:59
```

because timestamps can have:

- milliseconds,
- microseconds,
- nanoseconds,
- different precision.

The half-open pattern:

```text
[start, end)
```

avoids precision bugs.

---

# 27. Date Filtering by Month

Suppose you need all records in September 2026.

A robust timestamp range is:

```sql
created_at >= '2026-09-01'
AND created_at <  '2026-10-01'
```

This is usually preferable to:

```sql
EXTRACT(YEAR FROM created_at) = 2026
AND EXTRACT(MONTH FROM created_at) = 9
```

when an index on `created_at` matters.

The range predicate can often be more index-friendly.

---

# 28. Date Filtering by Year

For all records in 2026:

```sql
created_at >= '2026-01-01'
AND created_at <  '2027-01-01'
```

This is usually preferable to:

```sql
EXTRACT(YEAR FROM created_at) = 2026
```

for large indexed tables.

---

# 29. Date Filtering by Exact Date

Avoid relying on:

```sql
created_at = '2026-09-11'
```

when `created_at` is a timestamp.

Why?

The timestamp contains a time component.

For example:

```text
2026-09-11 10:15:00
```

is not equal to:

```text
2026-09-11 00:00:00
```

Use a range:

```sql
created_at >= '2026-09-11'
AND created_at <  '2026-09-12'
```

---

# 30. Date Filtering with DATE Columns

If the column is genuinely a `DATE` without time:

```sql
WHERE order_date = '2026-09-11'
```

may be completely appropriate.

The timestamp issue is primarily about columns containing a time component.

---

# 31. "Today"

A DBMS normally provides a current-date/current-timestamp function, but its name differs.

Examples include:

```text
CURRENT_DATE
CURRENT_TIMESTAMP
NOW()
GETDATE()
SYSDATETIME()
```

depending on the database.

The conceptual requirement:

```text
today's calendar date
```

is portable; the function name is not.

---

# 32. "Last 7 Days" — Critical Ambiguity

"Last 7 days" can mean different things.

### Meaning A — Rolling 168 hours

```text
current timestamp - 7 days
→ now
```

### Meaning B — Seven calendar dates

```text
today and six previous calendar days
```

These are not necessarily the same.

Interview questions often test whether you notice this distinction.

---

# 33. Rolling 7-Day Timestamp Window

If the requirement is:

> previous 7 × 24 hours

conceptually:

```sql
WHERE created_at >= CURRENT_TIMESTAMP - INTERVAL '7 days'
```

This is a rolling time window.

---

# 34. Rolling Calendar-Date Window

If the requirement is:

> records from the last seven calendar days

you may instead construct:

```text
start of today
minus 6 days
→ start of tomorrow
```

The exact SQL depends on the DBMS.

The important point is:

```text
rolling hours ≠ calendar-day window
```

unless the boundaries happen to align.

---

# 35. "This Month"

A common robust pattern is:

```text
created_at >= start_of_current_month
AND
created_at < start_of_next_month
```

Conceptually:

```sql
WHERE created_at >= month_start
  AND created_at < next_month_start
```

This is better than comparing only the extracted month because it keeps the year boundary correct.

---

# 36. "Previous Month"

Use:

```text
previous_month_start
≤ timestamp
<
current_month_start
```

Conceptually:

```sql
WHERE created_at >= previous_month_start
  AND created_at < current_month_start
```

This handles varying month lengths automatically.

---

# 37. "This Year"

Use:

```text
current_year_start
≤ timestamp
<
next_year_start
```

This is the same half-open interval principle.

---

# 38. Date Truncation

Many databases provide a date-truncation function such as:

```sql
DATE_TRUNC(...)
```

Conceptually:

```text
timestamp
    ↓
truncate to month
    ↓
month start
```

Example:

```text
2026-09-11 16:30
→ 2026-09-01 00:00
```

for month-level truncation.

Exact support and syntax vary.

---

# 39. Date Extraction vs Date Truncation

These are different.

### Extraction

```sql
EXTRACT(MONTH FROM created_at)
```

returns:

```text
9
```

### Truncation

```text
DATE_TRUNC('month', created_at)
```

returns conceptually:

```text
2026-09-01 00:00:00
```

Mental model:

```text
EXTRACT
→ gives component

TRUNCATE
→ gives boundary representing the component
```

---

# 40. Month-Level Grouping

Instead of:

```sql
GROUP BY
    EXTRACT(YEAR FROM created_at),
    EXTRACT(MONTH FROM created_at)
```

some systems allow:

```sql
GROUP BY DATE_TRUNC('month', created_at)
```

This produces a month-start value that naturally identifies:

```text
year + month
```

This is often convenient for time-series reporting.

---

# 41. Date Arithmetic with Intervals

An interval represents a duration such as:

```text
7 days
3 months
2 hours
30 minutes
```

Conceptually:

```sql
timestamp + INTERVAL '2 hours'
```

or:

```sql
timestamp - INTERVAL '30 minutes'
```

Syntax varies across databases.

---

# 42. Time Intervals

Time intervals are useful for:

- session duration,
- delivery time,
- response time,
- time between events,
- SLA measurement,
- processing duration.

Example:

```text
login  = 10:00
logout = 18:30
```

Elapsed:

```text
8 hours 30 minutes
```

---

# 43. Time Difference Pattern

Conceptually:

```text
end_time - start_time
```

or a dialect-specific difference function.

Example:

```sql
SELECT
    end_time - start_time AS duration
FROM sessions;
```

Whether this returns an interval, number, or other representation depends on the DBMS.

---

# 44. Difference in Specific Units

Sometimes you need:

```text
days
hours
minutes
seconds
```

A database may provide a function such as:

```text
DATEDIFF
TIMESTAMPDIFF
```

with a specified unit.

For example, conceptually:

```text
difference in days
difference in hours
difference in minutes
```

Always read the function's exact semantics:

> Some functions count boundary crossings rather than measuring a precise elapsed duration.

This distinction can produce surprising interview answers.

---

# 45. Boundary-Crossing Trap

Consider:

```text
start = 2026-01-01 23:59
end   = 2026-01-02 00:01
```

Elapsed time:

```text
2 minutes
```

But a day-boundary-based difference function may report:

```text
1 day
```

depending on the DBMS/function.

Therefore ask:

```text
Do they want elapsed duration?
or
Do they want calendar boundary difference?
```

---

# 46. Rolling Date Windows

Rolling windows are extremely common.

Examples:

```text
last 7 days
last 30 days
last 90 days
previous 24 hours
previous 12 hours
```

A rolling window is anchored to a moving point such as:

```text
CURRENT_TIMESTAMP
```

Conceptual form:

```sql
WHERE event_time >= CURRENT_TIMESTAMP - INTERVAL '30 days'
```

---

# 47. Rolling Window: Include Now

A common pattern is:

```sql
WHERE event_time >= start_time
  AND event_time < current_time
```

or simply:

```sql
WHERE event_time >= current_time - interval
```

depending on whether the upper bound needs to be explicit.

Use half-open intervals when precise boundaries matter.

---

# 48. Rolling 30-Day Example

Requirement:

> Find orders created in the previous 30 × 24 hours.

Conceptual:

```sql
SELECT *
FROM orders
WHERE created_at >= CURRENT_TIMESTAMP - INTERVAL '30 days';
```

This is a rolling time window.

It is not necessarily the same as:

```text
the previous 30 calendar dates.
```

---

# 49. Previous Calendar Month vs Last 30 Days

These are different.

### Last 30 days

```text
moving 30-day interval
```

### Previous month

```text
entire calendar month immediately before the current month
```

Example if today is in September:

```text
last 30 days
→ part of August + part of September

previous month
→ all of August
```

Interview questions often deliberately distinguish these.

---

# 50. Date Filtering and Indexes

Compare:

```sql
WHERE created_at >= '2026-09-01'
  AND created_at < '2026-10-01'
```

with:

```sql
WHERE EXTRACT(YEAR FROM created_at) = 2026
  AND EXTRACT(MONTH FROM created_at) = 9
```

The range form often preserves better index usability because the indexed column is compared directly against boundaries.

This is a common performance interview point.

---

# 51. Avoid Functions on Indexed Date Columns When Possible

Instead of:

```sql
WHERE DATE(created_at) = '2026-09-11'
```

prefer:

```sql
WHERE created_at >= '2026-09-11'
  AND created_at <  '2026-09-12'
```

when the database and query requirements permit it.

The range form avoids transforming every column value before comparison.

---

# 52. Month/Year Filtering Trap

This query:

```sql
WHERE EXTRACT(MONTH FROM created_at) = 9
```

returns September from:

```text
2024
2025
2026
...
```

If you need September 2026:

```sql
WHERE created_at >= '2026-09-01'
  AND created_at <  '2026-10-01'
```

or include year and month extraction.

---

# 53. Date Parts in ORDER BY

Example:

```sql
SELECT
    EXTRACT(YEAR FROM order_date) AS order_year,
    EXTRACT(MONTH FROM order_date) AS order_month,
    SUM(amount) AS revenue
FROM orders
GROUP BY
    EXTRACT(YEAR FROM order_date),
    EXTRACT(MONTH FROM order_date)
ORDER BY
    order_year,
    order_month;
```

This produces chronological year-month reporting.

---

# 54. Month Names and Sorting

If you create:

```text
January
February
March
...
```

as strings, alphabetical sorting can produce:

```text
April
August
December
February
...
```

Instead sort by a numeric month or month-start date.

Example:

```sql
ORDER BY
    EXTRACT(MONTH FROM order_date)
```

when analyzing one year.

For multiple years, sort by:

```text
year, month
```

or a month-start date.

---

# 55. Date Arithmetic Around Month-End

Be careful with:

```text
January 31
February 28/29
March 31
```

Adding one month is a calendar operation, not:

```text
+ 30 days
```

Similarly:

```text
1 year
```

and:

```text
365 days
```

are not always equivalent.

---

# 56. Time Zones

Timestamp questions can become incorrect if timezone semantics are ignored.

Example:

```text
2026-09-11 23:30 UTC
```

may be:

```text
2026-09-12
```

in another timezone.

Therefore, for global applications, distinguish:

```text
UTC timestamp
local timestamp
display timezone
business timezone
```

A "day" is not necessarily the same UTC interval for every business/reporting requirement.

---

# 57. CURRENT_DATE vs CURRENT_TIMESTAMP

Conceptually:

```text
CURRENT_DATE
→ current calendar date

CURRENT_TIMESTAMP
→ current date + time
```

Example:

```text
CURRENT_DATE
→ 2026-09-11

CURRENT_TIMESTAMP
→ 2026-09-11 16:30:00
```

Exact returned precision/timezone behavior depends on DBMS.

---

# 58. Date Filtering Template

For a fixed calendar day:

```sql
WHERE timestamp_column >= day_start
  AND timestamp_column <  next_day_start
```

For a month:

```sql
WHERE timestamp_column >= month_start
  AND timestamp_column <  next_month_start
```

For a year:

```sql
WHERE timestamp_column >= year_start
  AND timestamp_column <  next_year_start
```

Memorize this structure.

---

# 59. Date Arithmetic Template

Conceptually:

```sql
date_column + INTERVAL 'N days'
```

```sql
date_column - INTERVAL 'N days'
```

```sql
date_column + INTERVAL 'N months'
```

```sql
date_column - INTERVAL 'N months'
```

Use the syntax appropriate to the target DBMS.

---

# 60. Date Difference Template

Conceptually:

```text
end - start
```

or:

```text
DATE_DIFF(unit, start, end)
```

or:

```text
DATEDIFF(unit, start, end)
```

or:

```text
TIMESTAMPDIFF(unit, start, end)
```

depending on the DBMS.

The important interview skill is understanding:

```text
which date is later
which unit is required
whether the function measures elapsed time or boundaries
```

---

# 61. Common Interview Problem Types

Expect questions such as:

1. Find orders from the last 7 days.
2. Find users who signed up this month.
3. Find monthly revenue.
4. Find users who registered in 2025.
5. Calculate days between signup and first purchase.
6. Calculate session duration.
7. Find the previous month's sales.
8. Find customers active during the last 30 days.
9. Group orders by year and month.
10. Find weekend activity.
11. Find records from a specific calendar date.
12. Find events within 24 hours of another event.
13. Calculate average response time.
14. Find the rolling 7-day total.
15. Compare current month with previous month.

---

# 62. GATE / Database Theory

## 62.1 DATE Arithmetic Is Calendar-Aware

Adding:

```text
1 month
```

is not necessarily equivalent to adding:

```text
30 days
```

Adding:

```text
1 year
```

is not necessarily equivalent to adding:

```text
365 days
```

---

## 62.2 Date Components

A timestamp can conceptually be decomposed into:

```text
year
month
day
hour
minute
second
```

Functions can extract these components.

---

## 62.3 NULL Dates

A NULL date represents missing/unknown date information.

Expressions such as:

```sql
date_column + interval
```

generally produce NULL when `date_column` is NULL.

Likewise:

```sql
EXTRACT(YEAR FROM NULL)
```

produces NULL under ordinary SQL NULL propagation.

---

## 62.4 Date Comparison

Date/timestamp values can be compared using:

```text
<
<=
>
>=
=
<>
```

provided the types are compatible.

Chronological ordering follows the temporal values.

---

# 63. GATE-Style Solved Questions

> These are **GATE-style practice questions**, not claims of official GATE PYQs.

## Question 1 — Month Extraction

For:

```text
2026-09-17
```

what does:

```sql
EXTRACT(MONTH FROM date_value)
```

return?

### Solution

The month component is:

```text
September = 9
```

### Answer

```text
9
```

---

## Question 2 — Date Difference

Suppose:

```text
start_date = 2026-09-11
end_date   = 2026-09-20
```

What is the elapsed calendar-date difference in days?

### Solution

```text
20 - 11 = 9
```

### Answer

```text
9 days
```

The direction matters:

```text
start - end = -9
```

---

## Question 3 — Exact Calendar Date Filtering

`created_at` is a timestamp.

Which condition correctly retrieves all rows from September 11, 2026?

### Options

A:

```sql
created_at = '2026-09-11'
```

B:

```sql
created_at >= '2026-09-11'
AND created_at < '2026-09-12'
```

C:

```sql
EXTRACT(MONTH FROM created_at) = 9
```

D:

```sql
created_at > '2026-09-11'
```

### Solution

A can miss rows because the timestamp contains time.

C includes September from every year.

D excludes midnight and has no upper bound.

B precisely defines:

```text
[2026-09-11 00:00, 2026-09-12 00:00)
```

### Answer

```text
B
```

---

## Question 4 — Month Grouping

Why is grouping only by:

```sql
EXTRACT(MONTH FROM order_date)
```

incorrect for multi-year monthly reporting?

### Solution

January 2025 and January 2026 both produce:

```text
1
```

so they are grouped together.

Use:

```text
year + month
```

or a month-truncated date.

### Answer

Because month alone does not identify the year.

---

## Question 5 — Rolling Window

Suppose the current timestamp is:

```text
2026-09-11 18:00
```

A query asks for records from the previous 24 hours.

What is the lower boundary?

### Solution

Subtract 24 hours:

```text
2026-09-10 18:00
```

So the conceptual range is:

```text
2026-09-10 18:00
≤ event_time
≤ now
```

For precise SQL, a half-open upper boundary can be used depending on the requirement.

### Answer

The rolling window starts at:

```text
2026-09-10 18:00
```

---

## Question 6 — Calendar vs Rolling Time

Suppose the current time is:

```text
2026-09-11 15:00
```

What is the conceptual start of the previous 7 × 24-hour window?

### Solution

Subtract 7 days:

```text
2026-09-04 15:00
```

This is different from a calendar-date window beginning at midnight.

### Answer

```text
2026-09-04 15:00
```

---

# 64. Interview Practice

Try these without looking at the solutions.

### Q1
Find all orders placed on a particular calendar date when `created_at` is a timestamp.

### Q2
Find orders from the current month.

### Q3
Find orders from the previous calendar month.

### Q4
Find users created in the last 30 × 24 hours.

### Q5
Find users created in the last 7 calendar days.

### Q6
Calculate the number of days between signup and first purchase.

### Q7
Calculate session duration in hours.

### Q8
Group revenue by year and month.

### Q9
Find all events occurring on weekends.

### Q10
Find records from 2026 only.

### Q11
Find the number of orders per month in chronological order.

### Q12
Explain why `DATE(timestamp_column) = date_value` can be less index-friendly than a timestamp range.

---

# 65. Interview Solution Patterns

## Q1 — One Calendar Date

```sql
WHERE created_at >= :day_start
  AND created_at <  :next_day_start
```

---

## Q2 — Current Month

Conceptually:

```sql
WHERE created_at >= current_month_start
  AND created_at <  next_month_start
```

---

## Q3 — Previous Month

```sql
WHERE created_at >= previous_month_start
  AND created_at <  current_month_start
```

---

## Q4 — Last 30 × 24 Hours

Conceptually:

```sql
WHERE created_at >= CURRENT_TIMESTAMP - INTERVAL '30 days'
```

Use the target DBMS's interval syntax.

---

## Q5 — Last 7 Calendar Days

Define the calendar boundaries first:

```text
start-of-window
→ start of today minus appropriate number of calendar days

end
→ start of tomorrow
```

The exact number of days depends on whether "last 7 days" includes today.

Always clarify the intended business definition.

---

## Q6 — Days Between Dates

Conceptually:

```text
first_purchase_date - signup_date
```

Use the appropriate date-difference function for the target DBMS.

---

## Q7 — Session Duration

Conceptually:

```text
logout_time - login_time
```

Then convert/extract the result into hours using the target DBMS.

---

## Q8 — Revenue by Year/Month

```sql
SELECT
    EXTRACT(YEAR FROM order_date) AS year,
    EXTRACT(MONTH FROM order_date) AS month,
    SUM(amount) AS revenue
FROM orders
GROUP BY
    EXTRACT(YEAR FROM order_date),
    EXTRACT(MONTH FROM order_date)
ORDER BY
    year,
    month;
```

---

## Q9 — Weekend

Extract the day-of-week according to the target DBMS and filter the values corresponding to Saturday/Sunday.

Do not assume universal numeric weekday codes.

---

## Q10 — 2026

Prefer:

```sql
WHERE created_at >= '2026-01-01'
  AND created_at <  '2027-01-01'
```

for a timestamp column.

---

## Q11 — Chronological Monthly Orders

Use a year-month representation:

```sql
SELECT
    EXTRACT(YEAR FROM order_date) AS year,
    EXTRACT(MONTH FROM order_date) AS month,
    COUNT(*) AS order_count
FROM orders
GROUP BY
    EXTRACT(YEAR FROM order_date),
    EXTRACT(MONTH FROM order_date)
ORDER BY
    year,
    month;
```

---

## Q12 — Index Friendliness

A range:

```sql
created_at >= start
AND created_at < end
```

compares the indexed column directly to boundaries.

A function-wrapped column:

```sql
DATE(created_at) = ...
```

may prevent efficient use of a normal index, depending on the database and index design.

---

# 66. Common Interview Traps

## Trap 1 — `timestamp = date`

Wrong for general timestamp filtering:

```sql
created_at = '2026-09-11'
```

Prefer:

```sql
created_at >= '2026-09-11'
AND created_at < '2026-09-12'
```

---

## Trap 2 — Inclusive end-of-day timestamp

Avoid manually writing:

```text
23:59:59
```

because timestamps may have fractional seconds.

Prefer:

```text
>= start
< next boundary
```

---

## Trap 3 — Month without year

Wrong for multi-year reports:

```sql
GROUP BY EXTRACT(MONTH FROM date)
```

Use year + month.

---

## Trap 4 — Last 7 days ambiguity

Determine whether the problem means:

```text
168 rolling hours
```

or:

```text
7 calendar dates
```

---

## Trap 5 — Previous month vs last 30 days

These are not equivalent.

---

## Trap 6 — Day of week numbering

Different DBMSs use different weekday numbering.

---

## Trap 7 — Week numbering

ISO weeks and ordinary week numbers can differ around year boundaries.

---

## Trap 8 — Month arithmetic equals 30 days

Wrong assumption.

```text
1 month ≠ always 30 days
```

---

## Trap 9 — Year arithmetic equals 365 days

Wrong around leap years.

---

## Trap 10 — Ignoring timezone

A timestamp can belong to different calendar dates in different timezones.

---

# 67. High-Value Date Patterns

## Today

Conceptually:

```text
[start_of_today, start_of_tomorrow)
```

## Current month

```text
[current_month_start, next_month_start)
```

## Current year

```text
[current_year_start, next_year_start)
```

## Previous month

```text
[previous_month_start, current_month_start)
```

## Last 24 hours

```text
[current_timestamp - 24 hours, current_timestamp)
```

## Last 7 × 24 hours

```text
[current_timestamp - 7 days, current_timestamp)
```

## Specific calendar date

```text
[date, next_date)
```

Memorize the interval notation:

```text
[start, end)
```

---

# 68. Date + CASE

Date extraction often combines with `CASE`.

Example:

```sql
CASE
    WHEN order_date >= CURRENT_DATE - INTERVAL '7 days'
        THEN 'Recent'
    ELSE 'Older'
END
```

This can classify records into time-based categories.

---

# 69. Date + Conditional Aggregation

Example:

```sql
SELECT
    SUM(
        CASE
            WHEN created_at >= CURRENT_DATE - INTERVAL '30 days'
            THEN amount
            ELSE 0
        END
    ) AS recent_revenue
FROM orders;
```

This is a common combination of:

```text
date filtering logic
+
CASE
+
aggregation
```

---

# 70. Date + GROUP BY

Example:

```sql
SELECT
    EXTRACT(YEAR FROM created_at) AS year,
    EXTRACT(MONTH FROM created_at) AS month,
    COUNT(*) AS users
FROM users
GROUP BY
    EXTRACT(YEAR FROM created_at),
    EXTRACT(MONTH FROM created_at);
```

This is a basic monthly analytics pattern.

---

# 71. Date + JOIN

A common interview pattern is joining events within a time window.

Conceptually:

```sql
SELECT ...
FROM events e1
JOIN events e2
  ON e2.user_id = e1.user_id
 AND e2.event_time >= e1.event_time
 AND e2.event_time < e1.event_time + INTERVAL '24 hours';
```

This asks for related events occurring within a specified interval.

Exact interval syntax is dialect-specific.

---

# 72. Latest Record per Entity

Date/time columns are frequently used with:

```text
MAX(date)
```

or:

```text
ROW_NUMBER() OVER (
    PARTITION BY entity_id
    ORDER BY event_time DESC
)
```

Example:

```sql
SELECT
    user_id,
    MAX(created_at) AS latest_event
FROM events
GROUP BY user_id;
```

Window functions provide more control when the full row is required.

---

# 73. Rolling Date Metrics

Suppose you need daily revenue.

A later window-function topic can extend this to:

```text
7-day rolling revenue
30-day rolling revenue
```

The conceptual process is:

```text
daily aggregation
      ↓
order by date
      ↓
window over previous N days
```

Do not confuse this with a simple filter for the last N days.

A rolling metric produces a value for **each date**.

---

# 74. Fixed Window vs Rolling Window

### Fixed filter

```text
Give me revenue for the last 30 days.
```

Produces one selected time period.

### Rolling metric

```text
For every day, give me revenue over the previous 30 days.
```

Produces one value per date.

This distinction becomes very important in analytical SQL.

---

# 75. Date Truncation vs Extraction

Use extraction when you need:

```text
2026
9
11
```

Use truncation when you need:

```text
2026-09-01
```

as the representative month.

Mental model:

```text
EXTRACT → component
TRUNC   → period boundary
```

---

# 76. DBMS Syntax Cheat Sheet

The concepts are portable, but syntax varies.

| Task | Common approaches |
|---|---|
| Current date | `CURRENT_DATE` |
| Current timestamp | `CURRENT_TIMESTAMP`, `NOW()` |
| Extract year | `EXTRACT(YEAR FROM x)` |
| Extract month | `EXTRACT(MONTH FROM x)` |
| Date truncation | `DATE_TRUNC(...)` |
| Difference | `DATEDIFF`, `TIMESTAMPDIFF`, subtraction |
| Add interval | `+ INTERVAL ...` or DB-specific function |
| Weekday | `EXTRACT`, `DAYOFWEEK`, database-specific |
| Month start | truncation or date construction |

Learn the target DBMS syntax separately.

---

# 77. Mastery Checklist

You should be able to:

- [ ] Distinguish DATE from TIMESTAMP.
- [ ] Extract year.
- [ ] Extract month.
- [ ] Extract day.
- [ ] Extract hour.
- [ ] Extract minute.
- [ ] Extract second.
- [ ] Extract week.
- [ ] Determine day of week using the target DBMS.
- [ ] Add days.
- [ ] Subtract days.
- [ ] Add months.
- [ ] Subtract months.
- [ ] Add years.
- [ ] Calculate date differences.
- [ ] Calculate time differences.
- [ ] Understand difference direction.
- [ ] Distinguish elapsed time from calendar-boundary differences.
- [ ] Filter an exact calendar date from a timestamp.
- [ ] Filter a month using a timestamp range.
- [ ] Filter a year using a timestamp range.
- [ ] Filter the previous calendar month.
- [ ] Write rolling 24-hour filters.
- [ ] Write rolling 7-day filters.
- [ ] Distinguish rolling days from calendar days.
- [ ] Group by year and month.
- [ ] Avoid grouping by month alone across multiple years.
- [ ] Understand week/ISO-week edge cases.
- [ ] Understand month-end arithmetic.
- [ ] Understand leap-year implications.
- [ ] Understand basic timezone issues.
- [ ] Explain half-open intervals.
- [ ] Explain why range predicates can be more index-friendly.
- [ ] Recognize date/time interview patterns immediately.

---

# 78. Final Revision Sheet

## Extraction

```sql
EXTRACT(YEAR FROM date_col)
EXTRACT(MONTH FROM date_col)
EXTRACT(DAY FROM date_col)
EXTRACT(HOUR FROM timestamp_col)
EXTRACT(MINUTE FROM timestamp_col)
EXTRACT(SECOND FROM timestamp_col)
```

## Filtering a day

```sql
WHERE timestamp_col >= day_start
  AND timestamp_col < next_day_start
```

## Filtering a month

```sql
WHERE timestamp_col >= month_start
  AND timestamp_col < next_month_start
```

## Filtering a year

```sql
WHERE timestamp_col >= year_start
  AND timestamp_col < next_year_start
```

## Rolling interval

```sql
WHERE timestamp_col >= CURRENT_TIMESTAMP - INTERVAL 'N days'
```

## Date difference

```text
end - start
```

or a DBMS-specific difference function.

## Month grouping

```text
year + month
```

or:

```text
month-start/truncated date
```

## Core boundary rule

```text
[start, end)
```

means:

```text
>= start
< end
```

---

# 79. Final Interview Takeaways

The most important date/time skills are not memorizing dozens of functions. They are:

### 1. Correct boundaries

```text
>= start
< next boundary
```

### 2. Correct granularity

Know whether the problem asks for:

```text
date
day
week
month
year
hour
```

### 3. Correct interpretation

Distinguish:

```text
last 7 × 24 hours
```

from:

```text
last 7 calendar days
```

and:

```text
last 30 days
```

from:

```text
previous calendar month
```

### 4. Correct grouping

For monthly data across years:

```text
year + month
```

not month alone.

### 5. Correct difference semantics

Understand whether the database function measures:

```text
elapsed duration
```

or:

```text
calendar boundary crossings
```

### 6. Performance-aware filtering

Prefer direct timestamp ranges where appropriate:

```sql
created_at >= start
AND created_at < end
```

over unnecessarily wrapping the indexed column in a date function.

### 7. DBMS awareness

Date/time syntax varies significantly.

The concepts remain the same:

```text
extract
add/subtract
difference
truncate
filter
group
window
```

---

# 80. Core Mental Model

When you see a date/time SQL question, ask these questions in order:

```text
1. Is this DATE or TIMESTAMP?
        ↓
2. What granularity is required?
        ↓
3. Is the window rolling or calendar-based?
        ↓
4. What are the exact start/end boundaries?
        ↓
5. Is the end boundary inclusive or exclusive?
        ↓
6. Does timezone matter?
        ↓
7. Can a range predicate preserve index usability?
        ↓
8. Which DBMS syntax is required?
```

If you can answer those eight questions, most common SQL date/time interview problems become substantially easier.
