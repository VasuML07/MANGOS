# 23. Advanced Problem-Solving Patterns

> **Target:** FAANG + top product companies | SDE + ML/AI Engineer | Java  
> **Difficulty:** Easy → Medium → Hard, with Medium dominating  
> **Purpose:** Recognize the underlying pattern quickly, choose the right invariant/data structure, derive complexity, and implement reliably.

---

## 1. Pattern Recognition First

Advanced patterns are reusable ways of thinking. The goal is not to memorize isolated solutions.

For every problem, ask:

1. What is the input structure?
2. What is being optimized or counted?
3. Is there a sorted/order property?
4. Is the answer about a contiguous range?
5. Can I maintain a window or prefix state?
6. Can I transform the problem into a monotonic condition?
7. Does the problem ask for top/bottom `K`?
8. Are multiple sorted sequences being merged?
9. Are intervals/events involved?
10. Is the problem naturally recursive?
11. Is there a state with repeated subproblems?
12. Is connectivity changing?
13. Can the state be represented by bits?
14. Can the search space be split into two halves?
15. Can large coordinates be compressed?

The strongest interview skill is **pattern recognition before coding**.

---

# 2. Two Pointers

## 2.1 Core Idea

Use two indices to process an array/string while avoiding repeated work.

Common forms:

- left/right pointers moving toward each other
- slow/fast pointers
- two pointers moving in the same direction
- one pointer for each of two sorted arrays

Typical signal:

- sorted array
- pair/triplet condition
- remove duplicates
- partitioning
- subsequence comparison
- longest/shortest valid structure with ordered movement

## 2.2 Opposite-Direction Two Pointers

For a sorted array:

```text
left = 0
right = n - 1

while left < right:
    evaluate a[left] + a[right]

    if sum is too small:
        left++
    else if sum is too large:
        right--
    else:
        found
```

Why it works:

- increasing `left` cannot decrease the sum
- decreasing `right` cannot increase the sum

The sorted order eliminates entire groups of candidates.

### Complexity

- Time: `O(n)`
- Space: `O(1)` excluding output

## 2.3 Example: Two Sum II

```java
class Solution {
    public int[] twoSum(int[] numbers, int target) {
        int left = 0, right = numbers.length - 1;

        while (left < right) {
            long sum = (long) numbers[left] + numbers[right];

            if (sum == target) {
                return new int[]{left + 1, right + 1};
            }

            if (sum < target) {
                left++;
            } else {
                right--;
            }
        }

        return new int[]{-1, -1};
    }
}
```

## 2.4 Three Sum Pattern

Sort first.

```text
for i:
    skip duplicate i
    left = i + 1
    right = n - 1

    while left < right:
        evaluate a[i] + a[left] + a[right]
```

Complexity:

- Sorting: `O(n log n)`
- Two-pointer scan for each `i`: `O(n²)`
- Total: `O(n²)`

## 2.5 Same-Direction Two Pointers

Useful when one pointer represents the write position.

Examples:

- remove duplicates
- move zeroes
- partition arrays
- merge filtered values

Example:

```java
public int removeDuplicates(int[] nums) {
    if (nums.length == 0) return 0;

    int write = 1;

    for (int read = 1; read < nums.length; read++) {
        if (nums[read] != nums[read - 1]) {
            nums[write++] = nums[read];
        }
    }

    return write;
}
```

---

# 3. Sliding Window

## 3.1 Core Idea

Maintain a contiguous interval `[left, right]`.

Instead of recomputing every subarray:

```text
add right
remove left when invalid
```

Typical signals:

- longest/shortest substring
- subarray with condition
- at most `K`
- exactly `K`
- frequency constraints
- distinct elements
- positive numbers with sum constraint

## 3.2 Fixed-Size Window

For window size `k`:

```java
long sum = 0;

for (int i = 0; i < nums.length; i++) {
    sum += nums[i];

    if (i >= k) {
        sum -= nums[i - k];
    }

    if (i >= k - 1) {
        // use current window
    }
}
```

Complexity: `O(n)`.

## 3.3 Variable-Size Window

Template:

```java
int left = 0;

for (int right = 0; right < nums.length; right++) {
    add(nums[right]);

    while (windowIsInvalid()) {
        remove(nums[left]);
        left++;
    }

    updateAnswer(left, right);
}
```

## 3.4 Frequency Map

Classic substring pattern:

```java
Map<Character, Integer> freq = new HashMap<>();
int left = 0;
int answer = 0;

for (int right = 0; right < s.length(); right++) {
    char c = s.charAt(right);
    freq.put(c, freq.getOrDefault(c, 0) + 1);

    while (/* invalid */) {
        char x = s.charAt(left++);
        freq.put(x, freq.get(x) - 1);

        if (freq.get(x) == 0) {
            freq.remove(x);
        }
    }

    answer = Math.max(answer, right - left + 1);
}
```

## 3.5 Critical Limitation

Ordinary sliding-window sum logic often relies on **non-negative values**.

For arbitrary negative values, shrinking the left side does not necessarily make a sum smaller.

For negative values, consider:

- prefix sums
- hash maps
- monotonic structures
- specialized transformations

---

# 4. Fast/Slow Pointers

## 4.1 Core Idea

Use two pointers moving at different speeds.

Typical uses:

- linked-list cycle detection
- finding cycle entry
- middle of linked list
- duplicate number
- repeated-state detection

## 4.2 Floyd Cycle Detection

```java
boolean hasCycle(ListNode head) {
    ListNode slow = head;
    ListNode fast = head;

    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;

        if (slow == fast) {
            return true;
        }
    }

    return false;
}
```

Complexity:

- Time: `O(n)`
- Space: `O(1)`

## 4.3 Find Cycle Entry

After `slow == fast`:

```text
p1 = head
p2 = meeting point

move both one step at a time

where they meet = cycle entry
```

## 4.4 Find Middle

