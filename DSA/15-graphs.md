# 15 — Graphs

> **Goal:** Master graphs for FAANG/top-product SDE and ML/AI engineering interviews in Java, with enough theory for GATE-level questions and enough pattern recognition for hard LeetCode problems.

---

## 1. Why Graphs Matter

Graphs are one of the largest interview sections because the same abstraction appears in:

- Networks
- Dependencies
- Maps and routing
- Social relationships
- Scheduling
- Connectivity
- Build systems
- Recommendation systems
- State-space search
- Grid problems
- Minimum-cost connection
- Shortest paths
- Network reliability

The important skill is **not memorizing isolated algorithms**. It is identifying:

1. What the vertices represent.
2. What the edges represent.
3. Whether edges are directed.
4. Whether edges are weighted.
5. Whether the graph is static or changing.
6. Whether the task asks for reachability, ordering, shortest path, connectivity, or minimum spanning cost.
7. Which graph algorithm's assumptions are satisfied.

---

# 2. Graph Fundamentals

## 2.1 Basic terminology

A graph is usually written as:

\[
G=(V,E)
\]

where:

- \(V\) = set of vertices/nodes
- \(E\) = set of edges

### Vertex

A node/entity in the graph.

Examples:

- Person
- City
- Course
- Computer
- Grid cell

### Edge

A relationship/connection between two vertices.

Examples:

- Friendship
- Road
- Prerequisite
- Network cable

---

## 2.2 Directed graph

An edge has direction.

\[
u\rightarrow v
\]

means movement/relationship goes from `u` to `v`.

Example:

```text
0 → 1 → 2
```

The edge `0 → 1` does not automatically imply `1 → 0`.

---

## 2.3 Undirected graph

An edge has no direction.

```text
0 — 1
```

means:

```text
0 → 1
1 → 0
```

for traversal purposes.

---

## 2.4 Weighted graph

Every edge has a cost/weight.

```text
0 --5-- 1 --2-- 2
```

The weights may represent:

- Distance
- Time
- Price
- Energy
- Latency
- Risk

---

## 2.5 Unweighted graph

Edges do not carry meaningful numerical costs.

```text
0 — 1 — 2
```

For shortest-path problems in an unweighted graph, BFS is usually the first algorithm to consider.

---

## 2.6 Simple graph

A simple graph generally has:

- No self-loops.
- No parallel duplicate edges between the same pair of vertices.

Interview problems may relax these assumptions, so read the statement carefully.

---

## 2.7 Degree

For an undirected graph:

\[
degree(v)=\text{number of incident edges}
\]

For a directed graph:

- `indegree(v)` = number of incoming edges.
- `outdegree(v)` = number of outgoing edges.

For every directed graph:

\[
\sum indegree(v)=\sum outdegree(v)=|E|
\]

---

## 2.8 Path

A path is a sequence of vertices connected by edges.

```text
0 → 1 → 4 → 7
```

Path length may mean:

- Number of edges in an unweighted graph.
- Sum of weights in a weighted graph.

---

## 2.9 Cycle

A cycle returns to an already visited vertex through a non-trivial path.

Undirected:

```text
0 — 1
|   |
3 — 2
```

Directed:

```text
0 → 1 → 2
↑       ↓
└───────┘
```

---

## 2.10 Connected graph

An undirected graph is connected if every vertex is reachable from every other vertex.

A disconnected graph contains multiple connected components.

```text
0 — 1      2 — 3      4
```

There are 3 connected components.

---

## 2.11 Strongly connected

For a directed graph, a set of vertices is strongly connected if every vertex can reach every other vertex in that set.

Example:

```text
0 → 1
↑   ↓
└── 2
```

`0,1,2` form a strongly connected component.

---

# 3. Graph Classification — First Decision

Before choosing an algorithm, classify the graph.

| Property | Question |
|---|---|
| Direction | Directed or undirected? |
| Weight | Weighted or unweighted? |
| Sign | Can weights be negative? |
| Size | How large are \(V,E\)? |
| Connectivity | One component or many? |
| Query | Reachability, cycle, order, shortest path, MST, SCC? |
| Grid | Is the graph implicit in a matrix? |
| Dynamic | Are edges added/removed over time? |

This classification eliminates many wrong algorithms immediately.

---

# 4. Graph Representation

The three core representations are:

1. Adjacency matrix
2. Adjacency list
3. Edge list

---

# 5. Adjacency Matrix

An adjacency matrix uses a 2D array.

For an unweighted graph:

```text
matrix[u][v] = 1
```

if edge `u → v` exists.

For a weighted graph:

```text
matrix[u][v] = weight
```

---

## 5.1 Example

Edges:

```text
0 — 1
0 — 2
1 — 2
```

Matrix:

```text
    0 1 2
0   0 1 1
1   1 0 1
2   1 1 0
```

---

## 5.2 Complexity

| Operation | Complexity |
|---|---:|
| Space | \(O(V^2)\) |
| Check edge | \(O(1)\) |
| Add edge | \(O(1)\) |
| Remove edge | \(O(1)\) |
| Enumerate neighbors | \(O(V)\) |

---

## 5.3 Java

```java
int[][] graph = new int[n][n];

graph[u][v] = 1;
graph[v][u] = 1; // undirected
```

Weighted:

```java
int[][] graph = new int[n][n];

graph[u][v] = weight;
```

Be careful when `0` is a valid edge weight. Use another sentinel such as a large `INF` value if necessary.

---

# 6. Adjacency List

The most common interview representation.

For every vertex, store its neighbors.

```text
0 → [1, 2]
1 → [0, 2]
2 → [0, 1]
```

---

## 6.1 Complexity

For a sparse graph:

\[
O(V+E)
\]

space.

| Operation | Typical complexity |
|---|---:|
| Space | \(O(V+E)\) |
| Enumerate neighbors | \(O(deg(v))\) |
| Edge test | \(O(deg(v))\) |
| Add edge | \(O(1)\) amortized |

---

## 6.2 Java unweighted adjacency list

```java
List<List<Integer>> graph = new ArrayList<>();

for (int i = 0; i < n; i++) {
    graph.add(new ArrayList<>());
}

graph.get(u).add(v);
graph.get(v).add(u); // undirected
```

Directed:

```java
graph.get(u).add(v);
```

---

# 7. Weighted Adjacency List

A useful Java representation is:

```java
static class Edge {
    int to;
    int weight;

    Edge(int to, int weight) {
        this.to = to;
        this.weight = weight;
    }
}
```

Then:

```java
List<List<Edge>> graph = new ArrayList<>();

for (int i = 0; i < n; i++) {
    graph.add(new ArrayList<>());
}

graph.get(u).add(new Edge(v, w));
```

Undirected:

```java
graph.get(u).add(new Edge(v, w));
graph.get(v).add(new Edge(u, w));
```

---

# 8. Edge List

Store each edge directly.

```text
(u, v)
(u, v, weight)
```

Java:

```java
static class Edge {
    int u;
    int v;
    int w;

    Edge(int u, int v, int w) {
        this.u = u;
        this.v = v;
        this.w = w;
    }
}
```

Useful for:

- Kruskal
- Bellman-Ford
- Sorting edges by weight
- Problems where the input itself is an edge list

Space:

\[
O(E)
\]

---

# 9. Representation Comparison

| Representation | Space | Edge lookup | Neighbor iteration | Best use |
|---|---:|---:|---:|---|
| Matrix | \(O(V^2)\) | \(O(1)\) | \(O(V)\) | Dense graph |
| List | \(O(V+E)\) | \(O(deg(v))\) | \(O(deg(v))\) | Most traversal problems |
| Edge list | \(O(E)\) | \(O(E)\) | Not direct | Kruskal/Bellman-Ford |

### Interview default

Use an **adjacency list** unless the problem strongly suggests a matrix or edge list.

---

# 10. DFS — Depth-First Search

DFS explores one path as deeply as possible before backtracking.

Example:

```text
    0
   / \
  1   2
 / \
3   4
```

A possible DFS:

```text
0 → 1 → 3 → 4 → 2
```

DFS is based on:

- Recursion, or
- Explicit stack.

---

# 11. Recursive DFS

```java
static void dfs(
        int u,
        List<List<Integer>> graph,
        boolean[] visited
) {
    visited[u] = true;

    for (int v : graph.get(u)) {
        if (!visited[v]) {
            dfs(v, graph, visited);
        }
    }
}
```

---

## 11.1 Complexity

Each vertex is visited once.

Each adjacency-list edge is inspected a constant number of times.

\[
O(V+E)
\]

Space:

\[
O(V)
\]

including:

- Visited array.
- Recursion stack in the worst case.

---

# 12. DFS for a Disconnected Graph

Calling DFS from one vertex does not necessarily visit every vertex.

Use:

```java
for (int i = 0; i < n; i++) {
    if (!visited[i]) {
        dfs(i, graph, visited);
    }
}
```

This pattern is fundamental for:

- Connected components
- Counting components
- Detecting cycles in all components
- Bipartite checking
- General graph processing

---

# 13. Iterative DFS

Avoid recursion by using an explicit stack.

