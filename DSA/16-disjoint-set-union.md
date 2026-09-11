# 16 — Disjoint Set Union (DSU)

> **Goal:** Master Disjoint Set Union for FAANG/top-product SDE and ML/AI engineering interviews in Java, including implementation, connectivity, cycle detection, Kruskal's MST, and dynamic connectivity.

---

# 1. What Is Disjoint Set Union?

Disjoint Set Union (DSU), also called **Union-Find**, maintains a collection of **disjoint sets**.

It supports two fundamental operations:

1. **Find(x)** — determine which set/component contains `x`.
2. **Union(a, b)** — merge the sets containing `a` and `b`.

Example:

```text
Initially:

{0} {1} {2} {3} {4}

union(0,1)

{0,1} {2} {3} {4}

union(1,2)

{0,1,2} {3} {4}
```

The central question DSU answers is:

> Are two vertices currently in the same connected component?

---

# 2. Why DSU Matters

DSU is especially useful when edges are added over time and the main requirement is connectivity.

Common applications:

- Connected components
- Dynamic connectivity
- Cycle detection
- Kruskal's MST
- Redundant connection
- Network connectivity
- Merging groups
- Number of provinces
- Account/group merging
- Offline connectivity
- Image/grid component merging

DSU is usually **not** the first choice for:

- Shortest paths
- Traversal order
- Topological sorting
- Enumerating paths

For those, use graph traversal or shortest-path algorithms.

---

# 3. Core DSU Model

Each element has a parent.

```text
parent[x]
```

Initially:

```text
parent[x] = x
```

So every element is its own representative.

Example:

```text
parent:

0  1  2  3  4
|  |  |  |  |
0  1  2  3  4
```

After merging `0` and `1`:

```text
1 → 0
```

Possible parent array:

```text
parent = [0, 0, 2, 3, 4]
```

After merging `1` and `2`:

```text
2 → 0
```

Possible parent array:

```text
parent = [0, 0, 0, 3, 4]
```

The exact root chosen can vary depending on the union strategy.

---

# 4. Parent Array

The simplest DSU structure is:

```java
int[] parent;
```

Initialization:

```java
parent = new int[n];

for (int i = 0; i < n; i++) {
    parent[i] = i;
}
```

Invariant:

```text
parent[root] == root
```

Every node eventually points to a root.

---

# 5. Naive Find

The simplest `find` follows parent pointers until reaching a root.

```java
int find(int x) {
    while (parent[x] != x) {
        x = parent[x];
    }

    return x;
}
```

Example:

```text
3 → 2 → 1 → 0
```

Calling:

```java
find(3)
```

returns:

```text
0
```

---

# 6. Naive Union

To merge two sets:

```java
void union(int a, int b) {
    int ra = find(a);
    int rb = find(b);

    if (ra != rb) {
        parent[rb] = ra;
    }
}
```

This is logically correct.

However, without balancing, the parent tree can become very deep.

---

# 7. Why Naive DSU Can Become Slow

Suppose unions create:

```text
9 → 8 → 7 → 6 → 5 → 4 → 3 → 2 → 1 → 0
```

Then:

```text
find(9)
```

takes:

\[
O(n)
\]

in the worst case.

Repeated operations can therefore become expensive.

We need two optimizations:

1. Path compression.
2. Union by rank or union by size.

---

# 8. Recursive Find

A common implementation is:

```java
int find(int x) {
    if (parent[x] != x) {
        parent[x] = find(parent[x]);
    }

    return parent[x];
}
```

This does two things:

1. Finds the root.
2. Makes `x` point directly to the root.

That second step is **path compression**.

---

# 9. Path Compression

Suppose:

```text
        0
        |
        1
        |
        2
        |
        3
        |
        4
```

Before:

```text
4 → 3 → 2 → 1 → 0
```

Call:

```java
find(4)
```

After path compression:

```text
4 → 0
3 → 0
2 → 0
1 → 0
```

The structure becomes much flatter.

---

# 10. Path Compression Code

```java
int find(int x) {
    if (parent[x] != x) {
        parent[x] = find(parent[x]);
    }

    return parent[x];
}
```

The important line is:

```java
parent[x] = find(parent[x]);
```

not merely:

```java
find(parent[x]);
```

The assignment performs the compression.

---

# 11. Iterative Path Compression

For very deep structures, an iterative implementation can avoid recursion.

One simple two-pass version:

```java
int find(int x) {
    int root = x;

    while (parent[root] != root) {
        root = parent[root];
    }

    while (parent[x] != x) {
        int next = parent[x];
        parent[x] = root;
        x = next;
    }

    return root;
}
```

For normal interview constraints, recursive `find` is usually sufficient when union by rank/size is also used.

---

# 12. Union by Rank

Rank is an approximate measure of tree height.

Rule:

> Attach the root of the lower-rank tree below the root of the higher-rank tree.

If ranks are equal, choose one root and increment its rank.

---

## 12.1 Initialization

```java
int[] rank = new int[n];

Arrays.fill(rank, 0);
```

---

## 12.2 Union by Rank

```java
boolean union(int a, int b) {
    int ra = find(a);
    int rb = find(b);

    if (ra == rb) {
        return false;
    }

    if (rank[ra] < rank[rb]) {
        parent[ra] = rb;
    } else if (rank[ra] > rank[rb]) {
        parent[rb] = ra;
    } else {
        parent[rb] = ra;
        rank[ra]++;
    }

    return true;
}
```