```java
ListNode slow = head;
ListNode fast = head;

while (fast != null && fast.next != null) {
    slow = slow.next;
    fast = fast.next.next;
}

return slow;
```

---

# 5. Prefix Sum

## 5.1 Definition

For:

```text
a = [a0, a1, ..., an-1]
```

define:

```text
prefix[i + 1] = prefix[i] + a[i]
```

Then:

```text
sum(l, r) = prefix[r + 1] - prefix[l]
```

## 5.2 Why It Matters

It converts repeated range-sum queries from:

```text
O(length of range)
```

to:

```text
O(1)
```

after `O(n)` preprocessing.

## 5.3 Java

```java
long[] prefix = new long[nums.length + 1];

for (int i = 0; i < nums.length; i++) {
    prefix[i + 1] = prefix[i] + nums[i];
}

long rangeSum = prefix[r + 1] - prefix[l];
```

Use `long` when sums can exceed `int`.

## 5.4 Prefix Sum + HashMap

For subarray sum equal to `k`:

```java
Map<Long, Integer> count = new HashMap<>();
count.put(0L, 1);

long prefix = 0;
int answer = 0;

for (int x : nums) {
    prefix += x;

    answer += count.getOrDefault(prefix - k, 0);

    count.put(prefix, count.getOrDefault(prefix, 0) + 1);
}
```

Key equation:

```text
prefix[j] - prefix[i] = k
```

therefore:

```text
prefix[i] = prefix[j] - k
```

Complexity: `O(n)` expected time.

---

# 6. Difference Array

## 6.1 Core Idea

When there are many range updates:

```text
add value v to every position in [l, r]
```

do not update every element.

Instead:

```text
diff[l] += v
diff[r + 1] -= v
```

Then take a prefix sum.

## 6.2 Example

For:

```text
add 5 to [2, 6]
```

perform:

```text
diff[2] += 5
diff[7] -= 5
```

The effect appears from index `2` through `6`.

## 6.3 Java Template

```java
long[] diff = new long[n + 1];

void rangeAdd(long[] diff, int l, int r, long value) {
    diff[l] += value;
    diff[r + 1] -= value;
}

long[] build(long[] diff, int n) {
    long[] result = new long[n];
    long current = 0;

    for (int i = 0; i < n; i++) {
        current += diff[i];
        result[i] = current;
    }

    return result;
}
```

## 6.4 Prefix Sum vs Difference Array

| Task | Pattern |
|---|---|
| Many range queries, static array | Prefix sum |
| Many range additions, final array needed | Difference array |
| Many updates + many queries online | Fenwick/Segment Tree |

---

# 7. Binary Search

## 7.1 Core Idea

Binary search requires an ordered search space or a monotonic predicate.

Do not think only:

> "Search in a sorted array."

Think:

> "Can I eliminate half of the candidates after each test?"

## 7.2 Standard Template

```java
int left = 0;
int right = nums.length - 1;

while (left <= right) {
    int mid = left + (right - left) / 2;

    if (nums[mid] == target) {
        return mid;
    } else if (nums[mid] < target) {
        left = mid + 1;
    } else {
        right = mid - 1;
    }
}

return -1;
```

## 7.3 Lower Bound

First index with:

```text
a[i] >= target
```

```java
int lo = 0, hi = nums.length;

while (lo < hi) {
    int mid = lo + (hi - lo) / 2;

    if (nums[mid] >= target) {
        hi = mid;
    } else {
        lo = mid + 1;
    }
}

return lo;
```

## 7.4 Upper Bound

First index with:

```text
a[i] > target
```

```java
int lo = 0, hi = nums.length;

while (lo < hi) {
    int mid = lo + (hi - lo) / 2;

    if (nums[mid] > target) {
        hi = mid;
    } else {
        lo = mid + 1;
    }
}

return lo;
```

---

# 8. Binary Search on Answer

## 8.1 Core Idea

Sometimes the answer is not an array index.

It is a numerical value.

Suppose:

```text
x <= answer  => feasible
x > answer   => infeasible
```

Then feasibility is monotonic.

Binary search the answer.

## 8.2 Generic Structure

```java
long lo = minimumPossible;
long hi = maximumPossible;

while (lo < hi) {
    long mid = lo + (hi - lo) / 2;

    if (feasible(mid)) {
        hi = mid;
    } else {
        lo = mid + 1;
    }
}

return lo;
```

## 8.3 Typical Problems

- minimum capacity
- minimum eating speed
- minimum time
- maximum minimum distance
- allocate books
- split array
- aggressive cows
- shipping packages

## 8.4 Feasibility Function

The key is not binary search itself.

The key is designing:

```text
boolean feasible(answer)
```

with monotonic behavior.

---

# 9. Monotonic Stack

## 9.1 Core Idea

Maintain a stack whose values are monotonically increasing or decreasing.

Used for:

- next greater element
- next smaller element
- previous greater/smaller
- daily temperatures
- stock span
- largest rectangle in histogram
- contribution of each element
- removing digits

## 9.2 Next Greater Element

```java
int[] ans = new int[nums.length];
Arrays.fill(ans, -1);

Deque<Integer> stack = new ArrayDeque<>();

for (int i = nums.length - 1; i >= 0; i--) {
    while (!stack.isEmpty() && nums[stack.peek()] <= nums[i]) {
        stack.pop();
    }

    if (!stack.isEmpty()) {
        ans[i] = nums[stack.peek()];
    }

    stack.push(i);
}
```

## 9.3 Increasing vs Decreasing

Ask:

> What elements must be removed because they can never become the answer?

That determines the pop condition.

## 9.4 Amortized Complexity

Each element is:

- pushed at most once
- popped at most once

Therefore:

```text
O(n)
```

not `O(n²)`.

---

# 10. Monotonic Queue

## 10.1 Core Idea

Maintain candidates in monotonic order inside a deque.

Classic use:

> maximum/minimum of every sliding window of size `k`.

## 10.2 Sliding Window Maximum