```java
static void iterativeDFS(
        int start,
        List<List<Integer>> graph
) {
    int n = graph.size();
    boolean[] visited = new boolean[n];

    Deque<Integer> stack = new ArrayDeque<>();
    stack.push(start);

    while (!stack.isEmpty()) {
        int u = stack.pop();

        if (visited[u]) {
            continue;
        }

        visited[u] = true;

        for (int v : graph.get(u)) {
            if (!visited[v]) {
                stack.push(v);
            }
        }
    }
}
```

---

## 13.1 Mark-on-push variant

You can mark when pushing.

```java
static void iterativeDFS(
        int start,
        List<List<Integer>> graph
) {
    boolean[] visited = new boolean[graph.size()];
    Deque<Integer> stack = new ArrayDeque<>();

    stack.push(start);
    visited[start] = true;

    while (!stack.isEmpty()) {
        int u = stack.pop();

        for (int v : graph.get(u)) {
            if (!visited[v]) {
                visited[v] = true;
                stack.push(v);
            }
        }
    }
}
```

This avoids duplicate stack entries.

---

# 14. BFS — Breadth-First Search

BFS explores vertices level by level.

```text
        0
      /   \
     1     2
    / \     \
   3   4     5
```

BFS:

```text
0
1 2
3 4 5
```

Use a queue.

---

# 15. BFS Template

```java
static void bfs(
        int start,
        List<List<Integer>> graph
) {
    boolean[] visited = new boolean[graph.size()];
    Queue<Integer> queue = new ArrayDeque<>();

    queue.offer(start);
    visited[start] = true;

    while (!queue.isEmpty()) {
        int u = queue.poll();

        for (int v : graph.get(u)) {
            if (!visited[v]) {
                visited[v] = true;
                queue.offer(v);
            }
        }
    }
}
```

---

## 15.1 Complexity

\[
O(V+E)
\]

Time.

\[
O(V)
\]

auxiliary space.

---

# 16. BFS and Shortest Path

For an **unweighted graph**, BFS finds the shortest number of edges from the source.

Why?

BFS processes:

```text
distance 0
distance 1
distance 2
distance 3
...
```

So the first time a vertex is reached, the path has minimum number of edges.

---

## 16.1 Distance template

```java
static int[] bfsDistance(
        int start,
        List<List<Integer>> graph
) {
    int n = graph.size();

    int[] dist = new int[n];
    Arrays.fill(dist, -1);

    Queue<Integer> queue = new ArrayDeque<>();

    dist[start] = 0;
    queue.offer(start);

    while (!queue.isEmpty()) {
        int u = queue.poll();

        for (int v : graph.get(u)) {
            if (dist[v] == -1) {
                dist[v] = dist[u] + 1;
                queue.offer(v);
            }
        }
    }

    return dist;
}
```

---

# 17. Multi-Source BFS

Sometimes there are many starting points.

Instead of running BFS separately from each source, place **all sources into the queue initially**.

Example:

```text
sources = {0, 5, 8}
```

Initialize:

```java
for (int source : sources) {
    queue.offer(source);
    dist[source] = 0;
}
```

Then perform normal BFS.

---

## 17.1 Why it works

All sources begin at distance 0.

The BFS frontier expands simultaneously.

Therefore every vertex gets its distance to the **nearest source**.

---

## 17.2 Common applications

- Rotting Oranges
- Distance from nearest zero
- Nearest facility
- Infection/spread simulation
- Fire spreading
- Grid propagation

---

# 18. Multi-Source BFS Template

```java
static int[] multiSourceBFS(
        int n,
        List<List<Integer>> graph,
        List<Integer> sources
) {
    int[] dist = new int[n];
    Arrays.fill(dist, -1);

    Queue<Integer> queue = new ArrayDeque<>();

    for (int source : sources) {
        dist[source] = 0;
        queue.offer(source);
    }

    while (!queue.isEmpty()) {
        int u = queue.poll();

        for (int v : graph.get(u)) {
            if (dist[v] == -1) {
                dist[v] = dist[u] + 1;
                queue.offer(v);
            }
        }
    }

    return dist;
}
```

---

# 19. Grid as a Graph

A grid is usually an implicit graph.

For a 4-directional grid:

```text
(-1,0)
(1,0)
(0,-1)
(0,1)
```

Java:

```java
static final int[][] DIRS = {
    {-1, 0},
    {1, 0},
    {0, -1},
    {0, 1}
};
```

For 8 directions, include diagonals.

---

# 20. Grid Traversal Template

```java
for (int[] d : DIRS) {
    int nr = r + d[0];
    int nc = c + d[1];

    if (nr >= 0 && nr < rows &&
        nc >= 0 && nc < cols) {

        // process neighbor
    }
}
```

Always check boundaries before accessing the grid.

---

# 21. Connected Components

Given an undirected graph, count groups of mutually reachable vertices.

Example:

```text
0 — 1 — 2

3 — 4

5
```

Answer:

```text
3
```

---

## 21.1 DFS solution

```java
static int countComponents(
        int n,
        int[][] edges
) {
    List<List<Integer>> graph = new ArrayList<>();

    for (int i = 0; i < n; i++) {
        graph.add(new ArrayList<>());
    }

    for (int[] edge : edges) {
        int u = edge[0];
        int v = edge[1];

        graph.get(u).add(v);
        graph.get(v).add(u);
    }

    boolean[] visited = new boolean[n];
    int components = 0;

    for (int i = 0; i < n; i++) {
        if (!visited[i]) {
            components++;
            dfs(i, graph, visited);
        }
    }

    return components;
}
```

Complexity:

\[
O(V+E)
\]

---

# 22. Number of Islands

Classic grid problem.

A land cell is connected to neighboring land cells.

The grid itself acts as the graph.

---

## 22.1 DFS solution

```java
static int numIslands(char[][] grid) {
    int rows = grid.length;
    int cols = grid[0].length;

    int count = 0;

    for (int r = 0; r < rows; r++) {
        for (int c = 0; c < cols; c++) {
            if (grid[r][c] == '1') {
                count++;
                sink(grid, r, c);
            }
        }
    }

    return count;
}

static void sink(char[][] grid, int r, int c) {
    if (r < 0 || r >= grid.length ||
        c < 0 || c >= grid[0].length ||
        grid[r][c] != '1') {
        return;
    }

    grid[r][c] = '0';

    for (int[] d : DIRS) {
        sink(grid, r + d[0], c + d[1]);
    }
}
```

Time:

\[
O(RC)
\]

because every cell is processed at most a constant number of times.

---

# 23. Flood Fill

Start from a pixel and replace all connected pixels having the original color.

The pattern is:

1. Save original color.
2. If original equals new color, return immediately.
3. DFS/BFS from the starting cell.
4. Replace every reachable cell with the original color.

```java
static int[][] floodFill(
        int[][] image,
        int sr,
        int sc,
        int color
) {
    int original = image[sr][sc];

    if (original == color) {
        return image;
    }

    fill(image, sr, sc, original, color);
    return image;
}

static void fill(
        int[][] image,
        int r,
        int c,
        int original,
        int color
) {
    if (r < 0 || r >= image.length ||
        c < 0 || c >= image[0].length ||
        image[r][c] != original) {
        return;
    }

    image[r][c] = color;

    for (int[] d : DIRS) {
        fill(image, r + d[0], c + d[1],
             original, color);
    }
}
```

### Important bug

If `original == color` and you do not return, recursion may repeatedly revisit the same color region.

---

# 24. Cycle Detection — Big Picture

Cycle detection differs between directed and undirected graphs.

| Graph | Common method |
|---|---|
| Undirected | DFS with parent |
| Undirected | DSU |
| Directed | 3-color DFS |
| Directed | Kahn's algorithm |

Do not blindly reuse one template.

---

# 25. Cycle Detection in an Undirected Graph — DFS

Suppose:

```text
0 — 1 — 2
    |
    3
```

When traversing `1 → 0`, seeing `0` again is not a cycle because `0` is simply the parent.

Therefore we track the parent.

---

## 25.1 Template

```java
static boolean hasCycleUndirected(
        List<List<Integer>> graph
) {
    int n = graph.size();
    boolean[] visited = new boolean[n];

    for (int i = 0; i < n; i++) {
        if (!visited[i] &&
            dfsCycle(i, -1, graph, visited)) {
            return true;
        }
    }

    return false;
}

static boolean dfsCycle(
        int u,
        int parent,
        List<List<Integer>> graph,
        boolean[] visited
) {
    visited[u] = true;

    for (int v : graph.get(u)) {
        if (!visited[v]) {
            if (dfsCycle(v, u, graph, visited)) {
                return true;
            }
        } else if (v != parent) {
            return true;
        }
    }

    return false;
}
```

---

# 26. Cycle Detection in Undirected Graph — DSU

For an edge `(u,v)`:

- Find the root of `u`.
- Find the root of `v`.
- If roots are equal, adding this edge creates a cycle.
- Otherwise union the sets.

This is particularly convenient when the input is already an edge list.

---

# 27. Directed Cycle Detection

A normal `visited[]` array is insufficient.

Why?

