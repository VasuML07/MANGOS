# SQL Topic 08 — String Functions

> **Overall repository topic:** 31  
> **SQL topic number:** 8  
> **Difficulty focus:** Easy → Medium → Hard, with Medium dominant  
> **Goal:** Master common SQL string functions and pattern matching for interview problems.

---

# 1. Why String Functions Matter

String operations appear frequently in SQL interviews and real data work.

Typical tasks include:

- combining names,
- extracting prefixes/suffixes,
- extracting part of a string,
- measuring string length,
- normalizing case,
- removing unwanted spaces,
- replacing characters or substrings,
- searching for patterns,
- validating text formats,
- cleaning imported data.

The core functions in this topic are:

```text
CONCAT
SUBSTRING
LEFT
RIGHT
LENGTH
LOWER
UPPER
TRIM
REPLACE
```

Pattern matching primarily uses:

```sql
LIKE
```

and its wildcards:

```text
%   → zero or more characters
_   → exactly one character
```

---

# 2. Important Dialect Warning

String-function syntax varies more across SQL databases than basic `SELECT`, `WHERE`, `GROUP BY`, and `JOIN` syntax.

For example:

- `LENGTH()` is common, but some systems also use `LEN()`.
- `SUBSTRING()` syntax differs between database systems.
- `LEFT()` and `RIGHT()` are supported by several systems but are not equally portable.
- Concatenation may be implemented with `CONCAT()`, `||`, or other dialect-specific syntax.
- Character length vs byte length can differ depending on the function and database.

For interview preparation:

1. Learn the concept.
2. Learn the syntax expected by the database being tested.
3. Never assume every DBMS implements string functions identically.

Examples below use widely recognizable SQL syntax and explicitly note important dialect differences.

---

# 3. CONCAT

## Definition

`CONCAT` combines multiple strings into one string.

```sql
CONCAT(string1, string2, ...)
```

Example:

```sql
SELECT CONCAT(first_name, ' ', last_name) AS full_name
FROM employees;
```

If:

```text
first_name = 'Ravi'
last_name  = 'Kumar'
```

the result is:

```text
Ravi Kumar
```

---

# 4. CONCAT with Multiple Values

```sql
SELECT CONCAT(
    first_name,
    ' ',
    last_name,
    ' - ',
    department
) AS employee_label
FROM employees;
```

Example result:

```text
Ravi Kumar - Engineering
```

`CONCAT` is useful when constructing display strings from multiple columns.

---

# 5. CONCAT and NULL

NULL behavior can vary by DBMS and function implementation.

Do not blindly assume:

```sql
CONCAT(a, b)
```

behaves exactly like:

```sql
a || b
```

in every database.

A safe interview habit is to know the target dialect's NULL-concatenation rules.

If you need an explicit fallback:

```sql
CONCAT(
    COALESCE(first_name, ''),
    ' ',
    COALESCE(last_name, '')
)
```

This makes the desired behavior explicit.

---

# 6. String Concatenation Operator

Some SQL systems support:

```sql
first_name || ' ' || last_name
```

while others use:

```sql
CONCAT(first_name, ' ', last_name)
```

Some systems may also use `+` for string concatenation.

Therefore:

> `CONCAT()` is conceptually portable, but the exact concatenation syntax is database-dependent.

For interview questions, use the syntax expected by the stated DBMS.

---

# 7. SUBSTRING

## Definition

`SUBSTRING` extracts part of a string.

A common conceptual form is:

```sql
SUBSTRING(string, start_position, length)
```

Example:

```sql
SELECT SUBSTRING('DATABASE', 1, 4);
```

Conceptually:

```text
DATA
```

However, exact indexing and syntax are dialect-dependent.

---

# 8. SUBSTRING Positioning

A common convention is:

```text
D A T A B A S E
1 2 3 4 5 6 7 8
```

Then:

```sql
SUBSTRING('DATABASE', 1, 4)
```

returns:

```text
DATA
```

and:

```sql
SUBSTRING('DATABASE', 5, 4)
```

returns:

```text
BASE
```

Always verify the dialect when exact syntax matters.

---

# 9. SUBSTRING Without Explicit Length

Some databases allow syntax such as:

```sql
SUBSTRING(string FROM start_position)
```

