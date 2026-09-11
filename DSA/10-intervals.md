# 10 — Intervals

> **Goal:** Master interval problems as a core interview pattern. The central skill is recognizing when sorting, greedy processing, heaps, sweep-line events, or difference arrays turn a seemingly complex interval problem into a simple linear scan.

---

# 1. What Is an Interval?

An interval represents a continuous range:

```text
[start, end]
```

Examples:

```text
[1, 5]
[3, 7]
[10, 15]
```

Depending on the problem, endpoints may be:

- Inclusive: `[start, end]`
- Half-open: `[start, end)`
- Exclusive: `(start, end)`

Always determine the endpoint convention before coding.

For example:

```text
[1, 3] and [3, 5]
```

overlap if endpoints are inclusive.

But:

```text
[1, 3) and [3, 5)
```

do not overlap.

---

# 2. Why Interval Problems Matter

Interval problems appear frequently in interviews because they test:

- Sorting.
- Greedy algorithms.
- Two pointers.
- Priority queues.
- Sweep line.
- Difference arrays.
- Event processing.
- Boundary reasoning.
- Custom comparators.

A large percentage of interval problems follow a small number of reusable patterns.

---

# 3. The Core Interval Workflow

When you see intervals, ask:

```text
1. Are intervals already sorted?
2. If not, what should I sort by?
3. Do I need to merge?
4. Do I need to detect overlap?
5. Do I need the maximum simultaneous intervals?
6. Do I need the minimum resources?
7. Can I convert intervals into start/end events?
8. Can a difference array represent the changes?
9. Is this a greedy scheduling problem?
```

The most important decision is usually:

> **What ordering or event representation makes the required invariant easy to maintain?**

---

# 4. Sorting Intervals

The default representation is:

```java
int[][] intervals
```

Sort by start:

```java
Arrays.sort(
    intervals,
    (a, b) -> Integer.compare(a[0], b[0])
);
```

Sort by end:

```java
Arrays.sort(
    intervals,
    (a, b) -> Integer.compare(a[1], b[1])
);
```

Sort by start, then end:

```java
Arrays.sort(
    intervals,
    (a, b) -> {
        int cmp = Integer.compare(a[0], b[0]);
        if (cmp != 0) {
            return cmp;
        }
        return Integer.compare(a[1], b[1]);
    }
);
```

Avoid:

```java
(a, b) -> a[0] - b[0]
```

because subtraction can overflow.

Prefer:

```java
Integer.compare(a[0], b[0])
```

---

# 5. Merge Intervals

## Problem

Given intervals, merge all overlapping intervals.

Example:

```text
Input:
[[1,3], [2,6], [8,10], [9,12]]

Output:
[[1,6], [8,12]]
```

---

# 5.1 Key Observation

Sort by start.

After sorting:

```text
[1,3]
[2,6]
[8,10]
[9,12]
```

Maintain:

```text
currentStart
currentEnd
```

For the next interval:

```text
[nextStart, nextEnd]
```

If:

```text
nextStart <= currentEnd
```

the intervals overlap.

Extend:

```text
currentEnd = max(currentEnd, nextEnd)
```

Otherwise:

```text
save current interval
start a new interval
```

---

# 5.2 Java

```java
static int[][] merge(int[][] intervals) {
    if (intervals.length <= 1) {
        return intervals;
    }

    Arrays.sort(
        intervals,
        (a, b) -> Integer.compare(a[0], b[0])
    );

    List<int[]> result = new ArrayList<>();

    int start = intervals[0][0];
    int end = intervals[0][1];

    for (int i = 1; i < intervals.length; i++) {
        int nextStart = intervals[i][0];
        int nextEnd = intervals[i][1];

        if (nextStart <= end) {
            end = Math.max(end, nextEnd);
        } else {
            result.add(new int[]{start, end});

            start = nextStart;
            end = nextEnd;
        }
    }

    result.add(new int[]{start, end});

    return result.toArray(new int[result.size()][]);
}
```

Complexity:

```text
Sorting = O(n log n)
Scanning = O(n)

Total = O(n log n)
```

Output storage:

```text
O(n)
```

---

# 5.3 Why Sorting by Start Works

After sorting:

```text
start₁ <= start₂ <= start₃ ...
```

For the current merged interval `[start, end]`, every future interval starts no earlier than the current one.

Therefore the only question is:

```text
Does nextStart <= end?
```

If yes, merge.

If no, no later interval can overlap the current interval through an earlier start.

This is the invariant that makes the linear scan correct.

---

# 6. Insert Interval

## Problem

You have sorted, non-overlapping intervals and must insert:

```text
newInterval
```

while maintaining sorted, non-overlapping output.

Example:

```text
Intervals:
[[1,3], [6,9]]

New:
[2,5]

Output:
[[1,5], [6,9]]
```

---

# 6.1 Three Phases

For each interval:

### Phase 1 — Completely before new interval

```text
interval.end < newStart
```

Add it directly.

### Phase 2 — Overlapping

