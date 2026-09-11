# Bit Manipulation Basics

## 1. Why This Topic Matters

Bit manipulation is the ability to operate directly on the binary representation of an integer.

For FAANG and top-product interviews, the important skill is not memorizing random bit tricks. It is recognizing when a problem can be reduced to:

- inspecting one bit,
- changing one bit,
- counting bits,
- exploiting XOR cancellation,
- using powers of two,
- or compressing state into a bit representation.

Bit operations are especially useful because they are constant-time primitive operations on fixed-width Java integers.

Typical interview uses:

- checking whether a bit is set,
- setting/clearing/toggling flags,
- checking powers of two,
- counting set bits,
- finding a unique element with XOR,
- finding a missing number with XOR,
- representing subsets with bitmasks,
- state compression in dynamic programming,
- parity and checksum-style reasoning.

The goal of this topic is to make the following expressions intuitive rather than magical:

```java
x & 1
x | (1 << k)
x & ~(1 << k)
x ^ (1 << k)
x & (x - 1)
x & -x
x ^ y
```

---

## 2. Prerequisites

You should be comfortable with:

- decimal and binary representation,
- hexadecimal notation at a basic level,
- Java `int` and `long`,
- basic arithmetic,
- loops and conditionals,
- arrays.

You do **not** need advanced mathematics.

---

# 3. Core Concept

## 3.1 Integers Are Stored as Bits

An integer is represented using binary digits:

```text
decimal 13 = binary 1101
```

For an 8-bit illustration:

```text
13 = 00001101
        ^^^^
        bits
```

Each position represents a power of two:

```text
7 6 5 4 3 2 1 0   <- bit position
0 0 0 0 1 1 0 1
        8 4 1     <- contribution
```

Bit position `0` is the least significant bit (LSB).

The rightmost bit is the LSB.

The leftmost bit is the most significant bit (MSB) within the chosen width.

---

## 3.2 Bit Positions

For an integer:

```text
... b3 b2 b1 b0
```

the value of bit `k` is:

```text
2^k
```

Examples:

```text
bit 0 -> 1
bit 1 -> 2
bit 2 -> 4
bit 3 -> 8
bit 4 -> 16
```

Therefore:

```java
1 << k
```

creates a value with only bit `k` set.

Examples:

```java
1 << 0 = 0001
1 << 1 = 0010
1 << 2 = 0100
1 << 3 = 1000
```

This is the foundation of almost every basic bit-manipulation technique.

---

# 4. Bitwise Operators

Java provides these important bitwise operators for integers:

| Operator | Name | Meaning |
|---|---|---|
| `&` | AND | 1 only if both bits are 1 |
| `|` | OR | 1 if either bit is 1 |
| `^` | XOR | 1 if the bits are different |
| `~` | NOT | flips every bit |
| `<<` | left shift | shifts bits left |
| `>>` | signed right shift | shifts right and preserves sign |
| `>>>` | unsigned right shift | shifts right and fills with 0 |

---

# 5. AND

## 5.1 Rule

```text
1 & 1 = 1
1 & 0 = 0
0 & 1 = 0
0 & 0 = 0
```

Truth table:

| A | B | A & B |
|---|---|---|
| 0 | 0 | 0 |
| 0 | 1 | 0 |
| 1 | 0 | 0 |
| 1 | 1 | 1 |

Example:

```text
  1101
& 1011
------
  1001
```

So:

```java
13 & 11
```

produces:

```text
9
```

## 5.2 Main Uses

AND is commonly used to:

- test whether a bit is set,
- clear selected bits,
- extract selected bits,
- isolate the lowest set bit,
- remove the lowest set bit.

The most important interview expression is:

```java
x & (1 << k)
```

It checks whether bit `k` is set.

Example:

```text
x = 13 = 1101
k = 2

1 << 2 = 0100

1101
0100
----
0100
```

Non-zero result means bit 2 is set.

---

# 6. OR

## 6.1 Rule

```text
1 | 1 = 1
1 | 0 = 1
0 | 1 = 1
0 | 0 = 0
```

Example:

```text
  1101
| 0010
------
  1111
```

## 6.2 Main Use: Set a Bit

To set bit `k`:

```java
x = x | (1 << k);
```

or:

```java
x |= (1 << k);
```

Why?

The mask:

```text
1 << k
```

has only bit `k` equal to 1.

OR with it forces that bit to 1 without changing the other bits.

Example:

```text
x = 1001
k = 1

mask = 0010

1001
0010
----
1011
```

Bit 1 is now set.

---

# 7. XOR

## 7.1 Rule

XOR means "different".

```text
0 ^ 0 = 0
0 ^ 1 = 1
1 ^ 0 = 1
1 ^ 1 = 0
```

Truth table:

| A | B | A ^ B |
|---|---|---|
| 0 | 0 | 0 |
| 0 | 1 | 1 |
| 1 | 0 | 1 |
| 1 | 1 | 0 |

Example:

```text
  1101
^ 1011
------
  0110
```

So:

```java
13 ^ 11
```

is:

```text
6
```

---

## 7.2 XOR Properties

These are extremely important for interviews.

### Identity

```text
x ^ 0 = x
```

### Self-cancellation

```text
x ^ x = 0
```

### Commutativity

```text
x ^ y = y ^ x
```

### Associativity

```text
(x ^ y) ^ z = x ^ (y ^ z)
```

These properties allow rearranging a large XOR expression.

For example:

```text
a ^ b ^ c ^ b ^ a
```

Rearrange:

```text
a ^ a ^ b ^ b ^ c
```

Then:

```text
0 ^ 0 ^ c
= c
```

This is the core of the classic "single number" problem.

---

# 8. NOT

NOT flips every bit.

```text
~0 = all 1s
~1 = all 0s
```

For an 8-bit illustration:

```text
~ 00101100
  --------
  11010011
```

In Java, `int` is 32 bits, so:

```java
~x
```

flips all 32 bits.

## 8.1 Important Java Detail

Java uses signed two's-complement representation for `int`.

Therefore:

```java
~x == -x - 1
```

For example:

```java
~5
```

is:

```text
-6
```

because:

```text
~x = -x - 1
```

This identity is useful for understanding Java's negative integer behavior.

---

# 9. Left Shift

Operator:

```java
x << k
```

shifts bits left by `k` positions.

For positive values, a left shift by one is equivalent to multiplying by 2 when no relevant information is lost:

```text
0011 << 1 = 0110
```

Therefore:

```text
x << k ≈ x * 2^k
```

for ordinary positive values where overflow is not an issue.

Example:

```java
5 << 2
```

Binary:

```text
0101 << 2
= 10100
= 20
```

So:

```text
5 * 4 = 20
```

---

