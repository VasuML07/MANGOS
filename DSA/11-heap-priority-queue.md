# 11 — Heap / Priority Queue

> **Goal:** Master heaps and priority queues as both data structures and problem-solving tools. Focus on heap invariants, Java's `PriorityQueue`, Top K patterns, K-way merging, two-heaps, median maintenance, scheduling, priority-based greedy algorithms, and heap + hashmap combinations.

---

# 1. What Is a Heap?

A heap is a complete binary tree that satisfies a heap-order property.

Two primary forms:

```text
Min Heap:
parent <= children

Max Heap:
parent >= children
```

A heap is usually represented using an array rather than explicit tree nodes.

Example min heap:

```text
        1
      /   \
     3     5
    / \   /
   7   9 8
```

Array:

```text
[1, 3, 5, 7, 9, 8]
```

The important property is not that the entire array is sorted.

Only the parent-child ordering is guaranteed.

---

# 2. Complete Binary Tree

A binary heap is a **complete binary tree**.

That means:

- Every level except possibly the last is completely filled.
- The last level is filled from left to right.

This allows efficient array representation.

For zero-based indexing:

```text
parent(i) = (i - 1) / 2

left(i)   = 2*i + 1

right(i)  = 2*i + 2
```

For a valid child index, check:

```java
child < heapSize
```

---

# 3. Min Heap

A min heap satisfies:

```text
parent <= children
```

Therefore:

```text
root = minimum element
```

Example:

```text
        2
      /   \
     5     4
    / \   / \
   9   7 8   6
```

The minimum is always at:

```text
index 0
```

### Important

A min heap does **not** mean:

```text
array is sorted
```

For example:

```text
[2, 5, 4, 9, 7, 8, 6]
```

is a valid min heap but is not sorted.

---

# 4. Max Heap

A max heap satisfies:

```text
parent >= children
```

Therefore:

```text
root = maximum element
```

Example:

```text
        10
       /  \
      8    9
     / \  / \
    4  7 5  6
```

Array:

```text
[10, 8, 9, 4, 7, 5, 6]
```

---

# 5. Heap Operations

The fundamental operations are:

```text
peek
insert
delete/extract
heapify
build heap
```

Typical complexities:

| Operation | Complexity |
|---|---:|
| Peek min/max | O(1) |
| Insert | O(log n) |
| Extract min/max | O(log n) |
| Heapify one node | O(log n) |
| Build heap | O(n) |
| Search arbitrary value | O(n) |

The last point is important:

> A heap is not a binary search tree.

Searching for an arbitrary value is generally O(n).

---

# 6. Java PriorityQueue

Java's standard priority queue is:

```java
PriorityQueue<Integer>
```

By default:

```text
Min Heap
```

Example:

```java
PriorityQueue<Integer> pq = new PriorityQueue<>();

pq.offer(5);
pq.offer(2);
pq.offer(8);
pq.offer(1);

System.out.println(pq.peek());
```

Output:

```text
1
```

---

# 6.1 Insert

Use:

```java
pq.offer(x);
```

or:

```java
pq.add(x);
```

For interview code, `offer` clearly communicates queue insertion.

Complexity:

```text
O(log n)
```

---

# 6.2 Peek

```java
pq.peek();
```

Returns the smallest element in a min heap.

Complexity:

```text
O(1)
```

If empty:

```text
peek() returns null
```

---

# 6.3 Delete / Extract

```java
pq.poll();
```

Removes and returns the minimum element.

Complexity:

```text
O(log n)
```

---

# 7. Max Heap in Java

Java's `PriorityQueue` is a min heap by default.

For a max heap:

```java
PriorityQueue<Integer> maxHeap =
    new PriorityQueue<>(Comparator.reverseOrder());
```

Example:

```java
maxHeap.offer(5);
maxHeap.offer(2);
maxHeap.offer(8);

System.out.println(maxHeap.poll());
```

Output:

```text
8
```

---

# 8. Custom Comparator

For objects:

```java
PriorityQueue<int[]> pq =
    new PriorityQueue<>(
        (a, b) -> Integer.compare(a[0], b[0])
    );
```

This creates a min heap based on:

```text
a[0]
```

For descending priority:

```java
PriorityQueue<int[]> pq =
    new PriorityQueue<>(
        (a, b) -> Integer.compare(b[0], a[0])
    );
```

Avoid:

```java
(a, b) -> a[0] - b[0]
```

because integer subtraction can overflow.

---

# 9. Heapify

Heapify restores the heap property for a node when its children already satisfy the heap property.

For a max heap:

```text
root
 ↓
compare with children
 ↓
swap with larger child if necessary
 ↓
continue downward
```

This process is called:

```text
sift down
```

or:

```text
down-heap
```

---

# 9.1 Max Heapify

Suppose:

```text
        3
       / \
      8   5
```

For a max heap, `3` violates the property.

Largest child:

```text
8
```

Swap:

```text
        8
       / \
      3   5
```

Now the affected subtree must be checked again.

---

# 9.2 Java Heapify

```java
static void maxHeapify(int[] a, int heapSize, int i) {
    while (true) {
        int largest = i;

        int left = 2 * i + 1;
        int right = 2 * i + 2;

        if (left < heapSize && a[left] > a[largest]) {
            largest = left;
        }

        if (right < heapSize && a[right] > a[largest]) {
            largest = right;
        }

        if (largest == i) {
            break;
        }

        int temp = a[i];
        a[i] = a[largest];
        a[largest] = temp;

        i = largest;
    }
}
```

