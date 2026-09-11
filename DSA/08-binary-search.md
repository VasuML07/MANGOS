# 8. Binary Search

> **Serious mastery topic.**
>
> Binary search is not just "search an element in a sorted array." The real interview skill is recognizing a **monotonic condition** and finding the boundary where it changes.

---

# 1. What You Must Master

## Basic

- Binary search
- Lower bound
- Upper bound
- First occurrence
- Last occurrence
- Search insertion position

## Variations

- Search rotated sorted array
- Find minimum in rotated sorted array
- Peak element
- Mountain array
- Search 2D matrix

## Binary Search on Answer

- Minimum possible answer
- Maximum possible answer
- Capacity problems
- Allocation problems
- Scheduling problems
- Aggressive placement problems
- K-th answer problems

---

# 2. Why Binary Search Deserves Serious Mastery

Binary search reduces a search space by roughly half at every step.

```text
n
↓
n/2
↓
n/4
↓
n/8
↓
...
```

Therefore:

```text
Time = O(log n)
```

But the most important interview pattern is broader:

> If the answer space has a monotonic true/false property, binary search may work even when the input array is not sorted.

Example:

```text
0 0 0 0 1 1 1 1
        ↑
   first true
```

Binary search can find the transition.

This idea is called:

> **Binary Search on Answer / Binary Search on a Monotonic Predicate**

---

# 3. Prerequisites

Before mastering binary search, be comfortable with:

- Arrays
- Sorting
- Big-O analysis
- Integer arithmetic
- Prefix sums
- Greedy algorithms
- Basic two pointers
- Basic problem-solving invariants

For answer-based binary search, greedy feasibility checks are especially important.

---

# 4. Basic Binary Search

Given a sorted array:

```text
[1, 3, 5, 7, 9, 11]
```

find:

```text
7
```

Maintain:

```text
left
right
```

and inspect:

```text
mid = left + (right - left) / 2
```

Avoid:

```java
mid = (left + right) / 2;
```

because `left + right` can overflow for large integer indices.

---

# 5. Standard Binary Search

```java
class Solution {
    public int search(int[] nums, int target) {
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
    }
}
```

### Complexity

```text
Time:  O(log n)
Space: O(1)
```

---

# 6. The Most Important Binary Search Question

Before writing code, decide:

> **What does `left`, `right`, and `mid` represent?**

There are several valid binary-search conventions.

The most common:

```text
[left, right]
```

inclusive interval.

Then:

```java
while (left <= right)
```

and eliminate:

```java
mid + 1
mid - 1
```

Another useful convention is:

```text
[left, right)
```

half-open interval.

Then:

```java
while (left < right)
```

and usually:

```java
right = mid;
```

or:

```java
left = mid + 1;
```

Do not mix conventions.

---

# 7. Binary Search Invariant

An invariant describes what remains true throughout the search.

For ordinary exact search:

```text
If target exists, it is inside [left, right].
```

Each iteration preserves this.

When:

```text
nums[mid] < target
```

everything at or before `mid` is too small.

Therefore:

```java
left = mid + 1;
```

When:

```text
nums[mid] > target
```

everything at or after `mid` is too large.

Therefore:

```java
right = mid - 1;
```

---

# 8. Lower Bound

Lower bound means:

> Find the first index `i` such that `nums[i] >= target`.

Example:

```text
nums   = [1, 2, 4, 4, 4, 7, 9]
target = 4
```

Lower bound:

```text
index = 2
```

For:

```text
target = 5
```

lower bound:

```text
index = 5
```

because:

```text
nums[5] = 7 >= 5
```

---

# 9. Lower Bound Template

Use a half-open search interval:

```text
[0, n)
```

```java
int lowerBound(int[] nums, int target) {
    int left = 0;
    int right = nums.length;

    while (left < right) {
        int mid = left + (right - left) / 2;

        if (nums[mid] >= target) {
            right = mid;
        } else {
            left = mid + 1;
        }
    }

    return left;
}
```

The answer is:

```text
left
```

possibly:

```text
n
```

if no element satisfies the condition.

---

# 10. Lower Bound Mental Model

Imagine:

```text
nums[i] >= target

false false false true true true
                  ↑
             lower bound
```

Binary search finds:

> **First true**

This is more important than memorizing the code.

---

# 11. Upper Bound

Upper bound means:

> Find the first index `i` such that `nums[i] > target`.

Example:

```text
[1, 2, 4, 4, 4, 7, 9]
```

For:

```text
target = 4
```

upper bound:

```text
index = 5
```

because:

```text
nums[5] = 7 > 4
```

---

# 12. Upper Bound Template

```java
int upperBound(int[] nums, int target) {
    int left = 0;
    int right = nums.length;

    while (left < right) {
        int mid = left + (right - left) / 2;

        if (nums[mid] > target) {
            right = mid;
        } else {
            left = mid + 1;
        }
    }

    return left;
}
```

Again:

```text
answer = first true
```

where the predicate is:

```text
nums[i] > target
```

---

# 13. Lower Bound vs Upper Bound

| Operation | Condition for true |
|---|---|
| Lower bound | `nums[i] >= target` |
| Upper bound | `nums[i] > target` |

For:

```text
[1, 2, 4, 4, 4, 7]
```

and:

```text
target = 4
```

```text
lowerBound = 2
upperBound = 5
```

Therefore the number of occurrences is:

```text
upperBound - lowerBound
```

which gives:

```text
5 - 2 = 3
```

---

# 14. First Occurrence

Find the first position of `target`.

```java
int firstOccurrence(int[] nums, int target) {
    int left = 0;
    int right = nums.length - 1;
    int answer = -1;

    while (left <= right) {
        int mid = left + (right - left) / 2;

        if (nums[mid] == target) {
            answer = mid;
            right = mid - 1;
        } else if (nums[mid] < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }

    return answer;
}
```

Once found:

> Do not stop. Continue searching left.

---

# 15. Last Occurrence

Symmetric to first occurrence.

```java
int lastOccurrence(int[] nums, int target) {
    int left = 0;
    int right = nums.length - 1;
    int answer = -1;

    while (left <= right) {
        int mid = left + (right - left) / 2;

        if (nums[mid] == target) {
            answer = mid;
            left = mid + 1;
        } else if (nums[mid] < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }

    return answer;
}
```

Once found:

> Do not stop. Continue searching right.

---

# 16. Search Insertion Position

Given sorted:

```text
[1, 3, 5, 6]
```

Target:

```text
2
```

Answer:

```text
1
```

because `2` should be inserted before `3`.

This is exactly:

> **Lower bound**

```java
int searchInsert(int[] nums, int target) {
    return lowerBound(nums, target);
}
```

---

# 17. A Powerful Unification

These problems are not separate algorithms.

They are boundary searches.

```text
First occurrence
        ↓
First position equal to target

Lower bound
        ↓
First position >= target

Upper bound
        ↓
First position > target

Last occurrence
        ↓
Last position equal to target
```

Once lower/upper bound becomes intuitive, many binary-search variants become easier.

---

# 18. Java Built-In Binary Search

Java provides:

```java
Arrays.binarySearch(nums, target);
```

But interviewers often want the implementation.

Also, its return value is not simply:

```text
first occurrence
```

when duplicates exist.

For interviews:

> Know how to implement the boundary yourself.

---

# 19. Search Rotated Sorted Array

Example:

```text
[4,5,6,7,0,1,2]
```

This was originally sorted:

```text
[0,1,2,4,5,6,7]
```

but rotated.

Search for:

```text
0
```

in:

```text
O(log n)
```

---

# 20. Rotated Array Insight

At any midpoint, at least one half is sorted.

Example:

```text
[4,5,6,7,0,1,2]
    ↑
```

If:

```text
nums[left] <= nums[mid]
```

then:

```text
[left, mid]
```

is sorted.

Otherwise:

```text
[mid, right]
```

is sorted.

---

# 21. Search Rotated Sorted Array

```java
class Solution {
    public int search(int[] nums, int target) {
        int left = 0;
        int right = nums.length - 1;

        while (left <= right) {
            int mid = left + (right - left) / 2;

            if (nums[mid] == target) {
                return mid;
            }

            // Left half is sorted.
            if (nums[left] <= nums[mid]) {

                if (nums[left] <= target && target < nums[mid]) {
                    right = mid - 1;
                } else {
                    left = mid + 1;
                }

            } else {
                // Right half is sorted.
                if (nums[mid] < target && target <= nums[right]) {
                    left = mid + 1;
                } else {
                    right = mid - 1;
                }
            }
        }

        return -1;
    }
}
```

### Complexity

```text
Time: O(log n)
Space: O(1)
```

This standard version assumes distinct values.

---

# 22. Rotated Array With Duplicates

Duplicates complicate the decision.

Example:

```text
[2,2,2,3,2,2]
```

You can get:

```text
nums[left] == nums[mid] == nums[right]
```

and cannot determine which half is sorted from those values.

A common resolution:

```java
if (nums[left] == nums[mid]
        && nums[mid] == nums[right]) {
    left++;
    right--;
}
```

This can degrade to:

```text
O(n)
```

in the worst case.

Important:

> Distinct elements and duplicate elements are different complexity regimes.

---

# 23. Find Minimum in Rotated Sorted Array

Example:

```text
[4,5,6,7,0,1,2]
```

Answer:

```text
0
```

Use the relationship between:

```text
nums[mid]
nums[right]
```

---

# 24. Minimum in Rotated Array

```java
class Solution {
    public int findMin(int[] nums) {
        int left = 0;
        int right = nums.length - 1;

        while (left < right) {
            int mid = left + (right - left) / 2;

            if (nums[mid] > nums[right]) {
                left = mid + 1;
            } else {
                right = mid;
            }
        }

        return nums[left];
    }
}
```

### Why?

If:

```text
nums[mid] > nums[right]
```

then the minimum must be to the right of `mid`.

Otherwise:

```text
minimum is at mid or left of mid
```

Therefore:

```java
right = mid;
```

not:

```java
right = mid - 1;
```

because `mid` may itself be the minimum.

---

# 25. Peak Element

A peak is an element greater than its neighbors.

Example:

```text
[1,2,3,1]
```

Peak:

```text
3
```

We do not need to find the global maximum.

---

# 26. Peak Binary Search Insight

Compare:

```text
nums[mid]
nums[mid + 1]
```

If:

```text
nums[mid] < nums[mid + 1]
```

we are climbing.

Therefore a peak exists to the right.

```java
left = mid + 1;
```

Otherwise:

```java
right = mid;
```

because `mid` may be a peak.

---

# 27. Peak Element Java

