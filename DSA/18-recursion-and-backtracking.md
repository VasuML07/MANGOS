# 18. Recursion & Backtracking

> **Goal:** Master recursion and backtracking for FAANG/top product-company SDE and ML/AI interviews, while covering the corresponding GATE theory and reusable problem-solving patterns.
>
> **Language:** Java  
> **Difficulty target:** Easy → Medium → Hard, with Medium dominating.

---

# 1. Recursion Fundamentals

Recursion is a technique where a function solves a problem by calling itself on a smaller instance of the same problem.

Every recursive solution needs:

1. **Base case**
2. **Recursive case**
3. **Progress toward the base case**

Generic structure:

```java
void solve(State state) {
    if (baseCase(state)) {
        return;
    }

    makeChoice(state);
    solve(smallerState(state));
    undoChoice(state);
}
```

The `undoChoice` step becomes especially important in backtracking.

---

# 2. Base Case

The base case stops recursion.

Example:

```java
static void print(int n) {
    if (n == 0) {
        return;
    }

    System.out.println(n);
    print(n - 1);
}
```

Without:

```java
if (n == 0)
```

the recursion does not terminate.

### Common base cases

```text
n == 0
n == 1
index == array.length
left > right
remaining == 0
board is complete
path has required size
```

---

# 3. Recursive Case

The recursive case reduces the problem.

Example:

```java
static int factorial(int n) {
    if (n <= 1) return 1;

    return n * factorial(n - 1);
}
```

Mathematically:

```text
factorial(n) = n × factorial(n - 1)
factorial(1) = 1
```

---

# 4. Recursion Call Stack

Consider:

```java
factorial(4)
```

Expansion:

```text
factorial(4)
    → 4 × factorial(3)
             → 3 × factorial(2)
                      → 2 × factorial(1)
                               → 1
```

Unwinding:

```text
factorial(1) = 1
factorial(2) = 2
factorial(3) = 6
factorial(4) = 24
```

The recursive calls are stored on the **call stack**.

---

# 5. Recursion Tree

A recursion tree represents recursive calls as nodes.

For:

```java
T(n) = 2T(n - 1) + O(1)
```

the tree looks like:

```text
                 f(n)
              /       \
          f(n-1)     f(n-1)
          /   \       /   \
     f(n-2) f(n-2) f(n-2) f(n-2)
             ...
```

At depth `k`:

```text
2^k
```

calls may exist.

Depth:

```text
n
```

Therefore:

```text
T(n) = O(2^n)
```

---

# 6. Recursion Tree for Subsets

For every element, we have two choices:

```text
include
exclude
```

For `[a,b,c]`:

```text
                         []
                    /          \
                 [a]            []
                /   \          /   \
             [a,b]  [a]      [b]    []
              / \    / \      / \    / \
         [a,b,c] [a,b] [a,c] [a] [b,c] [b] [c] []
```

Number of subsets:

```text
2^n
```

Number of leaf choices:

```text
2^n
```

If copying each subset costs O(n), total output-sensitive work is:

```text
O(n × 2^n)
```

---

# 7. Recursion Tree for Permutations

At each level, choose one unused element.

For `n` distinct elements:

```text
n choices
(n - 1) choices
(n - 2) choices
...
1 choice
```

Number of leaves:

```text
n!
```

If each permutation requires O(n) output construction:

```text
O(n × n!)
```

---

# 8. Recursion vs Backtracking

### Recursion

A function calls itself.

### Backtracking

Backtracking is recursion plus:

```text
choose
→ explore
→ undo
```

Generic pattern:

```java
void backtrack(State state) {
    if (isComplete(state)) {
        record(state);
        return;
    }

    for (Choice choice : choices(state)) {
        if (!valid(choice)) continue;

        apply(choice);
        backtrack(state);
        undo(choice);
    }
}
```

The undo operation restores the state so another branch can be explored.

---

# 9. Backtracking State

Before writing code, identify:

```text
What decisions have already been made?
What choices are available?
What makes a choice invalid?
What state must be restored?
What constitutes a complete solution?
```

Typical state:

```text
index
path
used[]
remaining target
board
row/column
visited
constraints
```

---

# 10. The Universal Backtracking Template

```java
void backtrack(...) {

    if (baseCase) {
        result.add(...);
        return;
    }

    for (...) {

        if (invalid) {
            continue;
        }

        choose(...);

        backtrack(...);

        undo(...);
    }
}
```

### Mental model

```text
Decision
   ↓
Choice
   ↓
Constraint check
   ↓
Apply
   ↓
Recurse
   ↓
Undo
   ↓
Next choice
```

---

# 11. Subsets

Given:

```text
nums = [1, 2, 3]
```

all subsets are:

```text
[]
[1]
[2]
[3]
[1,2]
[1,3]
[2,3]
[1,2,3]
```

Count:

```text
2^n
```

---

## 11.1 Include/Exclude Method

At each index:

```text
include nums[index]
OR
exclude nums[index]
```

### Java

```java
import java.util.*;

public class Subsets {
    static List<List<Integer>> subsets(int[] nums) {
        List<List<Integer>> result = new ArrayList<>();
        backtrack(0, nums, new ArrayList<>(), result);
        return result;
    }

    static void backtrack(int index,
                          int[] nums,
                          List<Integer> path,
                          List<List<Integer>> result) {

        if (index == nums.length) {
            result.add(new ArrayList<>(path));
            return;
        }

        // Include
        path.add(nums[index]);
        backtrack(index + 1, nums, path, result);
        path.remove(path.size() - 1);

        // Exclude
        backtrack(index + 1, nums, path, result);
    }
}
```

### Complexity

Number of subsets:

```text
2^n
```

Copying each subset:

```text
O(n)
```

Total:

```text
O(n × 2^n)
```

---

# 12. Subsets With Duplicates

For:

```text
[1, 2, 2]
```

sort first:

```text
[1, 2, 2]
```