```text
interval.start <= newEnd
```

Merge it:

```text
newStart = min(newStart, interval.start)
newEnd = max(newEnd, interval.end)
```

### Phase 3 — Completely after

```text
interval.start > newEnd
```

Add the new interval, then append all remaining intervals.

---

# 6.2 Java

```java
static int[][] insert(
        int[][] intervals,
        int[] newInterval) {

    List<int[]> result = new ArrayList<>();

    int i = 0;
    int n = intervals.length;

    // Intervals completely before newInterval.
    while (i < n && intervals[i][1] < newInterval[0]) {
        result.add(intervals[i]);
        i++;
    }

    // Overlapping intervals.
    while (i < n && intervals[i][0] <= newInterval[1]) {
        newInterval[0] =
            Math.min(newInterval[0], intervals[i][0]);

        newInterval[1] =
            Math.max(newInterval[1], intervals[i][1]);

        i++;
    }

    result.add(newInterval);

    // Intervals completely after newInterval.
    while (i < n) {
        result.add(intervals[i]);
        i++;
    }

    return result.toArray(new int[result.size()][]);
}
```

Complexity:

```text
Time  = O(n)
Space = O(n) for output
```

No additional sorting is necessary because the input is already sorted.

---

# 7. Overlapping Intervals

The exact definition of overlap depends on endpoint semantics.

For inclusive intervals:

```text
[a, b]
[c, d]
```

they overlap when:

```text
c <= b
```

assuming:

```text
a <= c
```

after sorting by start.

They do not overlap when:

```text
c > b
```

Example:

```text
[1,5]
[5,8]
```

Inclusive interpretation:

```text
overlap
```

Half-open interpretation:

```text
[1,5)
[5,8)
```

no overlap.

---

# 7.1 Detect Any Overlap

Given arbitrary intervals:

1. Sort by start.
2. Compare each start with the previous maximum end.

Simple version:

```java
static boolean hasOverlap(int[][] intervals) {
    if (intervals.length <= 1) {
        return false;
    }

    Arrays.sort(
        intervals,
        (a, b) -> Integer.compare(a[0], b[0])
    );

    int previousEnd = intervals[0][1];

    for (int i = 1; i < intervals.length; i++) {
        if (intervals[i][0] <= previousEnd) {
            return true;
        }

        previousEnd = intervals[i][1];
    }

    return false;
}
```

For the standard sorted-by-start case, if an overlap is found, the problem is solved.

---

# 7.2 Why Use Maximum End?

For more general overlap scans, maintain:

```text
maxEnd = maximum end seen so far
```

Then:

```text
currentStart <= maxEnd
```

means the current interval intersects at least one previous interval.

This is especially useful when earlier intervals can extend farther than the immediately previous interval.

---

# 8. Non-Overlapping Intervals

A common problem:

> Remove the minimum number of intervals so that the remaining intervals do not overlap.

Example:

```text
[1,2]
[2,3]
[3,4]
[1,3]
```

One possible answer:

```text
remove [1,3]
```

Remaining:

```text
[1,2]
[2,3]
[3,4]
```

---

# 8.1 Greedy Strategy

Sort by **end time**.

Then repeatedly keep the interval that finishes earliest.

Why?

An interval ending earlier leaves more room for future intervals.

This is the classic interval scheduling greedy rule.

---

# 8.2 Java

```java
static int eraseOverlapIntervals(int[][] intervals) {
    if (intervals.length <= 1) {
        return 0;
    }

    Arrays.sort(
        intervals,
        (a, b) -> Integer.compare(a[1], b[1])
    );

    int kept = 1;
    int end = intervals[0][1];

    for (int i = 1; i < intervals.length; i++) {
        if (intervals[i][0] >= end) {
            kept++;
            end = intervals[i][1];
        }
    }

    return intervals.length - kept;
}
```

Complexity:

```text
O(n log n)
```

---

# 8.3 Alternative Interpretation

Instead of minimizing removals:

```text
min removals
```

maximize:

```text
number of non-overlapping intervals kept
```

Then:

```text
answer = n - maximumKept
```

This transformation is often the key insight.

---

# 9. Interval Scheduling

## Problem

Given intervals representing jobs/activities:

```text
[start, end]
```

select the maximum number of mutually non-overlapping intervals.

---

# 9.1 Greedy Rule

Sort by:

```text
end time ascending
```

Then:

```text
select earliest finishing compatible interval
```

Example:

```text
[1,4]
[3,5]
[0,6]
[5,7]
[3,9]
[5,9]
[6,10]
[8,11]
```

Choose:

```text
[1,4]
[5,7]
[8,11]
```

depending on endpoint convention.

---

# 9.2 Why Earliest Finish Is Optimal

Suppose two available intervals are:

```text
A = [1,4]
B = [2,5]
```

If you choose `A`, it frees the schedule at time `4`.

If you choose `B`, it frees the schedule only at time `5`.

Choosing `A` cannot reduce the number of future opportunities compared with choosing `B`.

This exchange argument establishes the greedy choice.

