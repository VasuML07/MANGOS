# 17. Greedy Algorithms

> **Goal:** Master greedy algorithms for FAANG/top product-company SDE and ML/AI interviews, while covering the corresponding GATE theory and problem-solving patterns.
>
> **Language:** Java  
> **Difficulty target:** Easy → Medium → Hard, with Medium dominating.

---

## 1. What Is a Greedy Algorithm?

A greedy algorithm builds a solution step by step by making the **locally best choice** at each step, without reconsidering earlier choices.

The critical question is not:

> “What choice looks best right now?”

It is:

> “Can I prove that taking the locally optimal choice can still lead to a globally optimal solution?”

Greedy is useful when a problem has a suitable combination of:

1. **Greedy-choice property** — there exists an optimal solution beginning with the greedy choice.
2. **Optimal substructure** — after making the choice, the remaining problem is itself optimally solvable.

Greedy is generally much faster than exhaustive search or dynamic programming, but it is also easier to apply incorrectly.

---

# 2. Greedy vs Other Techniques

| Technique | Main idea | Typical clue |
|---|---|---|
| Greedy | Make a provably safe local choice | Sorting + repeatedly taking best candidate |
| DP | Store solutions to overlapping subproblems | Same states recur; local choice may fail |
| Backtracking | Explore choices and undo | Need to enumerate/search combinations |
| Divide & Conquer | Split into independent subproblems | Merge solutions |
| Graph algorithms | Optimize over graph structure | Paths, connectivity, ordering |
| Binary Search on Answer | Search feasible answer space | Monotonic feasibility |

### Important

A problem containing words such as **minimum**, **maximum**, or **optimal** is not automatically greedy.

You need a correctness argument.

---

# 3. The Three Core Greedy Proof Ideas

## 3.1 Exchange Argument

Show that any optimal solution can be transformed so that it contains the greedy choice **without making the solution worse**.

This is the classic proof for interval scheduling.

Suppose the greedy algorithm chooses interval `G`, and an optimal solution chooses interval `O` first.

If:

```text
finish(G) <= finish(O)
```

replace `O` with `G`.

The replacement leaves at least as much room for all later intervals.

Therefore, there exists an optimal solution containing `G`.

---

## 3.2 Staying-Ahead Argument

Show that after every step, the greedy solution is at least as good as any competing solution according to an appropriate measure.

Example:

For interval scheduling, after selecting `k` intervals, greedy's finishing time is no later than the finishing time of any other solution containing `k` compatible intervals.

Therefore greedy can never lose future opportunities because of its earlier choices.

---

## 3.3 Cut / Structural Argument

A greedy choice is safe because of the structure of the problem.

Examples:

- Kruskal chooses a safe minimum-weight edge across a cut.
- Huffman combines the two lowest-frequency nodes.
- Some scheduling problems choose the earliest finishing job.

---

# 4. How to Recognize Greedy Problems

Look for:

- “maximum number of non-overlapping activities”
- “minimum number of resources”
- “minimum cost”
- “maximize profit”
- “choose as many as possible”
- “earliest/latest”
- “sort and repeatedly select”
- “take the best currently available item”
- “can complete all tasks?”
- “minimum removals”
- “assign resources optimally”
- “merge smallest/largest items”
- “deadline + profit”
- “intervals + priority queue”

### Strong signals

```text
Sort → scan once
Sort → two pointers
Sort → priority queue
Sort → choose feasible candidate
```

---

# 5. Activity Selection

## 5.1 Problem

Given activities with start and finish times, select the **maximum number of mutually compatible activities**.

Two activities are compatible when:

```text
start(next) >= finish(previous)
```

---

## 5.2 Correct Greedy Choice

Sort activities by **increasing finish time**.

Then repeatedly choose the first activity whose start time is at least the finish time of the previously selected activity.

### Why earliest finish?

An activity finishing earlier leaves the largest possible remaining time for future activities.

---

## 5.3 Example

Activities:

```text
A: (1, 2)
B: (3, 4)
C: (0, 6)
D: (5, 7)
E: (8, 9)
F: (5, 9)
```

Sorted by finish:

```text
A (1,2)
B (3,4)
C (0,6)
D (5,7)
E (8,9)
F (5,9)
```

Choose:

```text
A → B → D → E
```

Total:

```text
4
```

---

## 5.4 Java

```java
import java.util.*;

class Activity {
    int start, finish;

    Activity(int start, int finish) {
        this.start = start;
        this.finish = finish;
    }
}

public class ActivitySelection {
    static int maxActivities(Activity[] activities) {
        Arrays.sort(activities,
                Comparator.comparingInt(a -> a.finish));

        int count = 0;
        int lastFinish = Integer.MIN_VALUE;

        for (Activity a : activities) {
            if (a.start >= lastFinish) {
                count++;
                lastFinish = a.finish;
            }
        }

        return count;
    }
}
```

### Complexity

```text
Sorting: O(n log n)
Scan:    O(n)
Total:   O(n log n)
Space:   O(1) auxiliary, ignoring sorting implementation
```

---

# 6. Interval Scheduling