At the same recursion level, skip duplicates:

```java
if (i > start && nums[i] == nums[i - 1]) {
    continue;
}
```

### Critical distinction

Do **not** skip every duplicate globally.

You skip duplicates:

```text
at the same depth
```

because choosing the same value as the first choice of two sibling branches would create duplicate subsets.

---

# 13. Subsequences

A subsequence maintains the original relative order but does not need to be contiguous.

For:

```text
[1,2,3]
```

examples:

```text
[]
[1]
[2]
[3]
[1,2]
[1,3]
[2,3]
[1,2,3]
```

This is structurally the same include/exclude tree as subsets.

---

# 14. Subsequences vs Subarrays

| Property | Subsequence | Subarray |
|---|---|---|
| Must be contiguous? | No | Yes |
| Order preserved? | Yes | Yes |
| Number of possibilities | Up to 2^n | O(n²) |
| Typical technique | Recursion / DP | Sliding window / prefix / DP |

---

# 15. Permutations

Given:

```text
[1,2,3]
```

permutations:

```text
123
132
213
231
312
321
```

Count:

```text
3! = 6
```

---

## 15.1 Used Array Method

```java
import java.util.*;

public class Permutations {
    static List<List<Integer>> permute(int[] nums) {
        List<List<Integer>> result = new ArrayList<>();
        boolean[] used = new boolean[nums.length];

        backtrack(nums, used, new ArrayList<>(), result);
        return result;
    }

    static void backtrack(int[] nums,
                          boolean[] used,
                          List<Integer> path,
                          List<List<Integer>> result) {

        if (path.size() == nums.length) {
            result.add(new ArrayList<>(path));
            return;
        }

        for (int i = 0; i < nums.length; i++) {
            if (used[i]) continue;

            used[i] = true;
            path.add(nums[i]);

            backtrack(nums, used, path, result);

            path.remove(path.size() - 1);
            used[i] = false;
        }
    }
}
```

### Complexity

```text
O(n × n!)
```

for output generation.

---

# 16. Permutations With Duplicates

Sort:

```text
[1,1,2]
```

Then:

```java
if (i > 0 &&
    nums[i] == nums[i - 1] &&
    !used[i - 1]) {
    continue;
}
```

Why `!used[i - 1]`?

The duplicate is allowed when the previous identical value is already part of the current permutation path.

The restriction prevents equivalent sibling branches.

---

# 17. Combination Sum

Given:

```text
candidates = [2,3,6,7]
target = 7
```

solutions:

```text
[2,2,3]
[7]
```

A number can be reused.

---

## 17.1 Core Pattern

At index `i`:

```text
choose candidates[i]
→ remain unchanged in index
```

or:

```text
skip candidates[i]
→ move to i + 1
```

---

## 17.2 Java

```java
import java.util.*;

public class CombinationSum {
    static List<List<Integer>> combinationSum(
            int[] candidates, int target) {

        List<List<Integer>> result = new ArrayList<>();

        Arrays.sort(candidates);

        backtrack(0, target, candidates,
                  new ArrayList<>(), result);

        return result;
    }

    static void backtrack(int start,
                          int remaining,
                          int[] candidates,
                          List<Integer> path,
                          List<List<Integer>> result) {

        if (remaining == 0) {
            result.add(new ArrayList<>(path));
            return;
        }

        for (int i = start; i < candidates.length; i++) {
            if (candidates[i] > remaining) break;

            path.add(candidates[i]);

            // i, not i + 1: reuse allowed
            backtrack(i, remaining - candidates[i],
                      candidates, path, result);

            path.remove(path.size() - 1);
        }
    }
}
```

---

# 18. Combination Sum II

Difference:

```text
Each element can be used once.
```

Typical duplicate handling:

```java
if (i > start && candidates[i] == candidates[i - 1]) {
    continue;
}
```

Recursive call:

```java
backtrack(i + 1, ...);
```

### Key distinction

```text
Combination Sum:
backtrack(i, ...)
```

```text
Combination Sum II:
backtrack(i + 1, ...)
```

---

# 19. N-Queens

## 19.1 Problem

Place `n` queens on an `n × n` chessboard such that no two queens attack each other.

Queens cannot share:

```text
row
column
main diagonal
anti-diagonal
```

---

# 20. N-Queens Backtracking

Place exactly one queen in each row.

At row `r`, try every column.

A position `(r,c)` is safe if:

```text
column c is unused
main diagonal r-c is unused
anti-diagonal r+c is unused
```

---

## 20.1 Java

```java
import java.util.*;

public class NQueens {
    static List<List<String>> solveNQueens(int n) {
        List<List<String>> result = new ArrayList<>();

        char[][] board = new char[n][n];

        for (char[] row : board) {
            Arrays.fill(row, '.');
        }

        boolean[] cols = new boolean[n];
        boolean[] diag1 = new boolean[2 * n - 1];
        boolean[] diag2 = new boolean[2 * n - 1];

        backtrack(0, board, cols, diag1, diag2, result);

        return result;
    }

    static void backtrack(int row,
                          char[][] board,
                          boolean[] cols,
                          boolean[] diag1,
                          boolean[] diag2,
                          List<List<String>> result) {

        if (row == board.length) {
            List<String> solution = new ArrayList<>();

            for (char[] r : board) {
                solution.add(new String(r));
            }

            result.add(solution);
            return;
        }

        int n = board.length;

        for (int col = 0; col < n; col++) {
            int d1 = row - col + n - 1;
            int d2 = row + col;

            if (cols[col] || diag1[d1] || diag2[d2]) {
                continue;
            }

            board[row][col] = 'Q';
            cols[col] = true;
            diag1[d1] = true;
            diag2[d2] = true;

            backtrack(row + 1, board,
                      cols, diag1, diag2, result);

            board[row][col] = '.';
            cols[col] = false;
            diag1[d1] = false;
            diag2[d2] = false;
        }
    }
}
```

