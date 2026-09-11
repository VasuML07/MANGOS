# 21. Math / Number Theory

> **Goal:** Master the mathematics and number-theory patterns that repeatedly appear in LeetCode and FAANG/top product-company interviews.
>
> **Language:** Java  
> **Difficulty target:** Easy → Medium → Hard, with Medium dominating.
>
> **Scope:** GCD/LCM, Euclidean algorithm, primes, sieve, factorization, modular arithmetic, fast/modular exponentiation, combinations, permutations, probability basics, matrix exponentiation, geometry, coordinate geometry, and integer overflow handling.
>
> **Core principle:** Do not treat these as a separate mathematics course. Learn the small set of mathematical tools that lets you reduce LeetCode problems from simulation to efficient formulas and logarithmic/near-linear algorithms.

---

# 1. GCD

The **greatest common divisor** of two integers `a` and `b` is the largest positive integer dividing both.

Notation:

```text
gcd(a, b)
```

Example:

```text
gcd(24, 18) = 6
```

because:

```text
24 = 6 × 4
18 = 6 × 3
```

and no larger positive integer divides both.

---

# 2. Basic GCD Properties

Important properties:

```text
gcd(a, 0) = |a|
gcd(0, b) = |b|
gcd(a, a) = |a|
gcd(a, b) = gcd(b, a)
```

Also:

```text
gcd(a, b) = gcd(|a|, |b|)
```

for integer inputs.

---

# 3. Euclidean Algorithm

The central identity is:

```text
gcd(a, b) = gcd(b, a % b)
```

for `b != 0`.

Repeatedly replace:

```text
(a, b)
```

with:

```text
(b, a % b)
```

until:

```text
b = 0
```

Then:

```text
gcd = a
```

---

# 4. Why Euclidean Algorithm Works

Suppose:

```text
a = qb + r
```

Any common divisor of `a` and `b` also divides:

```text
a - qb = r
```

Conversely, any common divisor of `b` and `r` divides:

```text
qb + r = a
```

Therefore the common divisors are identical:

```text
gcd(a, b) = gcd(b, r)
```

where:

```text
r = a % b
```

---

# 5. Java GCD

```java
static long gcd(long a, long b) {
    a = Math.abs(a);
    b = Math.abs(b);

    while (b != 0) {
        long temp = a % b;
        a = b;
        b = temp;
    }

    return a;
}
```

Complexity:

```text
O(log(min(|a|, |b|)))
```

---

# 6. Recursive GCD

```java
static long gcd(long a, long b) {
    if (b == 0) {
        return Math.abs(a);
    }

    return gcd(b, a % b);
}
```

The iterative version avoids recursion depth concerns.

---

# 7. GCD of an Array

For:

```text
a[0], a[1], ..., a[n-1]
```

compute:

```text
g = gcd(a[0], a[1])
g = gcd(g, a[2])
...
```

---

## Java

```java
static long gcdArray(int[] nums) {
    long g = 0;

    for (int x : nums) {
        g = gcd(g, x);
    }

    return g;
}
```

The identity:

```text
gcd(0, x) = |x|
```

makes this convenient.

---

# 8. GCD Pattern Recognition

Think about GCD when the problem contains:

- common divisibility
- largest common factor
- reducing a fraction
- repeated equal-sized groups
- periodicity
- coprime conditions
- integer ratios
- simplifying integer pairs
- divisibility constraints

---

# 9. LCM

The **least common multiple** of positive integers `a` and `b` is the smallest positive number divisible by both.

Example:

```text
LCM(12, 18) = 36
```

---

# 10. GCD-LCM Identity

For positive integers:

```text
gcd(a, b) × lcm(a, b) = a × b
```

Therefore:

```text
lcm(a, b) = (a / gcd(a, b)) × b
```

Prefer division before multiplication to reduce overflow risk.

---

# 11. Java LCM

```java
static long lcm(long a, long b) {
    if (a == 0 || b == 0) {
        return 0;
    }

    return Math.abs((a / gcd(a, b)) * b);
}
```

If inputs can approach `Long.MAX_VALUE`, even this can overflow. Use overflow checks when required.

---

# 12. LCM of an Array

```java
static long lcmArray(int[] nums) {
    long result = 1;

    for (int x : nums) {
        result = lcm(result, x);
    }

    return result;
}
```

The result can grow extremely quickly, so constraints determine whether `long` is sufficient.

---

# 13. Extended Euclidean Algorithm

The extended Euclidean algorithm finds integers `x` and `y` such that:

```text
ax + by = gcd(a, b)
```

These are called **Bézout coefficients**.

Example:

```text
gcd(30, 18) = 6
```

One representation is:

```text
30(−1) + 18(2) = 6
```

---

# 14. Why Extended GCD Matters

It is useful for:

- modular inverses
- solving linear Diophantine equations
- number-theory constraints
- modular arithmetic when the modulus is not prime
- Chinese Remainder Theorem implementations

---

## Java

```java
static long[] extendedGcd(long a, long b) {
    if (b == 0) {
        return new long[]{a, 1, 0};
    }

    long[] next = extendedGcd(b, a % b);

    long g = next[0];
    long x1 = next[1];
    long y1 = next[2];

    long x = y1;
    long y = x1 - (a / b) * y1;

    return new long[]{g, x, y};
}
```

The returned array is:

```text
[gcd, x, y]
```

satisfying:

```text
a*x + b*y = gcd
```

---

# 15. Coprime Numbers

Two integers are **coprime** if:

```text
gcd(a, b) = 1
```

Examples:

```text
8 and 15
14 and 25
```

are coprime.

Important:

> Coprime does not mean both numbers are prime.

For example:

```text
8
```

is composite, but:

```text
gcd(8, 15) = 1
```

---

# 16. Prime Numbers

A prime number is an integer greater than `1` with exactly two positive divisors:

```text
1
itself
```

Examples:

```text
2, 3, 5, 7, 11, 13, 17
```

---

# 17. Composite Numbers

A composite number is an integer greater than `1` that has a divisor other than:

```text
1
itself
```

Examples:

```text
4, 6, 8, 9, 10, 12
```

Important:

```text
1 is neither prime nor composite.
```

---

# 18. Testing Whether a Number Is Prime

A number `n` is composite if it has a factor:

```text
d <= sqrt(n)
```

Therefore only test divisors up to:

```text
sqrt(n)
```

---

## Java

```java
static boolean isPrime(long n) {
    if (n < 2) {
        return false;
    }

    if (n % 2 == 0) {
        return n == 2;
    }

    for (long d = 3; d <= n / d; d += 2) {
        if (n % d == 0) {
            return false;
        }
    }

    return true;
}
```

Using:

```java
d <= n / d
```

instead of:

```java
d * d <= n
```

avoids multiplication overflow.

Complexity:

```text
O(sqrt(n))
```