Complexity:

```text
O(log n)
```

because the node can move down at most the height of the heap.

---

# 10. Min Heapify

For a min heap, choose the smallest child.

```java
static void minHeapify(int[] a, int heapSize, int i) {
    while (true) {
        int smallest = i;

        int left = 2 * i + 1;
        int right = 2 * i + 2;

        if (left < heapSize && a[left] < a[smallest]) {
            smallest = left;
        }

        if (right < heapSize && a[right] < a[smallest]) {
            smallest = right;
        }

        if (smallest == i) {
            break;
        }

        int temp = a[i];
        a[i] = a[smallest];
        a[smallest] = temp;

        i = smallest;
    }
}
```

---

# 11. Insert Into a Heap

Insertion occurs at the next available position to preserve completeness.

Then move the element upward until the heap property is restored.

This is:

```text
sift up
```

or:

```text
bubble up
```

Example min heap:

```text
[2, 5, 4, 8, 7]
```

Insert:

```text
1
```

Initially:

```text
[2, 5, 4, 8, 7, 1]
```

Compare with parent:

```text
1 < 4
```

Swap:

```text
[2, 5, 1, 8, 7, 4]
```

Compare again:

```text
1 < 2
```

Swap:

```text
[1, 5, 2, 8, 7, 4]
```

Heap restored.

Complexity:

```text
O(log n)
```

---

# 12. Delete Root From a Heap

To delete the root:

1. Replace root with last element.
2. Reduce heap size.
3. Sift down.

Example:

```text
        1
      /   \
     3     5
    / \   /
   7   9 8
```

Remove `1`.

Move last element `8` to root:

```text
        8
      /   \
     3     5
    / \
   7   9
```

Sift down:

```text
8 > 3
8 > 5
```

Choose smaller child `3`.

Final:

```text
        3
      /   \
     8     5
    / \
   7   9
```

Continue if necessary depending on the heap.

Complexity:

```text
O(log n)
```

---

# 13. Build Heap

Given an arbitrary array, build a heap using bottom-up heapification.

For an array of size `n`, the last non-leaf node is:

```text
n / 2 - 1
```

Therefore:

```java
for (int i = n / 2 - 1; i >= 0; i--) {
    heapify(a, n, i);
}
```

---

# 13.1 Why Build Heap Is O(n)

A common mistake is:

```text
n nodes × O(log n)
= O(n log n)
```

But most nodes are leaves or close to leaves.

Only a small number of nodes can travel large distances.

The total work is:

```text
O(n)
```

This is a standard heap interview and GATE concept.

---

# 14. Top K Pattern

Top K problems are among the highest-value heap patterns.

Typical questions:

- K largest elements.
- K smallest elements.
- K most frequent elements.
- K closest points.
- K closest numbers.
- K highest scores.

The central idea is:

> Maintain only the K elements that can still belong to the answer.

---

# 15. K Largest Elements

Suppose:

```text
nums = [3,2,1,5,6,4]
k = 2
```

Answer:

```text
[5,6]
```

Use a **min heap of size K**.

Why min heap?

The smallest element among the current top K is at the root.

When a larger candidate arrives:

```text
remove smallest
insert candidate
```

At the end, the heap contains the K largest elements.

---

# 15.1 Java

```java
static int[] kLargest(int[] nums, int k) {
    PriorityQueue<Integer> minHeap = new PriorityQueue<>();

    for (int x : nums) {
        minHeap.offer(x);

        if (minHeap.size() > k) {
            minHeap.poll();
        }
    }

    int[] result = new int[minHeap.size()];

    int i = 0;

    while (!minHeap.isEmpty()) {
        result[i++] = minHeap.poll();
    }

    return result;
}
```

Complexity:

```text
O(n log k)
```

Space:

```text
O(k)
```

---

# 16. K Smallest Elements

Reverse the structure.

Use a:

```text
max heap of size K
```

Why?

The largest element among the current K smallest elements is at the root.

When a smaller candidate arrives:

```text
remove largest
insert candidate
```

Complexity:

```text
O(n log k)
```

---

# 17. Top K Rule to Memorize

```text
K largest
→ min heap of size K

K smallest
→ max heap of size K
```

The heap root represents the element that is easiest to discard from the current candidate set.

This is more useful than memorizing the rule mechanically.

---

# 18. K-th Largest Element

To find the K-th largest:

```text
min heap of size K
```

Process every element.

At the end:

```text
heap.peek()
```

is the K-th largest.

Example:

```text
[3,2,1,5,6,4]
k = 2
```

Maintain:

```text
[2,3]
[3,5]
[5,6]
```

Final root:

```text
5
```

Answer:

```text
5
```

Complexity:

```text
O(n log k)
```

Space:

```text
O(k)
```

---

# 19. K-th Smallest Element

Use:

```text
max heap of size K
```

At the end:

```text
heap.peek()
```

is the K-th smallest.

Complexity:

```text
O(n log k)
```

---

# 20. Heap vs Quickselect for K-th Element

For K-th largest:

### Heap

```text
O(n log k)
```

Advantages:

- Simple.
- Predictable.
- Easy to extend to streaming input.
- Works naturally for Top K.

### Quickselect

Average:

```text
O(n)
```

Worst:

```text
O(n²)
```

Advantages:

- Better average asymptotic complexity for a single order statistic.
- In-place implementations are possible.

### Interview decision

If:

```text
k << n
```

heap is often attractive.

If:

```text
only one K-th element
```

and in-place/average linear performance matters:

```text
Quickselect
```

may be preferable.

---

# 21. K-Way Merge

Suppose there are K sorted arrays:

```text
A = [1,4,7]
B = [2,5,8]
C = [3,6,9]
```

Need one sorted result:

```text
[1,2,3,4,5,6,7,8,9]
```

Use a min heap containing the current smallest element from each list.

Initially:

```text
1
2
3
```

Pop `1`.

Insert the next element from A:

```text
2
3
4
```

Continue.

---

# 21.1 Complexity

Let:

```text
N = total number of elements
K = number of sorted lists
```

Each element is inserted/popped through a heap of size at most K.

Therefore:

```text
O(N log K)
```

Space:

```text
O(K)
```

excluding output.

This is much better than repeatedly scanning all K lists:

```text
O(NK)
```

---

# 22. K-Way Merge Java

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

static List<Integer> mergeKSortedArrays(
        int[][] arrays) {

    PriorityQueue<Node> pq =
        new PriorityQueue<>(
            Comparator.comparingInt(node -> node.value)
        );

    for (int row = 0; row < arrays.length; row++) {
        if (arrays[row].length > 0) {
            pq.offer(
                new Node(arrays[row][0], row, 0)
            );
        }
    }

    List<Integer> result = new ArrayList<>();

    while (!pq.isEmpty()) {
        Node node = pq.poll();

        result.add(node.value);

        int nextIndex = node.index + 1;

        if (nextIndex < arrays[node.row].length) {
            pq.offer(
                new Node(
                    arrays[node.row][nextIndex],
                    node.row,
                    nextIndex
                )
            );
        }
    }

    return result;
}
```

---

# 23. K-Way Merge Pattern

Whenever you see:

```text
multiple sorted sources
```

think:

```text
min heap
```

Examples:

- Merge K sorted arrays.
- Merge K sorted linked lists.
- Smallest range covering K lists.
- Multi-source sorted streams.
- External sorting.
- Multiple sorted iterators.

---

# 24. Two Heaps

Two heaps are useful when the data needs to be divided around a boundary.

Most important example:

```text
Median maintenance
```

Use:

```text
max heap → lower half
min heap → upper half
```

Example:

```text
Numbers:
[1, 5, 2, 10, 3]
```

Conceptually:

```text
lower half:
max heap

upper half:
min heap
```

---

# 25. Median Maintenance

For a sorted set:

```text
[1,2,3,4,5]
```

median:

```text
3
```

For:

```text
[1,2,3,4]
```

median:

```text
(2 + 3) / 2
```

The challenge is maintaining the median while numbers arrive dynamically.

---

# 26. Two-Heap Invariant

Maintain:

```text
maxHeap = lower half
minHeap = upper half
```

Required invariants:

```text
size(maxHeap) == size(minHeap)
```

or:

```text
size(maxHeap) == size(minHeap) + 1
```

And:

```text
maxHeap.peek() <= minHeap.peek()
```

Then:

### Odd number of elements

Median:

```text
maxHeap.peek()
```

### Even number

Median:

```text
(maxHeap.peek() + minHeap.peek()) / 2
```

Use `long` before addition when integer overflow is possible.

---

# 27. Median Finder Java

```java
class MedianFinder {
    private final PriorityQueue<Integer> lower =
        new PriorityQueue<>(Comparator.reverseOrder());

    private final PriorityQueue<Integer> upper =
        new PriorityQueue<>();

    public void addNum(int num) {
        if (lower.isEmpty() || num <= lower.peek()) {
            lower.offer(num);
        } else {
            upper.offer(num);
        }

        rebalance();
    }

    private void rebalance() {
        if (lower.size() > upper.size() + 1) {
            upper.offer(lower.poll());
        } else if (upper.size() > lower.size()) {
            lower.offer(upper.poll());
        }
    }

    public double findMedian() {
        if (lower.size() > upper.size()) {
            return lower.peek();
        }

        long left = lower.peek();
        long right = upper.peek();

        return (left + right) / 2.0;
    }
}
```

Complexity:

```text
addNum: O(log n)
findMedian: O(1)
```

Space:

```text
O(n)
```

---

# 28. Why Two Heaps Work

The heaps partition the data:

```text
lower values
      ↓
[ max heap ]

boundary

[ min heap ]
      ↑
upper values
```

The median is exactly at the boundary between the two halves.

Instead of sorting all values after every insertion:

```text
O(n log n) repeatedly
```

we maintain only the boundary.

This is a general design principle:

> If a query depends on the boundary between two groups, consider two balanced heaps.

---

# 29. Scheduling With Priority Queue

Heaps are natural for scheduling because the next task is often determined by:

```text
earliest deadline
smallest duration
highest priority
earliest finishing time
```

A priority queue lets us repeatedly extract the most important available task.

---

# 30. Task Scheduling Pattern

Suppose tasks become available over time.

Each task:

```text
arrival time
processing time
priority
```

Typical approach:

1. Sort tasks by arrival time.
2. Add all currently available tasks to a priority queue.
3. Extract the highest-priority task.
4. Process it.
5. Advance time.
6. Add newly available tasks.

This pattern appears in:

- CPU scheduling.
- Single-threaded CPU.
- Task execution.
- Job scheduling.
- Event simulation.

---

# 31. Generic Scheduling Template

```java
Arrays.sort(
    tasks,
    (a, b) -> Integer.compare(a[0], b[0])
);

