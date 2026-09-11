# 20. Bit Manipulation

> **Goal:** Master bit manipulation for FAANG/top product-company SDE and ML/AI engineer interviews, with the theory, Java implementation patterns, GATE-oriented fundamentals, advanced XOR techniques, and problem-recognition skills needed for competitive programming and interviews.
>
> **Language:** Java  
> **Difficulty target:** Easy → Medium → Hard, with Medium dominating.
>
> **Core idea:** Bit manipulation is about representing and transforming information using the binary representation of integers. Most interview problems reduce to a small set of operations involving `&`, `|`, `^`, `~`, `<<`, and `>>`.

---

# 1. Binary Representation

An integer is represented using bits:

```text
bit positions:
...  5 4 3 2 1 0
     32 16 8 4 2 1
```

For example:

```text
13 = 1101₂
```

because:

```text
13 = 8 + 4 + 1
```

Binary representation is the foundation of bit manipulation.

---

# 2. Java Bitwise Operators

| Operator | Name | Meaning |
|---|---|---|
| `&` | AND | bit is 1 if both bits are 1 |
| `|` | OR | bit is 1 if either bit is 1 |
| `^` | XOR | bit is 1 if bits differ |
| `~` | NOT | flips every bit |
| `<<` | left shift | shifts bits left |
| `>>` | arithmetic right shift | shifts right, preserving sign |
| `>>>` | logical right shift | shifts right, filling with zero |

---

# 3. Truth Tables

## AND

```text
A B | A&B
0 0 |  0
0 1 |  0
1 0 |  0
1 1 |  1
```

## OR

```text
A B | A|B
0 0 |  0
0 1 |  1
1 0 |  1
1 1 |  1
```

## XOR

```text
A B | A^B
0 0 |  0
0 1 |  1
1 0 |  1
1 1 |  0
```

XOR is especially important in interview problems.

---

# 4. Bitwise vs Logical Operators

Do not confuse:

```java
&
|
^
```

with:

```java
&&
||
!
```

Bitwise operators work on individual bits.

Logical operators work with boolean conditions.

Example:

```java
int x = 5 & 3;
```

whereas:

```java
boolean b = true && false;
```

---

# 5. XOR

The fundamental XOR properties are:

```text
x ^ 0 = x
x ^ x = 0
x ^ y = y ^ x
(x ^ y) ^ z = x ^ (y ^ z)
```

Therefore XOR is:

```text
commutative
associative
self-canceling
```

---

# 6. XOR Cancellation

The most important cancellation rule:

```text
x ^ x = 0
```

and:

```text
x ^ 0 = x
```

Therefore:

```text
a ^ b ^ a
=
(a ^ a) ^ b
=
b
```

This allows pairs to cancel automatically.

---

# 7. Single Number Pattern

If every number occurs twice except one number, XOR all numbers.

Example:

```text
[4, 1, 2, 1, 2]
```

Compute:

```text
4 ^ 1 ^ 2 ^ 1 ^ 2
```

Pairs cancel:

```text
1 ^ 1 = 0
2 ^ 2 = 0
```

so:

```text
answer = 4
```

---

## Java

```java
public class SingleNumber {
    static int singleNumber(int[] nums) {
        int answer = 0;

        for (int x : nums) {
            answer ^= x;
        }

        return answer;
    }
}
```

Complexity:

```text
Time: O(n)
Space: O(1)
```

---

# 8. XOR and Duplicate Cancellation

XOR can solve many problems where:

```text
every element appears an even number of times
except one
```

If an element appears:

```text
2 times → cancels
4 times → cancels
6 times → cancels
```

because:

```text
x ^ x = 0
```

and repeated XOR is associative.

---

# 9. Missing Number

Suppose an array contains all numbers from:

```text
0..n
```

except one.

Instead of computing a potentially overflowing arithmetic sum, XOR:

```text
0 ^ 1 ^ 2 ^ ... ^ n
```

with all array values.

Everything present cancels.

The missing number remains.

---

## Java

```java
public class MissingNumber {
    static int missingNumber(int[] nums) {
        int n = nums.length;
        int answer = n;

        for (int i = 0; i < n; i++) {
            answer ^= i;
            answer ^= nums[i];
        }

        return answer;
    }
}
```

Complexity:

```text
O(n) time
O(1) space
```

---

# 10. XOR of a Range

To compute XOR from:

```text
1..n
```

there is a four-case pattern based on:

```text
n % 4
```

```text
n % 4 == 0 → n
n % 4 == 1 → 1
n % 4 == 2 → n + 1
n % 4 == 3 → 0
```

Example:

```text
1 ^ 2 ^ 3 = 0
```

because:

```text
3 % 4 = 3
```

---

## Why the Pattern Repeats

Observe:

```text
1 ^ 2 ^ 3 ^ 4 = 4
1 ^ 2 ^ 3 ^ 4 ^ 5 = 1
1 ^ 2 ^ 3 ^ 4 ^ 5 ^ 6 = 7
1 ^ ... ^ 7 = 0
```

Then the pattern repeats modulo 4.

---

# 11. XOR Range [L, R]

Use:

```text
XOR(1..R)
^
XOR(1..L-1)
```

because values below `L` cancel.

```java
static int xorRange(int left, int right) {
    return xorTo(right) ^ xorTo(left - 1);
}

static int xorTo(int n) {
    switch (n & 3) {
        case 0: return n;
        case 1: return 1;
        case 2: return n + 1;
        default: return 0;
    }
}
```

Complexity:

```text
O(1)
```

---

# 12. Bit Masks

A bit mask is an integer whose bits encode a set of boolean conditions or selected elements.