```java
int[] maxSlidingWindow(int[] nums, int k) {
    int n = nums.length;
    int[] result = new int[n - k + 1];

    Deque<Integer> dq = new ArrayDeque<>();

    for (int i = 0; i < n; i++) {
        while (!dq.isEmpty() && dq.peekFirst() <= i - k) {
            dq.pollFirst();
        }

        while (!dq.isEmpty() && nums[dq.peekLast()] <= nums[i]) {
            dq.pollLast();
        }

        dq.offerLast(i);

        if (i >= k - 1) {
            result[i - k + 1] = nums[dq.peekFirst()];
        }
    }

    return result;
}
```

The deque front is always the best candidate.

Complexity: `O(n)` amortized.

---

# 11. Merge Intervals

## 11.1 Core Idea

Sort intervals by start time, then merge overlapping ranges.

```java
Arrays.sort(intervals, (a, b) -> Integer.compare(a[0], b[0]));

List<int[]> result = new ArrayList<>();

for (int[] current : intervals) {
    if (result.isEmpty() ||
        current[0] > result.get(result.size() - 1)[1]) {
        result.add(current.clone());
    } else {
        result.get(result.size() - 1)[1] =
            Math.max(result.get(result.size() - 1)[1], current[1]);
    }
}
```

Complexity:

- Sorting: `O(n log n)`
- Scan: `O(n)`
- Total: `O(n log n)`

## 11.2 Common Variants

- merge overlapping intervals
- insert interval
- interval intersections
- meeting rooms
- minimum arrows
- minimum removals
- maximum non-overlapping intervals

---

# 12. Sweep Line

## 12.1 Core Idea

Turn intervals into events.

For:

```text
[l, r]
```

create:

```text
start event at l
end event at r
```

Then sort events and maintain the active set/count.

## 12.2 Example: Maximum Overlap

```text
start: +1
end:   -1
```

Sort events and maintain:

```text
active += event.delta
answer = max(answer, active)
```

## 12.3 Endpoint Semantics

This is critical.

For half-open intervals:

```text
[l, r)
```

an event at `r` can occur before a start at the same coordinate.

For closed intervals:

```text
[l, r]
```

tie handling differs.

Always define whether endpoints overlap.

---

# 13. Top K

## 13.1 Core Idea

When only the best `K` elements are needed, do not fully sort everything unless necessary.

Use a heap.

For K largest:

- maintain a min-heap of size `K`

For K smallest:

- maintain a max-heap of size `K`

## 13.2 Java

```java
PriorityQueue<Integer> pq = new PriorityQueue<>();

for (int x : nums) {
    pq.offer(x);

    if (pq.size() > k) {
        pq.poll();
    }
}

int kthLargest = pq.peek();
```

Complexity:

```text
O(n log k)
```

Space:

```text
O(k)
```

## 13.3 Alternatives

- sorting: `O(n log n)`
- heap: `O(n log k)`
- Quickselect: expected `O(n)`

Use a heap when:

- streaming input
- online processing
- `k` is small
- simplicity matters

---

# 14. K-Way Merge

## 14.1 Core Idea

Merge `K` sorted sequences by keeping the smallest current element from each sequence.

Use a min-heap.

For `N` total elements:

```text
O(N log K)
```

## 14.2 Java Skeleton

```java
static class Node {
    int value;
    int row;
    int index;

    Node(int value, int row, int index) {
        this.value = value;
        this.row = row;
        this.index = index;
    }
}

PriorityQueue<Node> pq =
    new PriorityQueue<>(Comparator.comparingInt(a -> a.value));
```

Push the first element from each list.

Repeatedly:

1. remove minimum
2. output it
3. insert next element from the same list

## 14.3 Applications

- merge K sorted lists
- Kth smallest in sorted matrix
- smallest range covering K lists
- external sorting

---

# 15. Two Heaps

## 15.1 Core Idea

Maintain two halves of a dynamic ordered set:

```text
max-heap -> smaller half
min-heap -> larger half
```

Then the median is available at the heap tops.

## 15.2 Median Maintenance

```java
PriorityQueue<Integer> left =
    new PriorityQueue<>(Collections.reverseOrder());

PriorityQueue<Integer> right =
    new PriorityQueue<>();

void add(int x) {
    if (left.isEmpty() || x <= left.peek()) {
        left.offer(x);
    } else {
        right.offer(x);
    }

    if (left.size() > right.size() + 1) {
        right.offer(left.poll());
    }

    if (right.size() > left.size()) {
        left.offer(right.poll());
    }
}
```

Median:

```java
double median() {
    if (left.size() > right.size()) {
        return left.peek();
    }

    return ((long) left.peek() + right.peek()) / 2.0;
}
```

## 15.3 Invariant

```text
size(left) == size(right)
or
size(left) == size(right) + 1
```

and:

```text
max(left) <= min(right)
```

---

# 16. Backtracking

## 16.1 Core Idea

Explore a decision tree, undo the decision, then try another choice.

Template:

```java
void backtrack(State state) {
    if (isComplete(state)) {
        record(state);
        return;
    }

    for (Choice choice : choices(state)) {
        make(choice);
        backtrack(state);
        undo(choice);
    }
}
```

## 16.2 Typical Problems

- subsets
- permutations
- combinations
- combination sum
- N-Queens
- Sudoku
- word search
- palindrome partitioning

## 16.3 Pruning

Do not explore a branch if it cannot produce a valid solution.

Examples:

```text
sum > target
invalid placement
duplicate choice
violates constraint
remaining capacity insufficient
```

Pruning often determines practical performance.

---

# 17. Divide and Conquer

## 17.1 Core Idea

Break a problem into smaller independent subproblems.

Structure:

```text
divide
solve recursively
combine
```

General recurrence:

```text
T(n) = aT(n/b) + f(n)
```

## 17.2 Examples

- merge sort
- quicksort
- binary search
- closest pair of points
- inversion counting
- segment-tree construction

## 17.3 Merge Sort