```java
class Solution {
    public int findPeakElement(int[] nums) {
        int left = 0;
        int right = nums.length - 1;

        while (left < right) {
            int mid = left + (right - left) / 2;

            if (nums[mid] < nums[mid + 1]) {
                left = mid + 1;
            } else {
                right = mid;
            }
        }

        return left;
    }
}
```

Time:

```text
O(log n)
```

---

# 28. Mountain Array

A mountain array:

```text
strictly increasing
then
strictly decreasing
```

Example:

```text
[0,3,2,1]
```

The highest point is the mountain peak.

Binary search can find the peak.

Use:

```java
if (arr[mid] < arr[mid + 1])
    left = mid + 1;
else
    right = mid;
```

After finding the peak, search:

```text
ascending side
descending side
```

---

# 29. Search in Mountain Array

The problem usually has three steps:

```text
1. Find peak
2. Binary search ascending portion
3. If not found, binary search descending portion
```

For descending binary search, reverse the comparison.

### Ascending search

```java
if (arr[mid] < target) {
    left = mid + 1;
} else {
    right = mid - 1;
}
```

### Descending search

```java
if (arr[mid] > target) {
    left = mid + 1;
} else {
    right = mid - 1;
}
```

---

# 30. Search a 2D Matrix

Suppose rows are sorted and:

```text
first element of next row > last element of previous row
```

Example:

```text
1  3  5  7
10 11 16 20
23 30 34 60
```

Treat the matrix as a flattened sorted array.

For:

```text
m rows
n columns
```

virtual index:

```text
0 ... m*n - 1
```

Convert:

```java
row = index / n;
col = index % n;
```

---

# 31. 2D Matrix Binary Search

```java
class Solution {
    public boolean searchMatrix(int[][] matrix, int target) {
        int m = matrix.length;
        int n = matrix[0].length;

        int left = 0;
        int right = m * n - 1;

        while (left <= right) {
            int mid = left + (right - left) / 2;

            int row = mid / n;
            int col = mid % n;

            if (matrix[row][col] == target) {
                return true;
            } else if (matrix[row][col] < target) {
                left = mid + 1;
            } else {
                right = mid - 1;
            }
        }

        return false;
    }
}
```

Time:

```text
O(log(mn))
```

Space:

```text
O(1)
```

---

# 32. Important: Two Different "Sorted Matrix" Problems

Do not confuse:

### Matrix I

Entire matrix behaves like one sorted array.

Use:

```text
one binary search
```

### Matrix II

Each row is sorted and each column is sorted, but rows may overlap.

A different strategy is often better:

```text
start at top-right
```

and eliminate one row or column at a time.

The exact matrix structure determines the algorithm.

---

# 33. Binary Search on Answer

This is the most important advanced pattern.

Suppose the answer is an integer:

```text
x
```

and we can write:

```text
can(x)
```

where:

```text
can(x) = whether x is feasible
```

If feasibility is monotonic:

```text
false false false false true true true
                        ↑
                   first feasible
```

binary search can find the optimal answer.

---

# 34. Minimum Feasible Answer

Suppose:

```text
can(x)
```

means:

> Can we complete the task using capacity `x`?

If:

```text
x is feasible
```

then every larger value is also feasible.

Therefore:

```text
false false false true true true
                    ↑
              minimum feasible
```

Find the first `true`.

---

# 35. Maximum Feasible Answer

Sometimes:

```text
can(x)
```

means:

> Can we achieve value `x`?

and if `x` is feasible, all smaller values are feasible.

Then:

```text
true true true true false false
              ↑
        maximum feasible
```

Find the last `true`.

---

# 36. Binary Search on Answer Template

For minimum feasible answer:

```java
long left = low;
long right = high;

while (left < right) {
    long mid = left + (right - left) / 2;

    if (can(mid)) {
        right = mid;
    } else {
        left = mid + 1;
    }
}

return left;
```

For maximum feasible answer:

```java
long left = low;
long right = high;

while (left < right) {
    long mid = left + (right - left + 1) / 2;

    if (can(mid)) {
        left = mid;
    } else {
        right = mid - 1;
    }
}

return left;
```

Notice the upper-mid trick:

```java
(left + right + 1) / 2
```

It prevents an infinite loop when searching for the last feasible value.

---

# 37. The Three Questions for Answer Binary Search

Whenever you suspect binary search on answer, ask:

### 1. What is the answer range?

Example:

```text
minimum = 1
maximum = sum(weights)
```

### 2. Can I check a candidate answer?

Write:

```text
can(x)
```

### 3. Is feasibility monotonic?

Example:

```text
capacity 10 → impossible
capacity 11 → impossible
capacity 12 → possible
capacity 13 → possible
...
```

If yes:

```text
binary search
```

---

# 38. Capacity Problems

A classic example:

> Ship packages within D days.

Each package has a weight.

Choose the minimum ship capacity that can transport everything within `D` days.

The answer lies between:

```text
max(weights)
```

and:

```text
sum(weights)
```

---

# 39. Shipping Capacity Feasibility

For a candidate capacity `C`, greedily fill each day until adding another package would exceed `C`.

Count required days.

If:

```text
days <= D
```

then capacity `C` is feasible.

### Java

```java
boolean canShip(int[] weights, int days, int capacity) {
    int usedDays = 1;
    int current = 0;

    for (int w : weights) {
        if (current + w > capacity) {
            usedDays++;
            current = 0;
        }

        current += w;

        if (usedDays > days) {
            return false;
        }
    }

    return true;
}
```