## 9.1 Creating Bit Masks

The most common use is:

```java
1 << k
```

Examples:

```java
1 << 0  // 0001
1 << 1  // 0010
1 << 2  // 0100
1 << 3  // 1000
```

This gives a mask with exactly one bit set.

---

# 10. Right Shift

Java has two right-shift operators.

## 10.1 Signed Right Shift: `>>`

```java
x >> k
```

For positive numbers, zeros enter from the left.

For negative numbers, the sign bit is propagated.

Example:

```text
20 = 10100

20 >> 2
= 00101
= 5
```

For positive integers:

```text
x >> k ≈ x / 2^k
```

with integer truncation.

---

## 10.2 Unsigned Right Shift: `>>>`

```java
x >>> k
```

Zeros are shifted in from the left regardless of the sign.

This distinction matters when manipulating all 32 bits of a signed Java `int`.

Example:

```java
int x = -1;

x >> 1
```

remains:

```text
11111111111111111111111111111111
```

whereas:

```java
x >>> 1
```

becomes:

```text
01111111111111111111111111111111
```

which is:

```text
2147483647
```

### Interview Rule

For ordinary non-negative bit manipulation:

- `>>` is usually sufficient.
- Use `>>>` when you explicitly need zero-fill right shifting.

---

# 11. Set a Bit

## Goal

Make bit `k` equal to `1`.

### Formula

```java
x |= (1 << k);
```

### Example

```text
x = 1001
k = 1

mask = 0010

1001
0010
----
1011
```

### Java Template

```java
static int setBit(int x, int k) {
    return x | (1 << k);
}
```

### Complexity

```text
Time:  O(1)
Space: O(1)
```

---

# 12. Unset / Clear a Bit

## Goal

Make bit `k` equal to `0`.

### Formula

```java
x &= ~(1 << k);
```

Why?

First:

```java
1 << k
```

creates:

```text
00010000
```

Then:

```java
~(1 << k)
```

creates a mask with every bit set except bit `k`:

```text
11101111
```

ANDing with that mask clears bit `k`.

### Example

```text
x = 1111
k = 2

1 << 2 = 0100
~mask  = 1011

1111
1011
----
1011
```

### Java Template

```java
static int clearBit(int x, int k) {
    return x & ~(1 << k);
}
```

---

# 13. Toggle a Bit

## Goal

Flip bit `k`.

```text
0 -> 1
1 -> 0
```

### Formula

```java
x ^= (1 << k);
```

Why XOR?

```text
0 ^ 1 = 1
1 ^ 1 = 0
```

Therefore XOR with a mask containing a single 1 toggles that position.

### Example

```text
x = 1001
k = 1

mask = 0010

1001
0010
----
1011
```

Run it again:

```text
1011
0010
----
1001
```

### Java Template

```java
static int toggleBit(int x, int k) {
    return x ^ (1 << k);
}
```

---

# 14. Check Whether a Bit Is Set

This is one of the most important basic operations.

### Formula

```java
(x & (1 << k)) != 0
```

### Java

```java
static boolean isBitSet(int x, int k) {
    return (x & (1 << k)) != 0;
}
```

Example:

```text
x = 13 = 1101
```

Check bit 2:

```text
1101
0100
----
0100
```

Non-zero -> set.

Check bit 1:

```text
1101
0010
----
0000
```

Zero -> not set.

---

# 15. Count Set Bits

A set bit is a bit whose value is `1`.

Example:

```text
13 = 1101
```

has:

```text
3 set bits
```

There are several useful techniques.

---

## 15.1 Simple Bit-by-Bit Method

Check every bit.

```java
static int countSetBits(int x) {
    int count = 0;

    while (x != 0) {
        count += x & 1;
        x >>>= 1;
    }

    return count;
}
```

### Why it works

```java
x & 1
```

extracts the least significant bit.

Then:

```java
x >>>= 1;
```

moves the next bit into the least significant position.

### Complexity

For a fixed 32-bit integer:

```text
Time: O(32) = O(1)
Space: O(1)
```

For conceptual analysis over a `w`-bit integer:

```text
Time: O(w)
Space: O(1)
```

---

# 16. Brian Kernighan's Algorithm

A much more important interview trick is:

```java
x &= (x - 1);
```

This removes the **lowest set bit**.

Example:

```text
x     = 1011000
x - 1 = 1010111

1011000
1010111
-------
1010000
```

The rightmost `1` disappeared.

Therefore:

```java
while (x != 0) {
    x &= (x - 1);
    count++;
}
```

counts set bits.

### Java Template

```java
static int countSetBits(int x) {
    int count = 0;

    while (x != 0) {
        x &= (x - 1);
        count++;
    }

    return count;
}
```

### Complexity

If `k` is the number of set bits:

```text
Time:  O(k)
Space: O(1)
```

This can be much faster than checking every bit when `k` is small.

### Interview Insight

Whenever you see:

```java
x & (x - 1)
```

think:

> Remove the lowest set bit.

This is one of the highest-value bit-manipulation identities to memorize.

---

# 17. Lowest Set Bit

Another important expression is:

```java
x & -x
```

It isolates the lowest set bit.

Example:

```text
x = 12 = 1100

-x = 0100  (conceptually in the relevant width)

1100
0100
----
0100
```

Result:

```text
4
```

For:

```text
x = 40 = 101000
```

the lowest set bit is:

```text
001000
```

which is:

```text
8
```

So:

```java
x & -x
```

returns the value of the lowest set bit.

### Why?

In two's complement:

```text
-x = ~x + 1
```

The operation causes the lowest set bit of `x` to remain isolated when ANDed with `x`.

### Common Uses

- Fenwick trees,
- extracting the lowest set bit,
- subset/bitmask algorithms,
- advanced bit tricks.

---

# 18. Remove the Lowest Set Bit

Formula:

```java
x &= (x - 1);
```

Example:

```text
x = 10110100
x - 1 = 10110011

AND:
10110100
10110011
--------
10110000
```

The lowest `1` is removed.

This is different from:

```java
x & -x
```

which **isolates** the lowest set bit.

Remember:

```text
x & -x       -> keep only lowest set bit
x & (x - 1)  -> remove lowest set bit
```

---

# 19. Power of Two

A positive integer is a power of two if its binary representation contains exactly one set bit.

Examples:

```text
1  = 0001
2  = 0010
4  = 0100
8  = 1000
16 = 10000
```

Non-powers:

```text
3  = 0011
6  = 0110
10 = 1010
12 = 1100
```

Since:

```java
x & (x - 1)
```

removes one set bit, a number with exactly one set bit becomes zero.

Therefore:

```java
x > 0 && (x & (x - 1)) == 0
```

checks whether `x` is a power of two.