---

# 13. Why Rank Works

Without balancing:

```text
0
|
1
|
2
|
3
|
4
```

With rank balancing, trees tend to remain shallow.

The rank is **not necessarily the exact current height** after path compression.

Think of rank as a balancing heuristic.

---

# 14. Union by Size

Instead of rank, maintain:

```java
size[root]
```

representing the number of elements in the set.

Rule:

> Attach the smaller component below the larger component.

---

## 14.1 Initialization

```java
int[] size = new int[n];

Arrays.fill(size, 1);
```

---

## 14.2 Union by Size

```java
boolean union(int a, int b) {
    int ra = find(a);
    int rb = find(b);

    if (ra == rb) {
        return false;
    }

    if (size[ra] < size[rb]) {
        int temp = ra;
        ra = rb;
        rb = temp;
    }

    parent[rb] = ra;
    size[ra] += size[rb];

    return true;
}
```

---

# 15. Rank vs Size

| Feature | Union by Rank | Union by Size |
|---|---|---|
| Stored information | Rank | Component size |
| Balancing | Approximate height | Number of elements |
| Path compression compatible | Yes | Yes |
| Can know component size directly | No | Yes |
| Interview usefulness | High | High |

### Practical choice

Use **union by size** when the problem also asks for component sizes.

Otherwise either method is fine.

---

# 16. Complete DSU — Union by Size

This is a strong default Java template.

```java
static class DSU {
    private final int[] parent;
    private final int[] size;

    DSU(int n) {
        parent = new int[n];
        size = new int[n];

        for (int i = 0; i < n; i++) {
            parent[i] = i;
            size[i] = 1;
        }
    }

    int find(int x) {
        if (parent[x] != x) {
            parent[x] = find(parent[x]);
        }

        return parent[x];
    }

    boolean union(int a, int b) {
        int ra = find(a);
        int rb = find(b);

        if (ra == rb) {
            return false;
        }

        if (size[ra] < size[rb]) {
            int temp = ra;
            ra = rb;
            rb = temp;
        }

        parent[rb] = ra;
        size[ra] += size[rb];

        return true;
    }

    int componentSize(int x) {
        return size[find(x)];
    }
}
```

---

# 17. Complete DSU — Union by Rank

```java
static class DSU {
    private final int[] parent;
    private final int[] rank;

    DSU(int n) {
        parent = new int[n];
        rank = new int[n];

        for (int i = 0; i < n; i++) {
            parent[i] = i;
        }
    }

    int find(int x) {
        if (parent[x] != x) {
            parent[x] = find(parent[x]);
        }

        return parent[x];
    }

    boolean union(int a, int b) {
        int ra = find(a);
        int rb = find(b);

        if (ra == rb) {
            return false;
        }

        if (rank[ra] < rank[rb]) {
            parent[ra] = rb;
        } else if (rank[ra] > rank[rb]) {
            parent[rb] = ra;
        } else {
            parent[rb] = ra;
            rank[ra]++;
        }

        return true;
    }
}
```

---

# 18. DSU Invariants

These are more important than memorizing the implementation.

### Invariant 1

Every root satisfies:

```text
parent[root] = root
```

### Invariant 2

Following parent pointers eventually reaches a root.

### Invariant 3

All vertices in the same component have the same root.

### Invariant 4

Different roots represent different components.

### Invariant 5

`union(a,b)` changes connectivity only when:

```text
find(a) != find(b)
```

---

# 19. DSU Complexity

With:

- Path compression
- Union by rank

or:

- Path compression
- Union by size

the amortized complexity per operation is:

\[
O(\alpha(V))
\]

where:

\[
\alpha(V)
\]

is the inverse Ackermann function.

For all practical input sizes:

```text
α(V) is effectively a tiny constant.
```

A sequence of `M` operations costs approximately:

\[
O(M\alpha(V))
\]

---

# 20. Why Both Optimizations Matter

### Path compression

Makes existing paths flatter.

### Union by rank/size

Prevents tall trees from being created in the first place.

Together they produce extremely efficient DSU operations.

You should use both in serious interview solutions unless there is a specific reason not to.

---

# 21. Counting Connected Components

Suppose:

```text
n = 6
edges =

0 — 1
1 — 2
3 — 4
```

Components:

```text
{0,1,2}
{3,4}
{5}
```

Answer:

```text
3
```

---

## 21.1 DSU approach

Initialize:

```java
int components = n;
```

For every edge:

```java
if (dsu.union(u, v)) {
    components--;
}
```

At the end:

```java
return components;
```

---

# 22. Component Counting Template

```java
static int countComponents(
        int n,
        int[][] edges
) {
    DSU dsu = new DSU(n);
    int components = n;

    for (int[] edge : edges) {
        if (dsu.union(edge[0], edge[1])) {
            components--;
        }
    }

    return components;
}
```

Complexity:

\[
O((V+E)\alpha(V))
\]

which is effectively near-linear.

---

# 23. Why Successful Union Decreases Components

Initially:

\[
C=V
\]

because every vertex is isolated.

A successful union combines two different components:

\[
C\leftarrow C-1
\]

A union between vertices already in the same component changes nothing.

Therefore:

```text
successful unions = V - final component count
```

This is a highly reusable DSU invariant.

---

# 24. Cycle Detection with DSU

