# 22. Advanced Data Structures

> **Goal:** Master the advanced data structures that repeatedly appear in LeetCode, competitive programming, and FAANG/top product-company interviews.
>
> **Language:** Java  
> **Difficulty target:** Easy → Medium → Hard, with Medium dominating.
>
> **Depth:** These structures do not need equal depth. The highest priority is **Segment Tree + Lazy Propagation + Fenwick Tree**, followed by Sparse Table and the recognition/use of Ordered Sets, Interval Trees, Heap, Trie, DSU, Monotonic Stack, and Monotonic Queue.
>
> **Core principle:** Learn each structure by the problem it solves. Do not memorize implementation details without understanding the operation/query trade-off.

---

# 1. Advanced Data Structures — Big Picture

A normal array gives:

```text
point access → O(1)
```

but some problems require repeated operations such as:

```text
range sum
range minimum
range maximum
point update
range update
dynamic order statistics
```

A naive implementation may require:

```text
O(n)
```

per query.

Advanced data structures reduce this to approximately:

```text
O(log n)
```

or:

```text
O(1)
```

for appropriate operations.

---

# 2. Structure Selection Guide

| Requirement | Data Structure |
|---|---|
| Range sum + point update | Fenwick Tree |
| Prefix sum + point update | Fenwick Tree |
| Range sum + range update | Segment Tree + Lazy |
| Range min/max + point update | Segment Tree |
| Range min/max static array | Sparse Table |
| Range query + arbitrary update | Segment Tree |
| K-th/order-statistic dynamic set | Ordered Tree / TreeMap-like structure |
| Overlapping interval search | Interval Tree concept |
| Repeated minimum/maximum extraction | Heap |
| Prefix/string lookup | Trie |
| Dynamic connectivity | DSU |
| Next/previous greater element | Monotonic Stack |
| Sliding-window maximum/minimum | Monotonic Queue |

---

# 3. Segment Tree

A segment tree is a binary tree built over an array.

Each node represents an interval:

```text
[l, r]
```

and stores an aggregate for that interval.

Common aggregates:

```text
sum
minimum
maximum
gcd
AND
OR
XOR
```

---

# 4. Segment Tree Example

Array:

```text
[2, 1, 5, 3]
```

A sum segment tree conceptually contains:

```text
                 [0,3] = 11
                /       \
          [0,1] = 3     [2,3] = 8
          /    \        /    \
       [0,0]  [1,1]  [2,2]  [3,3]
          2      1      5      3
```

Every internal node combines its children.

---

# 5. Why Segment Trees?

Without a segment tree:

```text
range sum → O(n)
point update → O(1)
```

With a segment tree:

```text
range sum → O(log n)
point update → O(log n)
```

This is useful when both:

```text
queries
+
updates
```

are frequent.

---

# 6. Segment Tree Operations

For a standard tree:

### Build

```text
O(n)
```

### Point update

```text
O(log n)
```

### Range query

```text
O(log n)
```

### Range update without lazy propagation

Can degrade to:

```text
O(n)
```

if every affected leaf is updated individually.

Lazy propagation fixes this for suitable operations.

---

# 7. Segment Tree Storage

A segment tree can be stored in an array.

For recursive implementations, a common allocation is:

```text
4n
```

positions.

Example:

```java
long[] tree = new long[4 * n];
```

This is not the only possible layout, but it is a standard safe choice for recursive segment trees.

---

# 8. Range Sum Segment Tree

For every node:

```text
tree[node]
=
tree[leftChild]
+
tree[rightChild]
```

The query recursively visits only the nodes needed to cover the requested interval.

---

# 9. Segment Tree — Build

```java
static void build(
        int node,
        int left,
        int right,
        int[] arr,
        long[] tree) {

    if (left == right) {
        tree[node] = arr[left];
        return;
    }

    int mid = left + (right - left) / 2;

    build(node * 2, left, mid, arr, tree);
    build(node * 2 + 1, mid + 1, right, arr, tree);

    tree[node] =
        tree[node * 2]
        + tree[node * 2 + 1];
}
```

---

# 10. Segment Tree — Range Sum Query

```java
static long query(
        int node,
        int left,
        int right,
        int ql,
        int qr,
        long[] tree) {

    if (qr < left || right < ql) {
        return 0;
    }

    if (ql <= left && right <= qr) {
        return tree[node];
    }

    int mid = left + (right - left) / 2;

    long leftSum =
        query(node * 2, left, mid,
              ql, qr, tree);

    long rightSum =
        query(node * 2 + 1, mid + 1, right,
              ql, qr, tree);

    return leftSum + rightSum;
}
```

The identity element for sum is:

```text
0
```

because:

```text
x + 0 = x
```

---

# 11. Segment Tree — Point Update

Suppose:

```text
arr[index] = newValue
```

Update the leaf and recompute all ancestors.

```java
static void update(
        int node,
        int left,
        int right,
        int index,
        int value,
        long[] tree) {

    if (left == right) {
        tree[node] = value;
        return;
    }

    int mid = left + (right - left) / 2;

    if (index <= mid) {
        update(node * 2, left, mid,
               index, value, tree);
    } else {
        update(node * 2 + 1, mid + 1, right,
               index, value, tree);
    }

    tree[node] =
        tree[node * 2]
        + tree[node * 2 + 1];
}
```

Complexity:

```text
O(log n)
```

---

# 12. Range Minimum Segment Tree

For minimum:

```java
tree[node] =
    Math.min(
        tree[node * 2],
        tree[node * 2 + 1]
    );
```

The query's identity must be:

```text
+infinity
```

because:

```text
min(x, +∞) = x
```

For integer arrays:

```java
long INF = Long.MAX_VALUE;
```

---

# 13. Range Maximum Segment Tree

For maximum:

```java
tree[node] =
    Math.max(
        tree[node * 2],
        tree[node * 2 + 1]
    );
```

The query identity is:

```text
-infinity
```

because:

```text
max(x, -∞) = x
```

---

# 14. Segment Tree — General Pattern

A segment tree works when the interval aggregate can be combined efficiently.

Examples:

```text
sum:
a + b

min:
min(a,b)

max:
max(a,b)

gcd:
gcd(a,b)

xor:
a ^ b
```

The combine operation should have an appropriate identity element for non-overlapping query segments.

---

# 15. Associativity Matters

For the usual segment-tree aggregation pattern, the combine operation should be associative:

```text
combine(combine(a,b),c)
=
combine(a,combine(b,c))
```

Examples:

```text
sum
min
max
gcd
xor
AND
OR
```

This lets the tree combine independently computed interval pieces.

---

# 16. Segment Tree Query Cases

For query interval:

```text
[ql, qr]
```

and node interval:

```text
[left, right]
```

there are three cases.

### 1. No overlap

```text
qr < left
or
right < ql
```

Return identity.

### 2. Complete overlap

```text
ql <= left
and
right <= qr
```

Return stored aggregate.

### 3. Partial overlap

Recurse into both children and combine.

This three-case model should become automatic.

---

# 17. Segment Tree — Range Minimum Query Template