Then binary search:

```java
int left = 0;
int right = 0;

for (int w : weights) {
    left = Math.max(left, w);
    right += w;
}

while (left < right) {
    int mid = left + (right - left) / 2;

    if (canShip(weights, days, mid)) {
        right = mid;
    } else {
        left = mid + 1;
    }
}

return left;
```

---

# 40. Why Greedy Works in Capacity Problems

For a fixed capacity, placing packages in order and filling the current group as much as possible minimizes the number of groups needed.

If the greedy check requires:

```text
D or fewer days
```

then a more complicated arrangement cannot require fewer than the greedy minimum under the same sequential-order constraint.

The binary search only decides which capacity is sufficient.

This separation is important:

```text
Binary search → candidate answer
Greedy → feasibility check
```

---

# 41. Allocation Problems

Typical problem:

> Allocate books among `k` students so that the maximum pages assigned to one student is minimized.

Search the answer:

```text
maximum pages per student
```

Lower bound:

```text
max(single book)
```

Upper bound:

```text
sum(all books)
```

For candidate `X`:

```text
Can we allocate books using at most k students,
with each student receiving at most X pages?
```

If yes:

```text
try smaller X
```

If no:

```text
increase X
```

---

# 42. Allocation Feasibility Template

```java
boolean feasible(int[] pages, int k, long limit) {
    int students = 1;
    long current = 0;

    for (int p : pages) {
        if (current + p > limit) {
            students++;
            current = p;
        } else {
            current += p;
        }

        if (students > k) {
            return false;
        }
    }

    return true;
}
```

Then search:

```text
minimum feasible limit
```

---

# 43. Partition / Split Array Problems

Another common form:

> Split an array into `k` contiguous subarrays while minimizing the largest subarray sum.

Answer range:

```text
max(nums)
```

to:

```text
sum(nums)
```

Feasibility:

```text
How many groups are required if each group has sum <= X?
```

If required groups:

```text
<= k
```

then `X` is feasible.

This pattern appears repeatedly in:

- Book allocation
- Painter partition
- Split Array Largest Sum
- Workload balancing
- Capacity planning

---

# 44. Aggressive Placement Problems

Classic form:

> Place `k` objects in positions so that the minimum distance between any two objects is maximized.

Examples:

- Aggressive Cows
- Router placement
- Stall placement
- Wi-Fi access-point placement

Sort positions first.

Then binary search the answer:

```text
minimum allowed distance
```

---

# 45. Aggressive Placement Feasibility

Given candidate distance:

```text
d
```

greedily place an object at the earliest possible position.

Track:

```text
lastPlaced
count
```

Whenever:

```text
position - lastPlaced >= d
```

place another object.

If:

```text
count >= k
```

then `d` is feasible.

---

# 46. Aggressive Placement Java

```java
boolean canPlace(int[] positions, int k, int distance) {
    int count = 1;
    int last = positions[0];

    for (int i = 1; i < positions.length; i++) {
        if (positions[i] - last >= distance) {
            count++;
            last = positions[i];

            if (count >= k) {
                return true;
            }
        }
    }

    return false;
}
```

Binary search for:

```text
maximum feasible distance
```

---

# 47. Why Sort First?

Placement feasibility depends on distances between positions.

Sorting gives:

```text
p0 < p1 < p2 < ...
```

Then greedy placement can always choose the earliest feasible position.

This creates the maximum remaining space for future placements.

---

# 48. Scheduling Problems

Binary search on answer frequently appears in scheduling.

Examples:

- Minimum time to complete jobs
- Minimum machine capacity
- Maximum minimum workload
- Allocate tasks among workers
- Production rate
- Eating speed
- Processing speed

The generic structure:

```text
candidate rate/capacity/time
          ↓
     feasibility check
          ↓
       true/false
          ↓
    monotonic predicate
          ↓
     binary search
```

---

# 49. Koko Eating Bananas

Given piles and `h` hours, find minimum eating speed.

Candidate speed:

```text
k
```

Hours required:

```text
ceil(pile / k)
```

Total hours:

```text
sum(ceil(pile / k))
```

If:

```text
hours <= h
```

then speed is feasible.

Search:

```text
1 ... max(pile)
```

for the minimum feasible speed.

---

# 50. Avoid Floating Point in Ceiling Division

Instead of:

```java
Math.ceil((double) pile / speed)
```

use:

```java
(pile + speed - 1) / speed
```

For larger values, use `long`:

```java
(pile + speed - 1L) / speed
```

This avoids precision problems.

---

# 51. K-th Answer Problems

Some problems ask for:

```text
k-th smallest
k-th largest
k-th value satisfying a condition
```

Binary search can sometimes operate on the **value range**, not indices.

The key is to count:

```text
how many elements/objects are <= X?
```

Call:

```text
count(X)
```

If:

```text
count(X) >= k
```

then `X` may be large enough to contain the k-th answer.

This gives:

```text
false false false true true
                  ↑
              first X
```

---

# 52. K-th Smallest Number in a Multiplication Table

For a candidate `x`, count entries `<= x`.

For row `i`:

```text
i, 2i, 3i, ...
```

Number of values `<= x` is:

```text
min(n, x / i)
```

Then:

```text
count = Σ min(n, x / i)
```