PriorityQueue<int[]> pq =
    new PriorityQueue<>(
        (a, b) -> Integer.compare(a[1], b[1])
    );

long time = 0;
int i = 0;

while (i < tasks.length || !pq.isEmpty()) {

    if (pq.isEmpty()) {
        time = Math.max(time, tasks[i][0]);
    }

    while (i < tasks.length &&
           tasks[i][0] <= time) {
        pq.offer(tasks[i]);
        i++;
    }

    int[] task = pq.poll();

    time += task[2];

    // Process task.
}
```

The exact comparator and fields depend on the scheduling objective.

---

# 32. Priority-Based Greedy

A common pattern:

```text
At every step:
choose the currently available item with the best priority.
```

A heap makes this efficient.

Without a heap:

```text
scan all candidates
O(n) per selection
```

With a heap:

```text
insert candidate → O(log n)
extract best → O(log n)
```

This can reduce an O(n²)-style repeated-selection process to:

```text
O(n log n)
```

---

# 33. Priority-Based Greedy Examples

Common examples include:

- CPU task scheduling.
- Assigning jobs.
- Connecting ropes with minimum cost.
- Huffman coding.
- IPO-style capital selection.
- Maximum events.
- Task assignment.
- Meeting/resource allocation.
- Repeatedly selecting the smallest/largest item.
- Reorganizing strings.

The recurring pattern is:

```text
Candidates
   ↓
PriorityQueue
   ↓
best available choice
   ↓
update state
   ↓
add new candidates
```

---

# 34. Connect Ropes / Minimum Cost

Given rope lengths:

```text
[4,3,2,6]
```

Connect two ropes at a time.

Cost of joining:

```text
sum of their lengths
```

Goal:

```text
minimum total cost
```

Greedy strategy:

> Always combine the two smallest ropes.

Use a min heap.

Process:

```text
2 + 3 = 5
heap → [4,5,6]

4 + 5 = 9
heap → [6,9]

6 + 9 = 15
```

Total:

```text
5 + 9 + 15 = 29
```

---

# 35. Java

```java
static long minCostToConnectRopes(int[] ropes) {
    PriorityQueue<Integer> minHeap =
        new PriorityQueue<>();

    for (int x : ropes) {
        minHeap.offer(x);
    }

    long cost = 0;

    while (minHeap.size() > 1) {
        int a = minHeap.poll();
        int b = minHeap.poll();

        int sum = a + b;

        cost += sum;
        minHeap.offer(sum);
    }

    return cost;
}
```

Complexity:

```text
O(n log n)
```

Use `long` for total cost when values can be large.

---

# 36. Heap + HashMap

Sometimes a heap alone cannot support all required operations.

Example:

> Maintain frequencies and repeatedly access the most frequent item.

Use:

```text
HashMap
+
Heap
```

The hashmap stores:

```text
key → frequency
```

The heap stores candidates ordered by frequency.

---

# 37. Top K Frequent Elements

Example:

```text
[1,1,1,2,2,3]
k = 2
```

Frequencies:

```text
1 → 3
2 → 2
3 → 1
```

Answer:

```text
[1,2]
```

Approach:

1. Count frequencies with hashmap.
2. Maintain min heap of size K.
3. Heap priority = frequency.

---

# 37.1 Java

```java
static int[] topKFrequent(int[] nums, int k) {
    Map<Integer, Integer> freq = new HashMap<>();

    for (int x : nums) {
        freq.merge(x, 1, Integer::sum);
    }

    PriorityQueue<int[]> minHeap =
        new PriorityQueue<>(
            (a, b) -> Integer.compare(a[1], b[1])
        );

    for (Map.Entry<Integer, Integer> entry : freq.entrySet()) {
        minHeap.offer(
            new int[]{entry.getKey(), entry.getValue()}
        );

        if (minHeap.size() > k) {
            minHeap.poll();
        }
    }

    int[] result = new int[k];

    for (int i = k - 1; i >= 0; i--) {
        result[i] = minHeap.poll()[0];
    }

    return result;
}
```

Complexity:

```text
Frequency counting: O(n)
Heap: O(m log k)