```java
static long queryMin(
        int node,
        int left,
        int right,
        int ql,
        int qr,
        long[] tree) {

    if (qr < left || right < ql) {
        return Long.MAX_VALUE;
    }

    if (ql <= left && right <= qr) {
        return tree[node];
    }

    int mid = left + (right - left) / 2;

    return Math.min(
        queryMin(
            node * 2,
            left,
            mid,
            ql,
            qr,
            tree
        ),
        queryMin(
            node * 2 + 1,
            mid + 1,
            right,
            ql,
            qr,
            tree
        )
    );
}
```

---

# 18. Lazy Propagation

Lazy propagation is used when we need:

```text
range updates
+
range queries
```

efficiently.

Without lazy propagation, updating every element in a large interval may require:

```text
O(n)
```

time.

With lazy propagation, a whole segment can be updated in:

```text
O(log n)
```

by storing the pending update at the node instead of immediately pushing it to every descendant.

---

# 19. Core Lazy Idea

Suppose we want:

```text
add 5 to every element in [l,r]
```

and a segment-tree node represents exactly:

```text
[l,r]
```

Instead of visiting every leaf:

1. Update the node's aggregate.
2. Store a lazy value.
3. Push that lazy value to children only when their values are needed.

This is called:

```text
deferred propagation
```

---

# 20. Lazy Propagation — Range Add + Range Sum

For a node representing:

```text
[left, right]
```

with length:

```text
len = right - left + 1
```

adding:

```text
delta
```

to every element changes the segment sum by:

```text
delta × len
```

Therefore:

```java
tree[node] += delta * len;
lazy[node] += delta;
```

---

# 21. Apply Lazy Update

```java
static void apply(
        int node,
        int left,
        int right,
        long delta,
        long[] tree,
        long[] lazy) {

    long length = right - left + 1L;

    tree[node] += delta * length;
    lazy[node] += delta;
}
```

---

# 22. Push Lazy Value

When a node has a pending lazy value and we need to inspect its children:

```java
static void push(
        int node,
        int left,
        int right,
        long[] tree,
        long[] lazy) {

    if (lazy[node] == 0 || left == right) {
        return;
    }

    int mid = left + (right - left) / 2;
    long delta = lazy[node];

    apply(
        node * 2,
        left,
        mid,
        delta,
        tree,
        lazy
    );

    apply(
        node * 2 + 1,
        mid + 1,
        right,
        delta,
        tree,
        lazy
    );

    lazy[node] = 0;
}
```

---

# 23. Lazy Segment Tree — Range Update

```java
static void rangeAdd(
        int node,
        int left,
        int right,
        int ql,
        int qr,
        long delta,
        long[] tree,
        long[] lazy) {

    if (qr < left || right < ql) {
        return;
    }

    if (ql <= left && right <= qr) {
        apply(
            node,
            left,
            right,
            delta,
            tree,
            lazy
        );
        return;
    }

    push(node, left, right, tree, lazy);

    int mid = left + (right - left) / 2;

    rangeAdd(
        node * 2,
        left,
        mid,
        ql,
        qr,
        delta,
        tree,
        lazy
    );

    rangeAdd(
        node * 2 + 1,
        mid + 1,
        right,
        ql,
        qr,
        delta,
        tree,
        lazy
    );

    tree[node] =
        tree[node * 2]
        + tree[node * 2 + 1];
}
```

---

# 24. Lazy Segment Tree — Range Sum Query

```java
static long rangeSum(
        int node,
        int left,
        int right,
        int ql,
        int qr,
        long[] tree,
        long[] lazy) {

    if (qr < left || right < ql) {
        return 0;
    }

    if (ql <= left && right <= qr) {
        return tree[node];
    }

    push(node, left, right, tree, lazy);

    int mid = left + (right - left) / 2;

    return rangeSum(
               node * 2,
               left,
               mid,
               ql,
               qr,
               tree,
               lazy
           )
           +
           rangeSum(
               node * 2 + 1,
               mid + 1,
               right,
               ql,
               qr,
               tree,
               lazy
           );
}
```

Both:

```text
range add
range sum
```

run in:

```text
O(log n)
```

---

# 25. Lazy Propagation Example

Array:

```text
[1, 2, 3, 4]
```

Perform:

```text
add 5 to [0,3]
```

Instead of changing:

```text
1 → 6
2 → 7
3 → 8
4 → 9
```

individually, the root sum changes:

```text
10 → 30
```

because:

```text
10 + 5×4 = 30
```

and the node stores:

```text
lazy[root] = 5
```

The children do not need to be immediately updated.

---

# 26. Lazy Propagation Invariant

A useful invariant is:

> `tree[node]` already represents the correct aggregate for its entire segment, even if some of that update has not yet been pushed to descendants.

The lazy value means:

```text
children need to receive this pending transformation
```

not:

```text
current node is incorrect
```

This distinction prevents many implementation errors.

---

# 27. Range Assignment vs Range Addition

Range assignment:

```text
set every value in [l,r] to x
```

is more complicated than range addition.

A lazy structure generally needs:

```text
pending assignment flag/value
```

and potentially:

```text
pending addition
```

because assignment overrides previous values while addition composes with them.

When both operations exist, the order of lazy-tag composition matters.

---

# 28. Lazy Tag Composition

Suppose a segment has:

```text
set to 10
```

then:

```text
add 5
```

final result is:

```text
15
```

But:

```text
add 5
```

then:

```text
set to 10
```

final result is:

```text
10
```

Therefore lazy operations are not always commutative.

You must define exactly how tags compose.

---

# 29. Fenwick Tree / Binary Indexed Tree

A Fenwick Tree, or BIT, supports:

```text
prefix aggregate queries
+
point updates
```

efficiently.

For sums:

```text
update(index, delta)
query(index) → sum [1..index]
```

Both are:

```text
O(log n)
```

---

# 30. Fenwick Tree Structure

The key operation is:

```text
i += i & -i
```

for moving upward during an update.

And:

```text
i -= i & -i
```

for moving downward during a prefix query.

This uses the lowest set bit to determine the block represented by each Fenwick node.

---

# 31. Why Fenwick Trees Use 1-Based Indexing

Fenwick Trees are easiest with:

```text
index = 1..n
```

because:

```text
i & -i
```

must be nonzero for positive indices.

If the original array is zero-indexed, use:

```text
internalIndex = originalIndex + 1
```

---

# 32. Fenwick Tree — Point Update

```java
static void add(
        long[] bit,
        int n,
        int index,
        long delta) {

    for (int i = index;
         i <= n;
         i += i & -i) {

        bit[i] += delta;
    }
}
```

---

# 33. Fenwick Tree — Prefix Sum

```java
static long sum(
        long[] bit,
        int index) {

    long result = 0;

    for (int i = index;
         i > 0;
         i -= i & -i) {

        result += bit[i];
    }

    return result;
}
```

---

# 34. Fenwick Range Sum

To calculate:

```text
sum[l..r]
```

use:

```text
prefix(r) - prefix(l-1)
```

Therefore:

```java
static long rangeSum(
        long[] bit,
        int left,
        int right) {

    return sum(bit, right)
         - sum(bit, left - 1);
}
```

assuming one-based indices.

---

# 35. Fenwick Tree Class