Example:

```text
mask = 01010110
```

means certain positions are enabled.

Bit masks are used for:

- subsets
- permissions
- feature flags
- visited states
- DP states
- compact boolean storage

---

# 13. Set a Bit

To set bit `i` to `1`:

```java
x = x | (1 << i);
```

Example:

```text
x = 1000
i = 1
```

Mask:

```text
0010
```

Result:

```text
1010
```

The bit becomes 1 regardless of its previous value.

---

# 14. Clear a Bit

To clear bit `i`:

```java
x = x & ~(1 << i);
```

The target bit becomes:

```text
0
```

regardless of its previous value.

---

# 15. Toggle a Bit

To toggle bit `i`:

```java
x = x ^ (1 << i);
```

Behavior:

```text
0 → 1
1 → 0
```

---

# 16. Check a Bit

To determine whether bit `i` is set:

```java
boolean set =
    (x & (1 << i)) != 0;
```

This is one of the most frequently used bit operations.

---

# 17. Get a Bit

A reusable method:

```java
static int getBit(int x, int i) {
    return (x >>> i) & 1;
}
```

Use `>>>` when you want logical shifting.

---

# 18. Set/Clear/Toggle/Get Template

```java
static int setBit(int x, int i) {
    return x | (1 << i);
}

static int clearBit(int x, int i) {
    return x & ~(1 << i);
}

static int toggleBit(int x, int i) {
    return x ^ (1 << i);
}

static boolean isSet(int x, int i) {
    return (x & (1 << i)) != 0;
}

static int getBit(int x, int i) {
    return (x >>> i) & 1;
}
```

---

# 19. Lowest Set Bit

For a nonzero integer `x`:

```text
x & -x
```

isolates the lowest set bit.

Example:

```text
x = 1011000
```

then:

```text
x & -x = 0001000
```

assuming fixed-width two's-complement representation.

---

# 20. Why `x & -x` Works

In two's complement:

```text
-x = ~x + 1
```

All bits below the lowest set bit are zero in `x`, while the corresponding structure of `-x` leaves that lowest set bit isolated when ANDed.

This is useful for:

- removing the lowest set bit
- Fenwick trees
- subset enumeration
- bitmask algorithms
- counting/grouping by lowest differing bit

---

# 21. Remove Lowest Set Bit

Use:

```java
x = x & (x - 1);
```

Example:

```text
x     = 1011000
x - 1 = 1010111

AND   = 1010000
```

Exactly one set bit disappears: the lowest set bit.

This is the basis of Brian Kernighan's algorithm.

---

# 22. Power of 2

A positive integer is a power of two if and only if it contains exactly one set bit.

Therefore:

```java
x > 0 && (x & (x - 1)) == 0
```

---

## Java

```java
static boolean isPowerOfTwo(int x) {
    return x > 0 && (x & (x - 1)) == 0;
}
```

Examples:

```text
1   → true
2   → true
4   → true
8   → true
16  → true
6   → false
12  → false
```

---

# 23. Power of 4

A positive power of four is also a power of two, but its only set bit must be at an even bit position.

A common 32-bit Java test:

```java
static boolean isPowerOfFour(int n) {
    return n > 0
        && (n & (n - 1)) == 0
        && (n & 0x55555555) != 0;
}
```

Why:

```text
0x55555555
```

has 1s at even bit positions:

```text
...01010101
```

---

# 24. Count Set Bits

The number of `1` bits in an integer is its:

```text
population count
```

or:

```text
popcount
```

In Java:

```java
Integer.bitCount(x)
```

---

# 25. Brian Kernighan's Algorithm

Instead of checking every bit:

```text
x = x & (x - 1)
```

removes one set bit.

Therefore repeat until zero.

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

Complexity:

```text
O(k)
```

where `k` is the number of set bits.

For a fixed-width integer this is at most:

```text
O(32)
```

for Java `int`.

---

# 26. Count Set Bits — Java Built-in

For `int`:

```java
Integer.bitCount(x)
```

For `long`:

```java
Long.bitCount(x)
```

For interviews, know both:

```text
library method
```

and:

```text
Brian Kernighan
```

---

# 27. Odd/Even Using Bits

The least significant bit determines parity.

```text
x & 1
```

If:

```text
(x & 1) == 0
```

then `x` is even.

If:

```text
(x & 1) == 1
```

then `x` is odd.

---

# 28. Multiplication/Division by Powers of 2

For non-overflowing positive integer arithmetic:

```text
x << k
```

corresponds to:

```text
x × 2^k
```

and:

```text
x >> k
```

corresponds roughly to:

```text
floor(x / 2^k)
```

for nonnegative integers.

Do not use these blindly when overflow, negative values, or language-specific signed behavior matters.

---

# 29. Left Shift

```java
x << k
```

moves bits left by `k`.

Conceptually:

```text
001011
<< 2

101100
```

Bits shifted beyond the integer width are discarded.

For Java `int`, only the low five bits of the shift distance are used.

Thus:

```java
x << 32
```

behaves like:

```java
x << 0
```

This is an important Java-specific detail.

---

# 30. Right Shift

Java has two right-shift operators.

## Arithmetic right shift

```java
x >> k
```

replicates the sign bit.

## Logical right shift

```java
x >>> k
```

fills with zeros.

For nonnegative integers they produce the same value.

For negative integers they can differ substantially.

---

# 31. Java Integer Width

Java `int`:

```text
32 bits
```

Java `long`:

```text
64 bits
```

Java uses signed two's-complement representation for integer types.

Important constants:

```java
Integer.SIZE   // 32
Long.SIZE      // 64
```