Total: O(n + m log k)
```

where:

```text
m = number of distinct values
```

---

# 38. Heap + HashMap: Why Both?

The hashmap provides:

```text
fast lookup/update
```

The heap provides:

```text
fast access to best candidate
```

This combination is useful when the problem has two requirements:

```text
1. Update or retrieve state by key.
2. Repeatedly select the best key.
```

A single data structure may not support both efficiently.

---

# 39. Lazy Deletion

Java's `PriorityQueue` does not efficiently support arbitrary deletion by key.

A common technique is:

```text
lazy deletion
```

Instead of immediately removing stale heap entries:

1. Insert updated entries.
2. Keep the old entry.
3. When it reaches the top, check whether it is still valid.
4. Remove it if stale.

This is common in:

- Sliding window problems.
- Graph algorithms.
- Scheduling.
- Dijkstra variants.
- Dynamic priorities.

---

# 40. Heap + HashMap Lazy Deletion Example

Suppose:

```text
frequency changes
```

but the heap still contains an old frequency.

Heap entry:

```text
(value, oldFrequency)
```

Hashmap:

```text
value → currentFrequency
```

When the entry reaches the root:

```java
if (entry.frequency != freq.get(entry.value)) {
    pq.poll();
}
```

Then continue.

This avoids expensive arbitrary heap deletion.

---

# 41. Sliding Window Maximum and Heaps

Given:

```text
[1,3,-1,-3,5,3,6,7]
```

window size:

```text
3
```

Maximums:

```text
[3,3,5,5,6,7]
```

A max heap can track:

```text
(value, index)
```

When the largest element is outside the current window:

```text
discard it
```

This is another example of lazy deletion.

However, a monotonic deque achieves:

```text
O(n)
```

whereas the heap approach is typically:

```text
O(n log n)
```

or O(n log k) depending on implementation/analysis.

Therefore:

> Choose the data structure based on the required complexity and operation set.

---

# 42. Dijkstra and Priority Queue

A priority queue is central to Dijkstra's shortest-path algorithm.

Store:

```text
(distance, node)
```

in a min heap.

Repeatedly:

```text
extract node with smallest tentative distance
```

Then relax outgoing edges.

Typical complexity with adjacency lists and a binary heap:

```text
O((V + E) log V)
```

depending on implementation.

The key heap pattern is:

```text
best-known candidate
→ extract minimum
→ generate improved candidates
→ push them
```

---

# 43. Priority Queue in Graph Algorithms

Heaps commonly appear in:

- Dijkstra.
- Prim's MST.
- A* search.
- Best-first search.
- K-way graph/state exploration.
- Multi-source shortest paths.

Common entry:

```java
PriorityQueue<int[]> pq =
    new PriorityQueue<>(
        Comparator.comparingInt(a -> a[0])
    );
```

where:

```text
a[0] = priority
a[1] = node/state ID
```

---

# 44. Top K Closest Points

Given points:

```text
(x, y)
```

distance squared:

```text
x² + y²
```

For K closest points, maintain:

```text
max heap of size K
```

Why max heap?

The farthest point among the current K closest is the easiest candidate to remove.

Avoid square root:

```text
sqrt(x² + y²)
```

because comparing:

```text
x² + y²
```

gives the same ordering for non-negative distances.

Use `long` if coordinate multiplication can overflow `int`.

---

# 45. K Closest Pattern

General rule:

```text
Want K smallest by score
→ max heap size K

Want K largest by score
→ min heap size K
```

The score can be:

- Distance.
- Frequency.
- Cost.
- Time.
- Value.
- Priority.

---

# 46. K-Way Merge and Linked Lists

For K sorted linked lists:

```text
list1
list2
...
listK
```

put each list's head into a min heap.

Heap entry:

```text
ListNode
```

Comparator:

```java
PriorityQueue<ListNode> pq =
    new PriorityQueue<>(
        Comparator.comparingInt(node -> node.val)
    );
```

After popping a node:

```text
push node.next
```

Complexity:

```text
O(N log K)
```

where N is total number of nodes.

This is one of the most important applications of K-way merging.

---

# 47. Scheduling: Earliest Deadline

Suppose jobs have deadlines and available processing choices.

A min heap can store jobs by:

```text
deadline
```

Then:

```text
earliest deadline first
```

can be implemented efficiently.

But do not confuse:

```text
heap implementation
```

with:

```text
proof of greedy correctness
```

The heap only provides efficient selection. The scheduling rule still needs to be justified by the objective.

---

# 48. Scheduling: Available Tasks

A frequent interview pattern:

```text
tasks sorted by arrival time
+
priority queue ordered by priority
```

At time `t`:

```text
add all tasks with arrival <= t
choose best available task
```

This separates two orderings:

```text
arrival ordering
```

and:

```text
execution priority
```

Sorting handles the first.

The heap handles the second.

This is a very common combined pattern.

---

# 49. Priority Queue vs Sorting

Suppose all tasks are known in advance.

If you only need:

```text
sort once
```

sorting may be simpler.

If new candidates continuously become available and you repeatedly need:

```text
best current candidate
```

a heap is usually better.

### Think:

```text
Static ordering
→ sorting