In a directed graph, reaching a previously visited vertex does not necessarily mean a cycle.

We need to distinguish:

- Unvisited
- Currently in recursion path
- Fully processed

Use three states:

```text
0 = unvisited
1 = visiting
2 = processed
```

---

# 28. Directed Cycle — 3-State DFS

```java
static boolean hasDirectedCycle(
        List<List<Integer>> graph
) {
    int n = graph.size();
    int[] state = new int[n];

    for (int i = 0; i < n; i++) {
        if (state[i] == 0 &&
            dfsDirected(i, graph, state)) {
            return true;
        }
    }

    return false;
}

static boolean dfsDirected(
        int u,
        List<List<Integer>> graph,
        int[] state
) {
    state[u] = 1;

    for (int v : graph.get(u)) {
        if (state[v] == 1) {
            return true;
        }

        if (state[v] == 0 &&
            dfsDirected(v, graph, state)) {
            return true;
        }
    }

    state[u] = 2;
    return false;
}
```

### Key insight

An edge to a **currently active ancestor** means a directed cycle.

An edge to a fully processed vertex does not necessarily mean a cycle.

---

# 29. Topological Sorting

A topological ordering is an ordering of vertices such that for every directed edge:

\[
u\rightarrow v
\]

`u` appears before `v`.

It exists only for a **DAG**:

> Directed Acyclic Graph

Example:

```text
0 → 2
1 → 2
2 → 3
```

Possible ordering:

```text
0, 1, 2, 3
```

or:

```text
1, 0, 2, 3
```

Multiple valid topological orders may exist.

---

# 30. Important Topological-Sort Fact

A directed graph has a topological ordering **if and only if** it is acyclic.

Therefore topological sorting can also be used to detect directed cycles.

---

# 31. Kahn's Algorithm

Kahn's algorithm uses:

- Indegree array
- Queue of zero-indegree vertices

### Algorithm

1. Compute indegree of every vertex.
2. Add all vertices with indegree `0` to queue.
3. Remove one vertex.
4. Add it to answer.
5. Decrease indegree of its outgoing neighbors.
6. Whenever a neighbor reaches indegree `0`, enqueue it.
7. If fewer than `V` vertices are processed, a cycle exists.

---

## 31.1 Java

```java
static List<Integer> topoKahn(
        int n,
        List<List<Integer>> graph
) {
    int[] indegree = new int[n];

    for (int u = 0; u < n; u++) {
        for (int v : graph.get(u)) {
            indegree[v]++;
        }
    }

    Queue<Integer> queue = new ArrayDeque<>();

    for (int i = 0; i < n; i++) {
        if (indegree[i] == 0) {
            queue.offer(i);
        }
    }

    List<Integer> order = new ArrayList<>();

    while (!queue.isEmpty()) {
        int u = queue.poll();
        order.add(u);

        for (int v : graph.get(u)) {
            if (--indegree[v] == 0) {
                queue.offer(v);
            }
        }
    }

    if (order.size() != n) {
        return new ArrayList<>();
    }

    return order;
}
```

Complexity:

\[
O(V+E)
\]

---

# 32. DFS Topological Sort

Use DFS and push a vertex **after all descendants are processed**.

The postorder sequence is then reversed.

---

## 32.1 Java

```java
static List<Integer> topoDFS(
        int n,
        List<List<Integer>> graph
) {
    int[] state = new int[n];
    List<Integer> order = new ArrayList<>();

    for (int i = 0; i < n; i++) {
        if (state[i] == 0 &&
            !dfsTopo(i, graph, state, order)) {
            return new ArrayList<>();
        }
    }

    Collections.reverse(order);
    return order;
}

static boolean dfsTopo(
        int u,
        List<List<Integer>> graph,
        int[] state,
        List<Integer> order
) {
    state[u] = 1;

    for (int v : graph.get(u)) {
        if (state[v] == 1) {
            return false;
        }

        if (state[v] == 0 &&
            !dfsTopo(v, graph, state, order)) {
            return false;
        }
    }

    state[u] = 2;
    order.add(u);
    return true;
}
```

---

# 33. Kahn vs DFS Topological Sort

| Feature | Kahn | DFS |
|---|---|---|
| Main idea | Indegree | Postorder |
| Data structure | Queue | Recursion/stack |
| Cycle detection | Processed count < V | Back edge |
| Natural for | Dependency processing | Recursive graph logic |
| Complexity | \(O(V+E)\) | \(O(V+E)\) |

Use either when appropriate.

---

# 34. Course Schedule Pattern

Suppose:

```text
course [a] requires [b]
```

means:

```text
b → a
```

Prerequisite problems are directed graph problems.

### Course Schedule I

Question:

> Can all courses be completed?

Equivalent:

> Does the prerequisite graph contain a cycle?

Use:

- Kahn's algorithm, or
- DFS cycle detection.

### Course Schedule II

Question:

> Return a valid order.

Use topological sorting.

---

# 35. Undirected Graph — Bipartite Graph

A graph is bipartite if its vertices can be divided into two groups such that every edge connects vertices from opposite groups.

Equivalent:

> An undirected graph is bipartite iff it contains no odd-length cycle.

---

## 35.1 2-coloring

Use colors:

```text
0 = uncolored
1 = color A
2 = color B
```

For every edge:

```text
color[v] must differ from color[u]
```

---

## 35.2 BFS solution

```java
static boolean isBipartite(
        List<List<Integer>> graph
) {
    int n = graph.size();
    int[] color = new int[n];

    for (int start = 0; start < n; start++) {
        if (color[start] != 0) {
            continue;
        }

        Queue<Integer> queue = new ArrayDeque<>();
        queue.offer(start);
        color[start] = 1;

        while (!queue.isEmpty()) {
            int u = queue.poll();

            for (int v : graph.get(u)) {
                if (color[v] == 0) {
                    color[v] = 3 - color[u];
                    queue.offer(v);
                } else if (color[v] == color[u]) {
                    return false;
                }
            }
        }
    }

    return true;
}
```

Complexity:

\[
O(V+E)
\]

---

# 36. Bridges

A bridge is an edge whose removal increases the number of connected components.

Example:

```text
0 — 1 — 2
    |
    3
```

If edge `1—3` is removed, vertex `3` becomes disconnected.

---

# 37. Bridge Intuition

Run DFS and assign each vertex a discovery time:

```text
disc[u]
```

Maintain:

```text
low[u]
```

where `low[u]` is the smallest discovery time reachable from the DFS subtree of `u` using:

- Tree edges downward.
- At most one back edge upward.

For a DFS tree edge:

```text
u → v
```

it is a bridge if:

\[
low[v] > disc[u]
\]

### Why?

If:

\[
low[v] > disc[u]
\]

then the subtree rooted at `v` cannot reach `u` or any ancestor of `u` through another edge.

Therefore removing `u-v` disconnects that subtree.

---

# 38. Bridge Algorithm

```java
static int timer;

static List<List<Integer>> findBridges(
        int n,
        List<List<Integer>> graph
) {
    int[] disc = new int[n];
    int[] low = new int[n];

    Arrays.fill(disc, -1);

    List<List<Integer>> bridges = new ArrayList<>();

    timer = 0;

    for (int i = 0; i < n; i++) {
        if (disc[i] == -1) {
            dfsBridge(i, -1, graph, disc, low, bridges);
        }
    }

    return bridges;
}

static void dfsBridge(
        int u,
        int parent,
        List<List<Integer>> graph,
        int[] disc,
        int[] low,
        List<List<Integer>> bridges
) {
    disc[u] = low[u] = timer++;

    for (int v : graph.get(u)) {
        if (v == parent) {
            continue;
        }

        if (disc[v] == -1) {
            dfsBridge(v, u, graph, disc, low, bridges);

            low[u] = Math.min(low[u], low[v]);

            if (low[v] > disc[u]) {
                bridges.add(Arrays.asList(u, v));
            }
        } else {
            low[u] = Math.min(low[u], disc[v]);
        }
    }
}
```

### Multigraph caveat

If parallel edges are possible, `v == parent` is not sufficient because one parallel edge may be the actual back edge.

In that case store an **edge ID** and skip only the parent edge ID.

---

# 39. Articulation Points

An articulation point (cut vertex) is a vertex whose removal increases the number of connected components.

For a non-root DFS vertex `u`, if it has a child `v` such that:

\[
low[v]\ge disc[u]
\]

then `u` is an articulation point.

---

## 39.1 Root special case

For a DFS root:

```text
root is articulation point
iff it has at least 2 DFS children
```

The root condition differs from the non-root condition.

---

# 40. Articulation Point Template

