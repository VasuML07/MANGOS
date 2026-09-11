# 7. Queue & Deque

## 1. What You Must Master

Queue and deque problems are important for interviews because they form the foundation of:

- BFS
- Level-order traversal
- Shortest paths in unweighted graphs
- Multi-source BFS
- Sliding-window optimization
- Scheduling/simulation
- Producer-consumer style processing
- Monotonic deque techniques

### Core topics

- Queue implementation
- Circular queue
- Deque
- BFS
- Monotonic deque
- Sliding-window maximum
- Multi-source BFS
- Queue-based simulation

### Interview priority

| Topic | Priority | Typical difficulty |
|---|---|---|
| Queue basics | High | Easy |
| Circular queue | High | Easy–Medium |
| Deque | High | Easy |
| BFS | Very High | Easy–Medium |
| Grid BFS | Very High | Medium |
| Multi-source BFS | Very High | Medium |
| Monotonic deque | Very High | Medium |
| Sliding-window maximum | Essential | Hard/Medium |
| Queue simulation | High | Easy–Medium |

---

# 2. Queue Fundamentals

## 2.1 What is a Queue?

A queue follows:

> **FIFO — First In, First Out**

The first inserted element is the first removed.

Example:

```text
enqueue(10)
enqueue(20)
enqueue(30)

front → 10 20 30 ← rear
```

`dequeue()` removes `10`.

---

# 3. Core Queue Operations

| Operation | Meaning | Typical time |
|---|---|---:|
| Enqueue | Insert at rear | O(1) |
| Dequeue | Remove from front | O(1) |
| Front/Peek | Read front | O(1) |
| IsEmpty | Check empty | O(1) |
| Size | Number of elements | O(1) |

A queue is useful when processing must happen in arrival order.

---

# 4. Queue in Java

For interviews, use:

```java
Queue<Integer> queue = new ArrayDeque<>();

queue.offer(10);
queue.offer(20);
queue.offer(30);

System.out.println(queue.peek()); // 10
System.out.println(queue.poll()); // 10
```

Preferred methods:

```java
offer()
poll()
peek()
```

Avoid relying on exceptions from:

```java
add()
remove()
element()
```

unless their exception behavior is specifically needed.

---

# 5. Queue Implementation Using an Array

A naive implementation shifts every element after dequeue:

```java
for (int i = 1; i < size; i++) {
    arr[i - 1] = arr[i];
}
```

That makes dequeue:

```text
O(n)
```

A proper queue should avoid shifting.

Use:

```text
front
rear
size
```

and wrap around the array.

This leads to the **circular queue**.

---

# 6. Circular Queue

A circular queue treats the array as circular.

For capacity `C`:

```text
nextIndex = (index + 1) % C
```

Example:

```text
capacity = 5

0 1 2 3 4
↑       ↑
front   rear
```

After reaching index `4`, the next position is:

```text
(4 + 1) % 5 = 0
```

---

# 7. Circular Queue State

A clean implementation maintains:

```text
front = index of first element
rear  = index where next element will be inserted
size  = number of elements
```

Then:

```text
empty → size == 0
full  → size == capacity
```

This avoids ambiguity between full and empty states.

---

# 8. Circular Queue Java Implementation

```java
class MyCircularQueue {
    private final int[] data;
    private int front;
    private int rear;
    private int size;

    public MyCircularQueue(int k) {
        data = new int[k];
        front = 0;
        rear = 0;
        size = 0;
    }

    public boolean enQueue(int value) {
        if (isFull()) {
            return false;
        }

        data[rear] = value;
        rear = (rear + 1) % data.length;
        size++;

        return true;
    }

    public boolean deQueue() {
        if (isEmpty()) {
            return false;
        }

        front = (front + 1) % data.length;
        size--;

        return true;
    }

    public int Front() {
        return isEmpty() ? -1 : data[front];
    }

    public int Rear() {
        if (isEmpty()) {
            return -1;
        }

        int index = (rear - 1 + data.length) % data.length;
        return data[index];
    }

    public boolean isEmpty() {
        return size == 0;
    }

    public boolean isFull() {
        return size == data.length;
    }
}
```

### Complexity

Every operation:

```text
O(1)
```

Space:

```text
O(k)
```

---

# 9. Circular Queue: Common Bugs

## Bug 1 — Negative modulo

In Java:

```java
(-1) % n
```

is negative.

Use:

```java
(rear - 1 + n) % n
```

instead of:

```java
(rear - 1) % n
```

## Bug 2 — Confusing rear

Decide whether `rear` means:

```text
last element
```

or:

```text
next insertion position
```

The implementation above uses:

```text
rear = next insertion position
```

Do not mix the two definitions.

## Bug 3 — Full vs empty ambiguity

