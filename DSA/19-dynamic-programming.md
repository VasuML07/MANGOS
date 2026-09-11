# 19. Dynamic Programming

> **Goal:** Master Dynamic Programming (DP) for FAANG/top product-company SDE and ML/AI engineer interviews, while covering the corresponding GATE theory, recurrence analysis, state design, optimization techniques, and reusable Java patterns.
>
> **Language:** Java  
> **Difficulty target:** Easy → Medium → Hard, with Medium dominating.
>
> **Importance:** This is one of the largest and highest-value DSA topics. The objective is not to memorize individual DP solutions. The objective is to recognize **state → transition → base case → computation order → optimization**.

---

# 1. What Is Dynamic Programming?

Dynamic Programming solves problems by breaking them into subproblems, solving each relevant subproblem once, and reusing the result.

DP is most useful when a problem has:

1. **Overlapping subproblems**
2. **Optimal substructure**

The two standard implementations are:

```text
Top-down DP  = recursion + memoization
Bottom-up DP = tabulation
```

---

# 2. DP Mental Model

Every DP problem should be reduced to:

```text
STATE
  ↓
TRANSITION
  ↓
BASE CASE
  ↓
COMPUTATION ORDER
  ↓
ANSWER
```

Then ask:

```text
Can the state be compressed?
```

This is more important than memorizing code.

---

# 3. Overlapping Subproblems

Consider Fibonacci:

```text
F(n) = F(n-1) + F(n-2)
```

Naive recursion:

```text
F(5)
├── F(4)
│   ├── F(3)
│   └── F(2)
└── F(3)
    ├── F(2)
    └── F(1)
```

`F(3)` and `F(2)` are repeatedly computed.

DP stores them.

---

# 4. Optimal Substructure

A problem has optimal substructure when an optimal solution can be constructed from optimal solutions of relevant smaller subproblems.

Examples:

- shortest paths
- knapsack
- edit distance
- minimum path sum
- house robber

The exact mathematical conditions depend on the problem.

---

# 5. Memoization vs Tabulation

## Memoization

Start from the original problem.

```text
Recursive solution
+
cache
```

Example:

```java
int solve(int n) {
    if (n <= 1) return n;

    if (dp[n] != -1) {
        return dp[n];
    }

    return dp[n] = solve(n - 1) + solve(n - 2);
}
```

### Advantages

- Natural transition from recursion.
- Computes only reached states.
- Often easier to derive.

### Disadvantages

- Recursion stack.
- Function-call overhead.
- Potential stack overflow for deep states.

---

## Tabulation

Build states iteratively.

```text
base cases
→ smaller states
→ larger states
```

Example:

```java
int[] dp = new int[n + 1];

dp[0] = 0;
dp[1] = 1;

for (int i = 2; i <= n; i++) {
    dp[i] = dp[i - 1] + dp[i - 2];
}
```

### Advantages

- No recursion stack.
- Explicit computation order.
- Usually easier to optimize space.

### Disadvantages

- May compute states that are never needed.
- Correct iteration order must be identified.

---

# 6. How to Derive a DP

Use this sequence.

## Step 1 — Define the state

Ask:

> What information completely describes the remaining problem?

Examples:

```text
dp[i]
dp[i][j]
dp[index][capacity]
dp[row][col]
dp[index][transactions]
```

---

## Step 2 — Define the meaning

Never write:

```text
dp[i] = ...
```

without being able to say what it means.

Examples:

```text
dp[i] = maximum money obtainable from houses 0..i
```

```text
dp[i][j] = minimum edits needed to convert
           first i characters into first j characters
```

---

## Step 3 — Find the choices

Example House Robber:

```text
rob current house
or
skip current house
```

---

## Step 4 — Write the transition

If:

```text
rob i → dp[i-2] + nums[i]
skip i → dp[i-1]
```

then:

```text
dp[i] = max(dp[i-1], dp[i-2] + nums[i])
```

---

## Step 5 — Base cases

Determine the smallest valid states.

---

## Step 6 — Determine computation order

If:

```text
dp[i] depends on dp[i-1] and dp[i-2]
```

compute from left to right.

If:

```text
dp[i][j] depends on dp[i-1][j]
and dp[i][j-1]
```

compute row/column in dependency order.

---

## Step 7 — Optimize

Ask:

```text
Does the transition need the entire previous table?
```

If only one/two previous rows or states are needed:

```text
compress the state
```

---

# 7. DP Complexity

Typical DP complexity:

```text
number of states × transitions per state
```

Example:

```text
n states
O(1) transition
```

gives:

```text
O(n)
```

Example:

```text
n² states
O(1) transition
```

gives:

```text
O(n²)
```

Example:

```text
n² states
O(n) transition
```

gives:

```text
O(n³)
```

---

# 8. 1D DP

Typical state:

```text
dp[i]
```

Common examples:

- Fibonacci
- Climbing Stairs
- House Robber
- Decode Ways
- Coin Change
- Min Cost Climbing Stairs

---

# 9. Fibonacci

## Recurrence

```text
F(0) = 0
F(1) = 1

F(n) = F(n-1) + F(n-2)
```

Naive recursion:

```text
O(2^n)
```

DP:

```text
O(n)
```

---

## 9.1 Tabulation

```java
public class Fibonacci {
    static int fib(int n) {
        if (n <= 1) return n;

        int[] dp = new int[n + 1];

        dp[0] = 0;
        dp[1] = 1;

        for (int i = 2; i <= n; i++) {
            dp[i] = dp[i - 1] + dp[i - 2];
        }

        return dp[n];
    }
}
```

---

## 9.2 O(1) Space

Only the previous two values are required.

```java
static int fib(int n) {
    if (n <= 1) return n;

    int prev2 = 0;
    int prev1 = 1;

    for (int i = 2; i <= n; i++) {
        int current = prev1 + prev2;
        prev2 = prev1;
        prev1 = current;
    }

    return prev1;
}
```

Complexity:

```text
Time: O(n)
Space: O(1)
```

---

# 10. Climbing Stairs

You can climb:

```text
1 or 2 steps
```

Find number of ways to reach step `n`.

State:

```text
dp[i] = number of ways to reach step i
```

Transition:

```text
dp[i] = dp[i-1] + dp[i-2]
```

Base:

```text
dp[0] = 1
dp[1] = 1
```

Why `dp[0] = 1`?

There is exactly one way to make zero steps:

```text
choose nothing
```

---

## Java

```java
public class ClimbingStairs {
    static int climbStairs(int n) {
        if (n <= 1) return 1;

        int prev2 = 1;
        int prev1 = 1;

        for (int i = 2; i <= n; i++) {
            int current = prev1 + prev2;
            prev2 = prev1;
            prev1 = current;
        }

        return prev1;
    }
}
```

---

# 11. House Robber

Given money in houses, adjacent houses cannot both be robbed.

State:

```text
dp[i] = maximum money using houses 0..i
```

At house `i`:

```text
skip → dp[i-1]
rob  → dp[i-2] + nums[i]
```

Therefore:

```text
dp[i] = max(dp[i-1], dp[i-2] + nums[i])
```

---

## Java

```java
public class HouseRobber {
    static int rob(int[] nums) {
        int prev2 = 0;
        int prev1 = 0;

        for (int money : nums) {
            int current =
                Math.max(prev1, prev2 + money);

            prev2 = prev1;
            prev1 = current;
        }

        return prev1;
    }
}
```

---

# 12. Decode Ways

A digit string represents letters:

```text
1 → A
2 → B
...
26 → Z
```

Find number of valid decodings.

State:

```text
dp[i] = number of ways to decode first i characters
```

At position `i`:

### One-digit choice

Valid if current digit is:

```text
1..9
```

Then:

```text
dp[i] += dp[i-1]
```

### Two-digit choice

Valid if the two-digit number is:

```text
10..26
```

Then:

```text
dp[i] += dp[i-2]
```

---

## Java