or:

```sql
SUBSTRING(string, start_position)
```

depending on the dialect.

The general concept is:

```text
start here
→ return the remaining substring
```

---

# 10. LEFT

## Definition

`LEFT` returns a specified number of characters from the beginning of a string.

Common syntax:

```sql
LEFT(string, number_of_characters)
```

Example:

```sql
SELECT LEFT('DATABASE', 4);
```

Result:

```text
DATA
```

---

# 11. LEFT — Practical Example

Extract the first three characters of a product code:

```sql
SELECT
    LEFT(product_code, 3) AS prefix
FROM products;
```

If:

```text
product_code = 'IND-48291'
```

result:

```text
IND
```

This is useful for:

- prefixes,
- region codes,
- product families,
- standardized identifiers.

---

# 12. RIGHT

## Definition

`RIGHT` returns a specified number of characters from the end of a string.

Common syntax:

```sql
RIGHT(string, number_of_characters)
```

Example:

```sql
SELECT RIGHT('DATABASE', 4);
```

Result:

```text
BASE
```

---

# 13. RIGHT — Practical Example

Extract the last four characters of an identifier:

```sql
SELECT
    RIGHT(account_number, 4) AS last_four
FROM accounts;
```

Example:

```text
account_number = 'ACC983742'
```

Result:

```text
3742
```

This pattern is often used when displaying only the final portion of an identifier.

---

# 14. LEFT / RIGHT vs SUBSTRING

| Requirement | Useful function |
|---|---|
| First N characters | `LEFT` |
| Last N characters | `RIGHT` |
| Characters from a specified position | `SUBSTRING` |

Examples:

```sql
LEFT(name, 3)
```

```sql
RIGHT(name, 4)
```

```sql
SUBSTRING(name, 2, 5)
```

Mental model:

```text
LEFT       → beginning
RIGHT      → end
SUBSTRING  → arbitrary position
```

---

# 15. LENGTH

## Definition

`LENGTH` returns the length of a string according to the database's string-length semantics.

Example:

```sql
SELECT LENGTH('DATABASE');
```

Result:

```text
8
```

because:

```text
D A T A B A S E
1 2 3 4 5 6 7 8
```

---

# 16. LENGTH — Practical Example

Find usernames longer than 10 characters:

```sql
SELECT username
FROM users
WHERE LENGTH(username) > 10;
```

---

# 17. LENGTH vs Character/Byte Length

Be careful with Unicode and multibyte characters.

Some database systems provide separate functions for:

```text
character length
byte length
```

Therefore, if an interview question involves Unicode or multibyte encodings, identify whether it asks for:

- characters,
- bytes,
- code units.

Do not assume they are always identical.

---

# 18. LOWER

`LOWER` converts alphabetic characters to lowercase.

```sql
SELECT LOWER('Hello World');
```

Result:

```text
hello world
```

Practical example:

```sql
SELECT LOWER(email) AS normalized_email
FROM users;
```

This is commonly used for normalization.

---

# 19. UPPER

`UPPER` converts alphabetic characters to uppercase.

```sql
SELECT UPPER('Hello World');
```

Result:

```text
HELLO WORLD
```

Example:

```sql
SELECT UPPER(country_code)
FROM customers;
```

---

# 20. LOWER + UPPER for Normalization

Suppose user input may contain inconsistent capitalization:

```text
Ravi@example.com
ravi@example.com
RAVI@example.com
```

You can normalize the representation:

```sql
LOWER(email)
```

resulting conceptually in:

```text
ravi@example.com
```

Be careful:

> Normalizing the displayed/search value does not automatically change the stored data.

---

# 21. Case Sensitivity and Collation

`LOWER()` and `UPPER()` are not the same thing as case-insensitive comparison.

Whether:

```sql
WHERE name = 'ravi'
```

matches:

```text
Ravi
```

depends on the database's comparison/collation rules.

If you explicitly need a case-normalized comparison, a common technique is:

```sql
WHERE LOWER(name) = LOWER('Ravi')
```

But applying a function to a column can affect index usage depending on the database and available indexes.

For performance-sensitive queries, consider the database's collation or functional/indexed-expression capabilities.

---

# 22. TRIM

## Definition