Binary search the smallest `x` with:

```text
count >= k
```

This is a classic value-space binary search.

---

# 53. K-th Pair Distance

Another advanced form:

> Find the k-th smallest absolute difference among all pairs.

Do not enumerate all pairs.

Sort the array.

For candidate distance `d`, count how many pairs have:

```text
difference <= d
```

This count can be computed using two pointers.

Then binary search the smallest `d` where:

```text
count >= k
```

This is a major interview pattern:

```text
Binary search on answer
+
two-pointer feasibility/counting
```

---

# 54. Binary Search on Answer: Common Structure

```java
long low = minimumPossibleAnswer();
long high = maximumPossibleAnswer();

while (low < high) {
    long mid = low + (high - low) / 2;

    if (feasible(mid)) {
        high = mid;
    } else {
        low = mid + 1;
    }
}

return low;
```

For maximum feasible:

```java
long low = minimumPossibleAnswer();
long high = maximumPossibleAnswer();

while (low < high) {
    long mid = low + (high - low + 1) / 2;

    if (feasible(mid)) {
        low = mid;
    } else {
        high = mid - 1;
    }
}

return low;
```

---

# 55. How to Find the Answer Range

This is one of the most important skills.

## Capacity

```text
low = maximum individual workload
high = total workload
```

## Eating speed

```text
low = 1
high = maximum pile
```

## Minimum distance

```text
low = 0 or minimum possible distance
high = maxPosition - minPosition
```

## Maximum minimum value

Often:

```text
low = minimum possible value
high = maximum possible value
```

## K-th value

Often:

```text
low = minimum element/value
high = maximum element/value
```

Do not arbitrarily use:

```text
0 ... 1e9
```

when tighter bounds are available.

---

# 56. Monotonicity Is the Real Requirement

Binary search does not fundamentally require sorted data.

It requires a monotonic predicate.

Example:

```text
can(x)

x = 1 → false
x = 2 → false
x = 3 → false
x = 4 → true
x = 5 → true
x = 6 → true
```

This is binary-searchable.

But:

```text
false true false true true false
```

is not monotonic and cannot be searched this way.

---

# 57. First True vs Last True

Most answer-based problems are one of two forms.

## Minimum feasible

```text
false false false true true true
                  ↑
              first true
```

Use:

```java
if (feasible(mid)) {
    right = mid;
} else {
    left = mid + 1;
}
```

## Maximum feasible

```text
true true true false false
          ↑
       last true
```

Use:

```java
if (feasible(mid)) {
    left = mid;
} else {
    right = mid - 1;
}
```

For the second form, use upper mid:

```java
mid = left + (right - left + 1) / 2;
```

---

# 58. Binary Search With Long

Use `long` when:

- Sum can exceed `int`
- Product can exceed `int`
- Answer range is large
- Counting can become large
- `high` is based on an array sum

Example:

```java
long sum = 0;

for (int x : nums) {
    sum += x;
}
```

And:

```java
long mid = low + (high - low) / 2;
```

This avoids overflow.

---

# 59. Integer Overflow

Bad:

```java
int mid = (left + right) / 2;
```

Better:

```java
int mid = left + (right - left) / 2;
```

For arithmetic involving large values:

```java
long mid = left + (right - left) / 2;
```

Also watch:

```java
a * b
```

because multiplication may overflow before assignment to `long`.

Use:

```java
1L * a * b
```

when needed.

---

# 60. Binary Search Debugging

When your binary search gets stuck, inspect:

### 1. Interval convention

Are you using:

```text
[left, right]
```

or:

```text
[left, right)
```

?

### 2. Loop condition

Inclusive:

```java
while (left <= right)
```

Half-open:

```java
while (left < right)
```

### 3. Mid update

Are you accidentally doing:

```java
left = mid;
```

when `mid == left`?

### 4. Is the answer preserved?

If `mid` can be the answer, do not discard it.

Example:

```java
right = mid;
```

rather than:

```java
right = mid - 1;
```

### 5. Does the search shrink?

Every iteration must strictly reduce the search interval.

---

# 61. Common Binary Search Bugs

## Bug 1 — Infinite loop

```java
mid = left + (right - left) / 2;
left = mid;
```

If:

```text
left + 1 = right
```

then:

```text
mid = left
```

and nothing changes.

Use:

```java
left = mid + 1;
```

or upper mid for last-true searches.

---

## Bug 2 — Losing a possible answer

Suppose:

```text
mid
```

may itself be the minimum.

Do:

```java
right = mid;
```

not:

```java
right = mid - 1;
```

---

## Bug 3 — Wrong feasibility direction

For minimum answer:

```text
feasible → go left
not feasible → go right
```

For maximum answer:

```text
feasible → go right
not feasible → go left
```

---

## Bug 4 — Bad bounds

If:

```text
low
```

is impossible, the binary search may return an invalid result.

Bounds must represent the actual answer space.

---

## Bug 5 — Incorrect greedy check

Binary search cannot fix a wrong `feasible(x)`.

First make the feasibility check correct.

Then prove monotonicity.

Then binary search.

---

# 62. Binary Search Proof Strategy

For interviews, be able to explain three things:

## 1. Search space

Example:

```text
answer ∈ [max(weights), sum(weights)]
```

## 2. Feasibility

Example:

```text
can(capacity)
```

returns whether all packages can be shipped within `D` days.