---

# 32. NOT Operator

The bitwise complement:

```java
~x
```

flips every bit.

A key identity:

```text
~x = -x - 1
```

for signed two's-complement integers.

Therefore:

```text
~5 = -6
```

because:

```text
-5 - 1 = -6
```

---

# 33. XOR Swap

The classic XOR swap:

```java
a ^= b;
b ^= a;
a ^= b;
```

works under suitable conditions.

However, it is generally inferior to:

```java
int temp = a;
a = b;
b = temp;
```

for normal Java code.

### Interview relevance

Know the technique and its algebraic basis, but do not use it unnecessarily.

Potential aliasing concerns make the XOR version inappropriate in many real-world situations.

---

# 34. XOR for Two Unique Values

Suppose every number occurs twice except two numbers that occur once.

Example:

```text
[1, 2, 1, 3, 2, 5]
```

Unique values:

```text
3 and 5
```

XOR everything:

```text
xor = 3 ^ 5
```

This does not directly separate them.

Use a set bit where they differ.

---

# 35. Finding the Differing Bit

Let:

```text
xor = a ^ b
```

Since `a` and `b` differ, `xor` has at least one set bit.

Isolate one:

```text
diffBit = xor & -xor;
```

Now divide numbers into two groups:

```text
group 0 → diffBit is 0
group 1 → diffBit is 1
```

Each unique value goes to a different group.

Duplicate values still cancel inside their group.

---

## Java

```java
public class TwoUniqueNumbers {
    static int[] singleNumber(int[] nums) {
        int xor = 0;

        for (int x : nums) {
            xor ^= x;
        }

        int diffBit = xor & -xor;

        int a = 0;
        int b = 0;

        for (int x : nums) {
            if ((x & diffBit) == 0) {
                a ^= x;
            } else {
                b ^= x;
            }
        }

        return new int[]{a, b};
    }
}
```

Complexity:

```text
O(n) time
O(1) auxiliary space
```

---

# 36. XOR and Missing/Repeated Values

Many array problems can be transformed into:

```text
expected values XOR actual values
```

If matching values occur equally often, they cancel.

Typical clues:

```text
all numbers from a known range
one missing
one duplicate
pairs except one
two unique elements
```

---

# 37. Subsets Using Bit Masks

For `n` elements there are:

```text
2^n
```

subsets.

Each subset can be represented by an `n`-bit mask.

Example with:

```text
[10, 20, 30]
```

Mask:

```text
101
```

means:

```text
take 10
skip 20
take 30
```

---

# 38. Enumerating All Subsets

```java
import java.util.*;

public class BitmaskSubsets {
    static List<List<Integer>> subsets(int[] nums) {
        int n = nums.length;
        List<List<Integer>> result = new ArrayList<>();

        for (int mask = 0;
             mask < (1 << n);
             mask++) {

            List<Integer> current =
                new ArrayList<>();

            for (int i = 0; i < n; i++) {
                if ((mask & (1 << i)) != 0) {
                    current.add(nums[i]);
                }
            }

            result.add(current);
        }

        return result;
    }
}
```

Complexity:

```text
O(n × 2^n)
```

because there are `2^n` masks and up to `n` bits to inspect per mask.

---

# 39. Enumerating Submasks

For a mask `mask`, all non-empty submasks can be enumerated with:

```java
for (int sub = mask; sub > 0; sub = (sub - 1) & mask) {
    // sub is a submask of mask
}
```

This is an important advanced bitmask technique.

---

# 40. Why `(sub - 1) & mask` Works

Starting from:

```text
sub = mask
```

subtracting one:

```text
sub - 1
```

turns the lowest set bit into zero and changes lower bits.

ANDing with:

```text
mask
```

removes any bits not present in the original mask.

Thus the next smaller submask is produced.

---

# 41. Total Submask Enumeration Complexity

Across all masks, the total number of `(mask, submask)` pairs is:

```text
3^n
```

This is a classic result.

Reason:

For each bit there are three possibilities:

```text
not in mask
in mask but not submask
in both mask and submask
```

Therefore:

```text
3 × 3 × ... × 3 = 3^n
```

---

# 42. Bitwise Arithmetic

Bit operations can implement arithmetic concepts.

---

# 43. Add Without `+`

XOR computes addition without carry:

```text
a ^ b
```

AND identifies carry positions:

```text
a & b
```

Shift carries:

```text
(a & b) << 1
```

Therefore:

```text
sumWithoutCarry = a ^ b
carry = (a & b) << 1
```

Repeat until:

```text
carry == 0
```

---

## Java

```java
public class BitwiseAddition {
    static int add(int a, int b) {
        while (b != 0) {
            int carry = (a & b) << 1;
            a = a ^ b;
            b = carry;
        }

        return a;
    }
}
```

Complexity:

```text
O(number of bits)
```

for fixed-width integers.

---

# 44. Subtraction Without `-`

Use two's complement:

```text
a - b = a + (~b + 1)
```

So subtraction can be built from:

```text
addition
+
bitwise complement
```

The exact implementation must account for signed overflow behavior if the full Java `int` domain is allowed.

---

# 45. XOR Basis

An XOR basis is a linear-algebra-like structure over the field:

```text
GF(2)
```

where:

```text
addition = XOR
```

Instead of ordinary linear combinations:

```text
a*x + b*y
```

where coefficients are real/integer values, XOR basis uses coefficients:

```text
0 or 1
```

and combines vectors using XOR.

---

# 46. XOR Basis — Core Idea

Given integers:

```text
x1, x2, ..., xn
```