Activity selection and interval scheduling are essentially the same core problem when the objective is:

> **Maximum number of mutually non-overlapping intervals.**

### Correct strategy

```text
Sort by end time
→ take earliest-finishing compatible interval
→ repeat
```

### Common mistake

Sorting by:

- start time
- duration
- interval length

does **not** generally maximize the number of intervals.

### Counterexample to “earliest start”

```text
(1, 100)
(2, 3)
(4, 5)
(6, 7)
```

Choosing earliest start gives only one interval.

Choosing earliest finish gives:

```text
(2,3), (4,5), (6,7)
```

---

# 7. Interval Scheduling Variants

The word “interval” does not automatically imply the same greedy rule.

| Problem | Typical method |
|---|---|
| Maximum non-overlapping intervals | Sort by end |
| Minimum removals for non-overlap | Maximum non-overlap / sort by end |
| Minimum meeting rooms | Sort starts + ends / heap |
| Meeting rooms II | Min-heap |
| Merge intervals | Sort by start |
| Insert interval | Scan / merge |
| Maximum events attended | Sort by start + min-heap |
| Weighted interval scheduling | DP, not simple greedy |

### Critical distinction

**Weighted interval scheduling**:

Each interval has a value/profit.

You cannot simply choose earliest finishing intervals.

This becomes a DP problem.

---

# 8. Fractional Knapsack

## 8.1 Problem

Given items:

```text
value[i]
weight[i]
```

You may take a **fraction** of an item.

Maximize total value subject to capacity `W`.

---

## 8.2 Greedy Choice

Sort by:

```text
value / weight
```

in decreasing order.

Take the item with the highest value density first.

If it does not fit completely, take the fraction that fits.

---

## 8.3 Example

Capacity:

```text
W = 50
```

| Item | Value | Weight | Value/Weight |
|---|---:|---:|---:|
| A | 60 | 10 | 6 |
| B | 100 | 20 | 5 |
| C | 120 | 30 | 4 |

Take:

```text
A → 10 weight
B → 20 weight
C → 20/30 fraction
```

Value:

```text
60 + 100 + 120 × 20/30
= 240
```

---

## 8.4 Why It Works

If an available item has a larger value-per-weight ratio than another item, replacing some weight allocated to the lower-ratio item with the higher-ratio item cannot reduce the total value.

Therefore the highest-density item should be consumed first.

---

## 8.5 Java

```java
import java.util.*;

class Item {
    int value;
    int weight;

    Item(int value, int weight) {
        this.value = value;
        this.weight = weight;
    }
}

public class FractionalKnapsack {
    static double maxValue(Item[] items, int capacity) {
        Arrays.sort(items, (a, b) -> {
            double r1 = (double) a.value / a.weight;
            double r2 = (double) b.value / b.weight;
            return Double.compare(r2, r1);
        });

        double answer = 0.0;

        for (Item item : items) {
            if (capacity == 0) break;

            if (item.weight <= capacity) {
                answer += item.value;
                capacity -= item.weight;
            } else {
                answer += (double) item.value / item.weight * capacity;
                capacity = 0;
            }
        }

        return answer;
    }
}
```

### Complexity

```text
O(n log n) time
O(1) auxiliary space excluding sorting
```

---

# 9. Fractional vs 0/1 Knapsack

This distinction is extremely important.

| Property | Fractional | 0/1 |
|---|---|---|
| Can split item? | Yes | No |
| Greedy by value/weight | Correct | Not generally correct |
| Typical technique | Greedy | DP |
| Time | O(n log n) | O(nW) typical DP |

### Example

For 0/1 knapsack:

```text
Item A: value 60, weight 10
Item B: value 100, weight 20
Item C: value 120, weight 30
W = 50
```

Greedy by ratio can select A+B+C only if capacity permits; in other instances it can fail because an indivisible item may produce a better combination.

Never transfer the fractional-knapsack greedy proof to 0/1 knapsack.

---

# 10. Jump Game

## 10.1 Problem

Given:

```text
nums[i]
```

where `nums[i]` is the maximum jump length from index `i`.

Determine whether the last index is reachable.

---

## 10.2 Greedy Idea

Maintain the farthest reachable index:

```text
farthest = max(farthest, i + nums[i])
```

If:

```text
i > farthest
```

then index `i` cannot be reached and the answer is false.

---

## 10.3 Java

```java
public class JumpGame {
    static boolean canJump(int[] nums) {
        int farthest = 0;

        for (int i = 0; i < nums.length; i++) {
            if (i > farthest) {
                return false;
            }

            farthest = Math.max(farthest, i + nums[i]);

            if (farthest >= nums.length - 1) {
                return true;
            }
        }

        return true;
    }
}
```

### Complexity

```text
Time:  O(n)
Space: O(1)
```

---

# 11. Jump Game II

A common extension asks for the **minimum number of jumps**.

The greedy interpretation is:

- Current jump defines a reachable range.
- While scanning that range, find the farthest next reach.
- When reaching the end of the current range, commit to another jump.