## 3. Monotonicity

If capacity `C` works:

```text
C + 1
C + 2
...
```

also work.

Therefore:

```text
false false false true true true
```

and binary search is valid.

---

# 63. Binary Search Decision Tree

When you encounter a new problem:

```text
Is the input sorted?
       │
       ├── Yes
       │    ↓
       │  Exact search?
       │    ↓
       │  Boundary?
       │    ↓
       │  Rotated?
       │    ↓
       │  Peak?
       │
       └── No
            ↓
       Is there an answer range?
            ↓
       Can I test a candidate answer?
            ↓
       Is feasibility monotonic?
            ↓
          YES
            ↓
      Binary search answer
```

---

# 64. LeetCode Practice Roadmap

## Easy

### 1. Binary Search
- Pattern: Basic binary search
- Priority: Essential
- Core lesson: Search invariant

### 2. Search Insert Position
- Pattern: Lower bound
- Priority: Essential
- Core lesson: First position `>= target`

### 3. First Bad Version
- Pattern: First true
- Priority: Essential
- Core lesson: Binary search on monotonic predicate

### 4. Sqrt(x)
- Pattern: Binary search on answer
- Priority: High
- Core lesson: Maximum feasible integer

### 5. Valid Perfect Square
- Pattern: Binary search
- Priority: High
- Core lesson: Search numeric answer space

---

# 65. Medium

### 6. Find First and Last Position of Element in Sorted Array
- Pattern: Lower/upper boundary
- Priority: Essential
- Core lesson: First and last occurrence

### 7. Search in Rotated Sorted Array
- Pattern: Rotated binary search
- Priority: Essential
- Core lesson: One half is sorted

### 8. Find Minimum in Rotated Sorted Array
- Pattern: Boundary binary search
- Priority: Essential
- Core lesson: Preserve possible minimum

### 9. Find Peak Element
- Pattern: Binary search on slope
- Priority: Very High
- Core lesson: Increasing/decreasing direction

### 10. Search a 2D Matrix
- Pattern: Flattened binary search
- Priority: High
- Core lesson: Map 1D index to 2D

### 11. Koko Eating Bananas
- Pattern: Binary search on answer
- Priority: Essential
- Core lesson: Minimum feasible speed

### 12. Capacity To Ship Packages Within D Days
- Pattern: Binary search + greedy
- Priority: Essential
- Core lesson: Minimum feasible capacity

### 13. Split Array Largest Sum
- Pattern: Binary search + greedy
- Priority: Very High
- Core lesson: Partition feasibility

### 14. Find K Closest Elements
- Pattern: Binary search + window
- Priority: High
- Core lesson: Search the optimal window boundary

### 15. Time Based Key-Value Store
- Pattern: Binary search
- Priority: High
- Core lesson: Previous timestamp / upper-bound reasoning

### 16. Magnetic Force Between Two Balls
- Pattern: Aggressive placement
- Priority: Very High
- Core lesson: Maximum feasible minimum distance

---

# 66. Hard / Advanced

### 17. Median of Two Sorted Arrays
- Pattern: Binary search partition
- Priority: Essential
- Core lesson: Search partition rather than value

### 18. K-th Smallest Pair Distance
- Pattern: Binary search + counting
- Priority: Very High
- Core lesson: Count pairs <= candidate

### 19. K-th Smallest Number in Multiplication Table
- Pattern: Value-space binary search
- Priority: High
- Core lesson: Count values <= candidate

### 20. Minimum Number of Days to Make m Bouquets
- Pattern: Binary search + feasibility
- Priority: Very High
- Core lesson: Time as answer

### 21. Minimum Speed to Arrive on Time
- Pattern: Binary search on speed
- Priority: High
- Core lesson: Numeric feasibility

### 22. Minimum Time to Complete Trips
- Pattern: Binary search on time
- Priority: High
- Core lesson: Capacity accumulation

### 23. Minimum Limit of Balls in a Bag
- Pattern: Binary search on answer
- Priority: High
- Core lesson: Minimum feasible limit

---

# 67. Advanced Pattern Grouping

Do not solve these as isolated problems.

## Group A — Boundary search

- Lower bound
- Upper bound
- First occurrence
- Last occurrence
- Search insertion position

## Group B — Rotated arrays

- Search rotated array
- Minimum in rotated array
- Rotated array with duplicates

## Group C — Shape/slope

- Peak element
- Mountain array
- Bitonic arrays

## Group D — Capacity

- Ship packages
- Split array
- Book allocation
- Painter partition

## Group E — Rate/time

- Koko Eating Bananas
- Minimum speed
- Minimum time to complete trips

## Group F — Placement

- Aggressive Cows
- Magnetic Force
- Router placement

## Group G — K-th answer

- K-th pair distance
- K-th multiplication-table value
- K-th smallest in structured search space

---

# 68. Capacity Problems: Universal Template

```java
long low = maxElement(nums);
long high = sum(nums);

while (low < high) {
    long mid = low + (high - low) / 2;

    if (feasible(nums, mid)) {
        high = mid;
    } else {
        low = mid + 1;
    }
}

return low;
```

Feasibility:

```java
boolean feasible(int[] nums, long limit) {
    int groups = 1;
    long current = 0;

    for (int x : nums) {
        if (current + x > limit) {
            groups++;
            current = x;
        } else {
            current += x;
        }
    }

    return groups <= allowedGroups;
}
```