```java
class FenwickTree {
    private final int n;
    private final long[] bit;

    FenwickTree(int n) {
        this.n = n;
        this.bit = new long[n + 1];
    }

    void add(int index, long delta) {
        for (int i = index;
             i <= n;
             i += i & -i) {

            bit[i] += delta;
        }
    }

    long sum(int index) {
        long result = 0;

        for (int i = index;
             i > 0;
             i -= i & -i) {

            result += bit[i];
        }

        return result;
    }

    long rangeSum(int left, int right) {
        return sum(right)
             - sum(left - 1);
    }
}
```

---

# 36. Fenwick Tree Complexity

| Operation | Complexity |
|---|---:|
| Point update | `O(log n)` |
| Prefix query | `O(log n)` |
| Range sum | `O(log n)` |
| Memory | `O(n)` |

Compared with a segment tree, Fenwick Tree is usually:

```text
simpler
smaller
faster constants
```

but less general.

---

# 37. Fenwick vs Segment Tree

| Feature | Fenwick | Segment Tree |
|---|---|---|
| Prefix sum | Excellent | Yes |
| Range sum | Yes | Yes |
| Point update | Yes | Yes |
| Range min/max | Limited / specialized | Yes |
| Range update | Possible with advanced variants | Natural with lazy |
| Lazy propagation | No | Yes |
| Implementation | Simple | More complex |
| Memory | `O(n)` | Usually `O(4n)` recursive |
| Constants | Low | Higher |

### Rule

If the problem is:

```text
sum + point update
```

try Fenwick first.

If it is:

```text
arbitrary range query/update
```

consider a segment tree.

---

# 38. Inversion Counting

An inversion is a pair:

```text
i < j
```

such that:

```text
a[i] > a[j]
```

Example:

```text
[3, 1, 2]
```

Inversions:

```text
(3,1)
(3,2)
```

so the answer is:

```text
2
```

---

# 39. Fenwick Tree for Inversion Counting

Coordinate compression is required when values are large or negative.

Process the array from right to left.

For current value `x`:

```text
number of smaller values already seen
```

is the number of previous ranks:

```text
1..rank(x)-1
```

Then:

```text
inversions += query(rank(x)-1)
```

Finally:

```text
update(rank(x), 1)
```

---

# 40. Coordinate Compression

Suppose:

```text
[100, -5, 1000, 20]
```

Sort unique values:

```text
-5
20
100
1000
```

Assign ranks:

```text
-5    → 1
20    → 2
100   → 3
1000  → 4
```

Only relative ordering matters for inversion counting.

---

# 41. Inversion Counting — Java

```java
static long countInversions(int[] nums) {
    int n = nums.length;

    int[] sorted = nums.clone();
    java.util.Arrays.sort(sorted);

    java.util.Map<Integer, Integer> rank =
        new java.util.HashMap<>();

    int nextRank = 1;

    for (int x : sorted) {
        if (!rank.containsKey(x)) {
            rank.put(x, nextRank++);
        }
    }

    FenwickTree bit =
        new FenwickTree(nextRank);

    long inversions = 0;

    for (int i = n - 1; i >= 0; i--) {
        int r = rank.get(nums[i]);

        inversions += bit.sum(r - 1);
        bit.add(r, 1);
    }

    return inversions;
}
```

Complexity:

```text
O(n log n)
```

because of sorting and Fenwick operations.

---

# 42. Why Inversion Count Needs `long`

Maximum inversions:

```text
n(n-1)/2
```

For large `n`, this can exceed `int`.

Therefore:

```java
long inversions
```

should be used.

---

# 43. Sparse Table

A Sparse Table is primarily for:

```text
static range queries
```

where the array does not change.

Typical operations:

```text
range minimum
range maximum
GCD
```

Preprocessing:

```text
O(n log n)
```

Query:

```text
O(1)
```

for idempotent operations such as:

```text
min
max
gcd
```

---

# 44. Why Sparse Table Works

Precompute:

```text
st[k][i]
```

representing the aggregate over:

```text
[i, i + 2^k - 1]
```

Base level:

```text
st[0][i] = a[i]
```

Transition:

```text
st[k][i]
=
combine(
    st[k-1][i],
    st[k-1][i + 2^(k-1)]
)
```

---

# 45. Sparse Table for Range Minimum

```java
class SparseTable {
    private final int[][] st;
    private final int[] log;

    SparseTable(int[] arr) {
        int n = arr.length;

        log = new int[n + 1];

        for (int i = 2; i <= n; i++) {
            log[i] = log[i / 2] + 1;
        }

        int levels = log[n] + 1;
        st = new int[levels][n];

        System.arraycopy(
            arr, 0, st[0], 0, n
        );

        for (int k = 1; k < levels; k++) {
            int len = 1 << k;
            int half = len >> 1;

            for (int i = 0;
                 i + len <= n;
                 i++) {

                st[k][i] =
                    Math.min(
                        st[k - 1][i],
                        st[k - 1][i + half]
                    );
            }
        }
    }

    int minQuery(int left, int right) {
        int len = right - left + 1;
        int k = log[len];

        return Math.min(
            st[k][left],
            st[k][right - (1 << k) + 1]
        );
    }
}
```

---

# 46. Why Sparse Table Query Is O(1)

For an idempotent operation such as minimum:

```text
min(x,x) = x
```

We can cover `[L,R]` using two overlapping blocks of equal power-of-two length:

```text
2^k <= length
```

The overlap does not change the result.

Therefore:

```text
answer =
min(left block, right block)
```

in constant time.

---

# 47. Sparse Table Limitations

Sparse Table is excellent when:

```text
array is static
```

but it is not the first choice when frequent updates occur.

If values change:

```text
segment tree
```

is generally more appropriate.

---

# 48. Ordered Set / Ordered Tree

An ordered set maintains elements in sorted order while supporting operations such as:

```text
insert
delete
search
predecessor
successor
```

Typical balanced BST implementations provide:

```text
O(log n)
```

operations.

---

# 49. Java Ordered Structures

Java's standard library includes:

```java
TreeSet
TreeMap
```

These are balanced search-tree-based structures.

Example:

```java
TreeSet<Integer> set =
    new TreeSet<>();

set.add(10);
set.add(20);
set.add(30);

Integer lower = set.lower(25);
Integer higher = set.higher(25);
Integer floor = set.floor(25);
Integer ceil = set.ceiling(25);
```

---

# 50. Ordered Set Operations

Important Java methods:

```java
add(x)
remove(x)
contains(x)

lower(x)
floor(x)
ceiling(x)
higher(x)

first()
last()
```

These provide sorted-neighbor queries.

---

# 51. Important Ordered-Set Limitation

Java `TreeSet` does not directly provide:

```text
k-th smallest
rank of x
```

in `O(log n)`.

For those operations, competitive-programming solutions may use:

```text
order-statistics tree
```

or:

```text
Fenwick tree
Segment tree
Treap
```

depending on constraints.

---

# 52. Ordered Tree Concept

An ordered-statistics tree augments a balanced BST with subtree sizes.

Each node stores:

```text
key
left
right
subtreeSize
```

Then:

```text
rank(x)
```

and:

```text
k-th smallest
```

can be supported in:

```text
O(log n)
```

under a balanced-tree guarantee.

---

# 53. Interval Tree Concepts