`TRIM` removes unwanted leading and/or trailing characters, most commonly spaces.

Basic form:

```sql
TRIM(string)
```

Example:

```sql
SELECT TRIM('   Ravi   ');
```

Result:

```text
Ravi
```

It removes whitespace from the ends, not from the middle.

---

# 23. TRIM Does Not Remove Internal Spaces

Given:

```text
'Ravi Kumar'
```

this:

```sql
TRIM('Ravi Kumar')
```

still returns:

```text
Ravi Kumar
```

It does not turn it into:

```text
RaviKumar
```

If you want to remove internal spaces, use an appropriate replacement:

```sql
REPLACE('Ravi Kumar', ' ', '')
```

which produces:

```text
RaviKumar
```

---

# 24. Leading vs Trailing Whitespace

Many DBMSs also provide:

```text
LTRIM
RTRIM
```

Conceptually:

```text
LTRIM → remove leading whitespace
RTRIM → remove trailing whitespace
TRIM  → remove leading and trailing whitespace
```

Exact support varies by DBMS.

---

# 25. TRIM Specific Characters

Standard SQL supports forms of `TRIM` that can remove specified characters.

Conceptually:

```sql
TRIM(BOTH '-' FROM '---ABC---')
```

returns:

```text
ABC
```

Exact syntax differs across database systems.

Important distinction:

```text
TRIM removes characters from the ends.
```

It does not generally remove matching characters from the middle.

---

# 26. REPLACE

## Definition

`REPLACE` replaces occurrences of one substring with another.

Common syntax:

```sql
REPLACE(string, old_substring, new_substring)
```

Example:

```sql
SELECT REPLACE('hello world', 'world', 'SQL');
```

Result:

```text
hello SQL
```

---

# 27. REPLACE — Removing Characters

To remove spaces:

```sql
SELECT REPLACE('Ravi Kumar', ' ', '');
```

Result:

```text
RaviKumar
```

To remove hyphens:

```sql
REPLACE('123-456-789', '-', '')
```

Result:

```text
123456789
```

This is useful for data cleaning.

---

# 28. REPLACE — Multiple Occurrences

If the target substring occurs multiple times, `REPLACE` generally replaces each matching occurrence.

Example:

```sql
SELECT REPLACE('a-b-c-d', '-', '/');
```

Result:

```text
a/b/c/d
```

---

# 29. REPLACE Is Not the Same as TRIM

Compare:

```sql
TRIM('---ABC---')
```

with:

```sql
REPLACE('---ABC---', '-', '')
```

Conceptually:

```text
TRIM
→ removes specified characters from the ends

REPLACE
→ replaces occurrences throughout the string
```

Therefore:

```text
TRIM → boundary cleaning
REPLACE → substitution
```

---

# 30. Pattern Matching

Pattern matching asks whether a string follows a specified pattern.

The most common SQL mechanism is:

```sql
LIKE
```

Example:

```sql
SELECT *
FROM customers
WHERE name LIKE 'R%';
```

This finds names beginning with `R`.

---

# 31. LIKE Wildcard: %

`%` means:

```text
zero or more characters
```

Examples:

```sql
WHERE name LIKE 'A%'
```

means:

```text
starts with A
```

Examples matching:

```text
A
Arun
Alice
Amazon
```

---

# 32. LIKE Wildcard: %

Example:

```sql
WHERE name LIKE '%son'
```

means:

```text
ends with son
```

Possible matches:

```text
Jackson
Anderson
Wilson
```

---

# 33. LIKE with % on Both Sides

```sql
WHERE name LIKE '%ram%'
```

means:

```text
contains "ram" somewhere
```

Possible matches:

```text
Ramesh
program
grammar
```

Case sensitivity depends on the database/collation.

---

# 34. LIKE Wildcard: _

`_` means:

```text
exactly one character
```

Example:

```sql
WHERE code LIKE 'A_1'
```

Possible matches:

```text
AB1
AC1
AX1
```

But:

```text
A1
ABC1
```

do not match this three-character pattern.

---

# 35. `%` vs `_`

| Wildcard | Meaning |
|---|---|
| `%` | Zero or more characters |
| `_` | Exactly one character |

Memorize this.

---

# 36. Pattern Examples

Assume:

```text
name
----
Ravi
Rahul
Ramesh
Amit
Ram
```

### Starts with R

```sql
WHERE name LIKE 'R%'
```

Matches:

```text
Ravi
Rahul
Ramesh
Ram
```

### Ends with i

```sql
WHERE name LIKE '%i'
```

Matches:

```text
Ravi
Amit
```

### Contains `am`

```sql
WHERE name LIKE '%am%'
```

Matches strings containing that sequence according to the database's comparison rules.

### Exactly four characters

```sql
WHERE name LIKE '____'
```

Each `_` represents one character.

---

# 37. NOT LIKE

Use:

```sql
WHERE name NOT LIKE 'A%'
```

to find values that do not match the pattern.

As with other SQL predicates, NULL values require separate consideration because:

```text
NULL LIKE pattern
```

does not evaluate to TRUE.

---

# 38. Escaping Wildcards

Sometimes `%` or `_` is actual data rather than a wildcard.

For example, suppose you need to search for:

```text
50%
```

You need to escape `%`.

A common pattern is:

```sql
WHERE discount_code LIKE '50\%' ESCAPE '\'
```

Exact escaping syntax can vary by DBMS.

The concept is:

```text
wildcard character
      ↓
escape it
      ↓
treat it as literal text
```

---

# 39. LIKE and Case Sensitivity

Do not assume `LIKE` is always case-sensitive or always case-insensitive.

It depends on:

- DBMS,
- collation,
- data type,
- database configuration.

If you need explicit case normalization:

```sql
WHERE LOWER(name) LIKE 'r%'
```

can be used.

But again, applying functions to columns can have indexing/performance implications.

---

# 40. LIKE vs Equality

These are different:

```sql
WHERE name = 'Ravi'
```

means exact comparison.

```sql
WHERE name LIKE 'Ravi'
```

uses pattern-matching semantics, although without wildcards it may behave like an exact pattern match depending on the database.

```sql
WHERE name LIKE 'R%'
```

means:

```text
starts with R
```

---

# 41. Pattern Matching with SUBSTRING

You can combine functions and pattern logic.

Example:

```sql
WHERE LOWER(email) LIKE '%@gmail.com'
```

This checks the normalized string for the specified suffix.

Another example:

```sql
WHERE LEFT(code, 3) = 'IND'
```

checks a prefix.

There are often multiple valid formulations.

---

# 42. Function Composition

String functions can be nested.

Example:

```sql
LOWER(TRIM(email))
```

Execution conceptually:

```text
original email
    ↓
TRIM
    ↓
remove surrounding whitespace
    ↓
LOWER
    ↓
normalize case
```

Another example:

```sql
UPPER(REPLACE(phone, '-', ''))
```

This can normalize multiple properties at once.

---

# 43. Data Cleaning Pipeline

A common SQL data-cleaning pattern is:

```sql
LOWER(
    TRIM(
        email
    )
)
```

For more complicated normalization:

```sql
LOWER(
    REPLACE(
        TRIM(email),
        ' ',
        ''
    )
)
```

The exact cleaning logic should reflect the data requirements.

Do not blindly remove spaces from values where spaces are meaningful.

---

# 44. String Functions in WHERE

Example:

```sql
SELECT *
FROM users
WHERE LOWER(TRIM(email)) = 'ravi@example.com';
```

This can normalize both sides conceptually.

But applying functions to a column may prevent a normal index from being used efficiently in some systems.

For high-performance systems, consider:

- normalized stored columns,
- functional indexes,
- expression indexes,
- generated/computed columns,
- appropriate collations.

---

# 45. SARGability Consideration

Compare:

```sql
WHERE email = 'ravi@example.com'
```

with:

```sql
WHERE LOWER(email) = 'ravi@example.com'
```

The second applies a function to the column.

Depending on the database and indexes, that can make index use less effective.

This does not mean:

```text
never use functions in WHERE
```

It means:

> Understand the performance implications when the column is indexed and the query is performance-sensitive.

---

# 46. Prefix Search vs Contains Search

Compare:

```sql
WHERE name LIKE 'R%'
```

with:

```sql
WHERE name LIKE '%R%'
```

The first is a prefix search.

The second is a contains search.

A leading wildcard:

```text
%R
%R%
```