---

# 69. Placement Problems: Universal Template

```java
Arrays.sort(positions);

int low = 0;
int high = positions[positions.length - 1]
         - positions[0];

while (low < high) {
    int mid = low + (high - low + 1) / 2;

    if (canPlace(positions, k, mid)) {
        low = mid;
    } else {
        high = mid - 1;
    }
}

return low;
```

The answer is:

```text
maximum feasible minimum distance
```

---

# 70. Rate Problems: Universal Template

```java
long low = 1;
long high = maxValue;

while (low < high) {
    long mid = low + (high - low) / 2;

    if (canFinish(mid)) {
        high = mid;
    } else {
        low = mid + 1;
    }
}

return low;
```

Use when:

```text
higher rate → easier/faster
```

Therefore:

```text
false false false true true true
```

---

# 71. K-th Value: Universal Template

```java
long low = minimumValue;
long high = maximumValue;

while (low < high) {
    long mid = low + (high - low) / 2;

    if (countLessOrEqual(mid) >= k) {
        high = mid;
    } else {
        low = mid + 1;
    }
}

return low;
```

The key is the counting function:

```text
countLessOrEqual(x)
```

---

# 72. Binary Search vs Two Pointers

They can appear together.

Example:

> K-th smallest pair distance.

Outer layer:

```text
Binary search distance
```

Inner layer:

```text
Two pointers count pairs <= distance
```

This produces:

```text
O(n log W)
```

where `W` is the answer range.

This combination is common in advanced interviews.

---

# 73. Binary Search vs Heap

For k-th problems, ask:

### Can I directly maintain k candidates?

Maybe:

```text
Heap
```

### Can I efficiently count how many values are <= X?

Maybe:

```text
Binary search on value
```

Binary search is often preferable when:

```text
count(X)
```

is efficient and monotonic.

---

# 74. Binary Search vs Sorting

Sorting can cost:

```text
O(n log n)
```

Binary search costs:

```text
O(log n)
```

but only after the required ordering exists.

Do not claim binary search is always faster.

For a one-time unsorted search:

```text
linear search = O(n)
```

may be better than:

```text
sort + binary search = O(n log n)
```

unless multiple queries justify sorting.

---

# 75. Binary Search Interview Checklist

Before coding:

```text
1. What exactly am I searching?
2. Is it an index or an answer value?
3. What is the search interval?
4. What does mid represent?
5. What condition determines which half survives?
6. Is the predicate monotonic?
7. Am I finding first true or last true?
8. Can mid itself be the answer?
9. Could arithmetic overflow?
10. What are the edge cases?
```

---

# 76. Edge Cases

Always test:

### Empty array

```text
[]
```

### One element

```text
[5]
```

### Target at beginning

```text
[1,2,3]
target = 1
```

### Target at end

```text
target = 3
```

### Target absent

```text
target = 4
```

### All duplicates

```text
[2,2,2,2]
```

### Rotated at boundary

```text
[1,2,3,4,5]
```

### Already rotated minimally

```text
[5,1,2,3,4]
```

### Answer at lower bound

### Answer at upper bound

### Very large sums

Use:

```java
long
```

when necessary.

---

# 77. GATE / CS Theory Focus

For GATE-style preparation, know:

- Binary search algorithm
- Best/average/worst-case complexity
- Recurrence for binary search
- Comparison-based searching
- Sorted-array search
- First/last occurrence
- Lower/upper bound
- Search insertion position
- Binary search invariants
- Rotated sorted arrays
- Peak finding
- Binary search on monotonic predicates
- Amortized/complexity reasoning
- Integer overflow
- Iterative vs recursive binary search
- Search in matrices
- Parametric search / answer-space search

---

# 78. Three GATE-Style Solved Questions

## Question 1 — Binary Search Comparisons

A sorted array contains `n` distinct elements. Binary search is used to search for an element.

What is the worst-case asymptotic number of comparisons?

A. `O(1)`  
B. `O(log n)`  
C. `O(n)`  
D. `O(n log n)`

### Solution

Each comparison approximately halves the remaining search space:

```text
n
n/2
n/4
n/8
...
```

After `k` steps:

```text
n / 2^k ≈ 1
```

Therefore:

```text
2^k ≈ n
k ≈ log2(n)
```

### Answer

**B — O(log n)**

---

## Question 2 — Lower Bound

Consider:

```text
A = [1, 2, 2, 2, 5, 7]
```

What is the lower bound of `2`?

### Definition

Lower bound is the first index satisfying:

```text
A[i] >= 2
```

Check:

```text
A[0] = 1 < 2
A[1] = 2 >= 2
```

Therefore:

```text
lowerBound = 1
```

### Answer

```text
1
```

---

## Question 3 — Monotonic Feasibility

Suppose a problem has a candidate answer `x` with:

```text
feasible(10) = false
feasible(11) = false
feasible(12) = true
feasible(13) = true
feasible(14) = true
```

What should binary search find if the objective is to minimize the feasible answer?

### Solution

The predicate is:

```text
false false true true true
```

Therefore the answer is the:

```text
first true
```

which is:

```text
12
```

### Answer

```text
12
```

### Core concept

Binary search on answer requires a monotonic predicate.

---

# 79. Serious Mastery Problem Set

For strong FAANG/top-product preparation, do not stop after one solution.

For every major pattern, solve variants.