### Why diagonal formulas work

Main diagonal:

```text
r - c
```

Anti-diagonal:

```text
r + c
```

Since `r-c` can be negative, shift it:

```text
r - c + n - 1
```

---

# 21. N-Queens Complexity

The search has factorial-like growth.

A common upper-bound description is:

```text
O(n!)
```

with additional board/output work.

The exact practical behavior depends heavily on pruning.

Space:

```text
O(n²)
```

if storing the board, plus recursion/constraint structures.

---

# 22. Sudoku

## 22.1 Problem

Fill a `9 × 9` board so every:

- row
- column
- `3 × 3` box

contains digits:

```text
1 ... 9
```

without repetition.

This is a classic **constraint satisfaction problem**.

---

# 23. Sudoku Backtracking

For each empty cell:

```text
try digit 1..9
→ check row
→ check column
→ check box
→ place
→ recurse
→ undo if failure
```

---

## 23.1 Java

```java
public class SudokuSolver {

    static boolean solve(char[][] board) {
        for (int row = 0; row < 9; row++) {
            for (int col = 0; col < 9; col++) {

                if (board[row][col] != '.') {
                    continue;
                }

                for (char digit = '1'; digit <= '9'; digit++) {

                    if (!isValid(board, row, col, digit)) {
                        continue;
                    }

                    board[row][col] = digit;

                    if (solve(board)) {
                        return true;
                    }

                    board[row][col] = '.';
                }

                return false;
            }
        }

        return true;
    }

    static boolean isValid(char[][] board,
                            int row,
                            int col,
                            char digit) {

        for (int i = 0; i < 9; i++) {
            if (board[row][i] == digit) return false;
            if (board[i][col] == digit) return false;

            int boxRow = 3 * (row / 3) + i / 3;
            int boxCol = 3 * (col / 3) + i % 3;

            if (board[boxRow][boxCol] == digit) {
                return false;
            }
        }

        return true;
    }
}
```

---

# 24. Sudoku Optimization

Checking every row/column/box repeatedly is simple but can be optimized.

Maintain:

```text
rowMask[9]
colMask[9]
boxMask[9]
```

For digit `d`:

```text
bit = 1 << d
```

A digit is available when:

```text
(rowMask[r] & bit) == 0
&&
(colMask[c] & bit) == 0
&&
(boxMask[b] & bit) == 0
```

This reduces constraint checking to constant time per candidate.

---

# 25. Constraint Satisfaction Problems

A CSP consists of:

1. **Variables**
2. **Domains**
3. **Constraints**

Example: Sudoku

```text
Variables:
empty cells

Domain:
1..9

Constraints:
row uniqueness
column uniqueness
box uniqueness
```

N-Queens:

```text
Variables:
queen position for each row

Domain:
columns

Constraints:
no same column
no same diagonal
```

---

# 26. CSP Backtracking Template

```text
Select variable
↓
Try a value from its domain
↓
Check constraints
↓
Assign value
↓
Recurse
↓
If failure:
    undo assignment
↓
Try next value
```

---

# 27. Variable Ordering Heuristics

For difficult CSPs, choosing **which variable to assign next** matters.

## Minimum Remaining Values (MRV)

Choose the variable with the fewest legal values.

This is also called:

```text
most constrained variable
```

Example:

If:

```text
Cell A → {1,2,3,4}
Cell B → {7}
Cell C → {2,3}
```

choose:

```text
B
```

first.

---

# 28. Degree Heuristic

If multiple variables have similarly small domains, choose the variable involved in the largest number of constraints.

Reason:

```text
A highly connected variable can constrain many
remaining variables.
```

---

# 29. Least Constraining Value

When choosing a value, prefer the value that eliminates the fewest choices for neighboring variables.

This is called:

```text
LCV
```

It is useful in general CSP search.

---

# 30. Word Search

## Problem

Given a grid of characters and a word, determine whether the word exists by moving:

```text
up
down
left
right
```

without using the same cell more than once in a path.

---

# 31. Word Search Backtracking

At each cell:

```text
Does character match word[index]?
```

If yes:

```text
mark visited
→ search 4 neighbors
→ restore cell
```

---

## 31.1 Java

```java
public class WordSearch {
    static boolean exist(char[][] board, String word) {
        int rows = board.length;
        int cols = board[0].length;

        for (int r = 0; r < rows; r++) {
            for (int c = 0; c < cols; c++) {

                if (dfs(board, word, r, c, 0)) {
                    return true;
                }
            }
        }

        return false;
    }

    static boolean dfs(char[][] board,
                       String word,
                       int r,
                       int c,
                       int index) {

        if (index == word.length()) {
            return true;
        }

        if (r < 0 || r >= board.length ||
            c < 0 || c >= board[0].length ||
            board[r][c] != word.charAt(index)) {
            return false;
        }

        char original = board[r][c];
        board[r][c] = '#';

        int[][] dirs = {
            {1, 0},
            {-1, 0},
            {0, 1},
            {0, -1}
        };

        for (int[] d : dirs) {
            if (dfs(board, word,
                    r + d[0], c + d[1], index + 1)) {
                board[r][c] = original;
                return true;
            }
        }

        board[r][c] = original;
        return false;
    }
}
```

### Complexity

For board size `R × C` and word length `L`:

```text
O(R × C × 4 × 3^(L-1))
```

approximately, because after the first cell, the previous cell cannot immediately be reused.

Space:

```text
O(L)
```

recursion depth.

---

# 32. Backtracking on Grids

Common grid-backtracking problems:

- Word Search
- N-Queens-style board placement
- Sudoku
- Rat in a Maze
- Path enumeration
- Unique Paths III
- Crossword solving
- Minesweeper-style search variants
- Hamiltonian path variants

Generic structure:

```java
boolean dfs(int r, int c, State state) {
    if (goal) return true;

    if (invalid) return false;

    mark(r, c);

    for (direction : directions) {
        if (dfs(next, updatedState)) {
            return true;
        }
    }

    unmark(r, c);
    return false;
}
```