An interval tree stores intervals:

```text
[l, r]
```

and supports efficient overlap-related queries.

Typical question:

```text
Which stored intervals overlap [L,R]?
```

A common augmentation stores:

```text
max endpoint
```

for each subtree.

---

# 54. Interval Overlap

Two closed intervals:

```text
[a,b]
[c,d]
```

overlap if:

```text
max(a,c) <= min(b,d)
```

Equivalently:

```text
a <= d
and
c <= b
```

for closed intervals.

Be careful about whether the problem considers touching endpoints as overlap.

---

# 55. Interval Tree Search Concept

At a node with interval:

```text
[l,r]
```

and subtree maximum right endpoint:

```text
maxRight
```

If the current interval does not overlap the query and:

```text
left child's maxRight < queryLeft
```

then no interval in that left subtree can overlap the query.

This allows large parts of the tree to be skipped.

---

# 56. Interval Tree vs Merge Intervals

Do not confuse:

```text
offline interval merging
```

with:

```text
dynamic interval overlap queries
```

If all intervals are known once:

```text
sort + merge
```

is often simpler.

If intervals are dynamically inserted/deleted and overlap queries are frequent:

```text
interval-tree concepts
```

become relevant.

---

# 57. Heap

A heap is a complete binary tree satisfying a heap-order property.

### Min-heap

```text
parent <= children
```

### Max-heap

```text
parent >= children
```

The root is the minimum or maximum element.

---

# 58. Java PriorityQueue

Java's:

```java
PriorityQueue<Integer>
```

is a min-heap by default.

```java
PriorityQueue<Integer> minHeap =
    new PriorityQueue<>();
```

For max-heap:

```java
PriorityQueue<Integer> maxHeap =
    new PriorityQueue<>(
        java.util.Collections.reverseOrder()
    );
```

---

# 59. Heap Complexity

| Operation | Complexity |
|---|---:|
| Peek | `O(1)` |
| Insert | `O(log n)` |
| Remove root | `O(log n)` |
| Build heap | `O(n)` |
| Arbitrary search | `O(n)` |

---

# 60. Heap Recognition

Think heap when the problem repeatedly asks for:

```text
smallest
largest
top k
next event
best available item
priority
```

Typical patterns:

- Kth largest/smallest
- Top K frequent
- Merge K sorted lists
- Meeting scheduling
- Task scheduling
- Dijkstra
- Median using two heaps

---

# 61. Trie

A Trie stores strings by prefixes.

For:

```text
apple
app
ape
```

the shared prefix:

```text
app
```

is represented once.

Common operations:

```text
insert
search
prefix search
```

---

# 62. Trie Complexity

For word length:

```text
L
```

operations are typically:

```text
O(L)
```

assuming fixed alphabet access.

This is independent of the number of stored words in the basic asymptotic model.

---

# 63. Trie Java Template

```java
class Trie {
    static class Node {
        Node[] next = new Node[26];
        boolean word;
    }

    private final Node root = new Node();

    void insert(String s) {
        Node cur = root;

        for (char c : s.toCharArray()) {
            int idx = c - 'a';

            if (cur.next[idx] == null) {
                cur.next[idx] = new Node();
            }

            cur = cur.next[idx];
        }

        cur.word = true;
    }

    boolean search(String s) {
        Node node = find(s);
        return node != null && node.word;
    }

    boolean startsWith(String prefix) {
        return find(prefix) != null;
    }

    private Node find(String s) {
        Node cur = root;

        for (char c : s.toCharArray()) {
            int idx = c - 'a';

            if (cur.next[idx] == null) {
                return null;
            }

            cur = cur.next[idx];
        }

        return cur;
    }
}
```

---

# 64. DSU

Disjoint Set Union, also called Union-Find, maintains a partition of elements into disjoint sets.

Core operations:

```text
find(x)
union(a,b)
```

With:

```text
path compression
+
union by size/rank
```

amortized complexity is:

```text
O(alpha(n))
```

per operation, effectively constant for practical constraints.

---

# 65. DSU Recognition

Think DSU when the problem asks about:

- connected components under unions
- dynamic connectivity
- cycle detection in undirected graphs
- Kruskal's MST
- grouping entities
- merging accounts
- redundant connections

---

# 66. Monotonic Stack

A monotonic stack maintains elements in monotonic order.

It is commonly used for:

```text
next greater element
next smaller element
previous greater element
previous smaller element
```

---

# 67. Next Greater Element

For:

```text
[2,1,2,4,3]
```

for each element, find the first greater element to its right.

Use a decreasing stack of candidate values/indices.

---

## Java

```java
static int[] nextGreater(int[] nums) {
    int n = nums.length;
    int[] answer = new int[n];

    java.util.Arrays.fill(answer, -1);

    java.util.ArrayDeque<Integer> stack =
        new java.util.ArrayDeque<>();

    for (int i = 0; i < n; i++) {

        while (!stack.isEmpty()
                && nums[stack.peek()] < nums[i]) {

            int index = stack.pop();
            answer[index] = nums[i];
        }

        stack.push(i);
    }

    return answer;
}
```

Complexity:

```text
O(n)
```

because each index is pushed and popped at most once.

---

# 68. Monotonic Queue

A monotonic queue maintains candidate values in monotonic order while supporting:

```text
insert
remove expired elements
read current maximum/minimum
```

It is especially useful for:

```text
sliding-window maximum/minimum
```

---

# 69. Sliding Window Maximum

For:

```text
nums
```

and window size:

```text
k
```

maintain a decreasing deque of indices.

The front is always the maximum for the current window.

---

## Java

```java
static int[] maxSlidingWindow(
        int[] nums,
        int k) {

    int n = nums.length;

    if (n == 0 || k == 0) {
        return new int[0];
    }

    int[] answer = new int[n - k + 1];

    java.util.ArrayDeque<Integer> deque =
        new java.util.ArrayDeque<>();

    for (int i = 0; i < n; i++) {

        while (!deque.isEmpty()
                && deque.peekFirst() <= i - k) {
            deque.pollFirst();
        }

        while (!deque.isEmpty()
                && nums[deque.peekLast()] <= nums[i]) {
            deque.pollLast();
        }

        deque.offerLast(i);

        if (i >= k - 1) {
            answer[i - k + 1] =
                nums[deque.peekFirst()];
        }
    }

    return answer;
}
```

Complexity:

```text
O(n)
```

---

# 70. Monotonic Stack vs Queue

| Structure | Typical use |
|---|---|
| Monotonic stack | nearest greater/smaller |
| Monotonic queue | sliding-window extrema |

Both rely on:

```text
discarding dominated candidates
```

---

# 71. Segment Tree vs Fenwick — Interview Decision Tree

### Need only:

```text
prefix sum + point update
```

Use:

```text
Fenwick
```

### Need:

```text
range sum + point update
```

Use:

```text
Fenwick
```

or:

```text
Segment Tree
```

Fenwick is usually simpler.

### Need:

```text
range minimum/maximum + point update
```

Use:

```text
Segment Tree
```

### Need:

```text
range update + range query
```

Use:

```text
Lazy Segment Tree
```

### Array is static and query is min/max/GCD

Consider:

```text
Sparse Table
```

---