```java
public class JumpGameII {
    static int minJumps(int[] nums) {
        if (nums.length <= 1) return 0;

        int jumps = 0;
        int currentEnd = 0;
        int farthest = 0;

        for (int i = 0; i < nums.length - 1; i++) {
            farthest = Math.max(farthest, i + nums[i]);

            if (i == currentEnd) {
                jumps++;
                currentEnd = farthest;
            }
        }

        return jumps;
    }
}
```

### Pattern

This is essentially **range expansion / BFS-level thinking implemented greedily**.

---

# 12. Gas Station

## 12.1 Problem

There are `n` gas stations.

At station `i`:

```text
gas[i]
```

is available and traveling to the next station costs:

```text
cost[i]
```

Find a starting station from which the complete circular route is possible.

---

## 12.2 Key Observation

If:

```text
sum(gas) < sum(cost)
```

no solution exists.

Otherwise, there is at least one valid starting point.

Maintain:

```text
tank += gas[i] - cost[i]
```

If `tank < 0`, the current starting point cannot work.

Set:

```text
start = i + 1
tank = 0
```

---

## 12.3 Why Can We Skip All Earlier Starts?

Suppose starting from `start` causes the tank to become negative at station `i`.

Then every station between `start` and `i` also cannot be a valid start.

Why?

Because the amount available before reaching `i` from any intermediate start is no greater in the relevant prefix than what caused the failure.

Therefore the next possible candidate is:

```text
i + 1
```

---

## 12.4 Java

```java
public class GasStation {
    static int canCompleteCircuit(int[] gas, int[] cost) {
        int total = 0;
        int tank = 0;
        int start = 0;

        for (int i = 0; i < gas.length; i++) {
            int gain = gas[i] - cost[i];

            total += gain;
            tank += gain;

            if (tank < 0) {
                start = i + 1;
                tank = 0;
            }
        }

        return total >= 0 ? start : -1;
    }
}
```

### Complexity

```text
Time:  O(n)
Space: O(1)
```

---

# 13. Job Scheduling

“Job scheduling” can refer to several different problems.

Do not memorize one generic greedy rule.

## 13.1 Job Sequencing with Deadlines

Each job has:

```text
deadline
profit
```

Each job takes one unit of time.

Goal:

> Maximize total profit.

### Greedy strategy

1. Sort jobs by decreasing profit.
2. Place each job in the latest available slot on or before its deadline.

Why latest slot?

It preserves earlier slots for jobs with tighter deadlines.

---

## 13.2 Java

```java
import java.util.*;

class Job {
    char id;
    int deadline;
    int profit;

    Job(char id, int deadline, int profit) {
        this.id = id;
        this.deadline = deadline;
        this.profit = profit;
    }
}

public class JobSequencing {
    static int maxProfit(Job[] jobs) {
        Arrays.sort(jobs, (a, b) -> b.profit - a.profit);

        int maxDeadline = 0;
        for (Job job : jobs) {
            maxDeadline = Math.max(maxDeadline, job.deadline);
        }

        boolean[] used = new boolean[maxDeadline + 1];
        int profit = 0;

        for (Job job : jobs) {
            for (int slot = Math.min(job.deadline, maxDeadline);
                 slot >= 1;
                 slot--) {

                if (!used[slot]) {
                    used[slot] = true;
                    profit += job.profit;
                    break;
                }
            }
        }

        return profit;
    }
}
```

### Complexity

Basic implementation:

```text
O(n log n + nD)
```

where `D` is the maximum deadline.

With a DSU-based slot structure, slot assignment can be optimized further.

---

# 14. Job Sequencing: Important GATE Concept

If every job:

- takes exactly one unit of time,
- has a deadline,
- gives profit if completed by deadline,

then the classic greedy method is:

```text
Highest profit first
+
latest available slot
```

### Why not earliest slot?

Placing a job as early as possible wastes flexibility.

The latest available position preserves earlier positions for jobs that need them.

---

# 15. Meeting Scheduling

There are several common meeting problems.

## 15.1 Maximum Meetings in One Room

If meetings cannot overlap and all have equal value:

```text
Sort by end time
→ select compatible meetings
```

This is activity selection.

---

## 15.2 Minimum Meeting Rooms

Given meeting intervals, find the minimum number of rooms required.

This is **not** solved by simply selecting meetings.

### Method 1: Min-Heap

1. Sort meetings by start time.
2. Maintain a min-heap of meeting end times.
3. If the earliest ending meeting ends before the next meeting starts, reuse its room.
4. Otherwise allocate another room.

---

## 15.3 Java

```java
import java.util.*;

public class MeetingRooms {
    static int minRooms(int[][] meetings) {
        if (meetings.length == 0) return 0;

        Arrays.sort(meetings, Comparator.comparingInt(a -> a[0]));

        PriorityQueue<Integer> pq = new PriorityQueue<>();

        for (int[] meeting : meetings) {
            if (!pq.isEmpty() && pq.peek() <= meeting[0]) {
                pq.poll();
            }

            pq.offer(meeting[1]);
        }

        return pq.size();
    }
}
```

### Complexity