---

# 33. Generate Parentheses

## Problem

Generate all valid combinations of:

```text
n pairs
```

of parentheses.

For:

```text
n = 3
```

examples:

```text
((()))
(()())
(())()
()(())
()()()
```

Count:

```text
Catalan(n)
```

---

# 34. Parentheses Backtracking

Maintain:

```text
open
close
```

Rules:

```text
open < n
```

allows adding `(`.

```text
close < open
```

allows adding `)`.

The second condition prevents an invalid prefix.

---

## 34.1 Java

```java
import java.util.*;

public class GenerateParentheses {
    static List<String> generateParenthesis(int n) {
        List<String> result = new ArrayList<>();

        backtrack(n, 0, 0,
                  new StringBuilder(), result);

        return result;
    }

    static void backtrack(int n,
                          int open,
                          int close,
                          StringBuilder path,
                          List<String> result) {

        if (path.length() == 2 * n) {
            result.add(path.toString());
            return;
        }

        if (open < n) {
            path.append('(');

            backtrack(n, open + 1, close,
                      path, result);

            path.deleteCharAt(path.length() - 1);
        }

        if (close < open) {
            path.append(')');

            backtrack(n, open, close + 1,
                      path, result);

            path.deleteCharAt(path.length() - 1);
        }
    }
}
```

---

# 35. Why `close < open`?

At every prefix:

```text
number of closing parentheses
≤
number of opening parentheses
```

Otherwise a closing parenthesis would have no matching opening parenthesis.

Therefore:

```text
close < open
```

is a pruning constraint.

This is an important example of **constraint-based pruning**.

---

# 36. Palindrome Partitioning

## Problem

Partition a string so every resulting substring is a palindrome.

Example:

```text
s = "aab"
```

Solutions:

```text
[a, a, b]
[aa, b]
```

---

# 37. Palindrome Partitioning Strategy

At index `start`:

```text
choose end from start to n-1
```

If:

```text
s[start..end]
```

is a palindrome:

```text
add substring
→ recurse from end + 1
→ undo
```

---

## 37.1 Java

```java
import java.util.*;

public class PalindromePartitioning {
    static List<List<String>> partition(String s) {
        List<List<String>> result = new ArrayList<>();

        backtrack(0, s, new ArrayList<>(), result);

        return result;
    }

    static void backtrack(int start,
                          String s,
                          List<String> path,
                          List<List<String>> result) {

        if (start == s.length()) {
            result.add(new ArrayList<>(path));
            return;
        }

        for (int end = start; end < s.length(); end++) {

            if (!isPalindrome(s, start, end)) {
                continue;
            }

            path.add(s.substring(start, end + 1));

            backtrack(end + 1, s, path, result);

            path.remove(path.size() - 1);
        }
    }

    static boolean isPalindrome(String s,
                                int left,
                                int right) {

        while (left < right) {
            if (s.charAt(left) != s.charAt(right)) {
                return false;
            }

            left++;
            right--;
        }

        return true;
    }
}
```

---

# 38. Palindrome Partitioning Optimization

Precompute:

```text
palindrome[i][j]
```

where:

```text
palindrome[i][j] = true
```

if `s[i..j]` is a palindrome.

Then each palindrome check becomes:

```text
O(1)
```

after:

```text
O(n²)
```

preprocessing.

---

# 39. Letter Combinations of a Phone Number

Mapping:

```text
2 → abc
3 → def
4 → ghi
...
9 → wxyz
```

For each digit, choose one corresponding letter.

If there are `n` digits and each has approximately four choices, the search tree is roughly:

```text
4^n
```

---

## 39.1 Java

```java
import java.util.*;

public class LetterCombinations {
    static final String[] MAP = {
        "", "", "abc", "def",
        "ghi", "jkl", "mno",
        "pqrs", "tuv", "wxyz"
    };

    static List<String> letterCombinations(String digits) {
        List<String> result = new ArrayList<>();

        if (digits == null || digits.isEmpty()) {
            return result;
        }

        backtrack(0, digits,
                  new StringBuilder(), result);

        return result;
    }

    static void backtrack(int index,
                          String digits,
                          StringBuilder path,
                          List<String> result) {

        if (index == digits.length()) {
            result.add(path.toString());
            return;
        }

        String letters =
                MAP[digits.charAt(index) - '0'];

        for (char ch : letters.toCharArray()) {
            path.append(ch);

            backtrack(index + 1, digits,
                      path, result);

            path.deleteCharAt(path.length() - 1);
        }
    }
}
```

---

# 40. Combination vs Permutation

This distinction appears constantly in interviews.

## Combination

Order does not matter.

```text
[1,2] == [2,1]
```

Use a `start` index to avoid generating both orders.

---

## Permutation

Order matters.

```text
[1,2] != [2,1]
```

Use:

```text
used[]
```

or swap-based recursion.

---

# 41. Subsets vs Combinations

Subsets generally allow each input element to be:

```text
included / excluded
```

Combinations usually select:

```text
k elements
```

or satisfy a target.

Example:

```text
Subsets:
all possible selections

Combinations:
specific-size / constraint-based selections
```

---

# 42. Swap-Based Permutations

Another way to generate permutations is to swap the current index with every later index.

```java
static void permute(int[] nums, int index,
                    List<List<Integer>> result) {

    if (index == nums.length) {
        List<Integer> current = new ArrayList<>();

        for (int x : nums) {
            current.add(x);
        }

        result.add(current);
        return;
    }

    for (int i = index; i < nums.length; i++) {
        swap(nums, index, i);

        permute(nums, index + 1, result);

        swap(nums, index, i);
    }
}
```

The second swap is the backtracking step.

---

# 43. Backtracking Pruning

Pruning means eliminating a branch as soon as it cannot produce a valid solution.

Examples:

### Combination Sum

```java
if (candidates[i] > remaining) break;
```