If only `front` and `rear` are stored, a state where:

```text
front == rear
```

can represent either:

```text
empty
```

or:

```text
full
```

Track `size` or reserve one slot.

---

# 10. Deque

A **deque** (double-ended queue) allows insertion and deletion at both ends.

Operations:

```text
addFirst
addLast
removeFirst
removeLast
peekFirst
peekLast
```

Example:

```text
front ← 10 20 30 → back
```

You can:

```text
removeFirst() → 10
removeLast()  → 30
```

---

# 11. Deque in Java

Use:

```java
Deque<Integer> deque = new ArrayDeque<>();
```

### Front operations

```java
deque.offerFirst(x);
deque.pollFirst();
deque.peekFirst();
```

### Back operations

```java
deque.offerLast(x);
deque.pollLast();
deque.peekLast();
```

Equivalent stack operations:

```java
deque.push(x);
deque.pop();
deque.peek();
```

Equivalent queue operations:

```java
deque.offer(x);
deque.poll();
deque.peek();
```

This makes `Deque` one of the most useful Java interview data structures.

---

# 12. Queue vs Deque vs Stack

| Structure | Insert | Remove |
|---|---|---|
| Stack | One end | Same end |
| Queue | Rear | Front |
| Deque | Both ends | Both ends |

Java:

```java
Deque<Integer> dq = new ArrayDeque<>();
```

can implement both stack and queue behavior.

---

# 13. BFS — Breadth-First Search

BFS explores a graph level by level.

Example:

```text
       1
      / \
     2   3
    / \
   4   5
```

BFS order:

```text
1 → 2 → 3 → 4 → 5
```

Use a queue.

---

# 14. Basic Graph BFS

Given an adjacency list:

```java
List<List<Integer>> graph;
```

### Java

```java
class BFS {
    public void bfs(List<List<Integer>> graph, int start) {
        int n = graph.size();

        boolean[] visited = new boolean[n];
        Queue<Integer> queue = new ArrayDeque<>();

        queue.offer(start);
        visited[start] = true;

        while (!queue.isEmpty()) {
            int u = queue.poll();

            System.out.println(u);

            for (int v : graph.get(u)) {
                if (!visited[v]) {
                    visited[v] = true;
                    queue.offer(v);
                }
            }
        }
    }
}
```

### Complexity

For an adjacency-list graph:

```text
Time:  O(V + E)
Space: O(V)
```

---

# 15. Why Mark Visited When Enqueuing?

Use:

```java
visited[v] = true;
queue.offer(v);
```

at the same time.

Do not wait until dequeue unless you intentionally handle duplicate queue entries.

Otherwise multiple parents can enqueue the same vertex.

---

# 16. BFS Shortest Path in an Unweighted Graph

For an unweighted graph:

> BFS finds the shortest path in terms of number of edges.

Example:

```text
A -- B -- C
|         |
D -- E -- F
```

Starting from `A`, BFS discovers nodes by minimum edge distance.

Maintain:

```java
int[] dist = new int[n];
Arrays.fill(dist, -1);
```

Then:

```java
dist[start] = 0;
```

When discovering `v` from `u`:

```java
dist[v] = dist[u] + 1;
```

---

# 17. BFS Distance Template

```java
int[] dist = new int[n];
Arrays.fill(dist, -1);

Queue<Integer> queue = new ArrayDeque<>();

dist[start] = 0;
queue.offer(start);

while (!queue.isEmpty()) {
    int u = queue.poll();

    for (int v : graph.get(u)) {
        if (dist[v] != -1) {
            continue;
        }

        dist[v] = dist[u] + 1;
        queue.offer(v);
    }
}
```

This is one of the most reusable BFS templates.

---

# 18. BFS Parent Reconstruction

If the actual shortest path is required, store:

```java
parent[v] = u;
```

when `v` is first discovered.

Then reconstruct:

```text
target → parent[target] → ... → source
```

and reverse the result.

---

# 19. BFS Level Order

A common BFS pattern is processing one complete level at a time.

Use:

```java
int size = queue.size();

for (int i = 0; i < size; i++) {
    int node = queue.poll();

    // process current level
}
```

### Important

Capture:

```java
int size = queue.size();
```

before the loop.

Do not repeatedly use a changing queue size as the level boundary.

---

# 20. Binary Tree Level Order Traversal

For a tree:

```java
Queue<TreeNode> queue = new ArrayDeque<>();
queue.offer(root);

while (!queue.isEmpty()) {
    int size = queue.size();

    for (int i = 0; i < size; i++) {
        TreeNode node = queue.poll();

        // process node

        if (node.left != null) {
            queue.offer(node.left);
        }

        if (node.right != null) {
            queue.offer(node.right);
        }
    }
}
```