```text
Sorting: O(n log n)
Heap operations: O(n log n)
Total: O(n log n)
Space: O(n)
```

---

# 16. Minimum Platforms

Classic railway-platform problem:

Given arrival and departure times, find the minimum number of platforms required so no train waits.

### Greedy idea

Treat arrivals and departures as two sorted streams.

At the next event:

- arrival → need one more platform
- departure → release one platform

Track maximum simultaneous trains.

---

## 16.1 Two-Pointer Java

```java
import java.util.*;

public class MinimumPlatforms {
    static int findPlatform(int[] arrival, int[] departure) {
        Arrays.sort(arrival);
        Arrays.sort(departure);

        int i = 0;
        int j = 0;
        int platforms = 0;
        int answer = 0;

        while (i < arrival.length && j < departure.length) {
            if (arrival[i] <= departure[j]) {
                platforms++;
                answer = Math.max(answer, platforms);
                i++;
            } else {
                platforms--;
                j++;
            }
        }

        return answer;
    }
}
```

### Important boundary condition

If a train arriving exactly when another departs is considered to need a separate platform:

```text
arrival <= departure
```

is used.

If the problem explicitly allows immediate reuse at the same timestamp, the comparison may change.

Always follow the problem's interval convention.

---

# 17. Huffman Coding

## 17.1 Problem

Given characters with frequencies, construct a prefix-free binary code minimizing the weighted total code length.

---

## 17.2 Greedy Choice

Repeatedly combine the **two least frequent** nodes.

Algorithm:

```text
Insert all frequencies into min-heap

while heap size > 1:
    a = remove minimum
    b = remove minimum
    merged = a + b
    insert merged

remaining node = root
```

---

## 17.3 Example

Frequencies:

```text
A = 5
B = 9
C = 12
D = 13
E = 16
F = 45
```

Combine:

```text
5 + 9 = 14
12 + 13 = 25
14 + 16 = 30
25 + 30 = 55
45 + 55 = 100
```

The resulting binary tree gives shorter codes to higher-frequency symbols.

---

## 17.4 Why Two Minimum Frequencies?

In an optimal prefix code, two least-frequent symbols can be placed as sibling leaves at the greatest depth.

Combining them into a single pseudo-symbol reduces the problem to a smaller instance.

This is the central greedy argument behind Huffman coding.

---

## 17.5 Java

```java
import java.util.*;

class HuffmanNode {
    char ch;
    int freq;
    HuffmanNode left, right;

    HuffmanNode(char ch, int freq) {
        this.ch = ch;
        this.freq = freq;
    }

    HuffmanNode(int freq, HuffmanNode left, HuffmanNode right) {
        this.freq = freq;
        this.left = left;
        this.right = right;
    }
}

public class HuffmanCoding {
    static HuffmanNode buildTree(char[] chars, int[] freq) {
        PriorityQueue<HuffmanNode> pq =
                new PriorityQueue<>(Comparator.comparingInt(a -> a.freq));

        for (int i = 0; i < chars.length; i++) {
            pq.offer(new HuffmanNode(chars[i], freq[i]));
        }

        while (pq.size() > 1) {
            HuffmanNode a = pq.poll();
            HuffmanNode b = pq.poll();

            HuffmanNode merged =
                    new HuffmanNode(a.freq + b.freq, a, b);

            pq.offer(merged);
        }

        return pq.poll();
    }

    static void generateCodes(HuffmanNode node,
                              String code,
                              Map<Character, String> result) {
        if (node == null) return;

        if (node.left == null && node.right == null) {
            result.put(node.ch, code.isEmpty() ? "0" : code);
            return;
        }

        generateCodes(node.left, code + "0", result);
        generateCodes(node.right, code + "1", result);
    }
}
```

### Complexity

For `n` symbols:

```text
O(n log n)
```

---

# 18. Greedy + Sorting

Sorting often exposes the ordering that makes a greedy choice safe.

Common patterns:

### Pattern A — Sort by End

Used for:

- Activity selection
- Maximum non-overlapping intervals
- Minimum interval removals

```text
finish ↑
```

---

### Pattern B — Sort by Ratio

Used for:

- Fractional knapsack

```text
value / weight ↓
```

---

### Pattern C — Sort by Profit

Used for:

- Job sequencing

```text
profit ↓
```

---

### Pattern D — Sort by Start

Used for:

- Merge intervals
- Meeting room processing
- Event processing

```text
start ↑
```

---

### Pattern E — Sort Both Arrays

Used for:

- Minimum platforms
- Some assignment problems

---

# 19. Greedy + Heap

A heap is useful when the best choice changes dynamically.

General structure:

```text
Sort by one dimension
+
PriorityQueue for the best currently available choice
```

Common examples:

- Meeting Rooms II
- Huffman Coding
- Maximum Events Attended
- IPO
- Minimum cost to connect ropes
- Task scheduling variants
- Deadline scheduling

---

## 19.1 Minimum Cost to Connect Ropes

Given rope lengths, combine two smallest ropes repeatedly.

If combining ropes of lengths `a` and `b` costs:

```text
a + b
```

insert:

```text
a + b
```