```java
void mergeSort(int[] a, int l, int r) {
    if (l >= r) return;

    int mid = l + (r - l) / 2;

    mergeSort(a, l, mid);
    mergeSort(a, mid + 1, r);

    merge(a, l, mid, r);
}
```

Complexity:

```text
O(n log n)
```

Space for standard merge sort:

```text
O(n)
```

---

# 18. Greedy

## 18.1 Core Idea

Make the best locally justified choice while maintaining a globally optimal solution.

But greedy requires proof.

Common proof techniques:

### Exchange Argument

Show that an optimal solution can be transformed to contain the greedy choice without becoming worse.

### Staying Ahead

Show that after every step, the greedy solution is at least as good as any competing partial solution.

### Structural Proof

Show that an optimal solution must satisfy the greedy choice property.

## 18.2 Signals

- maximize number of non-overlapping intervals
- minimize resources
- local choice clearly dominates alternatives
- sorted order creates an exchange property
- scheduling/resource allocation

## 18.3 Warning

Do not use greedy merely because the problem "looks simple."

Counterexamples often exist.

---

# 19. Dynamic Programming

## 19.1 Core Idea

DP applies when:

1. subproblems overlap
2. an optimal/aggregate answer can be built from smaller states

Define:

```text
state
transition
base case
evaluation order
```

## 19.2 Recognition

Ask:

> If I make the first decision, does the remaining problem look like the same problem on a smaller state?

If yes, investigate DP.

## 19.3 Standard Template

```java
int[] dp = new int[n + 1];

dp[0] = base;

for (int i = 1; i <= n; i++) {
    dp[i] = transition(dp, i);
}

return dp[n];
```

## 19.4 Major DP Families

- 1D DP
- grid DP
- knapsack
- subsequence DP
- string DP
- interval DP
- tree DP
- state-machine DP
- bitmask DP
- digit DP

---

# 20. Graph Traversal

## 20.1 BFS

Use BFS for:

- shortest path in unweighted graphs
- minimum number of moves
- level order
- multi-source expansion

```java
Queue<Integer> q = new ArrayDeque<>();
boolean[] visited = new boolean[n];

q.offer(source);
visited[source] = true;

while (!q.isEmpty()) {
    int u = q.poll();

    for (int v : graph[u]) {
        if (!visited[v]) {
            visited[v] = true;
            q.offer(v);
        }
    }
}
```

Complexity:

```text
O(V + E)
```

## 20.2 DFS

Use DFS for:

- components
- reachability
- cycle detection
- path exploration
- topological ordering
- bridges/articulation points
- SCC algorithms

```java
void dfs(int u, List<Integer>[] graph, boolean[] visited) {
    visited[u] = true;

    for (int v : graph[u]) {
        if (!visited[v]) {
            dfs(v, graph, visited);
        }
    }
}
```

---

# 21. Topological Sort

## 21.1 Core Idea

Order vertices of a DAG so that:

```text
u -> v
```

means:

```text
u appears before v
```

Only directed acyclic graphs have a complete topological ordering.

## 21.2 Kahn's Algorithm

Maintain indegrees.

```java
int[] indegree = new int[n];

for (int u = 0; u < n; u++) {
    for (int v : graph[u]) {
        indegree[v]++;
    }
}

Queue<Integer> q = new ArrayDeque<>();

for (int i = 0; i < n; i++) {
    if (indegree[i] == 0) {
        q.offer(i);
    }
}

int count = 0;

while (!q.isEmpty()) {
    int u = q.poll();
    count++;

    for (int v : graph[u]) {
        if (--indegree[v] == 0) {
            q.offer(v);
        }
    }
}

boolean isDAG = count == n;
```

Complexity:

```text
O(V + E)
```

## 21.3 Common Applications

- Course Schedule
- dependency resolution
- build systems
- task ordering
- prerequisite graphs

---

# 22. Union-Find / DSU

## 22.1 Core Idea

Maintain dynamic connectivity between elements.

Operations:

```text
find(x)
union(a, b)
```

With:

- path compression
- union by size/rank

amortized complexity is approximately:

```text
O(alpha(n))
```

per operation.

## 22.2 Pattern Signals

- connected components
- redundant edge
- merging groups
- Kruskal
- dynamic connectivity
- "are these two items in the same group?"

## 22.3 Java

```java
class DSU {
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
        a = find(a);
        b = find(b);

        if (a == b) return false;

        if (size[a] < size[b]) {
            int t = a;
            a = b;
            b = t;
        }

        parent[b] = a;
        size[a] += size[b];

        return true;
    }
}
```

---

# 23. State Compression

## 23.1 Core Idea

Represent a large logical state using a compact encoding.

Example:

Instead of storing:

```text
visited = {A, C, F}
```

represent it as a bitmask:

```text
101001
```

This is especially useful when the number of relevant entities is small.

## 23.2 Common State Compression

- subset represented by mask
- visited cities
- selected features
- used resources
- boolean flags
- small categorical states

## 23.3 DP State Compression

Instead of:

```text
dp[position][visitedSet][otherState]
```

encode `visitedSet` as an integer.

This leads to:

```text
dp[position][mask]
```

for many bitmask DP problems.

---

# 24. Meet in the Middle

## 24.1 Core Idea

When brute force requires approximately:

```text
O(2^n)
```

and `n` is around 30–40, split the problem into two halves.

Each half has approximately:

```text
2^(n/2)
```

states.

This is dramatically smaller.

## 24.2 Subset Sum Pattern

Split:

```text
A = left half
B = right half
```

Generate all subset sums:

```text
L
R
```

Sort one list.

For each sum `x` in `L`, binary-search the best compatible value in `R`.

Typical complexity:

```text
O(2^(n/2) log 2^(n/2))
```

instead of:

```text
O(2^n)
```

## 24.3 Signals

- `n` too large for `2^n`
- `n` small enough that `2^(n/2)` is feasible
- subset/combinational optimization
- target sum / closest sum