can make ordinary B-tree index usage more difficult because the beginning of the value is unknown.

The exact optimizer behavior depends on the database.

---

# 47. String Function Summary

| Function | Main purpose | Example |
|---|---|---|
| `CONCAT` | Combine strings | `CONCAT(first, ' ', last)` |
| `SUBSTRING` | Extract arbitrary portion | `SUBSTRING(code, 2, 4)` |
| `LEFT` | First N characters | `LEFT(code, 3)` |
| `RIGHT` | Last N characters | `RIGHT(code, 4)` |
| `LENGTH` | String length | `LENGTH(name)` |
| `LOWER` | Lowercase | `LOWER(email)` |
| `UPPER` | Uppercase | `UPPER(code)` |
| `TRIM` | Remove surrounding characters/whitespace | `TRIM(name)` |
| `REPLACE` | Replace substring | `REPLACE(phone, '-', '')` |
| `LIKE` | Pattern matching | `name LIKE 'A%'` |

---

# 48. Function Selection Decision Tree

```text
Need to combine strings?
        ↓
     CONCAT

Need characters from the beginning?
        ↓
       LEFT

Need characters from the end?
        ↓
      RIGHT

Need characters from an arbitrary position?
        ↓
    SUBSTRING

Need string length?
        ↓
     LENGTH

Need lowercase?
        ↓
     LOWER

Need uppercase?
        ↓
     UPPER

Need to remove surrounding whitespace?
        ↓
      TRIM

Need to replace/remove a substring?
        ↓
    REPLACE

Need pattern matching?
        ↓
      LIKE
```

---

# 49. Common Interview Patterns

## Pattern 1 — Full name

```sql
CONCAT(first_name, ' ', last_name)
```

---

## Pattern 2 — First N characters

```sql
LEFT(code, 3)
```

---

## Pattern 3 — Last N characters

```sql
RIGHT(account_number, 4)
```

---

## Pattern 4 — Extract middle portion

```sql
SUBSTRING(code, start_position, length)
```

---

## Pattern 5 — Normalize case

```sql
LOWER(email)
```

---

## Pattern 6 — Remove surrounding spaces

```sql
TRIM(name)
```

---

## Pattern 7 — Remove punctuation

```sql
REPLACE(phone, '-', '')
```

---

## Pattern 8 — Prefix match

```sql
WHERE code LIKE 'IND%'
```

---

## Pattern 9 — Suffix match

```sql
WHERE email LIKE '%@company.com'
```

---

## Pattern 10 — Contains

```sql
WHERE description LIKE '%sql%'
```

---

## Pattern 11 — Exact N-character pattern

```sql
WHERE code LIKE 'A___'
```

---

# 50. GATE / Database Theory Concepts

## 50.1 String Operations Are Expressions

Functions such as:

```sql
LOWER(name)
LENGTH(name)
SUBSTRING(name, 1, 3)
```

produce values and can be used inside SQL expressions.

---

## 50.2 NULL Propagation

Many string functions return NULL when given NULL input, although exact behavior depends on the function/database.

For example, conceptually:

```sql
LOWER(NULL)
```

produces:

```text
NULL
```

Therefore, if you need a fallback:

```sql
COALESCE(LOWER(name), 'unknown')
```

---

## 50.3 LIKE and Three-Valued Logic

For:

```sql
WHERE name LIKE 'A%'
```

a NULL `name` does not satisfy the condition.

The predicate is not TRUE.

Therefore the row is not selected.

---

## 50.4 Wildcards

Remember:

```text
% → zero or more characters
_ → exactly one character
```

This is one of the most common theory questions.

---

# 51. GATE-Style Solved Questions

> These are **GATE-style practice questions**, not claims of official GATE PYQs.

## Question 1 — LENGTH

What is the result of:

```sql
LENGTH('DATABASE')
```

### Solution

Characters:

```text
D A T A B A S E
```

There are 8.

### Answer

```text
8
```

---

## Question 2 — LEFT and RIGHT

Evaluate:

```sql
LEFT('DATABASE', 3)
```

and:

```sql
RIGHT('DATABASE', 3)
```

### Solution

Beginning:

```text
DAT
```

End:

```text
ASE
```

### Answer

```text
LEFT  → DAT
RIGHT → ASE
```

