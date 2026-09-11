# 14 — Trie

> **Goal:** Master Trie and Bitwise Trie patterns for FAANG/top-product-company interviews. Focus on prefix-based search, dictionary operations, word-pattern problems, autocomplete, and using binary tries to maximize XOR.

---

# 1. What Is a Trie?

A **Trie** (prefix tree) is a tree-based data structure used primarily for storing strings by sharing common prefixes.

Example words:

```text
cat
car
care
dog
```

A Trie can represent them as:

```text
(root)
 ├── c
 │    └── a
 │         ├── t*
 │         └── r*
 │              └── e*
 │
 └── d
      └── o
           └── g*
```

`*` means:

```text
end of a stored word
```

The important property is:

> **Each root-to-node path represents a prefix.**

Therefore, prefix operations become natural.

---

# 2. Why Use a Trie?

Suppose we store:

```text
apple
app
application
apply
apt
```

All these strings share:

```text
ap
```

A Trie stores that shared structure once.

This makes the following operations efficient:

- exact word search
- word insertion
- prefix search
- finding all words with a prefix
- autocomplete
- dictionary matching
- word-pattern problems
- lexicographic traversal

For a word of length `L`:

```text
insert = O(L)
search = O(L)
prefix search = O(L)
```

The complexity depends on the string length, not directly on the number of stored words.

---

# 3. Trie vs HashSet

A `HashSet<String>` is excellent for exact membership:

```text
"apple" exists?
```

But it does not naturally answer:

```text
How many words start with "app"?
Which words start with "app"?
Give autocomplete suggestions for "app".
```

A Trie is designed around prefixes.

| Operation | HashSet | Trie |
|---|---:|---:|
| Exact word search | O(L) average | O(L) |
| Insert | O(L) average | O(L) |
| Prefix existence | Not natural | O(P) |
| Prefix enumeration | Poor | Natural |
| Autocomplete | Requires extra work | Natural |
| Shared prefixes | No structural sharing | Yes |

`L` = word length, `P` = prefix length.

---

# 4. Basic Trie Node

For lowercase English letters:

```java
static class TrieNode {
    TrieNode[] children = new TrieNode[26];
    boolean isEnd;
}
```

Mapping:

```text
'a' → 0
'b' → 1
...
'z' → 25
```

Use:

```java
int index = ch - 'a';
```

---

# 5. Basic Trie Implementation

```java
static class Trie {
    static class Node {
        Node[] children = new Node[26];
        boolean isEnd;
    }

    private final Node root = new Node();

    // methods...
}
```

The root does not represent a character.

It represents the starting point before the first character.

---

# 6. Insert

To insert:

```text
apple
```

follow/create:

```text
a → p → p → l → e
```

Then mark:

```text
e.isEnd = true
```

---

# 7. Trie Insert Java

```java
void insert(String word) {
    Node current = root;

    for (char ch : word.toCharArray()) {
        int index = ch - 'a';

        if (current.children[index] == null) {
            current.children[index] = new Node();
        }

        current = current.children[index];
    }

    current.isEnd = true;
}
```

### Complexity

For word length `L`:

```text
Time: O(L)
Extra traversal space: O(1)
```

The Trie itself may allocate up to:

```text
O(total characters)
```

nodes in the worst case.

---

# 8. Search Exact Word

To search for:

```text
apple
```

follow the same character path.

At the end:

```text
path exists
AND
isEnd == true
```

Both conditions are necessary.

---

# 9. Why `isEnd` Is Necessary

Suppose the Trie contains:

```text
apple
```

Then the path for:

```text
app
```

exists.

But if `app` was never inserted:

```text
isEnd at 'p' = false
```

Therefore:

```text
search("app") = false
```

This distinction is fundamental:

> **A prefix existing does not mean that the prefix itself is a stored word.**

---

# 10. Search Java

```java
boolean search(String word) {
    Node node = findNode(word);

    return node != null && node.isEnd;
}

private Node findNode(String text) {
    Node current = root;

    for (char ch : text.toCharArray()) {
        int index = ch - 'a';

        if (current.children[index] == null) {
            return null;
        }

        current = current.children[index];
    }

    return current;
}
```

### Complexity

```text
Time: O(L)
Space: O(1) auxiliary
```

---

# 11. Prefix Search

The question:

> Does any stored word start with this prefix?

is different from exact search.

For:

```text
words = ["apple", "app", "application"]
prefix = "appl"
```

the answer is:

```text
true
```

We only need to verify that the prefix path exists.

---

# 12. `startsWith()` Java

```java
boolean startsWith(String prefix) {
    return findNode(prefix) != null;
}
```

### Complexity

```text
Time: O(P)
Space: O(1) auxiliary
```

---

# 13. Complete Basic Trie

```java
static class Trie {
    static class Node {
        Node[] children = new Node[26];
        boolean isEnd;
    }

    private final Node root = new Node();

    void insert(String word) {
        Node current = root;

        for (char ch : word.toCharArray()) {
            int index = ch - 'a';

            if (current.children[index] == null) {
                current.children[index] = new Node();
            }

            current = current.children[index];
        }

        current.isEnd = true;
    }

    boolean search(String word) {
        Node node = findNode(word);
        return node != null && node.isEnd;
    }

    boolean startsWith(String prefix) {
        return findNode(prefix) != null;
    }

    private Node findNode(String text) {
        Node current = root;

        for (char ch : text.toCharArray()) {
            int index = ch - 'a';

            if (current.children[index] == null) {
                return null;
            }

            current = current.children[index];
        }

        return current;
    }
}
```