after sorting.

### Parentheses

```text
close < open
```

### N-Queens

```text
column / diagonal already occupied
```

### Sudoku

```text
digit violates a constraint
```

### Word Search

```text
cell already used
```

Pruning is often the difference between an unusable brute-force algorithm and a practical interview solution.

---

# 44. Sorting Before Backtracking

Sorting can make pruning possible.

Example:

```text
candidates = [2,3,6,7]
```

After sorting:

```text
if candidates[i] > remaining:
    break
```

because all later values are also too large.

Sorting can also help remove duplicate branches.

---

# 45. Duplicate Handling

Two common cases:

## Same value at same recursion depth

Skip:

```java
if (i > start && nums[i] == nums[i - 1]) {
    continue;
}
```

Used in:

```text
Subsets II
Combination Sum II
```

---

## Duplicate value in permutation

Use:

```java
if (i > 0 &&
    nums[i] == nums[i - 1] &&
    !used[i - 1]) {
    continue;
}
```

The key idea is:

> Avoid duplicate **branches**, not duplicate values globally.

---

# 46. Backtracking State Mutation

When modifying a shared object:

```java
path.add(x);
backtrack(...);
path.remove(path.size() - 1);
```

The remove operation is mandatory.

Likewise:

```java
used[i] = true;
backtrack(...);
used[i] = false;
```

And:

```java
board[r][c] = 'Q';
backtrack(...);
board[r][c] = '.';
```

### Rule

Every mutation should have a corresponding restoration.

---

# 47. Copying the Path

This is correct:

```java
result.add(new ArrayList<>(path));
```

This is usually wrong:

```java
result.add(path);
```

because `path` is mutable and will continue changing during backtracking.

Same issue with mutable arrays/boards.

---

# 48. Backtracking on Grids: Visited Strategies

Three common methods:

## Method 1 — Boolean matrix

```java
boolean[][] visited;
```

Mark and unmark.

---

## Method 2 — Modify board temporarily

```java
char original = board[r][c];
board[r][c] = '#';
...
board[r][c] = original;
```

Often simpler and O(1) extra space apart from recursion.

---

## Method 3 — Bitmask

Useful when the state is small enough to encode in bits.

---

# 49. Constraint Satisfaction Framework

A general CSP can be represented as:

```text
Variables:
X1, X2, ..., Xn

Domains:
D1, D2, ..., Dn

Constraints:
C1, C2, ..., Cm
```

Goal:

```text
assign every variable
while satisfying every constraint
```

Backtracking is a systematic search over assignments.

---

# 50. CSP Example — N-Queens

Variables:

```text
Q1, Q2, ..., Qn
```

where `Qi` represents the column selected for row `i`.

Domain:

```text
1 ... n
```

Constraints:

```text
Qi != Qj
|Qi - Qj| != |i - j|
```

The first constraint prevents shared columns.

The second prevents shared diagonals.

---

# 51. CSP Example — Sudoku

Each empty cell is a variable.

Domain:

```text
{1,...,9}
```

Constraints:

```text
same row → different values
same column → different values
same box → different values
```

---

# 52. Constraint Propagation

Instead of merely trying assignments, reduce domains whenever an assignment is made.

Example:

If a Sudoku cell receives:

```text
5
```

remove `5` from the domains of:

```text
same row
same column
same box
```

This can expose forced assignments before deeper recursion.

---

# 53. Forward Checking

After assigning a variable:

```text
remove its value from neighboring domains
```

If any neighbor has an empty domain:

```text
backtrack immediately
```

This is stronger than discovering the contradiction several levels later.

---

# 54. Backtracking Search Improvements

For harder CSPs:

1. **MRV** — choose variable with smallest domain.
2. **Degree heuristic** — break ties using most constraints.
3. **LCV** — choose value that constrains others least.
4. **Forward checking** — prune invalid future domains.
5. **Constraint propagation** — repeatedly reduce domains.

For standard LeetCode questions, a simpler implementation is usually preferred unless constraints require more.

---

# 55. Recursion Tree Complexity

Common branching structures:

### Binary recursion

```text
2^n
```

Examples:

```text
subsets
include/exclude
```

### Permutations

```text
n!
```

### Fixed branching `k`

```text
k^n
```

Example:

```text
phone digit combinations
```

### Grid word search

Approximately:

```text
R × C × 4 × 3^(L-1)
```

### N-Queens

Factorial-style search with substantial pruning.

---

# 56. Recursion Stack Complexity

If recursion depth is:

```text
n
```

then stack space is generally:

```text
O(n)
```

Examples:

- subsets → O(n)
- permutations → O(n)
- combination sum → up to O(target/minCandidate)
- parentheses → O(n)
- palindrome partitioning → O(n)
- word search → O(L)

---

# 57. Tail Recursion

A recursive call is tail-recursive if it is the final operation of the function.

Example:

```java
static void countDown(int n) {
    if (n == 0) return;

    System.out.println(n);
    countDown(n - 1);
}
```

Java does not generally guarantee tail-call optimization, so deep recursion can still cause `StackOverflowError`.

---

# 58. Recursion vs Iteration

| Aspect | Recursion | Iteration |
|---|---|---|
| Natural for trees | Yes | Less natural |
| Backtracking | Excellent | Usually cumbersome |
| Explicit stack needed | Runtime stack | Often manual |
| Deep input risk | Stack overflow | Usually safer |
| Code clarity | Often concise | Often more verbose for search |

For interview backtracking, recursion is usually the natural representation.

---

# 59. Common Recursion Mistakes

## Mistake 1 — Missing base case

Causes infinite recursion.

## Mistake 2 — No progress

Calling:

```java
solve(n)
```

from:

```java
solve(n)
```

without changing state.

## Mistake 3 — Forgetting to undo

Causes sibling branches to share incorrect state.

## Mistake 4 — Storing mutable references

Use copies when recording solutions.