### Java Template

```java
static boolean isPowerOfTwo(int x) {
    return x > 0 && (x & (x - 1)) == 0;
}
```

### Why `x > 0`?

Without it:

```java
0 & (-1) == 0
```

so zero would incorrectly appear to be a power of two.

Zero is not a power of two.

---

# 20. Power of Two: Examples

```text
x = 8

1000
0111
----
0000
```

Therefore 8 is a power of two.

For:

```text
x = 12

1100
1011
----
1000
```

Result is non-zero.

Therefore 12 is not a power of two.

---

# 21. XOR Tricks

XOR is the most important part of basic bit manipulation for coding interviews.

## Trick 1: Cancel Equal Values

```text
x ^ x = 0
```

Example:

```text
5 ^ 5 = 0
```

---

## Trick 2: XOR With Zero

```text
x ^ 0 = x
```

---

## Trick 3: Find One Unique Number

Suppose every number occurs twice except one:

```text
[4, 1, 2, 1, 2]
```

XOR everything:

```text
4 ^ 1 ^ 2 ^ 1 ^ 2
```

Rearrange:

```text
4 ^ (1 ^ 1) ^ (2 ^ 2)
```

Then:

```text
4 ^ 0 ^ 0
= 4
```

### Java

```java
static int singleNumber(int[] nums) {
    int result = 0;

    for (int num : nums) {
        result ^= num;
    }

    return result;
}
```

### Complexity

```text
Time:  O(n)
Space: O(1)
```

This works because XOR cancels equal values.

---

# 22. XOR Trick: Find Missing Number

Suppose:

```text
nums = [3, 0, 1]
```

The numbers should be:

```text
0, 1, 2, 3
```

Missing:

```text
2
```

XOR all indices and all values:

```text
(0 ^ 1 ^ 2 ^ 3) ^ (3 ^ 0 ^ 1)
```

Cancel equal values:

```text
2
```

### Java

```java
static int missingNumber(int[] nums) {
    int result = nums.length;

    for (int i = 0; i < nums.length; i++) {
        result ^= i;
        result ^= nums[i];
    }

    return result;
}
```

### Complexity

```text
Time:  O(n)
Space: O(1)
```

---

# 23. XOR Trick: Two Unique Numbers

Suppose every value occurs twice except two values:

```text
[1, 2, 1, 3, 2, 5]
```

Unique values:

```text
3 and 5
```

First XOR everything:

```text
1 ^ 2 ^ 1 ^ 3 ^ 2 ^ 5
= 3 ^ 5
```

The result is:

```text
3 ^ 5
```

Since `3 != 5`, at least one bit differs.

Find a set bit in:

```java
xor = 3 ^ 5;
```

A convenient choice is:

```java
int mask = xor & -xor;
```

This isolates one differing bit.

Then partition all numbers into two groups:

- numbers where that bit is 0,
- numbers where that bit is 1.

The two unique values go into different groups, while each duplicate pair remains in the same group and cancels.

### Java

```java
static int[] singleNumberTwo(int[] nums) {
    int xor = 0;

    for (int num : nums) {
        xor ^= num;
    }

    int mask = xor & -xor;

    int first = 0;
    int second = 0;

    for (int num : nums) {
        if ((num & mask) == 0) {
            first ^= num;
        } else {
            second ^= num;
        }
    }

    return new int[]{first, second};
}
```

### Complexity

```text
Time:  O(n)
Space: O(1)
```

---

# 24. XOR Trick: XOR From 1 to N

The XOR from:

```text
1 ^ 2 ^ 3 ^ ... ^ n
```

follows a pattern based on `n % 4`.

```text
n % 4 == 0 -> n
n % 4 == 1 -> 1
n % 4 == 2 -> n + 1
n % 4 == 3 -> 0
```

### Example

For:

```text
n = 7
```

because:

```text
7 % 4 = 3
```

the answer is:

```text
0
```

Indeed:

```text
1 ^ 2 ^ 3 ^ 4 ^ 5 ^ 6 ^ 7 = 0
```

### Range XOR

To find:

```text
L ^ (L+1) ^ ... ^ R
```

use:

```text
xor(1..R) ^ xor(1..L-1)
```

because all values before `L` cancel.

### Java

```java
static int xorFrom1ToN(int n) {
    switch (n & 3) {
        case 0:
            return n;
        case 1:
            return 1;
        case 2:
            return n + 1;
        default:
            return 0;
    }
}

static int xorRange(int left, int right) {
    return xorFrom1ToN(right) ^ xorFrom1ToN(left - 1);
}
```

---

# 25. Useful Bit Identities

These are worth memorizing.

| Expression | Meaning |
|---|---|
| `x & 1` | extract lowest bit |
| `x | (1 << k)` | set bit `k` |
| `x & ~(1 << k)` | clear bit `k` |
| `x ^ (1 << k)` | toggle bit `k` |
| `(x & (1 << k)) != 0` | test bit `k` |
| `x & (x - 1)` | remove lowest set bit |
| `x & -x` | isolate lowest set bit |
| `x ^ x` | `0` |
| `x ^ 0` | `x` |
| `x > 0 && (x & (x - 1)) == 0` | power of two |
| `x << k` | left shift by `k` |
| `x >> k` | signed right shift |
| `x >>> k` | unsigned right shift |

---

# 26. General Java Templates

## Check Bit

```java
static boolean isBitSet(int x, int k) {
    return (x & (1 << k)) != 0;
}
```

## Set Bit

```java
static int setBit(int x, int k) {
    return x | (1 << k);
}
```

## Clear Bit

```java
static int clearBit(int x, int k) {
    return x & ~(1 << k);
}
```

## Toggle Bit

```java
static int toggleBit(int x, int k) {
    return x ^ (1 << k);
}
```

## Count Set Bits

```java
static int countSetBits(int x) {
    int count = 0;

    while (x != 0) {
        x &= (x - 1);
        count++;
    }

    return count;
}
```

For negative values, use an explicit policy depending on whether you want the 32-bit representation:

```java
static int countSetBits32(int x) {
    return Integer.bitCount(x);
}
```

## Power of Two

```java
static boolean isPowerOfTwo(int x) {
    return x > 0 && (x & (x - 1)) == 0;
}
```

---

# 27. Java Built-ins You Should Know

Java provides useful bit-related methods.

```java
Integer.bitCount(x)
Integer.numberOfLeadingZeros(x)
Integer.numberOfTrailingZeros(x)
Integer.highestOneBit(x)
Integer.lowestOneBit(x)
Integer.reverse(x)
Integer.reverseBytes(x)
```

For `long`:

```java
Long.bitCount(x)
Long.numberOfLeadingZeros(x)
Long.numberOfTrailingZeros(x)
Long.highestOneBit(x)
Long.lowestOneBit(x)
Long.reverse(x)
```

## Interview Rule

Know how to implement the basic operation manually, even if Java provides a built-in.

For example:

```java
Integer.bitCount(x)
```

is useful in production code, but an interview may explicitly ask you to count bits without using a built-in.

---

# 28. Integer Width and Shift Pitfalls

Java `int` is 32 bits.

Java `long` is 64 bits.

A common mistake is assuming:

```java
1 << 32
```

creates a 32nd-bit mask.

It does not behave as many beginners expect because Java masks the shift distance for `int`.

For a 64-bit mask, use:

```java
1L << k
```

Example:

```java
long mask = 1L << 40;
```

not:

```java
int mask = 1 << 40;
```

For bit positions beyond 30 in an `int`, be especially careful because the sign bit is involved.

---

# 29. Signed Numbers and Two's Complement

Java represents signed integers using two's complement.

For a positive number:

```text
5 = 00000000 00000000 00000000 00000101
```

Negative numbers are represented by:

```text
invert bits + 1
```

For example:

```text
5:
00000101

invert:
11111010

+1:
11111011
```

This is `-5`.

This explains:

```java
-x == ~x + 1
```

and why:

```java
x & -x
```

can isolate the lowest set bit.

---

# 30. When NOT to Use Bit Manipulation

Do not use bit tricks just to look clever.

Avoid them when:

- ordinary arithmetic is clearer,
- readability is significantly worse,
- overflow behavior becomes difficult to reason about,
- the problem does not naturally involve binary state,
- Java's built-in operation is clearer and allowed.

For example, this:

```java
x *= 2;
```

is usually clearer than:

```java
x <<= 1;
```

unless the problem is explicitly about bit manipulation or the shift has semantic meaning.

The goal is not "use bits everywhere."

The goal is:

> Recognize when bits give a simpler or more efficient representation.

---

# 31. How to Recognize Bit-Manipulation Problems

Look for phrases such as:

- "binary representation"
- "bit"
- "set bit"
- "unset bit"
- "toggle"
- "lowest set bit"
- "number of 1s"
- "power of two"
- "appears once while others appear twice"
- "all numbers occur twice except..."
- "XOR"
- "mask"
- "subset"
- "flags"
- "state compression"
- "without using extra space"

Strong signals:

### Signal 1: Duplicate cancellation

If every value appears twice except one:

```text
XOR
```

should be your first thought.

### Signal 2: Exactly one set bit

Think:

```java
x > 0 && (x & (x - 1)) == 0
```

### Signal 3: Change one position

Think:

```java
1 << k
```

then combine it with:

```text
OR   -> set
AND  -> clear
XOR  -> toggle
```

### Signal 4: Count ones

Think:

```java
x &= x - 1
```

### Signal 5: Lowest set bit

Think:

```java
x & -x
```

---

# 32. Common Mistakes

## Mistake 1: Confusing `&` and `&&`

```java
&
```

is bitwise AND.

```java
&&
```

is logical AND.

They are not interchangeable.

---

## Mistake 2: Confusing `|` and `||`

```java
|
```

is bitwise OR.

```java
||
```

is logical OR.

---

## Mistake 3: Forgetting the Positive Check for Power of Two

Wrong:

```java
return (x & (x - 1)) == 0;
```

Correct:

```java
return x > 0 && (x & (x - 1)) == 0;
```

---

## Mistake 4: Using `1 << k` for a `long`

Wrong for large bit positions:

```java
long mask = 1 << k;
```

Use:

```java
long mask = 1L << k;
```

---

## Mistake 5: Ignoring Negative Numbers

Expressions such as:

```java
x >> k
```

behave differently for negative values because `>>` is sign-preserving.

Know whether the problem treats the number as:

- a mathematical integer,
- a signed Java integer,
- or a fixed-width bit pattern.

---

## Mistake 6: Thinking `x & (x - 1)` Checks Power of Two by Itself

It does not.

It **removes** the lowest set bit.

The power-of-two test is:

```java
x > 0 && (x & (x - 1)) == 0
```

---

## Mistake 7: Forgetting Operator Precedence

Prefer parentheses.

Instead of:

```java
if (x & 1 == 1)
```

write:

```java
if ((x & 1) == 1)
```

This is clearer and avoids precedence mistakes.

---

# 33. Problem-Solving Framework

When you see a bit-manipulation problem:

## Step 1: Ask What Is Being Represented?

Is the problem about:

- individual bits?
- a binary number?
- parity?
- duplicate cancellation?
- subsets?
- flags?
- compact state?

---

## Step 2: Identify the Required Bit Operation

Ask:

```text
Do I need to inspect a bit?
Set it?
Clear it?
Toggle it?
Count bits?
Remove a set bit?
Isolate a set bit?
Combine values using XOR?
```

---

## Step 3: Choose the Mask

For bit `k`:

```java
int mask = 1 << k;
```

Then:

```text
test   -> x & mask
set    -> x | mask
clear  -> x & ~mask
toggle -> x ^ mask
```

---

## Step 4: Look for Cancellation

If values repeat:

```text
XOR
```

may eliminate the duplicates.

---

## Step 5: Look for `x & (x - 1)`

Ask:

> Can removing one set bit solve the problem?

This is especially useful for:

- counting set bits,
- testing powers of two,
- reducing a number by its set-bit structure.

---

## Step 6: Check Integer Width

Decide whether the problem needs:

```java
int
```

or:

```java
long
```

and whether signed behavior matters.

---

## Step 7: Prove the Trick

Do not stop at "I remember this formula."

Explain why it works using binary representation.

This is important in interviews.

---

# 34. Complexity Patterns

Most primitive bit operations are:

```text
O(1) time
O(1) space
```

For a fixed-width integer, even scanning all bits is technically constant time.

However, interview explanations often use `w` for bit width:

| Technique | Time | Space |
|---|---:|---:|
| Check one bit | O(1) | O(1) |
| Set/clear/toggle one bit | O(1) | O(1) |
| Left/right shift | O(1) | O(1) |
| `x & (x - 1)` | O(1) | O(1) |
| Count bits by scanning | O(w) | O(1) |
| Kernighan count | O(k) | O(1) |
| XOR all array elements | O(n) | O(1) |
| XOR range using pattern | O(1) | O(1) |

Where:

- `w` = number of bits,
- `k` = number of set bits,
- `n` = number of array elements.

---

# 35. Easy → Medium → Hard Progression

## Easy

Master:

1. AND / OR / XOR / NOT
2. left/right shifts
3. test one bit
4. set one bit
5. clear one bit
6. toggle one bit
7. count set bits
8. power of two
9. XOR cancellation