Dynamic repeated best-choice extraction
→ priority queue
```

---

# 50. Heap vs TreeMap

A `PriorityQueue` gives:

```text
best element
insert
extract best
```

efficiently.

A `TreeMap` can additionally support:

```text
ordered keys
frequency counts
ceiling/floor
delete specific key
```

If you need arbitrary deletion or exact key counts, `TreeMap`/`TreeSet` may be more appropriate.

---

# 51. Heap Does Not Support Fast Arbitrary Search

Given:

```text
min heap
```

finding:

```text
value = 50
```

is not:

```text
O(log n)
```

It can be:

```text
O(n)
```

because heap ordering only gives information about parent-child relationships.

This is a common interview trap.

---

# 52. Heap Sort Connection

Heap Sort uses a heap to repeatedly extract the maximum/minimum.

For ascending order:

```text
build max heap
→ move max to end
→ reduce heap
→ heapify
```

Complexity:

```text
O(n log n)
```

and:

```text
O(1)
```

auxiliary space in the standard array implementation.

Heap data structures therefore connect directly to the sorting topic.

---

# 53. Common Heap Bugs

## Bug 1: Wrong child indices

For zero-based arrays:

```text
left = 2*i + 1
right = 2*i + 2
```

---

## Bug 2: Wrong heap type

Remember:

```text
K largest → min heap
K smallest → max heap
```

for fixed-size Top K.

---

## Bug 3: Forgetting to limit heap size

If solving Top K:

```java
if (pq.size() > k) {
    pq.poll();
}
```

Without this, memory can grow to O(n).

---

## Bug 4: Wrong comparator direction

For:

```text
max heap
```

in Java:

```java
Comparator.reverseOrder()
```

For custom values, reverse the comparison carefully.

---

## Bug 5: Comparator overflow

Bad:

```java
(a, b) -> a[0] - b[0]
```

Use:

```java
(a, b) -> Integer.compare(a[0], b[0])
```

---

## Bug 6: Forgetting heap invariants

For two heaps:

```text
lower.max <= upper.min
```

and their sizes must differ by at most one.

---

## Bug 7: Stale heap entries

With lazy deletion, always verify whether the popped entry is still valid.

---

# 54. Complexity Cheat Sheet

| Operation / Pattern | Time | Space |
|---|---:|---:|
| Peek | O(1) | O(1) |
| Insert | O(log n) | O(1) extra |
| Extract | O(log n) | O(1) extra |
| Heapify | O(log n) | O(1) iterative |
| Build heap | O(n) | O(1) extra |
| Top K | O(n log k) | O(k) |
| K-th largest | O(n log k) | O(k) |
| K-th smallest | O(n log k) | O(k) |
| K-way merge | O(N log K) | O(K) |
| Median insertion | O(log n) | O(n) |
| Median query | O(1) | O(n) |
| Scheduling with heap | Usually O(n log n) | O(n) |
| Heap + hashmap Top K | O(n + m log k) | O(m + k) |

---

# 55. Pattern Recognition

## Pattern 1 — Top K

Keywords:

```text
top k
k largest
k smallest
k most frequent
k closest
```

Think:

```text
fixed-size heap
```

---

## Pattern 2 — K-way Merge

Keywords:

```text
multiple sorted arrays
multiple sorted lists
k sorted streams
```

Think:

```text
min heap
```

---

## Pattern 3 — Median

Keywords:

```text
median
running median
streaming median
```

Think:

```text
max heap + min heap
```

---

## Pattern 4 — Repeated Best Choice

Keywords:

```text
always choose smallest
always choose largest
highest priority available
next best candidate
```

Think:

```text
priority queue
```

---

## Pattern 5 — Scheduling

Keywords:

```text
tasks arrive
available jobs
CPU scheduling
minimum rooms
earliest deadline
```

Think:

```text
sort by arrival
+
priority queue
```

---

## Pattern 6 — Dynamic Best Candidate

Keywords:

```text
new candidates appear
maintain current best
repeatedly extract best
```

Think:

```text
heap
```

---

## Pattern 7 — State + Priority

Keywords:

```text
frequency + best
distance + node
value + index
deadline + task
```

Think:

```text
hashmap / array for state
+
heap for priority
```

---

# 56. Advanced: Max Heap for Top K Smallest

Suppose:

```text
[7,1,5,3,9,2]
k = 3
```

Maintain max heap:

```text
[7]
[7,1]
[7,1,5]
```

Next:

```text
3
```

Since:

```text
3 < 7
```

remove `7`:

```text
[5,1,3]
```

Next:

```text
9
```

Ignore.

Next:

```text
2
```

remove `5`:

```text
[3,1,2]
```

Final:

```text
1,2,3
```

---

# 57. Advanced: Top K With Ties

If frequency ties need deterministic ordering, include a secondary comparator.

Example:

```text
higher frequency first
smaller value first
```

For a min heap retaining the best K, the "worst" candidate should be at the root.

Example comparator:

```java
PriorityQueue<int[]> pq =
    new PriorityQueue<>((a, b) -> {
        if (a[1] != b[1]) {
            return Integer.compare(a[1], b[1]);
        }

        return Integer.compare(b[0], a[0]);
    });
```

The exact direction depends on which candidate you want removed first.

Always define:

> Which element should be considered the worst among the retained K?

Then put that element at the heap root.

---

# 58. Advanced: Heap With Objects

Example:

```java
class Task {
    int id;
    int priority;
    int duration;

    Task(int id, int priority, int duration) {
        this.id = id;
        this.priority = priority;
        this.duration = duration;
    }
}
```

Priority queue:

```java
PriorityQueue<Task> pq =
    new PriorityQueue<>(
        Comparator
            .comparingInt((Task t) -> t.priority)
            .thenComparingInt(t -> t.id)
    );
```

This is cleaner than repeatedly storing unexplained array positions.

---

# 59. Advanced: Priority Queue With Multiple Criteria

Suppose:

```text
1. Higher priority first.
2. Shorter duration first.
3. Smaller ID first.
```

```java
PriorityQueue<Task> pq =
    new PriorityQueue<>(
        Comparator
            .comparingInt((Task t) -> t.priority)
            .reversed()
            .thenComparingInt(t -> t.duration)
            .thenComparingInt(t -> t.id)
    );