```java
public class DecodeWays {
    static int numDecodings(String s) {
        int n = s.length();

        if (n == 0 || s.charAt(0) == '0') {
            return 0;
        }

        int prev2 = 1;
        int prev1 = 1;

        for (int i = 1; i < n; i++) {
            int current = 0;

            if (s.charAt(i) != '0') {
                current += prev1;
            }

            int two =
                (s.charAt(i - 1) - '0') * 10
                + (s.charAt(i) - '0');

            if (two >= 10 && two <= 26) {
                current += prev2;
            }

            prev2 = prev1;
            prev1 = current;
        }

        return prev1;
    }
}
```

---

# 13. Coin Change

Given coin denominations and an amount, find the minimum number of coins needed.

Unlimited reuse is allowed.

State:

```text
dp[a] = minimum coins needed to make amount a
```

Transition:

```text
dp[a] = min(dp[a], dp[a-coin] + 1)
```

Base:

```text
dp[0] = 0
```

Initialize unreachable states with:

```text
amount + 1
```

or a sufficiently large sentinel.

---

## Java

```java
import java.util.*;

public class CoinChange {
    static int coinChange(int[] coins, int amount) {
        int INF = amount + 1;
        int[] dp = new int[amount + 1];

        Arrays.fill(dp, INF);
        dp[0] = 0;

        for (int a = 1; a <= amount; a++) {
            for (int coin : coins) {
                if (coin <= a) {
                    dp[a] = Math.min(
                        dp[a],
                        dp[a - coin] + 1
                    );
                }
            }
        }

        return dp[amount] == INF ? -1 : dp[amount];
    }
}
```

Complexity:

```text
O(amount × numberOfCoins)
```

---

# 14. Coin Change — Important Distinction

There are two classic versions.

## Minimum number of coins

```text
dp[amount] = minimum coins
```

## Number of combinations

```text
dp[amount] = number of ways
```

These are different DP states/objectives.

Also distinguish:

```text
combinations
```

from:

```text
permutations/order-sensitive ways
```

Loop ordering can change which one is counted.

---

# 15. Min Cost Climbing Stairs

Given the cost of each step, you can climb one or two steps.

State:

```text
dp[i] = minimum cost to reach step i
```

Transition:

```text
dp[i] = cost[i] + min(dp[i-1], dp[i-2])
```

Depending on the exact formulation, define whether `i` represents a stair or the top beyond the last stair.

Always align the state definition with the problem statement.

---

# 16. 2D DP

State generally looks like:

```text
dp[i][j]
```

Used when two dimensions matter.

Examples:

- Grid paths
- Unique Paths
- Minimum Path Sum
- Grid obstacles
- LCS
- Edit Distance

---

# 17. Unique Paths

An `m × n` grid allows movement:

```text
right
down
```

Find the number of paths from top-left to bottom-right.

State:

```text
dp[r][c] = number of ways to reach (r,c)
```

Transition:

```text
dp[r][c] =
    dp[r-1][c]
  + dp[r][c-1]
```

Base:

```text
dp[0][0] = 1
```

---

## Java

```java
public class UniquePaths {
    static int uniquePaths(int m, int n) {
        int[][] dp = new int[m][n];

        for (int r = 0; r < m; r++) {
            dp[r][0] = 1;
        }

        for (int c = 0; c < n; c++) {
            dp[0][c] = 1;
        }

        for (int r = 1; r < m; r++) {
            for (int c = 1; c < n; c++) {
                dp[r][c] =
                    dp[r - 1][c] +
                    dp[r][c - 1];
            }
        }

        return dp[m - 1][n - 1];
    }
}
```

Complexity:

```text
O(mn) time
O(mn) space
```

---

# 18. Unique Paths — 1D Optimization

Only the previous row is needed.

```java
static int uniquePaths(int m, int n) {
    int[] dp = new int[n];
    Arrays.fill(dp, 1);

    for (int r = 1; r < m; r++) {
        for (int c = 1; c < n; c++) {
            dp[c] += dp[c - 1];
        }
    }

    return dp[n - 1];
}
```

Space:

```text
O(n)
```

This is a **rolling-array/state-compression** technique.

---

# 19. Minimum Path Sum

Each cell has a cost. Move:

```text
right
down
```

Minimize total path cost.

State:

```text
dp[r][c] =
minimum cost to reach (r,c)
```

Transition:

```text
dp[r][c] =
grid[r][c] + min(
    dp[r-1][c],
    dp[r][c-1]
)
```

---

# 20. Grid Obstacles

If some cells are blocked:

```text
dp[r][c] = 0
```

for an obstacle.

Otherwise:

```text
dp[r][c] =
dp[r-1][c] + dp[r][c-1]
```

provided those states are valid.

The important insight is:

> The obstacle changes the transition availability, not the overall DP framework.

---

# 21. Knapsack DP

Knapsack is one of the most important DP families.

Main distinction:

```text
0/1 knapsack
→ each item at most once

unbounded knapsack
→ each item unlimited times
```

---

# 22. 0/1 Knapsack

Given:

```text
weight[i]
value[i]
capacity W
```

maximize value.

State:

```text
dp[i][w]
```

= maximum value using first `i` items with capacity `w`.

At item `i`:

```text
skip:
dp[i-1][w]

take:
value[i] + dp[i-1][w-weight[i]]
```

Therefore:

```text
dp[i][w] =
max(
    dp[i-1][w],
    value[i] + dp[i-1][w-weight[i]]
)
```

---

## 22.1 Java

```java
public class Knapsack01 {
    static int knapsack(int[] weight,
                        int[] value,
                        int W) {

        int n = weight.length;
        int[][] dp = new int[n + 1][W + 1];

        for (int i = 1; i <= n; i++) {
            for (int w = 0; w <= W; w++) {

                dp[i][w] = dp[i - 1][w];

                if (weight[i - 1] <= w) {
                    dp[i][w] = Math.max(
                        dp[i][w],
                        value[i - 1]
                        + dp[i - 1][w - weight[i - 1]]
                    );
                }
            }
        }

        return dp[n][W];
    }
}
```

Complexity:

```text
O(nW) time
O(nW) space
```

---

# 23. 0/1 Knapsack — 1D Optimization

Use:

```text
dp[w] = maximum value for capacity w
```

Iterate capacity **backward**:

```java
for (int i = 0; i < n; i++) {
    for (int w = W; w >= weight[i]; w--) {
        dp[w] = Math.max(
            dp[w],
            value[i] + dp[w - weight[i]]
        );
    }
}
```

### Why backward?

Backward iteration ensures `dp[w-weight[i]]` still represents the state **before using the current item**.

If you iterate forward, the same item can be used multiple times.

---

# 24. Unbounded Knapsack

Every item can be selected unlimited times.

The transition is similar:

```text
dp[w] =
max(dp[w],
    value[i] + dp[w-weight[i]])
```

but capacity is normally iterated **forward** when using a 1D formulation.

```java
for (int i = 0; i < n; i++) {
    for (int w = weight[i]; w <= W; w++) {
        dp[w] = Math.max(
            dp[w],
            value[i] + dp[w - weight[i]]
        );
    }
}
```

### Key interview distinction

```text
0/1 knapsack:
capacity loop ↓

unbounded knapsack:
capacity loop ↑
```

The ordering controls whether the current item can be reused.

---

# 25. Subset Sum

Given an array and target `S`, determine whether some subset sums to `S`.

State:

```text
dp[s] = whether sum s is achievable
```

Initialize:

```text
dp[0] = true
```

For each number `x`, iterate:

```text
s = S down to x
```

Transition:

```text
dp[s] |= dp[s-x]
```

Backward traversal prevents using the same element multiple times.

---

## Java

```java
public class SubsetSum {
    static boolean subsetSum(int[] nums, int target) {
        boolean[] dp = new boolean[target + 1];
        dp[0] = true;

        for (int x : nums) {
            for (int s = target; s >= x; s--) {
                dp[s] = dp[s] || dp[s - x];
            }
        }

        return dp[target];
    }
}
```

---

# 26. Partition Equal Subset Sum

Given an array, determine whether it can be divided into two subsets with equal sum.

Let:

```text
total = sum(nums)
```

If:

```text
total % 2 != 0
```