back into the heap.

### Java

```java
import java.util.*;

public class ConnectRopes {
    static long minCost(int[] ropes) {
        PriorityQueue<Integer> pq = new PriorityQueue<>();

        for (int x : ropes) {
            pq.offer(x);
        }

        long cost = 0;

        while (pq.size() > 1) {
            int a = pq.poll();
            int b = pq.poll();

            int merged = a + b;
            cost += merged;

            pq.offer(merged);
        }

        return cost;
    }
}
```

### Pattern

```text
Repeatedly combine the two smallest
→ min-heap
```

This is structurally the same greedy pattern as Huffman coding.

---

# 20. Greedy + Intervals

Many interview problems become easier after converting them into interval events.

## 20.1 Maximum Non-Overlapping Intervals

```text
sort by end
```

---

## 20.2 Minimum Meeting Rooms

```text
sort by start
+
min-heap of ends
```

---

## 20.3 Merge Intervals

```text
sort by start
→ merge overlapping intervals
```

Note:

> Merge intervals is not primarily a greedy optimization problem, but it uses a greedy scan after sorting.

---

## 20.4 Minimum Removals to Make Intervals Non-Overlapping

Equivalent to:

```text
n - maximum number of non-overlapping intervals
```

Therefore:

```text
sort by end
→ greedily retain maximum compatible intervals
```

---

## 20.5 Maximum Number of Events

Typical approach:

1. Sort events by start day.
2. For each day, add all events that have started to a min-heap by ending day.
3. Remove expired events.
4. Attend the event ending earliest.
5. Continue.

This is a classic:

```text
time sweep + heap
```

pattern.

---

# 21. Important Greedy Interval Problems

| Problem type | Main idea |
|---|---|
| Maximum activities | End-time greedy |
| Maximum meetings | End-time greedy |
| Minimum removals | Max non-overlap |
| Meeting rooms | Start sorting + min-heap |
| Merge intervals | Start sorting |
| Insert interval | Ordered scan |
| Attend maximum events | Day sweep + min-heap |
| Weighted interval scheduling | DP |
| Minimum arrows for balloons | End-coordinate greedy |
| Partition labels | Last occurrence + greedy boundary |

---

# 22. Gas Station Pattern

Gas Station represents a broader pattern:

> When a candidate fails at position `i`, prove that a whole range of candidates can be discarded.

This can turn an apparently quadratic search into a linear scan.

### Generic reasoning

```text
Try candidate
↓
Failure proves candidates in a range are impossible
↓
Jump directly past the failed range
```

This is often the key to recognizing an O(n) greedy solution.

---

# 23. Greedy Correctness Checklist

Before coding a greedy solution, answer:

### Question 1

What is the exact local choice?

Example:

```text
Choose the interval with earliest finish.
```

### Question 2

Why is that choice safe?

Use:

- exchange argument
- staying-ahead argument
- structural argument

### Question 3

After choosing it, what remains?

It should be a smaller instance of the same optimization problem or a structure that can be greedily continued.

### Question 4

Can a counterexample break the rule?

Try small adversarial cases.

### Question 5

Is the problem actually DP?

If the local choice can sacrifice a better future combination, greedy may fail.

---

# 24. Greedy Failure Examples

## 24.1 0/1 Knapsack

Ratio-based greedy is not generally optimal.

---

## 24.2 Weighted Interval Scheduling

Choosing the earliest-finishing interval may sacrifice a high-value interval.

Use DP.

---

## 24.3 Coin Change

For some denominations, choosing the largest coin first fails.

Example:

```text
coins = [1, 3, 4]
amount = 6
```

Greedy:

```text
4 + 1 + 1 = 3 coins
```

Optimal:

```text
3 + 3 = 2 coins
```

---

## 24.4 Longest Increasing Subsequence

A naive greedy choice does not directly solve the problem.

The standard optimal solution uses:

```text
DP
```

or:

```text
binary search + tails
```

---

# 25. Greedy vs DP Decision Framework

Ask:

```text
Can I make a local choice and prove it is always safe?
        |
       YES
        ↓
       Greedy
        |
       NO
        ↓
Do choices interact with future value?
        |
       YES
        ↓
       DP / other technique
```

---

# 26. Common Interview Mistakes

## Mistake 1 — Choosing by the wrong ordering

For interval scheduling:

```text
start time ≠ correct
duration ≠ correct
end time = correct
```

---

## Mistake 2 — Assuming every optimization is greedy

Always seek a proof.

---

## Mistake 3 — Confusing maximum count with maximum value

Maximum number of intervals:

```text
greedy by end
```

Maximum weighted interval value:

```text
DP
```

---

## Mistake 4 — Ignoring tie rules

For intervals, clarify:

```text
start >= previousEnd
```

or whether touching endpoints overlap.

---

## Mistake 5 — Integer division in ratios

Wrong:

```java
a.value / a.weight
```

when both are integers.

Use:

```java
(double) a.value / a.weight
```

---

## Mistake 6 — Overflow

For large values:

```java
long
```

may be required for accumulated cost/profit.

---

# 27. Java Greedy Templates