This is the baseline Trie implementation you should know.

---

# 14. Trie Node Design Choices

There are several ways to store children.

## Fixed Array

```java
Node[] children = new Node[26];
```

### Advantages

- Fast
- Simple
- O(1) child access for lowercase English
- Excellent for interviews

### Disadvantage

Potentially wastes memory when each node has few children.

---

## HashMap

```java
Map<Character, Node> children = new HashMap<>();
```

Useful when the alphabet is large or sparse.

Example:

```text
Unicode
mixed symbols
arbitrary character sets
```

### Tradeoff

Array:

```text
faster + more memory
```

HashMap:

```text
more flexible + additional overhead
```

---

# 15. Trie with HashMap Children

```java
static class Node {
    Map<Character, Node> children = new HashMap<>();
    boolean isEnd;
}
```

Insertion:

```java
void insert(String word) {
    Node current = root;

    for (char ch : word.toCharArray()) {
        current = current.children.computeIfAbsent(
                ch,
                k -> new Node()
        );
    }

    current.isEnd = true;
}
```

Search:

```java
boolean search(String word) {
    Node current = root;

    for (char ch : word.toCharArray()) {
        current = current.children.get(ch);

        if (current == null) {
            return false;
        }
    }

    return current.isEnd;
}
```

---

# 16. Delete from Trie

Deletion is more subtle than insertion.

Suppose:

```text
words:
app
apple
```

Trie:

```text
a → p → p*
          \
           l → e*
```

If we delete:

```text
apple
```

we can remove:

```text
l → e
```

But we cannot remove:

```text
a → p → p
```

because `app` is still a word.

---

# 17. Trie Delete Rules

A node can be removed only if:

```text
1. It is not the end of another word.
2. It has no children.
```

Otherwise the node must remain.

---

# 18. Recursive Trie Delete

```java
boolean delete(String word) {
    if (!search(word)) {
        return false;
    }

    delete(root, word, 0);
    return true;
}

private boolean delete(Node node, String word, int index) {
    if (index == word.length()) {
        node.isEnd = false;
        return hasNoChildren(node);
    }

    int childIndex = word.charAt(index) - 'a';
    Node child = node.children[childIndex];

    boolean shouldDeleteChild =
            delete(child, word, index + 1);

    if (shouldDeleteChild) {
        node.children[childIndex] = null;
    }

    return !node.isEnd && hasNoChildren(node);
}

private boolean hasNoChildren(Node node) {
    for (Node child : node.children) {
        if (child != null) {
            return false;
        }
    }

    return true;
}
```

### Complexity

For word length `L`:

```text
Time: O(L × alphabet)
```

with the fixed 26-array implementation because `hasNoChildren()` scans up to 26 children at each level.

Since 26 is a constant:

```text
O(L)
```

asymptotically.

---

# 19. More Efficient Delete with Child Count

For a general or large alphabet, scanning every child can be avoided by maintaining:

```java
int childCount;
```

Then a node is removable when:

```text
!isEnd && childCount == 0
```

This changes constant factors and is useful when the alphabet is large.

---

# 20. Word Dictionary

A Trie naturally acts as a dictionary.

Required operations may include:

```text
addWord(word)
search(word)
startsWith(prefix)
remove(word)
```

This is the foundation of many interview problems.

A dictionary Trie can also maintain metadata:

```text
frequency
word count
prefix count
terminal word
```

These extensions power autocomplete and ranking.

---

# 21. Prefix Search — Returning All Words

Suppose:

```text
words:
apple
app
application
apt
banana
```

Query:

```text
"app"
```

Expected:

```text
app
apple
application
```

### Strategy

1. Navigate to the node representing the prefix.
2. DFS from that node.
3. Whenever `isEnd == true`, construct the corresponding word.

---

# 22. Collect Words Under a Prefix

```java
List<String> wordsWithPrefix(String prefix) {
    Node node = findNode(prefix);

    List<String> result = new ArrayList<>();

    if (node == null) {
        return result;
    }

    StringBuilder current = new StringBuilder(prefix);
    collect(node, current, result);

    return result;
}

private void collect(
        Node node,
        StringBuilder current,
        List<String> result) {

    if (node.isEnd) {
        result.add(current.toString());
    }

    for (int i = 0; i < 26; i++) {
        Node child = node.children[i];

        if (child == null) {
            continue;
        }

        current.append((char) ('a' + i));

        collect(child, current, result);

        current.deleteCharAt(current.length() - 1);
    }
}
```

This is a classic:

```text
Trie + DFS + backtracking
```

pattern.

---

# 23. Complexity of Prefix Enumeration

Let:

```text
P = prefix length
K = number of returned words
S = total number of characters represented by those returned words
```

Navigation to the prefix:

```text
O(P)
```

Enumeration depends on the explored Trie subtree.

A useful output-sensitive description is:

```text
O(P + explored subtree size + output construction)
```

If all descendants must be examined, it can approach the size of the entire prefix subtree.

The important point:

> Finding the prefix is O(P); generating all matching words necessarily costs at least the amount of output produced.

---

# 24. Autocomplete

Autocomplete asks:

> Given a prefix, return likely or valid completions.

Example:

```text
dictionary:
car
card
care
career
cat
dog
```

Input:

```text
ca
```