```java
static int timer;

static boolean[] articulationPoints(
        int n,
        List<List<Integer>> graph
) {
    int[] disc = new int[n];
    int[] low = new int[n];
    boolean[] isArt = new boolean[n];

    Arrays.fill(disc, -1);

    timer = 0;

    for (int i = 0; i < n; i++) {
        if (disc[i] == -1) {
            dfsArt(i, -1, graph, disc, low, isArt);
        }
    }

    return isArt;
}

static void dfsArt(
        int u,
        int parent,
        List<List<Integer>> graph,
        int[] disc,
        int[] low,
        boolean[] isArt
) {
    disc[u] = low[u] = timer++;

    int children = 0;

    for (int v : graph.get(u)) {
        if (v == parent) {
            continue;
        }

        if (disc[v] == -1) {
            children++;

            dfsArt(v, u, graph, disc, low, isArt);

            low[u] = Math.min(low[u], low[v]);

            if (parent != -1 &&
                low[v] >= disc[u]) {
                isArt[u] = true;
            }
        } else {
            low[u] = Math.min(low[u], disc[v]);
        }
    }

    if (parent == -1 && children > 1) {
        isArt[u] = true;
    }
}
```

---

# 41. Bridges vs Articulation Points

| Property | Bridge | Articulation point |
|---|---|---|
| Removed object | Edge | Vertex |
| Condition | \(low[v] > disc[u]\) | \(low[v] \ge disc[u]\) |
| Root special case | No | Yes |
| Main idea | Subtree cannot bypass edge | Subtree cannot bypass vertex |

Memorize the inequality difference:

```text
Bridge:       low[v] >  disc[u]
Articulation: low[v] >= disc[u]
```

---

# 42. Shortest Paths — Algorithm Selection

This is one of the most important graph decision tables.

| Graph | Algorithm |
|---|---|
| Unweighted | BFS |
| Weights only 0/1 | 0-1 BFS |
| Weighted, all weights ≥ 0 | Dijkstra |
| Negative edges allowed | Bellman-Ford |
| All-pairs shortest paths | Floyd-Warshall |
| DAG shortest path | Topological-order relaxation |

### Critical rule

**Dijkstra does not support negative edge weights.**

---

# 43. BFS Shortest Path

For unweighted edges:

```text
A — B — C
 \      |
  D ——— E
```

BFS gives minimum number of edges.

Use:

```java
int[] dist = new int[n];
Arrays.fill(dist, -1);
```

Set source:

```java
dist[source] = 0;
```

Then:

```java
dist[v] = dist[u] + 1;
```

when `v` is first discovered.

---

# 44. Dijkstra's Algorithm

Dijkstra computes single-source shortest paths when all edge weights are non-negative.

Problem:

```text
source → every vertex
```

---

## 44.1 Core idea

Maintain the best known distance:

\[
dist[v]
\]

Repeatedly choose the unsettled vertex with minimum tentative distance.

Relax its outgoing edges.

For edge:

\[
u\rightarrow v
\]

with weight `w`:

\[
dist[v] = \min(dist[v], dist[u]+w)
\]

---

# 45. Dijkstra with Priority Queue

Java:

```java
static class State {
    int node;
    long dist;

    State(int node, long dist) {
        this.node = node;
        this.dist = dist;
    }
}
```

Comparator:

```java
PriorityQueue<State> pq =
        new PriorityQueue<>(
            Comparator.comparingLong(s -> s.dist)
        );
```

---

## 45.1 Implementation

```java
static long[] dijkstra(
        int source,
        List<List<Edge>> graph
) {
    int n = graph.size();
    long INF = Long.MAX_VALUE / 4;

    long[] dist = new long[n];
    Arrays.fill(dist, INF);

    PriorityQueue<State> pq =
            new PriorityQueue<>(
                Comparator.comparingLong(s -> s.dist)
            );

    dist[source] = 0;
    pq.offer(new State(source, 0));

    while (!pq.isEmpty()) {
        State cur = pq.poll();

        int u = cur.node;
        long d = cur.dist;

        if (d != dist[u]) {
            continue;
        }

        for (Edge edge : graph.get(u)) {
            int v = edge.to;
            long nd = d + edge.weight;

            if (nd < dist[v]) {
                dist[v] = nd;
                pq.offer(new State(v, nd));
            }
        }
    }

    return dist;
}
```

---

# 46. Dijkstra — Stale Entries

Java's `PriorityQueue` does not provide a direct decrease-key operation.

Suppose:

```text
dist[5] = 20
```

Later discover:

```text
dist[5] = 7
```

Insert another `(5,7)`.

The old `(5,20)` remains.

So we use:

```java
if (d != dist[u]) {
    continue;
}
```

This is called the **stale-entry check**.

It is a standard Java Dijkstra pattern.

---

# 47. Dijkstra Complexity

With adjacency list + binary heap:

\[
O((V+E)\log V)
\]

Often simplified to:

\[
O(E\log V)
\]

for connected sparse graphs.

Space:

\[
O(V+E)
\]

---

# 48. 0-1 BFS

If every edge weight is either:

```text
0 or 1
```

use a deque.

For edge weight `0`:

```java
deque.addFirst(v);
```

For edge weight `1`:

```java
deque.addLast(v);
```

This processes vertices in nondecreasing distance order without a heap.

---

## 48.1 Template

```java
static int[] zeroOneBFS(
        int source,
        List<List<Edge>> graph
) {
    int n = graph.size();
    int INF = Integer.MAX_VALUE / 4;

    int[] dist = new int[n];
    Arrays.fill(dist, INF);

    Deque<Integer> deque = new ArrayDeque<>();

    dist[source] = 0;
    deque.addFirst(source);

    while (!deque.isEmpty()) {
        int u = deque.removeFirst();

        for (Edge e : graph.get(u)) {
            int v = e.to;
            int nd = dist[u] + e.weight;

            if (nd < dist[v]) {
                dist[v] = nd;

                if (e.weight == 0) {
                    deque.addFirst(v);
                } else {
                    deque.addLast(v);
                }
            }
        }
    }

    return dist;
}
```

Complexity:

\[
O(V+E)
\]

under the standard 0/1 edge-weight setting.

---

# 49. Bellman-Ford

Bellman-Ford handles:

- Positive edges
- Zero edges
- Negative edges
- Negative-cycle detection reachable from the source

It computes single-source shortest paths.

---

# 50. Bellman-Ford Principle

Relax every edge repeatedly.

For a graph with `V` vertices, a shortest simple path contains at most:

\[
V-1
\]

edges.

Therefore perform `V-1` full relaxation passes.

---

## 50.1 Relaxation

For edge:

```text
u → v, weight w
```

if:

```text
dist[u] + w < dist[v]
```

update:

```text
dist[v] = dist[u] + w
```

---

# 51. Bellman-Ford Java

```java
static long[] bellmanFord(
        int n,
        List<Edge> edges,
        int source
) {
    long INF = Long.MAX_VALUE / 4;
    long[] dist = new long[n];

    Arrays.fill(dist, INF);
    dist[source] = 0;

    for (int i = 1; i <= n - 1; i++) {
        boolean changed = false;

        for (Edge e : edges) {
            if (dist[e.u] == INF) {
                continue;
            }

            long nd = dist[e.u] + e.w;

            if (nd < dist[e.v]) {
                dist[e.v] = nd;
                changed = true;
            }
        }

        if (!changed) {
            break;
        }
    }

    return dist;
}
```

---

# 52. Negative Cycle Detection

After `V-1` passes, perform one more pass.

If a reachable edge can still be relaxed, a reachable negative cycle exists.

```java
for (Edge e : edges) {
    if (dist[e.u] != INF &&
        dist[e.u] + e.w < dist[e.v]) {
        // negative cycle reachable from source
    }
}
```

### Important distinction

This detects a negative cycle **reachable from the source**.

An unrelated negative cycle in another disconnected component does not affect single-source distances.

---

# 53. Bellman-Ford Complexity

\[
O(VE)
\]

Time.

\[
O(V)
\]

extra distance space, excluding input edges.

---

# 54. Floyd-Warshall

Floyd-Warshall computes:

> Shortest paths between **every pair** of vertices.

Dynamic programming formulation:

\[
dist[i][j]
=
\min(
dist[i][j],
dist[i][k]+dist[k][j]
)
\]

for each intermediate vertex `k`.

---

# 55. Floyd-Warshall Initialization

```java
long INF = Long.MAX_VALUE / 4;
long[][] dist = new long[n][n];

for (int i = 0; i < n; i++) {
    Arrays.fill(dist[i], INF);
    dist[i][i] = 0;
}
```

For edge:

```java
dist[u][v] = Math.min(dist[u][v], w);
```

---

## 55.1 Core algorithm

```java
for (int k = 0; k < n; k++) {
    for (int i = 0; i < n; i++) {
        if (dist[i][k] == INF) {
            continue;
        }

        for (int j = 0; j < n; j++) {
            if (dist[k][j] == INF) {
                continue;
            }

            dist[i][j] = Math.min(
                dist[i][j],
                dist[i][k] + dist[k][j]
            );
        }
    }
}
```

Complexity:

\[
O(V^3)
\]

Space:

\[
O(V^2)
\]

---

# 56. Floyd-Warshall Negative Cycles

After completion:

```java
if (dist[i][i] < 0)
```

for some `i`, a negative cycle exists.

Why?

A negative-cost cycle allows a path from `i` back to itself with total negative cost.

---

# 57. Shortest-Path Decision Tree