Time:

```text
O(n)
```

Space:

```text
O(width)
```

---

# 21. Grid BFS

Many interview problems are graph problems disguised as matrices.

Treat every cell as a graph node.

Typical neighbors:

```text
up
down
left
right
```

Direction array:

```java
int[][] dirs = {
    {1, 0},
    {-1, 0},
    {0, 1},
    {0, -1}
};
```

Template:

```java
Queue<int[]> queue = new ArrayDeque<>();
boolean[][] visited = new boolean[m][n];

queue.offer(new int[]{sr, sc});
visited[sr][sc] = true;

while (!queue.isEmpty()) {
    int[] cell = queue.poll();

    for (int[] d : dirs) {
        int nr = cell[0] + d[0];
        int nc = cell[1] + d[1];

        if (nr < 0 || nr >= m || nc < 0 || nc >= n) {
            continue;
        }

        if (visited[nr][nc]) {
            continue;
        }

        visited[nr][nc] = true;
        queue.offer(new int[]{nr, nc});
    }
}
```

---

# 22. BFS on Implicit Graphs

You do not always have an adjacency list.

The graph may be implicit.

Examples:

- Grid movement
- Word transformations
- Lock combinations
- Knight moves
- State transitions
- Puzzle configurations

The key question is:

> From the current state, what states can I reach in one legal move?

Those generated states become BFS neighbors.

---

# 23. Multi-Source BFS

Normal BFS:

```text
one starting node
```

Multi-source BFS:

```text
many starting nodes
```

Initialize the queue with **all sources at distance 0**.

Then run ordinary BFS.

This computes the minimum distance from every node to the nearest source.

---

# 24. Multi-Source BFS Template

```java
Queue<int[]> queue = new ArrayDeque<>();

for (each source) {
    dist[source] = 0;
    queue.offer(source);
}

while (!queue.isEmpty()) {
    int[] cur = queue.poll();

    for (each neighbor) {
        if (dist[neighbor] != -1) {
            continue;
        }

        dist[neighbor] = dist[cur] + 1;
        queue.offer(neighbor);
    }
}
```

### Key idea

Do not BFS separately from every source.

That can be unnecessarily expensive.

Instead:

```text
all sources → queue initially
```

Then one BFS wave expands from all of them simultaneously.

---

# 25. Multi-Source BFS Example: Rotting Oranges

Grid contains:

```text
0 = empty
1 = fresh orange
2 = rotten orange
```

Every minute, rotten oranges infect adjacent fresh oranges.

Question:

> How long until all possible oranges rot?

Every initially rotten orange is a source.

Initialize:

```java
for every cell:
    if grid[r][c] == 2:
        queue.offer(new int[]{r, c});
```

Then process BFS level by level.

Each level represents one minute.

---

# 26. Rotting Oranges Java

```java
class Solution {
    public int orangesRotting(int[][] grid) {
        int m = grid.length;
        int n = grid[0].length;

        Queue<int[]> queue = new ArrayDeque<>();
        int fresh = 0;

        for (int r = 0; r < m; r++) {
            for (int c = 0; c < n; c++) {
                if (grid[r][c] == 2) {
                    queue.offer(new int[]{r, c});
                } else if (grid[r][c] == 1) {
                    fresh++;
                }
            }
        }

        int[][] dirs = {
            {1, 0},
            {-1, 0},
            {0, 1},
            {0, -1}
        };

        int minutes = 0;

        while (!queue.isEmpty() && fresh > 0) {
            int size = queue.size();

            for (int i = 0; i < size; i++) {
                int[] cur = queue.poll();

                for (int[] d : dirs) {
                    int nr = cur[0] + d[0];
                    int nc = cur[1] + d[1];

                    if (nr < 0 || nr >= m ||
                        nc < 0 || nc >= n ||
                        grid[nr][nc] != 1) {
                        continue;
                    }

                    grid[nr][nc] = 2;
                    fresh--;

                    queue.offer(new int[]{nr, nc});
                }
            }

            minutes++;
        }

        return fresh == 0 ? minutes : -1;
    }
}
```

Time:

```text
O(mn)
```

Space:

```text
O(mn)
```

---

# 27. Multi-Source BFS: Distance to Nearest Zero

Given a binary matrix, find distance from every cell to its nearest zero.

Instead of running BFS from every cell:

1. Put every `0` in the queue.
2. Set their distance to `0`.
3. Expand simultaneously.

This gives:

```text
O(mn)
```

rather than potentially:

```text
O((mn)^2)
```

---

# 28. Multi-Source BFS Recognition

Think multi-source BFS when the problem asks for:

- Distance to nearest source
- Spread from multiple starting points
- Infection
- Fire/water propagation
- Rotting
- Nearest zero
- Nearest gate
- Simultaneous expansion
- Minimum time until all reachable states are processed

The phrase:

> "nearest among multiple starting positions"

is a strong signal.

---

# 29. Monotonic Deque

A monotonic deque maintains elements in sorted order while supporting removal from both ends.

It is especially useful for:

- Sliding-window maximum
- Sliding-window minimum
- Range optimization
- Some dynamic programming optimizations

---

# 30. Why a Deque Instead of a Stack?

A monotonic stack usually needs:

```text
push/pop from one end
```

A monotonic deque needs:

```text
remove expired elements from front
remove dominated elements from back
insert new element at back
```

That requires both ends.

---

# 31. Sliding-Window Maximum

Given:

```text
nums = [1,3,-1,-3,5,3,6,7]
k = 3
```

Answer:

```text
[3,3,5,5,6,7]
```

Brute force:

```text
find max in every window
```

Complexity:

```text
O(nk)
```

We can do:

```text
O(n)
```

using a monotonic deque.

---

# 32. Sliding-Window Maximum Insight

Maintain a deque of **indices**.

The values corresponding to those indices are decreasing:

```text
front → largest ... smallest ← back
```

For each index `i`:

### Step 1 — Remove expired indices

If:

```text
deque.front <= i - k
```

remove it.

### Step 2 — Remove dominated indices

While:

```text
nums[deque.back] <= nums[i]
```

remove from back.

Why?

Because the new element is:

- newer
- at least as large

Therefore the old element can never become the maximum while both are relevant.

### Step 3 — Add current index

```java
deque.offerLast(i);
```

### Step 4 — Record answer

Once:

```text
i >= k - 1
```

the front is the maximum.

---

# 33. Sliding-Window Maximum Java

```java
class Solution {
    public int[] maxSlidingWindow(int[] nums, int k) {
        int n = nums.length;
        int[] ans = new int[n - k + 1];

        Deque<Integer> deque = new ArrayDeque<>();

        for (int i = 0; i < n; i++) {

            // Remove expired indices.
            while (!deque.isEmpty()
                    && deque.peekFirst() <= i - k) {
                deque.pollFirst();
            }

            // Remove elements dominated by nums[i].
            while (!deque.isEmpty()
                    && nums[deque.peekLast()] <= nums[i]) {
                deque.pollLast();
            }

            deque.offerLast(i);

            // Front is maximum for the current window.
            if (i >= k - 1) {
                ans[i - k + 1] = nums[deque.peekFirst()];
            }
        }

        return ans;
    }
}
```

### Complexity

Each index:

```text
enters once
leaves front at most once
leaves back at most once
```

Therefore:

```text
Time:  O(n)
Space: O(k)
```

---

# 34. Why Store Indices in Sliding Window?

Suppose the deque stored only values.

You could find the maximum, but you would not know whether it has left the window.

Indices solve both problems:

```text
value → nums[index]
index → expiration position
```

Therefore:

> For sliding-window monotonic deque problems, store indices.

---

# 35. Sliding-Window Minimum

Same pattern.

For minimum, maintain increasing values:

```text
front → smallest ... largest ← back
```

Remove from back while:

```java
nums[deque.peekLast()] >= nums[i]
```

Then:

```java
nums[deque.peekFirst()]
```

is the current minimum.

---

# 36. Monotonic Deque General Template

## Maximum

```java
while (!dq.isEmpty()
        && nums[dq.peekLast()] <= nums[i]) {
    dq.pollLast();
}
```

## Minimum

```java
while (!dq.isEmpty()
        && nums[dq.peekLast()] >= nums[i]) {
    dq.pollLast();
}
```

Always remove expired indices from the front first.

---

# 37. Deque Invariant

For sliding-window maximum:

```text
indices increase from front to back
values decrease from front to back
```

For sliding-window minimum:

```text
indices increase from front to back
values increase from front to back
```

This invariant is the core of the technique.

---

# 38. Queue-Based Simulation

Queues are natural when events must happen in order.

Examples:

- Task processing
- CPU scheduling
- Customer service
- Printer queue
- Traffic simulation
- Round-robin processing
- BFS-like spreading
- Time-based events

General structure:

```java
Queue<State> queue = new ArrayDeque<>();

while (!queue.isEmpty()) {
    State current = queue.poll();

    // process current state

    // enqueue newly available states
}
```

---

# 39. Queue Simulation: Time Steps

When each BFS layer represents one unit of time:

```java
int time = 0;

while (!queue.isEmpty()) {
    int size = queue.size();

    for (int i = 0; i < size; i++) {
        // process events occurring at this time
    }

    time++;
}
```

This is common in:

- Rotting Oranges
- Infection/spread
- Fire simulation
- Grid propagation

---

# 40. Queue Simulation: Important Rule

If processing one item creates another item that should be processed later:

```java
queue.offer(newState);
```

Do not recursively process it immediately unless the problem specifically requires depth-first behavior.

Queue simulation preserves chronological/FIFO order.

---

# 41. BFS vs DFS

| Requirement | Better choice |
|---|---|
| Shortest path in unweighted graph | BFS |
| Level-by-level traversal | BFS |
| Minimum number of moves | BFS |
| Nearest source | Multi-source BFS |
| Simultaneous spreading | Multi-source BFS |
| Explore all possibilities without distance requirement | DFS/BFS |
| Recursive tree traversal | DFS |
| Backtracking | DFS |

A key rule:

> BFS gives minimum number of edges in an unweighted graph because it processes states in nondecreasing distance order.

---

# 42. BFS on Weighted Graphs

Standard BFS is not generally correct when edges have arbitrary positive weights.

Use:

- BFS → unweighted edges
- 0-1 BFS → edge weights `0` or `1`
- Dijkstra → nonnegative arbitrary weights
- Bellman-Ford → supports negative edges

This distinction is important for interviews.

---

# 43. 0-1 BFS

When edge weights are only:

```text
0 or 1
```

a deque can replace Dijkstra's priority queue.

For an edge of weight `0`:

```java
deque.offerFirst(v);
```

For weight `1`:

```java
deque.offerLast(v);
```

This maintains states in distance order.

Complexity:

```text
O(V + E)
```

This is an advanced but valuable deque application.

---

# 44. 0-1 BFS Template

```java
Deque<Integer> deque = new ArrayDeque<>();

dist[source] = 0;
deque.offerFirst(source);

while (!deque.isEmpty()) {
    int u = deque.pollFirst();

    for (Edge e : graph[u]) {
        int v = e.to;
        int w = e.weight;

        if (dist[u] + w < dist[v]) {
            dist[v] = dist[u] + w;

            if (w == 0) {
                deque.offerFirst(v);
            } else {
                deque.offerLast(v);
            }
        }
    }
}
```

Use a stale-state check or suitable relaxation strategy when needed.

---

# 45. Common BFS Mistakes

## Mistake 1 — Marking visited too late

Prefer:

```java
visited[v] = true;
queue.offer(v);
```

when discovering `v`.

## Mistake 2 — Using BFS for weighted shortest paths

Standard BFS assumes equal edge cost.

## Mistake 3 — Forgetting disconnected components

If the graph may be disconnected:

```java
for (int i = 0; i < n; i++) {
    if (!visited[i]) {
        bfs(i);
    }
}
```

## Mistake 4 — Wrong level counting

Use:

```java
int size = queue.size();
```

before processing a level.

## Mistake 5 — Mutating a grid inconsistently

When using the grid itself as visited state:

```java
grid[nr][nc] = visitedState;
```

do it when enqueuing.

---

# 46. Common Monotonic Deque Mistakes

## Mistake 1 — Storing values instead of indices

For a sliding window, use:

```java
Deque<Integer>
```

where each integer is an index.

## Mistake 2 — Forgetting expired elements

Remove:

```java
dq.peekFirst() <= i - k
```

before using the front.

## Mistake 3 — Wrong monotonic direction

Maximum:

```text
decreasing values
```

Minimum:

```text
increasing values
```

## Mistake 4 — Forgetting dominated elements

For maximum:

```java
while (nums[dq.peekLast()] <= nums[i]) {
    dq.pollLast();
}
```

## Mistake 5 — Thinking every while-loop makes it O(n²)

Again, each index enters and leaves the deque a bounded number of times.

Total:

```text
O(n)
```

---

# 47. Complexity Summary

| Problem | Time | Space |
|---|---:|---:|
| Queue operation | O(1) | O(1) |
| Circular queue operation | O(1) | O(k) total |
| Graph BFS | O(V + E) | O(V) |
| Grid BFS | O(RC) | O(RC) |
| Multi-source BFS | O(V + E) / O(RC) | O(V) / O(RC) |
| Sliding-window maximum | O(n) | O(k) |
| Sliding-window minimum | O(n) | O(k) |
| 0-1 BFS | O(V + E) | O(V) |
| Queue simulation | Usually O(number of states/events) | Depends on queue |

---

# 48. LeetCode Practice Roadmap

## Easy

### 1. Number of Recent Calls
- Pattern: Queue
- Priority: High
- Core lesson: Remove expired events

### 2. Implement Queue using Stacks
- Pattern: Queue simulation
- Priority: High
- Core lesson: Understand FIFO using LIFO structures