Possible suggestions:

```text
car
card
care
career
cat
```

The Trie solves the prefix-navigation problem.

---

# 25. Basic Autocomplete

Basic autocomplete:

```text
prefix
→ find Trie node
→ DFS descendants
→ return words
```

If the problem only asks for all matching words, this is enough.

---

# 26. Ranked Autocomplete

Real autocomplete often needs:

```text
prefix
→ candidates
→ rank by frequency / score
→ top K
```

A Trie node can store:

```java
int frequency;
```

at terminal nodes.

Then DFS can collect candidates:

```java
class Suggestion {
    String word;
    int frequency;
}
```

After collecting:

```text
sort by frequency descending
```

or use a:

```text
min heap of size K
```

for top-K results.

This connects:

```text
Trie + DFS + Heap
```

which is a common advanced pattern.

---

# 27. Top-K Autocomplete Pattern

For a prefix:

```text
1. Navigate to prefix node.
2. DFS all candidate words.
3. Maintain min heap of size K.
4. Keep highest-frequency K words.
5. Extract and reverse if necessary.
```

If there are `M` candidate words:

```text
O(P + traversal + M log K)
```

rather than sorting all candidates:

```text
O(P + traversal + M log M)
```

when `K << M`.

---

# 28. Word Search

"Word Search" can refer to multiple Trie-based interview patterns.

The most important is:

> Search a word or multiple dictionary words while traversing a character board.

A Trie is useful because we can stop exploring a board path as soon as its current character sequence is not a prefix of any dictionary word.

This is much better than checking every candidate word independently.

---

# 29. Trie + Grid DFS

Suppose the dictionary contains:

```text
oath
pea
eat
rain
```

and the board contains characters.

Build a Trie from the dictionary.

During board DFS:

```text
current path
      |
      v
Trie child exists?
   /       \
 no         yes
 |           |
stop       continue
```

If:

```text
Trie node.isEnd == true
```

we found a dictionary word.

---

# 30. Why Trie Helps Word Search

Without a Trie:

```text
for every word:
    search board
```

This repeats work across common prefixes.

With a Trie:

```text
board path → Trie path
```

Shared prefixes are explored together.

For example:

```text
eat
eats
eater
```

all share:

```text
eat
```

The board traversal can share that prefix work.

---

# 31. Word Search Template

```java
static class WordTrieNode {
    WordTrieNode[] children = new WordTrieNode[26];
    String word;
}
```

Storing the complete word at terminal nodes can simplify result generation.

DFS:

```java
static void dfs(
        char[][] board,
        int r,
        int c,
        WordTrieNode node,
        List<String> result) {

    if (r < 0 || r >= board.length
            || c < 0 || c >= board[0].length) {
        return;
    }

    char ch = board[r][c];

    if (ch == '#') {
        return;
    }

    WordTrieNode next = node.children[ch - 'a'];

    if (next == null) {
        return;
    }

    if (next.word != null) {
        result.add(next.word);
        next.word = null;
    }

    board[r][c] = '#';

    dfs(board, r + 1, c, next, result);
    dfs(board, r - 1, c, next, result);
    dfs(board, r, c + 1, next, result);
    dfs(board, r, c - 1, next, result);

    board[r][c] = ch;
}
```

The Trie root is passed initially for each board cell.

---

# 32. Word Search Complexity

Let:

```text
R = board rows
C = board columns
L = maximum dictionary word length
```

A straightforward Trie + DFS solution has worst-case exponential board exploration, roughly bounded by:

```text
O(R × C × 4 × 3^(L-1))
```

for simple 4-directional movement without revisiting cells.

The Trie provides aggressive prefix pruning and is often dramatically faster in practice.

Do not claim Trie makes arbitrary board search polynomial; the board path exploration itself can still be exponential in the maximum word length.

---

# 33. Trie Pruning in Word Search

An important optimization is to remove exhausted Trie branches.

If a Trie node:

```text
isEnd == false
AND
has no children
```

after finding a word, it can potentially be detached from its parent.

This prevents future DFS calls from exploring a branch that can no longer produce a new answer.

This is an advanced optimization, not necessary for the basic solution.

---

# 34. Maximum XOR

Now we move from character Tries to a **bitwise Trie**.

Problem pattern:

> Given integers, find the maximum possible XOR of two numbers.

Example:

```text
nums = [3, 10, 5, 25, 2, 8]
```

The maximum XOR is:

```text
5 XOR 25 = 28
```

because:

```text
00101
11001
-----
11100 = 28
```

---

# 35. Why Greedy Bit Selection Works

XOR produces:

```text
0 XOR 0 = 0
1 XOR 1 = 0
0 XOR 1 = 1
1 XOR 0 = 1
```

To maximize a number, we want the highest bit possible to be `1`.

Therefore, at each bit:

> If the current number has bit `b`, prefer a stored number with the opposite bit `1-b`.

This is a greedy strategy from the most significant bit downward.

---

# 36. Bitwise Trie

Instead of characters:

```text
'a' ... 'z'
```

each node has exactly two possible children:

```text
0
1
```

Example number:

```text
5 = 101
```

Trie path:

```text
1 → 0 → 1
```

For fixed-width integers, process all relevant bits.

For Java `int`, a standard implementation uses:

```text
bits 30 ... 0
```

for non-negative values up to `2^31 - 1`.

If arbitrary signed integers are involved, carefully define the comparison goal and bit width.

---