# 72. Sparse Table vs Segment Tree

| Requirement | Sparse Table | Segment Tree |
|---|---|---|
| Static array | Excellent | Good |
| Point update | No | Yes |
| Range min | Excellent | Yes |
| Range max | Excellent | Yes |
| Range GCD | Excellent | Yes |
| O(1) idempotent query | Yes | No |
| O(log n) update | No | Yes |
| Memory | `O(n log n)` | `O(n)` |

---

# 73. Heap vs Segment Tree

Use a heap for:

```text
repeated global min/max
```

Use a segment tree for:

```text
range-based min/max
```

Example:

```text
"give me the minimum among all currently available tasks"
```

→ heap may fit.

```text
"give me the minimum in indices [l,r]"
```

→ segment tree/sparse table.

---

# 74. Trie vs HashMap

Use a hash map when:

```text
exact lookup
```

is the primary operation.

Use a trie when:

```text
prefix
+
string structure
```

matters.

Example:

```text
startsWith("app")
```

is naturally handled by a Trie.

---

# 75. DSU vs Graph Traversal

Use DFS/BFS when you need:

```text
explore a graph
```

Use DSU when you mainly need:

```text
merge components
+
ask whether two elements belong to the same component
```

For a sequence of union operations, DSU is often substantially simpler.

---

# 76. Segment Tree — Other Useful Aggregates

A segment tree can store:

```text
sum
min
max
gcd
xor
AND
OR
custom associative aggregate
```

But a complicated aggregate may require storing more information per node.

Example:

For maximum subarray sum, a node can store:

```text
sum
bestPrefix
bestSuffix
bestSubarray
```

Then two children can be merged.

---

# 77. Maximum Subarray Segment Tree Node

For each segment store:

```text
sum
prefix
suffix
best
```

Merge:

```text
sum =
left.sum + right.sum

prefix =
max(left.prefix,
    left.sum + right.prefix)

suffix =
max(right.suffix,
    right.sum + left.suffix)

best =
max(
    left.best,
    right.best,
    left.suffix + right.prefix
)
```

This illustrates a key segment-tree skill:

> Design a node summary that contains exactly the information required to merge two adjacent segments.

---

# 78. Segment Tree Node Design

When designing a custom segment tree, ask:

1. What must a node answer?
2. Can two child summaries produce the parent summary?
3. What is the identity element for a missing segment?
4. How does an update transform the node?
5. If lazy propagation is needed, how do update tags compose?

If these cannot be answered, the structure is not yet fully designed.

---

# 79. Lazy Propagation Design Checklist

For every lazy operation define:

### Node state

What aggregate is stored?

### Lazy tag

What pending operation is stored?

### Apply

How does the update change the aggregate?

### Push

How does the pending operation transfer to children?

### Merge

How are child aggregates combined?

### Composition

What happens when two updates arrive before a push?

This framework is more important than memorizing a single lazy implementation.

---

# 80. Fenwick Tree — Why It Works

For index:

```text
i
```

the value:

```text
i & -i
```

gives the size of the block represented by that Fenwick node.

Example:

```text
i = 12
1100₂

i & -i = 0100₂ = 4
```

So the node covers a block of length:

```text
4
```

ending at index `12`.

---

# 81. Fenwick Prefix Query Intuition

Starting from:

```text
i
```

we repeatedly subtract:

```text
i & -i
```

This jumps through disjoint blocks that exactly partition:

```text
[1..i]
```

Therefore the prefix sum is obtained in:

```text
O(log n)
```

jumps.

---

# 82. Fenwick Update Intuition

Starting from:

```text
i
```

we repeatedly add:

```text
i & -i
```

This visits every Fenwick node whose represented block contains the updated position.

Therefore a point update affects only:

```text
O(log n)
```

nodes.

---

# 83. Fenwick Tree for Frequency Tables

A Fenwick tree does not have to store sums of arbitrary numeric values.

It can store:

```text
frequency of each rank
```

This enables:

- inversion counting
- frequency prefix queries
- rank queries
- dynamic order statistics in bounded coordinate domains

For example:

```text
add(rank, 1)
```

means:

```text
one occurrence of this rank exists
```

---

# 84. Fenwick for K-th Element

If the Fenwick tree stores frequencies, we can find the smallest index whose prefix frequency reaches:

```text{k}
```

using binary lifting over the Fenwick structure.

This supports:

```text
k-th smallest
```

in approximately:

```text
O(log n)
```

for a compressed/bounded rank domain.

This is a powerful extension beyond ordinary prefix sums.

---

# 85. Coordinate Compression — General Pattern

Use coordinate compression when:

```text
values are huge
but only relative order matters
```

Steps:

1. Copy values.
2. Sort.
3. Remove duplicates.
4. Assign ranks.
5. Replace original values by ranks.

Common with:

```text
Fenwick
Segment Tree
inversion counting
order statistics
interval processing
```

---

# 86. Segment Tree Build Complexity

A segment tree has approximately:

```text
2n - 1
```

logical nodes for a full binary decomposition, although array implementations commonly allocate:

```text
4n
```

for convenience.

Every input element participates in a constant number of merge operations.

Therefore:

```text
build = O(n)
```

---

# 87. Segment Tree Query Complexity

At each level, only a constant number of boundary segments are partially explored.

The height is:

```text
O(log n)
```

Therefore a standard range query is:

```text
O(log n)
```

not:

```text
O(n)
```

---

# 88. Why Lazy Propagation Is Logarithmic

A range update can fully cover large tree nodes.

Instead of descending to every leaf, mark those complete nodes lazily.

Only the boundary portions of the range require recursive descent.

Across the tree's logarithmic height, this gives:

```text
O(log n)
```

for standard range-add/range-query implementations.

---

# 89. Advanced Segment Tree Variants

Know the concepts even if you do not memorize every implementation:

- range add + range sum
- range assignment + range sum
- range add + range minimum
- range add + range maximum
- range XOR
- maximum subarray
- merge-sort tree
- persistent segment tree
- dynamic segment tree
- segment tree beats

These are lower-priority than the standard segment tree and lazy propagation.

---

# 90. Merge Sort Tree Concept

A merge sort tree stores a sorted collection at each segment-tree node.

It can answer queries such as:

```text
how many values <= x occur in [l,r]?
```

Each visited node can binary-search its sorted list.

Typical complexity:

```text
O(log² n)
```

per query.

This is an advanced extension rather than a first-choice structure.

---

# 91. Persistent Segment Tree Concept

A persistent segment tree preserves previous versions after updates.

Instead of copying the entire tree:

```text
copy only nodes along the update path
```

Each update creates:

```text
O(log n)
```

new nodes.

Useful for:

- historical queries
- k-th number in ranges
- versioned arrays
- offline range-order statistics

This is a high-level concept to know for advanced interviews.

---

# 92. Dynamic Segment Tree Concept

When the coordinate range is enormous:

```text
[0, 10^18]
```

but only a small number of positions are actually touched, do not allocate a tree for every coordinate.

Create nodes dynamically only for visited segments.

This is called a:

```text
dynamic segment tree
```

---

# 93. Advanced Data Structure Complexity Table