## Medium

Then master:

1. Single Number
2. Missing Number
3. Counting Bits
4. Reverse Bits
5. two unique numbers
6. range XOR
7. bitwise range operations
8. XOR-based partitioning

## Hard / Advanced

Only after the basics are automatic:

1. XOR basis / linear basis
2. bitmask DP
3. subset enumeration
4. SOS DP
5. advanced state compression
6. bitwise trie
7. advanced range bitwise problems

Do not jump to bitmask DP before the basic identities are automatic.

---

# 36. Selected LeetCode Problems

The following are selected because they teach distinct patterns rather than repeating the same trick.

## 36.1 Single Number

- Platform: LeetCode
- Difficulty: Easy
- Topic: Bit Manipulation
- Pattern: XOR cancellation
- Priority: CORE
- Free: Yes

#### Official Link

https://leetcode.com/problems/single-number/

#### Why This Problem Matters

This is the canonical XOR-cancellation problem.

#### What You Should Notice

Every value appears twice except one.

#### Hint 1 — Observation

Equal values can cancel.

#### Hint 2 — Direction

Find an operation where:

```text
x op x = 0
```

#### Hint 3 — Pattern / Data Structure

Use XOR.

#### Hint 4 — Algorithm

Initialize `result = 0` and XOR every element.

#### Expected Approach

```text
result = 0
for every number:
    result ^= number
return result
```

#### Java Solution

```java
class Solution {
    public int singleNumber(int[] nums) {
        int result = 0;

        for (int num : nums) {
            result ^= num;
        }

        return result;
    }
}
```

#### Why It Works

Duplicates cancel:

```text
x ^ x = 0
```

and:

```text
0 ^ y = y
```

Therefore only the unique value remains.

#### Complexity

- Time: O(n)
- Space: O(1)

#### Common Mistakes

- using a frequency map unnecessarily,
- sorting when O(1) extra space is expected,
- forgetting that the cancellation property is specific to the required occurrence pattern.

#### Follow-Up Variations

- two unique values,
- values occurring three times,
- missing number.

#### Related Patterns

- XOR
- hashing
- frequency counting

#### Mastery Check

Can you derive the solution without memorizing it?

---

## 36.2 Number of 1 Bits

- Platform: LeetCode
- Difficulty: Easy
- Topic: Bit Manipulation
- Pattern: `n & (n - 1)`
- Priority: CORE
- Free: Yes

#### Official Link

https://leetcode.com/problems/number-of-1-bits/

#### Why This Problem Matters

It teaches the highest-value basic bit-counting identity.

#### What You Should Notice

Each application of:

```java
n &= n - 1;
```

removes exactly one set bit.

#### Hint 1 — Observation

You do not need to inspect every bit individually.

#### Hint 2 — Direction

Find an operation that removes a single `1`.

#### Hint 3 — Pattern / Data Structure

Use:

```java
n & (n - 1)
```

#### Hint 4 — Algorithm

Repeat until `n == 0`, incrementing the count after each removal.

#### Java Solution

```java
class Solution {
    public int hammingWeight(int n) {
        int count = 0;

        while (n != 0) {
            n &= (n - 1);
            count++;
        }

        return count;
    }
}
```

#### Why It Works

Each iteration removes exactly one set bit.

If the number contains `k` set bits, the loop executes exactly `k` times.

#### Complexity

- Time: O(k), where `k` is the number of set bits
- Space: O(1)

#### Common Mistakes

- assuming `n & (n - 1)` returns the lowest set bit; it removes it,
- forgetting that Java's `int` is fixed-width and signed.

#### Follow-Up Variations

- count set bits for every number from `0` to `n`,
- count bits in a `long`.

#### Related Patterns

- power of two
- lowest set bit

#### Mastery Check

Can you prove why `n & (n - 1)` removes the lowest set bit?

---

## 36.3 Power of Two

- Platform: LeetCode
- Difficulty: Easy
- Topic: Bit Manipulation
- Pattern: one-set-bit detection
- Priority: CORE
- Free: Yes

#### Official Link

https://leetcode.com/problems/power-of-two/

#### Why This Problem Matters

It combines binary representation with `n & (n - 1)`.

#### What You Should Notice

A positive power of two has exactly one set bit.

#### Hint 1 — Observation

Examples:

```text
1   -> 0001
2   -> 0010
4   -> 0100
8   -> 1000
```

#### Hint 2 — Direction

What happens when you remove the only set bit?

#### Hint 3 — Pattern / Data Structure

Use:

```java
n & (n - 1)
```

#### Hint 4 — Algorithm

Check:

```java
n > 0 && (n & (n - 1)) == 0
```

#### Java Solution

```java
class Solution {
    public boolean isPowerOfTwo(int n) {
        return n > 0 && (n & (n - 1)) == 0;
    }
}
```

#### Why It Works

A power of two contains exactly one set bit.

Removing it produces zero.

Any positive number with two or more set bits remains non-zero.

#### Complexity

- Time: O(1)
- Space: O(1)

#### Common Mistakes

- accepting zero,
- accepting negative values,
- checking only whether `n & (n - 1)` is zero without `n > 0`.

#### Follow-Up Variations

- power of four,
- power of two using logarithms,
- finding the nearest power of two.

#### Related Patterns

- set-bit counting
- masks

#### Mastery Check

Explain why the positive check is necessary.

---

## 36.4 Missing Number

- Platform: LeetCode
- Difficulty: Easy
- Topic: Bit Manipulation
- Pattern: XOR cancellation
- Priority: CORE
- Free: Yes

#### Official Link

https://leetcode.com/problems/missing-number/

#### Why This Problem Matters

It shows how XOR can replace arithmetic-sum approaches while avoiding overflow concerns.

#### What You Should Notice

The array contains numbers from `0` through `n`, with one missing.

#### Hint 1 — Observation

The complete set can be paired with the observed set.

#### Hint 2 — Direction

XOR equal values together.

#### Hint 3 — Pattern / Data Structure

XOR indices, values, and the final `n`.

#### Hint 4 — Algorithm

Start with `result = n`, then XOR:

```text
i
nums[i]
```

for every index.

#### Java Solution

```java
class Solution {
    public int missingNumber(int[] nums) {
        int result = nums.length;

        for (int i = 0; i < nums.length; i++) {
            result ^= i;
            result ^= nums[i];
        }

        return result;
    }
}
```

#### Why It Works

Every present value appears once from the expected range and once from the array, so they cancel.

Only the missing value remains.

#### Complexity

- Time: O(n)
- Space: O(1)

#### Common Mistakes

- starting with `0` but forgetting to include `n`,
- sorting unnecessarily,
- using a set when O(1) extra space is expected.