# 37. Bitwise Trie Node

```java
static class BitNode {
    BitNode[] child = new BitNode[2];
}
```

---

# 38. Insert Number into Bitwise Trie

```java
static void insertNumber(BitNode root, int num) {
    BitNode current = root;

    for (int bit = 30; bit >= 0; bit--) {
        int b = (num >>> bit) & 1;

        if (current.child[b] == null) {
            current.child[b] = new BitNode();
        }

        current = current.child[b];
    }
}
```

Use:

```java
>>>
```

for unsigned right shift when extracting bits.

---

# 39. Find Maximum XOR for One Number

For each bit:

```text
current bit = b
preferred bit = 1 - b
```

If preferred branch exists, take it.

Otherwise take the same-bit branch.

```java
static int maxXorWith(BitNode root, int num) {
    BitNode current = root;
    int answer = 0;

    for (int bit = 30; bit >= 0; bit--) {
        int b = (num >>> bit) & 1;
        int wanted = 1 - b;

        if (current.child[wanted] != null) {
            answer |= (1 << bit);
            current = current.child[wanted];
        } else {
            current = current.child[b];
        }
    }

    return answer;
}
```

---

# 40. Maximum XOR of Two Numbers

```java
static int findMaximumXOR(int[] nums) {
    BitNode root = new BitNode();

    for (int num : nums) {
        insertNumber(root, num);
    }

    int answer = 0;

    for (int num : nums) {
        answer = Math.max(answer, maxXorWith(root, num));
    }

    return answer;
}
```

### Complexity

For `n` numbers and `B` bits:

```text
Build: O(nB)
Queries: O(nB)

Total: O(nB)
Space: O(nB)
```

For 31 fixed bits:

```text
O(n)
```

asymptotically with respect to `n`, though with a large constant.

---

# 41. Example: Maximum XOR

Numbers:

```text
5  = 00101
25 = 11001
```

For `5`:

```text
0 0 1 0 1
```

To maximize XOR:

```text
prefer 1 when current bit is 0
prefer 0 when current bit is 1
```

For `25`:

```text
1 1 0 0 1
```

XOR:

```text
0 0 1 0 1
1 1 0 0 1
-----------
1 1 1 0 0
```

which is:

```text
28
```

The bitwise Trie makes these opposite-bit choices efficiently.

---

# 42. Maximum XOR — Critical Insight

The algorithm is not:

```text
choose the numerically largest number
```

It is:

```text
at each highest bit:
    choose the branch that makes XOR bit = 1
```

This is a **lexicographic maximization of bits from MSB to LSB**.

That is why greedy works.

---

# 43. Pair Constraint: Avoid Using the Same Number

If the problem requires two distinct indices, do not blindly build a Trie containing the current number and query it if only one occurrence exists.

Safe patterns include:

### Pattern A

For each number:

```text
query against previously inserted numbers
then insert current number
```

This guarantees the partner comes from an earlier index.

```java
static int maximumXorDistinct(int[] nums) {
    if (nums.length < 2) {
        throw new IllegalArgumentException("Need at least two numbers");
    }

    BitNode root = new BitNode();
    insertNumber(root, nums[0]);

    int answer = 0;

    for (int i = 1; i < nums.length; i++) {
        answer = Math.max(
                answer,
                maxXorWith(root, nums[i])
        );

        insertNumber(root, nums[i]);
    }

    return answer;
}
```

This finds the maximum XOR among pairs of distinct positions.

---

# 44. Signed Integers and Bitwise Trie

For non-negative integers, using bits:

```text
30 ... 0
```

is straightforward.

For arbitrary Java `int` values, the sign bit at bit `31` changes signed numerical ordering.

For **maximum XOR as a bit pattern interpreted as a non-negative 32-bit value**, process:

```text
31 ... 0
```

using unsigned shifts:

```java
int b = (num >>> bit) & 1;
```

If the problem defines some other signed-order objective, adapt the representation carefully.

The key interview habit:

> **Clarify the integer domain before choosing the bit range.**

---

# 45. Bitwise Trie Applications

Bitwise Tries are useful for:

- maximum XOR pair
- maximum XOR with each query
- XOR queries under constraints
- minimum XOR
- finding numbers with desired XOR prefixes
- binary-number prefix grouping
- some bitwise optimization problems

The general pattern is:

```text
number
→ binary representation
→ Trie of 0/1 decisions
→ greedy bit-by-bit traversal
```

---

# 46. Maximum XOR With Queries

A common advanced variant:

> For each query `x`, find the maximum `x XOR num` among numbers satisfying `num <= m`.

A plain bitwise Trie is insufficient because the query has an additional constraint.

Use:

```text
sort numbers by value
sort queries by limit m
incrementally insert numbers <= m
query maximum XOR
```

This is:

```text
offline sorting + bitwise Trie
```

---

# 47. Offline Maximum XOR Pattern

Suppose numbers:

```text
[3, 5, 10, 25]
```

and queries:

```text
(x, m)
```

For each query:

```text
insert every number <= m
then maximize XOR with x
```

Because queries are processed in increasing `m`, each number is inserted only once.

Complexity:

```text
sorting = O(n log n + q log q)
Trie work = O((n + q)B)
```

where:

```text
B = number of processed bits
```

This is a high-value advanced Trie pattern.

---

# 48. Trie for Word Dictionary With Wildcards

Another classic problem:

```text
addWord("bad")
addWord("dad")
addWord("mad")
```