we want a compact basis from which every XOR combination of the input values can be represented.

A basis stores vectors with distinct highest set-bit positions, similar in spirit to Gaussian elimination.

---

# 47. XOR Basis Insertion

Process bits from high to low.

For a number `x`:

1. Find its highest set bit.
2. If the basis already has a vector with that highest bit, XOR it away.
3. Otherwise store `x` as the basis vector for that bit.

---

## Java

```java
public class XorBasis {
    private final int[] basis = new int[32];

    void insert(int x) {
        for (int bit = 31; bit >= 0; bit--) {

            if ((x & (1 << bit)) == 0) {
                continue;
            }

            if (basis[bit] == 0) {
                basis[bit] = x;
                return;
            }

            x ^= basis[bit];
        }
    }

    boolean canRepresent(int x) {
        for (int bit = 31; bit >= 0; bit--) {

            if ((x & (1 << bit)) == 0) {
                continue;
            }

            if (basis[bit] == 0) {
                return false;
            }

            x ^= basis[bit];
        }

        return true;
    }
}
```

---

# 48. XOR Basis Applications

XOR basis can solve advanced problems involving:

- maximum subset XOR
- whether a target XOR is representable
- XOR linear independence
- maximum XOR under subset selection
- range/query XOR problems
- competitive-programming linear basis problems

The key conceptual connection is:

```text
XOR basis ≈ Gaussian elimination over GF(2)
```

---

# 49. Maximum Subset XOR

Once a basis is built, maximize XOR greedily from the highest bit.

Start:

```text
answer = 0
```

For each basis vector from high bit to low:

```text
candidate = answer ^ basis[bit]
```

If:

```text
candidate > answer
```

take it.

---

## Java

```java
static int maxSubsetXor(int[] nums) {
    int[] basis = new int[32];

    for (int x : nums) {
        for (int bit = 31; bit >= 0; bit--) {

            if ((x & (1 << bit)) == 0) {
                continue;
            }

            if (basis[bit] == 0) {
                basis[bit] = x;
                break;
            }

            x ^= basis[bit];
        }
    }

    int answer = 0;

    for (int bit = 31; bit >= 0; bit--) {
        answer = Math.max(
            answer,
            answer ^ basis[bit]
        );
    }

    return answer;
}
```

---

# 50. XOR Basis Rank

The number of nonzero basis vectors is the XOR rank.

If:

```text
rank = r
```

then there are:

```text
2^r
```

distinct XOR combinations generated by the basis.

This assumes combinations are formed using each basis vector with coefficient:

```text
0 or 1
```

---

# 51. XOR Basis and Linear Dependence

A new number is linearly dependent on the current basis if, after elimination, it becomes:

```text
0
```

That means it can be represented as the XOR of existing basis vectors.

This is the XOR analogue of Gaussian elimination.

---

# 52. Signed Integers and Highest Bit

For advanced XOR basis code, signed Java `int` requires care because bit 31 is the sign bit.

For non-negative values, many implementations use:

```text
bits 30..0
```

If the full signed integer domain matters, define the comparison/order semantics explicitly.

For `long`, use:

```java
1L << bit
```

rather than:

```java
1 << bit
```

---

# 53. Bit Masks for Feature Sets

Suppose there are at most 26 boolean properties.

Represent them as:

```text
bits 0..25
```

Then:

```java
mask |= 1 << feature;
```

sets a feature.

Check:

```java
(mask & (1 << feature)) != 0
```

This can replace a boolean array when the number of features is small.

---

# 54. Power Set Representation

A subset of:

```text
{0,1,2,...,n-1}
```

can be represented as:

```text
mask
```

where:

```text
bit i = 1
```

means:

```text
element i is selected
```

This representation is fundamental to:

```text
bitmask DP
```

---

# 55. Common Bitmask DP Operations

### Add item

```java
mask | (1 << i)
```

### Remove item

```java
mask & ~(1 << i)
```

### Check item

```java
(mask & (1 << i)) != 0
```

### Toggle item

```java
mask ^ (1 << i)
```

### Count selected items

```java
Integer.bitCount(mask)
```

---

# 56. Bit Tricks for Powers

### Is power of 2

```java
x > 0 && (x & (x - 1)) == 0
```

### Remove lowest set bit

```java
x & (x - 1)
```

### Isolate lowest set bit

```java
x & -x
```

### Count set bits

```java
Integer.bitCount(x)
```

### Check odd

```java
(x & 1) != 0
```

---

# 57. Reverse Bits

A common advanced problem is reversing the 32 bits of an integer.

Idea:

```text
answer = 0
```

For each bit:

```text
answer <<= 1
answer |= x & 1
x >>>= 1
```

Use logical shift when processing all 32 bits.

---

## Java

```java
static int reverseBits(int x) {
    int answer = 0;

    for (int i = 0; i < 32; i++) {
        answer <<= 1;
        answer |= (x & 1);
        x >>>= 1;
    }

    return answer;
}
```

Complexity:

```text
O(32) = O(1)
```

for Java `int`.

---

# 58. Find the Lowest Set Bit Position

If:

```text
x != 0
```

then the index of the lowest set bit can be found using:

```java
Integer.numberOfTrailingZeros(x)
```

Example:

```text
x = 40 = 101000
```

lowest set bit is position:

```text
3
```

because:

```text
40 = 32 + 8
```

---

# 59. Find Highest Set Bit Position

For positive `x`:

```java
31 - Integer.numberOfLeadingZeros(x)
```

returns the index of the highest set bit.

For `long`:

```java
63 - Long.numberOfLeadingZeros(x)
```

---