```text
Shortest path?
        |
        +-- Unweighted ----------------> BFS
        |
        +-- Weights are 0 or 1 --------> 0-1 BFS
        |
        +-- All weights non-negative --> Dijkstra
        |
        +-- Negative weights ----------> Bellman-Ford
        |
        +-- Every pair ----------------> Floyd-Warshall
        |
        +-- DAG -----------------------> Topological relaxation
```

This decision tree should become automatic.

---

# 58. Minimum Spanning Tree

An MST is a spanning tree of a connected, weighted, undirected graph with minimum total edge weight.

A spanning tree:

- Includes every vertex.
- Is connected.
- Contains no cycle.
- Has exactly:

\[
V-1
\]

edges.

---

# 59. MST vs Shortest Path

Do not confuse them.

### Shortest path

Minimizes distance from a source to a destination.

### MST

Connects **all vertices** with minimum total edge weight.

An MST does not generally give shortest paths from a source.

---

# 60. Kruskal's Algorithm

Kruskal builds the MST by considering edges from smallest weight to largest.

Algorithm:

1. Sort all edges by weight.
2. Initially every vertex is a separate component.
3. Take the next smallest edge.
4. If its endpoints are in different components, add it.
5. Union the components.
6. Stop after `V-1` edges.

The component test is done using DSU.

---

# 61. Kruskal Java

```java
static List<Edge> kruskal(
        int n,
        List<Edge> edges
) {
    edges.sort(Comparator.comparingInt(e -> e.w));

    DSU dsu = new DSU(n);
    List<Edge> mst = new ArrayList<>();

    for (Edge e : edges) {
        if (dsu.union(e.u, e.v)) {
            mst.add(e);

            if (mst.size() == n - 1) {
                break;
            }
        }
    }

    return mst;
}
```

Complexity:

Sorting dominates:

\[
O(E\log E)
\]

DSU operations are nearly constant amortized.

---

# 62. Prim's Algorithm

Prim grows one MST from a starting vertex.

At every step:

> Choose the minimum-weight edge that connects the current tree to a new vertex.

Priority queue implementation:

```java
static long prim(
        int n,
        List<List<Edge>> graph
) {
    boolean[] used = new boolean[n];

    PriorityQueue<State> pq =
            new PriorityQueue<>(
                Comparator.comparingLong(s -> s.dist)
            );

    pq.offer(new State(0, 0));

    long total = 0;
    int count = 0;

    while (!pq.isEmpty()) {
        State cur = pq.poll();

        int u = cur.node;

        if (used[u]) {
            continue;
        }

        used[u] = true;
        total += cur.dist;
        count++;

        for (Edge e : graph.get(u)) {
            if (!used[e.to]) {
                pq.offer(new State(e.to, e.weight));
            }
        }
    }

    return count == n ? total : -1;
}
```

Complexity with adjacency list + binary heap:

\[
O(E\log V)
\]

---

# 63. Kruskal vs Prim

| Property | Kruskal | Prim |
|---|---|---|
| Starts with | No single tree | One vertex |
| Main structure | DSU | Priority queue |
| Main operation | Sort edges | Expand frontier |
| Good representation | Edge list | Adjacency list |
| Complexity | \(O(E\log E)\) | \(O(E\log V)\) |
| Naturally handles disconnected graph | Produces forest | Must restart for forest |

---

# 64. MST Properties

### Cut property

For any cut, a minimum-weight edge crossing that cut is safe for some MST.

### Cycle property

For a cycle, a strictly heaviest edge cannot belong to an MST.

### Distinct edge weights

If all edge weights are distinct, the MST is unique.

---

# 65. Disjoint Set Union — DSU

Also called:

- Union-Find
- Disjoint Set Union

It maintains a collection of disjoint sets.

Supports:

- `find(x)` — identify the component/root.
- `union(a,b)` — merge components.

---

# 66. DSU Structure

Initially:

```text
0   1   2   3
```

Each vertex is its own component.

After:

```text
union(0,1)
union(1,2)
```

we may have:

```text
    0
    |
    1
    |
    2
```

All three have the same representative.

---

# 67. Path Compression

During `find(x)`, point nodes directly toward the root.

```java
int find(int x) {
    if (parent[x] != x) {
        parent[x] = find(parent[x]);
    }

    return parent[x];
}
```

This dramatically reduces future find operations.

---

# 68. Union by Size

Attach the smaller tree to the larger tree.

```java
boolean union(int a, int b) {
    a = find(a);
    b = find(b);

    if (a == b) {
        return false;
    }

    if (size[a] < size[b]) {
        int temp = a;
        a = b;
        b = temp;
    }

    parent[b] = a;
    size[a] += size[b];

    return true;
}
```

---

# 69. Complete DSU Java Template

```java
static class DSU {
    int[] parent;
    int[] size;

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
}
```

---

# 70. DSU Complexity

With:

- Path compression
- Union by size/rank

the amortized cost is approximately:

\[
O(\alpha(V))
\]

per operation, where \(\alpha\) is the inverse Ackermann function.

For practical constraints, this is effectively constant.

---

# 71. When to Use DSU

Use DSU when the problem asks about:

- Dynamic connectivity by edge additions.
- Whether adding an edge creates a cycle.
- Number of connected components.
- Merging groups.
- Redundant connections.
- Minimum spanning tree via Kruskal.
- Network connectivity.
- Connectivity after processing edges offline.

DSU is generally less suitable for explicit shortest-path problems.

---

# 72. Strongly Connected Components

An SCC of a directed graph is a maximal set of vertices where every vertex can reach every other vertex.

Example:

```text
0 → 1 → 2
↑       ↓
└───────┘
```

These vertices belong to one SCC.

A directed graph can have multiple SCCs.

---

# 73. SCC Condensation Graph

Compress every SCC into one super-node.

The resulting graph is always a DAG.

This is called the **condensation graph**.

This is a useful theoretical fact and appears in advanced graph problems.

---

# 74. Kosaraju's Algorithm

Kosaraju finds SCCs in two DFS passes.

### Pass 1

Run DFS on the original graph.

Record vertices in finishing order.

### Pass 2

Reverse every edge.

Process vertices in decreasing finishing time.

Each DFS in the reversed graph produces one SCC.

Complexity:

\[
O(V+E)
\]

---

# 75. Why Transpose Works

If two vertices belong to the same SCC, reachability exists in both directions.

Reversing all edges preserves this mutual reachability within an SCC.

The finishing-order property ensures that the second DFS extracts one SCC at a time.

---

# 76. Kosaraju Java

```java
static List<List<Integer>> kosaraju(
        int n,
        List<List<Integer>> graph
) {
    boolean[] visited = new boolean[n];
    List<Integer> order = new ArrayList<>();

    for (int i = 0; i < n; i++) {
        if (!visited[i]) {
            dfsFinish(i, graph, visited, order);
        }
    }

    List<List<Integer>> rev = new ArrayList<>();

    for (int i = 0; i < n; i++) {
        rev.add(new ArrayList<>());
    }

    for (int u = 0; u < n; u++) {
        for (int v : graph.get(u)) {
            rev.get(v).add(u);
        }
    }

    Collections.reverse(order);
    Arrays.fill(visited, false);

    List<List<Integer>> components = new ArrayList<>();

    for (int u : order) {
        if (!visited[u]) {
            List<Integer> component = new ArrayList<>();
            dfsCollect(u, rev, visited, component);
            components.add(component);
        }
    }

    return components;
}

static void dfsFinish(
        int u,
        List<List<Integer>> graph,
        boolean[] visited,
        List<Integer> order
) {
    visited[u] = true;

    for (int v : graph.get(u)) {
        if (!visited[v]) {
            dfsFinish(v, graph, visited, order);
        }
    }

    order.add(u);
}

static void dfsCollect(
        int u,
        List<List<Integer>> graph,
        boolean[] visited,
        List<Integer> component
) {
    visited[u] = true;
    component.add(u);

    for (int v : graph.get(u)) {
        if (!visited[v]) {
            dfsCollect(v, graph, visited, component);
        }
    }
}
```

---

# 77. Tarjan's Algorithm for SCC

Tarjan finds SCCs in one DFS.

It uses:

- Discovery time
- Low-link value
- Stack
- `onStack[]`

For each vertex:

```text
disc[u]
low[u]
```

When:

\[
low[u] = disc[u]
\]

`u` is the root of an SCC.

Pop the stack until `u` is removed.

---

# 78. Tarjan SCC Core

```java
static class TarjanSCC {
    List<List<Integer>> graph;
    int[] disc;
    int[] low;
    boolean[] onStack;
    Deque<Integer> stack;
    int time;

    List<List<Integer>> components = new ArrayList<>();

    TarjanSCC(List<List<Integer>> graph) {
        this.graph = graph;

        int n = graph.size();

        disc = new int[n];
        low = new int[n];
        onStack = new boolean[n];
        stack = new ArrayDeque<>();

        Arrays.fill(disc, -1);

        for (int i = 0; i < n; i++) {
            if (disc[i] == -1) {
                dfs(i);
            }
        }
    }

    void dfs(int u) {
        disc[u] = low[u] = time++;

        stack.push(u);
        onStack[u] = true;

        for (int v : graph.get(u)) {
            if (disc[v] == -1) {
                dfs(v);
                low[u] = Math.min(low[u], low[v]);
            } else if (onStack[v]) {
                low[u] = Math.min(low[u], disc[v]);
            }
        }

        if (low[u] == disc[u]) {
            List<Integer> component = new ArrayList<>();

            while (true) {
                int v = stack.pop();
                onStack[v] = false;
                component.add(v);

                if (v == u) {
                    break;
                }
            }

            components.add(component);
        }
    }
}
```