Then:

```text
search("pad") → false
search("bad") → true
search(".ad") → true
search("b..") → true
```

The `.` character can represent any character.

A Trie handles this by:

```text
normal character → follow one child
'.' → recursively try all existing children
```

---

# 49. Word Dictionary Java

```java
static class WordDictionary {
    static class Node {
        Node[] children = new Node[26];
        boolean isEnd;
    }

    private final Node root = new Node();

    void addWord(String word) {
        Node current = root;

        for (char ch : word.toCharArray()) {
            int index = ch - 'a';

            if (current.children[index] == null) {
                current.children[index] = new Node();
            }

            current = current.children[index];
        }

        current.isEnd = true;
    }

    boolean search(String word) {
        return dfs(root, word, 0);
    }

    private boolean dfs(
            Node node,
            String word,
            int index) {

        if (node == null) {
            return false;
        }

        if (index == word.length()) {
            return node.isEnd;
        }

        char ch = word.charAt(index);

        if (ch != '.') {
            return dfs(
                    node.children[ch - 'a'],
                    word,
                    index + 1
            );
        }

        for (Node child : node.children) {
            if (child != null
                    && dfs(child, word, index + 1)) {
                return true;
            }
        }

        return false;
    }
}
```

---

# 50. Word Dictionary Complexity

Without wildcards:

```text
O(L)
```

With wildcard characters:

Worst case can branch across the alphabet.

For alphabet size `A` and wildcard count `W`, the worst-case search can approach:

```text
O(A^W × L)
```

subject to the actual Trie structure.

For lowercase English:

```text
A = 26
```

The Trie still provides strong pruning when many prefixes do not exist.

---

# 51. Trie + Backtracking Pattern

Several Trie problems share this structure:

```text
Trie
 +
DFS / backtracking
```

Examples:

```text
autocomplete
word search
wildcard dictionary
generate words from prefix
lexicographic enumeration
```

General structure:

```java
void dfs(Node node, StringBuilder path) {
    if (node.isEnd) {
        // process word
    }

    for (each possible child) {
        if (child exists) {
            path.append(character);
            dfs(child, path);
            path.deleteCharAt(path.length() - 1);
        }
    }
}
```

Recognize this as a reusable template.

---

# 52. Lexicographic Ordering With Trie

If children are stored in:

```text
'a' → 'z'
```

order and DFS visits them in increasing order, terminal words are produced in lexicographic order.

Example:

```text
app
apple
apt
bat
```

The Trie itself provides a natural lexicographic traversal.

This is useful when a problem asks:

> Return all words in dictionary order.

---

# 53. Trie Prefix Counts

An advanced extension is storing:

```java
int prefixCount;
```

at each node.

When inserting:

```text
apple
```

increment the count along:

```text
a
ap
app
appl
apple
```

Then:

```text
prefixCount at "app"
```

can answer:

> How many inserted words start with `"app"`?

---

# 54. Prefix Count Example

Insert:

```text
apple
app
application
apt
```

At prefix:

```text
app
```

the count is:

```text
3
```

because:

```text
app
apple
application
```

start with `"app"`.

This can be implemented in:

```text
O(L)
```

per insertion and:

```text
O(P)
```

per prefix-count query.

---

# 55. Prefix Count + Delete

If supporting deletion:

```text
insert:
    increment prefixCount along path

delete:
    decrement prefixCount along path
```

Then a node's count tells how many currently active words pass through it.

Be careful to distinguish:

```text
prefixCount
```

from:

```text
isEnd
```

They represent different information.

---

# 56. Counting Words Equal to a String

If duplicates are allowed, a boolean:

```java
boolean isEnd;
```

is insufficient.

Use:

```java
int wordCount;
```

At the terminal node:

```text
wordCount = number of times the exact word was inserted
```

Then:

```text
countExact(word)
```

takes:

```text
O(L)
```

---

# 57. Counting Distinct Words

If duplicates should be ignored:

```text
isEnd = true
```

is enough.

If duplicates are meaningful:

```text
wordCount++
```

at the terminal node.

This distinction often appears in design-style interview questions.

---

# 58. Memory Optimization

A fixed array:

```java
Node[] children = new Node[26];
```

is simple but memory-heavy.

Suppose there are many Trie nodes and each node has only one child.

Each node still reserves space for:

```text
26 references
```

A `HashMap<Character, Node>` uses memory more proportional to actual branching, but has greater per-entry overhead and slower lookup constants.

For interviews:

```text
fixed array → default choice for lowercase a-z
HashMap → large/sparse alphabet
```

---

# 59. Compressed Trie / Radix Tree

A standard Trie stores one character per edge.

A **compressed Trie** or **Radix Tree** combines chains with no branching.

Instead of:

```text
c → a → r → e
```

it may store:

```text
"care"
```

on one edge when compression is possible.

Benefits:

```text
less memory
fewer nodes
```

Tradeoff:

```text
more complicated implementation
```

Usually not needed for standard LeetCode Trie problems, but useful conceptually for system-design discussions.

---

# 60. Trie vs BST

Both are tree structures, but they organize different information.

| Feature | Trie | BST |
|---|---|---|
| Primary data | Strings/prefixes | Ordered keys |
| Navigation | Character by character | Compare whole key |
| Prefix queries | Excellent | Requires range logic |
| Exact string search | O(L) | O(log n) balanced by key comparisons |
| Lexicographic enumeration | Natural | Natural for ordered string keys |
| Memory | Can be high | Usually lower per key |
| Shared prefixes | Yes | No structural sharing of prefixes |
| Bitwise variant | Yes | Different structure |