## Mistake 5 — Incorrect duplicate skipping

Duplicates must usually be skipped at the correct recursion depth.

## Mistake 6 — Incorrect index movement

Example:

```text
Combination Sum → reuse allowed
```

so recurse on:

```text
i
```

whereas a use-once variant recurses on:

```text
i + 1
```

---

# 60. GATE Theory

## 60.1 Recurrence

A recursive algorithm can often be analyzed with:

```text
T(n) = number of recursive calls × T(smaller input)
       + non-recursive work
```

Examples:

```text
T(n) = T(n-1) + O(1)
     = O(n)
```

```text
T(n) = 2T(n-1) + O(1)
     = O(2^n)
```

---

## 60.2 Binary Recursion

For:

```text
T(n) = 2T(n-1) + c
```

expansion gives:

```text
T(n)
= 2[2T(n-2)+c] + c
= 4T(n-2)+3c
= ...
= 2^n T(0) + (2^n - 1)c
```

Therefore:

```text
T(n) = Θ(2^n)
```

---

## 60.3 Factorial Recursion

For permutation generation:

```text
T(n) ≈ n × T(n-1)
```

so:

```text
T(n) = Θ(n!)
```

Output copying may add another factor:

```text
Θ(n × n!)
```

---

## 60.4 Catalan Numbers

The number of valid parenthesis strings containing `n` pairs is:

```text
C_n = 1/(n+1) × binomial(2n,n)
```

Asymptotically:

```text
C_n = Θ(4^n / n^(3/2))
```

---

## 60.5 Backtracking Search Tree

If every level has `b` choices and depth is `d`:

```text
O(b^d)
```

before considering pruning.

Pruning reduces the practical search space but does not necessarily change the worst-case asymptotic bound.

---

# 61. GATE-Style Solved Questions

> These are **GATE-style practice questions**, not claimed as verbatim official GATE PYQs.

---

## Question 1 — Recursion Tree

Consider:

```text
T(n) = 2T(n-1) + 3
T(0) = 1
```

Find the asymptotic complexity.

### Solution

Expand:

```text
T(n)
= 2T(n-1) + 3
= 4T(n-2) + 6 + 3
= 8T(n-3) + 12 + 6 + 3
```

After `k` expansions:

```text
T(n) = 2^k T(n-k) + 3(2^k - 1)
```

Set:

```text
k = n
```

Then:

```text
T(n) = 2^n T(0) + 3(2^n - 1)
```

Therefore:

```text
T(n) = Θ(2^n)
```

### Answer

```text
Θ(2^n)
```

---

## Question 2 — Subsets

An array contains `n` distinct elements. A program generates every subset and copies each generated subset into an output list.

What is the asymptotic time complexity?

### Solution

Every element has two choices:

```text
include
exclude
```

Therefore:

```text
number of subsets = 2^n
```

A subset can contain up to `n` elements, and copying it can require O(n).

Therefore:

```text
O(n × 2^n)
```

### Answer

```text
Θ(n × 2^n)
```

for explicit output generation.

---

## Question 3 — Permutations

A backtracking algorithm generates all permutations of `n` distinct elements. At each leaf, it copies the permutation into an output array.

What is the output-sensitive time complexity?

### Solution

Number of permutations:

```text
n!
```

Each permutation contains:

```text
n
```

elements.

Copying all permutations:

```text
n × n!
```

### Answer

```text
Θ(n × n!)
```

---

## Question 4 — Parentheses

How many valid parenthesis sequences exist for `n = 4` pairs?

### Solution

The number is the fourth Catalan number:

```text
C4 = 1/(4+1) × C(8,4)
```

```text
C(8,4) = 70
```

Therefore:

```text
C4 = 70 / 5 = 14
```

### Answer

```text
14
```

---

## Question 5 — N-Queens Constraint

For an `n × n` board, a queen is placed at `(r,c)`. Which conditions must hold to place another queen at `(r2,c2)`?

### Solution

No shared column:

```text
c != c2
```

No shared main diagonal:

```text
r - c != r2 - c2
```

No shared anti-diagonal:

```text
r + c != r2 + c2
```

Equivalently:

```text
|r-r2| != |c-c2|
```

### Answer

All three constraints must hold.

---

## Question 6 — Backtracking State

Consider:

```java
path.add(x);
solve(...);
path.remove(path.size() - 1);
```

Why is the final `remove` necessary?

### Solution

`path` is shared among recursive calls.

After exploring the branch containing `x`, the next sibling branch must see the path as it existed before `x` was selected.

Therefore:

```text
add → recurse → remove
```

restores the previous state.

Without the remove operation, choices from one branch leak into another branch.

### Answer

The removal is the **backtracking/undo step** that restores the state.

---

# 62. LeetCode Roadmap

## Phase 1 — Basic Recursion

Master:

1. Fibonacci Number
2. Power of Two
3. Reverse String
4. Reverse Linked List
5. Maximum Depth of Binary Tree
6. Same Tree
7. Symmetric Tree
8. Binary Tree Paths

Goal:

```text
Base case
+
recursive state
+
return values
```

---

# 63. Phase 2 — Subsets and Subsequences

Master:

1. Subsets
2. Subsets II
3. Subsequences
4. Letter Case Permutation
5. Find Subsequences
6. Combination Sum
7. Combination Sum II
8. Combination Sum III

Core skills:

```text
start index
include/exclude
duplicate skipping
remaining target
```

---

# 64. Phase 3 — Permutations

Master:

1. Permutations
2. Permutations II
3. Next Permutation
4. Letter Combinations of a Phone Number
5. Beautiful Arrangement
6. Construct the Lexicographically Largest Valid Sequence

Core skills:

```text
used[]
duplicate control
choice ordering
```

---

# 65. Phase 4 — Constraint Backtracking

Master:

1. Generate Parentheses
2. Palindrome Partitioning
3. Word Search
4. N-Queens
5. Sudoku Solver
6. Restore IP Addresses
7. Matchsticks to Square
8. Partition to K Equal Sum Subsets