Complexity:

\[
O(V+E)
\]

---

# 79. Kosaraju vs Tarjan SCC

| Feature | Kosaraju | Tarjan |
|---|---|---|
| DFS passes | 2 | 1 |
| Transpose graph | Yes | No |
| Stack | Not required for SCC extraction | Required |
| Complexity | \(O(V+E)\) | \(O(V+E)\) |
| Implementation | Often easier to reason about | More compact but subtle |

For interviews, know the conceptual difference rather than memorizing code blindly.

---

# 80. Tarjan: SCC vs Bridges

Both use `disc[]` and `low[]`, but the contexts differ.

### Undirected bridges/articulation

Use DFS tree relationships and undirected back edges.

Bridge:

\[
low[v] > disc[u]
\]

Articulation:

\[
low[v] \ge disc[u]
\]

### Directed SCC

Use a stack and only certain edges to vertices currently on the stack.

SCC root:

\[
low[u] = disc[u]
\]

Do not mix the conditions.

---

# 81. Network Connectivity

Typical problem:

> There are `n` computers and cables. What is the minimum number of cable moves required to connect the entire network?

First ask:

### Is there enough total edge capacity?

To connect `n` vertices, at least:

\[
n-1
\]

edges are necessary.

If:

\[
E<n-1
\]

answer is impossible.

If enough edges exist, and there are `C` connected components, minimum operations:

\[
C-1
\]

---

# 82. Network Connectivity via DSU

Process every edge.

Initially:

```text
components = n
```

Every successful union reduces components by one.

At the end:

```text
answer = components - 1
```

provided there were at least `n-1` edges.

---

## 82.1 Java

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

# 83. Redundant Connection

Given an edge list that started as a tree and then received one extra edge:

- A tree has no cycle.
- The extra edge creates one cycle.
- DSU detects the first edge whose endpoints already have the same root.

Template:

```java
for (int[] edge : edges) {
    if (!dsu.union(edge[0], edge[1])) {
        return edge;
    }
}
```

---

# 84. Number of Provinces

A province is a connected component in an undirected graph represented by an adjacency matrix.

Approaches:

- DFS
- BFS
- DSU

This is a useful representation-conversion problem.

---

# 85. Grid Graph vs Explicit Graph

Do not automatically construct an adjacency list for a grid.

For an `R × C` grid:

- Vertices are usually cells.
- Edges are implied by neighboring coordinates.

Constructing an explicit graph can waste memory.

Instead, traverse neighbors directly using direction arrays.

Complexity usually remains:

\[
O(RC)
\]

---

# 86. Common Grid Patterns

| Problem pattern | Typical algorithm |
|---|---|
| Count regions | DFS/BFS |
| Flood fill | DFS/BFS |
| Shortest unweighted path | BFS |
| Nearest source | Multi-source BFS |
| Spread over time | Multi-source BFS |
| Weighted movement | Dijkstra/0-1 BFS |
| Connected components | DFS/BFS/DSU |

---

# 87. Graph Traversal Invariants

### DFS

Before recursively entering `v`:

```text
v is not visited
```

After marking:

```text
visited[v] = true
```

Never recursively enter an already processed vertex unless the algorithm specifically needs that edge.

### BFS

When using a distance array:

```text
dist[v] != -1
```

means `v` has already been discovered.

### Dijkstra

When processing a non-stale state:

```text
current distance = best known distance
```

### DSU

Two vertices have the same root iff they currently belong to the same component.

---

# 88. Common Graph Bugs

## Bug 1 — Forgetting disconnected components

Wrong:

```java
dfs(0);
```

when all components matter.

Correct:

```java
for (int i = 0; i < n; i++) {
    if (!visited[i]) {
        dfs(i);
    }
}
```

---

## Bug 2 — Using Dijkstra with negative edges

Invalid.

Use Bellman-Ford or another algorithm whose assumptions fit the problem.

---

## Bug 3 — Forgetting both directions

For an undirected edge:

```text
u — v
```

add:

```java
u → v
v → u
```

---

## Bug 4 — Wrong directed edge direction

If:

```text
course a requires course b
```

the dependency edge is:

```text
b → a
```

---

## Bug 5 — Treating every visited neighbor as an undirected cycle

In undirected DFS, ignore the parent edge.

---

## Bug 6 — Using one visited array for directed cycle detection

Use 3 states or an equivalent recursion-stack mechanism.

---

## Bug 7 — Incorrect articulation root condition

Root is an articulation point only when it has at least two DFS children.

---

## Bug 8 — Bridge inequality mistake

Bridge:

```text
low[v] > disc[u]
```

not:

```text
low[v] >= disc[u]
```

---

## Bug 9 — Articulation inequality mistake

Non-root articulation condition:

```text
low[v] >= disc[u]
```

---

## Bug 10 — Integer overflow

For weighted shortest paths, use `long` when path sums can exceed `int`.

Do not use:

```java
Integer.MAX_VALUE + weight
```

without checking reachability first.

---

# 89. Graph Algorithm Complexity Table

Let:

- `V` = number of vertices
- `E` = number of edges

| Algorithm | Time | Typical space |
|---|---:|---:|
| DFS | \(O(V+E)\) | \(O(V)\) |
| BFS | \(O(V+E)\) | \(O(V)\) |
| Multi-source BFS | \(O(V+E)\) | \(O(V)\) |
| Kahn | \(O(V+E)\) | \(O(V)\) |
| DFS topo | \(O(V+E)\) | \(O(V)\) |
| Bipartite check | \(O(V+E)\) | \(O(V)\) |
| Bridges | \(O(V+E)\) | \(O(V)\) |
| Articulation points | \(O(V+E)\) | \(O(V)\) |
| Dijkstra + heap | \(O((V+E)\log V)\) | \(O(V+E)\) |
| 0-1 BFS | \(O(V+E)\) | \(O(V)\) |
| Bellman-Ford | \(O(VE)\) | \(O(V)\) |
| Floyd-Warshall | \(O(V^3)\) | \(O(V^2)\) |
| Kruskal | \(O(E\log E)\) | \(O(V+E)\) |
| Prim + heap | \(O(E\log V)\) | \(O(V+E)\) |
| DSU operation | \(O(\alpha(V))\) amortized | \(O(V)\) |
| Kosaraju | \(O(V+E)\) | \(O(V+E)\) |
| Tarjan SCC | \(O(V+E)\) | \(O(V)\) |

---

# 90. Choosing the Representation

### Use adjacency matrix when:

- `V` is small.
- Graph is dense.
- Constant-time edge lookup matters.

### Use adjacency list when:

- Graph is sparse.
- Traversal is needed.
- `V` and `E` can be large.

### Use edge list when:

- Sorting edges.
- Kruskal.
- Bellman-Ford.
- Input naturally comes as edges.

---

# 91. Choosing DFS vs BFS

### DFS

Prefer when:

- Exploring entire components.
- Detecting cycles.
- Backtracking.
- Topological sort.
- Bridges/articulation points.
- SCC algorithms.
- Recursive structural logic.

### BFS

Prefer when:

- Shortest number of edges.
- Level-by-level processing.
- Minimum number of moves.
- Multi-source expansion.
- Bipartite coloring.

---

# 92. Graph Pattern Recognition

When you see:

### "Can I reach...?"

Think:

```text
DFS / BFS
```

### "How many groups/components?"

Think:

```text
DFS / BFS / DSU
```

### "Minimum number of moves" on unweighted states

Think:

```text
BFS
```

### "Nearest source"

Think:

```text
Multi-source BFS
```

### "Prerequisites/dependencies"

Think:

```text
Directed graph + topological sort
```

### "Can all tasks be completed?"

Think:

```text
Directed cycle detection
```

### "No odd cycle / two groups"

Think:

```text
Bipartite coloring
```

### "Shortest weighted path"

Check weights:

```text
0/1      → 0-1 BFS
nonnegative → Dijkstra
negative  → Bellman-Ford
```

### "Connect everything at minimum total cost"

Think:

```text
MST
```

Then:

```text
Kruskal + DSU
or
Prim + PQ
```

### "Critical roads"

Think:

```text
Bridges
```

### "Critical cities"

Think:

```text
Articulation points
```

### "Mutually reachable groups in directed graph"

Think:

```text
SCC
```

### "Dynamic connectivity"

Think:

```text
DSU
```

---

# 93. A More Formal Graph Decision Framework