| Structure | Main operation | Typical complexity |
|---|---|---:|
| Segment Tree | Range query | `O(log n)` |
| Segment Tree | Point update | `O(log n)` |
| Lazy Segment Tree | Range update | `O(log n)` |
| Lazy Segment Tree | Range query | `O(log n)` |
| Fenwick | Prefix query | `O(log n)` |
| Fenwick | Point update | `O(log n)` |
| Sparse Table | Static RMQ | `O(1)` query |
| Sparse Table | Preprocessing | `O(n log n)` |
| TreeSet | Insert/search/delete | `O(log n)` |
| Heap | Insert/remove root | `O(log n)` |
| Heap | Peek | `O(1)` |
| Trie | Insert/search | `O(L)` |
| DSU | Union/find | `O(alpha(n))` amortized |
| Monotonic Stack | Full scan | `O(n)` |
| Monotonic Queue | Full scan | `O(n)` |

---

# 94. GATE Theory — Segment Trees

A segment tree:

- represents intervals hierarchically
- has logarithmic height
- supports range queries efficiently
- supports point updates in `O(log n)`
- can support range updates with lazy propagation

For an array of `n` elements, common recursive storage:

```text
O(n)
```

often implemented as:

```text
4n
```

array slots.

---

# 95. GATE Theory — Fenwick Tree

Fenwick Tree stores partial prefix information.

Core operations:

```text
i += i & -i
i -= i & -i
```

both require:

```text
O(log n)
```

iterations.

It is particularly efficient for:

```text
prefix sums
+
point updates
```

---

# 96. GATE Theory — Sparse Table

Sparse Table:

```text
preprocessing = O(n log n)
```

and for idempotent operations such as minimum:

```text
query = O(1)
```

The array must be static for the standard constant-time-query formulation.

---

# 97. GATE Theory — Heap

A binary heap is a complete binary tree.

For `n` elements:

```text
height = O(log n)
```

Therefore:

```text
insert = O(log n)
delete root = O(log n)
peek = O(1)
```

Building a heap bottom-up is:

```text
O(n)
```

---

# 98. GATE Theory — Trie

For maximum key length:

```text
L
```

basic Trie operations are:

```text
O(L)
```

assuming constant-time child access.

Trie performance depends on:

```text
alphabet size
number of nodes
key length
```

---

# 99. GATE Theory — DSU

With:

```text
path compression
+
union by rank/size
```

the amortized cost is:

```text
O(alpha(n))
```

where `alpha` is the inverse Ackermann function.

For practical input sizes:

```text
alpha(n)
```

is extremely small.

---

# 100. GATE Theory — Monotonic Stack

A monotonic stack can process:

```text
next greater/smaller
```

in:

```text
O(n)
```

because each element is:

```text
pushed at most once
popped at most once
```

This is an important amortized-analysis example.

---

# 101. GATE Theory — Monotonic Queue

A monotonic deque supports sliding-window extrema in:

```text
O(n)
```

because each element enters and leaves the deque at most once.

The algorithm is therefore:

```text
amortized O(1) per element
```

and:

```text
O(n)
```

overall.

---

# 102. GATE-Style Solved Questions

> These are **GATE-style practice questions**, not claimed as verbatim official GATE PYQs.

---

## Question 1 — Segment Tree

An array contains `n` elements. A standard segment tree is built for range-sum queries and point updates. What is the asymptotic complexity of each operation?

### Solution

The tree height is:

```text
O(log n)
```

A point update changes one leaf and its ancestors:

```text
O(log n)
```

A range query visits only a logarithmic number of boundary paths plus fully covered nodes:

```text
O(log n)
```

### Answer

```text
Build: O(n)
Range query: O(log n)
Point update: O(log n)
```

---

## Question 2 — Lazy Propagation

A segment tree node represents `100` elements and stores their sum. A range update adds `7` to every element in this node's interval. By how much should the stored sum increase?

### Solution

There are:

```text
100
```

elements.

Each increases by:

```text
7
```

Therefore total increase:

```text
100 × 7 = 700
```

### Answer

```text
700
```

The lazy value should record the pending:

```text
+7
```

update.

---

## Question 3 — Fenwick Tree

What is the purpose of:

```java
i += i & -i;
```

in a Fenwick Tree update?

### Solution

```text
i & -i
```

isolates the lowest set bit of `i`.

Adding it moves to the next Fenwick node whose stored range contains the updated position.

The process visits all relevant ancestors.

### Answer

```text
It moves from a Fenwick node to the next node
that must incorporate the point update.
```

---

## Question 4 — Fenwick Inversion Count

For:

```text
[3, 1, 2]
```

how many inversions exist?

### Solution

Pairs:

```text
(3,1) → inversion
(3,2) → inversion
(1,2) → not inversion
```

Therefore:

```text
2
```

A Fenwick solution can process from right to left:

```text
2 → add rank
1 → query smaller → 0
3 → query smaller → 2
```

### Answer

```text
2
```

---

## Question 5 — Sparse Table

A static array has `n` elements. A Sparse Table is built for range minimum queries. What are the preprocessing and query complexities?

### Solution

The table contains approximately:

```text
n log n
```

states.

Therefore:

```text
preprocessing = O(n log n)
```

For RMQ, two overlapping power-of-two blocks can be combined using:

```text
min
```

because minimum is idempotent.

Thus:

```text
query = O(1)
```

### Answer

```text
O(n log n) preprocessing
O(1) query
```

---

## Question 6 — Monotonic Stack

What is the total time complexity of finding the next greater element for all elements using a monotonic stack?

### Solution

Each element:

```text
is pushed once
```

and:

```text
is popped at most once
```

Therefore total stack operations are:

```text
O(n)
```

### Answer

```text
O(n)
```

---

# 103. LeetCode Roadmap — Priority 1

## Segment Tree

Master:

1. Range Sum Query — Mutable
2. Range Minimum Query
3. Range Maximum Query
4. Point update + range query
5. Range update + range query
6. Lazy propagation
7. Maximum subarray segment tree
8. Custom associative segment-tree nodes

Focus on:

```text
build
query
update
merge
identity
lazy
push
```

---

# 104. LeetCode Roadmap — Priority 2

## Fenwick Tree

Master:

1. Prefix sum + update
2. Range sum + point update
3. Inversion Count
4. Coordinate compression + BIT
5. Frequency/rank queries
6. K-th element using Fenwick
7. Prefix frequency problems

Focus on:

```text
i += i & -i
i -= i & -i
```

---

# 105. LeetCode Roadmap — Priority 3

## Sparse Table

Master:

1. Static RMQ
2. Static range maximum
3. Static GCD
4. Idempotent range queries
5. Binary-lifting-style preprocessing intuition

Focus on:

```text
st[k][i]
```

and:

```text
log[length]
```

---

# 106. LeetCode Roadmap — Priority 4

## Ordered Structures

Master:

- TreeSet
- TreeMap
- lower/floor/ceiling/higher
- predecessor/successor
- coordinate compression
- order-statistics concept

---

# 107. LeetCode Roadmap — Priority 5

## Heap

Master:

- Kth largest
- Kth smallest
- Top K
- Merge K sorted sequences
- Task scheduling
- Dijkstra
- Two heaps / median
- Custom comparators

---