Core skills:

```text
constraints
pruning
state restoration
```

---

# 66. Phase 5 — Grid Backtracking

Master:

1. Word Search
2. Unique Paths III
3. Rat in a Maze variants
4. Sudoku Solver
5. N-Queens
6. Crossword-style constraint problems
7. Hamiltonian path variants

Focus on:

```text
visited
directions
boundary checks
mark/unmark
```

---

# 67. Problem Recognition Table

| Problem wording | Pattern |
|---|---|
| “Generate all subsets” | Include/exclude |
| “Generate all subsequences” | Include/exclude |
| “All permutations” | `used[]` / swapping |
| “Choose k elements” | Start index |
| “Combination sum” | Start index + remaining target |
| “Reuse elements” | Recurse with same index |
| “Use each once” | Recurse with `i + 1` |
| “No duplicate combinations” | Sort + same-level skip |
| “Valid parentheses” | Open/close constraints |
| “Palindrome partition” | Choose substring + palindrome check |
| “Find word in grid” | DFS + backtracking |
| “Place queens” | Row + column + diagonals |
| “Fill Sudoku” | CSP + constraints |
| “All valid assignments” | CSP backtracking |

---

# 68. Pattern: Include / Exclude

Use when each element has a binary decision:

```text
take
or
skip
```

Template:

```java
void solve(int index) {
    if (index == n) {
        record();
        return;
    }

    choose(index);
    solve(index + 1);
    undo();

    skip(index);
    solve(index + 1);
}
```

Typical:

```text
subsets
subsequences
some selection problems
```

---

# 69. Pattern: Start Index

Use when order does not matter.

```java
for (int i = start; i < n; i++) {
    path.add(nums[i]);

    backtrack(i + 1, ...);

    path.remove(path.size() - 1);
}
```

This prevents generating:

```text
[1,2]
[2,1]
```

as separate combinations.

---

# 70. Pattern: Used Array

Use when order matters.

```java
for (int i = 0; i < n; i++) {
    if (used[i]) continue;

    used[i] = true;
    path.add(nums[i]);

    backtrack(...);

    path.remove(path.size() - 1);
    used[i] = false;
}
```

Typical:

```text
permutations
```

---

# 71. Pattern: Remaining Target

For target-based problems:

```text
remaining = target - selectedValue
```

Prune when:

```text
remaining < 0
```

or, after sorting:

```text
candidate > remaining
```

Typical:

```text
Combination Sum
partitioning
subset target problems
```

---

# 72. Pattern: Grid DFS + Backtracking

Template:

```java
boolean dfs(int r, int c) {
    if (goal) return true;

    if (invalid) return false;

    mark();

    for (direction : directions) {
        if (dfs(nextRow, nextCol)) {
            unmark();
            return true;
        }
    }

    unmark();
    return false;
}
```

Important:

```text
mark before recursion
unmark after recursion
```

---

# 73. Pattern: Constraint Placement

Examples:

```text
N-Queens
Sudoku
crossword
graph coloring
```

At each step:

```text
choose variable
→ try candidate
→ check constraints
→ recurse
→ undo
```

---

# 74. Graph Coloring as CSP

Given a graph and `k` colors:

```text
assign one color to each vertex
```

Constraint:

```text
adjacent vertices must have different colors
```

Backtracking:

```text
color vertex 0
→ color vertex 1
→ ...
→ if conflict, undo
```

This is a useful extension of the CSP pattern.

---

# 75. Hamiltonian Path Backtracking

A Hamiltonian path visits every vertex exactly once.

State:

```text
current vertex
visited[]
path
```

At each step:

```text
try an unvisited neighbor
```

If no valid extension exists:

```text
backtrack
```

This demonstrates why general backtracking can become exponential.

---

# 76. Optimization: Choose the Most Constrained Option

A powerful general heuristic:

> Branch first on the choice with the fewest possibilities.

Examples:

### Sudoku

Choose the empty cell with the fewest candidates.

### N-Queens

Ordering rows/columns strategically can reduce search.

### CSP

Use MRV.

This does not necessarily change worst-case complexity, but can drastically improve practical performance.

---

# 77. When Backtracking Is Appropriate

Use backtracking when the problem asks for:

- all solutions
- all combinations
- all permutations
- all valid configurations
- existence of a configuration under constraints
- exact arrangement
- constraint satisfaction
- exhaustive search over a manageable decision space

Strong wording signals:

```text
generate all
find every
enumerate
arrange
place
partition
choose
construct
solve
```

---

# 78. When Not to Use Backtracking

Do not automatically use backtracking when:

- a greedy solution is provably sufficient
- a DP state captures overlapping subproblems
- a graph algorithm gives a polynomial-time solution
- the input is too large for exponential search
- the problem only asks for a count and DP/combinatorics can compute it directly

Backtracking is often the **search framework**, not necessarily the final optimization.

---

# 79. Backtracking vs DP

Consider a decision tree.

### Backtracking

Explores:

```text
choice → choice → choice
```

and prunes invalid branches.

### DP

Merges equivalent states.

If many branches reach the same state:

```text
Backtracking:
recomputes it

DP:
memoizes it
```

This is one of the most important connections between the techniques.

---

# 80. Memoization + Backtracking

Some recursive search problems can be improved by memoizing states.

Example state:

```text
(index, remaining)
```

If the same state occurs repeatedly, store the result.

Conceptually:

```text
Backtracking
+
state caching
=
memoized search / DP
```

The key is identifying a sufficiently small state representation.

---

# 81. Backtracking Optimization Checklist

Before coding, ask:

```text
1. What is my state?
2. What are my choices?
3. What is the base case?
4. What constraints invalidate a choice?
5. Can I sort to enable pruning?
6. Can I skip duplicate branches?
7. Can I choose the most constrained variable?
8. What must be undone?
9. Can equivalent states be memoized?
10. What is the worst-case branching factor?
```