#### Follow-Up Variations

- more than one missing value,
- one duplicate and one missing value.

#### Related Patterns

- XOR cancellation
- cyclic placement
- arithmetic sum

#### Mastery Check

Derive the XOR expression without looking at the solution.

---

## 36.5 Counting Bits

- Platform: LeetCode
- Difficulty: Easy
- Topic: Bit Manipulation / Dynamic Programming
- Pattern: remove lowest set bit / recurrence
- Priority: IMPORTANT
- Free: Yes

#### Official Link

https://leetcode.com/problems/counting-bits/

#### Why This Problem Matters

It connects bit manipulation with dynamic programming.

#### What You Should Notice

For every `i`, the number of set bits can be derived from a smaller number.

Useful relations include:

```text
bits[i] = bits[i >> 1] + (i & 1)
```

or:

```text
bits[i] = bits[i & (i - 1)] + 1
```

#### Hint 1 — Observation

Even and odd numbers behave differently in their lowest bit.

#### Hint 2 — Direction

Remove or shift the lowest bit.

#### Hint 3 — Pattern / Data Structure

Use previously computed values.

#### Hint 4 — Algorithm

Use:

```java
bits[i] = bits[i >> 1] + (i & 1);
```

#### Java Solution

```java
class Solution {
    public int[] countBits(int n) {
        int[] bits = new int[n + 1];

        for (int i = 1; i <= n; i++) {
            bits[i] = bits[i >> 1] + (i & 1);
        }

        return bits;
    }
}
```

#### Why It Works

Right shifting removes the lowest bit.

The removed bit is exactly:

```java
i & 1
```

So:

```text
setBits(i)
=
setBits(i >> 1)
+
lowestBit(i)
```

#### Complexity

- Time: O(n)
- Space: O(n)

#### Common Mistakes

- writing an O(n log n) solution when O(n) is expected,
- confusing the problem with a single-number bit count.

#### Follow-Up Variations

- count bits only in a range,
- use the `i & (i - 1)` recurrence.

#### Related Patterns

- dynamic programming
- bit shifts

#### Mastery Check

Can you derive both recurrences?

---

## 36.6 Reverse Bits

- Platform: LeetCode
- Difficulty: Easy
- Topic: Bit Manipulation
- Pattern: extract and rebuild bits
- Priority: IMPORTANT
- Free: Yes

#### Official Link

https://leetcode.com/problems/reverse-bits/

#### Why This Problem Matters

It teaches systematic bit extraction and construction.

#### Hint 1 — Observation

Process one bit at a time.

#### Hint 2 — Direction

Extract the lowest bit using:

```java
n & 1
```

#### Hint 3 — Pattern / Data Structure

Shift the result left and the input right.

#### Hint 4 — Algorithm

Repeat exactly 32 times.

#### Java Solution

```java
class Solution {
    public int reverseBits(int n) {
        int result = 0;

        for (int i = 0; i < 32; i++) {
            result <<= 1;
            result |= (n & 1);
            n >>>= 1;
        }

        return result;
    }
}
```

#### Why It Works

Each iteration:

1. extracts the current lowest bit,
2. appends it to `result`,
3. shifts `n` right,
4. repeats for all 32 bits.

#### Complexity

- Time: O(32) = O(1)
- Space: O(1)

#### Common Mistakes

- using `>>` instead of `>>>` when treating the input as a 32-bit pattern,
- looping only until `n == 0`,
- forgetting that leading zeroes are still part of the 32-bit representation.

#### Follow-Up Variations

- reverse only the lowest `k` bits,
- reverse bytes,
- reverse a binary string.

#### Related Patterns

- bit extraction
- shifts
- masks

#### Mastery Check

Can you explain why the loop must run 32 times?

---

## 36.7 Single Number II

- Platform: LeetCode
- Difficulty: Medium
- Topic: Bit Manipulation
- Pattern: per-bit counting
- Priority: IMPORTANT
- Free: Yes

#### Official Link

https://leetcode.com/problems/single-number-ii/

#### Why This Problem Matters

It demonstrates how bit counts can generalize XOR reasoning when values appear three times.

#### What You Should Notice

If every number appears three times except one, ordinary XOR does not directly solve the problem because:

```text
x ^ x ^ x = x
```

#### Hint 1 — Observation

Look at each bit position independently.

#### Hint 2 — Direction

Count how many numbers have each bit set.

#### Hint 3 — Pattern / Data Structure

Use modulo 3.

#### Hint 4 — Algorithm

For every bit:

```text
count ones
count % 3
```

The remainder identifies whether the unique number has that bit set.

#### Java Solution

```java
class Solution {
    public int singleNumber(int[] nums) {
        int result = 0;

        for (int bit = 0; bit < 32; bit++) {
            int count = 0;

            for (int num : nums) {
                if ((num & (1 << bit)) != 0) {
                    count++;
                }
            }

            if (count % 3 != 0) {
                result |= (1 << bit);
            }
        }

        return result;
    }
}
```

#### Why It Works

Every repeated number contributes its bit exactly three times.

Therefore its contribution disappears modulo 3.

The unique number contributes once, leaving remainder 1.

#### Complexity

- Time: O(32n) = O(n)
- Space: O(1)

#### Common Mistakes

- trying ordinary XOR,
- using a frequency map when constant extra space is required,
- forgetting to handle the sign bit correctly.

#### Follow-Up Variations

- every value appears `k` times except one,
- design a constant-space finite-state solution.

#### Related Patterns

- bit counting
- XOR
- state machines

#### Mastery Check

Can you generalize the modulo-3 argument to modulo-`k`?

---

# 37. Fully Explained Representative Problems

## Problem A — Set, Clear, and Toggle a Bit

### Question

Given an integer `x` and bit position `k`, implement:

1. set bit `k`,
2. clear bit `k`,
3. toggle bit `k`.

### Solution

Create:

```java
int mask = 1 << k;
```

Then:

```java
set    -> x | mask
clear  -> x & ~mask
toggle -> x ^ mask
```

### Complete Java

```java
static int setBit(int x, int k) {
    return x | (1 << k);
}

static int clearBit(int x, int k) {
    return x & ~(1 << k);
}

static int toggleBit(int x, int k) {
    return x ^ (1 << k);
}
```

### Example

```text
x = 10 = 1010
k = 1
```

Set:

```text
1010
0010
----
1010
```

It was already set.

Clear:

```text
1010
1101
----
1000
```

Toggle:

```text
1010
0010
----
1000
```

### Key Lesson

The same mask is combined with a different operator depending on the required operation.

---

## Problem B — Count Set Bits

### Question

Count the number of `1` bits in an integer.

### Brute-force approach

Inspect each bit:

```java
while (x != 0) {
    count += x & 1;
    x >>>= 1;
}
```

This works.

### Better bit trick

Use:

```java
x &= x - 1;
```

Every iteration removes one set bit.

### Complete Java

```java
static int countSetBits(int x) {
    int count = 0;

    while (x != 0) {
        x &= (x - 1);
        count++;
    }

    return count;
}
```

### Example

```text
x = 10110100
```

First:

```text
10110100
10110011
--------
10110000
```

Second:

```text
10110000
10101111
--------
10100000
```

Continue until zero.

There are four set bits, so the answer is `4`.

### Key Lesson

Whenever a problem asks you to repeatedly remove set bits, think:

```java
x &= x - 1;
```

---

## Problem C — Find the Unique Number

### Question

Every element appears exactly twice except one. Find the unique element in O(n) time and O(1) extra space.

### Example

```text
[4, 1, 2, 1, 2]
```

### Reasoning

A hash map works, but uses O(n) space.

Sorting works, but changes the time complexity.

XOR gives exactly the required behavior:

```text
x ^ x = 0
x ^ 0 = x
```

Therefore:

```text
4 ^ 1 ^ 2 ^ 1 ^ 2
```

can be rearranged to:

```text
4 ^ (1 ^ 1) ^ (2 ^ 2)
```

which becomes:

```text
4 ^ 0 ^ 0
= 4
```

### Java

```java
static int findUnique(int[] nums) {
    int result = 0;

    for (int num : nums) {
        result ^= num;
    }

    return result;
}
```

### Complexity

```text
Time: O(n)
Space: O(1)
```

### Key Lesson

The important skill is not memorizing "Single Number = XOR."

The transferable reasoning is:

> If repeated values can be paired and canceled by an associative operation, look for that operation.

---

# 38. PYQs

The directly supplied topic is coding-oriented bit manipulation. Reliable GATE sources contain several closely related XOR/Boolean-algebra PYQs rather than three exact Java-style bit-manipulation questions. The following are therefore selected as **verified XOR-focused PYQs**, not falsely presented as direct coding-bit questions.

Official GATE paper repositories are available through the GATE organizing institutes, and GATE Overflow preserves the question text and solutions. 

---

## PYQ 1 — GATE CSE 1998, Question 1.13

### Question

What happens when a bit-string is XORed with itself `n` times in the form:

```text
B ⊕ (B ⊕ (B ⊕ ... n times))
```

Options include:

1. complements when `n` is even
2. complements when `n` is odd
3. divides by `2^n` always
4. remains unchanged when `n` is even

### What Is Being Tested

- XOR cancellation
- XOR identity
- parity of repeated XOR operations

### Approach

Use:

```text
B ⊕ B = 0
B ⊕ 0 = B
```

The wording "XORed n-times" matters.

If there are an even number of XOR operations, the expression contains an odd number of copies of `B`, leaving `B`.

If there are an odd number of XOR operations, the expression contains an even number of copies of `B`, leaving `0`.

### Step-by-Step Solution

For two XOR operations:

```text
B ⊕ B ⊕ B
```

Group the first two:

```text
(B ⊕ B) ⊕ B
= 0 ⊕ B
= B
```

So the result is unchanged.

For three XOR operations:

```text
B ⊕ B ⊕ B ⊕ B
```

Group pairs:

```text
(B ⊕ B) ⊕ (B ⊕ B)
= 0 ⊕ 0
= 0
```

Therefore:

```text
even number of XOR operations -> B
odd number of XOR operations  -> 0
```

### Final Answer

**Option 4 — remains unchanged when `n` is even.**

### Why This Works

XOR is associative and:

```text
B ^ B = 0
```

so copies cancel in pairs.

### Common Trap

Do not confuse:

```text
n = number of XOR operations
```

with:

```text
n = number of copies of B
```

### Generalizable Lesson

Whenever a sequence contains repeated XORs, count occurrences and cancel pairs.

Source: GATE Overflow, GATE CSE 1998 Q1.13.

---

## PYQ 2 — GATE CSE 2016 Set 2, Question 8

### Question

Let:

```text
x1 ⊕ x2 ⊕ x3 ⊕ x4 = 0
```

where `x1, x2, x3, x4` are Boolean variables.

Which statement must always be true?

1. `x1x2x3x4 = 0`
2. `x1x3 + x2 = 0`
3. `x1' ⊕ x3' = x2' ⊕ x4'`
4. `x1 + x2 + x3 + x4 = 0`

### What Is Being Tested

- XOR parity
- Boolean XOR identities
- reasoning from an XOR constraint

### Approach

From:

```text
x1 ⊕ x2 ⊕ x3 ⊕ x4 = 0
```

the number of ones among the four variables must be even.

Now simplify option 3.

For Boolean variables:

```text
x' ⊕ y' = x ⊕ y
```

Therefore:

```text
x1' ⊕ x3'
= x1 ⊕ x3
```

and:

```text
x2' ⊕ x4'
= x2 ⊕ x4
```

From the original equation:

```text
x1 ⊕ x2 ⊕ x3 ⊕ x4 = 0
```

rearrange:

```text
x1 ⊕ x3 = x2 ⊕ x4
```

Therefore option 3 is always true.

### Final Answer

**Option 3.**

### Why This Works

XOR expresses parity. A four-variable XOR equal to zero means the parity of the first pair equals the parity of the second pair.

### Common Trap

Do not interpret Boolean `+` as ordinary arithmetic addition.

In Boolean algebra:

```text
1 + 1 = 1
```

whereas XOR gives:

```text
1 ⊕ 1 = 0
```

### Generalizable Lesson

When an XOR expression equals zero:

```text
a ^ b ^ c ^ d = 0
```

you can rearrange it to obtain relationships such as:

```text
a ^ b = c ^ d
```

Source: GATE Overflow, GATE CSE 2016 Set 2 Q8.

---

## PYQ 3 — GATE CSE 2019, Question 6

### Question

Which of the following is **NOT** a valid identity?

1. `(x ⊕ y) ⊕ z = x ⊕ (y ⊕ z)`
2. `(x + y) ⊕ z = x ⊕ (y + z)`
3. `x ⊕ y = x + y`, if `xy = 0`
4. `x ⊕ y = (xy + x'y')'`

### What Is Being Tested

- XOR associativity
- XOR versus OR
- XOR truth table
- XOR/XNOR relationship

### Approach

Check each identity.

### Option 1

XOR is associative:

```text
(x ^ y) ^ z = x ^ (y ^ z)
```

Valid.

### Option 2

In general:

```text
(x + y) ^ z
```

is not equal to:

```text
x ^ (y + z)
```

A counterexample is:

```text
x = 1
y = 0
z = 1
```