impossible.

Otherwise target:

```text
total / 2
```

Now solve:

```text
subset sum(target)
```

This is a classic reduction.

---

# 27. Target Sum

Assign either:

```text
+
```

or:

```text
-
```

to each number to reach target `T`.

Let:

```text
P = sum of positive-assigned numbers
N = sum of negative-assigned numbers
```

Then:

```text
P + N = total
P - N = T
```

Adding:

```text
2P = total + T
```

Therefore:

```text
P = (total + T) / 2
```

The problem can be transformed into counting subsets with that sum, when the derived target is a valid non-negative integer.

This is an important DP reduction.

---

# 28. Coin Change as Knapsack

Coin change is an unbounded-knapsack family because:

```text
each denomination can be used repeatedly
```

For minimum coins:

```text
minimization DP
```

For number of combinations:

```text
counting DP
```

The same item/reuse structure appears, but the state objective changes.

---

# 29. Subsequences DP

Important problems:

- Longest Increasing Subsequence
- Longest Common Subsequence
- Longest Common Substring
- Edit Distance
- Distinct Subsequences

These problems require careful state definitions.

---

# 30. Longest Increasing Subsequence

Given an array, find the length of the longest strictly increasing subsequence.

---

## 30.1 O(n²) DP

State:

```text
dp[i] =
length of LIS ending at i
```

Transition:

```text
if nums[j] < nums[i]:

dp[i] = max(dp[i], dp[j] + 1)
```

---

## Java

```java
import java.util.*;

public class LIS {
    static int lengthOfLIS(int[] nums) {
        int n = nums.length;
        int[] dp = new int[n];

        Arrays.fill(dp, 1);

        int answer = 0;

        for (int i = 0; i < n; i++) {
            for (int j = 0; j < i; j++) {
                if (nums[j] < nums[i]) {
                    dp[i] =
                        Math.max(dp[i], dp[j] + 1);
                }
            }

            answer = Math.max(answer, dp[i]);
        }

        return answer;
    }
}
```

Complexity:

```text
O(n²)
```

---

# 31. LIS O(n log n) Optimization

Maintain:

```text
tails[len-1]
```

as the smallest possible tail value of an increasing subsequence of that length.

For each value:

```text
binary search first tail >= value
replace it
```

The array does not necessarily contain the actual LIS.

It stores information sufficient to determine the LIS length.

---

## Java

```java
import java.util.*;

public class LISOptimized {
    static int lengthOfLIS(int[] nums) {
        int[] tails = new int[nums.length];
        int size = 0;

        for (int x : nums) {
            int left = 0;
            int right = size;

            while (left < right) {
                int mid = left + (right - left) / 2;

                if (tails[mid] < x) {
                    left = mid + 1;
                } else {
                    right = mid;
                }
            }

            tails[left] = x;

            if (left == size) {
                size++;
            }
        }

        return size;
    }
}
```

Complexity:

```text
O(n log n)
```

---

# 32. Longest Common Subsequence

Given strings `A` and `B`.

State:

```text
dp[i][j] =
LCS length of first i chars of A
and first j chars of B
```

If:

```text
A[i-1] == B[j-1]
```

then:

```text
dp[i][j] = dp[i-1][j-1] + 1
```

Otherwise:

```text
dp[i][j] =
max(
    dp[i-1][j],
    dp[i][j-1]
)
```

---

## Java

```java
public class LCS {
    static int longestCommonSubsequence(
            String a, String b) {

        int n = a.length();
        int m = b.length();

        int[][] dp = new int[n + 1][m + 1];

        for (int i = 1; i <= n; i++) {
            for (int j = 1; j <= m; j++) {

                if (a.charAt(i - 1) ==
                    b.charAt(j - 1)) {

                    dp[i][j] =
                        dp[i - 1][j - 1] + 1;

                } else {
                    dp[i][j] =
                        Math.max(
                            dp[i - 1][j],
                            dp[i][j - 1]
                        );
                }
            }
        }

        return dp[n][m];
    }
}
```

Complexity:

```text
O(nm) time
O(nm) space
```

---

# 33. Longest Common Substring

Unlike LCS, the matching characters must be **contiguous**.

State:

```text
dp[i][j] =
length of longest common substring
ending at A[i-1], B[j-1]
```

If characters match:

```text
dp[i][j] = dp[i-1][j-1] + 1
```

Otherwise:

```text
dp[i][j] = 0
```

Track a global maximum.

### Critical distinction

```text
LCS:
mismatch → max(top, left)

Longest common substring:
mismatch → 0
```

---

# 34. Edit Distance

Operations:

```text
insert
delete
replace
```

State:

```text
dp[i][j] =
minimum operations to convert
first i chars of A into first j chars of B
```

If characters match:

```text
dp[i][j] = dp[i-1][j-1]
```

Otherwise:

```text
dp[i][j] =
1 + min(
    dp[i-1][j],     // delete
    dp[i][j-1],     // insert
    dp[i-1][j-1]    // replace
)
```

---

## Java

```java
public class EditDistance {
    static int minDistance(String a, String b) {
        int n = a.length();
        int m = b.length();

        int[][] dp = new int[n + 1][m + 1];

        for (int i = 0; i <= n; i++) {
            dp[i][0] = i;
        }

        for (int j = 0; j <= m; j++) {
            dp[0][j] = j;
        }

        for (int i = 1; i <= n; i++) {
            for (int j = 1; j <= m; j++) {

                if (a.charAt(i - 1) ==
                    b.charAt(j - 1)) {

                    dp[i][j] =
                        dp[i - 1][j - 1];

                } else {

                    dp[i][j] = 1 + Math.min(
                        dp[i - 1][j],
                        Math.min(
                            dp[i][j - 1],
                            dp[i - 1][j - 1]
                        )
                    );
                }
            }
        }

        return dp[n][m];
    }
}
```

Complexity:

```text
O(nm)
```

---

# 35. Distinct Subsequences

Given strings `s` and `t`, count how many subsequences of `s` equal `t`.

State:

```text
dp[i][j] =
number of ways first i chars of s
can form first j chars of t
```

If:

```text
s[i-1] == t[j-1]
```

two choices exist:

```text
use s[i-1]
or
skip s[i-1]
```

Therefore:

```text
dp[i][j] =
dp[i-1][j-1] + dp[i-1][j]
```

If they differ:

```text
dp[i][j] = dp[i-1][j]
```

Base:

```text
dp[i][0] = 1
```

The empty string is a subsequence of every prefix.

---

# 36. String DP

Important problems:

- Palindromic Subsequences
- Palindromic Substrings
- Word Break
- Regex Matching
- Wildcard Matching
- Edit Distance
- Distinct Subsequences

---

# 37. Palindromic Substrings

A substring is contiguous.

Define:

```text
palindrome[i][j]
```

as whether:

```text
s[i..j]
```

is a palindrome.

Transition:

```text
palindrome[i][j] =
s[i] == s[j]
AND
(
    j - i <= 1
    OR palindrome[i+1][j-1]
)
```

Base:

```text
length 1 → true
length 2 → true if equal
```

Process by increasing substring length.

---

# 38. Palindromic Subsequences

A subsequence is not required to be contiguous.

A common state:

```text
dp[i][j] =
number of palindromic subsequences
in s[i..j]
```

If:

```text
s[i] == s[j]
```

a standard recurrence is:

```text
dp[i][j] =
dp[i+1][j]
+
dp[i][j-1]
+
1
```

for the appropriate counting definition.

If:

```text
s[i] != s[j]
```

then:

```text
dp[i][j] =
dp[i+1][j]
+
dp[i][j-1]
-
dp[i+1][j-1]
```

The exact recurrence depends on whether the problem asks for distinct or non-distinct palindromic subsequences.

### Important

Do not confuse:

```text
palindromic substring
```

with:

```text
palindromic subsequence
```

---

# 39. Word Break

Given a string and dictionary, determine whether the string can be segmented into dictionary words.

State:

```text
dp[i] =
whether prefix s[0..i-1] can be segmented
```

Transition:

```text
dp[i] = true
if there exists j < i such that:

dp[j] == true
AND
s[j..i-1] is in dictionary
```

---

## Java

```java
import java.util.*;

public class WordBreak {
    static boolean wordBreak(
            String s,
            List<String> wordDict) {

        Set<String> dict =
            new HashSet<>(wordDict);

        int n = s.length();
        boolean[] dp = new boolean[n + 1];

        dp[0] = true;

        for (int i = 1; i <= n; i++) {

            for (int j = 0; j < i; j++) {

                if (dp[j] &&
                    dict.contains(s.substring(j, i))) {

                    dp[i] = true;
                    break;
                }
            }
        }

        return dp[n];
    }
}
```

Typical complexity with substring/hash assumptions:

```text
O(n²)
```

with implementation-dependent string hashing/copying costs.

A trie can be useful when dictionary lookup/character traversal is important.

---

# 40. Wildcard Matching

Pattern symbols:

```text
?
```

matches one character.

```text
*
```

matches zero or more characters.

State:

```text
dp[i][j] =
whether first i chars of string
match first j chars of pattern
```

If:

```text
p[j-1] == s[i-1]
```

or:

```text
p[j-1] == '?'
```

then:

```text
dp[i][j] = dp[i-1][j-1]
```

If:

```text
p[j-1] == '*'
```

then:

```text
dp[i][j] =
dp[i][j-1]       // '*' matches empty
OR
dp[i-1][j]       // '*' consumes one character
```

---

# 41. Regex Matching

For simplified regular-expression matching with:

```text
.
*
```

the state is similar but the meaning of `*` differs from wildcard matching.

For `*`:

```text
zero occurrences:
dp[i][j-2]

one or more:
if current characters match:
dp[i-1][j]
```

### Critical distinction

Do not copy the wildcard `*` transition into regex matching.

The two `*` symbols have different semantics.

---

# 42. Interval DP

Interval DP solves problems where the state represents a contiguous interval:

```text
dp[l][r]
```

Typical structure:

```text
choose a split k between l and r
→ solve left interval
→ solve right interval
→ combine
```

Examples:

- Matrix Chain Multiplication
- Burst Balloons
- Palindrome Partitioning
- Optimal BST-style problems

---

# 43. Matrix Chain Multiplication

Given matrices:

```text
A1 × A2 × ... × An
```

find the minimum number of scalar multiplications.

Matrix multiplication is associative, but different parenthesizations have different costs.

---

## 43.1 State

```text
dp[i][j] =
minimum multiplication cost
for matrices i..j
```

Transition:

```text
dp[i][j] =
min over k:
    dp[i][k]
  + dp[k+1][j]
  + cost of multiplying the two resulting matrices
```

If dimensions are stored in:

```text
p[0], p[1], ..., p[n]
```

then:

```text
cost =
p[i-1] × p[k] × p[j]
```

for the standard indexing convention.

---

## Java

```java
public class MatrixChainMultiplication {
    static long matrixChain(int[] p) {
        int n = p.length - 1;
        long[][] dp = new long[n][n];

        for (int len = 2; len <= n; len++) {

            for (int i = 0;
                 i + len - 1 < n;
                 i++) {

                int j = i + len - 1;
                dp[i][j] = Long.MAX_VALUE;

                for (int k = i; k < j; k++) {

                    long cost =
                        dp[i][k]
                        + dp[k + 1][j]
                        + (long) p[i]
                          * p[k + 1]
                          * p[j + 1];

                    dp[i][j] =
                        Math.min(dp[i][j], cost);
                }
            }
        }

        return dp[0][n - 1];
    }
}
```

Complexity:

```text
O(n³) time
O(n²) space
```

---

# 44. Burst Balloons

Given balloon values, burst all balloons to maximize coins.

The difficulty is that choosing the first balloon creates complicated dependencies.

A powerful transformation is:

> Think about the **last** balloon burst in an interval.

Add virtual boundary balloons:

```text
1
```

at both ends.

State:

```text
dp[l][r] =
maximum coins from bursting balloons
strictly between l and r
```

If `k` is the last balloon burst:

```text
dp[l][r] =
max(
    dp[l][k]
    + dp[k][r]
    + nums[l] * nums[k] * nums[r]
)
```

over all:

```text
l < k < r
```

This is classic interval DP.

---

# 45. Palindrome Partitioning DP

Backtracking can enumerate all palindrome partitions.

DP can instead compute the **minimum number of cuts**.

State:

```text
dp[i] =
minimum cuts needed for prefix ending at i
```

or use a 2D palindrome table:

```text
palindrome[i][j]
```

Then update the cut DP whenever:

```text
s[i..j]
```

is a palindrome.

This demonstrates that the same problem can have both:

```text
backtracking
```

and:

```text
DP
```

formulations depending on the required output.

---

# 46. Tree DP

Tree DP uses states attached to nodes.

A typical pattern:

```text
solve(node)
→ solve(left)
→ solve(right)
→ combine
```

The tree has no cycles when rooted, so child subproblems naturally form a dependency structure.

---

# 47. House Robber III

Each tree node contains money.

Adjacent parent/child nodes cannot both be robbed.

Use two states per node:

```text
rob[node]  = maximum if this node is robbed
skip[node] = maximum if this node is skipped
```

If node is robbed:

```text
rob[node] =
node.val
+ skip[left]
+ skip[right]
```

If skipped:

```text
skip[node] =
max(rob[left], skip[left])
+
max(rob[right], skip[right])
```

---

## Java

```java
public class HouseRobberIII {

    static class TreeNode {
        int val;
        TreeNode left;
        TreeNode right;

        TreeNode(int val) {
            this.val = val;
        }
    }

    static int rob(TreeNode root) {
        int[] result = solve(root);
        return Math.max(result[0], result[1]);
    }

    // [robThis, skipThis]
    static int[] solve(TreeNode node) {
        if (node == null) {
            return new int[]{0, 0};
        }

        int[] left = solve(node.left);
        int[] right = solve(node.right);

        int robThis =
            node.val + left[1] + right[1];

        int skipThis =
            Math.max(left[0], left[1])
            + Math.max(right[0], right[1]);

        return new int[]{robThis, skipThis};
    }
}
```

---

# 48. Maximum Independent Set on a Tree

House Robber III is a weighted version of the **maximum independent set on a tree** pattern.

An independent set is a set of vertices where no adjacent vertices are both selected.

States:

```text
take[node]
skip[node]
```

This pattern generalizes to many tree optimization problems.

---

# 49. Subtree DP

Typical subtree state:

```text
dp[node][state]
```

where the state describes what happens at the current node.

Examples:

- selected/not selected
- color/state assigned
- number of selected nodes
- best value under a condition
- matching states

Generic:

```java
int[] solve(Node node) {
    if (node == null) {
        return baseState;
    }

    int[] left = solve(node.left);
    int[] right = solve(node.right);

    return combine(node, left, right);
}
```

---

# 50. State-Machine DP

Many problems involve a small number of recurring states.

Stock trading is the canonical example.

Instead of thinking:

```text
buy/sell transactions
```

think:

```text
What state am I in today?
```

Examples:

```text
holding
not holding
cooldown
transactions remaining
```

---

# 51. Stock Buy/Sell — Unlimited Transactions

State:

```text
hold[i] = maximum profit after day i while holding stock
cash[i] = maximum profit after day i while not holding
```

Transitions:

```text
hold =
max(
    previous hold,
    previous cash - price
)

cash =
max(
    previous cash,
    previous hold + price
)
```

---

## Java

```java
public class StockUnlimited {
    static int maxProfit(int[] prices) {
        int hold = -prices[0];
        int cash = 0;

        for (int i = 1; i < prices.length; i++) {
            int prevHold = hold;
            int prevCash = cash;

            hold = Math.max(
                prevHold,
                prevCash - prices[i]
            );

            cash = Math.max(
                prevCash,
                prevHold + prices[i]
            );
        }

        return cash;
    }
}
```

Space:

```text
O(1)
```

---

# 52. Stock With Cooldown