# 108. LeetCode Roadmap — Priority 6

## Trie

Master:

- insert
- search
- prefix search
- Word Dictionary
- Word Search II
- maximum XOR trie
- autocomplete-style prefix queries

---

# 109. LeetCode Roadmap — Priority 7

## DSU

Master:

- connected components
- Number of Provinces
- Redundant Connection
- Accounts Merge
- Kruskal
- dynamic connectivity
- grid connectivity

---

# 110. LeetCode Roadmap — Priority 8

## Monotonic Stack

Master:

- Next Greater Element
- Daily Temperatures
- Stock Span
- Largest Rectangle in Histogram
- maximal rectangle
- previous/next smaller
- contribution techniques

---

# 111. LeetCode Roadmap — Priority 9

## Monotonic Queue

Master:

- Sliding Window Maximum
- Sliding Window Minimum
- shortest/longest constrained subarray patterns
- deque optimization of DP

---

# 112. Segment Tree Problem Recognition

Think:

```text
segment tree
```

when the problem says:

```text
many range queries
+
many updates
```

Especially when the query is:

```text
sum
min
max
gcd
xor
```

and updates prevent static preprocessing.

---

# 113. Fenwick Problem Recognition

Think:

```text
Fenwick
```

when you see:

```text
prefix sum
+
point update
```

or:

```text
frequency
+
rank
+
inversion
```

The simplest valid data structure is usually the best one.

---

# 114. Sparse Table Problem Recognition

Think:

```text
Sparse Table
```

when:

```text
array is static
+
many range min/max/GCD queries
```

and query speed matters.

---

# 115. Ordered Set Problem Recognition

Think:

```text
TreeSet / ordered set
```

when you need:

```text
closest smaller
closest larger
predecessor
successor
dynamic sorted values
```

---

# 116. Heap Problem Recognition

Think:

```text
heap
```

when repeatedly extracting:

```text
minimum
maximum
highest priority
```

is the central operation.

---

# 117. Trie Problem Recognition

Think:

```text
Trie
```

when:

```text
prefixes matter
```

or when strings/bit sequences are compared from:

```text
most significant / first character
```

A binary Trie can also solve maximum XOR problems.

---

# 118. DSU Problem Recognition

Think:

```text
DSU
```

when the graph is changing through:

```text
union operations
```

and the key question is:

```text
are x and y connected?
```

---

# 119. Monotonic Stack Recognition

Think:

```text
monotonic stack
```

when the problem asks:

```text
first greater to left/right
first smaller to left/right
nearest greater/smaller
```

or has:

```text
histogram
span
contribution
```

structure.

---

# 120. Monotonic Queue Recognition

Think:

```text
monotonic deque
```

when:

```text
sliding window
+
maximum/minimum
```

appears.

It removes values that can never become the answer again.

---

# 121. Dominance Principle

The central idea behind monotonic structures:

Suppose a newer candidate is:

```text
better
```

and will remain valid longer than an older candidate.

Then the older candidate can be discarded.

This is why:

```text
monotonic stack
monotonic queue
```

achieve linear complexity.

---

# 122. Amortized Complexity

Do not say:

```text
one pop = O(1)
```

therefore:

```text
n operations = O(n)
```

without justification.

The real argument is:

> Each element can be inserted once and removed at most once.

Therefore the total number of removals is at most:

```text
n
```

and total work is:

```text
O(n)
```

This is amortized analysis.

---

# 123. Advanced Data Structure Selection Table

| Query/update pattern | Preferred structure |
|---|---|
| Static range min | Sparse Table |
| Static range max | Sparse Table |
| Static range GCD | Sparse Table |
| Dynamic range min | Segment Tree |
| Dynamic range max | Segment Tree |
| Dynamic range sum | Fenwick or Segment Tree |
| Range update + range query | Lazy Segment Tree |
| Prefix frequency | Fenwick |
| Inversions | Fenwick + compression |
| Dynamic predecessor | TreeSet |
| Dynamic successor | TreeSet |
| Global min/max extraction | Heap |
| Prefix strings | Trie |
| Connectivity under unions | DSU |
| Next greater | Monotonic Stack |
| Sliding max | Monotonic Queue |

---

# 124. Common Mistakes

## Segment Tree

- Wrong interval boundaries
- Wrong identity element
- Forgetting to recompute parent
- Incorrect midpoint
- Mixing inclusive and exclusive ranges
- Forgetting lazy push
- Applying lazy update to the wrong segment length

---

## Lazy Propagation

- Updating children but not parent
- Forgetting to clear lazy tag
- Wrong tag composition
- Treating assignment and addition as interchangeable
- Returning stale child values without pushing when necessary

---

## Fenwick Tree

- Using zero index directly
- Forgetting `index + 1`
- Writing `i += i & -i` incorrectly
- Writing `i -= i & -i` incorrectly
- Using `int` for inversion count

---

## Sparse Table

- Trying to use it for frequently changing data
- Incorrect logarithm table
- Wrong block boundaries
- Assuming every operation supports O(1) overlapping-block queries

The standard O(1) query technique relies on an idempotent operation.

---

## TreeSet

- Assuming it supports duplicates
- Assuming it supports k-th element directly
- Confusing `lower` with `floor`
- Confusing `higher` with `ceiling`

---

## Heap

- Assuming arbitrary search is O(log n)
- Using a min-heap when max behavior is required
- Incorrect custom comparator
- Overflow in comparator subtraction

Avoid:

```java
(a, b) -> a - b
```

when overflow is possible.

Prefer:

```java
Integer.compare(a, b)
```

---

## Trie

- Wrong character indexing
- Excessive memory for huge alphabets
- Forgetting terminal-word state
- Mixing prefix existence with complete-word existence

---

## DSU

- Forgetting path compression
- Forgetting union by size/rank
- Incorrect parent initialization
- Counting components incorrectly after union

---

## Monotonic Stack/Queue

- Storing values when indices are needed
- Forgetting expired window indices
- Choosing increasing vs decreasing order incorrectly
- Mishandling equal values

---

# 125. Java Comparator Overflow

This is dangerous:

```java
Arrays.sort(
    arr,
    (a, b) -> a - b
);
```

because:

```text
a - b
```

can overflow.

Use:

```java
Arrays.sort(
    arr,
    (a, b) -> Integer.compare(a, b)
);
```

or:

```java
Long.compare(a, b)
```

for `long`.

This is especially important in:

```text
heaps
TreeSet/TreeMap comparators
sorting interval endpoints
```

---

# 126. Segment Tree Memory

A recursive segment tree commonly uses:

```java
new long[4 * n]
```

and lazy propagation may require:

```java
new long[4 * n]
```

again for lazy values.

Therefore memory is:

```text
O(n)
```

but constants are higher than Fenwick.

For very large `n`, iterative segment trees may be more memory-efficient.

---

# 127. Iterative Segment Tree Concept

An iterative segment tree often stores leaves starting around:

```text
n
```

in an array of size:

```text
2n
```

for power-of-two-friendly formulations.

Point update:

```text
update leaf
move upward
```

Range query:

```text
move boundaries inward
```

This can provide a compact and fast implementation for point updates + range queries.

---

# 128. Iterative Segment Tree — Sum