---

# 61. Trie vs HashMap

A `HashMap<String, Value>` gives direct exact-key lookup.

A Trie is better when queries depend on prefixes.

Example:

```text
HashMap:
"apple" → value

Trie:
a → p → p → ...
```

If asked:

```text
"How many words begin with app?"
```

a Trie is structurally aligned with the question.

---

# 62. Character Trie Complexity

Let:

```text
N = number of inserted words
L = average word length
S = total number of characters across all inserted words
```

Worst-case node count:

```text
O(S)
```

Insertion:

```text
O(word length)
```

Exact search:

```text
O(word length)
```

Prefix existence:

```text
O(prefix length)
```

Prefix enumeration:

```text
O(prefix length + explored subtree/output)
```

Memory:

```text
O(S × alphabet representation factor)
```

For fixed-array nodes, the practical memory constant can be large.

---

# 63. Bitwise Trie Complexity

Let:

```text
N = number of numbers
B = number of processed bits
```

Insert:

```text
O(B)
```

Maximum-XOR query:

```text
O(B)
```

For all numbers:

```text
O(NB)
```

Memory:

```text
O(NB)
```

For fixed 31/32-bit integers, `B` is constant with respect to `N`.

---

# 64. Important Trie Invariants

## Character Trie

At each node:

```text
children[c]
```

represents the next character.

`isEnd` means:

```text
a complete inserted word ends here
```

## Prefix Search

A prefix is valid if:

```text
every character has a corresponding path
```

## Exact Search

A word exists if:

```text
path exists
AND
terminal flag/count says it is a word
```

## Delete

Remove nodes only when:

```text
not terminal
AND
no children
```

## Autocomplete

```text
prefix node
→ enumerate descendants
```

## Word Search

```text
board path
→ Trie prefix
```

## Bitwise Trie

At every bit:

```text
prefer opposite bit
```

to maximize XOR.

---

# 65. Common Trie Bugs

## Bug 1 — Treating prefix as a complete word

If:

```text
apple
```

is inserted, this does not imply:

```text
app
```

was inserted.

Always check:

```java
isEnd
```

---

## Bug 2 — Incorrect character indexing

For lowercase letters:

```java
int index = ch - 'a';
```

Do not use arbitrary numeric character values as array indices.

---

## Bug 3 — Deleting shared nodes

For:

```text
app
apple
```

deleting `apple` must not destroy the path for `app`.

---

## Bug 4 — Forgetting backtracking

When collecting words:

```java
path.append(ch);
dfs(...);
path.deleteCharAt(path.length() - 1);
```

Without the final deletion, later words contain incorrect prefixes.

---

## Bug 5 — Wildcard search branching incorrectly

For `.`:

```text
try all existing children
```

not all 26 values regardless of whether a child exists.

---

## Bug 6 — Maximum XOR chooses the same bit

To maximize XOR:

```text
wanted = 1 - currentBit
```

The opposite bit is preferred.

---

## Bug 7 — Processing bits in the wrong direction

For maximum XOR, process:

```text
MSB → LSB
```

because higher-order bits dominate lower-order bits.

---

## Bug 8 — Wrong signed integer assumptions

Know whether the problem treats numbers as:

```text
non-negative integers
```

or:

```text
full signed 32-bit integers
```

before choosing bit `30` vs bit `31` and interpreting the result.

---

# 66. Interview Pattern Recognition

| Problem wording | Pattern |
|---|---|
| Implement Trie | Character Trie |
| Insert/search word | Trie path |
| Starts with prefix | Prefix traversal |
| Return words for prefix | Trie + DFS |
| Autocomplete | Trie + DFS + optional heap |
| Dictionary with `.` wildcard | Trie + backtracking |
| Search many words in board | Trie + grid DFS |
| Maximum XOR pair | Bitwise Trie |
| Maximum XOR under `num <= m` | Offline sorting + Bitwise Trie |
| Count words with prefix | Prefix counts |
| Count duplicate words | Terminal frequency |
| Lexicographic output | Trie DFS in character order |

---

# 67. Trie Decision Framework

When you see a problem:

```text
                String problem
                      |
              Prefix involved?
                /           \
              yes            no
               |              |
             Trie        consider HashMap/
                            DP/other
               |
        +------+------+---------+
        |      |      |         |
     exact   prefix  words     ranking
     search  query   output      |
                         |       |
                        DFS     Heap
```

For integer XOR:

```text
                XOR optimization
                       |
                binary representation
                       |
                 Bitwise Trie
                       |
                MSB → LSB greedy
```

---

# 68. Word Dictionary vs Trie

A standard Trie:

```text
search("apple")
```

follows exactly one path.

A wildcard dictionary:

```text
search("a..le")
```

must branch whenever it encounters:

```text
.
```

Therefore:

```text
Trie + DFS
```

is the correct mental model.

---

# 69. Autocomplete Design

A production-style autocomplete system may store at each Trie node:

```text
children
terminal
frequency
topSuggestions
```

Then querying:

```text
"app"
```

can be:

```text
navigate to "app"
→ return precomputed top suggestions
```

This can make query latency very small at the cost of additional memory and update complexity.

For interviews, first implement:

```text
Trie + DFS
```

Then discuss cached top-K suggestions if asked about scaling.