```text
                    GRAPH
                      |
          +-----------+-----------+
          |                       |
      Directed                Undirected
          |                       |
     +----+----+             +----+----+
     |         |             |         |
   Cycle?   Ordering?      Cycle?   Bipartite?
     |         |             |         |
   3-state   Topo          DFS/DSU   2-color
   DFS       Kahn/DFS
     |
     +-------------------------------+
                                     |
                              SCC / reachability
                              Kosaraju / Tarjan


Weighted?
    |
    +-- No ------------------------> BFS
    |
    +-- 0/1 -----------------------> 0-1 BFS
    |
    +-- Nonnegative ---------------> Dijkstra
    |
    +-- Negative ------------------> Bellman-Ford
    |
    +-- All pairs -----------------> Floyd-Warshall


Connect all with minimum cost?
    |
    +--> MST
          |
          +--> Kruskal + DSU
          |
          +--> Prim + PQ
```

---

# 94. Important Theoretical Facts for GATE

## 94.1 Handshaking lemma

For an undirected graph:

\[
\sum_{v\in V} degree(v)=2E
\]

Therefore the number of odd-degree vertices is always even.

---

## 94.2 Directed degree sum

For a directed graph:

\[
\sum indegree(v)=E
\]

and:

\[
\sum outdegree(v)=E
\]

---

## 94.3 Complete undirected graph

A complete graph with `V` vertices has:

\[
E=\frac{V(V-1)}{2}
\]

---

## 94.4 Complete directed graph

Without self-loops, a complete directed graph has:

\[
E=V(V-1)
\]

---

## 94.5 Tree

An undirected tree with `V` vertices has:

\[
E=V-1
\]

A connected undirected graph with `V-1` edges is a tree.

An acyclic undirected graph with `V-1` edges is connected and therefore a tree.

---

## 94.6 Forest

A forest is an acyclic undirected graph.

If it has `V` vertices and `C` components:

\[
E=V-C
\]

---

## 94.7 Connected graph minimum edges

A connected undirected graph with `V` vertices requires at least:

\[
V-1
\]

edges.

---

## 94.8 DAG

A directed acyclic graph always has at least one topological ordering.

A directed graph with a cycle cannot have a topological ordering.

---

## 94.9 BFS shortest-path property

BFS gives shortest path lengths measured in number of edges in an unweighted graph.

---

## 94.10 Dijkstra condition

Dijkstra is valid when all edge weights are non-negative.

---

## 94.11 Negative cycles

A reachable negative cycle means shortest path distances may not be well-defined because the cycle can be traversed repeatedly to decrease cost without bound.

---

## 94.12 MST edge count

Every spanning tree of a connected graph has:

\[
V-1
\]

edges.

---

## 94.13 SCC condensation

Contracting each SCC into one vertex produces a DAG.

---

# 95. GATE-Style Solved Question 1 — Degree Sum

An undirected graph has 12 vertices. The degrees of 11 vertices are:

```text
1, 2, 2, 3, 3, 4, 4, 5, 5, 6, 7
```

The degree of the remaining vertex is 4.

Find the number of edges.

### Solution

Sum degrees:

\[
1+2+2+3+3+4+4+5+5+6+7+4
\]

\[
=46
\]

By the handshaking lemma:

\[
2E=46
\]

Therefore:

\[
E=23
\]

### Answer

\[
\boxed{23}
\]

---

# 96. GATE-Style Solved Question 2 — Tree Edges

A connected undirected graph has 20 vertices and 19 edges.

Can it contain a cycle?

### Solution

A connected graph with `V` vertices and exactly `V-1` edges is a tree.

Here:

\[
E=19=20-1
\]

Therefore the graph is a tree and contains no cycle.

### Answer

\[
\boxed{\text{No}}
\]

---

# 97. GATE-Style Solved Question 3 — Topological Ordering

Consider:

```text
0 → 1
0 → 2
1 → 3
2 → 3
```

Which of the following can be a valid topological order?

```text
A. 0,1,2,3
B. 1,0,2,3
C. 0,2,1,3
D. 3,0,1,2
```

### Solution

Every source must precede its destination.

`0` must precede `1` and `2`.

`1` and `2` must precede `3`.

Therefore:

- A: valid.
- B: invalid because `0` must precede `1`.
- C: valid.
- D: invalid because `3` must come after `1` and `2`.

### Answer

\[
\boxed{A,C}
\]

Multiple topological orders can exist.

---

# 98. GATE-Style Solved Question 4 — Shortest Path

A graph has only non-negative edge weights. Which algorithm is the standard choice for single-source shortest paths?

### Solution

The graph is weighted and all weights are non-negative.

Therefore Dijkstra is applicable.

BFS is appropriate only when edge costs are effectively equal/unweighted.

Bellman-Ford is more general because it supports negative edges, but is slower.

### Answer

\[
\boxed{\text{Dijkstra}}
\]

---

# 99. GATE-Style Solved Question 5 — Bipartite Graph

An undirected graph contains an odd cycle.

Can it be bipartite?

### Solution

A graph is bipartite if and only if it contains no odd cycle.

Therefore the existence of an odd cycle makes 2-coloring impossible.

### Answer

\[
\boxed{\text{No}}
\]

---

# 100. GATE-Style Solved Question 6 — MST

A connected weighted undirected graph has 8 vertices.

How many edges does every MST contain?

### Solution

Every spanning tree on `V` vertices contains:

\[
V-1
\]

edges.

Therefore:

\[
8-1=7
\]

### Answer

\[
\boxed{7}
\]

---

# 101. Advanced Graph Topics to Know

For strong FAANG preparation, the following should also be recognized even if they are separate advanced problems:

- DAG shortest paths
- Eulerian path/circuit
- Hamiltonian path concepts
- State-space graphs
- Word transformation graphs
- Implicit graphs
- Bidirectional BFS
- A* concept
- Minimum-cost connectivity
- Offline connectivity
- SCC condensation
- Low-link algorithms
- Critical edges/nodes
- Graph cloning
- Graph serialization
- Constraint graphs
- Union-Find with metadata
- 2-SAT concept
- Maximum flow / min-cut
- Bipartite matching

The current section focuses on the core graph algorithms requested above. Flow and matching are normally treated as separate advanced topics.

---

# 102. LeetCode Graph Roadmap

## Level 1 — Foundation

### 1. Number of Islands — LC 200

Learn:

- Grid as graph
- DFS
- BFS
- Connected components

### 2. Flood Fill — LC 733

Learn:

- Grid DFS
- Boundary checks
- Visited-state mutation

### 3. Clone Graph — LC 133

Learn:

- Graph traversal
- Mapping old nodes to cloned nodes

### 4. Number of Provinces — LC 547

Learn:

- Matrix representation
- DFS/BFS
- DSU

---

# 103. LeetCode — BFS / Multi-Source

### 5. Rotting Oranges — LC 994

Learn:

- Multi-source BFS
- Level-by-level time simulation

### 6. Word Ladder — LC 127

Learn:

- Implicit graph
- BFS
- State transformation

---

# 104. LeetCode — Directed Graphs

### 7. Course Schedule — LC 207

Learn:

- Directed cycle detection
- Kahn
- DFS states

### 8. Course Schedule II — LC 210

Learn:

- Topological ordering
- Dependency graph

---

# 105. LeetCode — Bipartite / DSU

### 9. Is Graph Bipartite? — LC 785

Learn:

- 2-coloring
- BFS/DFS
- Disconnected graphs

### 10. Redundant Connection — LC 684

Learn:

- DSU
- Cycle detection

### 11. Number of Operations to Make Network Connected — LC 1319

Learn:

- DSU
- Components
- Edge-count feasibility

---

# 106. LeetCode — Shortest Paths

### 12. Network Delay Time — LC 743

Learn:

- Dijkstra
- Priority queue
- Stale entries

### 13. Cheapest Flights Within K Stops — LC 787

Learn:

- Bounded relaxations
- Bellman-Ford-style reasoning
- State constraints

### 14. Path With Minimum Effort — LC 1631

Learn:

- Dijkstra
- Alternative MST reasoning

---

# 107. LeetCode — MST

### 15. Min Cost to Connect All Points — LC 1584

Learn:

- Prim
- Kruskal
- MST modeling

---

# 108. LeetCode — Bridges

### 16. Critical Connections in a Network — LC 1192

Learn:

- Tarjan low-link
- Discovery time
- Bridges

---

# 109. Additional High-Value Graph Problems

After the core list, study:

- Evaluate Division — LC 399
- Reconstruct Itinerary — LC 332
- Pacific Atlantic Water Flow — LC 417
- Surrounded Regions — LC 130
- Keys and Rooms — LC 841
- All Paths From Source Lead to Destination — LC 1059
- Minimum Number of Vertices to Reach All Nodes — LC 1557
- Find Eventual Safe States — LC 802
- Shortest Path in Binary Matrix — LC 1091
- 01 Matrix — LC 542
- As Far from Land as Possible — LC 1162
- Minimum Genetic Mutation — LC 433
- Accounts Merge — LC 721
- Most Stones Removed with Same Row or Column — LC 947

---

# 110. Graph Problem Ladder

## Stage 1

Master:

```text
DFS
BFS
Grid traversal
Connected components
Cycle detection
```

## Stage 2

Master:

```text
Multi-source BFS
Bipartite
Topological sort
Kahn
Course Schedule
DSU
```