### 3. Design Circular Queue
- Pattern: Circular queue
- Priority: Essential
- Core lesson: Modulo indexing

### 4. First Unique Character in a String
- Pattern: Frequency + queue-style reasoning
- Priority: Medium
- Core lesson: Preserve arrival order after counting

---

# 49. Medium

### 5. Binary Tree Level Order Traversal
- Pattern: BFS
- Priority: Essential
- Core lesson: Level processing

### 6. Rotting Oranges
- Pattern: Multi-source BFS
- Priority: Essential
- Core lesson: Time-layered propagation

### 7. 01 Matrix
- Pattern: Multi-source BFS
- Priority: Very High
- Core lesson: Distance to nearest source

### 8. Open the Lock
- Pattern: BFS on implicit graph
- Priority: High
- Core lesson: State-space BFS

### 9. Perfect Squares
- Pattern: BFS / shortest path interpretation
- Priority: Medium
- Core lesson: Minimum number of transitions

### 10. Sliding Window Maximum
- Pattern: Monotonic deque
- Priority: Essential
- Core lesson: Expiration + dominance

### 11. Dota2 Senate
- Pattern: Queue simulation
- Priority: Medium
- Core lesson: Process participants in order

### 12. Time Needed to Buy Tickets
- Pattern: Queue simulation
- Priority: Medium
- Core lesson: Repeated FIFO processing

### 13. Reveal Cards In Increasing Order
- Pattern: Queue/deque simulation
- Priority: Medium
- Core lesson: Simulate index movement

### 14. Design Hit Counter
- Pattern: Queue / timestamps
- Priority: High
- Core lesson: Expire old events

### 15. Number of Islands
- Pattern: BFS/DFS
- Priority: Essential
- Core lesson: Grid as graph

---

# 50. Hard / Advanced

### 16. Shortest Path in a Grid with Obstacles Elimination
- Pattern: BFS + state
- Priority: Very High
- Core lesson: State includes more than coordinates

### 17. Word Ladder
- Pattern: BFS
- Priority: Essential
- Core lesson: Implicit graph + shortest transformation

### 18. Sliding Window Maximum
- Pattern: Monotonic deque
- Priority: Essential
- Core lesson: O(n) window maximum

### 19. Jump Game VI
- Pattern: DP + monotonic deque
- Priority: Very High
- Core lesson: Optimize sliding-window maximum over DP

### 20. Shortest Path in Binary Matrix
- Pattern: BFS
- Priority: High
- Core lesson: Grid shortest path

### 21. Bus Routes
- Pattern: BFS
- Priority: Very High
- Core lesson: Compress state/graph representation

---

# 51. Recommended Mastery Order

## Phase 1 — Queue fundamentals

1. FIFO concept
2. Array queue
3. Circular queue
4. `Queue`
5. `Deque`

## Phase 2 — BFS

6. Graph BFS
7. BFS distance
8. BFS shortest path
9. Tree level order
10. Grid BFS
11. Implicit-state BFS

## Phase 3 — Multi-source BFS

12. Rotting Oranges
13. 01 Matrix
14. Nearest-source problems
15. Spread/infection problems

## Phase 4 — Monotonic deque

16. Deque invariants
17. Sliding-window maximum
18. Sliding-window minimum
19. DP + deque

## Phase 5 — Advanced

20. 0-1 BFS
21. State-space BFS
22. Large-grid optimization
23. Queue simulations

---

# 52. Pattern Recognition

| Problem wording | Think |
|---|---|
| First in, first out | Queue |
| Process in arrival order | Queue |
| Level by level | BFS |
| Minimum moves | BFS |
| Shortest path, unweighted | BFS |
| Nearest source | Multi-source BFS |
| Multiple starting points | Multi-source BFS |
| Spread over time | Multi-source BFS |
| Maximum in every window | Monotonic deque |
| Minimum in every window | Monotonic deque |
| Remove expired window elements | Deque |
| 0/1 edge weights | 0-1 BFS |
| Repeated chronological events | Queue simulation |
| State transitions | BFS |
| Shortest transformation | BFS |

---

# 53. Queue + BFS Master Template

```java
Queue<Integer> queue = new ArrayDeque<>();
boolean[] visited = new boolean[n];

queue.offer(source);
visited[source] = true;

while (!queue.isEmpty()) {
    int u = queue.poll();

    for (int v : graph.get(u)) {
        if (!visited[v]) {
            visited[v] = true;
            queue.offer(v);
        }
    }
}
```

---

# 54. BFS Level Template

```java
Queue<Integer> queue = new ArrayDeque<>();
queue.offer(source);

int level = 0;

while (!queue.isEmpty()) {
    int size = queue.size();

    for (int i = 0; i < size; i++) {
        int u = queue.poll();

        // process node at current level

        // enqueue next-level states
    }

    level++;
}
```