# 60. Isolate All Set Bits

To iterate through set bits:

```java
while (x != 0) {
    int bit = x & -x;

    // process bit

    x -= bit;
}
```

or:

```java
while (x != 0) {
    int bit = x & -x;

    // process bit

    x &= x - 1;
}
```

The second version removes the lowest set bit.

---

# 61. Gray Code Connection

Gray code changes only one bit between consecutive values.

Binary reflected Gray code:

```text
gray = n ^ (n >> 1)
```

This is useful in:

- combinatorial generation
- hardware-related algorithms
- state transitions
- bitmask traversal

---

# 62. Hamming Distance

The Hamming distance between two integers is the number of differing bit positions.

Compute:

```text
x ^ y
```

Then count set bits.

---

## Java

```java
static int hammingDistance(int x, int y) {
    return Integer.bitCount(x ^ y);
}
```

Why?

A bit in:

```text
x ^ y
```

is `1` exactly when the corresponding bits differ.

---

# 63. Hamming Weight

Hamming weight is:

```text
number of set bits
```

So:

```java
Integer.bitCount(x)
```

computes the Hamming weight of an `int`.

---

# 64. AND/OR/XOR Aggregate Patterns

For an array:

```text
AND of all values
OR of all values
XOR of all values
```

each operation has different algebraic behavior.

### XOR

Pairs cancel:

```text
x ^ x = 0
```

### AND

Once a bit becomes zero, it remains zero in the aggregate.

### OR

Once a bit becomes one, it remains one in the aggregate.

These monotonicity/cancellation properties often reveal the intended algorithm.

---

# 65. Bitwise AND Over a Range

For a range:

```text
[L, R]
```

the bitwise AND is often found by repeatedly removing differing lower bits or by finding the common binary prefix.

Observation:

```text
If a bit changes anywhere within [L,R],
that bit cannot remain 1 in the final AND.
```

A standard algorithm:

```java
static int rangeBitwiseAnd(int left, int right) {
    int shift = 0;

    while (left < right) {
        left >>= 1;
        right >>= 1;
        shift++;
    }

    return left << shift;
}
```

For nonnegative range values, this finds the common prefix.

---

# 66. XOR Basis — Conceptual Summary

Think:

```text
ordinary linear algebra:
addition → + / -
elimination → Gaussian elimination

XOR linear algebra:
addition → XOR
elimination → XOR
field → GF(2)
```

This is one of the most important advanced bit-manipulation concepts.

---

# 67. GATE Theory — Bitwise Operators

For bitwise operations:

```text
AND → intersection of 1-bits
OR  → union of 1-bits
XOR → positions where bits differ
NOT → complement
```

Example:

```text
1010
1100
----
AND = 1000
OR  = 1110
XOR = 0110
```

---

# 68. GATE Theory — XOR Properties

Important identities:

```text
A ^ 0 = A
A ^ A = 0
A ^ B = B ^ A
(A ^ B) ^ C = A ^ (B ^ C)
A ^ B ^ B = A
```

These explain cancellation-based array algorithms.

---

# 69. GATE Theory — Two's Complement

For an integer `x`:

```text
-x = ~x + 1
```

Therefore:

```text
x & -x
```

isolates the lowest set bit.

And:

```text
~x = -x - 1
```

---

# 70. GATE Theory — Power of Two

For positive `x`:

```text
x is power of 2
iff
x & (x-1) == 0
```

Reason:

A power of two has exactly one set bit.

Subtracting one turns:

```text
1000...0
```

into:

```text
0111...1
```

so the AND becomes zero.

---

# 71. GATE Theory — Brian Kernighan

The transformation:

```text
x → x & (x-1)
```

removes exactly one set bit.

Therefore:

```text
number of loop iterations
=
number of set bits
```

This gives:

```text
O(k)
```

where `k` is the Hamming weight.

---

# 72. GATE Theory — Bitmask Subsets

For `n` elements:

```text
number of subsets = 2^n
```

because every element has two choices:

```text
selected
not selected
```

A mask of `n` bits represents one subset.

---

# 73. GATE Theory — Submask Enumeration

The loop:

```java
for (int sub = mask;
     sub > 0;
     sub = (sub - 1) & mask)
```

enumerates every non-empty submask exactly once.

Across all masks:

```text
O(3^n)
```

submask-mask pairs occur.

---

# 74. GATE Theory — Bitwise Arithmetic

XOR performs binary addition without carrying.

AND identifies positions where a carry is generated.

Shift moves the carry:

```text
carry = (a & b) << 1
```

Repeat until no carry remains.

Thus:

```text
a + b
```

can be implemented using only:

```text
XOR
AND
shift
```

---

# 75. GATE Theory — XOR Basis

An XOR basis represents a vector space over:

```text
GF(2)
```

where:

```text
1 + 1 = 0
```

corresponds to:

```text
1 XOR 1 = 0
```

Gaussian-elimination-style reduction can therefore be performed with XOR.

---

# 76. GATE-Style Solved Questions

> These are **GATE-style practice questions**, not claimed as verbatim official GATE PYQs.

---

## Question 1 — XOR Cancellation

An array contains:

```text
[7, 4, 5, 4, 5]
```

Every element except one occurs exactly twice. Find the unique element using XOR.

### Solution

```text
7 ^ 4 ^ 5 ^ 4 ^ 5
```

Rearrange using associativity and commutativity:

```text
7 ^ (4 ^ 4) ^ (5 ^ 5)
```

Since:

```text
x ^ x = 0
```

we get:

```text
7 ^ 0 ^ 0 = 7
```

### Answer

```text
7
```

---