---

# 19. Why Check Only Up to sqrt(n)?

If:

```text
n = a × b
```

and both:

```text
a > sqrt(n)
b > sqrt(n)
```

then:

```text
a × b > n
```

which is impossible.

Therefore every composite number has at least one factor:

```text
<= sqrt(n)
```

---

# 20. Sieve of Eratosthenes

The Sieve of Eratosthenes finds all primes up to `n`.

Initialize:

```text
isPrime[i] = true
```

for candidate numbers.

Then:

1. Start at `2`.
2. If `p` is still prime, mark its multiples composite.
3. Start marking at:

```text
p × p
```

because smaller multiples have already been handled by smaller primes.

---

# 21. Sieve Java Implementation

```java
static boolean[] sieve(int n) {
    boolean[] isPrime = new boolean[n + 1];

    if (n >= 2) {
        java.util.Arrays.fill(isPrime, true);
        isPrime[0] = false;
        isPrime[1] = false;
    }

    for (int p = 2; p <= n / p; p++) {
        if (!isPrime[p]) {
            continue;
        }

        for (int multiple = p * p;
             multiple <= n;
             multiple += p) {
            isPrime[multiple] = false;
        }
    }

    return isPrime;
}
```

Complexity:

```text
Time: O(n log log n)
Space: O(n)
```

---

# 22. Why Start Sieve at p²?

For prime:

```text
p
```

its smaller multiples are:

```text
2p
3p
4p
...
(p-1)p
```

Each has a factor smaller than `p`, so it was already marked.

The first potentially unmarked multiple is:

```text
p²
```

---

# 23. List All Primes

```java
static java.util.List<Integer> primesUpTo(int n) {
    boolean[] prime = sieve(n);
    java.util.List<Integer> result =
        new java.util.ArrayList<>();

    for (int i = 2; i <= n; i++) {
        if (prime[i]) {
            result.add(i);
        }
    }

    return result;
}
```

---

# 24. Sieve Pattern Recognition

Use a sieve when:

```text
n is moderate
```

and the problem asks about:

- many primality queries
- all primes up to `n`
- prime counts
- smallest prime factors
- repeated factorization
- prime-related preprocessing

A single primality query usually does not require a full sieve.

---

# 25. Smallest Prime Factor (SPF)

Instead of only storing:

```text
isPrime[i]
```

store:

```text
spf[i]
```

where `spf[i]` is the smallest prime factor of `i`.

Example:

```text
spf[12] = 2
spf[15] = 3
spf[49] = 7
```

This makes repeated prime factorization efficient.

---

# 26. SPF Sieve

```java
static int[] buildSPF(int n) {
    int[] spf = new int[n + 1];

    for (int i = 0; i <= n; i++) {
        spf[i] = i;
    }

    if (n >= 0) spf[0] = 0;
    if (n >= 1) spf[1] = 1;

    for (int p = 2; p <= n / p; p++) {
        if (spf[p] != p) {
            continue;
        }

        for (int x = p * p; x <= n; x += p) {
            if (spf[x] == x) {
                spf[x] = p;
            }
        }
    }

    return spf;
}
```

---

# 27. Prime Factorization

Every integer greater than `1` can be represented uniquely as a product of primes:

```text
n = p1^a1 × p2^a2 × ...
```

Example:

```text
360
= 2^3 × 3^2 × 5
```

This is the **Fundamental Theorem of Arithmetic**.

---

# 28. Trial-Division Factorization

```java
static java.util.Map<Long, Integer> factorize(long n) {
    java.util.Map<Long, Integer> factors =
        new java.util.LinkedHashMap<>();

    for (long p = 2; p <= n / p; p++) {
        while (n % p == 0) {
            factors.merge(p, 1, Integer::sum);
            n /= p;
        }
    }

    if (n > 1) {
        factors.merge(n, 1, Integer::sum);
    }

    return factors;
}
```

Complexity in the worst case:

```text
O(sqrt(n))
```

---

# 29. Factorization Using SPF

For many values bounded by `N`:

1. Build SPF once.
2. Repeatedly take:

```text
p = spf[n]
```

3. Divide by `p`.

---

## Java

```java
static java.util.List<Integer> factorizeWithSPF(
        int x, int[] spf) {

    java.util.List<Integer> factors =
        new java.util.ArrayList<>();

    while (x > 1) {
        factors.add(spf[x]);
        x /= spf[x];
    }

    return factors;
}
```

This returns prime factors with multiplicity.

---

# 30. Distinct Prime Factors

If only distinct factors are needed:

```java
static java.util.List<Integer> distinctFactors(
        int x, int[] spf) {

    java.util.List<Integer> result =
        new java.util.ArrayList<>();

    while (x > 1) {
        int p = spf[x];
        result.add(p);

        while (x % p == 0) {
            x /= p;
        }
    }

    return result;
}
```

---

# 31. Number of Divisors

If:

```text
n = p1^a1 × p2^a2 × ... × pk^ak
```

then the number of positive divisors is:

```text
(a1 + 1)(a2 + 1)...(ak + 1)
```

Example:

```text
360 = 2^3 × 3^2 × 5^1
```

Number of divisors:

```text
(3+1)(2+1)(1+1)
= 4 × 3 × 2
= 24
```

---

# 32. Sum of Divisors

For:

```text
n = p1^a1 × p2^a2 × ... × pk^ak
```

the sum of all positive divisors is:

```text
Π (1 + p + p² + ... + p^a)
```

For:

```text
360 = 2^3 × 3^2 × 5
```

sum:

```text
(1+2+4+8)
×
(1+3+9)
×
(1+5)
```

---

# 33. Divisor Enumeration

To enumerate divisors without factorization:

```java
static java.util.List<Long> divisors(long n) {
    java.util.List<Long> result =
        new java.util.ArrayList<>();

    for (long d = 1; d <= n / d; d++) {
        if (n % d == 0) {
            result.add(d);

            if (d != n / d) {
                result.add(n / d);
            }
        }
    }

    return result;
}
```

Complexity:

```text
O(sqrt(n))
```

---

# 34. Modular Arithmetic

For modulus `m`:

```text
a mod m
```

is the remainder after division by `m`.

Core identities:

```text
(a + b) mod m
=
((a mod m) + (b mod m)) mod m

(a - b) mod m
=
((a mod m) - (b mod m)) mod m

(a × b) mod m
=
((a mod m) × (b mod m)) mod m
```

These allow intermediate values to be reduced.

---

# 35. Why Modulo Is Useful in LeetCode

It is commonly needed because:

- answers may be enormous
- combinatorial counts grow exponentially
- repeated multiplication can overflow
- the problem explicitly asks for an answer modulo some number

A common modulus is:

```text
1,000,000,007
```

which is prime.

---

# 36. Modular Addition

In Java:

```java
static long addMod(long a, long b, long mod) {
    return (a % mod + b % mod) % mod;
}
```

For values already normalized to `[0, mod)` and where the sum fits in `long`, this is sufficient.

---

# 37. Modular Subtraction

Java's `%` can return a negative remainder.

Therefore normalize:

```java
static long subMod(long a, long b, long mod) {
    return ((a % mod - b % mod) + mod) % mod;
}
```

Example:

```text
(3 - 5) mod 7 = 5
```

not:

```text
-2
```

---

# 38. Modular Multiplication

For ordinary LeetCode constraints where:

```text
a, b < 1e9+7
```

their product fits within signed `long`.

```java
long product = (a * b) % mod;
```

For larger values, multiplication itself can overflow `long`; use a safer multiplication strategy when constraints require it.

---

# 39. Modular Exponentiation

Computing:

```text
a^b mod m
```

naively takes:

```text
O(b)
```

multiplications.

Binary exponentiation reduces this to:

```text
O(log b)
```

---

# 40. Fast Exponentiation

Observe:

If `b` is even:

```text
a^b = (a^(b/2))²
```

If `b` is odd:

```text
a^b = a × a^(b-1)
```

Repeated squaring reduces the exponent exponentially.

---

# 41. Iterative Fast Power

```java
static long fastPow(long base, long exp) {
    long result = 1;

    while (exp > 0) {
        if ((exp & 1) != 0) {
            result *= base;
        }

        base *= base;
        exp >>= 1;
    }

    return result;
}
```

Complexity:

```text
O(log exp)
```

---

# 42. Modular Fast Power

```java
static long modPow(
        long base,
        long exp,
        long mod) {

    base %= mod;
    long result = 1 % mod;

    while (exp > 0) {
        if ((exp & 1) != 0) {
            result = (result * base) % mod;
        }

        base = (base * base) % mod;
        exp >>= 1;
    }

    return result;
}
```

Complexity:

```text
O(log exp)
```

---

# 43. Why Binary Exponentiation Works

Example:

```text
13 = 1101₂
```

Therefore:

```text
a^13
=
a^(8+4+1)
=
a^8 × a^4 × a
```

Repeated squaring computes:

```text
a
a²
a⁴
a⁸
```

and multiplies only the powers corresponding to set bits.

---

# 44. Modular Inverse

For prime modulus `p` and:

```text
a % p != 0
```

Fermat's Little Theorem gives:

```text
a^(p-1) ≡ 1 (mod p)
```

Therefore:

```text
a^(-1)
≡
a^(p-2) mod p
```

So:

```java
static long modInversePrime(
        long a, long p) {

    return modPow(a, p - 2, p);
}
```

---

# 45. When Fermat's Inverse Applies

The simple formula:

```text
a^(p-2) mod p
```

requires:

```text
p is prime
gcd(a, p) = 1
```

Do not use it blindly for arbitrary composite moduli.

For a general modulus, extended Euclidean algorithm can find an inverse when:

```text
gcd(a, mod) = 1
```

---

# 46. Modular Division

There is generally no direct:

```text
a / b mod m
```

operation.

Instead, if the inverse exists:

```text
a / b
≡
a × b^(-1) mod m
```

For prime `m`:

```text
b^(-1) = b^(m-2) mod m
```

when `b` is not divisible by `m`.

---

# 47. Combinations

The number of ways to choose `r` elements from `n` elements is:

```text
C(n, r)
=
n! / (r!(n-r)!)
```

Also written:

```text
nCr
```

---

# 48. Combination Identities

Important:

```text
C(n, 0) = 1
C(n, n) = 1
C(n, r) = C(n, n-r)
```

Pascal identity:

```text
C(n, r)
=
C(n-1, r-1)
+
C(n-1, r)
```

---

# 49. Pascal Triangle

Rows:

```text
1
1 1
1 2 1
1 3 3 1
1 4 6 4 1
```

Each interior value is:

```text
C(n, r)
```

This can compute combinations in:

```text
O(n²)
```

without modular inverses.

---

# 50. Exact nCr With Overflow Awareness

Do not compute:

```text
n!
```

directly if `n` can be moderately large.

Instead use:

```text
C(n,r)
=
C(n,r-1) × (n-r+1) / r
```

and reduce `r` using:

```text
r = min(r, n-r)
```

For very large exact results, `BigInteger` may be required.

---

# 51. nCr Modulo Prime

For many queries with fixed maximum `N`, precompute:

```text
fact[i] = i! mod MOD
invFact[i] = (i!)^(-1) mod MOD
```

Then:

```text
C(n,r)
=
fact[n] × invFact[r] × invFact[n-r] mod MOD
```

---

# 52. Java nCr Modulo

```java
static final long MOD = 1_000_000_007L;

static long modPow(long a, long e) {
    long result = 1;

    while (e > 0) {
        if ((e & 1) != 0) {
            result = result * a % MOD;
        }

        a = a * a % MOD;
        e >>= 1;
    }

    return result;
}

static long[] factorials(int n) {
    long[] fact = new long[n + 1];
    fact[0] = 1;

    for (int i = 1; i <= n; i++) {
        fact[i] = fact[i - 1] * i % MOD;
    }

    return fact;
}

static long nCrMod(
        int n,
        int r,
        long[] fact) {

    if (r < 0 || r > n) {
        return 0;
    }

    long numerator = fact[n];

    long denominator =
        modPow(fact[r], MOD - 2)
        * modPow(fact[n - r], MOD - 2)
        % MOD;

    return numerator * denominator % MOD;
}
```

For many queries, precompute inverse factorials instead of calling modular exponentiation twice per query.

---

# 53. Efficient Factorial + Inverse Factorial Precomputation

```java
static class Comb {
    static final long MOD = 1_000_000_007L;

    long[] fact;
    long[] invFact;

    Comb(int n) {
        fact = new long[n + 1];
        invFact = new long[n + 1];

        fact[0] = 1;

        for (int i = 1; i <= n; i++) {
            fact[i] = fact[i - 1] * i % MOD;
        }

        invFact[n] = modPow(fact[n], MOD - 2);

        for (int i = n; i >= 1; i--) {
            invFact[i - 1] =
                invFact[i] * i % MOD;
        }
    }

    long choose(int n, int r) {
        if (r < 0 || r > n) {
            return 0;
        }

        return fact[n]
            * invFact[r] % MOD
            * invFact[n - r] % MOD;
    }

    static long modPow(long a, long e) {
        long result = 1;

        while (e > 0) {
            if ((e & 1) != 0) {
                result = result * a % MOD;
            }

            a = a * a % MOD;
            e >>= 1;
        }

        return result;
    }
}
```

Complexity:

```text
Preprocessing: O(N)
Each query: O(1)
Memory: O(N)
```

---

# 54. Permutations

The number of ways to arrange `r` distinct elements selected from `n` is:

```text
P(n,r)
=
n! / (n-r)!
```

Examples:

```text
P(5,2)
=
5 × 4
=
20
```

---

# 55. Relationship Between Combinations and Permutations

```text
P(n,r) = C(n,r) × r!
```

Reason:

1. Choose the `r` elements.
2. Arrange them in `r!` orders.

---

# 56. Repeated Elements

If there are `n` total objects and multiplicities:

```text
a1, a2, ..., ak
```

the number of distinct permutations is:

```text
n! / (a1! a2! ... ak!)
```

Example:

```text
AABC
```

has:

```text
4! / 2!
= 12
```

distinct arrangements.

---

# 57. Probability Basics

Probability of an event:

```text
P(A)
=
number of favorable outcomes
/
number of total equally likely outcomes
```

Example:

A fair die:

```text
P(rolling 4) = 1/6
```

---

# 58. Complement Rule

Often easier:

```text
P(A) = 1 - P(not A)
```

This is especially useful for:

```text
at least one
```

questions.

Example:

Probability of at least one success:

```text
1 - P(no successes)
```

---

# 59. Independent Events

If events `A` and `B` are independent:

```text
P(A and B)
=
P(A) × P(B)
```

Example:

Two independent coin tosses:

```text
P(H,H)
=
1/2 × 1/2
=
1/4
```

---

# 60. Conditional Probability

Conditional probability:

```text
P(A | B)
=
P(A and B) / P(B)
```

provided:

```text
P(B) > 0
```

Independence means:

```text
P(A | B) = P(A)
```

---

# 61. Expected Value

For a discrete random variable:

```text
E[X]
=
Σ x × P(X=x)
```

Linearity of expectation:

```text
E[X + Y]
=
E[X] + E[Y]
```

This holds even when `X` and `Y` are not independent.

This is frequently useful in expected-value counting problems.

---

# 62. Binomial Probability

For `n` independent trials with success probability `p`, the probability of exactly `k` successes is:

```text
C(n,k) p^k (1-p)^(n-k)
```

For a fair binary process:

```text
p = 1/2
```

so:

```text
P(k successes)
=
C(n,k) / 2^n
```

---

# 63. Probability in LeetCode

Probability questions usually require:

- combinations
- permutations
- expected value
- DP
- complementary probability
- modular arithmetic when exact fractions are represented modulo a prime

The mathematical model matters more than memorizing formulas.

---

# 64. Matrix Exponentiation

Matrix exponentiation computes:

```text
A^k
```

in:

```text
O(d³ log k)
```

for a `d × d` matrix using ordinary cubic multiplication.

It becomes especially useful when a recurrence can be represented as a matrix transformation.

---

# 65. Fibonacci Matrix

Fibonacci recurrence:

```text
F(n) = F(n-1) + F(n-2)
```

can be represented as:

```text
[ F(n)   ]   [1 1] [F(n-1)]
[ F(n-1) ] = [1 0] [F(n-2)]
```

Therefore:

```text
[ F(n)   ] =
[1 1]^(n-1)
[ F(n-1) ]

×

[ F(1) ]
[ F(0) ]
```

---

# 66. Matrix Multiplication

For matrices `A` and `B`:

```text
C[i][j]
=
Σ A[i][k] × B[k][j]
```

For `d × d` matrices:

```text
O(d³)
```

time.

---

# 67. Java Matrix Multiplication

```java
static long[][] multiply(
        long[][] a,
        long[][] b,
        long mod) {

    int n = a.length;
    int m = b[0].length;
    int common = b.length;

    long[][] c = new long[n][m];

    for (int i = 0; i < n; i++) {
        for (int k = 0; k < common; k++) {
            if (a[i][k] == 0) {
                continue;
            }

            for (int j = 0; j < m; j++) {
                c[i][j] =
                    (c[i][j]
                    + a[i][k] * b[k][j]) % mod;
            }
        }
    }

    return c;
}
```

---

# 68. Matrix Fast Exponentiation

```java
static long[][] matrixPow(
        long[][] base,
        long exp,
        long mod) {

    int n = base.length;
    long[][] result = identity(n);

    while (exp > 0) {
        if ((exp & 1) != 0) {
            result = multiply(result, base, mod);
        }

        base = multiply(base, base, mod);
        exp >>= 1;
    }

    return result;
}

static long[][] identity(int n) {
    long[][] id = new long[n][n];

    for (int i = 0; i < n; i++) {
        id[i][i] = 1;
    }

    return id;
}
```

Complexity:

```text
O(n³ log exp)
```

for an `n × n` matrix.

---

# 69. When Matrix Exponentiation Is Useful

Look for:

```text
linear recurrence
```

such as:

```text
F(n) = aF(n-1) + bF(n-2)
```

or more generally:

```text
F(n)
=
c1F(n-1)
+
c2F(n-2)
+
...
+
ckF(n-k)
```

If `n` is enormous, ordinary DP is impossible.

Matrix exponentiation can reduce the recurrence to:

```text
O(k³ log n)
```

for a `k × k` state matrix.

---

# 70. Geometry Basics

Geometry problems in LeetCode usually require:

- coordinates
- distances
- slopes
- cross products
- orientation
- collinearity
- rectangles
- circles
- area
- intersection logic

Floating-point geometry should be avoided when an exact integer formulation exists.

---

# 71. Coordinate Geometry

For points:

```text
A(x1, y1)
B(x2, y2)
```

difference vector:

```text
(dx, dy)
=
(x2-x1, y2-y1)
```

This vector is the basis of:

- slope
- distance
- direction
- orientation
- line comparisons

---

# 72. Distance Between Two Points

Euclidean distance:

```text
d
=
sqrt(
    (x2-x1)^2
    +
    (y2-y1)^2
)
```

Often you only need to compare distances.

Then avoid `sqrt`.

Compare:

```text
dx² + dy²
```

directly.

---

# 73. Squared Distance

```java
static long squaredDistance(
        long x1, long y1,
        long x2, long y2) {

    long dx = x2 - x1;
    long dy = y2 - y1;

    return dx * dx + dy * dy;
}
```

If coordinates can be large enough that squaring overflows `long`, use a wider representation or checked arithmetic.

---

# 74. Slope

For two points:

```text
slope = (y2-y1)/(x2-x1)
```

But directly using floating point can create precision problems.

For exact comparison, represent the slope as a reduced pair:

```text
dy / dx
```

using:

```text
g = gcd(|dy|, |dx|)
```

then:

```text
dy /= g
dx /= g
```

and normalize the sign.

---

# 75. Canonical Slope Representation

```java
static long[] normalizeSlope(
        long dx, long dy) {

    if (dx == 0) {
        return new long[]{1, 0};
    }

    if (dy == 0) {
        return new long[]{0, 1};
    }

    long g = gcd(Math.abs(dx), Math.abs(dy));

    dx /= g;
    dy /= g;

    if (dx < 0) {
        dx = -dx;
        dy = -dy;
    }

    return new long[]{dy, dx};
}
```