Consider an undirected graph.

Process edges one by one.

For edge:

```text
u — v
```

### Case 1

```text
find(u) != find(v)
```

The endpoints belong to different components.

Adding the edge cannot create a cycle.

Union them.

### Case 2

```text
find(u) == find(v)
```

There is already a path between `u` and `v`.

Adding another edge between them creates a cycle.

---

# 25. DSU Cycle Detection

```java
static boolean hasCycle(
        int n,
        int[][] edges
) {
    DSU dsu = new DSU(n);

    for (int[] edge : edges) {
        int u = edge[0];
        int v = edge[1];

        if (!dsu.union(u, v)) {
            return true;
        }
    }

    return false;
}
```

---

# 26. Why DSU Detects a Cycle

Suppose:

```text
0 — 1 — 2
```

Then:

```text
find(0) == find(2)
```

because `0` and `2` are already connected.

Adding:

```text
0 — 2
```

creates:

```text
0 — 1 — 2
 \_______/
```

Therefore the failed union identifies a cycle-forming edge.

---

# 27. DSU Cycle Detection vs DFS

| Situation | Prefer |
|---|---|
| Graph already represented as adjacency list | DFS is natural |
| Input is an edge stream/list | DSU is convenient |
| Need actual traversal/path | DFS/BFS |
| Need only connectivity | DSU |
| Dynamic edge additions | DSU |

DSU is especially elegant when the input arrives as edges.

---

# 28. Kruskal's Minimum Spanning Tree

Kruskal is one of the most important applications of DSU.

Given a connected weighted undirected graph:

```text
1 --4-- 2
|      /
2    1
|  /
3
```

Kruskal chooses edges from smallest weight to largest while avoiding cycles.

---

# 29. Kruskal Algorithm

### Step 1

Put all edges in an edge list.

### Step 2

Sort edges by increasing weight.

### Step 3

Initialize DSU.

### Step 4

Process each edge:

```text
if endpoints are in different components:
    add edge to MST
    union endpoints
else:
    skip edge
```

### Step 5

Stop after:

\[
V-1
\]

edges have been selected.

---

# 30. Kruskal Example

Edges:

```text
A-B = 4
A-C = 2
B-C = 1
B-D = 3
C-D = 5
```

Sort:

```text
B-C = 1
A-C = 2
B-D = 3
A-B = 4
C-D = 5
```

Process:

### Edge B-C = 1

Different components.

Add.

```text
{B,C}
```

### Edge A-C = 2

Different components.

Add.

```text
{A,B,C}
```

### Edge B-D = 3

Different components.

Add.

```text
{A,B,C,D}
```

We now have:

\[
V-1=3
\]

edges.

MST cost:

\[
1+2+3=6
\]

---

# 31. Kruskal Java

```java
static class WeightedEdge {
    int u;
    int v;
    int w;

    WeightedEdge(int u, int v, int w) {
        this.u = u;
        this.v = v;
        this.w = w;
    }
}
```

Algorithm:

```java
static long kruskal(
        int n,
        List<WeightedEdge> edges
) {
    edges.sort(
        Comparator.comparingInt(e -> e.w)
    );

    DSU dsu = new DSU(n);

    long total = 0;
    int used = 0;

    for (WeightedEdge e : edges) {
        if (dsu.union(e.u, e.v)) {
            total += e.w;
            used++;

            if (used == n - 1) {
                break;
            }
        }
    }

    if (used != n - 1) {
        return -1;
    }

    return total;
}
```

Use `long` for total weight when the sum may exceed `int`.

---

# 32. Kruskal Complexity

Sorting:

\[
O(E\log E)
\]

DSU operations:

\[
O(E\alpha(V))
\]

Overall:

\[
O(E\log E)
\]

Sorting dominates.

---

# 33. Why Kruskal Produces an MST

Kruskal repeatedly chooses the cheapest edge that connects two currently separate components.

By the **cut property**, the lightest edge crossing an appropriate cut is safe for an MST.

Skipping edges whose endpoints are already connected prevents cycles.

After `V-1` successful unions, the selected edges form a spanning tree with minimum possible total weight.

---

# 34. Kruskal vs Prim

Both solve the MST problem but grow the solution differently.

### Kruskal

Thinks in terms of:

```text
global edge ordering
```

Use:

```text
sort edges + DSU
```

### Prim

Thinks in terms of:

```text
expand current tree
```

Use:

```text
priority queue
```

---

# 35. Kruskal vs Prim Table

| Property | Kruskal | Prim |
|---|---|---|
| Main idea | Smallest global edges | Smallest frontier edge |
| Main DS | DSU | Priority queue |
| Input convenience | Edge list | Adjacency list |
| Sorting | Yes | Not necessarily all edges |
| Cycle prevention | DSU | Visited |
| Typical complexity | \(O(E\log E)\) | \(O(E\log V)\) |

---

# 36. Dynamic Connectivity

Dynamic connectivity asks questions such as:

> After adding these edges, are `u` and `v` connected?

Example:

```text
Initially:

0   1   2   3
```

Operations:

```text
add(0,1)
query(0,1) → true

add(1,2)
query(0,2) → true

query(0,3) → false
```

DSU is ideal when connectivity changes through **edge additions**.

---

# 37. Dynamic Connectivity Template