## Question 2 — Lowest Set Bit

For:

```text
x = 40
```

find:

```text
x & (-x)
```

### Solution

Binary:

```text
40 = 101000₂
```

The lowest set bit is:

```text
001000₂
```

which is:

```text
8
```

### Answer

```text
8
```

---

## Question 3 — Brian Kernighan

How many iterations does:

```java
while (x != 0) {
    x &= x - 1;
    count++;
}
```

execute for:

```text
x = 44
```

### Solution

```text
44 = 101100₂
```

There are three set bits.

Each iteration removes exactly one set bit.

Therefore:

```text
iterations = 3
```

### Answer

```text
3
```

---

## Question 4 — Power of Two

Which of the following values satisfy:

```text
x > 0 && (x & (x - 1)) == 0
```

```text
A. 12
B. 16
C. 18
D. 20
```

### Solution

```text
16 = 10000₂
```

has exactly one set bit.

For all other choices, more than one bit is set.

### Answer

```text
B. 16
```

---

## Question 5 — Subset Count

An array contains `8` distinct elements. How many subsets can be represented using an 8-bit mask?

### Solution

Each of the 8 elements has two states:

```text
0 → not selected
1 → selected
```

Therefore:

```text
2^8 = 256
```

### Answer

```text
256
```

---

## Question 6 — XOR Basis

A set of integers has XOR rank `5`. How many distinct XOR combinations can be generated by the basis?

### Solution

Each of the five independent basis vectors has two choices:

```text
0 → excluded
1 → included
```

Therefore:

```text
2^5 = 32
```

distinct combinations exist.

### Answer

```text
32
```

---

# 77. LeetCode Roadmap

## Phase 1 — Basic Bit Manipulation

Master:

1. Single Number
2. Number of 1 Bits
3. Counting Bits
4. Reverse Bits
5. Power of Two
6. Power of Four
7. Missing Number
8. Hamming Distance
9. Hamming Weight
10. Add Binary

Focus:

```text
XOR
AND
shifts
popcount
x & (x-1)
```

---

# 78. Phase 2 — XOR Patterns

Master:

1. Single Number
2. Single Number II
3. Single Number III
4. Missing Number
5. Find the Difference
6. XOR Queries of a Subarray
7. Maximum XOR of Two Numbers in an Array

Focus:

```text
cancellation
partition by differing bit
prefix XOR
XOR trie
```

---

# 79. Phase 3 — Bitmasking

Master:

1. Subsets
2. Letter Case Permutation
3. Gray Code
4. Iterator/combination problems with masks
5. Small-set state problems
6. Bitmask DP

Focus:

```text
mask
set
clear
toggle
check
enumerate
```

---

# 80. Phase 4 — Advanced Bit Tricks

Master:

1. Counting Bits
2. Reverse Bits
3. Range Bitwise AND
4. Maximum XOR
5. Minimum flips / bit differences
6. Bitwise arithmetic
7. Submask enumeration
8. XOR basis concepts

---

# 81. Problem Recognition

When you see:

### Every value appears twice except one

Think:

```text
XOR
```

---

### Every value appears an even number of times except one

Think:

```text
XOR
```

---

### Two unique values among pairs

Think:

```text
total XOR
→ lowest differing bit
→ partition
→ XOR each group
```

---

### Missing number from a known range

Think:

```text
XOR expected
^
XOR actual
```

---

### Exactly one set bit

Think:

```text
power of 2
```

---

### Need to remove one set bit

Think:

```text
x & (x - 1)
```

---

### Need lowest set bit

Think:

```text
x & -x
```

---

### Need count of set bits

Think:

```text
Integer.bitCount(x)
```

or:

```text
Brian Kernighan
```

---

### Need enumerate subsets

Think:

```text
0 .. (1 << n) - 1
```

---

### Need enumerate submasks

Think:

```text
sub = (sub - 1) & mask
```

---

### Small set of items in a state

Think:

```text
bitmask
```

---

### Maximum XOR

Think:

```text
bitwise greedy
```

or:

```text
XOR trie
```

or, for subset combinations:

```text
XOR basis
```

---

# 82. Common Mistakes

## Mistake 1 — Confusing `^` with exponentiation

In Java:

```java
a ^ b
```

means:

```text
bitwise XOR
```

not:

```text
a raised to b
```

---

## Mistake 2 — Forgetting positivity in power-of-two test

Use:

```java
x > 0 && (x & (x - 1)) == 0
```

because:

```text
0 & (-1) = 0
```

but zero is not a power of two.

---

## Mistake 3 — Using `>>` when logical shift is required

For negative values:

```java
>> 
```

preserves the sign bit.

Use:

```java
>>>
```

when you need zero-fill behavior.

---

## Mistake 4 — Shift overflow

Java shifts are performed within the fixed width of the type.

For `int`:

```text
32-bit
```

and shift distances are masked to five bits.

For `long`:

```text
64-bit
```

and shift distances are masked to six bits.

---

## Mistake 5 — `1 << 31`

This produces:

```text
Integer.MIN_VALUE
```

because bit 31 is the sign bit.

For long masks, use:

```java
1L << bit
```

---

## Mistake 6 — Assuming XOR solves every duplicate problem

XOR is directly useful when multiplicities have cancellation properties compatible with XOR.

If elements occur:

```text
three times
```

ordinary XOR does not simply leave one copy.

The problem may require:

```text
bit counting modulo 3
```

or another technique.

---

# 83. Single Number II Pattern

If every element occurs three times except one, ordinary XOR is insufficient.

For each bit:

```text
count set occurrences
```

and take:

```text
count % 3
```