## Stage 3

Master:

```text
Dijkstra
0-1 BFS
Bellman-Ford
MST
Kruskal
Prim
```

## Stage 4

Master:

```text
Bridges
Articulation points
SCC
Kosaraju
Tarjan
```

## Stage 5

Recognize:

```text
Implicit graphs
State-space graphs
Bidirectional BFS
Eulerian paths
Advanced connectivity
Flow/matching
```

---

# 111. Java Graph Coding Checklist

Before submitting a graph solution, check:

### Representation

- [ ] Correct number of vertices.
- [ ] Correct edge direction.
- [ ] Both directions added for undirected graph.
- [ ] Correct weight representation.

### Traversal

- [ ] Visited state initialized.
- [ ] Disconnected components handled if required.
- [ ] Grid boundaries checked.
- [ ] Parent handled for undirected cycle DFS.
- [ ] Recursion depth is safe enough.

### Shortest path

- [ ] Edge-weight assumptions verified.
- [ ] Dijkstra only for non-negative weights.
- [ ] Stale PQ entries handled.
- [ ] `long` used where required.
- [ ] Unreachable nodes handled.

### Topological sort

- [ ] Edge direction models prerequisite correctly.
- [ ] Indegree computed correctly.
- [ ] Cycle handled.

### MST

- [ ] Graph is undirected and weighted.
- [ ] DSU initialized correctly for Kruskal.
- [ ] Exactly `V-1` edges required for a connected MST.
- [ ] Disconnected input handled as a forest if appropriate.

### Tarjan

- [ ] Discovery times initialized.
- [ ] Low values updated correctly.
- [ ] Root articulation rule handled.
- [ ] Bridge uses `>`.
- [ ] Articulation uses `>=`.
- [ ] Parallel-edge issue considered if relevant.

---

# 112. Graph Interview Templates

## Template A — Reachability

```java
visited[source] = true;

for (int next : graph.get(source)) {
    if (!visited[next]) {
        dfs(next);
    }
}
```

---

## Template B — BFS Shortest Distance

```java
dist[source] = 0;
queue.offer(source);

while (!queue.isEmpty()) {
    int u = queue.poll();

    for (int v : graph.get(u)) {
        if (dist[v] == -1) {
            dist[v] = dist[u] + 1;
            queue.offer(v);
        }
    }
}
```

---

## Template C — Directed Cycle

```java
state[u] = 1;

for (int v : graph.get(u)) {
    if (state[v] == 1) {
        return true;
    }

    if (state[v] == 0 && dfs(v)) {
        return true;
    }
}

state[u] = 2;
```

---

## Template D — Kahn

```java
compute indegree

enqueue all indegree-0 vertices

while (!queue.isEmpty()) {
    u = queue.poll();
    order.add(u);

    for (v : graph[u]) {
        if (--indegree[v] == 0) {
            queue.offer(v);
        }
    }
}

if (order.size() != n) {
    cycle exists;
}
```

---

## Template E — Dijkstra

```java
dist[source] = 0;
pq.offer(source);

while (!pq.isEmpty()) {
    state = pq.poll();

    if (stale(state)) {
        continue;
    }

    for (edge : graph[state.node]) {
        relax(edge);
    }
}
```

---

## Template F — DSU

```java
for (edge : edges) {
    if (union(edge.u, edge.v)) {
        // merged
    } else {
        // already connected
    }
}
```

---

## Template G — Bridge

```java
disc[u] = low[u] = timer++;

for (v : graph[u]) {
    if (v is parent) continue;

    if (v is unvisited) {
        dfs(v);

        low[u] = min(low[u], low[v]);

        if (low[v] > disc[u]) {
            // bridge
        }
    } else {
        low[u] = min(low[u], disc[v]);
    }
}
```

---

# 113. What to Memorize vs What to Understand

## Memorize

- DFS template
- BFS template
- Grid directions
- Directed 3-state cycle detection
- Kahn
- Dijkstra PQ pattern
- DSU implementation
- Bridge condition
- Articulation condition

## Understand deeply

- Why BFS gives shortest unweighted paths.
- Why Dijkstra fails with negative edges.
- Why Bellman-Ford detects negative cycles.
- Why topological order exists exactly for DAGs.
- Why DSU detects cycles.
- Why MST differs from shortest paths.
- Why low-link values identify bridges/articulation points.
- Why SCC condensation is a DAG.
- Why multi-source BFS computes distance to the nearest source.

---

# 114. Graph Mastery Checklist

You should be able to solve these without looking at notes:

## Representation

- [ ] Build adjacency matrix.
- [ ] Build adjacency list.
- [ ] Build weighted adjacency list.
- [ ] Process edge list.

## Traversal

- [ ] Recursive DFS.
- [ ] Iterative DFS.
- [ ] BFS.
- [ ] Multi-source BFS.
- [ ] Disconnected graph traversal.
- [ ] Grid DFS/BFS.

## Basic

- [ ] Connected components.
- [ ] Number of Islands.
- [ ] Flood Fill.
- [ ] Undirected cycle detection.
- [ ] Grid traversal.

## Directed

- [ ] Directed cycle detection.
- [ ] Kahn.
- [ ] DFS topological sort.
- [ ] Course Schedule.
- [ ] Course Schedule II.

## Undirected

- [ ] Bipartite graph.
- [ ] DSU cycle detection.
- [ ] Bridges.
- [ ] Articulation points.

## Shortest path

- [ ] BFS shortest path.
- [ ] Dijkstra.
- [ ] 0-1 BFS.
- [ ] Bellman-Ford.
- [ ] Floyd-Warshall.
- [ ] Select the correct algorithm from constraints.

## MST

- [ ] Kruskal.
- [ ] Prim.
- [ ] Cut property.
- [ ] Cycle property.
- [ ] DSU.

## Advanced

- [ ] SCC definition.
- [ ] Kosaraju.
- [ ] Tarjan SCC.
- [ ] Low-link reasoning.
- [ ] Network connectivity.

---

# 115. Final Revision Sheet

## Graph representations

```text
Matrix:
space O(V²)
edge lookup O(1)

Adjacency list:
space O(V+E)
neighbor traversal O(deg(v))

Edge list:
space O(E)
excellent for Kruskal/Bellman-Ford
```

## Traversal

```text
DFS → deep exploration
BFS → level exploration
Multi-source BFS → nearest source / simultaneous spread
```

## Cycles

```text
Undirected:
DFS + parent
or DSU

Directed:
3-state DFS
or Kahn
```

## Ordering

```text
Topological sort
only for DAGs
```

## Bipartite

```text
2-coloring
⇔ no odd cycle
```

## Shortest paths

```text
Unweighted → BFS
0/1 weights → 0-1 BFS
Nonnegative → Dijkstra
Negative edges → Bellman-Ford
All pairs → Floyd-Warshall
DAG → topological relaxation
```

## MST

```text
Kruskal → sort edges + DSU
Prim → priority queue + grow tree
```

## DSU

```text
find
union
path compression
union by size/rank
```

## Low-link

```text
Bridge:
low[v] > disc[u]

Articulation:
low[v] >= disc[u]
(non-root)

Root articulation:
DFS children > 1
```

## SCC

```text
Kosaraju:
DFS + transpose + DFS

Tarjan:
one DFS + low-link + stack
```

---

# 116. Final Graph Mastery Standard

Do not consider Graphs mastered merely because you can reproduce DFS and BFS.

You should be able to look at an unfamiliar problem and answer:

1. **What are the vertices?**
2. **What are the edges?**
3. **Is the graph directed?**
4. **Are edges weighted?**
5. **Can weights be negative?**
6. **Is the graph implicit, such as a grid or state space?**
7. **Is the problem about reachability?**
8. **Is it about connected components?**
9. **Is it about cycles?**
10. **Is it about ordering/dependencies?**
11. **Is it about bipartite coloring?**
12. **Is it about shortest paths?**
13. **Is it about connecting everything cheaply?**
14. **Is it about critical edges or vertices?**
15. **Is it about mutually reachable directed groups?**
16. **Are connectivity operations dynamic enough for DSU?**

The core mental mapping should become:

```text
Reachability
    → DFS / BFS

Components
    → DFS / BFS / DSU

Unweighted shortest path
    → BFS

Nearest source / spread
    → Multi-source BFS

Directed dependencies
    → Topological sort

Directed cycle
    → 3-state DFS / Kahn

Undirected cycle
    → Parent DFS / DSU

Two groups
    → Bipartite coloring

0/1 shortest path
    → 0-1 BFS

Nonnegative weighted shortest path
    → Dijkstra

Negative edges
    → Bellman-Ford

All-pairs shortest path
    → Floyd-Warshall

Minimum-cost connection
    → MST

MST with sorted edges
    → Kruskal + DSU

MST with frontier expansion
    → Prim + PQ

Critical edge
    → Bridge / low-link

Critical vertex
    → Articulation point / low-link

Mutually reachable directed group
    → SCC

SCC
    → Kosaraju / Tarjan
```

This mapping is the foundation for solving unfamiliar graph problems efficiently.