After selling, the next day may be a cooldown day.

States can be:

```text
hold
sold
rest
```

A common transition formulation:

```text
hold[i] =
max(
    hold[i-1],
    rest[i-1] - price[i]
)

sold[i] =
hold[i-1] + price[i]

rest[i] =
max(
    rest[i-1],
    sold[i-1]
)
```

The exact state definitions should be fixed before coding.

---

# 53. Stock With Transaction Fee

If every completed transaction incurs a fee:

```text
cash =
max(
    cash,
    hold + price - fee
)
```

and:

```text
hold =
max(
    hold,
    cash - price
)
```

This remains a small state-machine DP.

---

# 54. Stock With Transaction Limits

If at most `k` transactions are allowed:

```text
dp[transactions][holding]
```

or:

```text
dp[day][transactions][holding]
```

can represent the state.

A typical state:

```text
dp[t][0] = max profit with t transactions, not holding
dp[t][1] = max profit with t transactions, holding
```

When selling:

```text
transaction count changes
```

depending on whether the formulation counts a transaction at buy or sell.

Be consistent.

---

# 55. State-Machine Recognition

When a problem says:

- buy/sell
- hold/not hold
- cooldown
- limited transactions
- fee
- states/modes
- transitions between conditions

think:

```text
small finite state machine
+
DP over time
```

---

# 56. Bitmask DP

Bitmask DP is used when the state includes a subset of a small set of objects.

If there are `n` objects:

```text
mask
```

has:

```text
2^n
```

possible subsets.

Bit `i` indicates whether object `i` is included.

---

# 57. Bit Operations

Test bit:

```java
(mask & (1 << i)) != 0
```

Set bit:

```java
mask | (1 << i)
```

Clear bit:

```java
mask & ~(1 << i)
```

Toggle:

```java
mask ^ (1 << i)
```

Count bits:

```java
Integer.bitCount(mask)
```

---

# 58. TSP-Style Bitmask DP

Traveling Salesman Problem:

> Visit every city exactly once and return to the start with minimum total cost.

State:

```text
dp[mask][i]
```

= minimum cost to visit the cities in `mask` and end at city `i`.

Transition:

```text
dp[mask | (1<<j)][j]
=
min(
    dp[mask][i] + dist[i][j]
)
```

where `j` is an unvisited city.

Complexity:

```text
O(2^n × n²)
```

space:

```text
O(2^n × n)
```

This is exponential, but dramatically better than brute-force `O(n!)` enumeration for moderate `n`.

---

# 59. Assignment Problems

Suppose `n` workers must be assigned to `n` tasks.

State:

```text
dp[mask]
```

can represent the minimum cost after assigning tasks represented by `mask`.

If:

```text
k = Integer.bitCount(mask)
```

then worker `k` is the next worker to assign.

Try every unassigned task:

```text
newMask = mask | (1 << task)
```

Transition:

```text
dp[newMask] =
min(
    dp[newMask],
    dp[mask] + cost[k][task]
)
```

Complexity:

```text
O(n × 2^n)
```

states/transitions up to the standard bound.

---

# 60. Subset-State DP

Use bitmask DP when:

```text
n is small
AND
the relevant state is a subset
```

Common clues:

- visit each item once
- assign unique tasks
- select a subset
- visited/unvisited set
- partition a small set
- pair/group elements

Typical feasible `n` is often around:

```text
15–22
```

depending on the transition count, language, and time limit.

Do not treat this as a hard universal limit.

---

# 61. DP Optimization

Main optimization techniques:

1. Space optimization
2. State compression
3. Rolling arrays
4. Memoization → tabulation
5. Reducing transition complexity
6. Exploiting monotonicity/structure when applicable

---

# 62. Space Optimization

If:

```text
dp[i]
```

only depends on:

```text
dp[i-1], dp[i-2]
```

store two variables.

If:

```text
dp[i][j]
```

only depends on the previous row and current row:

```text
O(nm) → O(m)
```

space.

---

# 63. Rolling Arrays

For 2D DP:

```text
previous row
current row
```

instead of storing the entire matrix.

Example:

```java
int[] prev = new int[n];
int[] curr = new int[n];

for (int r = 0; r < m; r++) {
    // compute curr
    int[] temp = prev;
    prev = curr;
    curr = temp;
}
```

Often a single row is enough if the transition is carefully ordered.

---

# 64. In-Place 2D Compression

Example:

```text
dp[c]
```

represents the current row.

For a transition:

```text
dp[c] = dp[c] + dp[c-1]
```

the values have the correct meaning if columns are processed left-to-right.

This is why **iteration direction is part of the DP proof**, not just a coding detail.

---

# 65. Memoization → Tabulation

Start with:

```text
recursive state
```

Then:

1. Memoize repeated states.
2. Identify dependencies.
3. Determine a topological/computation order.
4. Replace recursion with loops.
5. Compress state if possible.

This is a practical method for deriving bottom-up DP.

---

# 66. State Compression

A full state may be:

```text
dp[i][j][k]
```

but perhaps `k` only depends on a small number of previous states.

If older states are never referenced again:

```text
remove them
```

The goal is:

```text
store only information required by future transitions
```

---

# 67. DP Iteration Direction

This is a major interview/GATE concept.

### 0/1 Knapsack

```text
capacity ↓
```

because an item can be used once.

### Unbounded Knapsack

```text
capacity ↑
```

because an item can be reused.

### Subset Sum

```text
sum ↓
```

because each number is used once.

### Grid DP

Usually:

```text
top-left → bottom-right
```

because dependencies come from earlier cells.

---

# 68. DP as a DAG

A useful theoretical interpretation:

> Many DP problems can be viewed as finding values on a Directed Acyclic Graph of states.

Each state is a node.

A transition is a directed edge.

If dependencies form a DAG, states can be evaluated in topological order.

This explains why:

```text
memoization
```

and:

```text
tabulation
```

produce the same recurrence result.

---

# 69. DP and Shortest/Longest Paths in DAGs

For a DAG:

```text
dp[v] =
best value reaching v
```

can be computed in topological order.

This connects:

```text
graph DP
```

with ordinary DP.

The key requirement is that dependencies do not create cycles.

---

# 70. Common DP State Types

## Prefix DP

```text
dp[i]
```

Examples:

```text
House Robber
Decode Ways
Word Break
```

---

## Grid DP

```text
dp[r][c]
```

Examples:

```text
Unique Paths
Minimum Path Sum
```

---

## Two-String DP

```text
dp[i][j]
```

Examples:

```text
LCS
Edit Distance
Distinct Subsequences
```

---

## Knapsack DP

```text
dp[item][capacity]
```

or compressed:

```text
dp[capacity]
```

---

## Interval DP

```text
dp[l][r]
```

Examples:

```text
MCM
Burst Balloons
Palindrome problems
```

---

## Tree DP

```text
dp[node][state]
```

Examples:

```text
House Robber III
Maximum Independent Set on Tree
```

---

## State Machine

```text
dp[day][state]
```

Examples:

```text
stocks
cooldown
fees
transaction limits
```

---

## Bitmask DP

```text
dp[mask][last]
```

or:

```text
dp[mask]
```

Examples:

```text
TSP
assignment
subset state
```

---

# 71. DP Failure Modes

## Mistake 1 — State does not contain enough information

If future decisions depend on something not represented in the state, the recurrence is invalid.

---

## Mistake 2 — State contains unnecessary information

This increases complexity.

Example:

```text
dp[day][full history]
```

may be reducible to:

```text
dp[day][holding]
```

---

## Mistake 3 — Wrong base case

The recurrence can be correct while the final result is wrong because initialization is wrong.

---

## Mistake 4 — Wrong loop direction

Especially common in:

```text
0/1 knapsack
subset sum
unbounded knapsack
```

---

## Mistake 5 — Confusing subsequence and substring

```text
subsequence → gaps allowed
substring → contiguous
```

---

## Mistake 6 — Counting combinations vs permutations

Loop order matters.

---

## Mistake 7 — Integer overflow

Counting DP may become very large.

Use:

```java
long
```

when required by constraints.