The remaining bit pattern is the unique value.

This illustrates an important principle:

> The correct bit technique depends on the frequency structure.

---

# 84. Counting Bits for Every Number

For:

```text
0..n
```

there is a useful recurrence:

```text
bits[i] = bits[i >> 1] + (i & 1)
```

because:

```text
i >> 1
```

removes the least significant bit.

---

## Java

```java
static int[] countBits(int n) {
    int[] bits = new int[n + 1];

    for (int i = 1; i <= n; i++) {
        bits[i] =
            bits[i >> 1] + (i & 1);
    }

    return bits;
}
```

Complexity:

```text
O(n)
```

---

# 85. Alternative Count-Bits Recurrence

Another recurrence:

```text
bits[i] = bits[i & (i - 1)] + 1
```

because:

```text
i & (i - 1)
```

removes one set bit.

Thus:

```java
bits[i] =
    bits[i & (i - 1)] + 1;
```

---

# 86. Prefix XOR

For an array:

```text
a[0], a[1], ..., a[n-1]
```

define:

```text
prefix[i+1] =
prefix[i] ^ a[i]
```

Then XOR of range `[l, r]` is:

```text
prefix[r+1] ^ prefix[l]
```

because the prefix before `l` cancels.

This is analogous to prefix sums, but with XOR.

---

# 87. XOR Trie Connection

For maximum XOR between two numbers:

```text
choose the opposite bit whenever possible
```

because:

```text
1 ^ 0 = 1
```

is better than:

```text
1 ^ 1 = 0
```

A binary trie stores bits from most significant to least significant.

This provides a greedy bit-by-bit solution.

Typical complexity:

```text
O(32n)
```

for `int` values.

---

# 88. XOR Basis vs XOR Trie

| Technique | Best suited for |
|---|---|
| XOR cancellation | frequency pairs / missing values |
| XOR prefix | range XOR |
| XOR trie | maximum XOR between values |
| XOR basis | XOR combinations/subsets |
| Bit counting | frequency modulo k |
| Bitmask DP | subset state |

Recognizing which structure matches the problem is more important than memorizing individual implementations.

---

# 89. Advanced Bit Trick: Isolate a Mask

Suppose:

```text
x = 10110100
```

Then:

```text
x & -x
```

isolates:

```text
00000100
```

and:

```text
x & (x - 1)
```

produces:

```text
10110000
```

This pair should become automatic.

---

# 90. Advanced Bit Trick: Clear Lowest Set Bit

```java
x &= x - 1;
```

Use when:

- counting bits
- iterating set bits
- removing selected features
- manipulating subset masks

---

# 91. Advanced Bit Trick: Extract Lowest Set Bit

```java
int low = x & -x;
```

Use when:

- finding a differing bit
- partitioning by a bit
- enumerating set bits
- Fenwick tree indexing

---

# 92. Advanced Bit Trick: Test Whether Two Numbers Differ at a Bit

```java
boolean different =
    ((a ^ b) & (1 << bit)) != 0;
```

More simply:

```java
((a ^ b) & mask) != 0
```

This directly identifies bit-level differences.

---

# 93. Advanced Bit Trick: Common Prefix

For range operations such as range AND:

```text
keep the common high-order prefix
zero the differing suffix
```

Repeatedly shift both values right until they become equal, then shift the common prefix back.

This is the conceptual basis of the range AND algorithm.

---

# 94. Bitwise Identities to Memorize

```text
x ^ 0 = x
x ^ x = 0

x & 0 = 0
x & allOnes = x

x | 0 = x
x | allOnes = allOnes

x & (x - 1)
→ removes lowest set bit

x & -x
→ isolates lowest set bit

~x
= -x - 1

-x
= ~x + 1
```

---

# 95. Java Bit API to Know

```java
Integer.bitCount(x)
Integer.numberOfLeadingZeros(x)
Integer.numberOfTrailingZeros(x)
Integer.highestOneBit(x)
Integer.lowestOneBit(x)
Integer.reverse(x)
Integer.reverseBytes(x)
Integer.rotateLeft(x, distance)
Integer.rotateRight(x, distance)
```

For `long`:

```java
Long.bitCount(x)
Long.numberOfLeadingZeros(x)
Long.numberOfTrailingZeros(x)
Long.highestOneBit(x)
Long.lowestOneBit(x)
Long.reverse(x)
Long.rotateLeft(x, distance)
Long.rotateRight(x, distance)
```

Know the standard library equivalents, but also understand the underlying operations.

---

# 96. Bit Manipulation Complexity

Most operations on a Java `int` are:

```text
O(1)
```

because the width is fixed at 32 bits.

Similarly, operations on `long` are:

```text
O(1)
```

with fixed 64-bit width.

However, when discussing a generalized `w`-bit model:

```text
bit-by-bit algorithms may be O(w)
```

This distinction matters in theoretical analysis.

---

# 97. GATE Complexity Perspective

If an integer is treated as a fixed-width machine word:

```text
bitwise operation = O(1)
```

If the number of bits is considered part of the input size:

```text
processing every bit = O(w)
```

GATE questions may implicitly rely on machine-word assumptions, so read the model carefully.

---

# 98. Interview Strategy

When a problem appears to involve bits:

### Step 1

Write a small binary example.

### Step 2

Look for:

```text
cancellation
```

### Step 3

Look for:

```text
one set bit
```

### Step 4

Try:

```text
x & (x - 1)
x & -x
```

### Step 5

Check whether a subset can be represented by a mask.

### Step 6

If maximizing XOR:

```text
XOR trie / greedy bits
```

### Step 7