---

# 9.3 Generic Java

```java
static int maxNonOverlapping(int[][] intervals) {
    if (intervals.length == 0) {
        return 0;
    }

    Arrays.sort(
        intervals,
        (a, b) -> Integer.compare(a[1], b[1])
    );

    int count = 0;
    int lastEnd = Integer.MIN_VALUE;

    for (int[] interval : intervals) {
        if (interval[0] >= lastEnd) {
            count++;
            lastEnd = interval[1];
        }
    }

    return count;
}
```

---

# 10. Meeting Rooms

## Problem

Determine whether one person can attend all meetings.

Example:

```text
[[0,30], [5,10], [15,20]]
```

Cannot attend all because:

```text
[0,30]
```

overlaps both other meetings.

---

# 10.1 Approach

Sort by start time.

Then check:

```text
current.start < previous.end
```

for overlap.

If the problem treats touching intervals as non-overlapping, use:

```text
current.start >= previous.end
```

as the compatible condition.

---

# 10.2 Java

```java
static boolean canAttendMeetings(int[][] intervals) {
    if (intervals.length <= 1) {
        return true;
    }

    Arrays.sort(
        intervals,
        (a, b) -> Integer.compare(a[0], b[0])
    );

    for (int i = 1; i < intervals.length; i++) {
        if (intervals[i][0] < intervals[i - 1][1]) {
            return false;
        }
    }

    return true;
}
```

Complexity:

```text
O(n log n)
```

---

# 11. Meeting Rooms II

## Problem

Given meeting intervals, find the minimum number of rooms required.

Example:

```text
[0,30]
[5,10]
[15,20]
```

At time `5`:

```text
[0,30]
[5,10]
```

are active.

Therefore at least:

```text
2 rooms
```

are required.

---

# 11.1 Heap Solution

Sort meetings by start.

Maintain a min-heap of end times.

The heap contains the end time of each currently occupied room.

For every meeting:

```text
If earliest ending meeting has ended:
    reuse its room.

Otherwise:
    allocate another room.
```

---

# 11.2 Java

```java
static int minMeetingRooms(int[][] intervals) {
    if (intervals.length == 0) {
        return 0;
    }

    Arrays.sort(
        intervals,
        (a, b) -> Integer.compare(a[0], b[0])
    );

    PriorityQueue<Integer> minHeap = new PriorityQueue<>();

    for (int[] interval : intervals) {
        if (!minHeap.isEmpty()
                && minHeap.peek() <= interval[0]) {
            minHeap.poll();
        }

        minHeap.offer(interval[1]);
    }

    return minHeap.size();
}
```

Complexity:

```text
Sorting: O(n log n)
Heap:    O(n log n)

Total:   O(n log n)
Space:   O(n)
```

---

# 11.3 Why a Min-Heap?

We only need to know:

```text
Which occupied room becomes free first?
```

That is exactly the minimum end time.

A min-heap gives:

```text
minimum end time
```

in:

```text
O(log n)
```

per insertion/removal.

---

# 12. Minimum Rooms / Resources

Meeting rooms are a special case of a broader problem:

> Given intervals representing resource usage, find the minimum number of resources required so that overlapping intervals can coexist.

Examples:

- Meeting rooms.
- Classrooms.
- CPU resources.
- Servers.
- Machines.
- Parking spaces.
- Examination halls.
- Hospital rooms.
- Rental equipment.

The fundamental quantity is:

```text
maximum number of simultaneously active intervals
```

Therefore:

```text
minimum resources
=
maximum overlap
```

---

# 13. Meeting Rooms II With Two Sorted Arrays

Instead of a heap:

1. Extract all starts.
2. Extract all ends.
3. Sort both.
4. Use two pointers.

Example:

```text
starts:
[0, 5, 15]

ends:
[10, 20, 30]
```

Maintain:

```text
rooms
maxRooms
```

If next start occurs before the earliest end:

```text
rooms++
```

Otherwise:

```text
rooms--
```

---

# 13.1 Java

```java
static int minMeetingRoomsTwoPointers(int[][] intervals) {
    int n = intervals.length;

    if (n == 0) {
        return 0;
    }

    int[] starts = new int[n];
    int[] ends = new int[n];

    for (int i = 0; i < n; i++) {
        starts[i] = intervals[i][0];
        ends[i] = intervals[i][1];
    }

    Arrays.sort(starts);
    Arrays.sort(ends);

    int start = 0;
    int end = 0;

    int rooms = 0;
    int maxRooms = 0;

    while (start < n) {
        if (starts[start] < ends[end]) {
            rooms++;
            maxRooms = Math.max(maxRooms, rooms);
            start++;
        } else {
            rooms--;
            end++;
        }
    }

    return maxRooms;
}
```

### Endpoint note

The condition:

```java
starts[start] < ends[end]
```

means a meeting ending exactly when another begins can reuse the same room.

For inclusive overlap semantics, the tie rule may need to change.

---

# 14. Sweep Line

Sweep Line converts intervals into events.