## 27.1 Sort Array of Objects

```java
Arrays.sort(arr, Comparator.comparingInt(x -> x.end));
```

Descending:

```java
Arrays.sort(arr, (a, b) -> Integer.compare(b.profit, a.profit));
```

---

## 27.2 Priority Queue

Min-heap:

```java
PriorityQueue<Integer> pq = new PriorityQueue<>();
```

Max-heap:

```java
PriorityQueue<Integer> pq =
        new PriorityQueue<>(Collections.reverseOrder());
```

Custom:

```java
PriorityQueue<Job> pq =
        new PriorityQueue<>(
            Comparator.comparingInt(j -> j.deadline)
        );
```

---

## 27.3 Two Pointers

```java
Arrays.sort(a);
Arrays.sort(b);

int i = 0;
int j = 0;

while (i < a.length && j < b.length) {
    // process smaller event
}
```

Useful for:

- platforms
- event sweeps
- assignment problems

---

# 28. Greedy Pattern Table

| Pattern | Sort / structure | Choice |
|---|---|---|
| Activity selection | End ascending | Earliest finish |
| Interval scheduling | End ascending | Compatible earliest finish |
| Fractional knapsack | Ratio descending | Highest density |
| Jump Game | Scan | Farthest reach |
| Jump Game II | Scan range | Farthest next reach |
| Gas Station | Scan | Restart after failure |
| Job sequencing | Profit descending | Latest free slot |
| Meeting rooms | Start ascending + min-heap | Earliest ending room |
| Minimum platforms | Arrival/departure sorting | Process next event |
| Huffman | Min-heap | Two smallest |
| Connect ropes | Min-heap | Two smallest |
| Event attendance | Start sorting + min-heap | Earliest ending available event |

---

# 29. GATE Theory

## 29.1 Greedy Choice Property

A problem has the greedy-choice property if an optimal solution can be constructed by making a locally optimal choice first.

---

## 29.2 Optimal Substructure

An optimal solution contains optimal solutions to appropriate subproblems.

Greedy often relies on both concepts, although the exact proof structure depends on the problem.

---

## 29.3 Exchange Argument

A standard way to establish greedy correctness.

General form:

```text
Let G be the greedy choice.
Let O be some optimal choice.

Show that O can be replaced by G
without worsening the objective.

Therefore an optimal solution exists
that begins with G.
```

---

## 29.4 Huffman Coding

For `n` symbols:

```text
n - 1
```

merge operations are performed.

The standard priority-queue implementation runs in:

```text
O(n log n)
```

---

## 29.5 Fractional Knapsack

If items are divisible, sorting by:

```text
value / weight
```

in descending order is optimal.

---

## 29.6 Activity Selection

Sorting by increasing finish time yields an optimal maximum-cardinality schedule.

---

## 29.7 Job Sequencing

Under the unit-time job and deadline assumptions:

```text
profit descending
+
latest available slot
```

maximizes total profit.

---

# 30. GATE-Style Solved Questions

> These are **GATE-style practice questions**, not claimed as verbatim official GATE PYQs.

---

## Question 1 — Activity Selection

Consider activities:

```text
A = (1, 4)
B = (3, 5)
C = (0, 6)
D = (5, 7)
E = (3, 9)
F = (5, 9)
G = (6, 10)
H = (8, 11)
I = (8, 12)
J = (2, 14)
K = (12, 16)
```

What is the maximum number of mutually compatible activities?

### Solution

Sort by finish time:

```text
A (1,4)
B (3,5)
C (0,6)
D (5,7)
E (3,9)
F (5,9)
G (6,10)
H (8,11)
I (8,12)
J (2,14)
K (12,16)
```

Greedy:

```text
A → D → H → K
```

Count:

```text
4
```

### Answer

```text
4
```

---

## Question 2 — Fractional Knapsack

Capacity:

```text
W = 50
```

Items:

| Item | Value | Weight |
|---|---:|---:|
| 1 | 100 | 20 |
| 2 | 120 | 30 |
| 3 | 60 | 10 |

Find the maximum value.

### Solution

Ratios:

```text
1: 100/20 = 5
2: 120/30 = 4
3: 60/10 = 6
```

Order:

```text
3 → 1 → 2
```

Take:

```text
Item 3: weight 10, value 60
Item 1: weight 20, value 100
```

Remaining capacity:

```text
50 - 30 = 20
```

Take:

```text
20/30 of item 2
```

Value:

```text
60 + 100 + 120 × (20/30)
= 160 + 80
= 240
```

### Answer

```text
240
```

---

## Question 3 — Gas Station

Given:

```text
gas  = [1, 2, 3, 4, 5]
cost = [3, 4, 5, 1, 2]
```

Find the valid starting index.

### Solution

Net gain:

```text
[-2, -2, -2, +3, +3]
```

Total:

```text
0
```

So a solution can exist.

Start:

```text
0
```

At station 0:

```text
tank = -2
```

Failure.

Therefore restart at:

```text
1
```

Station 1:

```text
tank = -2
```

Failure.

Restart:

```text
2
```

Station 2:

```text
tank = -2
```

Failure.

Restart:

```text
3
```

Station 3:

```text
tank = 3
```

Station 4:

```text
tank = 6
```

Station 0:

```text
tank = 4
```

Station 1:

```text
tank = 2
```

Station 2:

```text
tank = 0
```

Complete circuit.

### Answer

```text
3
```

---

## Question 4 — Meeting Rooms

Meetings:

```text
[0,30]
[5,10]
[15,20]
```

Minimum number of rooms?

### Solution

Sort by start:

```text
[0,30]
[5,10]
[15,20]
```

Heap of end times:

```text
30
```

Next meeting starts at 5:

```text
5 < 30
```

Need another room.

Heap:

```text
10, 30
```

Next meeting starts at 15:

```text
10 <= 15
```

Reuse room ending at 10.

Heap:

```text
20, 30
```

Maximum heap size:

```text
2
```

### Answer

```text
2
```

---

## Question 5 — Huffman Coding

Frequencies are:

```text
5, 9, 12, 13, 16, 45
```

What is the minimum weighted external path length?

### Solution

Merge:

```text
5 + 9  = 14
12 + 13 = 25
14 + 16 = 30
25 + 30 = 55
45 + 55 = 100
```

Total merge cost:

```text
14 + 25 + 30 + 55 + 100
= 224
```

### Answer

```text
224
```

The sum of Huffman merge costs equals the weighted external path length.

---

## Question 6 — Minimum Platforms

Arrival:

```text
[900, 940, 950, 1100, 1500, 1800]
```

Departure:

```text
[910, 1200, 1120, 1130, 1900, 2000]
```

Find the minimum number of platforms.

### Solution

Sort:

```text
Arrival:
900 940 950 1100 1500 1800

Departure:
910 1120 1130 1200 1900 2000
```

Process:

```text
900 arrival → 1
910 departure → 0
940 arrival → 1
950 arrival → 2
1100 arrival → 3
1120 departure → 2
1130 departure → 1
1200 departure → 0
1500 arrival → 1
1800 arrival → 2
```

Maximum:

```text
3
```

### Answer

```text
3
```

---

# 31. LeetCode Roadmap

## Easy

Start with:

1. Assign Cookies
2. Lemonade Change
3. Best Time to Buy and Sell Stock
4. Can Place Flowers
5. Maximum 69 Number
6. Split a String in Balanced Strings

Focus on recognizing whether a local choice is safe.

---

## Medium — Priority

These are especially important:

1. Jump Game
2. Jump Game II
3. Gas Station
4. Partition Labels
5. Non-overlapping Intervals
6. Minimum Number of Arrows to Burst Balloons
7. Task Scheduler
8. Hand of Straights
9. Boats to Save People
10. Queue Reconstruction by Height
11. Car Pooling
12. Meeting Rooms II
13. Maximum Number of Events That Can Be Attended
14. Remove K Digits
15. Task scheduling variants
16. Two City Scheduling
17. Minimum Cost to Hire K Workers
18. IPO

---

## Hard

Master these after the medium set:

1. Candy
2. Minimum Number of Refueling Stops
3. Patching Array
4. Create Maximum Number
5. Course Schedule III
6. Minimum Cost to Hire K Workers
7. Trapping Rain Water II
8. Employee Free Time
9. Construct Target Array With Multiple Sums

---

# 32. Problem-Pattern Mapping

When you see:

### “Maximum number of non-overlapping intervals”

Think:

```text
Sort by end
```

### “Maximum value and fractional items”

Think:

```text
value / weight
```

### “Minimum rooms”

Think:

```text
start sorting + min-heap
```

### “Two smallest repeatedly”

Think:

```text
min-heap
```

### “Jobs + deadlines + profit”

Think:

```text
profit descending + latest free slot
```

### “Reach as far as possible”

Think:

```text
farthest reachable index
```

### “Circular gas route”

Think:

```text
total balance + restart after negative prefix
```

### “Arrival/departure events”

Think:

```text
sort events / two pointers
```

---

# 33. Greedy Decision Tree

```text
                 Optimization problem
                         |
              Can choices be proved safe?
                    /             \
                  YES              NO
                   |                |
               Greedy          DP / other
                   |
          Is there an ordering?
             /            \
           YES             NO
            |               |
        Sort first      Heap / scan /
            |           structural rule
            |
    What is the ordering?
       |
       +-- earliest finish → interval scheduling
       |
       +-- ratio → fractional knapsack
       |
       +-- profit → job sequencing
       |
       +-- start time → interval/event sweep
       |
       +-- frequency → Huffman / heap
```

---

# 34. Advanced Greedy Patterns

## 34.1 Greedy + Two Pointers

Useful when two sorted event streams interact.

Examples:

- Minimum platforms
- Assigning resources
- Matching problems

---

## 34.2 Greedy + Heap

Use when:

```text
Candidates become available over time
+
need the best candidate among them
```

Examples:

```text
events
meetings
tasks
Huffman
IPO
```

---

## 34.3 Greedy + Sorting + DSU