```java
DSU dsu = new DSU(n);

for (Operation op : operations) {
    if (op.type == ADD) {
        dsu.union(op.u, op.v);
    } else if (op.type == QUERY) {
        boolean connected =
                dsu.find(op.u) == dsu.find(op.v);
    }
}
```

---

# 38. DSU Is Naturally Incremental

DSU efficiently handles:

```text
add edge
ask connectivity
```

It does not naturally support:

```text
remove edge
```

because removing an edge can split a component and DSU does not maintain enough information to undo arbitrary historical unions.

For deletion-heavy dynamic connectivity, more advanced techniques are required.

---

# 39. Offline Connectivity

Some problems give all operations in advance.

This may allow DSU to be used by:

- Processing edges in reverse.
- Ignoring deletions initially.
- Adding edges when they become active in reverse time.
- Answering connectivity queries at the correct reversed state.

This is a major competitive-programming pattern.

---

# 40. Reverse Processing Example

Suppose operations are:

```text
add(0,1)
query(0,1)
remove(0,1)
query(0,1)
```

Forward deletion is difficult for ordinary DSU.

If the problem can be reformulated in reverse, removals become additions.

Reverse:

```text
query
add(0,1)
query
```

DSU can handle the additions.

---

# 41. Network Connectivity

A classic problem:

> Connect `n` computers using existing cables, where a cable can be moved.

Necessary condition:

\[
E\ge n-1
\]

If fewer than `n-1` cables exist, connection is impossible.

Otherwise, if there are `C` connected components, minimum cable moves:

\[
C-1
\]

---

# 42. DSU Network Connectivity

```java
static int makeConnected(
        int n,
        int[][] connections
) {
    if (connections.length < n - 1) {
        return -1;
    }

    DSU dsu = new DSU(n);

    int components = n;

    for (int[] edge : connections) {
        if (dsu.union(edge[0], edge[1])) {
            components--;
        }
    }

    return components - 1;
}
```

---

# 43. Redundant Connection

Given a graph that started as a tree and received one extra edge.

The extra edge creates a cycle.

DSU solution:

```java
static int[] findRedundantConnection(
        int[][] edges
) {
    int n = edges.length;
    DSU dsu = new DSU(n + 1);

    for (int[] edge : edges) {
        if (!dsu.union(edge[0], edge[1])) {
            return edge;
        }
    }

    return new int[0];
}
```

The first failed union identifies an edge whose endpoints were already connected.

---

# 44. Number of Provinces

Suppose the input is an adjacency matrix.

For every pair of connected cities:

```java
if (isConnected[i][j] == 1) {
    dsu.union(i, j);
}
```

Then count distinct roots.

Alternatively, maintain a component counter and decrement after successful unions.

---

# 45. Accounts Merge Pattern

Suppose accounts contain emails:

```text
Account A:
a@mail.com
b@mail.com

Account B:
b@mail.com
c@mail.com
```

Because `b@mail.com` occurs in both accounts, they belong to the same group.

Map each email to an ID.

Then union emails appearing in the same account.

After processing all accounts:

```text
same DSU root
→ same merged account
```

The key pattern is:

> Convert arbitrary entities into integer IDs, then run DSU.

---

# 46. DSU with Metadata

DSU can maintain information about each component.

Examples:

- Component size
- Sum
- Minimum
- Maximum
- Number of active nodes
- Custom aggregate

When merging:

```text
metadata[newRoot]
=
combine(metadata[rootA], metadata[rootB])
```

---

# 47. DSU with Component Size

With union by size:

```java
int componentSize(int x) {
    return size[find(x)];
}
```

This is useful for:

- Largest connected component
- Size after each union
- Connectivity statistics

---

# 48. DSU with Component Sum

Suppose every vertex has a value.

Maintain:

```java
long[] sum;
```

Initially:

```java
sum[i] = value[i];
```

When merging:

```java
sum[ra] += sum[rb];
```

after choosing `ra` as the new root.

This generalizes DSU beyond simple connectivity.

---

# 49. DSU with Component Minimum

Maintain:

```java
int[] min;
```

On merge:

```java
min[ra] = Math.min(min[ra], min[rb]);
```

The metadata must always live at the current representative/root.

---

# 50. A Generic DSU Mental Model

Think of each component as a set with:

```text
representative
    +
metadata
```

`find(x)` tells you:

```text
which set?
```

`union(a,b)` performs:

```text
merge set A + set B
```

Any aggregate that can be merged efficiently can potentially be stored at the root.

---

# 51. DSU and Connected Components

For a static undirected graph:

### DFS/BFS

Good when you need to:

- Visit vertices.
- Explore neighbors.
- Construct traversal.
- Inspect paths.

### DSU

Good when you need to:

- Process edges.
- Merge components.
- Answer connectivity.
- Count components.

Both can compute connected components, but their strengths differ.

---

# 52. DSU vs DFS/BFS

| Requirement | DSU | DFS/BFS |
|---|---|---|
| Connectivity | Excellent | Excellent |
| Edge additions | Excellent | Recompute/maintain separately |
| Traversal | Poor fit | Excellent |
| Path discovery | No | Yes |
| Cycle detection | Excellent for edge list | Excellent |
| Shortest path | No | BFS/Dijkstra |
| Components | Excellent | Excellent |
| Dynamic merging | Excellent | Less convenient |

---

# 53. DSU for Grid Connectivity

A grid can also use DSU.

Map cell:

\[
(r,c)
\]

to integer:

\[
id=r\cdot C+c
\]

For every pair of adjacent active cells:

```java
dsu.union(id1, id2);
```

This is useful when the problem involves **incrementally activating cells**.

---

# 54. Number of Islands II Pattern

Suppose cells become land one at a time.

After each activation:

1. Create the new land component.
2. Increase island count.
3. Check four neighbors.
4. If a neighboring cell is already land:
   - Union the two cells.
   - If the union succeeds, decrement island count.

This gives an efficient incremental solution.

---

# 55. Grid-to-DSU Mapping

For:

```text
rows × cols
```

use:

```java
int id = r * cols + c;
```

Reverse mapping if needed:

```java
int r = id / cols;
int c = id % cols;
```

Allocate DSU for:

```java
rows * cols
```

cells.

---

# 56. Dynamic Island Count

Core pattern:

```java
int islands = 0;

activate(r, c);
islands++;

for (neighbor : fourDirections) {
    if (isLand(neighbor)) {
        if (dsu.union(id(r,c), id(neighbor))) {
            islands--;
        }
    }
}
```

The critical condition is:

```java
if (dsu.union(...))
```

because only a **successful** merge reduces the number of components.

---

# 57. Common DSU Mistakes

## Mistake 1 — Forgetting to find roots

Wrong:

```java
parent[b] = a;
```

without determining whether `a` and `b` are roots.

Correct:

```java
int ra = find(a);
int rb = find(b);
```

then merge roots.

---

## Mistake 2 — Merging equal roots

If:

```java
ra == rb
```

do not merge again.

This condition is often exactly what cycle detection needs.

---

## Mistake 3 — Updating size of the wrong node

After union by size:

```java
parent[rb] = ra;
size[ra] += size[rb];
```

Do not update `size[rb]` as though it remains a representative.

---

## Mistake 4 — Incrementing rank after every union

Wrong:

```java
rank[ra]++;
```

after every merge.

Correct:

> Increase rank only when the two root ranks are equal and one root is attached to the other.

---

## Mistake 5 — Forgetting path compression

A logically correct DSU can still become unnecessarily slow.

Use:

```java
parent[x] = find(parent[x]);
```

---

## Mistake 6 — Confusing rank with size

Rank and size are different concepts.

```text
rank ≈ balancing heuristic
size = number of elements
```

---

## Mistake 7 — Using DSU for shortest paths

DSU only maintains component membership.

It does not store shortest distances.

---

## Mistake 8 — Assuming DSU handles deletions

Standard DSU efficiently supports merges, not arbitrary edge removals.

---

# 58. DSU Complexity Comparison

| Implementation | Worst-case find | Typical overall behavior |
|---|---:|---|
| Naive parent | \(O(V)\) | Can become linear-chain |
| Path compression only | Very efficient amortized | Good |
| Union by rank only | Logarithmic-style height bound | Good |
| Union by size only | Logarithmic-style height bound | Good |
| Both optimizations | \(O(\alpha(V))\) amortized | Excellent |

For interviews:

```text
path compression + rank/size
```

is the standard robust implementation.

---

# 59. DSU Operation Sequence

Suppose:

```text
n = 5
```

Operations:

```text
union(0,1)
union(2,3)
union(1,2)
find(3)
union(3,4)
```

After first two:

```text
{0,1}
{2,3}
{4}
```

After:

```text
union(1,2)
```

we get:

```text
{0,1,2,3}
{4}
```

`find(3)` returns the representative of `{0,1,2,3}`.

Then:

```text
union(3,4)
```

creates:

```text
{0,1,2,3,4}
```

---

# 60. DSU State Should Be Visualized as Sets

When debugging DSU, do not focus only on the parent array.

Think:

```text
Set A
Set B
Set C
```

Then ask:

```text
Does union merge two sets?
Does find return the same representative for every member?
```

This makes DSU bugs easier to identify.

---

# 61. Kruskal with DSU — Detailed Reasoning

Suppose sorted edges are:

```text
e1, e2, e3, e4, ...
```

For every edge:

```text
if find(u) != find(v):
    choose edge
    union(u,v)
```

Why skip an edge when:

```text
find(u) == find(v)
```

Because a path already connects `u` and `v`.

Adding another edge between them creates a cycle.

An MST cannot contain a cycle.

---

# 62. Disconnected Graph and Kruskal

If the graph is disconnected, a single MST does not exist.

Kruskal can still produce a:

> Minimum Spanning Forest

Each connected component gets its own minimum spanning tree.

The number of selected edges becomes:

\[
V-C
\]

where `C` is the number of connected components.

---

# 63. MST Edge Count Through DSU

Initially:

\[
C=V
\]

Every successful Kruskal union:

\[
C\leftarrow C-1
\]

To obtain a connected spanning tree:

\[
C=1
\]

Therefore successful unions:

\[
V-1
\]

Exactly the required number of MST edges.

---

# 64. Dynamic Connectivity — Component Counter

A useful optimization is maintaining:

```java
int components = n;
```

Then:

```java
if (dsu.union(u, v)) {
    components--;
}
```

Now:

```java
components
```

always represents the current number of connected components after all processed additions.

This avoids scanning all vertices after every operation.

---

# 65. Dynamic Connectivity Queries

To answer:

```text
Are u and v connected?
```

use:

```java
dsu.find(u) == dsu.find(v)
```