Instead of processing an entire interval:

```text
[start, end]
```

process changes at its boundaries.

For each interval:

```text
start → +1
end   → -1
```

Then scan events in sorted order.

The running sum represents:

```text
number of active intervals
```

The maximum running sum is:

```text
maximum overlap
```

---

# 14.1 Example

Intervals:

```text
[1,5]
[2,6]
[4,8]
```

Events:

```text
1 → +1
2 → +1
4 → +1
5 → -1
6 → -1
8 → -1
```

Running count:

```text
time  count
1      1
2      2
4      3
5      2
6      1
8      0
```

Maximum overlap:

```text
3
```

Therefore:

```text
minimum resources = 3
```

---

# 14.2 Java Event Representation

```java
static int maxOverlap(int[][] intervals) {
    List<int[]> events = new ArrayList<>();

    for (int[] interval : intervals) {
        events.add(new int[]{interval[0], +1});
        events.add(new int[]{interval[1], -1});
    }

    events.sort((a, b) -> {
        int cmp = Integer.compare(a[0], b[0]);

        if (cmp != 0) {
            return cmp;
        }

        // End before start at the same time.
        return Integer.compare(a[1], b[1]);
    });

    int active = 0;
    int answer = 0;

    for (int[] event : events) {
        active += event[1];
        answer = Math.max(answer, active);
    }

    return answer;
}
```

The tie-breaking rule depends on whether intervals are treated as:

```text
[start, end)
```

or:

```text
[start, end]
```

Never choose the tie rule blindly.

---

# 15. Sweep Line: Why It Works

An interval changes the active count only at its boundaries.

Between two consecutive event coordinates:

```text
no interval starts or ends
```

Therefore:

```text
active count is constant
```

We only need to process boundary changes.

This is the fundamental sweep-line idea:

```text
continuous-looking problem
        ↓
boundary events
        ↓
sort events
        ↓
linear scan
```

---

# 16. Sweep Line Applications

Common applications:

- Maximum overlapping intervals.
- Meeting rooms.
- Minimum resources.
- Number of active users.
- Website traffic.
- CPU load.
- Skyline-style problems.
- Calendar conflicts.
- Range coverage.
- Geometric interval problems.
- Collision/event processing.

---

# 17. Difference Array for Intervals

A difference array records **changes** rather than full values.

Suppose:

```text
diff[x]
```

represents how the active count changes at coordinate `x`.

For interval:

```text
[l, r)
```

do:

```text
diff[l]++
diff[r]--
```

Then calculate prefix sums.

The prefix sum at position `x` gives:

```text
number of active intervals at x
```

---

# 17.1 Example

Intervals:

```text
[1,4)
[2,5)
[3,6)
```

Apply:

```text
[1,4):
diff[1]++
diff[4]--

[2,5):
diff[2]++
diff[5]--

[3,6):
diff[3]++
diff[6]--
```

Conceptually:

```text
diff:
index: 0 1 2 3 4 5 6
value: 0 1 1 1 -1 -1 -1
```

Prefix sum:

```text
index: 0 1 2 3 4 5 6
active:0 1 2 3 2 1 0
```

Maximum active count:

```text
3
```

---

# 17.2 Java

For a small bounded coordinate range:

```java
static int maxOverlapDifferenceArray(
        int[][] intervals,
        int maxCoordinate) {

    int[] diff = new int[maxCoordinate + 2];

    for (int[] interval : intervals) {
        int start = interval[0];
        int end = interval[1];

        diff[start]++;
        diff[end]--;
    }

    int active = 0;
    int maxActive = 0;

    for (int x = 0; x <= maxCoordinate; x++) {
        active += diff[x];
        maxActive = Math.max(maxActive, active);
    }

    return maxActive;
}
```

This assumes half-open semantics:

```text
[start, end)
```

and a manageable integer coordinate range.

---

# 18. Difference Array vs Sweep Line

They solve closely related problems.

| Property | Difference Array | Sweep Line |
|---|---|---|
| Coordinate range | Usually small/bounded | Can be huge |
| Representation | Indexed array | Sorted events |
| Main operation | Prefix sum | Event scan |
| Typical complexity | O(n + C) | O(n log n) |
| Memory | O(C) | O(n) |
| Coordinate compression | Often unnecessary if bounded | Natural |
| Large coordinates | Poor | Good |

Where:

```text
C = coordinate range
```

---

# 19. Coordinate Compression + Difference Array

Suppose coordinates are huge:

```text
[1000000000, 1000000010]
[5000000000, 5000000020]
```

A direct difference array is impossible.

Instead:

1. Collect all relevant coordinates.
2. Sort unique coordinates.
3. Map them to compressed indices.
4. Apply difference updates.
5. Prefix-sum over compressed coordinates.

However, when the physical distance between coordinates matters, remember that compressed indices no longer represent actual distances.

For:

```text
[1, 100]
[100, 101]
```

the difference between coordinates is important if calculating covered length.

Compression must therefore preserve the coordinate values separately.