```

Always check that the comparator matches the intended priority exactly.

---

# 60. Heap + Greedy Proof

A heap does not automatically make an algorithm greedy-correct.

There are two separate questions:

### Correctness

Why is choosing the current highest-priority candidate optimal?

### Efficiency

How can we find that candidate efficiently?

The heap answers:

```text
efficiency
```

The greedy proof answers:

```text
correctness
```

This distinction matters in interviews.

---

# 61. Huffman Coding

Huffman coding repeatedly combines the two least frequent symbols.

Use:

```text
min heap
```

Algorithm:

```text
1. Insert all frequencies.
2. Extract two smallest.
3. Combine them.
4. Insert their sum.
5. Repeat.
```

This is structurally identical to the minimum-cost rope problem.

Complexity:

```text
O(n log n)
```

The deeper pattern is:

```text
repeatedly combine two smallest items
```

→ min heap.

---

# 62. IPO / Capital Selection Pattern

A common greedy problem:

- Each project has required capital.
- Each project provides profit.
- You can choose at most K projects.
- At every step, choose the most profitable currently affordable project.

Use:

```text
sort projects by required capital
+
max heap by profit
```

Process:

```text
add all affordable projects
→ choose maximum profit
→ update capital
→ repeat
```

This is a classic:

```text
availability ordering + priority ordering
```

pattern.

---

# 63. Maximum Events Pattern

Suppose events have:

```text
start day
end day
```

At each day:

1. Add events starting today.
2. Remove expired events.
3. Attend the event with the earliest end date.

Use:

```text
min heap of end dates
```

This is another scheduling pattern:

```text
time sweep
+
priority queue
+
greedy choice
```

---

# 64. Heap + HashMap Design Pattern

A powerful general structure is:

```text
HashMap<Key, State>
PriorityQueue<Candidate>
```

The hashmap answers:

```text
What is the current state of this key?
```

The heap answers:

```text
Which candidate currently has the best priority?
```

Examples:

```text
Top K frequency
Dynamic scheduling
Lazy deletion
Graph shortest paths
Task prioritization
Streaming selection
```

---

# 65. When NOT to Use a Heap

Do not use a heap just because the problem contains the word:

```text
minimum
maximum
```

Consider alternatives.

### Need maximum of every sliding window?

Monotonic deque can achieve:

```text
O(n)
```

### Need arbitrary ordered search?

TreeSet / TreeMap may be better.

### Need K-th element once?

Quickselect may be better.

### Need all elements sorted?

Sorting may be simpler.

### Need frequency lookup only?

HashMap may be enough.

The correct data structure depends on the operations required.

---

# 66. Heap vs Monotonic Deque

Example:

```text
Sliding Window Maximum
```

Heap:

```text
O(n log k)
```

Monotonic deque:

```text
O(n)
```

But a heap is more general.

Use a heap when you need:

```text
arbitrary priority
```

Use a monotonic deque when the priority is determined by:

```text
monotonic value + window expiration
```

---

# 67. Heap vs Balanced BST

| Requirement | Heap | TreeMap/TreeSet |
|---|---|---|
| Get min/max | Excellent | Excellent |
| Insert | O(log n) | O(log n) |
| Remove root | O(log n) | O(log n) |
| Arbitrary key deletion | Poor | Excellent |
| Search key | O(n) | O(log n) |
| Predecessor/successor | No direct support | Yes |
| Frequency map | Separate structure | Can encode counts |
| Simplicity | High | Moderate |

---

# 68. Heap Interview Checklist

Before using a heap, ask:

```text
1. What is the priority?
2. Do I need min or max?
3. Is the candidate set dynamic?
4. Do I repeatedly need the best candidate?
5. Can I restrict the heap to K elements?
6. Are new candidates becoming available over time?
7. Do I need arbitrary deletion?
8. Do I need to know the identity of the candidate?
9. Can stale entries appear?
10. Would a deque, TreeMap, sorting, or Quickselect be better?
```

---

# 69. Java PriorityQueue Cheat Sheet

## Min heap

```java
PriorityQueue<Integer> pq =
    new PriorityQueue<>();
```

## Max heap

```java
PriorityQueue<Integer> pq =
    new PriorityQueue<>(Comparator.reverseOrder());
```

## Insert

```java
pq.offer(x);
```

## Peek

```java
pq.peek();
```

## Remove root

```java
pq.poll();
```

## Size

```java
pq.size();
```

## Empty

```java
pq.isEmpty();
```

## Custom comparator

```java
PriorityQueue<int[]> pq =
    new PriorityQueue<>(
        (a, b) -> Integer.compare(a[0], b[0])
    );
```

## Multiple criteria

```java
PriorityQueue<Task> pq =
    new PriorityQueue<>(
        Comparator
            .comparingInt((Task t) -> t.priority)
            .thenComparingInt(t -> t.id)
    );