---

## Question 3 — REPLACE

Evaluate:

```sql
REPLACE('A-B-C', '-', '')
```

### Solution

Every `-` is replaced with an empty string.

```text
A-B-C
 ↓
ABC
```

### Answer

```text
ABC
```

---

## Question 4 — LIKE %

Which rows match:

```sql
WHERE name LIKE 'A%'
```

for:

```text
Alice
Bob
Arun
Sam
A
```

### Solution

`A%` means:

```text
starts with A
```

Matches:

```text
Alice
Arun
A
```

### Answer

```text
Alice
Arun
A
```

---

## Question 5 — LIKE _

Which values match:

```sql
WHERE code LIKE 'A_1'
```

from:

```text
AB1
A11
ABC1
A1
AX1
```

### Solution

The pattern contains:

```text
A
_
1
```

so exactly three characters are required.

Matches:

```text
AB1
A11
AX1
```

### Answer

```text
AB1
A11
AX1
```

---

## Question 6 — Function Composition

Evaluate conceptually:

```sql
LOWER(
    TRIM('  SQL Interview  ')
)
```

### Solution

First:

```text
TRIM → 'SQL Interview'
```

Then:

```text
LOWER → 'sql interview'
```

### Answer

```text
sql interview
```

---

# 52. Interview Practice Questions

Try these independently.

### Q1
Create a `full_name` from `first_name` and `last_name`.

### Q2
Extract the first three characters of a product code.

### Q3
Extract the last four digits of an account number.

### Q4
Find users whose usernames contain `admin`.

### Q5
Find emails ending in `@company.com`.

### Q6
Normalize an email using lowercase and trimming.

### Q7
Remove hyphens from phone numbers.

### Q8
Count the number of characters in each product name.

### Q9
Extract five characters starting at position 3.

### Q10
Find codes consisting of exactly five characters where the first character is `A`.

---

# 53. Interview Solutions

## Q1

```sql
CONCAT(first_name, ' ', last_name)
```

## Q2

```sql
LEFT(product_code, 3)
```

## Q3

```sql
RIGHT(account_number, 4)
```

## Q4

```sql
WHERE username LIKE '%admin%'
```

## Q5

```sql
WHERE email LIKE '%@company.com'
```

## Q6

```sql
LOWER(TRIM(email))
```

## Q7

```sql
REPLACE(phone, '-', '')
```

## Q8

```sql
LENGTH(product_name)
```

## Q9

```sql
SUBSTRING(product_code, 3, 5)
```

Check the exact syntax/indexing convention for the target DBMS.

## Q10

```sql
WHERE code LIKE 'A____'
```

There are five positions:

```text
A + four underscores
```

---

# 54. Advanced Interview Patterns

## 54.1 Domain Extraction

For a basic email:

```text
ravi@example.com
```

the domain can often be extracted with dialect-specific string functions.

One possible approach is based on the position of `@`, but the exact expression differs across DBMSs.

Conceptually:

```text
find @
    ↓
extract everything after @
```

This is a good application of:

```text
SUBSTRING
+
position/search function
```

---

# 55. Prefix Classification

Suppose product codes are:

```text
IND-1001
USA-2001
JPN-3001
```

Classify by prefix:

```sql
CASE
    WHEN LEFT(product_code, 3) = 'IND' THEN 'India'
    WHEN LEFT(product_code, 3) = 'USA' THEN 'USA'
    WHEN LEFT(product_code, 3) = 'JPN' THEN 'Japan'
    ELSE 'Other'
END
```

This combines:

```text
LEFT + CASE
```

---

# 56. String Cleaning Before Comparison

Suppose imported names may have surrounding spaces and inconsistent case:

```text
'  Ravi  '
'ravi'
'RAVI'
```

A normalized comparison can be:

```sql
WHERE LOWER(TRIM(name)) = 'ravi'
```

This is useful in data-cleaning and matching problems.

---

# 57. Combining String Functions with CASE

Example:

```sql
CASE
    WHEN LENGTH(TRIM(username)) = 0 THEN 'Invalid'
    WHEN LENGTH(TRIM(username)) < 5 THEN 'Short'
    ELSE 'Valid'
END
```

This demonstrates:

```text
TRIM
+
LENGTH
+
CASE
```

---

# 58. Pattern Matching for Validation

A basic format check may use:

```sql
WHERE code LIKE 'IND-%'
```

This means the code starts with:

```text
IND-
```

followed by zero or more characters.

Important limitation:

> `LIKE` is pattern matching, not a full regular-expression validation system.

If exact structural validation is needed, some DBMSs provide regex functions/operators.

---

# 59. LIKE Is Not Regex

Do not confuse:

```sql
LIKE
```

with regular expressions.

`LIKE` primarily provides:

```text
% → zero or more
_ → exactly one
```

Regex systems can express much more complex patterns.

For example, regex may support:

```text
character classes
quantifiers
alternation
anchors
groups
```

The exact regex features are database-dependent.

For this topic, master `LIKE` first.

---

# 60. Common Mistakes

## Mistake 1 — Confusing `%` and `_`

Wrong:

```text
% → exactly one
_ → any number
```

Correct:

```text
% → zero or more
_ → exactly one
```

---

## Mistake 2 — Thinking TRIM removes internal spaces

```sql
TRIM('Ravi Kumar')
```

does not normally produce:

```text
RaviKumar
```

Use:

```sql
REPLACE('Ravi Kumar', ' ', '')
```

if removing internal spaces is actually intended.

---

## Mistake 3 — Using equality for prefix matching

Wrong:

```sql
WHERE name = 'A%'
```

Correct:

```sql
WHERE name LIKE 'A%'
```

---

## Mistake 4 — Forgetting NULL behavior

String functions and `LIKE` interact with NULL.

Do not expect:

```sql
LOWER(NULL)
```

to produce a string.

---

## Mistake 5 — Assuming all DBMSs use identical SUBSTRING syntax

Always check the target dialect.

---

## Mistake 6 — Assuming LIKE is universally case-insensitive

Case behavior depends on the database/collation.

---

## Mistake 7 — Using a leading wildcard without considering performance

```sql
LIKE '%abc%'
```

is a contains search and can be harder for ordinary indexes to optimize than a prefix search.

---

# 61. Performance Considerations

String functions are often cheap enough for ordinary reporting queries, but they can become expensive on large datasets.

Potential concerns:

```sql
WHERE LOWER(email) = 'ravi@example.com'
```

or:

```sql
WHERE TRIM(name) = 'Ravi'
```

The function is applied to many column values.

Possible solutions include:

- storing normalized values,
- generated/computed columns,
- functional indexes,
- expression indexes,
- appropriate collations,
- database-specific text-search features.

Do not optimize prematurely. First ensure the query semantics are correct.

---

# 62. Interview Decision Table

| Requirement | Pattern |
|---|---|
| Combine first and last name | `CONCAT` |
| First N characters | `LEFT` |
| Last N characters | `RIGHT` |
| Middle portion | `SUBSTRING` |
| String length | `LENGTH` |
| Normalize lowercase | `LOWER` |
| Normalize uppercase | `UPPER` |
| Remove surrounding spaces | `TRIM` |
| Replace/remove substring | `REPLACE` |
| Starts with | `LIKE 'x%'` |
| Ends with | `LIKE '%x'` |
| Contains | `LIKE '%x%'` |
| Exactly N arbitrary characters | `LIKE '____...'` |
| Literal `%` or `_` | Escape wildcard |

---

# 63. Master Pattern Library

## Full name

```sql
CONCAT(first_name, ' ', last_name)
```

## Normalize text

```sql
LOWER(TRIM(text_column))
```

## Remove punctuation

```sql
REPLACE(text_column, '-', '')
```

## Prefix

```sql
LEFT(code, 3)
```

## Suffix

```sql
RIGHT(code, 4)
```

## Arbitrary substring

```sql
SUBSTRING(code, start_position, length)
```

## Length filter

```sql
WHERE LENGTH(code) = 5
```

## Starts with

```sql
WHERE code LIKE 'ABC%'
```

## Ends with

```sql
WHERE code LIKE '%XYZ'
```

## Contains

```sql
WHERE code LIKE '%SQL%'
```

## One-character wildcard

```sql
WHERE code LIKE 'A_1'
```

---

# 64. GATE Quick Revision

### Q: `%` means?