---

# 20. Interval Coverage

A related problem:

> Find total length covered by at least one interval.

Example:

```text
[1,5]
[3,7]
[10,12]
```

Merge first:

```text
[1,7]
[10,12]
```

Coverage:

```text
(7 - 1) + (12 - 10)
= 6 + 2
= 8
```

For half-open or continuous intervals, this formulation is straightforward.

---

# 20.1 Merged Interval Coverage

```java
static long totalCoveredLength(int[][] intervals) {
    if (intervals.length == 0) {
        return 0;
    }

    Arrays.sort(
        intervals,
        (a, b) -> Integer.compare(a[0], b[0])
    );

    long total = 0;

    int start = intervals[0][0];
    int end = intervals[0][1];

    for (int i = 1; i < intervals.length; i++) {
        int nextStart = intervals[i][0];
        int nextEnd = intervals[i][1];

        if (nextStart <= end) {
            end = Math.max(end, nextEnd);
        } else {
            total += (long) end - start;

            start = nextStart;
            end = nextEnd;
        }
    }

    total += (long) end - start;

    return total;
}
```

Use `long` when coordinate differences or totals can exceed `int`.

---

# 21. Interval Intersection

Given two lists of sorted, non-overlapping intervals, find their intersections.

Example:

```text
A:
[0,2]
[5,10]

B:
[1,5]
[8,12]
```

Intersections:

```text
[1,2]
[5,5]
[8,10]
```

Use two pointers.

For:

```text
A = [aStart, aEnd]
B = [bStart, bEnd]
```

intersection is:

```text
left  = max(aStart, bStart)
right = min(aEnd, bEnd)
```

If:

```text
left <= right
```

there is an intersection under inclusive semantics.

Then advance the interval that ends first.

---

# 21. Two-Pointer Interval Intersection Java

```java
static List<int[]> intervalIntersection(
        int[][] first,
        int[][] second) {

    List<int[]> result = new ArrayList<>();

    int i = 0;
    int j = 0;

    while (i < first.length && j < second.length) {
        int left = Math.max(first[i][0], second[j][0]);
        int right = Math.min(first[i][1], second[j][1]);

        if (left <= right) {
            result.add(new int[]{left, right});
        }

        if (first[i][1] < second[j][1]) {
            i++;
        } else {
            j++;
        }
    }

    return result;
}
```

Complexity:

```text
O(n + m)
```

because the inputs are already sorted.

---

# 22. Interval Containment

An interval:

```text
[a,b]
```

contains:

```text
[c,d]
```

if:

```text
a <= c
and
d <= b
```

After sorting by:

```text
start ascending
```

containment can often be detected using a maximum end value.

For example:

```text
[1,10]
[2,5]
```

The second interval is contained in the first.

---

# 23. Sorting Rule Selection

A major interview skill is knowing what to sort by.

| Problem | Typical ordering |
|---|---|
| Merge intervals | Start ascending |
| Insert interval | Already sorted by start |
| Detect overlap | Start ascending |
| Remove minimum overlaps | End ascending |
| Maximum compatible intervals | End ascending |
| Meeting rooms | Start ascending |
| Meeting rooms II | Start ascending |
| Sweep line | Event coordinate |
| Event sweep | Time + tie-breaking |
| Custom scheduling | Depends on objective |

Do not memorize one universal interval comparator.

---

# 24. Why End-Time Greedy Works

For maximum compatible intervals:

```text
Choose earliest finishing interval.
```

Suppose an optimal solution chooses:

```text
B
```

as its first interval.

Suppose:

```text
A.end <= B.end
```

where `A` is the earliest finishing available interval.

Replace `B` with `A`.

Any interval that could come after `B` can also come after `A`, because:

```text
A.end <= B.end
```

Therefore there is an optimal solution beginning with `A`.

This is an **exchange argument**.

Recognizing this proof pattern is important for interview and GATE-level greedy questions.

---

# 25. Minimum Resources = Maximum Overlap

This is one of the most important interval identities:

```text
minimum resources required
=
maximum simultaneous active intervals
```

Example:

```text
[1,5]
[2,4]
[3,6]
```

At time `3`:

```text
all three are active
```

Therefore:

```text
minimum resources = 3
```

This immediately suggests:

- Min-heap.
- Sweep line.
- Difference array.

---

# 26. Heap vs Sweep Line

Both can solve minimum meeting rooms.

## Min-heap

Use when you want to track:

```text
which resource becomes free next
```

Typical:

```text
sort by start
min-heap of end times
```

## Sweep line

Use when you only need:

```text
maximum number of simultaneous intervals
```

Typical:

```text
start = +1
end = -1
sort events
prefix sum
```

### Interview decision

Ask:

> Do I need to know the identity/state of the earliest finishing resource, or only the maximum active count?

If only the count is required, sweep line is often simpler.

---

# 27. Difference Array vs Prefix Sum

The difference array itself does not contain the final active count.

It stores changes:

```text
diff[x]
```

Then:

```text
active[x]
=
active[x - 1] + diff[x]
```

Therefore:

```text
prefix sum(diff)
```

produces the actual interval coverage.

This pattern generalizes beyond intervals.

---

# 28. Range Additions

Difference arrays are especially useful for:

> Apply many range additions and obtain the final values.

For an update:

```text
add value v to [l, r]
```

do:

```text
diff[l] += v
diff[r + 1] -= v
```

Then prefix sum.

For interval activity with half-open intervals:

```text
[l, r)
```

use:

```text
diff[l]++
diff[r]--
```

The exact endpoint update is determined by the interval convention.

---

# 29. Common Interval Bugs

## Bug 1: Wrong overlap condition

For inclusive intervals:

```text
nextStart <= currentEnd
```

means overlap.

For half-open intervals:

```text
nextStart < currentEnd
```

means overlap.

---

## Bug 2: Sorting by the wrong field

For:

```text
merge intervals
```

usually sort by start.

For:

```text
maximum number of non-overlapping intervals
```

usually sort by end.

---

## Bug 3: Forgetting to update maximum end

When merging or detecting overlap, an interval may extend farther than the immediately previous end.

Use:

```java
end = Math.max(end, nextEnd);
```

---

## Bug 4: Incorrect event tie-breaking

If a meeting ends exactly when another begins, decide whether the room can be reused.

This determines whether:

```text
end
```

comes before:

```text
start
```

at the same coordinate.

---

## Bug 5: Modifying input unintentionally

Sorting:

```java
Arrays.sort(intervals, ...)
```

mutates the input array.

If input preservation matters, copy first.

---

## Bug 6: Integer overflow

For large coordinates:

```java
end - start
```

may overflow `int`.

Use:

```java
(long) end - start
```

when appropriate.

---

## Bug 7: Difference-array bounds

If using:

```java
diff[r + 1]--
```

make sure the array has enough capacity.

---

# 30. Interval Problem Classification

When you see a problem, classify it.

## Type A — Merge

Keywords:

```text
merge
combine
union
consolidate
```

Pattern:

```text
sort by start
scan
```

---

## Type B — Conflict Detection

Keywords:

```text
overlap
conflict
can attend all
collision
```

Pattern:

```text
sort by start
compare boundaries
```

---

## Type C — Maximum Compatible Set

Keywords:

```text
maximum number
most activities
non-overlapping
remove minimum conflicts
```

Pattern:

```text
sort by end
greedy
```

---

## Type D — Minimum Resources

Keywords:

```text
minimum rooms
minimum machines
minimum servers
minimum classrooms
```

Pattern:

```text
maximum overlap
```

Use:

```text
heap
or
sweep line
or
difference array
```

---

## Type E — Event Processing

Keywords:

```text
at each time
active
simultaneous
maximum traffic
```

Pattern:

```text
sweep line
```

---

## Type F — Bounded Integer Coordinates

Keywords:

```text
range
days
small time domain
```

Pattern:

```text
difference array
```

---

# 31. Advanced Pattern: Sweep Line With Event Objects

For more complicated problems, events can contain:

```text
time
type
id
value
```

Example:

```java
class Event {
    int time;
    int type;
    int id;

    Event(int time, int type, int id) {
        this.time = time;
        this.type = type;
        this.id = id;
    }
}
```

Then:

```java
events.sort(
    Comparator
        .comparingInt((Event e) -> e.time)
        .thenComparingInt(e -> e.type)
);
```

This is useful when an event must carry more information than:

```text
+1 / -1
```

---

# 32. Advanced Pattern: Maximum Overlap With IDs

Sometimes you need:

> Which resource is free first?

Then an event count alone is insufficient.

Use:

```text
PriorityQueue
```

with resource IDs.

Example:

```java
PriorityQueue<int[]> pq =
    new PriorityQueue<>(
        Comparator.comparingInt(a -> a[0])
    );
```

where:

```text
a[0] = end time
a[1] = resource ID
```

This lets you recover the actual resource rather than only the count.

---

# 33. Interval Scheduling With Weights

Standard interval scheduling maximizes:

```text
number of intervals
```

But if every interval has a value:

```text
[start, end, profit]
```

and the goal is:

```text
maximum total profit
```

the simple end-time greedy algorithm no longer works.

This becomes **Weighted Interval Scheduling**.

Typical approach:

1. Sort by end time.
2. For each interval, find the latest compatible interval.
3. Dynamic programming.

Recurrence:

```text
dp[i]
=
max(
    dp[i - 1],
    profit[i] + dp[previousCompatible(i)]
)
```

This is an important distinction:

```text
maximum count → greedy
maximum weighted value → DP
```

---

# 34. Interval Scheduling vs Meeting Rooms

These look similar but require different objectives.

### Interval Scheduling

Goal:

```text
maximize intervals selected
```

Typical:

```text
sort by end
greedy
```

### Meeting Rooms II

Goal:

```text
minimize resources
```

Typical:

```text
maximum overlap
heap / sweep line
```