```

---

# 70. GATE / CS Theory Focus

Know precisely:

### Heap structure

```text
Complete binary tree
```

### Array representation

```text
parent = (i - 1) / 2
left   = 2i + 1
right  = 2i + 2
```

### Operations

```text
peek          O(1)
insert        O(log n)
extract root  O(log n)
heapify       O(log n)
build heap    O(n)
```

### Important distinction

```text
Heap ≠ sorted array
Heap ≠ binary search tree
```

### Applications

```text
Priority scheduling
Top K
K-way merge
Heap Sort
Median maintenance
Graph algorithms
Greedy algorithms
```

---

# 71. GATE-Style Solved Questions

> These are **GATE-style practice questions**, not claims about a particular official GATE year.

---

## Question 1 — Build Heap Complexity

An arbitrary array of `n` elements is converted into a binary heap using the standard bottom-up heap construction algorithm.

What is the asymptotic complexity?

A. O(log n)

B. O(n)

C. O(n log n)

D. O(n²)

### Solution

Bottom-up construction starts from the last non-leaf node and heapifies downward.

Although one heapify can take:

```text
O(log n)
```

most nodes are near the leaves and require very little work.

The total work is:

```text
O(n)
```

Therefore:

```text
Answer = B
```

---

## Question 2 — K-th Largest

For:

```text
[7, 2, 9, 4, 1, 8]
```

find the 2nd largest element using a min heap of size 2.

### Solution

Process:

```text
7 → [7]
2 → [2,7]
9 → [7,9]
4 → [7,9]   // 4 discarded
1 → [7,9]   // 1 discarded
8 → [8,9]
```

The root is:

```text
8
```

Therefore:

```text
2nd largest = 8
```

Complexity:

```text
O(n log k)
```

---

## Question 3 — Median With Two Heaps

Numbers arrive in this order:

```text
5, 2, 10, 4
```

What is the median after all insertions?

### Solution

Sorted values:

```text
[2,4,5,10]
```

There are four values.

Therefore:

```text
median = (4 + 5) / 2
       = 4.5
```

Using two heaps:

```text
lower max heap → [2,4]
upper min heap → [5,10]
```

Therefore:

```text
median = (4 + 5) / 2
       = 4.5
```

---

# 72. Serious Mastery Checklist

## Heap Fundamentals

- [ ] Complete binary tree.
- [ ] Min heap property.
- [ ] Max heap property.
- [ ] Array representation.
- [ ] Parent/child formulas.
- [ ] Peek.
- [ ] Insert.
- [ ] Extract.
- [ ] Heapify.
- [ ] Build heap in O(n).

## Java

- [ ] `PriorityQueue`.
- [ ] Min heap.
- [ ] Max heap.
- [ ] Custom comparator.
- [ ] Multiple comparator criteria.
- [ ] Object-based heap.
- [ ] Avoid comparator overflow.

## Top K

- [ ] K largest.
- [ ] K smallest.
- [ ] K-th largest.
- [ ] K-th smallest.
- [ ] K most frequent.
- [ ] K closest.
- [ ] Fixed-size heap.

## K-Way

- [ ] Merge K arrays.
- [ ] Merge K linked lists.
- [ ] K sorted streams.
- [ ] O(N log K) analysis.

## Two Heaps

- [ ] Lower half max heap.
- [ ] Upper half min heap.
- [ ] Balance sizes.
- [ ] Median retrieval.
- [ ] Streaming median.

## Scheduling

- [ ] Arrival sorting.
- [ ] Available-task heap.
- [ ] Priority-based execution.
- [ ] Earliest deadline.
- [ ] Resource scheduling.
- [ ] Maximum events.

## Heap + HashMap

- [ ] Frequency + heap.
- [ ] Top K frequent.
- [ ] Lazy deletion.
- [ ] Current state validation.

## Advanced

- [ ] Dijkstra.
- [ ] Prim.
- [ ] Huffman.
- [ ] Minimum-cost rope connection.
- [ ] IPO-style greedy.
- [ ] Heap vs Quickselect.
- [ ] Heap vs monotonic deque.
- [ ] Heap vs TreeMap.

---

# 73. Final Revision Sheet

```text
HEAP / PRIORITY QUEUE
│
├── Fundamentals
│   ├── Complete binary tree
│   ├── Min heap
│   ├── Max heap
│   ├── Heapify
│   ├── Insert
│   └── Extract
│
├── Top K
│   ├── K largest → min heap
│   ├── K smallest → max heap
│   ├── K-th largest
│   ├── K-th smallest
│   ├── K frequent
│   └── K closest
│
├── K-Way Merge
│   ├── Arrays
│   ├── Linked lists
│   └── Sorted streams
│
├── Two Heaps
│   ├── Lower half → max heap
│   ├── Upper half → min heap
│   └── Median
│
├── Scheduling
│   ├── Arrival ordering
│   ├── Available tasks
│   ├── Priority
│   └── Greedy
│
└── Heap + HashMap
    ├── Frequencies
    ├── Dynamic state
    └── Lazy deletion
```

### Highest-value rules

```text
MIN HEAP
→ smallest element at root

MAX HEAP
→ largest element at root

K LARGEST
→ min heap of size K

K SMALLEST
→ max heap of size K

K-WAY MERGE
→ min heap containing one candidate from each source

MEDIAN
→ max heap for lower half
→ min heap for upper half

REPEATED BEST CHOICE
→ priority queue

DYNAMIC STATE + BEST CANDIDATE
→ hashmap + heap

BUILD HEAP
→ O(n)

INSERT / EXTRACT
→ O(log n)

PEEK
→ O(1)
```

### Final interview principle

A heap is most useful when the problem repeatedly asks:

```text
"What is the best available candidate right now?"
```

If candidates change dynamically, a heap avoids repeatedly scanning all candidates.

The core transformation is:

```text
Many candidates
      ↓
Define priority
      ↓
PriorityQueue
      ↓
Extract best candidate
      ↓
Update state
      ↓
Add newly eligible candidates
      ↓
Repeat
```

Master this pattern and heaps become a reusable tool across Top K, scheduling, greedy algorithms, streaming data, graph algorithms, and K-way merging.