If combining arbitrary XOR subsets:

```text
XOR basis
```

---

# 99. One-Page Bit Manipulation Cheat Sheet

```text
AND:
x & y

OR:
x | y

XOR:
x ^ y

NOT:
~x

Left shift:
x << k

Arithmetic right shift:
x >> k

Logical right shift:
x >>> k

Check bit i:
(x & (1 << i)) != 0

Set bit i:
x | (1 << i)

Clear bit i:
x & ~(1 << i)

Toggle bit i:
x ^ (1 << i)

Get bit i:
(x >>> i) & 1

Remove lowest set bit:
x & (x - 1)

Isolate lowest set bit:
x & -x

Power of 2:
x > 0 && (x & (x - 1)) == 0

Count bits:
Integer.bitCount(x)

Lowest set-bit position:
Integer.numberOfTrailingZeros(x)

Highest set-bit position:
31 - Integer.numberOfLeadingZeros(x)

Odd:
(x & 1) != 0

All subsets:
0 .. (1 << n) - 1

Enumerate submasks:
sub = (sub - 1) & mask

Prefix XOR:
prefix[i+1] = prefix[i] ^ a[i]

Range XOR:
prefix[r+1] ^ prefix[l]

Add without +:
xor + carry loop

XOR basis:
Gaussian elimination over GF(2)
```

---

# 100. Final Mastery Checklist

## Fundamentals

- [ ] Binary representation
- [ ] AND
- [ ] OR
- [ ] XOR
- [ ] NOT
- [ ] Left shift
- [ ] Arithmetic right shift
- [ ] Logical right shift
- [ ] Java integer widths
- [ ] Two's complement

## XOR

- [ ] XOR cancellation
- [ ] Single Number
- [ ] Missing Number
- [ ] Two unique numbers
- [ ] XOR range
- [ ] Prefix XOR
- [ ] Single Number II frequency pattern
- [ ] Single Number III partition pattern

## Bit Masks

- [ ] Set bit
- [ ] Clear bit
- [ ] Toggle bit
- [ ] Check bit
- [ ] Get bit
- [ ] Feature masks
- [ ] Subset representation
- [ ] Submask enumeration

## Bit Tricks

- [ ] `x & (x - 1)`
- [ ] `x & -x`
- [ ] Power of 2
- [ ] Power of 4
- [ ] Count bits
- [ ] Brian Kernighan
- [ ] Leading zeros
- [ ] Trailing zeros
- [ ] Highest/lowest one bit

## Bitwise Arithmetic

- [ ] Add without `+`
- [ ] Understand carry generation
- [ ] Understand two's complement subtraction
- [ ] Understand shift arithmetic
- [ ] Understand overflow

## Advanced

- [ ] Bitmask subsets
- [ ] Gray code
- [ ] Hamming distance
- [ ] Range AND
- [ ] XOR trie concept
- [ ] XOR basis
- [ ] GF(2) interpretation
- [ ] Maximum subset XOR
- [ ] XOR linear independence

---

# 101. Final Revision Questions

Before an interview, you should be able to answer immediately:

### Q1. Why does XOR cancel pairs?

```text
x ^ x = 0
```

---

### Q2. How do you remove the lowest set bit?

```text
x & (x - 1)
```

---

### Q3. How do you isolate the lowest set bit?

```text
x & -x
```

---

### Q4. How do you check if a positive number is a power of two?

```text
(x & (x - 1)) == 0
```

---

### Q5. How do you count set bits?

```text
Integer.bitCount(x)
```

or:

```text
x &= x - 1
```

repeatedly.

---

### Q6. How do you represent all subsets?

```text
0..(1<<n)-1
```

---

### Q7. How do you enumerate submasks?

```text
sub = (sub - 1) & mask
```

---

### Q8. How do you add without `+`?

```text
sum = a ^ b
carry = (a & b) << 1
```

repeat until carry is zero.

---

### Q9. How do you find two unique numbers among pairs?

```text
total XOR
→ isolate differing bit
→ partition
→ XOR groups
```

---

### Q10. What is an XOR basis?

```text
Gaussian-elimination-style basis
over GF(2)
```

---

# 102. Final Mastery Standard

You have mastered **Bit Manipulation** when you can:

1. Convert between decimal and binary mentally for small values.
2. Explain AND, OR, XOR, NOT, and shifts.
3. Use XOR cancellation without hesitation.
4. Solve Single Number.
5. Solve Missing Number using XOR.
6. Solve the two-unique-elements problem.
7. Set, clear, toggle, and check arbitrary bits.
8. Explain two's complement.
9. Use `x & (x - 1)` correctly.
10. Use `x & -x` correctly.
11. Check powers of two and four.
12. Count set bits using both Java APIs and Brian Kernighan's algorithm.
13. Enumerate all subsets with bit masks.
14. Enumerate all submasks efficiently.
15. Use prefix XOR for range queries.
16. Understand bitwise arithmetic.
17. Distinguish `>>` from `>>>` in Java.
18. Handle shift-width and sign-bit issues.
19. Recognize Hamming-distance problems.
20. Recognize maximum-XOR problems.
21. Know when an XOR trie is appropriate.
22. Understand XOR basis as linear algebra over GF(2).
23. Build an XOR basis for advanced subset-XOR problems.
24. Derive maximum subset XOR from a basis.
25. Identify whether the problem is fundamentally about cancellation, masks, bit counting, greedy bit choices, or XOR linear algebra.

> **Core principle:**  
> **Do not memorize isolated bit tricks. Understand what information each operation preserves, removes, or isolates. Once that is clear, most bit-manipulation patterns become derivable.**