Useful when selecting jobs/edges and maintaining available resources efficiently.

Example:

```text
Job sequencing with many large deadlines
```

A DSU can find the latest free slot efficiently.

---

## 34.4 Greedy + Binary Search

Some advanced problems combine greedy feasibility with binary search.

Pattern:

```text
Binary search answer
→ greedily test whether answer is feasible
```

Examples can include partitioning, scheduling, and capacity problems.

---

# 35. Interview-Level Complexity Summary

| Problem | Time | Space |
|---|---:|---:|
| Activity selection | O(n log n) | O(1) aux |
| Fractional knapsack | O(n log n) | O(1) aux |
| Jump Game | O(n) | O(1) |
| Jump Game II | O(n) | O(1) |
| Gas Station | O(n) | O(1) |
| Job sequencing | O(n log n + nD) | O(D) |
| Meeting Rooms II | O(n log n) | O(n) |
| Minimum platforms | O(n log n) | O(1) aux |
| Huffman | O(n log n) | O(n) |
| Connect ropes | O(n log n) | O(n) |
| Non-overlapping intervals | O(n log n) | O(1) aux |
| Merge intervals | O(n log n) | O(1) aux |

`D` = maximum deadline.

---

# 36. Mastery Checklist

You should be able to explain and implement each without looking up the solution.

## Fundamentals

- [ ] Define greedy algorithms.
- [ ] Explain greedy-choice property.
- [ ] Explain optimal substructure.
- [ ] Give an exchange argument.
- [ ] Give a staying-ahead argument.
- [ ] Distinguish greedy from DP.

## Scheduling

- [ ] Activity selection
- [ ] Interval scheduling
- [ ] Non-overlapping intervals
- [ ] Meeting Rooms II
- [ ] Minimum platforms
- [ ] Job sequencing with deadlines

## Classic Greedy

- [ ] Fractional knapsack
- [ ] Jump Game
- [ ] Jump Game II
- [ ] Gas Station
- [ ] Huffman coding
- [ ] Connect ropes

## Pattern combinations

- [ ] Greedy + sorting
- [ ] Greedy + heap
- [ ] Greedy + intervals
- [ ] Greedy + two pointers
- [ ] Greedy + DSU
- [ ] Greedy feasibility + binary search

## Failure recognition

- [ ] Explain why 0/1 knapsack needs DP.
- [ ] Explain why weighted interval scheduling needs DP.
- [ ] Construct a coin-change counterexample to naive greedy.
- [ ] Identify when local choices interact with future value.

---

# 37. Final Revision Sheet

## Activity Selection

```text
Sort by finish ↑
Take compatible intervals
```

## Fractional Knapsack

```text
Sort by value/weight ↓
Take maximum possible fraction
```

## Jump Game

```text
Maintain farthest reachable index
```

## Jump Game II

```text
Expand current reachable range
Commit jump at range boundary
```

## Gas Station

```text
If total gas < total cost → -1
If current tank < 0 → restart after failure
```

## Job Sequencing

```text
Sort profit ↓
Place each job in latest free slot <= deadline
```

## Meeting Rooms

```text
Sort starts
Min-heap of end times
Reuse earliest ending room
```

## Minimum Platforms

```text
Sort arrivals
Sort departures
Two pointers
Track maximum overlap
```

## Huffman

```text
Min-heap
Repeatedly merge two smallest frequencies
```

## Intervals

```text
Max count → sort by end
Merge → sort by start
Rooms → start + min-heap
Weighted intervals → DP
```

---

# 38. One-Page Greedy Mental Model

```text
GREEDY
│
├── Prove local choice is safe
│
├── SORT
│   ├── End time
│   ├── Start time
│   ├── Ratio
│   └── Profit
│
├── HEAP
│   ├── Two smallest
│   ├── Earliest ending
│   └── Best available candidate
│
├── SCAN
│   ├── Farthest reach
│   ├── Running balance
│   └── Event overlap
│
└── VERIFY
    ├── Exchange argument
    ├── Staying ahead
    └── Counterexample search
```

The most important interview skill is not memorizing “greedy problems.” It is learning to identify **what ordering makes the local choice provably safe**.

---

# 39. Final Mastery Standard

You have mastered Greedy Algorithms when you can:

1. Recognize a likely greedy structure within a few minutes.
2. State the greedy choice precisely.
3. Explain why the choice is safe.
4. Produce an exchange/staying-ahead proof when appropriate.
5. Implement the solution in Java without template dependence.
6. Identify whether sorting, a heap, two pointers, or a scan is required.
7. Handle interval boundary conditions correctly.
8. Distinguish fractional knapsack from 0/1 knapsack.
9. Distinguish ordinary interval scheduling from weighted interval scheduling.
10. Solve medium greedy problems consistently.
11. Derive complexity rather than memorizing it.
12. Construct counterexamples when a greedy rule is suspicious.
13. Recognize hybrid patterns such as greedy + heap and greedy + intervals.
14. Explain the algorithm clearly in an interview before writing code.

**Core principle:**

> **Greedy is not “choose what looks best.” Greedy is “choose what can be proven safe now.”**