Example:

```java
boolean connected =
        dsu.find(u) == dsu.find(v);
```

Time:

\[
O(\alpha(V))
\]

amortized.

---

# 66. Dynamic Connectivity Example

Operations:

```text
add 0 1
add 1 2
query 0 2
query 0 3
add 2 3
query 0 3
```

Results:

```text
true
false
true
```

Reason:

```text
0 — 1 — 2
```

connects `0` and `2`, but not `3`.

After adding `2—3`:

```text
0 — 1 — 2 — 3
```

all four are connected.

---

# 67. DSU Problem Recognition

When a problem says:

### "Merge these groups"

Think:

```text
DSU
```

### "Are these two nodes connected after adding edges?"

Think:

```text
DSU
```

### "Number of connected components after each edge addition"

Think:

```text
DSU + component counter
```

### "Does this edge create a cycle?"

Think:

```text
DSU
```

### "Extra edge in a tree"

Think:

```text
DSU
```

### "Minimum cost to connect all points"

Think:

```text
MST
→ Kruskal + DSU
or Prim
```

### "Edges are processed incrementally"

Think:

```text
DSU
```

### "Edges are deleted"

Standard DSU is not enough; investigate offline reverse processing or advanced dynamic-connectivity techniques.

---

# 68. DSU Decision Tree

```text
Need connectivity information?
        |
        +-- Edges are added/merged?
        |       |
        |       +--> DSU
        |
        +-- Need traversal/path?
        |       |
        |       +--> DFS / BFS
        |
        +-- Need shortest path?
        |       |
        |       +--> BFS / Dijkstra / 0-1 BFS / Bellman-Ford
        |
        +-- Need minimum-cost connection?
        |       |
        |       +--> MST
        |             |
        |             +--> Kruskal + DSU
        |             +--> Prim
        |
        +-- Need cycle from edge stream?
                |
                +--> DSU
```

---

# 69. DSU Java Cheat Sheet

## Initialization

```java
for (int i = 0; i < n; i++) {
    parent[i] = i;
    size[i] = 1;
}
```

## Find

```java
int find(int x) {
    if (parent[x] != x) {
        parent[x] = find(parent[x]);
    }
    return parent[x];
}
```

## Union by size

```java
boolean union(int a, int b) {
    int ra = find(a);
    int rb = find(b);

    if (ra == rb) {
        return false;
    }

    if (size[ra] < size[rb]) {
        int t = ra;
        ra = rb;
        rb = t;
    }

    parent[rb] = ra;
    size[ra] += size[rb];

    return true;
}
```

## Connectivity

```java
find(a) == find(b)
```

## Cycle

```java
if (!union(a, b)) {
    // cycle-forming edge
}
```

## Components

```java
int components = n;

if (union(a, b)) {
    components--;
}
```

---

# 70. GATE / CS Theory — DSU

DSU is an abstract data structure for maintaining a partition of elements into disjoint sets.

The two classical operations are:

```text
MAKE-SET
FIND-SET
UNION
```

Depending on the implementation, `MAKE-SET` may be performed during initialization.

---

# 71. GATE-Style Solved Question 1 — Parent Array

Initially:

```text
parent[i] = i
```

for:

```text
i = 0,1,2,3
```

Suppose:

```text
parent[1] = 0
parent[2] = 1
parent[3] = 2
```

What does `find(3)` return?

### Solution

Follow parent pointers:

```text
3 → 2 → 1 → 0
```

Therefore root:

```text
0
```

### Answer

\[
\boxed{0}
\]

---

# 72. GATE-Style Solved Question 2 — Components

A graph has 10 vertices.

After processing all edges using DSU, there are 4 distinct connected components.

How many successful union operations occurred?

### Solution

Initially:

\[
10
\]

components.

Each successful union decreases the component count by exactly one.

Therefore:

\[
10-4=6
\]

successful unions.

### Answer

\[
\boxed{6}
\]

---

# 73. GATE-Style Solved Question 3 — Cycle Detection

Edges are processed in this order:

```text
(0,1)
(1,2)
(2,3)
(0,3)
```

At which edge does DSU first detect a cycle?

### Solution

After the first three edges:

```text
0 — 1 — 2 — 3
```

All four vertices are already connected.

For:

```text
(0,3)
```

we have:

```text
find(0) == find(3)
```

Therefore `union(0,3)` fails.

### Answer

\[
\boxed{(0,3)}
\]

---

# 74. GATE-Style Solved Question 4 — MST

A connected graph contains 7 vertices.

How many successful DSU unions does Kruskal perform before obtaining an MST?

### Solution

An MST contains:

\[
V-1
\]

edges.

Therefore:

\[
7-1=6
\]

successful unions.

### Answer

\[
\boxed{6}
\]

---

# 75. GATE-Style Solved Question 5 — Union by Rank

Two roots have equal rank.

During union by rank, root `B` is attached below root `A`.

What happens to the rank?

### Solution

When two roots have equal rank, attaching one below the other increases the rank of the new root by one.

Therefore:

```text
rank[A]++
```

### Answer

\[
\boxed{\text{Rank of the new root increases by 1}}
\]

---

# 76. GATE-Style Solved Question 6 — Dynamic Connectivity

There are 5 isolated vertices.

The following edges are added:

```text
0-1
1-2
3-4
2-3
```

How many connected components remain?