---

# 82. Java Backtracking Skeleton

```java
static void backtrack(
        int start,
        List<Integer> path,
        List<List<Integer>> result) {

    if (/* complete */) {
        result.add(new ArrayList<>(path));
        return;
    }

    for (int i = start; i < n; i++) {

        if (/* invalid */) {
            continue;
        }

        path.add(...);

        backtrack(i + 1, path, result);

        path.remove(path.size() - 1);
    }
}
```

Adapt:

```text
start
```

to:

```text
used[]
remaining
board
index
constraints
```

depending on the problem.

---

# 83. Final Complexity Cheat Sheet

| Problem | Search structure | Typical complexity |
|---|---|---:|
| Subsets | Binary tree | O(n·2^n) |
| Subsequences | Binary tree | O(n·2^n) |
| Permutations | Factorial tree | O(n·n!) |
| Phone combinations | k-ary tree | O(n·4^n) upper bound |
| Generate parentheses | Catalan output | O(n·C_n) output-sensitive |
| Combination Sum | Exponential | Depends on target/candidates |
| N-Queens | Factorial-style | O(n!) upper-bound style |
| Sudoku | Constraint search | Exponential worst case |
| Word Search | Grid branching | O(RC·3^L) approximate |
| Palindrome partitioning | Partition tree | Exponential |

These bounds describe worst-case/search behavior and may be substantially smaller in practice due to pruning.

---

# 84. Mastery Checklist

## Recursion

- [ ] Explain base case.
- [ ] Explain recursive case.
- [ ] Trace call stack.
- [ ] Draw recursion trees.
- [ ] Solve simple recurrences.
- [ ] Calculate recursion depth.
- [ ] Identify stack-space usage.

## Backtracking

- [ ] Write choose → recurse → undo.
- [ ] Define state.
- [ ] Define choices.
- [ ] Define constraints.
- [ ] Implement pruning.
- [ ] Restore mutable state correctly.
- [ ] Copy paths when recording answers.

## Subsets / Subsequences

- [ ] Subsets
- [ ] Subsets II
- [ ] Subsequences
- [ ] Include/exclude
- [ ] Same-level duplicate skipping

## Permutations

- [ ] Permutations
- [ ] Permutations II
- [ ] Used array
- [ ] Swap-based method
- [ ] Duplicate handling

## Combinations

- [ ] Combination Sum
- [ ] Combination Sum II
- [ ] Combination Sum III
- [ ] Start-index technique
- [ ] Remaining-target pruning

## Constraint Problems

- [ ] N-Queens
- [ ] Sudoku
- [ ] Word Search
- [ ] Generate Parentheses
- [ ] Palindrome Partitioning
- [ ] Letter Combinations
- [ ] Grid backtracking
- [ ] CSP formulation

## Advanced

- [ ] MRV
- [ ] Degree heuristic
- [ ] LCV
- [ ] Forward checking
- [ ] Constraint propagation
- [ ] Memoized search
- [ ] Recognize when DP replaces repeated backtracking

---

# 85. Final Revision Sheet

## Recursion

```text
Base case
+
smaller problem
+
progress
```

## Backtracking

```text
Choose
→ Check
→ Apply
→ Recurse
→ Undo
```

## Subsets

```text
include / exclude
→ 2^n
```

## Subsequences

```text
include / exclude
while preserving order
```

## Permutations

```text
choose unused element
→ n!
```

## Combination Sum

```text
start index
+
remaining target
+
same index if reuse is allowed
```

## N-Queens

```text
row
+
column
+
(r-c) diagonal
+
(r+c) diagonal
```

## Sudoku

```text
cell
+
candidate digit
+
row/column/box constraints
```

## Word Search

```text
match character
→ mark
→ 4 directions
→ unmark
```

## Generate Parentheses

```text
open < n
close < open
```

## Palindrome Partitioning

```text
choose substring
→ palindrome?
→ recurse
→ undo
```

## Phone Combinations

```text
one digit
→ choose one letter
→ recurse
```

## CSP

```text
variable
→ domain value
→ constraint check
→ recurse
→ undo
```

---

# 86. One-Page Mental Model

```text
RECURSION
│
├── Base case
├── Smaller state
├── Call stack
└── Recurrence
        │
        ▼
BACKTRACKING
│
├── State
├── Choices
├── Constraints
├── Choose
├── Recurse
└── Undo
        │
        ├── Include/Exclude
        │      └── Subsets / Subsequences
        │
        ├── Start Index
        │      └── Combinations
        │
        ├── Used[]
        │      └── Permutations
        │
        ├── Grid DFS
        │      └── Word Search / Paths
        │
        └── CSP
               ├── N-Queens
               ├── Sudoku
               ├── Parentheses
               └── Other constraint problems
```

---

# 87. Final Mastery Standard

You have mastered **Recursion & Backtracking** when you can:

1. Identify the recursive state immediately.
2. Define a correct base case.
3. Explain exactly how the state becomes smaller.
4. Draw the recursion tree for a small input.
5. Derive the basic complexity from the tree.
6. Write the choose → recurse → undo pattern from memory.
7. Generate subsets and subsequences.
8. Generate permutations with and without duplicates.
9. Solve combination-sum variants.
10. Solve N-Queens using column and diagonal constraints.
11. Solve Sudoku using constraint checking.
12. Solve Word Search using grid DFS + restoration.
13. Generate valid parentheses using pruning constraints.
14. Partition strings using recursive substring choices.
15. Recognize CSP structure.
16. Apply pruning rather than blindly exploring every branch.
17. Handle duplicate branches correctly.
18. Know when memoization/DP is more appropriate.
19. Analyze recursion depth and stack space.
20. Explain the state, transition, pruning, and complexity clearly in an interview.

> **Core principle:**  
> **Backtracking is systematic search: make a choice, enforce constraints, explore the choice, and restore the state before trying the next choice.**