## Boundary

- Binary Search
- Search Insert Position
- First/Last Position
- First Bad Version
- Find Right Interval
- Time Based Key-Value Store

## Rotated

- Search Rotated Sorted Array
- Search Rotated Sorted Array II
- Find Minimum in Rotated Sorted Array
- Find Minimum II
- Rotation count

## Peak

- Find Peak Element
- Peak Index in a Mountain Array
- Find in Mountain Array

## Capacity

- Koko Eating Bananas
- Ship Packages
- Split Array Largest Sum
- Minimum Days to Make Bouquets
- Minimum Time to Complete Trips

## Placement

- Aggressive Cows
- Magnetic Force Between Two Balls
- Router placement
- Maximize minimum distance

## K-th

- K-th Smallest Pair Distance
- K-th Smallest Number in Multiplication Table
- K-th Smallest in Sorted Matrix
- K-th value using monotonic counting

---

# 80. Mastery Progression

## Level 1 — Basic

You should be able to write:

```text
exact binary search
lower bound
upper bound
first occurrence
last occurrence
```

without looking at a template.

## Level 2 — Structural variations

You should recognize:

```text
rotated array
peak
mountain
2D sorted matrix
```

within seconds.

## Level 3 — Answer search

You should be able to derive:

```text
answer range
feasibility function
monotonicity
first/last feasible
```

from a new problem.

## Level 4 — Advanced combinations

You should solve:

```text
binary search + greedy
binary search + two pointers
binary search + counting
binary search + prefix sums
```

without confusing which layer does what.

---

# 81. Final Pattern Recognition Cheat Sheet

| Signal | Pattern |
|---|---|
| Sorted array | Binary search |
| Find target | Exact binary search |
| First `>= x` | Lower bound |
| First `> x` | Upper bound |
| First occurrence | Lower-bound style |
| Last occurrence | Upper-bound style |
| Insert position | Lower bound |
| Rotated sorted | Find sorted half |
| Minimum rotated array | Compare `mid` with `right` |
| Peak | Compare `mid` with `mid + 1` |
| Mountain | Find peak, search both sides |
| Sorted matrix | Flatten or matrix-specific search |
| Minimum feasible value | First true |
| Maximum feasible value | Last true |
| Capacity | Binary search + greedy |
| Allocation | Binary search + greedy |
| Scheduling rate/time | Binary search + feasibility |
| Maximum minimum distance | Binary search + greedy placement |
| K-th value | Binary search + counting |

---

# 82. Final Binary Search Formula Sheet

## Exact search

```java
while (left <= right) {
    int mid = left + (right - left) / 2;

    if (a[mid] == target) return mid;

    if (a[mid] < target)
        left = mid + 1;
    else
        right = mid - 1;
}
```

## Lower bound

```java
while (left < right) {
    int mid = left + (right - left) / 2;

    if (a[mid] >= target)
        right = mid;
    else
        left = mid + 1;
}
```

## Upper bound

```java
while (left < right) {
    int mid = left + (right - left) / 2;

    if (a[mid] > target)
        right = mid;
    else
        left = mid + 1;
}
```

## Minimum feasible

```java
while (left < right) {
    long mid = left + (right - left) / 2;

    if (feasible(mid))
        right = mid;
    else
        left = mid + 1;
}
```

## Maximum feasible

```java
while (left < right) {
    long mid = left + (right - left + 1) / 2;

    if (feasible(mid))
        left = mid;
    else
        right = mid - 1;
}
```

---

# 83. One-Page Revision Sheet

```text
BINARY SEARCH
│
├── Basic
│   ├── Exact search
│   ├── Lower bound
│   ├── Upper bound
│   ├── First occurrence
│   ├── Last occurrence
│   └── Insertion position
│
├── Structural Variants
│   ├── Rotated sorted array
│   ├── Minimum rotated array
│   ├── Peak
│   ├── Mountain
│   └── Sorted 2D matrix
│
└── Binary Search on Answer
    │
    ├── Minimum feasible
    │   └── First true
    │
    ├── Maximum feasible
    │   └── Last true
    │
    ├── Capacity
    │   ├── Shipping
    │   ├── Allocation
    │   └── Partition
    │
    ├── Rate / Time
    │   ├── Eating speed
    │   ├── Production
    │   └── Scheduling
    │
    ├── Placement
    │   ├── Aggressive placement
    │   └── Maximum minimum distance
    │
    └── K-th
        ├── Value-space search
        └── Count <= X
```

## The core mental model

```text
Don't ask only:

"Is the array sorted?"

Ask:

"Can I define a monotonic predicate?"
```

Then:

```text
Candidate answer
      ↓
feasible(candidate)
      ↓
false false false true true true
                  ↓
             binary search
```

## The most important problems

```text
1. Binary Search
2. Search Insert Position
3. First and Last Position
4. Search in Rotated Sorted Array
5. Find Minimum in Rotated Sorted Array
6. Find Peak Element
7. Search a 2D Matrix
8. Koko Eating Bananas
9. Capacity To Ship Packages Within D Days
10. Split Array Largest Sum
11. Magnetic Force Between Two Balls
12. Minimum Days to Make m Bouquets
13. K-th Smallest Pair Distance
14. Median of Two Sorted Arrays
```

## Final rule

> **Binary search is a boundary-finding technique. The boundary may be an index in a sorted array, or an optimal value in an answer space.**