### Solution

Initially:

```text
{0} {1} {2} {3} {4}
```

After `0-1`:

```text
{0,1} {2} {3} {4}
```

After `1-2`:

```text
{0,1,2} {3} {4}
```

After `3-4`:

```text
{0,1,2} {3,4}
```

After `2-3`:

```text
{0,1,2,3,4}
```

Therefore:

\[
C=1
\]

### Answer

\[
\boxed{1}
\]

---

# 77. LeetCode DSU Roadmap

## Foundation

### 1. Redundant Connection — LC 684

Learn:

- Parent array
- Find
- Union
- Cycle detection

### 2. Number of Provinces — LC 547

Learn:

- Components
- Matrix-to-DSU conversion

---

# 78. Intermediate DSU

### 3. Number of Operations to Make Network Connected — LC 1319

Learn:

- Component counting
- Successful unions
- Edge-count feasibility

### 4. Accounts Merge — LC 721

Learn:

- Mapping arbitrary entities to IDs
- Union by shared identifiers
- Group reconstruction

### 5. Most Stones Removed with Same Row or Column — LC 947

Learn:

- DSU over transformed entities
- Component reasoning

---

# 79. Advanced DSU / MST

### 6. Min Cost to Connect All Points — LC 1584

Learn:

- Kruskal
- Prim comparison
- DSU
- MST modeling

### 7. Number of Islands II — LC 305

Learn:

- Dynamic connectivity
- Grid-to-ID mapping
- Incremental component counting

---

# 80. Related Graph Problems

After mastering DSU, revisit:

- Course Schedule
- Is Graph Bipartite?
- Critical Connections in a Network
- Network Delay Time
- Cheapest Flights Within K Stops

These are graph problems where DSU is generally **not** the primary algorithm.

The ability to recognize when **not** to use DSU is part of mastery.

---

# 81. DSU Mastery Ladder

## Level 1 — Implementation

Master:

```text
parent
find
union
```

## Level 2 — Optimization

Master:

```text
path compression
union by rank
union by size
```

## Level 3 — Connectivity

Master:

```text
component count
same-component queries
dynamic edge additions
```

## Level 4 — Graph Applications

Master:

```text
cycle detection
redundant connection
network connectivity
```

## Level 5 — MST

Master:

```text
Kruskal
cut property
minimum spanning forest
```

## Level 6 — Advanced

Master:

```text
DSU with metadata
grid connectivity
Number of Islands II
offline reverse processing
entity/group merging
```

---

# 82. Important DSU Distinctions

## Union by rank vs union by size

```text
rank → balancing heuristic
size → component cardinality
```

## Find vs union

```text
find(x)
→ representative

union(a,b)
→ merge representatives
```

## Successful vs failed union

```text
successful union
→ two components become one

failed union
→ already same component
→ useful for cycle detection
```

---

# 83. DSU and Graph Theory Connection

For an undirected graph:

```text
vertices = elements
edges = union operations
connected components = DSU sets
```

Processing edges incrementally is therefore equivalent to maintaining the graph's connectivity partition.

This gives the central interpretation:

\[
\text{DSU set} \leftrightarrow \text{connected component}
\]

---

# 84. When DSU Is Better Than Repeated DFS

Suppose there are:

```text
10^5 vertices
10^5 edge additions
10^5 connectivity queries
```

Re-running DFS/BFS after every addition can become too expensive.

DSU processes each addition/query in near-constant amortized time.

This is the primary reason DSU exists in interview and competitive-programming problems.

---

# 85. DSU Does Not Preserve Paths

Suppose:

```text
0 — 1 — 2 — 3
```

DSU can answer:

```text
Are 0 and 3 connected?
```

Yes.

But DSU cannot directly answer:

```text
What is the actual path from 0 to 3?
```

For path reconstruction, maintain graph structure and use DFS/BFS or an appropriate path algorithm.

---

# 86. DSU Does Not Store Distance

Similarly, DSU cannot answer:

```text
Shortest distance from 0 to 7?
```

It only knows whether they are in the same set.

For distances use:

- BFS
- 0-1 BFS
- Dijkstra
- Bellman-Ford
- Floyd-Warshall

depending on the graph.

---

# 87. DSU + Sorting Pattern

A common interview structure is:

```text
sort edges/events
      ↓
process in order
      ↓
union components
      ↓
detect first/optimal merge
```

This appears in:

- Kruskal
- Earliest time everyone becomes connected
- Minimum-cost connectivity
- Threshold connectivity
- Offline grouping

Recognize this pattern quickly.

---

# 88. Threshold Connectivity

Suppose each edge has a weight and the question asks:

> At what minimum threshold do vertices `u` and `v` become connected?

Sort edges by weight.

Process:

```text
smallest → largest
```

Union each edge.

The first weight at which:

```text
find(u) == find(v)
```

is the answer.

This is a powerful DSU + sorting pattern.

---

# 89. Offline Connectivity by Threshold

Suppose there are many queries:

```text
(u, v, limit)
```

meaning:

> Are `u` and `v` connected using edges whose weight satisfies the threshold?

Sort:

1. Edges by weight.
2. Queries by threshold.

Process queries in increasing threshold, adding eligible edges to DSU.

Then each query becomes:

```java
find(u) == find(v)
```

This can transform a difficult repeated-connectivity problem into an efficient offline DSU solution.

---