A canonical representation lets equal slopes map to the same key.

---

# 76. Collinearity

Three points:

```text
A(x1,y1)
B(x2,y2)
C(x3,y3)
```

are collinear if:

```text
(x2-x1)(y3-y1)
=
(y2-y1)(x3-x1)
```

This avoids division entirely.

---

# 77. Cross Product

For vectors:

```text
A = (ax, ay)
B = (bx, by)
```

the 2D cross product is:

```text
cross(A,B)
=
ax*by - ay*bx
```

Interpretation:

```text
> 0 → counterclockwise orientation
< 0 → clockwise orientation
= 0 → collinear
```

---

# 78. Orientation of Three Points

For:

```text
A
B
C
```

compute:

```text
cross(B-A, C-A)
```

That is:

```text
(xB-xA)(yC-yA)
-
(yB-yA)(xC-xA)
```

This is a core geometry primitive.

---

# 79. Triangle Area

Twice the signed area of triangle `ABC` is:

```text
cross(B-A, C-A)
```

Therefore:

```text
area
=
|cross| / 2
```

To avoid floating point, store:

```text
2 × area
```

as an integer.

---

# 80. Shoelace Formula

For polygon vertices:

```text
(x1,y1), ..., (xn,yn)
```

twice the signed area is:

```text
Σ (xi yi+1 - yi xi+1)
```

with indices wrapping:

```text
n+1 → 1
```

Then:

```text
area = |sum| / 2
```

---

# 81. Rectangle Basics

For an axis-aligned rectangle:

```text
width  = |x2-x1|
height = |y2-y1|
area   = width × height
```

For overlap of two axis-aligned rectangles:

```text
left   = max(left1, left2)
right  = min(right1, right2)

bottom = max(bottom1, bottom2)
top    = min(top1, top2)
```

If:

```text
right <= left
```

or:

```text
top <= bottom
```

there is no positive-area overlap.

---

# 82. Circle Basics

For center:

```text
(cx, cy)
```

and point:

```text
(x, y)
```

compare squared distance:

```text
(x-cx)^2 + (y-cy)^2
```

with:

```text
r²
```

Avoid:

```text
Math.sqrt(...)
```

when only a comparison is required.

---

# 83. Geometry Pattern Recognition

### Comparing distances

Use:

```text
squared distance
```

---

### Collinearity

Use:

```text
cross product = 0
```

---

### Orientation

Use:

```text
cross product sign
```

---

### Slopes in hash maps

Use:

```text
reduced (dy, dx)
```

not floating-point slopes.

---

### Polygon area

Use:

```text
shoelace formula
```

---

# 84. Overflow Handling

Overflow is one of the most common hidden bugs in integer-heavy LeetCode solutions.

Java:

```text
int → 32-bit signed
long → 64-bit signed
```

Arithmetic can silently overflow.

---

# 85. Integer Range

Java `int`:

```text
-2^31
to
2^31 - 1
```

approximately:

```text
-2.147 × 10^9
to
 2.147 × 10^9
```

Java `long`:

```text
-2^63
to
2^63 - 1
```

---

# 86. Promote Before Multiplication

This is dangerous:

```java
long x = a * b;
```

if both `a` and `b` are `int`.

The multiplication occurs as `int` first.

Use:

```java
long x = (long) a * b;
```

Now multiplication occurs in `long`.

---

# 87. Division Before Multiplication

For:

```text
lcm(a,b)
```

prefer:

```java
(a / gcd(a, b)) * b
```

rather than:

```java
a * b / gcd(a, b)
```

The first reduces the magnitude before multiplication.

This does not guarantee safety for every possible input, but it significantly reduces unnecessary overflow.

---

# 88. Avoid `mid = (l+r)/2`

For binary search:

```java
int mid = (l + r) / 2;
```

can overflow.

Prefer:

```java
int mid = l + (r - l) / 2;
```

---

# 89. Avoid `d*d <= n` When Needed

Instead of:

```java
for (long d = 2; d * d <= n; d++)
```

use:

```java
for (long d = 2; d <= n / d; d++)
```

This avoids multiplication overflow.

---

# 90. `Math.abs(Integer.MIN_VALUE)`

A Java edge case:

```java
Math.abs(Integer.MIN_VALUE)
```

still returns:

```text
Integer.MIN_VALUE
```

because the positive value:

```text
2^31
```

cannot be represented by `int`.

Similarly:

```java
Math.abs(Long.MIN_VALUE)
```

cannot be represented as a positive `long`.

Use a wider type or special handling if these values are possible.

---

# 91. `Math.multiplyExact`

Java provides checked arithmetic:

```java
Math.addExact(...)
Math.subtractExact(...)
Math.multiplyExact(...)
```

They throw:

```text
ArithmeticException
```

on overflow.

This is useful when correctness requires explicit overflow detection.

---

# 92. BigInteger

When exact integer values exceed `long`, use:

```java
java.math.BigInteger
```

Example:

```java
import java.math.BigInteger;

BigInteger a = new BigInteger("100000000000000000000");
BigInteger b = new BigInteger("200000000000000000000");

BigInteger c = a.multiply(b);
```

Common LeetCode solutions usually avoid `BigInteger` if the constraints allow fixed-width arithmetic, but it is important to know when it is necessary.

---

# 93. Overflow in Modular Multiplication

Even when the final answer is:

```text
(a × b) % MOD
```

the product:

```text
a × b
```

must be representable before `%` is applied.

For ordinary:

```text
MOD = 1e9+7
```

using `long` is safe because:

```text
(1e9+6)^2
≈ 1e18
```

which is below:

```text
Long.MAX_VALUE ≈ 9.22e18
```

For much larger moduli, a different multiplication method may be necessary.

---

# 94. Negative Modulo in Java

Java:

```java
-5 % 3
```

returns:

```text
-2
```

not:

```text
1
```

To normalize into:

```text
[0, mod)
```

use:

```java
((x % mod) + mod) % mod
```

or:

```java
Math.floorMod(x, mod)
```

---

# 95. `Math.floorMod`

For clean modular normalization:

```java
long normalized =
    Math.floorMod(x, mod);
```

This is particularly useful when negative coordinates or subtraction are involved.

---

# 96. Common Number-Theory Identities

Memorize:

```text
gcd(a,b) = gcd(b, a%b)

gcd(a,b) × lcm(a,b) = a×b

a^0 = 1

a^(x+y) = a^x × a^y

a^(2k) = (a^k)^2

C(n,r) = C(n,n-r)

C(n,r) = C(n-1,r-1)+C(n-1,r)

P(n,r) = n!/(n-r)!

P(n,r) = C(n,r)×r!
```

---