---

# 25. Coordinate Compression

## 25.1 Core Idea

Map large sparse coordinates to dense ranks while preserving order.

Example:

```text
[1000000000, 10, 500000, 10]
```

unique sorted values:

```text
[10, 500000, 1000000000]
```

mapping:

```text
10          -> 0
500000      -> 1
1000000000  -> 2
```

## 25.2 Why

A data structure may need an array of size equal to the coordinate range.

If coordinates are huge but only `m` coordinates appear, use:

```text
O(m)
```

instead of:

```text
O(maxCoordinate)
```

## 25.3 Applications

- Fenwick Tree
- Segment Tree
- inversion counting
- interval problems
- sparse 2D grids
- ranking
- offline queries

## 25.4 Important Warning

Coordinate compression preserves **order**, not physical distance.

For:

```text
x = [10, 1000]
```

compressed coordinates may be:

```text
[0, 1]
```

The difference `1` does not mean the original distance was `1`.

If actual gaps matter, store coordinate values separately.

---

# 26. Line Sweep

Line sweep is closely related to sweep line, but is often used as a broader algorithmic strategy:

```text
sort important coordinates/events
move through them once
maintain active information
```

Common applications:

- interval overlap
- meeting rooms
- rectangle union
- skyline
- computational geometry
- event scheduling

Typical tools:

- sorting
- heap
- TreeMap
- difference map
- coordinate compression
- Fenwick Tree
- Segment Tree

---

# 27. Bitmasking

## 27.1 Core Idea

Represent Boolean choices with bits.

For `n` items:

```text
mask in [0, 2^n)
```

Bit `i` indicates whether item `i` is selected.

## 27.2 Basic Operations

```java
boolean set = (mask & (1 << i)) != 0;

int withBit = mask | (1 << i);

int withoutBit = mask & ~(1 << i);

int toggled = mask ^ (1 << i);
```

## 27.3 Enumerate Subsets

```java
for (int mask = 0; mask < (1 << n); mask++) {
    for (int i = 0; i < n; i++) {
        if ((mask & (1 << i)) != 0) {
            // i is selected
        }
    }
}
```

Complexity:

```text
O(n * 2^n)
```

## 27.4 Enumerate Submasks

For every mask:

```java
for (int sub = mask; sub > 0; sub = (sub - 1) & mask) {
    // sub is a non-empty submask of mask
}
```

This is a major advanced bitmask technique.

---

# 28. Pattern Combinations

Hard interview problems frequently combine multiple patterns.

## 28.1 Sliding Window + HashMap

Example signals:

```text
longest substring
at most K distinct
frequency constraints
```

## 28.2 Prefix Sum + HashMap

Signals:

```text
subarray sum
count of subarrays
negative values
```

## 28.3 Binary Search + Greedy

Signals:

```text
minimum possible maximum
maximum possible minimum
capacity/time/distance
```

Structure:

```text
binary search answer
    ↓
greedy feasibility check
```

## 28.4 Heap + Sweep Line

Useful for:

- meeting rooms
- active intervals
- resource allocation
- event scheduling

## 28.5 Monotonic Queue + DP

Useful when:

```text
dp[i] = best dp[j] over a sliding range
```

The deque maintains candidate states.

## 28.6 Coordinate Compression + Fenwick Tree

Useful when:

- values are huge
- only relative ordering matters
- dynamic prefix/rank queries are required

## 28.7 Bitmask + DP

Useful when:

- state is a subset
- `n` is small
- repeated subset states exist

Typical complexity:

```text
O(n * 2^n)
```

or:

```text
O(n² * 2^n)
```

## 28.8 Graph + DSU

Useful for:

- connectivity
- cycle detection in undirected graphs
- Kruskal
- merging components

---

# 29. Choosing the Pattern

| Problem signal | First pattern to consider |
|---|---|
| Sorted array + pair condition | Two pointers |
| Contiguous range + constraint | Sliding window |
| Linked-list cycle | Fast/slow pointers |
| Many static range sums | Prefix sum |
| Many range additions | Difference array |
| Sorted/monotonic search space | Binary search |
| Minimum/maximum numerical answer | Binary search on answer |
| Next greater/smaller | Monotonic stack |
| Window min/max | Monotonic queue |
| Overlapping intervals | Merge intervals |
| Events over coordinates | Sweep line |
| Best `K` elements | Heap / Top K |
| Merge sorted streams | K-way merge |
| Dynamic median | Two heaps |
| Enumerate valid choices | Backtracking |
| Recursive independent halves | Divide and conquer |
| Local choice with proof | Greedy |
| Repeated states + optimal substructure | DP |
| Reachability / unweighted shortest path | BFS/DFS |
| Dependencies | Topological sort |
| Dynamic connectivity | DSU |
| Small set of Boolean features | State compression |
| `n ≈ 30–40`, exponential search | Meet in the middle |
| Huge sparse coordinates | Coordinate compression |
| Subset states | Bitmasking |

---

# 30. Pattern Selection Decision Tree

```text
Is the input sorted?
├── Yes
│   ├── Pair/triplet relationship → Two pointers
│   └── Search target/threshold → Binary search
│
Is the problem about a contiguous range?
├── Yes
│   ├── Fixed size → Fixed sliding window
│   ├── Maintainable validity → Variable sliding window
│   └── Arbitrary negative sums → Prefix sum / hashmap / specialized method
│
Is the problem about intervals/events?
├── Yes
│   ├── Merge overlaps → Merge intervals
│   ├── Active count/resources → Sweep line + heap
│   └── Huge coordinates → Coordinate compression
│
Need next greater/smaller?
├── Yes → Monotonic stack
│
Need window maximum/minimum?
├── Yes → Monotonic queue
│
Need best K?
├── Yes → Heap / Quickselect
│
Need merge of sorted sources?
├── Yes → K-way merge
│
Need dynamic median?
├── Yes → Two heaps
│
Need all combinations?
├── Yes
│   ├── Small n → Backtracking / bitmasking
│   └── n around 30–40 → Meet in the middle
│
Are subproblems repeated?
├── Yes → Dynamic programming
│
Is there a local-choice proof?
├── Yes → Greedy
│
Is the problem a graph?
├── Yes
│   ├── Reachability → DFS/BFS
│   ├── Unweighted shortest path → BFS
│   ├── Dependencies → Topological sort
│   └── Connectivity merging → DSU
```