---

# 70. Trie With Cached Top-K Suggestions

At every prefix node, maintain the best `K` words.

On insertion:

```text
word
→ walk every prefix node
→ update its top-K list
```

Then:

```text
autocomplete(prefix)
```

becomes approximately:

```text
O(P + K)
```

for retrieving stored suggestions, assuming the top-K data is maintained appropriately.

Tradeoff:

```text
faster queries
vs
more expensive updates and more memory
```

This is a useful system-design extension.

---

# 71. Word Search Optimization

For multi-word board search:

```text
dictionary
→ build Trie
→ DFS from every cell
→ stop immediately when Trie path does not exist
```

This gives a strong pruning mechanism.

The key insight:

> **The Trie tells the board DFS whether the current path can possibly become a dictionary word.**

This is why the combination is much stronger than independent word searches.

---

# 72. Maximum XOR Optimization

For repeated maximum-XOR queries:

```text
build Bitwise Trie once
```

Then each query:

```text
O(B)
```

This is substantially better than comparing against every stored number:

```text
O(NB)
```

per query.

For `Q` queries:

Naive:

```text
O(NQB)
```

Trie:

```text
O(NB + QB)
```

assuming all stored numbers are eligible for all queries.

---

# 73. GATE / CS Theory

## 73.1 Trie Height

For a standard character Trie:

```text
height ≤ maximum stored word length
```

If the longest word has length `L`:

```text
height = O(L)
```

---

## 73.2 Trie Space

If total characters inserted are:

```text
S
```

the number of Trie nodes is at most:

```text
S + 1
```

including the root, ignoring possible node sharing that reduces the actual count.

Thus:

```text
O(S)
```

nodes.

The actual memory depends heavily on child representation.

---

## 73.3 Prefix Search

Searching for a prefix of length `P` requires at most:

```text
P
```

character transitions.

Therefore:

```text
O(P)
```

for prefix existence.

---

## 73.4 Lexicographic Traversal

If children are visited in character order:

```text
a → b → c → ... → z
```

DFS outputs stored words in lexicographic order.

---

## 73.5 Trie and Alphabet Size

For a fixed alphabet size `A`:

```text
array child lookup = O(1)
```

If `A` grows with the input, memory and operation constants must be considered.

A sparse map can reduce wasted child storage.

---

# 74. GATE-Style Solved Question 1 — Prefix vs Word

> **Question:** A Trie contains the word `"apple"` but the word `"app"` was never inserted. What are the results of:
>
> ```text
> search("app")
> startsWith("app")
> ```

### Solution

The path:

```text
a → p → p
```

exists because it is a prefix of `"apple"`.

Therefore:

```text
startsWith("app") = true
```

But the terminal flag at the second `p` is false because `"app"` itself was not inserted.

Therefore:

```text
search("app") = false
```

### Answer

```text
search("app")     = false
startsWith("app") = true
```

---

# 75. GATE-Style Solved Question 2 — Trie Complexity

> **Question:** A Trie stores `N` strings. The maximum length of any string is `L`. What is the worst-case time complexity of searching for one string?

### Solution

At most one Trie edge is followed per character.

Therefore:

```text
O(L)
```

The number of stored words `N` does not directly appear in the lookup complexity.

### Answer

```text
O(L)
```

assuming constant-time child access.

---

# 76. GATE-Style Solved Question 3 — Maximum XOR

> **Question:** Why does a bitwise Trie for maximum XOR process bits from the most significant bit toward the least significant bit?

### Solution

Suppose two candidate XOR results differ at the highest bit where they differ.

If one has:

```text
1
```

and the other has:

```text
0
```

at that bit, the first number is larger regardless of all lower bits.

Therefore maximizing XOR requires making the highest possible bit equal to `1` first.

For each bit:

```text
current = b
preferred = 1 - b
```

### Answer

```text
MSB → LSB
```

because higher-order bits dominate lower-order bits.

---

# 77. Additional GATE-Style Practice

## Question 4

A Trie stores:

```text
cat
car
dog
```

How many root-to-terminal paths correspond to complete words?

### Solution

Each inserted word has a terminal node.

Therefore:

```text
3
```

complete word paths exist.

Shared prefixes do not change the number of stored words.

### Answer

```text
3
```

---

## Question 5

If the maximum length of a word in a Trie is `L`, what is the maximum number of edges on a root-to-node path?

### Solution

At most one edge is consumed per character.

Therefore:

```text
L
```

### Answer

```text
L
```

---

## Question 6

For a lowercase English Trie using a 26-element child array, what is the child lookup complexity for one character?

### Solution

The character maps directly to an array index:

```java
children[ch - 'a']
```

Array indexing is constant time.

### Answer

```text
O(1)
```

---

# 78. LeetCode Roadmap

## Level 1 — Core Trie

1. Implement Trie
2. Implement Trie II / prefix counting
3. Search exact word
4. Prefix search
5. Word frequency / dictionary operations

## Level 2 — Interview Core

6. Design Add and Search Words Data Structure
7. Word Search II
8. Replace Words
9. Longest Word in Dictionary
10. Lexicographic Trie traversal
11. Prefix-based autocomplete

## Level 3 — Bitwise Trie

12. Maximum XOR of Two Numbers in an Array
13. Maximum XOR With an Element From Array
14. Maximum/minimum XOR query variants
15. Offline XOR queries with limits

### Practice order