# 97. Common Modular Identities

```text
(a+b)%m
=
((a%m)+(b%m))%m

(a-b)%m
=
((a%m)-(b%m)+m)%m

(a*b)%m
=
((a%m)*(b%m))%m
```

provided the intermediate multiplication does not overflow.

---

# 98. Fast Power Recognition

If the problem asks for:

```text
x^n
```

and:

```text
n is large
```

think:

```text
binary exponentiation
```

If it asks:

```text
x^n mod M
```

think:

```text
modular exponentiation
```

---

# 99. Combination Recognition

If a problem asks:

```text
choose k positions
choose k elements
number of groups
number of ways to select
```

think:

```text
nCr
```

If order matters:

```text
nPr
```

---

# 100. Probability Recognition

### "At least one"

Think:

```text
1 - P(none)
```

### "Exactly k successes"

Think:

```text
C(n,k)p^k(1-p)^(n-k)
```

when trials are independent and identical.

### Expected number

Think:

```text
linearity of expectation
```

---

# 101. Matrix Exponentiation Recognition

Think matrix exponentiation when:

```text
recurrence
+
huge n
+
linear combination of previous states
```

For example:

```text
F(n)=3F(n-1)+2F(n-2)
```

with:

```text
n up to 10^18
```

is a strong matrix-exponentiation signal.

---

# 102. Geometry Recognition

### Three points

Think:

```text
cross product
```

### Distance comparison

Think:

```text
squared distance
```

### Equal slopes

Think:

```text
reduced dy/dx
```

### Polygon area

Think:

```text
shoelace
```

### Axis-aligned rectangle overlap

Think:

```text
max(left)
min(right)
max(bottom)
min(top)
```

---

# 103. GATE-Style Solved Questions

> These are **GATE-style practice questions**, not claimed as verbatim official GATE PYQs.

---

## Question 1 — Euclidean Algorithm

How many recursive reductions are performed by:

```text
gcd(48, 18)
```

using:

```text
gcd(a,b)=gcd(b,a%b)
```

until the second argument becomes zero?

### Solution

```text
gcd(48,18)
→ gcd(18,12)
→ gcd(12,6)
→ gcd(6,0)
```

There are:

```text
3 reductions
```

The answer is:

```text
6
```

### Final Answer

```text
gcd = 6
reductions = 3
```

---

## Question 2 — Sieve Complexity

Which asymptotic time complexity is associated with the standard Sieve of Eratosthenes for finding all primes up to `n`?

```text
A. O(log n)
B. O(sqrt(n))
C. O(n log log n)
D. O(n²)
```

### Solution

The total marking work of the standard sieve is:

```text
O(n log log n)
```

### Answer

```text
C
```

---

## Question 3 — Divisor Count

Find the number of positive divisors of:

```text
360
```

### Solution

Prime factorization:

```text
360 = 2^3 × 3^2 × 5^1
```

Number of divisors:

```text
(3+1)(2+1)(1+1)
=
4×3×2
=
24
```

### Answer

```text
24
```

---

## Question 4 — Modular Exponentiation

Compute:

```text
3^10 mod 7
```

### Solution

Repeated squaring:

```text
3² = 9 ≡ 2 mod 7

3⁴ ≡ 2² = 4 mod 7

3⁸ ≡ 4² = 16 ≡ 2 mod 7
```

Then:

```text
3^10
=
3^8 × 3²
```

so:

```text
3^10 mod 7
=
2 × 2
=
4
```

### Answer

```text
4
```

---

## Question 5 — Combinations

How many ways can 4 elements be selected from 10 distinct elements?

### Solution

```text
C(10,4)
=
10! / (4!6!)
```

Simplify:

```text
= (10×9×8×7)/(4×3×2×1)
= 210
```

### Answer

```text
210
```

---

## Question 6 — Geometry / Cross Product

Determine whether:

```text
A = (1,1)
B = (3,3)
C = (5,5)
```

are collinear.

### Solution

Compute:

```text
B-A = (2,2)
C-A = (4,4)
```

Cross product:

```text
2×4 - 2×4
= 0
```

Therefore the points are collinear.

### Answer

```text
Yes
```

---

# 104. LeetCode Roadmap

## Phase 1 — GCD / LCM / Divisibility

Master problems involving:

- GCD of arrays
- LCM
- fractions
- coprime values
- divisibility
- periodicity
- integer ratios
- Euclidean algorithm

Core tools:

```text
gcd
lcm
extended gcd
```

---

# 105. Phase 2 — Prime Numbers

Master:

- primality testing
- counting primes
- Sieve of Eratosthenes
- smallest prime factor
- prime factorization
- divisor enumeration
- divisor counts
- divisor sums

Core tools:

```text
sqrt(n)
sieve
SPF
factorization
```

---

# 106. Phase 3 — Modular Arithmetic

Master:

- modulo identities
- normalization
- modular multiplication
- fast exponentiation
- modular exponentiation
- modular inverse
- Fermat's Little Theorem
- extended-GCD inverse
- modular combinations

Core tools:

```text
MOD
modPow
inverse
factorials
inverse factorials
```

---

# 107. Phase 4 — Combinatorics

Master:

- combinations
- permutations
- Pascal triangle
- factorials
- repeated-element permutations
- binomial probability
- counting arrangements
- choose-and-arrange problems

Core recognition:

```text
selection → nCr
ordering → nPr
selection + ordering → nCr × r!
```

---

# 108. Phase 5 — Probability

Master:

- complementary probability
- independent events
- conditional probability
- expected value
- binomial probability
- counting-based probability
- probability DP

Do not memorize probability formulas without understanding the sample space.

---

# 109. Phase 6 — Matrix Exponentiation

Master:

- matrix multiplication
- identity matrix
- binary matrix exponentiation
- Fibonacci
- k-th order linear recurrence
- state-transition matrices

Primary signal:

```text
linear recurrence
+
huge n
```

---

# 110. Phase 7 — Geometry

Master:

- coordinate differences
- squared distance
- slope normalization
- collinearity
- cross product
- orientation
- triangle area
- shoelace formula
- rectangle intersection
- circle comparisons

Primary rule:

> Prefer exact integer comparisons over floating-point calculations.

---

# 111. Phase 8 — Overflow

Master:

- `int` vs `long`
- promotion before multiplication
- safe midpoint
- division before multiplication
- negative modulo
- `Math.abs` edge cases
- multiplication overflow
- modular multiplication
- `BigInteger`

---

# 112. Java Templates

## GCD

```java
static long gcd(long a, long b) {
    a = Math.abs(a);
    b = Math.abs(b);

    while (b != 0) {
        long t = a % b;
        a = b;
        b = t;
    }

    return a;
}
```

## LCM

```java
static long lcm(long a, long b) {
    if (a == 0 || b == 0) {
        return 0;
    }

    return Math.abs((a / gcd(a, b)) * b);
}
```