The intervals may be identical, but the optimization objective changes the algorithm.

---

# 35. Interval Partitioning

Interval partitioning asks:

> Partition intervals into the minimum number of groups such that no two overlapping intervals are in the same group.

This is essentially the same structure as:

```text
minimum meeting rooms
```

The answer equals:

```text
maximum overlap
```

A min-heap can assign each interval to the earliest available group.

---

# 36. Resource Assignment With Heap

Suppose:

```text
[1,4]
[2,5]
[4,7]
[6,8]
```

Sort by start.

At `[1,4]`:

```text
Room 1
```

At `[2,5]`:

```text
Room 2
```

At `[4,7]`:

```text
Room 1 becomes free
reuse Room 1
```

At `[6,8]`:

```text
Room 2 becomes free
reuse Room 2
```

Minimum rooms:

```text
2
```

If you need the actual room assignments, store:

```text
(endTime, roomId)
```

in the heap.

---

# 37. Interval Merging Invariant

For merge intervals:

```text
[start, end]
```

maintain:

```text
all intervals processed so far
→ represented by result intervals
```

For the current interval:

```text
[currentStart, currentEnd]
```

Invariant:

```text
currentEnd
=
furthest right endpoint of the merged group
```

When overlap occurs:

```text
currentEnd =
max(currentEnd, nextEnd)
```

This invariant is more important than memorizing the code.

---

# 38. Interval Scheduling Invariant

After processing the first `i` intervals sorted by end:

```text
selected intervals form a maximum-size compatible set
```

for the processed prefix, with the selected set ending as early as possible among optimal choices.

This is why the earliest-finish greedy choice can safely continue.

---

# 39. Sweep Line Invariant

After processing all events up to coordinate `x`:

```text
active
=
number of intervals active immediately after applying the chosen tie-order at x
```

Therefore:

```text
maxActive
```

is the maximum simultaneous overlap under the chosen endpoint semantics.

---

# 40. Difference Array Invariant

For each coordinate `x`:

```text
active(x)
=
sum(diff[0 ... x])
```

Each interval contributes:

```text
+1
```

at its start and:

```text
-1
```

when it stops contributing.

Therefore prefix sums recover coverage.

---

# 41. Complexity Summary

| Problem | Typical Approach | Time | Space |
|---|---|---:|---:|
| Merge intervals | Sort + scan | O(n log n) | O(n) output |
| Insert interval | Linear scan | O(n) | O(n) output |
| Detect overlap | Sort + scan | O(n log n) | O(1)/O(n) depending implementation |
| Remove overlaps | Sort by end + greedy | O(n log n) | O(1) extra |
| Interval scheduling | Sort by end + greedy | O(n log n) | O(1) extra |
| Meeting Rooms | Sort by start | O(n log n) | O(1) extra |
| Meeting Rooms II | Sort + heap | O(n log n) | O(n) |
| Meeting Rooms II | Sorted starts/ends | O(n log n) | O(n) |
| Maximum overlap | Sweep line | O(n log n) | O(n) |
| Bounded coordinates | Difference array | O(n + C) | O(C) |
| Interval intersection | Two pointers | O(n + m) | O(k) output |
| Weighted scheduling | Sort + DP + binary search | O(n log n) | O(n) |

---

# 42. LeetCode Roadmap

## Easy

Focus on:

- Basic interval overlap.
- Simple interval insertion.
- Simple merging.
- Interval intersection.
- Basic sorting of intervals.
- Range updates with bounded coordinates.

## Medium

Prioritize:

- Merge Intervals.
- Insert Interval.
- Non-overlapping Intervals.
- Meeting Rooms.
- Meeting Rooms II.
- Interval List Intersections.
- Minimum resources.
- Event scheduling.
- Maximum events.
- Sweep-line counting.
- Difference-array range updates.
- Coordinate compression.

## Hard

Practice:

- Weighted interval scheduling variants.
- Advanced sweep-line problems.
- Skyline-style problems.
- Complex resource assignment.
- Dynamic interval coverage.
- Coordinate compression + segment tree/Fenwick tree.
- Multi-event scheduling.
- Advanced interval DP.

---

# 43. GATE / CS Theory Focus

Know these concepts precisely:

### Interval overlap

For inclusive:

```text
[a,b] and [c,d]

overlap if:

max(a,c) <= min(b,d)
```

For half-open:

```text
[a,b) and [c,d)

overlap if:

max(a,c) < min(b,d)
```

### Interval scheduling

```text
Sort by earliest finishing time.
```

### Resource allocation

```text
Minimum resources = maximum simultaneous overlap.
```

### Sweep line

```text
Convert ranges to boundary events.
Sort events.
Maintain active state.
```

### Difference array

```text
Range update
→ boundary changes
→ prefix sum
```

---

# 44. GATE-Style Solved Questions

> These are **GATE-style practice questions**, not claims about a particular official GATE year.

---

## Question 1 — Minimum Rooms

Meetings are:

```text
[1,4]
[2,5]
[6,8]
[3,7]
```