---

# 72. DP vs Greedy

A useful question:

> Can the best local decision be proven safe?

If yes:

```text
Greedy
```

If no and subproblems overlap:

```text
DP
```

Example:

```text
Fractional Knapsack → Greedy
0/1 Knapsack → DP
```

---

# 73. DP vs Backtracking

Backtracking:

```text
explores choices
```

DP:

```text
merges equivalent states
```

If many different decision sequences reach the same state:

```text
memoization can eliminate repeated work
```

This is the fundamental bridge between the two.

---

# 74. DP Problem-Solving Workflow

Use this exact process in interviews:

```text
1. Start with brute force.
2. Identify repeated subproblems.
3. Define the state.
4. Write the recurrence.
5. Establish base cases.
6. Memoize.
7. Analyze states × transitions.
8. Convert to tabulation if useful.
9. Optimize space.
10. Test edge cases.
```

---

# 75. Edge Cases

Always test:

```text
n = 0
n = 1
empty string
empty grid
single row
single column
all obstacles
zero target
target unreachable
duplicate values
negative values where allowed
very large values
```

For stock problems:

```text
prices length = 1
monotonically increasing
monotonically decreasing
```

For string DP:

```text
empty pattern
empty target
one-character strings
repeated characters
```

---

# 76. GATE Theory — Recurrences

A recurrence describes an algorithm or mathematical sequence in terms of smaller instances.

Examples:

```text
T(n) = T(n-1) + O(1)
```

gives:

```text
O(n)
```

and:

```text
T(n) = 2T(n-1) + O(1)
```

gives:

```text
O(2^n)
```

DP can reduce exponential recursive computation to polynomial time when the number of distinct states is polynomial.

---

# 77. GATE Theory — Memoization Complexity

Suppose there are:

```text
S
```

distinct states.

Each state has:

```text
K
```

possible transitions.

Then:

```text
Time = O(S × K)
```

assuming O(1) transition work.

Space:

```text
O(S)
```

plus recursion stack for top-down DP.

---

# 78. GATE Theory — Fibonacci

Naive recursion:

```text
T(n) = T(n-1) + T(n-2) + O(1)
```

has exponential growth.

Memoized/tabulated Fibonacci has:

```text
n + 1
```

states and O(1) work per state:

```text
O(n)
```

Space can be:

```text
O(n)
```

or:

```text
O(1)
```

with two-value compression.

---

# 79. GATE Theory — 0/1 Knapsack

Standard DP:

```text
O(nW)
```

time and:

```text
O(nW)
```

space.

Space can be reduced to:

```text
O(W)
```

because each row only depends on the previous row.

Important caveat:

> `O(nW)` is pseudo-polynomial, not polynomial in the bit-length of `W`.

---

# 80. GATE Theory — LCS

For strings of lengths `n` and `m`:

```text
O(nm)
```

time using standard DP.

Space:

```text
O(nm)
```

or:

```text
O(min(n,m))
```

for length-only computation with row compression.

---

# 81. GATE Theory — Edit Distance

Standard Levenshtein edit distance:

```text
O(nm)
```

time.

The state represents:

```text
minimum edits for prefixes
```

with transitions:

```text
insert
delete
replace
```

---

# 82. GATE Theory — Matrix Chain Multiplication

For `n` matrices:

```text
O(n³)
```

time.

Reason:

```text
O(n²)
```

interval states, and:

```text
O(n)
```

possible split positions per interval.

Therefore:

```text
O(n² × n) = O(n³)
```

---

# 83. GATE Theory — Bitmask DP

For TSP with `n` cities:

```text
2^n
```

subsets and:

```text
n
```

possible ending cities.

Each state may consider:

```text
O(n)
```

next cities.

Therefore:

```text
O(2^n × n²)
```

time.

---

# 84. GATE-Style Solved Questions

> These are **GATE-style practice questions**, not claimed as verbatim official GATE PYQs.

---

## Question 1 — Fibonacci DP

A naive recursive Fibonacci algorithm computes:

```text
F(n) = F(n-1) + F(n-2)
```

A memoized version stores every computed `F(k)` for `0 <= k <= n`.

What are the time and auxiliary-space complexities?

### Solution

There are:

```text
n + 1
```

distinct states.

Each state performs O(1) arithmetic and at most two memoized lookups.

Therefore:

```text
Time = O(n)
```

The memoization array stores:

```text
O(n)
```

values.

The recursion stack can also reach:

```text
O(n)
```

### Answer

```text
Time: O(n)
Space: O(n)
```

---

## Question 2 — 0/1 Knapsack Loop Direction

A 1D 0/1 knapsack implementation updates:

```java
for (int w = weight[i]; w <= W; w++) {
    dp[w] = Math.max(
        dp[w],
        value[i] + dp[w - weight[i]]
    );
}
```

Why can this produce an incorrect 0/1 knapsack solution?

### Solution

The loop moves forward.

After updating:

```text
dp[w - weight[i]]
```

using item `i`, a later iteration may use that newly updated state again.

Thus item `i` can be counted multiple times.

That corresponds to an **unbounded** interpretation.

For 0/1 knapsack, use:

```java
for (int w = W; w >= weight[i]; w--)
```

so the previous-item state is used.

### Answer

Forward iteration can reuse the same item multiple times; 0/1 knapsack requires backward capacity iteration.

---

## Question 3 — LCS

Let:

```text
A = "ABCBDAB"
B = "BDCABA"
```

Find the LCS length.

### Solution

A valid LCS is:

```text
BCBA
```

which has length:

```text
4
```

Another valid LCS may exist, but only the length matters.

The standard DP uses:

```text
if A[i-1] == B[j-1]:
    dp[i][j] = dp[i-1][j-1] + 1
else:
    dp[i][j] = max(dp[i-1][j], dp[i][j-1])
```

### Answer

```text
4
```

---

## Question 4 — Grid DP

Consider:

```text
1  3  1
1  5  1
4  2  1
```

Movement is only right or down. Find the minimum path sum.

### Solution

Start:

```text
dp[0][0] = 1
```

First row:

```text
1
1+3 = 4
4+1 = 5
```

First column:

```text
1
1+1 = 2
2+4 = 6
```

Remaining cells:

```text
dp[1][1] = 5 + min(4,2) = 7

dp[1][2] = 1 + min(5,7) = 6

dp[2][1] = 2 + min(2,7) = 4

dp[2][2] = 1 + min(6,4) = 5
```

### Answer

```text
5
```

Path:

```text
1 → 1 → 1 → 1 → 1
```

---

## Question 5 — Edit Distance

Find the minimum number of insertions, deletions, or replacements required to convert:

```text
"horse"
```

to:

```text
"ros"
```

### Solution

One optimal sequence:

```text
horse
→ rorse     replace h with r
→ rose      delete r? 
```

A cleaner sequence is:

```text
horse
→ hose      delete r
→ rose      replace h with r
→ ros       delete e
```

Total:

```text
3
```

The standard DP confirms:

```text
dp[5][3] = 3
```

### Answer

```text
3
```

---

## Question 6 — Bitmask DP

A TSP-style DP uses:

```text
dp[mask][i]
```

for `n` cities.

How many states are there, and what is the standard transition complexity?

### Solution

There are:

```text
2^n
```

possible masks.

For each mask, there can be:

```text
n
```

possible ending cities.

Therefore states:

```text
O(n × 2^n)
```

Each state can consider up to:

```text
O(n)
```

next cities.

Total:

```text
O(n² × 2^n)
```

### Answer

```text
States: O(n × 2^n)
Time:   O(n² × 2^n)
```

---

# 85. LeetCode Roadmap

## Phase 1 — Basic 1D DP

Master:

1. Fibonacci Number
2. Climbing Stairs
3. Min Cost Climbing Stairs
4. House Robber
5. House Robber II
6. Decode Ways
7. Maximum Product Subarray
8. Perfect Squares
9. Integer Break

Goal:

```text
dp[i]
+
base cases
+
constant-size transition
```

---

# 86. Phase 2 — Knapsack

Master:

1. 0/1 Knapsack variants
2. Partition Equal Subset Sum
3. Target Sum
4. Coin Change
5. Coin Change II
6. Combination Sum IV
7. Ones and Zeroes
8. Last Stone Weight II
9. Unbounded knapsack variants

Focus:

```text
0/1 vs unbounded
count vs optimize
sum vs capacity
loop direction
```

---

# 87. Phase 3 — Grid DP

Master:

1. Unique Paths
2. Unique Paths II
3. Minimum Path Sum
4. Triangle
5. Dungeon Game
6. Maximal Square
7. Minimum Falling Path Sum
8. Cherry Pickup variants

Focus:

```text
dp[r][c]
movement constraints
obstacles
min/max/count
space compression
```

---

# 88. Phase 4 — Subsequences and Strings

Master:

1. Longest Increasing Subsequence
2. Longest Common Subsequence
3. Longest Common Substring variants
4. Edit Distance
5. Distinct Subsequences
6. Word Break
7. Palindromic Substrings
8. Longest Palindromic Subsequence
9. Interleaving String
10. Wildcard Matching
11. Regular Expression Matching

---

# 89. Phase 5 — Interval DP

Master:

1. Matrix Chain Multiplication
2. Burst Balloons
3. Palindrome Partitioning
4. Strange Printer
5. Minimum Cost to Cut a Stick
6. Stone Game interval variants

Core structure:

```text
dp[l][r]
+
split k
```

---

# 90. Phase 6 — Tree DP

Master:

1. House Robber III
2. Binary Tree Maximum Path Sum
3. Maximum Independent Set on Tree
4. Diameter-style tree DP
5. Subtree state problems
6. Tree matching variants

Focus:

```text
return multiple states from child
→ combine at parent
```

---

# 91. Phase 7 — State-Machine DP

Master:

1. Best Time to Buy and Sell Stock
2. Stock II
3. Stock III
4. Stock IV
5. Stock with Cooldown
6. Stock with Transaction Fee
7. State-based scheduling problems

Focus:

```text
state definitions
+
state transitions
+
transaction accounting
```

---

# 92. Phase 8 — Bitmask DP

Master:

1. Traveling Salesman Problem
2. Assignment Problem
3. Minimum Hamiltonian Path
4. Small-set scheduling
5. Subset-state optimization
6. Partitioning small sets

Focus:

```text
mask
+
last/current item
+
transition to unused items
```

---

# 93. Problem Recognition Table

| Problem clue | DP pattern |
|---|---|
| Ways to reach position | 1D DP |
| Minimum cost along sequence | 1D DP |
| Adjacent selection restriction | House Robber style |
| Decode string | Prefix DP |
| Minimum coins | Unbounded knapsack |
| Exact subset sum | 0/1 knapsack |
| Equal partition | Subset sum |
| `+/-` target | Target Sum reduction |
| Grid paths | 2D/grid DP |
| Two strings | `dp[i][j]` |
| Longest common sequence | LCS |
| Contiguous common match | Longest common substring |
| Convert one string to another | Edit distance |
| Count ways to form target subsequence | Distinct subsequences |
| Dictionary segmentation | Word Break |
| `dp[l][r]` | Interval DP |
| Split interval at `k` | Interval DP |
| Tree node has selected/not selected | Tree DP |
| Buy/hold/sell/cooldown | State-machine DP |
| Small subset of objects | Bitmask DP |

---

# 94. DP Pattern: “Take or Skip”

This is one of the most reusable patterns.

At an item:

```text
take
OR
skip
```

Examples:

```text
House Robber
0/1 Knapsack
Subset Sum
Target Sum variants
```

Generic:

```java
dp[state] = combine(
    takeTransition,
    skipTransition
);
```

---

# 95. DP Pattern: “Prefix”

Use when the problem asks about prefixes:

```text
first i characters
first i elements
amount i
position i
```

Examples:

```text
Decode Ways
Word Break
House Robber
Coin Change
```

---

# 96. DP Pattern: “Two Prefixes”

Use:

```text
dp[i][j]
```

when comparing two sequences.

Examples:

```text
LCS
Edit Distance
Distinct Subsequences
```

The first dimension usually represents a prefix of the first string, and the second dimension a prefix of the second.

---

# 97. DP Pattern: “Interval”

Use:

```text
dp[l][r]
```

when the problem asks about a contiguous range and splitting that range creates independent subproblems.

Examples:

```text
MCM
Burst Balloons
Palindrome partitioning
```

---

# 98. DP Pattern: “Node + State”

Use:

```text
dp[node][state]
```

when each tree node has a local condition.

Examples:

```text
rob / skip
selected / not selected
color/state
```

---

# 99. DP Pattern: “Time + State”

Use:

```text
dp[day][state]
```

when a system evolves over time.

Examples:

```text
stock trading
cooldown
fees
transaction limits
```

---

# 100. DP Pattern: “Mask + State”

Use:

```text
dp[mask][last]
```

when the problem needs:

```text
which elements have been used
+
where/current state
```

Examples:

```text
TSP
Hamiltonian path
subset assignment
```

---

# 101. Counting DP vs Optimization DP

DP objectives commonly include:

### Counting

```text
number of ways
```

Use:

```text
+
```

or another counting operation.

### Minimization

```text
minimum cost
minimum operations
```

Use:

```text
min
```

### Maximization

```text
maximum value
maximum length
```

Use:

```text
max
```

### Boolean feasibility

```text
possible or impossible
```

Use:

```text
OR
```

Recognizing the objective often makes the transition obvious.

---

# 102. DP State Design Examples

## House Robber

```text
dp[i] = max money from first i houses
```

## Coin Change

```text
dp[a] = minimum coins for amount a
```

## LCS

```text
dp[i][j] = LCS of first i and first j chars
```

## Edit Distance

```text
dp[i][j] = minimum edits between two prefixes
```

## MCM

```text
dp[l][r] = minimum cost for matrix interval
```

## Tree DP

```text
dp[node][state] = best subtree result under state
```

## TSP

```text
dp[mask][last] = minimum cost for visited set ending at last
```

---

# 103. How to Check a DP State

A valid state should satisfy:

### Sufficiency

It contains everything needed for future decisions.

### Uniqueness

A state represents one well-defined subproblem.

### Reusability

Different paths reaching the same state should have the same remaining problem.

### Acyclic dependency

For ordinary bottom-up DP, dependencies must have a valid computation order.

---

# 104. State Equivalence

Two recursion paths can share one DP state if:

```text
their future possibilities and objective from that point are identical.
```

This is the deep reason memoization works.

Example:

If two different ways reach:

```text
(index = 7, remaining = 10)
```

and the remaining problem is identical, both should use the same memoized result.

---

# 105. When DP Does Not Work Directly

DP may be unsuitable when:

- the state space is enormous
- there is no manageable state representation
- subproblems do not overlap
- the problem requires an exponential subset state for large `n`
- a simpler greedy/graph/math solution exists
- transition computation itself is too expensive

The goal is not to force DP onto every optimization problem.

---

# 106. Advanced DP Connections

DP combines naturally with:

```text
Binary Search
Greedy
Graphs
Trees
Bitmasking
Prefix sums
Monotonic structures
Trie
Hashing
```

Examples:

```text
LIS = DP + binary search
Word Break = DP + hashing/trie
Tree optimization = tree traversal + DP
TSP = graph + bitmask DP
Stock = state machine + DP
```

---

# 107. Interview Template

When asked to solve a DP problem, explain in this order:

```text
1. Brute-force choices
2. Repeated subproblem
3. State definition
4. Transition
5. Base cases
6. Evaluation order
7. Complexity
8. Space optimization
```

Example:

> “Let `dp[i]` be the maximum value achievable using the first `i` elements. At element `i`, either skip it, giving `dp[i-1]`, or take it, giving the appropriate previous state plus its value. Therefore the recurrence is ...”

This communicates reasoning rather than memorization.

---

# 108. Final Complexity Summary