## Fast Power

```java
static long pow(long a, long e) {
    long result = 1;

    while (e > 0) {
        if ((e & 1) != 0) {
            result *= a;
        }

        a *= a;
        e >>= 1;
    }

    return result;
}
```

## Modular Power

```java
static long modPow(
        long a,
        long e,
        long mod) {

    a %= mod;
    long result = 1 % mod;

    while (e > 0) {
        if ((e & 1) != 0) {
            result = result * a % mod;
        }

        a = a * a % mod;
        e >>= 1;
    }

    return result;
}
```

## Prime Test

```java
static boolean isPrime(long n) {
    if (n < 2) {
        return false;
    }

    if (n % 2 == 0) {
        return n == 2;
    }

    for (long d = 3; d <= n / d; d += 2) {
        if (n % d == 0) {
            return false;
        }
    }

    return true;
}
```

## Sieve

```java
static boolean[] sieve(int n) {
    boolean[] prime = new boolean[n + 1];

    if (n < 2) {
        return prime;
    }

    java.util.Arrays.fill(prime, true);
    prime[0] = false;
    prime[1] = false;

    for (int p = 2; p <= n / p; p++) {
        if (!prime[p]) {
            continue;
        }

        for (int x = p * p; x <= n; x += p) {
            prime[x] = false;
        }
    }

    return prime;
}
```

## Squared Distance

```java
static long dist2(
        long x1, long y1,
        long x2, long y2) {

    long dx = x2 - x1;
    long dy = y2 - y1;

    return dx * dx + dy * dy;
}
```

## Cross Product

```java
static long cross(
        long ax, long ay,
        long bx, long by) {

    return ax * by - ay * bx;
}
```

---

# 113. Complexity Cheat Sheet

| Technique | Time | Space |
|---|---:|---:|
| GCD | `O(log min(a,b))` | `O(1)` iterative |
| Prime test | `O(sqrt(n))` | `O(1)` |
| Sieve | `O(n log log n)` | `O(n)` |
| Trial factorization | `O(sqrt(n))` worst case | `O(1)` excluding output |
| SPF preprocessing | approximately `O(n log log n)` for this sieve style | `O(n)` |
| Divisor enumeration | `O(sqrt(n))` | `O(sqrt(n))` output |
| Fast power | `O(log n)` | `O(1)` |
| Modular power | `O(log n)` | `O(1)` |
| Pascal triangle | `O(n²)` | `O(n²)` |
| Factorial precompute | `O(n)` | `O(n)` |
| nCr query with precomputation | `O(1)` | `O(n)` preprocessing |
| Matrix multiplication | `O(k³)` | `O(k²)` |
| Matrix exponentiation | `O(k³ log n)` | `O(k²)` |

---

# 114. Common Mistakes

## Mistake 1 — Using `%` incorrectly with negatives

Java:

```java
-2 % 5
```

is:

```text
-2
```

Normalize when a mathematical modulo in `[0,m)` is required.

---

## Mistake 2 — Computing factorials directly

Avoid:

```java
long fact = 1;
for (...) {
    fact *= i;
}
```

when `n` is large.

Factorials overflow extremely quickly.

Use:

```text
modular factorials
```

or:

```text
BigInteger
```

depending on the problem.

---

## Mistake 3 — Floating-point slopes

Avoid:

```java
double slope = (double) dy / dx;
```

for exact equality/hash-map problems.

Use:

```text
gcd normalization
```

instead.

---

## Mistake 4 — `sqrt` for comparisons

Instead of:

```java
Math.sqrt(dx*dx + dy*dy) <= r
```

compare:

```text
dx² + dy² <= r²
```

directly.

---

## Mistake 5 — Integer overflow before assignment

Bad:

```java
long result = a * b;
```

if `a` and `b` are `int`.

Better:

```java
long result = (long) a * b;
```

---

## Mistake 6 — Forgetting `1` is not prime

```text
1 → neither prime nor composite
```

---

## Mistake 7 — Applying Fermat's inverse to composite moduli

The formula:

```text
a^(mod-2) mod mod
```

requires the appropriate prime-modulus conditions.

For general modulus:

```text
extended Euclid
```

is the standard tool when the inverse exists.

---

## Mistake 8 — Building a sieve for one tiny query

For one primality check:

```text
O(sqrt(n))
```

is often simpler and more appropriate.

Use a sieve when there are many queries or a bounded range.

---

# 115. Advanced Problem Connections

## GCD + Arrays

```text
gcd(a1,a2,...,an)
```

can be folded left-to-right.

---

## GCD + Prefix/Suffix

For queries asking for the GCD after removing one element:

```text
prefixGcd[i]
suffixGcd[i]
```

can answer each removal in:

```text
O(1)
```

after:

```text
O(n)
```

preprocessing.

---

# 116. Prefix/Suffix GCD Pattern

For array:

```text
a[0..n-1]
```

define:

```text
prefix[i]
=
gcd(a[0],...,a[i])
```

and:

```text
suffix[i]
=
gcd(a[i],...,a[n-1])
```

If removing `a[i]`:

```text
gcd(
    prefix[i-1],
    suffix[i+1]
)
```

handles all remaining values.

Boundary cases use:

```text
gcd(0,x)=x
```

---

# 117. GCD + Fractions

For:

```text
numerator / denominator
```

reduce by:

```text
g = gcd(numerator, denominator)
```

then:

```text
numerator /= g
denominator /= g
```

Normalize the denominator to be positive.

---

# 118. LCM + Scheduling

LCM often appears when multiple periodic events repeat.

If events occur every:

```text
a
```

and:

```text
b
```

units, their simultaneous repetition occurs every:

```text
lcm(a,b)
```

subject to the problem's starting offsets.

---

# 119. Prime Factorization + GCD/LCM

If:

```text
a = Π p^α
b = Π p^β
```

then:

```text
gcd(a,b)
=
Π p^min(α,β)
```

and:

```text
lcm(a,b)
=
Π p^max(α,β)
```

This is useful when a problem gives prime-exponent structure directly.

---

# 120. Modular Arithmetic + DP

Many counting DP problems have:

```text
dp[state]
```

whose values grow exponentially.

If the problem asks:

```text
answer mod 1e9+7
```

apply modulo during transitions:

```java
dp[next] =
    (dp[next] + dp[current]) % MOD;
```

Do not wait until the very end if intermediate values can overflow.

---

# 121. Modular Arithmetic + Combinations

A common pattern:

```text
answer = C(n,r) mod MOD
```

with:

```text
n
```

large enough that direct factorials are impossible.

For prime `MOD` and manageable `n`, use:

```text
fact
invFact
```

precomputation.

---

# 122. Geometry + Hashing

When grouping points by lines or slopes:

```text
normalize(dx,dy)
```

then use the normalized pair as a hash key.