```text
Basic Trie
→ insert/search
→ startsWith
→ delete
→ prefix enumeration
→ autocomplete
→ wildcard dictionary
→ Trie + backtracking
→ Trie + board DFS
→ Bitwise Trie
→ maximum XOR
→ constrained XOR queries
```

---

# 79. Java Cheat Sheet

## Node

```java
static class Node {
    Node[] children = new Node[26];
    boolean isEnd;
}
```

## Insert

```java
Node cur = root;

for (char ch : word.toCharArray()) {
    int i = ch - 'a';

    if (cur.children[i] == null) {
        cur.children[i] = new Node();
    }

    cur = cur.children[i];
}

cur.isEnd = true;
```

## Search

```java
Node cur = root;

for (char ch : word.toCharArray()) {
    cur = cur.children[ch - 'a'];

    if (cur == null) return false;
}

return cur.isEnd;
```

## Prefix

```java
return findNode(prefix) != null;
```

## Delete

```text
unmark terminal
→ remove unused suffix nodes
```

## Autocomplete

```text
prefix node
→ DFS descendants
```

## Wildcard

```text
normal char → one child
'.' → all existing children
```

## Maximum XOR

```text
bitwise Trie
→ MSB to LSB
→ prefer opposite bit
```

---

# 80. Mastery Checklist

You should be able to implement these without looking at notes.

### Trie fundamentals

- [ ] Explain what a Trie stores.
- [ ] Explain why common prefixes are shared.
- [ ] Implement a lowercase Trie.
- [ ] Implement insert.
- [ ] Implement exact search.
- [ ] Implement `startsWith`.
- [ ] Explain `isEnd`.
- [ ] Explain Trie time and space complexity.
- [ ] Compare fixed arrays vs HashMaps.

### Deletion

- [ ] Delete a leaf word.
- [ ] Delete a word that is a prefix of another word.
- [ ] Delete a word that has another word as its prefix.
- [ ] Delete shared suffix paths correctly.
- [ ] Explain when a Trie node can be removed.

### Prefix problems

- [ ] Find all words with a prefix.
- [ ] Generate lexicographic words.
- [ ] Count words with a prefix.
- [ ] Count exact duplicate words.
- [ ] Build basic autocomplete.
- [ ] Explain top-K autocomplete using a heap.

### Word problems

- [ ] Implement wildcard dictionary search.
- [ ] Understand Trie + DFS.
- [ ] Solve multi-word board search using Trie + backtracking.
- [ ] Explain prefix pruning.

### Bitwise Trie

- [ ] Represent an integer as bits.
- [ ] Implement 0/1 Trie insertion.
- [ ] Query maximum XOR.
- [ ] Explain why opposite bits are preferred.
- [ ] Process bits MSB → LSB.
- [ ] Handle distinct-pair constraints.
- [ ] Solve offline maximum-XOR queries with limits.
- [ ] Understand signed vs non-negative integer bit ranges.

### Theory

- [ ] Know Trie height.
- [ ] Know maximum node count.
- [ ] Know prefix-search complexity.
- [ ] Understand alphabet-size effects.
- [ ] Compare Trie vs HashMap.
- [ ] Compare Trie vs BST.
- [ ] Understand compressed Trie / Radix Tree conceptually.

---

# 81. Final Revision Sheet

```text
TRIE
----
Tree of characters.
Each path represents a prefix.

NODE
----
children[]
isEnd

INSERT
------
follow/create character path
mark terminal
O(L)

SEARCH
------
follow path
check isEnd
O(L)

PREFIX SEARCH
-------------
follow prefix path
do NOT require isEnd
O(P)

DELETE
------
unmark terminal
remove node only if:
    !isEnd
    AND no children

AUTOCOMPLETE
------------
prefix node
→ DFS descendants
→ optional frequency/top-K ranking

WORD DICTIONARY
---------------
exact word:
    Trie path + isEnd

wildcard:
    normal char → one child
    '.' → try all children

WORD SEARCH
-----------
board DFS
+
Trie prefix pruning

BITWISE TRIE
------------
each node has:
    child[0]
    child[1]

MAX XOR
-------
process MSB → LSB
current bit = b
prefer 1 - b

COMPLEXITY
----------
Character Trie:
insert/search = O(L)
prefix = O(P)

Bitwise Trie:
insert/query = O(B)
all-pairs maximum XOR pattern = O(NB)

SPACE
-----
Character Trie = O(total characters)
Bitwise Trie = O(NB)

CORE PATTERNS
-------------
Trie + DFS
Trie + Backtracking
Trie + Heap
Bitwise Trie + Greedy
Offline Sorting + Bitwise Trie
```

---

# 82. Final Mental Model

The entire topic can be reduced to two major structures.

## Character Trie

```text
                 Trie
                   |
          character-by-character
                   |
        +----------+----------+
        |          |          |
      exact      prefix     words
      search     search      output
        |          |          |
      isEnd       path       DFS
                              |
                         autocomplete
                         word search
                         lexicographic output
```

## Bitwise Trie

```text
              Integer
                 |
          binary representation
                 |
             0 / 1 Trie
                 |
           MSB → LSB
                 |
        prefer opposite bit
                 |
           maximum XOR
```

The most important principle is:

> **A Trie turns prefix information into structure. A character Trie exploits shared prefixes; a bitwise Trie exploits shared binary prefixes. Once you recognize the required prefix structure, the correct traversal usually becomes straightforward.**