What is the minimum number of rooms required, assuming a meeting ending at time `t` frees its room for another meeting starting at `t`?

### Solution

At time range around `3`:

```text
[1,4]
[2,5]
[3,7]
```

Three meetings overlap.

At time `6`:

```text
[2,5] ended
[1,4] ended
```

Only:

```text
[3,7]
[6,8]
```

are active.

Therefore maximum simultaneous meetings:

```text
3
```

Hence:

```text
Minimum rooms = 3
```

---

## Question 2 — Interval Scheduling

Consider:

```text
[1,4]
[3,5]
[0,6]
[5,7]
[3,9]
[5,9]
[6,10]
[8,11]
```

Using the earliest-finish-time greedy strategy, which intervals can be selected to maximize the number of non-overlapping intervals?

### Solution

Sort by end:

```text
[1,4]
[3,5]
[0,6]
[5,7]
[3,9]
[5,9]
[6,10]
[8,11]
```

Choose:

```text
[1,4]
```

Next compatible interval:

```text
[5,7]
```

Next compatible interval:

```text
[8,11]
```

Therefore one maximum-size selection is:

```text
[1,4]
[5,7]
[8,11]
```

Count:

```text
3
```

---

## Question 3 — Difference Array

Intervals are half-open:

```text
[1,4)
[2,5)
[3,6)
```

What is the maximum number of simultaneously active intervals?

### Solution

Apply:

```text
[1,4):
diff[1] += 1
diff[4] -= 1

[2,5):
diff[2] += 1
diff[5] -= 1

[3,6):
diff[3] += 1
diff[6] -= 1
```

Prefix counts:

```text
time     active
1          1
2          2
3          3
4          2
5          1
6          0
```

Maximum:

```text
3
```

Therefore:

```text
Answer = 3
```

---

# 45. Serious Mastery Checklist

## Core

- [ ] Understand interval endpoint semantics.
- [ ] Sort intervals by start.
- [ ] Sort intervals by end.
- [ ] Detect overlap.
- [ ] Merge intervals.
- [ ] Insert intervals.
- [ ] Find interval intersections.

## Greedy

- [ ] Interval scheduling.
- [ ] Maximum non-overlapping intervals.
- [ ] Minimum removals.
- [ ] Exchange argument for earliest finish.

## Resources

- [ ] Meeting Rooms.
- [ ] Meeting Rooms II.
- [ ] Minimum rooms.
- [ ] Minimum machines.
- [ ] Resource assignment with heap.

## Sweep Line

- [ ] Convert intervals into events.
- [ ] Sort events.
- [ ] Handle equal-coordinate tie-breaking.
- [ ] Track active count.
- [ ] Find maximum overlap.

## Difference Array

- [ ] Range increment.
- [ ] Start/end boundary updates.
- [ ] Prefix sum.
- [ ] Bounded coordinate problems.
- [ ] Difference array vs sweep line.

## Advanced

- [ ] Coordinate compression.
- [ ] Weighted interval scheduling.
- [ ] Interval partitioning.
- [ ] Interval coverage.
- [ ] Advanced sweep line.
- [ ] Heap + resource IDs.
- [ ] Interval DP.

---

# 46. Final Revision Sheet

```text
INTERVALS
│
├── Merge
│   ├── Sort by start
│   ├── Detect overlap
│   └── Extend end
│
├── Insert
│   ├── Before
│   ├── Overlapping
│   └── After
│
├── Overlap
│   ├── Endpoint semantics
│   └── Start/end comparison
│
├── Non-overlapping
│   ├── Sort by end
│   └── Greedy
│
├── Scheduling
│   ├── Earliest finish
│   └── Exchange argument
│
├── Meeting Rooms
│   └── Sort by start
│
├── Meeting Rooms II
│   ├── Min heap
│   ├── Two sorted arrays
│   └── Maximum overlap
│
├── Minimum Resources
│   └── Maximum simultaneous intervals
│
├── Sweep Line
│   ├── Start event
│   ├── End event
│   ├── Sort
│   └── Active count
│
└── Difference Array
    ├── + at start
    ├── - at end
    └── Prefix sum
```

### Highest-value patterns

```text
MERGE
sort by start
→ scan
→ merge if nextStart <= currentEnd

MAXIMUM NON-OVERLAPPING
sort by end
→ greedily choose earliest finish

MINIMUM RESOURCES
maximum overlap
→ heap / sweep line / difference array

SWEEP LINE
interval
→ start/end events
→ sort
→ maintain active state

DIFFERENCE ARRAY
range update
→ boundary changes
→ prefix sum
```

### Final interview rule

Do not start coding immediately after seeing intervals.

First identify the objective:

```text
Merge?
Conflict?
Maximum compatible set?
Minimum resources?
Maximum overlap?
Coverage?
Scheduling?
Range updates?
```

Then choose the representation:

```text
sort by start
sort by end
min-heap
two pointers
events
difference array
coordinate compression
DP
```

The same interval data can require completely different algorithms depending on the question being asked.