Left side:

```text
(1 + 0) ^ 1
= 1 ^ 1
= 0
```

Right side:

```text
1 ^ (0 + 1)
= 1 ^ 1
= 0
```

This example happens to match, so choose a better counterexample:

```text
x = 1
y = 1
z = 0
```

Left:

```text
(1 + 1) ^ 0
= 1 ^ 0
= 1
```

Right:

```text
1 ^ (1 + 0)
= 1 ^ 1
= 0
```

Therefore option 2 is invalid.

### Option 3

If:

```text
xy = 0
```

then both variables cannot be 1.

So XOR and OR agree:

```text
x ^ y = x + y
```

Valid under the condition.

### Option 4

XOR is the complement of XNOR:

```text
x ^ y = (xy + x'y')'
```

Valid.

### Final Answer

**Option 2.**

### Why This Works

The key distinction is:

```text
XOR = OR when the inputs cannot both be 1.
```

Without that condition, OR and XOR differ at:

```text
1, 1
```

### Common Trap

Do not assume Boolean operators follow ordinary arithmetic identities.

### Generalizable Lesson

Know the XOR truth table and the identities:

```text
x ^ x = 0
x ^ 0 = x
x ^ y = y ^ x
(x ^ y) ^ z = x ^ (y ^ z)
```

Source: GATE Overflow, GATE CSE 2019 Q6.

---

# 39. Pattern Recognition Checklist

When you see a problem, ask:

### Basic Bit Operations

- Do I need a specific bit?
- Is the bit position given?
- Can I create a mask using `1 << k`?

### Set / Clear / Toggle

```text
set    -> OR
clear  -> AND with inverted mask
toggle -> XOR
```

### Counting

- Does the problem ask for number of `1`s?
- Can I use `x & (x - 1)`?

### Power of Two

- Does the number need exactly one set bit?

Think:

```java
x > 0 && (x & (x - 1)) == 0
```

### XOR

- Do elements occur in pairs?
- Is one element unique?
- Is one number missing?
- Are there exactly two unique elements?

Think:

```text
XOR cancellation
```

### Lowest Set Bit

If the problem talks about the lowest/rightmost `1`:

```java
x & -x
```

### Removing Set Bits

```java
x & (x - 1)
```

### Bitmask State

If a problem has a small fixed number of boolean choices:

```text
bitmask
```

may be the right representation.

---

# 40. Mastery Checklist

Do not consider this topic mastered until you can do all of the following without looking up formulas.

## Fundamentals

- [ ] Explain binary representation.
- [ ] Explain bit positions.
- [ ] Explain AND.
- [ ] Explain OR.
- [ ] Explain XOR.
- [ ] Explain NOT.
- [ ] Explain left shift.
- [ ] Explain signed right shift.
- [ ] Explain unsigned right shift.

## Single-Bit Operations

- [ ] Check bit `k`.
- [ ] Set bit `k`.
- [ ] Clear bit `k`.
- [ ] Toggle bit `k`.

You should immediately know:

```java
(x & (1 << k)) != 0
x | (1 << k)
x & ~(1 << k)
x ^ (1 << k)
```

## High-Value Tricks

- [ ] Explain `x & (x - 1)`.
- [ ] Explain `x & -x`.
- [ ] Check a power of two.
- [ ] Count set bits.
- [ ] Explain XOR cancellation.
- [ ] Find one unique number.
- [ ] Find a missing number using XOR.
- [ ] Explain why XOR is associative and commutative.
- [ ] Explain the XOR-from-1-to-N modulo-4 pattern.

## Java

- [ ] Know `int` is 32-bit.
- [ ] Know `long` is 64-bit.
- [ ] Know when to use `1L << k`.
- [ ] Understand `>>` versus `>>>`.
- [ ] Understand negative integers at the bit level.
- [ ] Know the basic `Integer` bit utilities.

## Interview Skill

- [ ] Recognize bit manipulation from problem wording.
- [ ] Derive the trick instead of blindly memorizing it.
- [ ] Explain the binary reasoning aloud.
- [ ] State time and space complexity.
- [ ] Handle edge cases.
- [ ] Know when a normal arithmetic or data-structure solution is clearer.

---

# 41. What to Study Next

After these basics, the natural progression is:

1. **Advanced XOR patterns**
2. **Bitmasking**
3. **Subset generation using bitmasks**
4. **Bitmask DP**
5. **Bitwise AND/OR range problems**
6. **Trie-based bitwise problems**
7. **Advanced number-theoretic bit tricks**

For interview preparation, do not rush into bitmask DP until the basic operations are automatic.

The next high-value topic is **Bitmasking and subset enumeration**, because it builds directly on:

```java
1 << k
x & mask
x | mask
x ^ mask
```

---

# 42. One-Page Cheat Sheet

## Operators

```text
&   AND
|   OR
^   XOR
~   NOT
<<  left shift
>>  signed right shift
>>> unsigned right shift
```

## Single Bit

```java
int mask = 1 << k;

check  -> (x & mask) != 0
set    -> x | mask
clear  -> x & ~mask
toggle -> x ^ mask
```

## Set-Bit Tricks

```java
x & (x - 1)
```

Removes the lowest set bit.

```java
x & -x
```

Isolates the lowest set bit.

```java
x > 0 && (x & (x - 1)) == 0
```

Checks whether `x` is a power of two.

## XOR

```text
x ^ 0 = x
x ^ x = 0
x ^ y = y ^ x
(x ^ y) ^ z = x ^ (y ^ z)
```

Use XOR when equal values can cancel.

## Shifts

```java
x << k
```

roughly multiplies positive `x` by `2^k`.

```java
x >> k
```

signed right shift.

```java
x >>> k
```

zero-fill right shift.

## Bit Counting

```java
while (x != 0) {
    x &= x - 1;
    count++;
}
```

## Java Width

```text
int  -> 32 bits
long -> 64 bits
```

Use:

```java
1L << k
```

for long masks.

---

# 43. Final Interview Mental Model

If you remember only one decision tree, use this:

```text
Need one bit?
    |
    +-- inspect -> AND
    +-- set     -> OR
    +-- clear   -> AND + NOT
    +-- toggle  -> XOR

Need number of set bits?
    |
    +-- x & (x - 1)

Need lowest set bit?
    |
    +-- x & -x

Need power of two?
    |
    +-- x > 0 && (x & (x - 1)) == 0

Duplicates cancel?
    |
    +-- XOR

Need a compact set of boolean states?
    |
    +-- bitmask
```

The important skill is pattern recognition.

Do not memorize bit manipulation as a bag of unrelated tricks. Understand the binary invariant behind each operation. Once that is clear, most of the formulas become derivable.