The same idea applies to:

- directions
- vectors
- rays
- equal slopes

Canonicalization is the key.

---

# 123. Geometry + Overflow

The cross product:

```text
ax*by - ay*bx
```

can overflow if coordinates are large.

If constraints approach the square root of `Long.MAX_VALUE`, `long` may not be sufficient.

Always inspect coordinate bounds before choosing the numeric type.

---

# 124. Number Theory + Binary Search

Some problems combine:

```text
divisibility
+
monotonicity
```

and become binary-search problems.

Examples include finding the smallest value satisfying a divisor/count condition.

Do not assume number theory requires a purely mathematical solution; it often supplies the predicate for another algorithm.

---

# 125. Number Theory + Hashing

Prime factorization can create canonical signatures.

Example:

```text
12 = 2² × 3
18 = 2 × 3²
```

Representing factor exponents lets you compare numbers by:

- equal prime structure
- anagrams of factors
- products
- divisibility relations

The exact signature depends on the problem.

---

# 126. Number Theory + Bit Manipulation

Fast exponentiation uses the same binary idea as bit manipulation:

```text
if ((exp & 1) != 0)
```

and:

```text
exp >>= 1
```

Therefore binary exponentiation is an important bridge between:

```text
Math / Number Theory
```

and:

```text
Bit Manipulation
```

---

# 127. What to Memorize vs Derive

## Memorize

```text
gcd(a,b)=gcd(b,a%b)

lcm=a/gcd(a,b)*b

prime test → sqrt(n)

sieve → start multiples at p²

number of divisors:
Π(ai+1)

fast power → binary exponentiation

C(n,r)=n!/(r!(n-r)!)

P(n,r)=n!/(n-r)!

Fermat inverse:
a^(p-2) mod p

distance → squared distance

orientation → cross product

polygon area → shoelace
```

## Derive

You should be able to derive:

- why Euclid works
- why sieve starts at `p²`
- why `sqrt(n)` is sufficient
- why fast exponentiation is logarithmic
- why Fermat gives the modular inverse
- why squared distance avoids `sqrt`
- why cross product detects orientation
- why factorial/inverse-factorial computes nCr

---

# 128. Final Pattern Recognition Table

| Problem clue | First technique to consider |
|---|---|
| Common divisor | GCD |
| Common multiple | LCM |
| Many prime queries | Sieve |
| Factor many bounded integers | SPF |
| Factor one integer | Trial division |
| Number of divisors | Prime exponents |
| Huge exponent | Fast power |
| Huge exponent + modulus | Modular exponentiation |
| Divide under modulus | Modular inverse |
| Choose elements | nCr |
| Arrange elements | nPr |
| At least one event | Complement |
| Expected count | Linearity of expectation |
| Huge linear recurrence | Matrix exponentiation |
| Compare point distances | Squared distance |
| Three-point direction | Cross product |
| Collinearity | Cross product = 0 |
| Equal slopes | Reduced `(dy,dx)` |
| Polygon area | Shoelace |
| Large integer arithmetic | `long` / `BigInteger` |
| Negative modular result | `floorMod` / normalization |

---

# 129. Final Mastery Checklist

## GCD / LCM

- [ ] Define GCD
- [ ] Euclidean algorithm
- [ ] Recursive and iterative GCD
- [ ] GCD of an array
- [ ] LCM
- [ ] GCD-LCM identity
- [ ] Extended Euclidean algorithm
- [ ] Bézout identity
- [ ] Coprime numbers

## Primes

- [ ] Prime definition
- [ ] `sqrt(n)` primality test
- [ ] Sieve of Eratosthenes
- [ ] Sieve complexity
- [ ] SPF
- [ ] Prime factorization
- [ ] Divisor enumeration
- [ ] Number of divisors
- [ ] Sum of divisors

## Modular Arithmetic

- [ ] Addition modulo
- [ ] Subtraction modulo
- [ ] Multiplication modulo
- [ ] Negative modulo
- [ ] Fast exponentiation
- [ ] Modular exponentiation
- [ ] Modular inverse
- [ ] Fermat's Little Theorem
- [ ] Extended-GCD inverse
- [ ] Modular division

## Combinatorics

- [ ] nCr
- [ ] nPr
- [ ] Pascal identity
- [ ] Factorial precomputation
- [ ] Inverse factorials
- [ ] Repeated-element permutations
- [ ] Counting interpretations

## Probability

- [ ] Sample space
- [ ] Complement
- [ ] Independence
- [ ] Conditional probability
- [ ] Expected value
- [ ] Linearity of expectation
- [ ] Binomial probability

## Matrix Exponentiation

- [ ] Matrix multiplication
- [ ] Identity matrix
- [ ] Binary matrix exponentiation
- [ ] Fibonacci
- [ ] Linear recurrences
- [ ] Complexity `O(k³ log n)`

## Geometry

- [ ] Coordinate differences
- [ ] Squared distance
- [ ] Slope normalization
- [ ] Collinearity
- [ ] Cross product
- [ ] Orientation
- [ ] Triangle area
- [ ] Shoelace formula
- [ ] Rectangle intersection
- [ ] Circle comparisons

## Overflow

- [ ] `int` range
- [ ] `long` range
- [ ] Cast before multiplication
- [ ] Safe midpoint
- [ ] Safe `d*d <= n`
- [ ] LCM overflow awareness
- [ ] Negative modulo
- [ ] `Math.abs` edge cases
- [ ] `Math.multiplyExact`
- [ ] `BigInteger`

---

# 130. Final Revision Sheet

Before an interview, these should be automatic:

```text
GCD:
gcd(a,b) = gcd(b,a%b)

LCM:
a/gcd(a,b)*b

Prime:
check d <= n/d

Sieve:
mark from p*p

Factorization:
repeated prime division

Divisor count:
Π(ai+1)

Fast power:
square base, halve exponent

Mod power:
same + % MOD after operations

Prime inverse:
a^(MOD-2) mod MOD

nCr:
fact[n] * invFact[r] * invFact[n-r]

nPr:
fact[n] * invFact[n-r]

At least one:
1 - P(none)

Expected value:
Σ xP(x)

Matrix power:
O(k³ log n)

Distance:
dx² + dy²

Collinear:
cross = 0

Orientation:
cross sign

Polygon area:
shoelace

Safe midpoint:
l + (r-l)/2

Safe multiplication:
(long)a*b

Negative modulo:
floorMod(x,m)
```

> **Core principle:**  
> **For LeetCode, Math / Number Theory is primarily a pattern-recognition toolkit. Know when a problem is asking for GCD, factorization, modular arithmetic, combinatorics, logarithmic exponentiation, recurrence acceleration, exact geometry, or overflow-safe arithmetic. The goal is not to perform more mathematics; it is to replace expensive computation with the right mathematical invariant or formula.**