```java
class IterativeSegTree {
    private final int n;
    private final long[] tree;

    IterativeSegTree(int[] arr) {
        n = arr.length;
        tree = new long[2 * n];

        for (int i = 0; i < n; i++) {
            tree[n + i] = arr[i];
        }

        for (int i = n - 1; i > 0; i--) {
            tree[i] =
                tree[i * 2]
                + tree[i * 2 + 1];
        }
    }

    void update(int index, long value) {
        int p = n + index;
        tree[p] = value;

        while (p > 1) {
            p >>= 1;
            tree[p] =
                tree[p * 2]
                + tree[p * 2 + 1];
        }
    }

    long query(int left, int right) {
        long result = 0;

        for (left += n, right += n;
             left <= right;
             left >>= 1, right >>= 1) {

            if ((left & 1) == 1) {
                result += tree[left++];
            }

            if ((right & 1) == 0) {
                result += tree[right--];
            }
        }

        return result;
    }
}
```

This uses inclusive query bounds.

---

# 129. Advanced Priority

For this topic, prioritize learning in this order:

```text
1. Segment Tree
2. Lazy Propagation
3. Fenwick Tree
4. Sparse Table
5. Monotonic Stack
6. Monotonic Queue
7. Heap
8. Trie
9. DSU
10. Ordered Set / Tree
11. Interval Tree concepts
```

You should still recognize all structures, but implementation depth should be highest for the first three.

---

# 130. Final Mastery Checklist

## Segment Tree

- [ ] Understand interval decomposition
- [ ] Build
- [ ] Range sum
- [ ] Range minimum
- [ ] Range maximum
- [ ] Point update
- [ ] Query overlap cases
- [ ] Identity elements
- [ ] Merge function
- [ ] Iterative segment tree
- [ ] Custom node summaries

## Lazy Propagation

- [ ] Understand deferred updates
- [ ] Range add
- [ ] Range sum
- [ ] Apply
- [ ] Push
- [ ] Lazy invariant
- [ ] Tag composition
- [ ] Range assignment concept

## Fenwick

- [ ] 1-based indexing
- [ ] Prefix query
- [ ] Point update
- [ ] Range sum
- [ ] `i & -i`
- [ ] Inversion counting
- [ ] Coordinate compression
- [ ] Frequency queries
- [ ] K-th element concept

## Sparse Table

- [ ] Static-array requirement
- [ ] `st[k][i]`
- [ ] Power-of-two intervals
- [ ] Log table
- [ ] O(1) RMQ
- [ ] Idempotent operations

## Ordered Structures

- [ ] TreeSet
- [ ] TreeMap
- [ ] lower
- [ ] floor
- [ ] ceiling
- [ ] higher
- [ ] predecessor/successor
- [ ] Order-statistics concept

## Heap

- [ ] Min-heap
- [ ] Max-heap
- [ ] PriorityQueue
- [ ] Custom comparator
- [ ] Top K
- [ ] K-way merge
- [ ] Two heaps

## Trie

- [ ] Insert
- [ ] Search
- [ ] Prefix search
- [ ] Word termination
- [ ] Binary Trie concept

## DSU

- [ ] Parent
- [ ] Find
- [ ] Union
- [ ] Path compression
- [ ] Union by size/rank
- [ ] Dynamic connectivity

## Monotonic Stack

- [ ] Increasing stack
- [ ] Decreasing stack
- [ ] Next greater
- [ ] Next smaller
- [ ] Previous greater
- [ ] Previous smaller
- [ ] Histogram

## Monotonic Queue

- [ ] Deque
- [ ] Sliding maximum
- [ ] Sliding minimum
- [ ] Expired indices
- [ ] Dominated candidates
- [ ] Amortized O(n)

---

# 131. Final Revision Sheet

```text
SEGMENT TREE

Build:
O(n)

Range query:
O(log n)

Point update:
O(log n)

Range update:
O(log n) with lazy propagation

Node:
interval aggregate

Query cases:
no overlap
complete overlap
partial overlap

Lazy:
store pending update
push only when needed


FENWICK

Point update:
i += i & -i

Prefix query:
i -= i & -i

Both:
O(log n)

Range sum:
prefix(r)-prefix(l-1)

Inversions:
coordinate compression
+
right-to-left
+
query(rank-1)


SPARSE TABLE

Static array

Preprocess:
O(n log n)

RMQ query:
O(1)

Useful:
min/max/GCD


ORDERED SET

TreeSet:
lower
floor
ceiling
higher

Operations:
O(log n)


HEAP

Peek:
O(1)

Insert:
O(log n)

Remove root:
O(log n)

Build:
O(n)


TRIE

Insert/search:
O(L)

Best for:
prefix problems


DSU

find + union:
near O(1) amortized

Use:
dynamic connectivity


MONOTONIC STACK

Next/previous greater/smaller

Total:
O(n)


MONOTONIC QUEUE

Sliding min/max

Total:
O(n)


COORDINATE COMPRESSION

sort
→ unique
→ rank


KEY DECISION

prefix sum + point update
→ Fenwick

range query + point update
→ Fenwick or Segment Tree

range update + range query
→ Lazy Segment Tree

static RMQ
→ Sparse Table

predecessor/successor
→ TreeSet

global min/max
→ Heap

prefix strings
→ Trie

union/connectivity
→ DSU

next greater/smaller
→ Monotonic Stack

sliding max/min
→ Monotonic Queue
```

---

# 132. Final Mastery Standard

You have mastered **Advanced Data Structures** when you can:

1. Recognize when an array requires more than `O(n)` range processing.
2. Build a standard segment tree from scratch.
3. Implement range sum.
4. Implement range minimum/maximum.
5. Implement point updates.
6. Explain the three segment-tree overlap cases.
7. Choose correct identity values.
8. Design a custom node merge operation.
9. Implement lazy range-add + range-sum.
10. Explain the lazy invariant.
11. Explain lazy-tag composition.
12. Build a Fenwick Tree from scratch.
13. Explain `i & -i`.
14. Perform prefix queries and point updates.
15. Convert prefix sums into range sums.
16. Count inversions with Fenwick + coordinate compression.
17. Explain when Fenwick is preferable to a segment tree.
18. Build and query a Sparse Table for static RMQ.
19. Explain why Sparse Table can answer idempotent queries in `O(1)`.
20. Use `TreeSet` for predecessor/successor problems.
21. Know the difference between a heap and a range-query structure.
22. Implement a Trie and recognize prefix problems.
23. Use DSU for connectivity/merging problems.
24. Recognize monotonic-stack problems immediately.
25. Recognize monotonic-queue sliding-window problems.
26. Explain why monotonic structures are `O(n)` amortized.
27. Use coordinate compression when values are large but ordering matters.
28. Understand the concepts of persistent, dynamic, and merge-sort segment trees.
29. Choose the **simplest data structure that satisfies the required operations**.

> **Core principle:**  
> **Advanced data structures are not about using the most complicated structure. They are about matching the operation pattern to the cheapest structure: Fenwick for simple prefix aggregates, Segment Tree for flexible dynamic ranges, Lazy Segment Tree for deferred range updates, Sparse Table for static idempotent queries, and monotonic structures when dominated candidates can be discarded.**