---

# 31. GATE Theory: High-Yield Concepts

## 31.1 Amortized Analysis

An operation can occasionally be expensive while the total sequence is cheap.

Examples:

- monotonic stack
- dynamic array resizing
- DSU operations

Do not judge complexity from the worst cost of one operation alone.

## 31.2 Binary Search Correctness

Binary search requires a monotonic predicate.

If:

```text
false false false true true true
```

then finding the first `true` is valid.

Similarly:

```text
true true true false false
```

supports finding the last `true`.

## 31.3 Prefix Sum

For:

```text
P[i] = a[0] + ... + a[i-1]
```

range sum is:

```text
P[r + 1] - P[l]
```

## 31.4 Difference Array

Difference arrays invert prefix accumulation.

If:

```text
D[i] = A[i] - A[i-1]
```

then:

```text
A[i] = D[0] + D[1] + ... + D[i]
```

## 31.5 Topological Ordering

A directed graph has a topological ordering iff it is acyclic.

For Kahn's algorithm:

```text
processed vertices < V
```

implies a cycle exists.

## 31.6 DSU

With path compression and union by rank/size:

```text
m operations on n elements:
O(m α(n))
```

where `α(n)` is the inverse Ackermann function.

## 31.7 Monotonic Stack

Each element is usually:

- inserted once
- removed once

giving:

```text
O(n)
```

amortized time.

## 31.8 Meet in the Middle

Splitting an exponential search of size `2^n` into two halves changes the number of generated states to roughly:

```text
2^(n/2) + 2^(n/2)
```

## 31.9 Coordinate Compression

Compression is order-preserving:

```text
x < y  => rank(x) < rank(y)
```

but does not preserve numeric distance.

## 31.10 Bitmask State Space

With `n` binary choices:

```text
2^n
```

possible masks exist.

Therefore bitmask algorithms are normally practical only for relatively small `n`.

---

# 32. Solved GATE-Style Questions

> These are **GATE-style practice questions**, not claims about exact official GATE wording/year.

## Question 1 — Sliding Window

Given:

```text
A = [2, 1, 3, 2, 4]
```

find the minimum length subarray whose sum is at least `7`.

### Solution

Use a variable sliding window because all values are positive.

Windows:

```text
[2,1,3,2] = 8
```

length `4`.

Shrink:

```text
[1,3,2] = 6
```

invalid.

Continue:

```text
[3,2,4] = 9
```

length `3`.

Shrink:

```text
[2,4] = 6
```

invalid.

Answer:

```text
3
```

### Key Pattern

Positive values + minimum contiguous length + sum threshold → sliding window.

---

## Question 2 — Prefix Sum

For:

```text
A = [3, -2, 4, 1, -3]
```

how many subarrays have sum `2`?

### Solution

Prefix sums:

```text
0, 3, 1, 5, 6, 3
```

For every current prefix `P`, look for:

```text
P - 2
```

At prefix `3`, need `1`; one previous occurrence.

At prefix `1`, need `-1`; none.

At prefix `5`, need `3`; one occurrence.

At prefix `6`, need `4`; none.

At final `3`, need `1`; one occurrence.

Total:

```text
3
```

### Key Pattern

Subarray sum with arbitrary positive/negative values → prefix sum + frequency map.

---

## Question 3 — Binary Search on Answer

Suppose a machine must process `100` units in at most `10` days. Processing speed is constant per day.

A feasibility function `feasible(x)` returns true when speed `x` is sufficient.

If:

```text
feasible(x) = false for x < 10
feasible(x) = true  for x >= 10
```

what is the minimum feasible speed?

### Solution

The predicate is:

```text
F F F F F F F F F T T T ...
```

Binary search for the first true.

Answer:

```text
10
```

### Key Pattern

Minimum numerical answer + monotonic feasibility → binary search on answer.

---

## Question 4 — Monotonic Stack

For:

```text
A = [2, 1, 5, 3, 4]
```

find the next greater element of each value.

### Solution

Results:

```text
2 -> 5
1 -> 5
5 -> -1
3 -> 4
4 -> -1
```

Therefore:

```text
[5, 5, -1, 4, -1]
```

### Key Pattern

Next greater/smaller → monotonic stack.

---

## Question 5 — Topological Sort

Consider edges:

```text
0 -> 1
0 -> 2
1 -> 3
2 -> 3
```

Give one valid topological ordering.

### Solution

Indegrees:

```text
0: 0
1: 1
2: 1
3: 2
```

Start with `0`.

Then:

```text
0
```

reduces indegrees of `1` and `2` to zero.

Choose:

```text
1
```

then:

```text
2
```

and finally:

```text
3
```

One valid answer:

```text
[0, 1, 2, 3]
```

Another valid answer:

```text
[0, 2, 1, 3]
```

### Key Point

Topological ordering is generally **not unique**.

---

## Question 6 — Meet in the Middle

A subset-sum problem has:

```text
n = 40
```

elements.

Compare brute force with meet in the middle.

### Solution

Brute force:

```text
2^40 ≈ 1.1 × 10^12
```

subset states.

Meet in the middle:

```text
2^20 + 2^20
```

which is approximately:

```text
2,097,152
```

generated states.

Thus the split approach is dramatically smaller.

### Key Pattern

When `n` is around `30–40`, inspect meet in the middle before attempting `2^n`.

---

# 33. LeetCode Roadmap

## Tier 1 — Core Recognition

Master first:

- Two Sum II
- Remove Duplicates from Sorted Array
- Move Zeroes
- Valid Palindrome
- Best Time to Buy and Sell Stock
- Maximum Average Subarray I
- Longest Substring Without Repeating Characters
- Linked List Cycle
- Middle of the Linked List
- Range Sum Query
- Merge Intervals
- Binary Search

## Tier 2 — Core Medium Patterns

Then:

- 3Sum
- Container With Most Water
- Longest Repeating Character Replacement
- Permutation in String
- Subarray Sum Equals K
- Minimum Size Subarray Sum
- Daily Temperatures
- Sliding Window Maximum
- Kth Largest Element in an Array
- Merge k Sorted Lists
- Find Median from Data Stream
- Course Schedule
- Number of Provinces
- Network Delay / shortest-path family
- Combination Sum
- Word Search
- House Robber
- Partition Equal Subset Sum

## Tier 3 — Advanced

Then:

- Trapping Rain Water
- Largest Rectangle in Histogram
- Minimum Window Substring
- Split Array Largest Sum
- Capacity To Ship Packages Within D Days
- Smallest Range Covering Elements from K Lists
- Skyline Problem
- N-Queens
- Word Break II
- Burst Balloons
- Shortest Path Visiting All Nodes
- Traveling Salesman / assignment-style bitmask DP
- closest subset-sum / meet-in-the-middle problems

---

# 34. Pattern-by-Pattern Difficulty Ladder

| Pattern | Easy | Medium | Hard |
|---|---|---|---|
| Two pointers | High | Very High | Medium |
| Sliding window | Medium | Very High | High |
| Fast/slow | High | High | Medium |
| Prefix sum | High | Very High | High |
| Difference array | Medium | High | Medium |
| Binary search | High | Very High | High |
| Binary search on answer | Low | Very High | Very High |
| Monotonic stack | Medium | Very High | High |
| Monotonic queue | Low | High | High |
| Merge intervals | High | Very High | Medium |
| Sweep line | Low | High | Very High |
| Top K | High | Very High | High |
| K-way merge | Medium | High | High |
| Two heaps | Low | High | High |
| Backtracking | High | Very High | Very High |
| Divide and conquer | High | High | High |
| Greedy | High | Very High | Very High |
| DP | Medium | Very High | Very High |
| Graph traversal | High | Very High | High |
| Topological sort | Medium | High | High |
| DSU | Medium | High | High |
| State compression | Low | High | Very High |
| Meet in the middle | Low | Medium | Very High |
| Coordinate compression | Low | High | Very High |
| Bitmasking | Medium | High | Very High |

---

# 35. Common Failure Modes

## 35.1 Using Sliding Window with Negative Numbers

Wrong assumption:

```text
shrink window => sum always decreases
```

This is not generally true with negative values.

## 35.2 Binary Search Without Monotonicity

If feasibility alternates:

```text
T F T F T
```

ordinary binary search is invalid.

## 35.3 Wrong Monotonic Stack Direction

The required answer determines whether smaller or larger elements should be removed.

## 35.4 Sorting When Order Must Be Preserved

Sorting can destroy the relationship between:

- original indices
- original sequence
- temporal order

## 35.5 Greedy Without Proof

A locally attractive decision is not automatically globally optimal.

## 35.6 Using `int` for Large Prefix Sums

Use:

```java
long
```

when accumulated values may exceed `2^31 - 1`.

## 35.7 Ignoring Interval Endpoint Semantics

Clarify:

```text
[l, r]
[l, r)
(l, r)
```

before implementing event ordering.

## 35.8 Overusing Bitmasking

`2^n` grows rapidly.

For large `n`, bitmask enumeration is usually impossible.

## 35.9 Coordinate Compression Used as Distance Compression

Ranks preserve order, not gaps.

## 35.10 Recursion Depth in Java

For very deep recursion, iterative DFS or explicit stacks may be safer.

---

# 36. Complexity Cheat Sheet

| Pattern | Typical Time | Typical Space |
|---|---:|---:|
| Two pointers | `O(n)` | `O(1)` |
| Sliding window | `O(n)` | `O(k)` / `O(alphabet)` |
| Fast/slow | `O(n)` | `O(1)` |
| Prefix sum preprocessing | `O(n)` | `O(n)` |
| Difference array | `O(n + q)` | `O(n)` |
| Binary search | `O(log n)` | `O(1)` |
| Binary search on answer | `O(log R × feasibility)` | depends |
| Monotonic stack | `O(n)` amortized | `O(n)` |
| Monotonic queue | `O(n)` amortized | `O(k)` |
| Merge intervals | `O(n log n)` | `O(n)` output |
| Sweep line | `O(n log n)` | `O(n)` |
| Top K heap | `O(n log k)` | `O(k)` |
| K-way merge | `O(N log k)` | `O(k)` |
| Two heaps | `O(log n)` per insertion | `O(n)` |
| Backtracking | exponential | recursion/state dependent |
| Divide and conquer | recurrence-dependent | recurrence-dependent |
| Greedy | often `O(n log n)` | usually low |
| DP | state × transitions | state-dependent |
| BFS/DFS | `O(V + E)` | `O(V)` |
| Topological sort | `O(V + E)` | `O(V)` |
| DSU | `O(alpha(n))` amortized | `O(n)` |
| Bitmask enumeration | `O(n2^n)` | `O(2^n)` if stored |
| Meet in middle | `O(2^(n/2))` states | `O(2^(n/2))` |
| Coordinate compression | `O(n log n)` | `O(n)` |

---

# 37. Interview Pattern Recognition Checklist

Before coding:

- [ ] Is the array sorted?
- [ ] Is the answer contiguous?
- [ ] Can I use two pointers?
- [ ] Can I maintain a sliding window?
- [ ] Are negative values present?
- [ ] Would prefix sums help?
- [ ] Are there many range updates?
- [ ] Is the answer numerical and monotonic?
- [ ] Do I need next greater/smaller?
- [ ] Do I need window min/max?
- [ ] Are only the best `K` items required?
- [ ] Are there multiple sorted lists?
- [ ] Is there a dynamic median?
- [ ] Are intervals/events involved?
- [ ] Can sorting expose a greedy structure?
- [ ] Is there a recursive decomposition?
- [ ] Are subproblems repeated?
- [ ] Is the problem a graph?
- [ ] Are there prerequisites/dependencies?
- [ ] Is connectivity changing?
- [ ] Is the state a subset of a small set?
- [ ] Is `n` around 30–40?
- [ ] Are coordinates huge but sparse?
- [ ] Can I combine two patterns?

---

# 38. Final Revision Sheet

## Two Pointers

```text
sorted + pair relationship
→ left/right
```

## Sliding Window

```text
contiguous + maintainable condition
→ expand right, shrink left
```

## Fast/Slow

```text
cycle / middle / repeated state
→ different pointer speeds
```

## Prefix Sum

```text
many range sums
→ prefix[r+1] - prefix[l]
```

## Difference Array

```text
many range additions
→ diff[l]++, diff[r+1]--
```

## Binary Search

```text
ordered/monotonic search space
→ halve candidates
```

## Binary Search on Answer

```text
numerical answer + monotonic feasibility
→ search minimum/maximum feasible value
```

## Monotonic Stack

```text
next/previous greater/smaller
→ maintain candidate stack
```

## Monotonic Queue

```text
sliding-window min/max
→ maintain best candidates in deque
```

## Merge Intervals

```text
overlapping ranges
→ sort by start, merge
```

## Sweep Line

```text
interval/event interactions
→ sorted events + active state
```

## Top K

```text
only K extremes needed
→ heap
```

## K-Way Merge

```text
K sorted streams
→ min-heap
```

## Two Heaps

```text
dynamic median
→ max-heap + min-heap
```

## Backtracking

```text
choices + constraints
→ choose → recurse → undo
```

## Divide and Conquer

```text
independent subproblems
→ divide → solve → combine
```

## Greedy

```text
local choice + proof
→ choose safest/best candidate
```

## DP

```text
overlapping subproblems + optimal substructure
→ state + transition
```

## Graph Traversal

```text
reachability / components / paths
→ BFS / DFS
```

## Topological Sort

```text
dependencies in DAG
→ indegree or DFS ordering
```

## DSU

```text
dynamic connectivity
→ find + union
```

## State Compression

```text
large logical state
→ compact encoding
```

## Meet in the Middle

```text
n ≈ 30–40 + exponential search
→ split into halves
```

## Coordinate Compression

```text
huge sparse coordinates
→ order-preserving ranks
```

## Bitmasking

```text
small number of binary choices
→ integer mask
```

---

# 39. Mastery Standard

You have mastered Advanced Patterns when you can:

1. Identify the likely pattern within a few minutes.
2. Explain **why** the pattern applies before coding.
3. State the invariant.
4. Select the correct data structure.
5. Derive the time and space complexity.
6. Recognize when a pattern's assumptions fail.
7. Combine two or more patterns in one problem.
8. Implement the pattern from memory in Java.
9. Explain correctness during an interview.
10. Solve unfamiliar problems by adapting the pattern rather than memorizing a solution.

The final goal is not:

```text
"I know 25 patterns."
```

It is:

```text
"I can look at a new problem,
identify its structural properties,
select a pattern,
prove that the pattern is valid,
and implement it correctly."
```

---

# 40. One-Page Interview Mental Model

```text
SORTED?
  └─ Pair/triplet → Two pointers
  └─ Search → Binary search

CONTIGUOUS RANGE?
  └─ Window condition → Sliding window
  └─ Arbitrary signed sum → Prefix sum / hashmap

LINKED LIST?
  └─ Cycle/middle → Fast/slow

RANGE OPERATIONS?
  └─ Static queries → Prefix sum
  └─ Offline range additions → Difference array
  └─ Online updates + queries → Fenwick / Segment Tree

MONOTONIC CONDITION?
  └─ Search answer → Binary search on answer

NEXT GREATER/SMALLER?
  └─ Monotonic stack

WINDOW MIN/MAX?
  └─ Monotonic queue

INTERVALS?
  └─ Merge
  └─ Active events → Sweep line

TOP K?
  └─ Heap / Quickselect

K SORTED SOURCES?
  └─ K-way merge

DYNAMIC MEDIAN?
  └─ Two heaps

CHOICES + CONSTRAINTS?
  └─ Backtracking

INDEPENDENT RECURSIVE HALVES?
  └─ Divide and conquer

LOCAL OPTIMUM?
  └─ Greedy, but prove it

REPEATED SUBPROBLEMS?
  └─ DP

GRAPH?
  └─ Reachability → BFS/DFS
  └─ Dependencies → Topological sort
  └─ Connectivity merging → DSU

SMALL SUBSET STATE?
  └─ State compression / bitmask DP

n ≈ 30–40?
  └─ Meet in the middle

HUGE SPARSE COORDINATES?
  └─ Coordinate compression

EVENTS ALONG AN AXIS?
  └─ Line/sweep line
```

---

# 41. Final Takeaway

Advanced patterns are the bridge between:

```text
Data structures
        +
Algorithms
        +
Problem recognition
```

For FAANG-level preparation, the important skill is **pattern composition**.

A hard problem may look like one problem but actually require:

```text
sorting
  ↓
two pointers
  ↓
heap
```

or:

```text
binary search on answer
  ↓
greedy feasibility
```

or:

```text
coordinate compression
  ↓
Fenwick Tree
```

or:

```text
bitmask
  ↓
DP
```

or:

```text
graph traversal
  ↓
topological ordering
  ↓
DP
```

or:

```text
meet in the middle
  ↓
sorting
  ↓
binary search
```

The target is therefore not memorization of individual solutions. Build a mental library of **structural signals, invariants, proof ideas, and reusable Java templates**.