```text
Zero or more characters
```

### Q: `_` means?

```text
Exactly one character
```

### Q: Function for first N characters?

```text
LEFT
```

### Q: Function for last N characters?

```text
RIGHT
```

### Q: Function for arbitrary substring?

```text
SUBSTRING
```

### Q: Function for string length?

```text
LENGTH
```

### Q: Lowercase?

```text
LOWER
```

### Q: Uppercase?

```text
UPPER
```

### Q: Remove leading/trailing whitespace?

```text
TRIM
```

### Q: Replace occurrences?

```text
REPLACE
```

### Q: Combine strings?

```text
CONCAT
```

---

# 65. Mastery Checklist

You should be able to:

- [ ] Explain what string functions do.
- [ ] Use `CONCAT`.
- [ ] Combine first and last names.
- [ ] Explain NULL behavior relevant to concatenation.
- [ ] Use `SUBSTRING`.
- [ ] Extract a substring from a specified position.
- [ ] Explain dialect differences in substring syntax.
- [ ] Use `LEFT`.
- [ ] Use `RIGHT`.
- [ ] Distinguish `LEFT`, `RIGHT`, and `SUBSTRING`.
- [ ] Use `LENGTH`.
- [ ] Understand character-length vs byte-length considerations.
- [ ] Use `LOWER`.
- [ ] Use `UPPER`.
- [ ] Explain why case normalization is not identical to collation.
- [ ] Use `TRIM`.
- [ ] Distinguish `TRIM` from internal-space removal.
- [ ] Use `REPLACE`.
- [ ] Remove punctuation using `REPLACE`.
- [ ] Use `LIKE`.
- [ ] Explain `%`.
- [ ] Explain `_`.
- [ ] Write prefix searches.
- [ ] Write suffix searches.
- [ ] Write contains searches.
- [ ] Write exact-length wildcard patterns.
- [ ] Escape literal `%` and `_`.
- [ ] Understand NULL behavior with `LIKE`.
- [ ] Recognize case-sensitivity differences across DBMSs.
- [ ] Understand basic string-function performance concerns.
- [ ] Combine string functions with `CASE`.
- [ ] Combine string functions with aggregation and filtering.
- [ ] Solve string-function interview problems without relying on memorized examples.

---

# 66. Final Revision Sheet

## Functions

```text
CONCAT     → combine
SUBSTRING  → arbitrary portion
LEFT       → beginning
RIGHT      → end
LENGTH     → length
LOWER      → lowercase
UPPER      → uppercase
TRIM       → surrounding whitespace/characters
REPLACE    → substitute substring
```

## LIKE

```text
% → zero or more characters
_ → exactly one character
```

## Patterns

```sql
-- Starts with ABC
LIKE 'ABC%'

-- Ends with ABC
LIKE '%ABC'

-- Contains ABC
LIKE '%ABC%'

-- Exactly 4 characters
LIKE '____'

-- Starts with A and has exactly 3 characters
LIKE 'A__'
```

## Cleaning

```sql
LOWER(TRIM(email))
```

## Remove characters

```sql
REPLACE(phone, '-', '')
```

## Prefix

```sql
LEFT(code, 3)
```

## Suffix

```sql
RIGHT(code, 4)
```

## Substring

```sql
SUBSTRING(code, start_position, length)
```

---

# 67. Final Takeaway

The core skill is mapping the requirement to the correct string operation:

```text
combine              → CONCAT
take from beginning  → LEFT
take from end        → RIGHT
take from middle     → SUBSTRING
measure              → LENGTH
normalize lowercase  → LOWER
normalize uppercase  → UPPER
clean boundaries     → TRIM
replace/remove text  → REPLACE
search a pattern     → LIKE
```

The two pattern-matching rules to memorize are:

```text
% → zero or more characters
_ → exactly one character
```

And the most important interview distinction is:

```text
TRIM
→ cleans the boundaries

REPLACE
→ substitutes occurrences throughout the string
```

Finally, remember that string-function syntax and behavior can vary by DBMS, particularly for:

- `SUBSTRING`,
- concatenation,
- string length,
- NULL handling,
- case sensitivity,
- pattern escaping.

For interviews, master the concepts first and then match the syntax to the stated SQL dialect.