# 90. Earliest Full Connectivity

Suppose edges arrive with timestamps.

Question:

> At what earliest timestamp are all vertices connected?

Sort/process edges chronologically.

Maintain:

```java
components = n;
```

After each successful union:

```java
components--;
```

The first time:

```text
components == 1
```

the current timestamp is the answer.

---

# 91. DSU with Rollback — Advanced Awareness

A more advanced structure is:

> DSU with rollback

It allows certain union operations to be undone.

This can be used with:

- Offline dynamic connectivity
- Segment-tree-over-time techniques
- Backtracking over connectivity states

It is substantially more advanced than ordinary DSU.

For standard FAANG interviews, understand the concept; implement it only when the problem specifically requires it.

---

# 92. Persistent DSU — Awareness

Persistent variants attempt to preserve historical states.

They are advanced data-structure techniques and are generally outside the core interview DSU syllabus.

Know that ordinary DSU is:

```text
excellent for monotonic merging
```

but not naturally persistent or deletion-friendly.

---

# 93. DSU Complexity Summary

Let:

- `V` = number of elements/vertices
- `E` = number of edges
- `Q` = number of connectivity queries

### Initialization

\[
O(V)
\]

### One find

\[
O(\alpha(V))
\]

amortized with both optimizations.

### One union

\[
O(\alpha(V))
\]

amortized.

### Process `E` edges

\[
O(E\alpha(V))
\]

excluding sorting.

### Kruskal

\[
O(E\log E)
\]

because sorting dominates.

### `Q` connectivity queries

\[
O(Q\alpha(V))
\]

after initialization.

---

# 94. DSU Final Pattern Sheet

```text
parent[x]
    ↓
represents the next parent pointer

find(x)
    ↓
returns component representative

union(a,b)
    ↓
merges two representatives

path compression
    ↓
flattens find paths

union by rank
    ↓
attach smaller-rank root

union by size
    ↓
attach smaller component

same root
    ↓
same component

failed union
    ↓
already connected
    ↓
cycle in edge-stream problems

successful union
    ↓
components--

Kruskal
    ↓
sort edges + DSU

dynamic additions
    ↓
DSU

deletions
    ↓
ordinary DSU is insufficient
```

---

# 95. Final DSU Interview Checklist

## Implementation

- [ ] Initialize parent array.
- [ ] Implement `find`.
- [ ] Understand path compression.
- [ ] Implement union by rank.
- [ ] Implement union by size.
- [ ] Explain why roots are merged.

## Connectivity

- [ ] Count components.
- [ ] Check whether two nodes are connected.
- [ ] Maintain a component counter.
- [ ] Process incremental edge additions.

## Cycle detection

- [ ] Detect cycle from an edge list.
- [ ] Explain why failed union means cycle.
- [ ] Know when DFS is more natural.

## MST

- [ ] Explain MST.
- [ ] Implement Kruskal.
- [ ] Sort edges.
- [ ] Use DSU to reject cycles.
- [ ] Stop after `V-1` successful unions.
- [ ] Handle disconnected graphs as forests when required.
- [ ] Compare Kruskal with Prim.

## Advanced

- [ ] DSU with component size.
- [ ] DSU with custom metadata.
- [ ] Grid-to-ID DSU.
- [ ] Number of Islands II pattern.
- [ ] Threshold connectivity.
- [ ] Offline reverse processing.
- [ ] Understand rollback DSU conceptually.

---

# 96. Final Revision Sheet

## Parent

```java
parent[i] = i;
```

## Find

```java
int find(int x) {
    if (parent[x] != x) {
        parent[x] = find(parent[x]);
    }
    return parent[x];
}
```

## Union by size

```java
int ra = find(a);
int rb = find(b);

if (ra == rb) {
    return false;
}

if (size[ra] < size[rb]) {
    int t = ra;
    ra = rb;
    rb = t;
}

parent[rb] = ra;
size[ra] += size[rb];

return true;
```

## Connectivity

```java
find(a) == find(b)
```

## Component count

```java
int components = n;

if (union(a, b)) {
    components--;
}
```

## Cycle

```java
if (!union(a, b)) {
    // cycle
}
```

## Kruskal

```text
sort edges by weight
for each edge:
    if union(u,v):
        add edge
stop at V-1 selected edges
```

## Complexity

```text
find/union:
O(α(V)) amortized

Kruskal:
O(E log E)
```

---

# 97. Final DSU Mastery Standard

You should consider DSU mastered only when you can look at a problem and immediately determine whether it is a **component-merging problem**.

The core mapping should become:

```text
Parent array
    → represents DSU forest

Find
    → identify representative

Union
    → merge components

Path compression
    → flatten parent paths

Union by rank
    → balance using rank

Union by size
    → balance using component size

Same representative
    → same connected component

Failed union
    → cycle in incremental undirected edge processing

Successful union
    → component count decreases

Kruskal
    → sort edges + union if roots differ

Dynamic connectivity
    → incremental union + find

Grid activation
    → cell-to-ID + union neighbors

Threshold connectivity
    → sort edges/queries + DSU

Offline deletions
    → consider reverse processing

Component metadata
    → store aggregate at representative
```

The key limitation to remember is:

> **DSU is a connectivity structure, not a traversal or shortest-path structure.**

If the question is fundamentally:

> “Which vertices have become connected as edges are added?”

DSU should be one of the first tools you consider.