| DP family | Typical state | Typical time |
|---|---|---:|
| Fibonacci | `dp[i]` | O(n) |
| House Robber | `dp[i]` | O(n) |
| Decode Ways | `dp[i]` | O(n) |
| Coin Change | `dp[amount]` | O(nA) |
| Grid | `dp[r][c]` | O(RC) |
| 0/1 Knapsack | `dp[i][w]` | O(nW) |
| LIS basic | `dp[i]` | O(n²) |
| LIS optimized | tails | O(n log n) |
| LCS | `dp[i][j]` | O(nm) |
| Edit Distance | `dp[i][j]` | O(nm) |
| Distinct Subsequences | `dp[i][j]` | O(nm) |
| Word Break | `dp[i]` | O(n²) typical |
| Interval DP | `dp[l][r]` | Often O(n³) |
| Tree DP | `dp[node][state]` | Often O(n) |
| Stock state DP | `dp[day][state]` | O(nk) or O(n) |
| TSP bitmask | `dp[mask][last]` | O(2^n n²) |

For knapsack, `W`/`A` denote numeric capacity/amount, so these are pseudo-polynomial where applicable.

---

# 109. Mastery Checklist

## Fundamentals

- [ ] Define overlapping subproblems.
- [ ] Define optimal substructure.
- [ ] Derive a state.
- [ ] Explain state meaning in one sentence.
- [ ] Write transitions.
- [ ] Identify base cases.
- [ ] Determine computation order.
- [ ] Analyze states × transitions.
- [ ] Convert memoization to tabulation.
- [ ] Optimize space.

## 1D DP

- [ ] Fibonacci
- [ ] Climbing Stairs
- [ ] House Robber
- [ ] Decode Ways
- [ ] Coin Change
- [ ] Min Cost Climbing Stairs

## Grid DP

- [ ] Unique Paths
- [ ] Grid paths
- [ ] Minimum Path Sum
- [ ] Obstacles
- [ ] 1D row compression

## Knapsack

- [ ] 0/1 Knapsack
- [ ] Unbounded Knapsack
- [ ] Subset Sum
- [ ] Partition Equal Subset Sum
- [ ] Target Sum
- [ ] Coin Change
- [ ] Coin Change II
- [ ] Loop-direction reasoning

## Subsequences

- [ ] LIS O(n²)
- [ ] LIS O(n log n)
- [ ] LCS
- [ ] Longest Common Substring
- [ ] Edit Distance
- [ ] Distinct Subsequences

## String DP

- [ ] Palindromic Substrings
- [ ] Palindromic Subsequences
- [ ] Word Break
- [ ] Wildcard Matching
- [ ] Regex Matching

## Interval DP

- [ ] Matrix Chain Multiplication
- [ ] Burst Balloons
- [ ] Palindrome Partitioning
- [ ] Split-at-k recurrence

## Tree DP

- [ ] House Robber III
- [ ] Maximum Independent Set style
- [ ] Subtree state DP
- [ ] Multiple-state returns

## State Machine

- [ ] Stock buy/sell
- [ ] Unlimited transactions
- [ ] Cooldown
- [ ] Transaction fees
- [ ] Transaction limits

## Bitmask DP

- [ ] Bit operations
- [ ] TSP
- [ ] Assignment
- [ ] Subset-state DP

## Optimization

- [ ] Memoization
- [ ] Tabulation
- [ ] Rolling arrays
- [ ] State compression
- [ ] Space optimization
- [ ] Recognize reducible dimensions

---

# 110. Final Revision Sheet

## DP Core

```text
State
→ Transition
→ Base case
→ Order
→ Answer
→ Compress
```

## Fibonacci

```text
dp[i] = dp[i-1] + dp[i-2]
```

## House Robber

```text
dp[i] = max(
    dp[i-1],
    dp[i-2] + nums[i]
)
```

## Coin Change

```text
dp[a] = min(
    dp[a],
    dp[a-coin] + 1
)
```

## Unique Paths

```text
dp[r][c] =
dp[r-1][c] + dp[r][c-1]
```

## 0/1 Knapsack

```text
take / skip
capacity ↓
```

## Unbounded Knapsack

```text
reuse allowed
capacity ↑
```

## Subset Sum

```text
boolean dp
sum ↓
```

## Partition Equal Subset

```text
total even?
→ target = total/2
→ subset sum
```

## Target Sum

```text
P = (total + target) / 2
→ subset-count DP
```

## LIS

```text
dp[i] = LIS ending at i
```

or:

```text
tails + binary search
```

## LCS

```text
match → diagonal + 1
mismatch → max(top,left)
```

## Longest Common Substring

```text
match → diagonal + 1
mismatch → 0
```

## Edit Distance

```text
match → diagonal
mismatch → 1 + min(
    insert,
    delete,
    replace
)
```

## Distinct Subsequences

```text
match →
use + skip

mismatch →
skip
```

## Palindrome

```text
s[l] == s[r]
AND
inside is palindrome
```

## Word Break

```text
dp[i] =
exists j:
dp[j] && word(j,i)
```

## Interval DP

```text
dp[l][r]
→ try every split k
→ combine left + right + current cost
```

## Tree DP

```text
child states
→ combine at parent
```

## Stock DP

```text
day
+
state:
holding / not holding / cooldown / transactions
```

## Bitmask DP

```text
mask = used subset
dp[mask][last]
```

---

# 111. One-Page DP Mental Model

```text
                         DYNAMIC PROGRAMMING
                                  |
             ┌────────────────────┼────────────────────┐
             |                    |                    |
           1D DP               2D DP               Specialized
             |                    |                    |
       dp[i] / prefix        dp[i][j]            ┌─────┼─────┐
             |                    |               |     |     |
       Fibonacci             Grid DP          Interval Tree State
       Stairs                LCS              DP       DP   Machine
       Robber                Edit Distance     |       |      |
       Decode                Strings          MCM    Robber  Stocks
       Coin Change           Subsequences     Burst  III
                                               Balloons
             |
          Knapsack
             |
      ┌──────┴────────┐
      |               |
    0/1            Unbounded
      |               |
  take/skip        reuse
  capacity ↓       capacity ↑
      |
  Subset Sum
  Partition
  Target Sum

                 BITMASK DP
                      |
                mask + state
                      |
               TSP / Assignment
```

---

# 112. The Five Questions to Ask on Every DP Problem

Before coding, answer these five questions:

### 1. What does `dp[...]` mean?

If you cannot answer this precisely, do not code yet.

### 2. What choices are available?

For example:

```text
take / skip
insert / delete / replace
move right / down
buy / sell / hold
split at k
choose next unvisited item
```

### 3. What is the transition?

Write it mathematically first.

### 4. What are the base cases?

Define the smallest states.

### 5. Can the state be compressed?

After the correct DP works, inspect dependencies and reduce memory.

---

# 113. Final Mastery Standard

You have mastered **Dynamic Programming** when you can:

1. Recognize overlapping subproblems.
2. Explain optimal substructure.
3. Convert brute-force recursion into memoization.
4. Define a precise DP state.
5. Write a correct transition.
6. Establish correct base cases.
7. Determine computation order.
8. Analyze states × transitions.
9. Implement top-down and bottom-up versions.
10. Optimize 1D DP to O(1) when possible.
11. Compress 2D DP to O(n) when dependencies permit.
12. Solve 0/1 and unbounded knapsack without confusing loop directions.
13. Reduce partition and target-sum problems to subset-sum DP.
14. Distinguish subsequence from substring.
15. Solve LCS and edit distance.
16. Recognize interval DP.
17. Recognize tree DP.
18. Model stock problems as state machines.
19. Use bitmask DP for small-set state spaces.
20. Explain why the DP state is sufficient.
21. Identify when a DP state is too large.
22. Distinguish counting, minimization, maximization, and feasibility DP.
23. Derive complexity from the number of states and transitions.
24. Recognize when DP should be replaced by greedy, graph algorithms, binary search, or another technique.
25. Solve medium DP problems consistently and approach hard DP problems systematically.

> **Core principle:**  
> **DP is not memorizing recurrences. It is identifying a compact state that makes repeated subproblems identical, then computing that state efficiently.**