---

# 55. Multi-Source BFS Template

```java
Queue<int[]> queue = new ArrayDeque<>();

for (each source) {
    queue.offer(source);
    dist[source] = 0;
}

while (!queue.isEmpty()) {
    int[] cur = queue.poll();

    for (each neighbor) {
        if (dist[neighbor] != -1) {
            continue;
        }

        dist[neighbor] = dist[cur] + 1;
        queue.offer(neighbor);
    }
}
```

---

# 56. Sliding-Window Maximum Template

```java
Deque<Integer> dq = new ArrayDeque<>();

for (int i = 0; i < n; i++) {

    // 1. Remove expired indices
    while (!dq.isEmpty()
            && dq.peekFirst() <= i - k) {
        dq.pollFirst();
    }

    // 2. Remove dominated values
    while (!dq.isEmpty()
            && nums[dq.peekLast()] <= nums[i]) {
        dq.pollLast();
    }

    // 3. Add current
    dq.offerLast(i);

    // 4. Record answer
    if (i >= k - 1) {
        ans[i - k + 1] = nums[dq.peekFirst()];
    }
}
```

---

# 57. BFS State Design

For harder BFS problems, first define the complete state.

For example:

```text
(x, y)
```

may not be enough.

A state might be:

```text
(row, col, remainingEliminations)
```

or:

```text
(node, keysMask)
```

or:

```text
(word)
```

The BFS must distinguish states that can have different future possibilities.

### Important rule

> If two visits to the same position have different remaining resources or capabilities, they may be different BFS states.

---

# 58. Visited-State Design

Simple graph:

```java
boolean[] visited
```

Grid:

```java
boolean[][] visited
```

Grid + extra state:

```java
boolean[][][] visited
```

Bitmask state:

```java
boolean[][] visitedByMask
```

The visited structure must represent the full state, not just one coordinate.

---

# 59. BFS Memory Optimization

For some grid problems, you can modify the grid itself:

```java
grid[nr][nc] = visitedValue;
```

instead of allocating:

```java
boolean[][] visited
```

This saves auxiliary memory.

But only do this when modifying the input is allowed.

---

# 60. Queue Simulation Checklist

Before coding a simulation, identify:

1. What is the initial queue?
2. What does one queue element represent?
3. When does an element leave?
4. What new elements can it create?
5. Does processing happen by event order or time layer?
6. Do newly created elements act immediately or later?
7. What condition ends the simulation?
8. Can an event be inserted more than once?

This prevents most simulation bugs.

---

# 61. Deque Checklist

For sliding-window problems, explicitly maintain:

```text
1. Window boundary
2. Expired indices
3. Monotonic invariant
4. Current answer at front
```

For maximum:

```text
values decrease
```

For minimum:

```text
values increase
```

---

# 62. Interview Decision Framework

## Ask 1

Is this about processing items in arrival order?

```text
Queue
```

## Ask 2

Is it level-by-level or shortest path in an unweighted graph?

```text
BFS
```

## Ask 3

Are there multiple simultaneous starting points?

```text
Multi-source BFS
```

## Ask 4

Is there a fixed-size sliding window asking for max/min?

```text
Monotonic deque
```

## Ask 5

Are edge weights only `0` and `1`?

```text
0-1 BFS
```

## Ask 6

Is the problem repeatedly processing events in chronological order?

```text
Queue simulation
```

---

# 63. GATE / CS Theory Focus

For GATE-style preparation, know:

- Queue ADT
- FIFO principle
- Array queue
- Circular queue
- Queue overflow/underflow
- Front/rear representation
- Modulo arithmetic in circular queues
- Deque
- BFS traversal
- BFS complexity
- BFS shortest path in unweighted graphs
- Level-order traversal
- Graph representation and BFS complexity
- Multi-source BFS
- Monotonic deque
- Amortized analysis
- 0-1 BFS concept
- Queue applications and simulations

---

# 64. Three GATE-Style Solved Questions

## Question 1 — Circular Queue

A circular queue has capacity `5`. Initially:

```text
front = 0
rear = 0
size = 0
```

After inserting four elements, deleting two elements, and inserting three more elements, what is the final number of elements?

### Solution

Initial:

```text
size = 0
```

Insert four:

```text
size = 4
```

Delete two:

```text
size = 2
```

Insert three:

```text
size = 5
```

The queue is full.

### Answer

```text
5
```

### Concept

Circular movement changes indices but does not change the number of stored elements.

---

## Question 2 — BFS Shortest Distance

Consider an unweighted graph:

```text
0 -- 1 -- 3
|         |
2 --------
```

Starting BFS from vertex `0`, what is the minimum number of edges required to reach vertex `3`?

### Solution

Start:

```text
distance[0] = 0
```

From `0`:

```text
distance[1] = 1
distance[2] = 1
```

From `1`:

```text
distance[3] = 2
```

Therefore:

```text
0 → 1 → 3
```

has length `2`.

### Answer

```text
2
```

### Concept

BFS discovers vertices in nondecreasing shortest-path distance for unweighted graphs.

---

## Question 3 — Monotonic Deque Complexity

An algorithm processes an array of `n` elements. Every element is inserted into a deque once and removed from the deque at most once from either end.

What is the total time complexity?

A. `O(log n)`  
B. `O(n)`  
C. `O(n log n)`  
D. `O(n²)`

### Solution

Each element has bounded total deque operations:

```text
one insertion
at most one effective removal
```

Therefore total operations are proportional to `n`.

### Answer

**B — O(n)**

### Concept

This is amortized linear-time processing.

---

# 65. Advanced Connections

Queue and deque should not be studied in isolation.

## Queue → BFS

```text
Queue
  ↓
BFS
  ↓
Shortest path
```

## Multi-source BFS

```text
Many sources
     ↓
One initial queue
     ↓
Simultaneous BFS wave
     ↓
Nearest-source distance
```

## Deque → Sliding Window

```text
Deque
  ↓
Remove expired front
  ↓
Remove dominated back
  ↓
Front = optimum
```

## Deque → 0-1 BFS

```text
weight 0 → front
weight 1 → back
```

These are different applications of the same data-structure idea.

---

# 66. Final Mastery Checklist

You are ready to move on only when you can solve these without looking up the pattern.

### Queue

- [ ] Explain FIFO
- [ ] Implement queue with array
- [ ] Explain front/rear
- [ ] Implement circular queue
- [ ] Explain modulo indexing
- [ ] Handle full/empty states
- [ ] Use Java `Queue`
- [ ] Use Java `Deque`

### BFS

- [ ] Graph BFS
- [ ] BFS with distance
- [ ] BFS shortest path
- [ ] BFS parent reconstruction
- [ ] Level-order BFS
- [ ] Grid BFS
- [ ] Implicit graph BFS
- [ ] Disconnected graph BFS

### Multi-source BFS

- [ ] Initialize all sources
- [ ] Compute nearest-source distance
- [ ] Process time layers
- [ ] Solve Rotting Oranges
- [ ] Solve 01 Matrix
- [ ] Recognize simultaneous spreading

### Monotonic deque

- [ ] Sliding-window maximum
- [ ] Sliding-window minimum
- [ ] Explain deque invariant
- [ ] Store indices
- [ ] Remove expired indices
- [ ] Remove dominated indices
- [ ] Explain amortized O(n)

### Advanced

- [ ] 0-1 BFS
- [ ] BFS with extra state
- [ ] Queue-based simulation
- [ ] Time-step simulation
- [ ] Event-order simulation

---

# 67. One-Page Revision Sheet

```text
QUEUE & DEQUE
│
├── Queue
│   ├── FIFO
│   ├── offer
│   ├── poll
│   └── peek
│
├── Circular Queue
│   ├── front
│   ├── rear
│   ├── size
│   └── (index + 1) % capacity
│
├── Deque
│   ├── add/remove first
│   ├── add/remove last
│   └── Stack + Queue behavior
│
├── BFS
│   ├── Graph BFS
│   ├── Grid BFS
│   ├── Level order
│   ├── Shortest path
│   └── Implicit graph
│
├── Multi-source BFS
│   ├── Multiple initial sources
│   ├── Nearest source
│   ├── Spread/infection
│   └── Time layers
│
├── Monotonic Deque
│   ├── Sliding max
│   ├── Sliding min
│   └── DP optimization
│
├── Advanced
│   ├── 0-1 BFS
│   └── State-space BFS
│
└── Simulation
    ├── Events
    ├── Scheduling
    ├── Time steps
    └── FIFO processing
```

## Core rules

```text
Queue:
FIFO

BFS:
Shortest path in unweighted graphs

Multi-source BFS:
Put all sources in queue at distance 0

Sliding maximum:
Deque values decrease

Sliding minimum:
Deque values increase

0-1 BFS:
weight 0 → front
weight 1 → back

Monotonic deque:
Each index enters/leaves a bounded number of times
→ O(n)
```

## Most important problems

```text
1. Binary Tree Level Order Traversal
2. Number of Islands
3. Rotting Oranges
4. 01 Matrix
5. Word Ladder
6. Shortest Path in Binary Matrix
7. Open the Lock
8. Sliding Window Maximum
9. Jump Game VI
10. Bus Routes
11. Design Circular Queue
12. 0-1 BFS problems
```
