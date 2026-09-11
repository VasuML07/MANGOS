# Arrays

## 1. Why This Topic Matters

Arrays are one of the highest-frequency data structures in coding interviews.

The important part is not learning how to loop over an array. It is learning how to transform an apparently brute-force array problem into a linear or near-linear solution using reusable patterns:

- prefix/suffix preprocessing,
- two pointers,
- sliding windows,
- hashing,
- sorting + scanning,
- in-place modification,
- Kadane's algorithm,
- difference arrays,
- merge techniques,
- frequency counting.

Array problems are also the foundation for many later topics:

- strings,
- hashing,
- stacks and queues,
- binary search,
- greedy algorithms,
- dynamic programming,
- intervals,
- matrices,
- bitmasking.

For interview preparation, arrays should become automatic. When you see an array problem, you should quickly classify whether it is primarily:

```text
Traversal
Prefix/Suffix
Two Pointers
Sliding Window
Hashing
Sorting + Scanning
Greedy
Kadane / DP
Intervals
In-place Rearrangement
```

---

# 2. Prerequisites

You should know:

- Java arrays,
- loops,
- conditionals,
- functions,
- basic sorting,
- `HashMap`,
- `HashSet`,
- basic time and space complexity.

You should understand the difference between:

- subarray,
- subsequence,
- subset,
- interval.

---

# 3. Core Array Concepts

## 3.1 Array Traversal

The simplest operation is scanning every element.

```java
for (int i = 0; i < nums.length; i++) {
    // use nums[i]
}
```

Enhanced loop:

```java
for (int x : nums) {
    // use x
}
```

Use indexed traversal when you need:

- neighboring elements,
- indices,
- modification,
- two pointers.

Use enhanced loops when you only need values.

### Complexity

```text
Time:  O(n)
Space: O(1)
```

---

# 4. In-Place Modification

An in-place algorithm modifies the original array using O(1) auxiliary space, ignoring the output when the problem explicitly allows it.

Example:

```java
static void reverse(int[] nums) {
    int left = 0;
    int right = nums.length - 1;

    while (left < right) {
        int temp = nums[left];
        nums[left] = nums[right];
        nums[right] = temp;

        left++;
        right--;
    }
}
```

This is important because interviewers frequently ask for:

```text
O(n) time
O(1) extra space
```

### Typical in-place patterns

- swapping,
- two pointers,
- reverse,
- partitioning,
- cyclic placement,
- array rotation,
- removing duplicates,
- moving zeroes.

### Important distinction

The following:

```java
int[] result = new int[n];
```

uses O(n) auxiliary/output memory.

The following:

```java
int left = 0;
int right = n - 1;
```

uses O(1) auxiliary memory.

---

# 5. Prefix Sum

## 5.1 Core Idea

A prefix sum stores cumulative sums.

For:

```text
nums = [2, 4, 1, 3]
```

prefix sums can be:

```text
[2, 6, 7, 10]
```

A safer interview convention is often to use a prefix array of length `n + 1`:

```text
nums:   2   4   1   3
prefix: 0   2   6   7   10
        ^
        empty prefix
```

Then the sum of `nums[l..r]` is:

```java
prefix[r + 1] - prefix[l]
```

### Java Template

```java
static long[] buildPrefixSum(int[] nums) {
    long[] prefix = new long[nums.length + 1];

    for (int i = 0; i < nums.length; i++) {
        prefix[i + 1] = prefix[i] + nums[i];
    }

    return prefix;
}
```

### Range Sum

```java
static long rangeSum(long[] prefix, int left, int right) {
    return prefix[right + 1] - prefix[left];
}
```

### Complexity

Preprocessing:

```text
Time:  O(n)
Space: O(n)
```

Each range query:

```text
Time: O(1)
```

---

# 6. Why Prefix Sum Works

For:

```text
nums = [2, 4, 1, 3]
```

prefix:

```text
[0, 2, 6, 7, 10]
```

Suppose we want:

```text
nums[1..3]
```

which is:

```text
4 + 1 + 3 = 8
```

Use:

```text
prefix[4] - prefix[1]
= 10 - 2
= 8
```

The part before `left` cancels.

This is the central prefix-sum idea:

> Store cumulative information so that a range query becomes subtraction of two prefix states.

---

# 7. Prefix Sum + HashMap

This is one of the most important array interview patterns.

Suppose we want the number of subarrays whose sum equals `k`.

Let:

```text
prefix[i] = sum of elements before index i
```

For a subarray `(j ... i]` to have sum `k`:

```text
prefix[i] - prefix[j] = k
```

Therefore:

```text
prefix[j] = prefix[i] - k
```

So while scanning:

1. compute current prefix sum,
2. check how many times `prefix - k` has appeared,
3. add that count,
4. record current prefix.

### Java Template

```java
static int subarraySum(int[] nums, int k) {
    Map<Integer, Integer> frequency = new HashMap<>();
    frequency.put(0, 1);

    int prefix = 0;
    int count = 0;

    for (int x : nums) {
        prefix += x;

        count += frequency.getOrDefault(prefix - k, 0);

        frequency.put(prefix,
                frequency.getOrDefault(prefix, 0) + 1);
    }

    return count;
}
```

### Why initialize `0 -> 1`?

It represents an empty prefix.

If:

```text
prefix == k
```

then:

```text
prefix - k = 0
```

and the subarray starts at index `0`.

### Complexity

```text
Time:  O(n)
Space: O(n)
```

---

# 8. Prefix Sum: When to Use It

Use prefix sums when:

- many range sums are queried,
- a subarray sum is important,
- the array is static or changes rarely,
- a condition can be expressed using differences between prefix states.

Strong signals:

```text
sum from L to R
sum of every subarray
number of subarrays with sum K
subarray sum equals 0
longest subarray with a target sum
```

### When NOT to use it

Do not build a prefix array just to calculate one total sum.

A simple loop is enough.

---

# 9. Suffix Sum

A suffix sum stores information from the right side.

For:

```text
nums = [2, 4, 1, 3]
```

a suffix representation can be:

```text
[10, 8, 4, 3]
```

Meaning:

```text
suffix[i] = nums[i] + nums[i+1] + ... + nums[n-1]
```

### Java Template

```java
static long[] buildSuffixSum(int[] nums) {
    int n = nums.length;
    long[] suffix = new long[n];

    if (n == 0) {
        return suffix;
    }

    suffix[n - 1] = nums[n - 1];

    for (int i = n - 2; i >= 0; i--) {
        suffix[i] = nums[i] + suffix[i + 1];
    }

    return suffix;
}
```

### Common Use

Suffix information is useful when each position needs:

```text
information about everything to its right
```

Examples:

- product except self,
- right-side maximum,
- right-side minimum,
- partition calculations,
- trapping rain water variants.

---

# 10. Prefix + Suffix Combination

Some problems require information from both sides.

Example:

```text
Product of Array Except Self
```

For each position:

```text
answer[i] =
product of elements left of i
*
product of elements right of i
```

Conceptually:

```text
prefix product
        *
suffix product
```

The standard O(1)-extra-space solution stores the left product in the output array and accumulates the right product during a reverse pass.

---

# 11. Difference Arrays

A difference array is the reverse idea of prefix sums.

Prefix sums turn:

```text
individual values -> cumulative values
```

Difference arrays turn:

```text
range updates -> endpoint changes
```

Suppose:

```text
nums = [0, 0, 0, 0, 0]
```

and we want to add `5` to indices `[1, 3]`.

Instead of updating:

```text
1, 2, 3
```

we can record:

```text
diff[1] += 5
diff[4] -= 5
```

Then reconstruct using a running sum.

Conceptually:

```text
diff:
[0, 5, 0, 0, -5]

prefix:
[0, 5, 5, 5, 0]
```

The final array is:

```text
[0, 5, 5, 5, 0]
```

---

# 12. Difference Array Template

For range addition `[left, right]`:

```java
diff[left] += value;

if (right + 1 < diff.length) {
    diff[right + 1] -= value;
}
```

After all updates:

```java
int running = 0;

for (int i = 0; i < n; i++) {
    running += diff[i];
    nums[i] += running;
}
```

### Complexity

If there are `m` range updates:

```text
Naive:
O(m * n) in the worst case

Difference array:
O(m + n)
```

### Recognition Signal

If a problem says:

> Apply many operations that add/subtract a value over a contiguous range.

Think:

```text
difference array
```

---

# 13. Kadane's Algorithm

Kadane's algorithm finds the maximum sum of a contiguous subarray in O(n).

Example:

```text
[-2, 1, -3, 4, -1, 2, 1, -5, 4]
```

Answer:

```text
6
```

from:

```text
[4, -1, 2, 1]
```

---

# 14. Kadane's Core Idea

At every index, ask:

> Is it better to extend the previous subarray or start a new subarray here?

Let:

```text
current = best subarray sum ending at this index
best    = best sum seen anywhere
```

Transition:

```text
current = max(nums[i], current + nums[i])
best = max(best, current)
```

### Java Template

```java
static int maxSubarraySum(int[] nums) {
    int current = nums[0];
    int best = nums[0];

    for (int i = 1; i < nums.length; i++) {
        current = Math.max(nums[i], current + nums[i]);
        best = Math.max(best, current);
    }

    return best;
}
```

### Complexity

```text
Time:  O(n)
Space: O(1)
```

---

# 15. Why Kadane Works

Suppose:

```text
current = -5
next = 7
```

Continuing gives:

```text
-5 + 7 = 2
```

Starting fresh gives:

```text
7
```

So we choose:

```text
7
```

A negative prefix cannot improve the sum of a future positive subarray.

That is the key invariant.

---

# 16. Kadane With Indices

Sometimes the problem asks for the actual subarray, not just its sum.

```java
static int[] maxSubarray(int[] nums) {
    int current = nums[0];
    int best = nums[0];

    int currentStart = 0;
    int bestStart = 0;
    int bestEnd = 0;

    for (int i = 1; i < nums.length; i++) {
        if (nums[i] > current + nums[i]) {
            current = nums[i];
            currentStart = i;
        } else {
            current += nums[i];
        }

        if (current > best) {
            best = current;
            bestStart = currentStart;
            bestEnd = i;
        }
    }

    return new int[]{bestStart, bestEnd, best};
}
```

### Important Edge Case

Do not initialize:

```java
current = 0;
best = 0;
```

if the problem requires a non-empty subarray.

For:

```text
[-5, -2, -8]
```

the answer should be:

```text
-2
```

not:

```text
0
```

---

# 17. Maximum Product Subarray

Kadane's idea can be adapted, but multiplication introduces a new issue:

```text
negative * negative = positive
```

Therefore at every index we need both:

```text
max ending here
min ending here
```

### Java Template

```java
static int maxProduct(int[] nums) {
    int maxEnding = nums[0];
    int minEnding = nums[0];
    int answer = nums[0];

    for (int i = 1; i < nums.length; i++) {
        int x = nums[i];

        int oldMax = maxEnding;
        int oldMin = minEnding;

        maxEnding = Math.max(
                x,
                Math.max(oldMax * x, oldMin * x)
        );

        minEnding = Math.min(
                x,
                Math.min(oldMax * x, oldMin * x)
        );

        answer = Math.max(answer, maxEnding);
    }

    return answer;
}
```

This is an important example of adapting an array pattern rather than applying Kadane mechanically.

---

# 18. Array Rotation

Rotation moves elements around a circular boundary.

Example:

```text
[1,2,3,4,5,6,7]
```

Rotate right by `3`:

```text
[5,6,7,1,2,3,4]
```

A standard O(n) / O(1) method uses three reversals.

---

# 19. Right Rotation by K — Reversal Method

For:

```text
[1 2 3 4 5 6 7]
k = 3
```

Reverse all:

```text
[7 6 5 4 3 2 1]
```

Reverse first `k`:

```text
[5 6 7 4 3 2 1]
```

Reverse remaining:

```text
[5 6 7 1 2 3 4]
```

### Java

```java
static void rotateRight(int[] nums, int k) {
    int n = nums.length;

    if (n == 0) {
        return;
    }

    k %= n;

    reverse(nums, 0, n - 1);
    reverse(nums, 0, k - 1);
    reverse(nums, k, n - 1);
}

static void reverse(int[] nums, int left, int right) {
    while (left < right) {
        int temp = nums[left];
        nums[left] = nums[right];
        nums[right] = temp;

        left++;
        right--;
    }
}
```

### Complexity

```text
Time:  O(n)
Space: O(1)
```

### Important

Always normalize:

```java
k %= n;
```

Otherwise:

```text
k > n
```

creates unnecessary work and can break assumptions.

---

# 20. Rearrangement

Array rearrangement problems ask you to move elements according to a rule while often preserving some constraint.

Common examples:

- move zeroes,
- partition positives/negatives,
- separate even/odd,
- Dutch National Flag,
- rearrange by sign,
- move duplicates,
- cyclic placement.

The main question is:

> Can I partition the array in-place using one or two pointers?

---

# 21. Move Zeroes

Goal:

```text
[0,1,0,3,12]
```

becomes:

```text
[1,3,12,0,0]
```

while preserving the relative order of non-zero elements.

### Two-Pointer Solution

Use:

```text
write = next position for a non-zero element
```

### Java

```java
static void moveZeroes(int[] nums) {
    int write = 0;

    for (int x : nums) {
        if (x != 0) {
            nums[write++] = x;
        }
    }

    while (write < nums.length) {
        nums[write++] = 0;
    }
}
```

### Complexity

```text
Time:  O(n)
Space: O(1)
```

---

# 22. Frequency Counting

Frequency counting maps each value to its occurrence count.

```java
Map<Integer, Integer> frequency = new HashMap<>();

for (int x : nums) {
    frequency.put(x, frequency.getOrDefault(x, 0) + 1);
}
```

Use it for:

- duplicates,
- majority,
- anagrams,
- frequency comparisons,
- top-k frequency,
- counting occurrences.

### If the value range is small

An array is often better:

```java
int[] freq = new int[maxValue + 1];
```

This can reduce overhead and improve performance.

But only use it when the value range is manageable.

---

# 23. Frequency Counting vs Sorting

Suppose:

```text
nums = [4, 1, 2, 4, 2, 4]
```

To find frequencies:

### HashMap

```text
Time: O(n) average
Space: O(n)
```

### Sorting

```text
Time: O(n log n)
Space: depends on sorting implementation
```

Sorting can still be preferable if:

- you need ordered values,
- you need to scan groups,
- the problem already benefits from sorted order.

---

# 24. Two Pointers

Two pointers maintain two indices that move according to an invariant.

Common forms:

```text
left -> right
```

or:

```text
slow -> fast
```

or:

```text
left and right moving toward each other
```

---

# 25. Opposite-Direction Two Pointers

Typical signal:

> The array is sorted and you need a pair satisfying a condition.

Example:

```text
[1,2,3,4,6]
target = 6
```

Start:

```text
left = 0
right = n - 1
```

If:

```text
nums[left] + nums[right] < target
```

move `left`.

If:

```text
sum > target
```

move `right`.

### Java

```java
static int[] twoSumSorted(int[] nums, int target) {
    int left = 0;
    int right = nums.length - 1;

    while (left < right) {
        long sum = (long) nums[left] + nums[right];

        if (sum == target) {
            return new int[]{left, right};
        }

        if (sum < target) {
            left++;
        } else {
            right--;
        }
    }

    return new int[]{-1, -1};
}
```

### Complexity

```text
Time: O(n)
Space: O(1)
```

---

# 26. Same-Direction Two Pointers

Used for:

- removing duplicates,
- compaction,
- partitioning,
- merging,
- filtering.

Example:

```text
[0,1,0,3,12]
```

Use:

```text
read -> scans every element
write -> marks next valid position
```

The read pointer never moves backward.

---

# 27. Sliding Window

Sliding window is a specialized two-pointer pattern for contiguous ranges.

It maintains:

```text
[left, right]
```

and updates the window incrementally.

Typical signals:

- longest subarray,
- shortest subarray,
- contiguous segment,
- at most K,
- exactly K,
- fixed number of elements,
- maximum/minimum over a window.

---

# 28. Fixed-Size Sliding Window

Suppose the window size is `k`.

For:

```text
[2,1,5,1,3,2]
k = 3
```

windows are:

```text
[2,1,5]
[1,5,1]
[5,1,3]
[1,3,2]
```

Instead of recomputing every sum:

```text
newWindowSum =
oldWindowSum
- element leaving
+ element entering
```

### Java

```java
static int maxWindowSum(int[] nums, int k) {
    if (k <= 0 || k > nums.length) {
        throw new IllegalArgumentException();
    }

    int windowSum = 0;

    for (int i = 0; i < k; i++) {
        windowSum += nums[i];
    }

    int best = windowSum;

    for (int right = k; right < nums.length; right++) {
        windowSum += nums[right];
        windowSum -= nums[right - k];

        best = Math.max(best, windowSum);
    }

    return best;
}
```

### Complexity

```text
Time:  O(n)
Space: O(1)
```

---

# 29. Variable-Size Sliding Window

Here the window expands and contracts according to a condition.

Generic structure:

```java
int left = 0;

for (int right = 0; right < nums.length; right++) {
    // add nums[right]

    while (/* window invalid */) {
        // remove nums[left]
        left++;
    }

    // window [left, right] is valid
}
```

The key is defining:

> What makes the current window invalid?

---

# 30. Important Sliding-Window Warning

Do not automatically use sliding window for every subarray problem.

Sliding window often relies on a monotonic property.

For example, for:

```text
positive numbers
```

when a sum becomes too large, removing elements from the left decreases the sum.

But with arbitrary negative numbers:

```text
[2, -5, 10]
```

the sum is not monotonic.

For arbitrary integers, prefix sum + hashmap is often the correct pattern for exact-sum problems.

---

# 31. Fast/Slow Pointers

Fast/slow pointers are especially common for:

- in-place filtering,
- removing duplicates,
- cycle detection,
- partitioning.

For arrays, think:

```text
fast -> explores
slow -> maintains valid region
```

Example:

```text
Remove duplicates from sorted array
```

The slow pointer marks where the next distinct element should be placed.

---

# 32. Sorting + Scanning

Sorting often exposes structure.

After sorting:

```text
[1,1,2,2,2,5,7]
```

duplicates become adjacent.

This enables a linear scan.

Typical problems:

- merge intervals,
- duplicate detection,
- 3Sum,
- meeting intervals,
- grouping equal values,
- closest pairs,
- rearrangement.

Complexity usually becomes:

```text
O(n log n)
```

because of sorting.

---

# 33. Merge-Based Techniques

Merge-based array problems usually involve:

- two sorted arrays,
- intervals,
- divide and conquer,
- sorted subproblems.

Basic merge:

```java
static int[] mergeSorted(int[] a, int[] b) {
    int[] result = new int[a.length + b.length];

    int i = 0;
    int j = 0;
    int k = 0;

    while (i < a.length && j < b.length) {
        if (a[i] <= b[j]) {
            result[k++] = a[i++];
        } else {
            result[k++] = b[j++];
        }
    }

    while (i < a.length) {
        result[k++] = a[i++];
    }

    while (j < b.length) {
        result[k++] = b[j++];
    }

    return result;
}
```

### Complexity

```text
Time: O(n + m)
Space: O(n + m)
```

---

# 34. Subarray vs Subsequence vs Subset

This distinction is critical.

## Subarray

Contiguous.

```text
[2,3,4]
```

from:

```text
[1,2,3,4,5]
```

is a subarray.

## Subsequence

Order is preserved, but elements do not need to be contiguous.

```text
[1,3,5]
```

is a subsequence.

## Subset

Order usually does not matter.

This distinction completely changes the algorithm.

### Recognition

If the problem says:

```text
contiguous
consecutive segment
continuous portion
```

think:

```text
subarray
```

---

# 35. Intervals

Intervals are arrays of pairs:

```text
[start, end]
```

Typical tasks:

- merge overlapping intervals,
- insert an interval,
- remove minimum intervals,
- find intersections,
- schedule meetings.

The dominant pattern is:

```text
sort by start
scan from left to right
```

---

# 36. Merge Intervals

Example:

```text
[[1,3],[2,6],[8,10],[15,18]]
```

becomes:

```text
[[1,6],[8,10],[15,18]]
```

### Algorithm

1. Sort by start.
2. Keep the current merged interval.
3. If the next interval overlaps, extend the end.
4. Otherwise, save the current interval and start a new one.

### Java

```java
static int[][] mergeIntervals(int[][] intervals) {
    if (intervals.length <= 1) {
        return intervals;
    }

    Arrays.sort(intervals,
            Comparator.comparingInt(a -> a[0]));

    List<int[]> merged = new ArrayList<>();

    int start = intervals[0][0];
    int end = intervals[0][1];

    for (int i = 1; i < intervals.length; i++) {
        if (intervals[i][0] <= end) {
            end = Math.max(end, intervals[i][1]);
        } else {
            merged.add(new int[]{start, end});
            start = intervals[i][0];
            end = intervals[i][1];
        }
    }

    merged.add(new int[]{start, end});

    return merged.toArray(new int[merged.size()][]);
}
```

### Complexity

```text
Time:  O(n log n)
Space: O(n)
```

The dominant operation is sorting.

---

# 37. Duplicates

There are multiple duplicate patterns.

## Need to detect any duplicate

Use:

```java
HashSet
```

Average:

```text
O(n) time
O(n) space
```

## Array is sorted

Duplicates are adjacent.

Use two pointers:

```text
slow = next unique position
fast = scan
```

## Values are constrained to `1..n`

Consider:

- cyclic placement,
- index marking,
- sign marking.

Do not use these tricks unless the constraints justify them.

---

# 38. Missing Numbers

Missing-number problems often have special constraints.

Common approaches:

### XOR

Useful when:

```text
numbers form a known range
one value is missing
```

### Arithmetic Sum

```text
expected - actual
```

But be careful about integer overflow.

### Cyclic Placement

Useful when:

```text
values are in a known index-related range
```

### Sign Marking

Can work when values can safely be used as indices.

The correct method depends on constraints.

---

# 39. Majority Element

A majority element occurs more than:

```text
floor(n / 2)
```

times.

The key O(1)-space solution is Boyer-Moore Voting.

---

# 40. Boyer-Moore Voting

Maintain:

```text
candidate
count
```

Rules:

```text
count == 0 -> choose current value as candidate

current == candidate -> count++

otherwise -> count--
```

### Java

```java
static int majorityElement(int[] nums) {
    int candidate = 0;
    int count = 0;

    for (int x : nums) {
        if (count == 0) {
            candidate = x;
        }

        count += (x == candidate) ? 1 : -1;
    }

    return candidate;
}
```

This works when the problem guarantees that a majority element exists.

### Complexity

```text
Time:  O(n)
Space: O(1)
```

---

# 41. Product of Array Except Self

Given:

```text
[1,2,3,4]
```

return:

```text
[24,12,8,6]
```

The key observation:

```text
answer[i]
=
product of everything before i
*
product of everything after i
```

### O(n) / O(1) extra space

```java
static int[] productExceptSelf(int[] nums) {
    int n = nums.length;
    int[] answer = new int[n];

    int prefix = 1;

    for (int i = 0; i < n; i++) {
        answer[i] = prefix;
        prefix *= nums[i];
    }

    int suffix = 1;

    for (int i = n - 1; i >= 0; i--) {
        answer[i] *= suffix;
        suffix *= nums[i];
    }

    return answer;
}
```

### Why It Works

After the first pass:

```text
answer[i] = product of elements before i
```

After the reverse pass:

```text
answer[i] *= product of elements after i
```

### Complexity

```text
Time:  O(n)
Extra space: O(1)
Output: O(n)
```

The official LeetCode statement explicitly requires O(n) time and no division; it also gives the O(1)-extra-space follow-up. 

---

# 42. Circular Arrays

Circular arrays connect the end back to the beginning.

For:

```text
[1,2,3,4,5]
```

after index `4`, the next index is:

```text
0
```

Use:

```java
next = (i + 1) % n;
```

and:

```java
previous = (i - 1 + n) % n;
```

---

# 43. Maximum Circular Subarray

For a circular maximum subarray, there are two possibilities:

### Case 1

The maximum subarray does not wrap.

Use Kadane:

```text
maxKadane
```

### Case 2

The maximum subarray wraps.

That means we remove the minimum-sum middle portion.

Therefore:

```text
circularMaximum
=
totalSum - minimumSubarraySum
```

So:

```text
answer = max(
    normalMaximum,
    totalSum - minimumSubarray
)
```

### Important Edge Case

If all elements are negative:

```text
totalSum - minimumSubarray = 0
```

which would incorrectly represent an empty subarray.

So if:

```text
maxKadane < 0
```

return `maxKadane`.

### Java

```java
static int maxSubarraySumCircular(int[] nums) {
    int total = 0;

    int currentMax = nums[0];
    int maxSum = nums[0];

    int currentMin = nums[0];
    int minSum = nums[0];

    for (int i = 0; i < nums.length; i++) {
        total += nums[i];

        if (i > 0) {
            currentMax = Math.max(nums[i], currentMax + nums[i]);
            maxSum = Math.max(maxSum, currentMax);

            currentMin = Math.min(nums[i], currentMin + nums[i]);
            minSum = Math.min(minSum, currentMin);
        }
    }

    if (maxSum < 0) {
        return maxSum;
    }

    return Math.max(maxSum, total - minSum);
}
```

### Complexity

```text
Time: O(n)
Space: O(1)
```

---

# 44. Fixed vs Variable Sliding Window

| Pattern | Window size | Typical use |
|---|---|---|
| Fixed window | exactly `k` | maximum sum of k elements |
| Variable window | changes | longest/shortest valid subarray |
| Prefix + hashmap | not maintained explicitly | exact sum with negative values |
| Two pointers | depends | sorted arrays / partitioning |

### Important

Do not confuse:

```text
subarray sum = k
```

with:

```text
longest subarray with sum <= k
```

The first may require prefix + hashmap.

The second may support sliding window when values satisfy the required monotonic constraints.

---

# 45. Prefix Sum vs Sliding Window

## Use Prefix Sum + HashMap

When:

```text
exact sum
negative values allowed
count subarrays
longest subarray with target sum
```

## Use Sliding Window

When:

```text
contiguous window
window validity changes monotonically
positive/non-negative constraints support shrinking
```

## Use Prefix Sum Alone

When:

```text
many static range-sum queries
```

This distinction prevents many wrong solutions.

---

# 46. Difference Array vs Prefix Sum

They solve opposite-looking problems.

### Prefix Sum

Input:

```text
individual values
```

Output:

```text
range information
```

### Difference Array

Input:

```text
range updates
```

Output:

```text
final individual values
```

Think:

```text
Prefix:
values -> cumulative

Difference:
range changes -> values
```

---

# 47. Sorting + Scanning vs HashMap

### HashMap

Use when:

- order does not matter,
- lookup is the priority,
- expected O(n) solution matters.

### Sorting + Scanning

Use when:

- order is useful,
- neighboring equal values matter,
- two pointers become possible,
- intervals need ordering.

A common interview tradeoff is:

```text
O(n) expected + O(n) space
```

versus:

```text
O(n log n) + less auxiliary space
```

Do not automatically choose the asymptotically faster approach if the problem's constraints make the simpler solution preferable.

---

# 48. Array Problem Decision Framework

When you receive an array problem:

## Step 1 — What is the output asking for?

```text
single value?
indices?
modified array?
subarray?
count?
boolean?
intervals?
```

## Step 2 — Is contiguity important?

If yes:

```text
subarray
```

Then consider:

```text
Kadane
sliding window
prefix sum
prefix + hashmap
```

## Step 3 — Is the array sorted?

If yes:

```text
two pointers
binary search
merge
```

may become available.

## Step 4 — Are many range queries involved?

Think:

```text
prefix sum
```

## Step 5 — Are many range updates involved?

Think:

```text
difference array
```

## Step 6 — Are values repeated?

Think:

```text
HashMap
HashSet
sorting
frequency array
```

## Step 7 — Is there a special value range?

If values are related to indices:

```text
cyclic sort
index marking
```

may be useful.

## Step 8 — Is O(1) extra space required?

Look for:

```text
two pointers
in-place swaps
Boyer-Moore
cyclic placement
prefix/suffix using output
```

---

# 49. Common Mistakes

## Mistake 1: Confusing Subarray and Subsequence

A subarray is contiguous.

A subsequence does not need to be contiguous.

---

## Mistake 2: Using Sliding Window With Negative Numbers

Many sliding-window sum arguments depend on monotonicity.

Negative values can break that assumption.

---

## Mistake 3: Incorrect Kadane Initialization

Wrong:

```java
int current = 0;
int best = 0;
```

for non-empty maximum-subarray problems.

Correct:

```java
int current = nums[0];
int best = nums[0];
```

---

## Mistake 4: Integer Overflow

Use `long` when cumulative values may exceed `int`.

For example:

```java
long prefix = 0;
```

---

## Mistake 5: Forgetting `k %= n`

For rotation:

```java
k %= n;
```

---

## Mistake 6: Off-by-One Prefix Errors

With:

```java
prefix = new long[n + 1];
```

range `[l, r]` is:

```java
prefix[r + 1] - prefix[l]
```

---

## Mistake 7: Incorrect Interval Overlap Condition

For closed intervals:

```text
[start, end]
```

these overlap when:

```text
nextStart <= currentEnd
```

So:

```java
if (next[0] <= currentEnd)
```

is the usual merge condition.

---

## Mistake 8: Ignoring Output Space

For:

```text
Product of Array Except Self
```

the output array is normally not counted as auxiliary space when the problem explicitly says so.

---

## Mistake 9: Using HashMap When an O(1)-Space Pattern Exists

For example:

```text
majority element
```

can use Boyer-Moore when a majority is guaranteed.

---

## Mistake 10: Sorting When Order Must Be Preserved

Sorting destroys original positions and order.

Only sort if the problem permits it or you have a strategy for recovering what you need.

---

# 50. Complexity Patterns

| Technique | Time | Extra Space |
|---|---:|---:|
| Basic traversal | O(n) | O(1) |
| In-place two pointers | O(n) | O(1) |
| Prefix sum construction | O(n) | O(n) |
| Prefix + hashmap | O(n) average | O(n) |
| Difference array | O(n + updates) | O(n) |
| Kadane | O(n) | O(1) |
| Rotation by reversal | O(n) | O(1) |
| Frequency HashMap | O(n) average | O(n) |
| Sorting + scanning | O(n log n) | depends |
| Fixed sliding window | O(n) | O(1) |
| Variable sliding window | O(n) | O(1) to O(n), depending on state |
| Merge sorted arrays | O(n + m) | O(n + m) for output |
| Merge intervals | O(n log n) | O(n) output/result storage |
| Boyer-Moore | O(n) | O(1) |

---

# 51. Easy → Medium → Hard Progression

## Easy

Master:

1. array traversal
2. in-place swaps
3. frequency counting
4. prefix sum
5. two pointers
6. fixed sliding window
7. basic rotation
8. basic rearrangement
9. majority element

## Medium

Then master:

1. prefix + hashmap
2. variable sliding window
3. Kadane variants
4. product except self
5. 3Sum
6. merge intervals
7. subarray problems
8. cyclic placement
9. circular subarrays
10. partitioning

## Hard

Then:

1. Trapping Rain Water
2. First Missing Positive
3. Median of Two Sorted Arrays
4. Sliding Window Maximum
5. Minimum Window Substring
6. advanced interval problems
7. hard circular-array variants
8. advanced in-place rearrangement

Medium should dominate serious interview practice.

---

# 52. Selected LeetCode Problems

The following selection is deliberately limited to representative problems rather than a huge list. Problem pages were checked against the current LeetCode pages/search results before inclusion.

---

## 52.1 Two Sum

- Platform: LeetCode
- Difficulty: Easy
- Topic: Arrays / Hashing
- Pattern: Frequency/lookup map
- Priority: CORE
- Free: Yes

#### Official Link

https://leetcode.com/problems/two-sum/

#### Why This Problem Matters

It teaches the basic transformation:

```text
O(n²) pair search
```

into:

```text
O(n) expected lookup
```

#### What You Should Notice

For each `x`, the required partner is:

```text
target - x
```

#### Hint 1 — Observation

Brute force checks every pair.

#### Hint 2 — Direction

Can you remember values you have already seen?

#### Hint 3 — Pattern / Data Structure

Use a `HashMap` from value to index.

#### Hint 4 — Algorithm

For each number:

1. compute complement,
2. check whether it exists,
3. otherwise store the current value.

#### Java Solution

```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        Map<Integer, Integer> map = new HashMap<>();

        for (int i = 0; i < nums.length; i++) {
            int complement = target - nums[i];

            if (map.containsKey(complement)) {
                return new int[]{map.get(complement), i};
            }

            map.put(nums[i], i);
        }

        return new int[0];
    }
}
```

#### Why It Works

Every current element only needs its complement.

Previously seen elements are stored for O(1)-average lookup.

#### Complexity

- Time: O(n) average
- Space: O(n)

#### Common Mistakes

- storing before checking and accidentally using the same element,
- using nested loops,
- forgetting duplicate values such as `[3,3]`.

#### Follow-Up Variations

- sorted array,
- return values rather than indices,
- 3Sum.

#### Related Patterns

- hashing
- two pointers

#### Mastery Check

Can you explain why the map lookup replaces the inner loop?

---

## 52.2 Best Time to Buy and Sell Stock

- Platform: LeetCode
- Difficulty: Easy
- Topic: Arrays
- Pattern: running minimum / greedy scan
- Priority: CORE
- Free: Yes

#### Official Link

https://leetcode.com/problems/best-time-to-buy-and-sell-stock/

#### Why This Problem Matters

It teaches how to turn a pair-search problem into a one-pass invariant.

#### What You Should Notice

For each selling day, the best buying day is simply the minimum price seen earlier.

#### Hint 1 — Observation

You only sell after buying.

#### Hint 2 — Direction

Track the smallest price so far.

#### Hint 3 — Pattern / Data Structure

Running minimum.

#### Hint 4 — Algorithm

At every price:

```text
profit = price - minimumSeen
```

then update the minimum.

#### Java Solution

```java
class Solution {
    public int maxProfit(int[] prices) {
        int minPrice = Integer.MAX_VALUE;
        int best = 0;

        for (int price : prices) {
            minPrice = Math.min(minPrice, price);
            best = Math.max(best, price - minPrice);
        }

        return best;
    }
}
```

#### Why It Works

For every possible selling position, `minPrice` represents the cheapest valid buying position before it.

#### Complexity

- Time: O(n)
- Space: O(1)

#### Common Mistakes

- allowing buying after selling,
- trying every pair,
- confusing this with unlimited transactions.

#### Follow-Up Variations

- unlimited transactions,
- transaction fee,
- cooldown,
- at most two transactions.

#### Related Patterns

- prefix minimum
- greedy

#### Mastery Check

Can you state the invariant maintained by `minPrice`?

---

## 52.3 Maximum Subarray

- Platform: LeetCode
- Difficulty: Medium
- Topic: Arrays
- Pattern: Kadane's algorithm
- Priority: CORE
- Free: Yes

#### Official Link

https://leetcode.com/problems/maximum-subarray/

#### Why This Problem Matters

Kadane is one of the most important array algorithms.

#### What You Should Notice

At each element:

```text
extend previous subarray
```

or:

```text
start new subarray
```

#### Hint 1 — Observation

A negative accumulated prefix can hurt a future sum.

#### Hint 2 — Direction

Track the best subarray ending exactly at the current position.

#### Hint 3 — Pattern / Data Structure

Kadane.

#### Hint 4 — Algorithm

```java
current = max(nums[i], current + nums[i])
best = max(best, current)
```

#### Java Solution

```java
class Solution {
    public int maxSubArray(int[] nums) {
        int current = nums[0];
        int best = nums[0];

        for (int i = 1; i < nums.length; i++) {
            current = Math.max(nums[i], current + nums[i]);
            best = Math.max(best, current);
        }

        return best;
    }
}
```

#### Why It Works

`current` is the maximum sum of a subarray ending at `i`.

The recurrence considers the only two possibilities:

```text
start at i
extend a subarray ending at i - 1
```

#### Complexity

- Time: O(n)
- Space: O(1)

#### Common Mistakes

- initializing with zero,
- confusing subarray and subsequence,
- returning the final `current` rather than global `best`.

#### Follow-Up Variations

- maximum product,
- maximum circular subarray,
- return the actual indices.

#### Related Patterns

- dynamic programming
- prefix/suffix
- divide and conquer

#### Mastery Check

Explain why a negative prefix can be discarded.

---

## 52.4 Rotate Array

- Platform: LeetCode
- Difficulty: Medium
- Topic: Arrays
- Pattern: reversal / in-place
- Priority: CORE
- Free: Yes

#### Official Link

https://leetcode.com/problems/rotate-array/

#### Why This Problem Matters

It is a canonical in-place transformation.

#### What You Should Notice

A right rotation can be decomposed into three reversals.

#### Hint 1 — Observation

Normalize `k`.

#### Hint 2 — Direction

Reverse the whole array first.

#### Hint 3 — Pattern / Data Structure

Three reversals.

#### Hint 4 — Algorithm

```text
reverse all
reverse first k
reverse remaining n-k
```

#### Java Solution

```java
class Solution {
    public void rotate(int[] nums, int k) {
        int n = nums.length;
        k %= n;

        reverse(nums, 0, n - 1);
        reverse(nums, 0, k - 1);
        reverse(nums, k, n - 1);
    }

    private void reverse(int[] nums, int left, int right) {
        while (left < right) {
            int temp = nums[left];
            nums[left] = nums[right];
            nums[right] = temp;

            left++;
            right--;
        }
    }
}
```

#### Why It Works

The three reversals restore the internal order of each rotated block while moving the blocks into their new positions.

#### Complexity

- Time: O(n)
- Space: O(1)

#### Common Mistakes

- forgetting `k %= n`,
- incorrect reverse boundaries,
- failing on `k = 0`.

#### Follow-Up Variations

- left rotation,
- rotate using cyclic replacement,
- rotate strings.

#### Related Patterns

- in-place
- two pointers

#### Mastery Check

Derive the three-reversal method on paper.

---

## 52.5 Product of Array Except Self

- Platform: LeetCode
- Difficulty: Medium
- Topic: Arrays
- Pattern: prefix + suffix
- Priority: CORE
- Free: Yes

#### Official Link

https://leetcode.com/problems/product-of-array-except-self/

#### Why This Problem Matters

It is a canonical prefix/suffix problem.

#### What You Should Notice

Each output element needs:

```text
left product × right product
```

#### Hint 1 — Observation

Do not divide.

#### Hint 2 — Direction

Store the left product in the answer array.

#### Hint 3 — Pattern / Data Structure

Prefix pass + suffix pass.

#### Hint 4 — Algorithm

First pass stores prefix products.

Second pass multiplies by a running suffix product.

#### Java Solution

```java
class Solution {
    public int[] productExceptSelf(int[] nums) {
        int n = nums.length;
        int[] answer = new int[n];

        int prefix = 1;

        for (int i = 0; i < n; i++) {
            answer[i] = prefix;
            prefix *= nums[i];
        }

        int suffix = 1;

        for (int i = n - 1; i >= 0; i--) {
            answer[i] *= suffix;
            suffix *= nums[i];
        }

        return answer;
    }
}
```

#### Why It Works

At index `i`:

```text
answer[i]
=
nums[0] ... nums[i-1]
×
nums[i+1] ... nums[n-1]
```

#### Complexity

- Time: O(n)
- Space: O(1) extra, excluding output

#### Common Mistakes

- using division,
- mishandling zeroes,
- allocating separate prefix and suffix arrays when O(1) extra space is required.

#### Follow-Up Variations

- sum instead of product,
- prefix/suffix maximum,
- product on circular arrays.

#### Related Patterns

- prefix sum
- suffix
- two-pass techniques

#### Mastery Check

Can you explain how the output array replaces a separate prefix array?

---

## 52.6 3Sum

- Platform: LeetCode
- Difficulty: Medium
- Topic: Arrays
- Pattern: sorting + two pointers
- Priority: CORE
- Free: Yes

#### Official Link

https://leetcode.com/problems/3sum/

#### Why This Problem Matters

This is one of the canonical two-pointer problems.

#### What You Should Notice

Fix one value and reduce the remaining problem to 2Sum on a sorted array.

#### Hint 1 — Observation

Three nested loops are too slow.

#### Hint 2 — Direction

Sort first.

#### Hint 3 — Pattern / Data Structure

Fix `i`, then use `left/right`.

#### Hint 4 — Algorithm

For each `i`:

```text
target = -nums[i]
```

and solve two-sum with two pointers.

Skip duplicates carefully.

#### Java Solution

```java
class Solution {
    public List<List<Integer>> threeSum(int[] nums) {
        Arrays.sort(nums);

        List<List<Integer>> result = new ArrayList<>();

        for (int i = 0; i < nums.length - 2; i++) {
            if (i > 0 && nums[i] == nums[i - 1]) {
                continue;
            }

            int left = i + 1;
            int right = nums.length - 1;

            while (left < right) {
                long sum = (long) nums[i] + nums[left] + nums[right];

                if (sum == 0) {
                    result.add(Arrays.asList(
                            nums[i],
                            nums[left],
                            nums[right]
                    ));

                    int leftValue = nums[left];
                    int rightValue = nums[right];

                    while (left < right && nums[left] == leftValue) {
                        left++;
                    }

                    while (left < right && nums[right] == rightValue) {
                        right--;
                    }
                } else if (sum < 0) {
                    left++;
                } else {
                    right--;
                }
            }
        }

        return result;
    }
}
```

#### Why It Works

Sorting makes the sum monotonic as pointers move.

#### Complexity

- Time: O(n²)
- Space: O(1) auxiliary, excluding output and sorting implementation details

#### Common Mistakes

- forgetting to sort,
- duplicate triplets,
- integer overflow,
- moving the wrong pointer.

#### Follow-Up Variations

- 3Sum closest,
- 4Sum,
- kSum.

#### Related Patterns

- two pointers
- sorting + scanning

#### Mastery Check

Explain why sorting allows pointer movement to be directional.

---

## 52.7 Two Sum II — Input Array Is Sorted

- Platform: LeetCode
- Difficulty: Medium
- Topic: Arrays
- Pattern: opposite-direction two pointers
- Priority: IMPORTANT
- Free: Yes

#### Official Link

https://leetcode.com/problems/two-sum-ii-input-array-is-sorted/

#### Why This Problem Matters

It demonstrates the cleanest opposite-pointer invariant.

#### Java Solution

```java
class Solution {
    public int[] twoSum(int[] numbers, int target) {
        int left = 0;
        int right = numbers.length - 1;

        while (left < right) {
            int sum = numbers[left] + numbers[right];

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

#### Complexity

- Time: O(n)
- Space: O(1)

#### Mastery Check

Why is moving `left` correct when the sum is too small?

---

## 52.8 Subarray Sum Equals K

- Platform: LeetCode
- Difficulty: Medium
- Topic: Arrays
- Pattern: prefix sum + hashmap
- Priority: CORE
- Free: Yes

#### Official Link

https://leetcode.com/problems/subarray-sum-equals-k/

#### Why This Problem Matters

It teaches one of the most reusable subarray transformations.

#### Hint 1 — Observation

For current prefix `P`, we need an earlier prefix:

```text
P - k
```

#### Hint 2 — Direction

Count previous prefix sums.

#### Hint 3 — Pattern / Data Structure

HashMap of prefix frequency.

#### Hint 4 — Algorithm

```text
prefix += x
answer += frequency[prefix-k]
frequency[prefix]++
```

#### Java Solution

```java
class Solution {
    public int subarraySum(int[] nums, int k) {
        Map<Integer, Integer> map = new HashMap<>();
        map.put(0, 1);

        int prefix = 0;
        int answer = 0;

        for (int x : nums) {
            prefix += x;

            answer += map.getOrDefault(prefix - k, 0);

            map.put(prefix, map.getOrDefault(prefix, 0) + 1);
        }

        return answer;
    }
}
```

#### Why It Works

If:

```text
currentPrefix - oldPrefix = k
```

then:

```text
oldPrefix = currentPrefix - k
```

Every matching earlier prefix gives one valid subarray.

#### Complexity

- Time: O(n) average
- Space: O(n)

#### Common Mistakes

- using sliding window with negative numbers,
- forgetting `map.put(0, 1)`,
- storing before checking and misunderstanding the prefix relation.

#### Follow-Up Variations

- longest subarray with sum K,
- subarray sum divisible by K,
- zero-sum subarray.

#### Related Patterns

- prefix sum
- hashing

#### Mastery Check

Derive the hashmap key `prefix - k`.

---

## 52.9 Majority Element

- Platform: LeetCode
- Difficulty: Easy
- Topic: Arrays
- Pattern: Boyer-Moore voting
- Priority: CORE
- Free: Yes

#### Official Link

https://leetcode.com/problems/majority-element/

#### Why This Problem Matters

It is a canonical O(n) / O(1)-space array problem.

#### Java Solution

```java
class Solution {
    public int majorityElement(int[] nums) {
        int candidate = 0;
        int count = 0;

        for (int x : nums) {
            if (count == 0) {
                candidate = x;
            }

            count += (x == candidate) ? 1 : -1;
        }

        return candidate;
    }
}
```

#### Why It Works

A true majority appears more than all other elements combined.

Pairing a majority occurrence against a different element cannot eliminate all majority occurrences.

#### Complexity

- Time: O(n)
- Space: O(1)

#### Common Mistakes

- using the algorithm when no majority is guaranteed without a verification pass,
- confusing majority `> n/2` with `> n/3`.

#### Follow-Up Variations

- Majority Element II,
- verify candidate,
- weighted voting interpretations.

#### Related Patterns

- frequency counting
- cancellation

#### Mastery Check

Explain the cancellation argument.

---

## 52.10 Merge Intervals

- Platform: LeetCode
- Difficulty: Medium
- Topic: Arrays / Intervals
- Pattern: sorting + scanning
- Priority: CORE
- Free: Yes

#### Official Link

https://leetcode.com/problems/merge-intervals/

#### Why This Problem Matters

It is the canonical interval pattern.

#### Java Solution

```java
class Solution {
    public int[][] merge(int[][] intervals) {
        if (intervals.length <= 1) {
            return intervals;
        }

        Arrays.sort(intervals,
                Comparator.comparingInt(a -> a[0]));

        List<int[]> result = new ArrayList<>();

        int start = intervals[0][0];
        int end = intervals[0][1];

        for (int i = 1; i < intervals.length; i++) {
            if (intervals[i][0] <= end) {
                end = Math.max(end, intervals[i][1]);
            } else {
                result.add(new int[]{start, end});
                start = intervals[i][0];
                end = intervals[i][1];
            }
        }

        result.add(new int[]{start, end});

        return result.toArray(new int[result.size()][]);
    }
}
```

#### Complexity

- Time: O(n log n)
- Space: O(n) for output/result storage

#### Common Mistakes

- forgetting to sort,
- checking only whether endpoints are equal,
- failing to extend the maximum end.

#### Mastery Check

State the overlap invariant after sorting.

---

## 52.11 Longest Consecutive Sequence

- Platform: LeetCode
- Difficulty: Medium
- Topic: Arrays / Hashing
- Pattern: HashSet + sequence starts
- Priority: IMPORTANT
- Free: Yes

#### Official Link

https://leetcode.com/problems/longest-consecutive-sequence/

#### Why This Problem Matters

It demonstrates how to avoid sorting and still achieve expected O(n).

#### Key Idea

Only start counting when:

```java
!set.contains(x - 1)
```

Then count:

```text
x, x+1, x+2, ...
```

### Java

```java
class Solution {
    public int longestConsecutive(int[] nums) {
        Set<Integer> set = new HashSet<>();

        for (int x : nums) {
            set.add(x);
        }

        int best = 0;

        for (int x : set) {
            if (!set.contains(x - 1)) {
                int current = x;
                int length = 1;

                while (set.contains(current + 1)) {
                    current++;
                    length++;
                }

                best = Math.max(best, length);
            }
        }

        return best;
    }
}
```

#### Complexity

- Time: O(n) expected
- Space: O(n)

#### Mastery Check

Why does starting only at sequence beginnings prevent repeated scanning?

---

## 52.12 First Missing Positive

- Platform: LeetCode
- Difficulty: Hard
- Topic: Arrays
- Pattern: cyclic placement / index mapping
- Priority: CORE
- Free: Yes

#### Official Link

https://leetcode.com/problems/first-missing-positive/

#### Why This Problem Matters

It is a major in-place array pattern.

#### Key Observation

For a valid value `x`:

```text
x should be at index x - 1
```

for values in:

```text
1..n
```

### Java

```java
class Solution {
    public int firstMissingPositive(int[] nums) {
        int n = nums.length;

        for (int i = 0; i < n; i++) {
            while (nums[i] >= 1
                    && nums[i] <= n
                    && nums[nums[i] - 1] != nums[i]) {

                int correctIndex = nums[i] - 1;

                int temp = nums[i];
                nums[i] = nums[correctIndex];
                nums[correctIndex] = temp;
            }
        }

        for (int i = 0; i < n; i++) {
            if (nums[i] != i + 1) {
                return i + 1;
            }
        }

        return n + 1;
    }
}
```

#### Complexity

- Time: O(n) amortized
- Space: O(1)

#### Common Mistakes

- infinite swapping when duplicates exist,
- using an invalid index,
- ignoring non-positive values.

#### Mastery Check

Why does the first index containing the wrong value identify the answer?

---

## 52.13 Trapping Rain Water

- Platform: LeetCode
- Difficulty: Hard
- Topic: Arrays
- Pattern: two pointers / prefix-suffix maxima
- Priority: CORE
- Free: Yes

#### Official Link

https://leetcode.com/problems/trapping-rain-water/

#### Why This Problem Matters

It combines two pointers with a strong invariant.

#### Key Idea

Water at position `i` is:

```text
min(maxLeft, maxRight) - height[i]
```

The two-pointer solution avoids storing both arrays.

### Java

```java
class Solution {
    public int trap(int[] height) {
        int left = 0;
        int right = height.length - 1;

        int leftMax = 0;
        int rightMax = 0;
        int water = 0;

        while (left < right) {
            if (height[left] <= height[right]) {
                if (height[left] >= leftMax) {
                    leftMax = height[left];
                } else {
                    water += leftMax - height[left];
                }

                left++;
            } else {
                if (height[right] >= rightMax) {
                    rightMax = height[right];
                } else {
                    water += rightMax - height[right];
                }

                right--;
            }
        }

        return water;
    }
}
```

#### Complexity

- Time: O(n)
- Space: O(1)

#### Common Mistakes

- using the wrong side's maximum,
- confusing global maximum with side maximum,
- off-by-one pointer errors.

#### Mastery Check

Explain why processing the shorter boundary is safe.

---

# 53. Fully Explained Problems

## Problem 1 — Longest Subarray With Sum K

### Question

Given an integer array, find the length of the longest contiguous subarray whose sum equals `k`.

Negative values are allowed.

### Why Sliding Window Is Dangerous

Suppose:

```text
[1, -1, 5, -2, 3]
```

The sum does not move monotonically when the window expands or shrinks.

So the usual positive-number sliding-window argument fails.

### Prefix-Sum Derivation

Let:

```text
prefix[i] = sum from 0 through i
```

For subarray:

```text
j + 1 ... i
```

the sum is:

```text
prefix[i] - prefix[j]
```

We need:

```text
prefix[i] - prefix[j] = k
```

therefore:

```text
prefix[j] = prefix[i] - k
```

To maximize length:

```text
i - j
```

we should store the **earliest** index at which each prefix sum occurred.

### Java

```java
static int longestSubarraySumK(int[] nums, int k) {
    Map<Long, Integer> firstIndex = new HashMap<>();

    long prefix = 0;
    int best = 0;

    for (int i = 0; i < nums.length; i++) {
        prefix += nums[i];

        if (prefix == k) {
            best = i + 1;
        }

        firstIndex.putIfAbsent(prefix, i);

        Integer previous = firstIndex.get(prefix - k);

        if (previous != null) {
            best = Math.max(best, i - previous);
        }
    }

    return best;
}
```

### Why Store the Earliest Index?

Suppose the same prefix sum appears at:

```text
index 2
index 7
```

For maximizing length at a future index `i`, index `2` is always better because:

```text
i - 2 > i - 7
```

Therefore we should never replace the earliest occurrence.

### Complexity

```text
Time: O(n) expected
Space: O(n)
```

### Generalizable Lesson

For:

```text
longest subarray with exact sum
```

and arbitrary integers:

```text
prefix sum + earliest index hashmap
```

should be one of your first thoughts.

---

## Problem 2 — Maximum Subarray

### Question

Find the maximum sum of a non-empty contiguous subarray.

Example:

```text
[-2,1,-3,4,-1,2,1,-5,4]
```

Answer:

```text
6
```

### State Definition

Let:

```text
current
```

be the maximum sum of a subarray ending at the current index.

At `nums[i]`, there are only two relevant choices:

```text
start a new subarray at i
```

or:

```text
extend the previous best-ending-here subarray
```

Therefore:

```text
current = max(nums[i], current + nums[i])
```

The global answer is:

```text
best = max(best, current)
```

### Java

```java
static int maxSubarraySum(int[] nums) {
    int current = nums[0];
    int best = nums[0];

    for (int i = 1; i < nums.length; i++) {
        current = Math.max(nums[i], current + nums[i]);
        best = Math.max(best, current);
    }

    return best;
}
```

### Complexity

```text
Time: O(n)
Space: O(1)
```

### Generalizable Lesson

When a problem asks for the best contiguous subarray and the objective is additive, think about whether the best state ending at each index can be maintained.

---

## Problem 3 — Merge Intervals

### Question

Merge all overlapping intervals.

Example:

```text
[[1,3],[2,6],[8,10],[15,18]]
```

### Step 1 — Sort

Sort by start:

```text
[1,3]
[2,6]
[8,10]
[15,18]
```

### Step 2 — Maintain Current Interval

Start:

```text
[1,3]
```

Next:

```text
[2,6]
```

Since:

```text
2 <= 3
```

they overlap.

Merge:

```text
[1,6]
```

Next:

```text
[8,10]
```

Since:

```text
8 > 6
```

there is no overlap.

Save `[1,6]`.

Start `[8,10]`.

### Complexity

Sorting dominates:

```text
O(n log n)
```

### Generalizable Lesson

For interval problems, sorting often converts a complicated global-overlap problem into a local left-to-right scan.

---

# 54. 3 Fully Explained & Solved PYQs

The following are verified GATE CSE previous-year questions that directly test array algorithms/concepts.

---

## PYQ 1 — GATE CSE 1994, Question 7

### Question

An array `A` contains `n` integers. The elements must be shifted cyclically to the left by `K` places in linear time without using another array. An incomplete algorithm using cycle-based movement is provided, and the blanks must be filled.

### What Is Being Tested

- array rotation,
- cyclic movement,
- in-place rearrangement,
- modular indexing,
- O(n) rotation.

### Approach

A left rotation by `K` moves:

```text
A[i] -> A[(i - K + n) % n]
```

The cycle decomposition is based on repeatedly moving:

```text
j = (j + K) % n
```

until a cycle closes.

### Step-by-Step Idea

Suppose:

```text
A = [1,2,3,4,5,6,7]
K = 2
```

The movement forms cycles:

```text
0 -> 2 -> 4 -> 6 -> 1 -> 3 -> 5 -> 0
```

Each element is moved exactly once.

Therefore:

```text
Time = O(n)
```

and:

```text
Extra space = O(1)
```

### Final Answer

The blanks must implement:

1. continuation until all positions have been processed,
2. cycle movement using `(j + K) % n`,
3. temporary-value movement,
4. restoration of the final temporary value,
5. advancing to the next unprocessed cycle.

The essential invariant is:

> Every cycle is processed without losing the element currently stored in the temporary variable.

### Why This Works

Rotation by `K` is a permutation of indices. Every index belongs to exactly one cycle.

### Common Trap

A naive approach shifts the array one position `K` times, which can become:

```text
O(nK)
```

instead of:

```text
O(n)
```

### Generalizable Lesson

When an in-place rearrangement is a permutation of indices, look for cycle decomposition.

Source: GATE CSE 1994 Q7. citeturn1search2

---

## PYQ 2 — GATE CSE 2000, Question 2.15

### Question

Given an array `s[1...n]` and a procedure:

```text
reverse(s, i, j)
```

the following operations are performed:

```text
reverse(s, 1, k)
reverse(s, k+1, n)
reverse(s, 1, n)
```

What does this sequence do?

Options include:

1. rotates `s` left by `k` positions
2. leaves `s` unchanged
3. reverses all elements
4. none of the above

### What Is Being Tested

- array reversal,
- rotation,
- in-place transformation,
- recognizing composite operations.

### Approach

Represent the array as two blocks:

```text
A B
```

where:

```text
A = first k elements
B = remaining n-k elements
```

First reversal:

```text
reverse(A) B
```

Second reversal:

```text
reverse(A) reverse(B)
```

Third reversal reverses the entire sequence:

```text
B A
```

That is exactly a left rotation by `k`.

### Example

Let:

```text
S = [1,2,3,4,5,6,7]
k = 2
```

After first reversal:

```text
[2,1,3,4,5,6,7]
```

After second:

```text
[2,1,7,6,5,4,3]
```

After third:

```text
[3,4,5,6,7,1,2]
```

This is the original array rotated left by `2`.

### Final Answer

**Option 1 — rotates `s` left by `k` positions.**

### Why This Works

The identity is:

```text
reverse(A + B)
```

after separately reversing both blocks, producing:

```text
B + A
```

### Common Trap

Do not confuse the operation with a right rotation.

### Generalizable Lesson

Three reversals can implement an in-place rotation in O(n) time and O(1) auxiliary space.

Source: GATE CSE 2000 Q2.15. citeturn1search11turn1search12

---

## PYQ 3 — GATE CSE 2019, Question 25

### Question

Given:

```text
A = [-5, -10, 6, 3, -1, -2, 13, 4, -9, -1, 4, 12, -3, 0]
```

define:

```text
S(i,j) = sum of A[k] for k=i...j
```

Find the maximum possible value of `S(i,j)`.

### What Is Being Tested

- subarrays,
- maximum subarray sum,
- Kadane's algorithm,
- divide and conquer,
- prefix reasoning.

### Step-by-Step Solution

Start with:

```text
current = -5
best = -5
```

At `-10`:

```text
max(-10, -5 + -10)
= -10
```

At `6`:

```text
max(6, -10 + 6)
= 6
```

At `3`:

```text
max(3, 6 + 3)
= 9
```

At `-1`:

```text
max(-1, 9 - 1)
= 8
```

At `-2`:

```text
6
```

At `13`:

```text
19
```

At `4`:

```text
23
```

At `-9`:

```text
14
```

At `-1`:

```text
13
```

At `4`:

```text
17
```

At `12`:

```text
29
```

At `-3`:

```text
26
```

At `0`:

```text
26
```

The maximum encountered is:

```text
29
```

The corresponding subarray is:

```text
[6, 3, -1, -2, 13, 4, -9, -1, 4, 12]
```

### Final Answer

```text
29
```

### Why This Works

At every index, the optimal subarray ending there either:

```text
starts at the current element
```

or:

```text
extends the optimal subarray ending at the previous index
```

This is exactly Kadane's recurrence.

### Common Trap

The question asks for a **subarray**, not an arbitrary subsequence.

The elements must be contiguous.

### Generalizable Lesson

When the objective is maximum sum over contiguous ranges, derive the best value ending at each index.

Source: GATE CSE 2019 Q25. citeturn1search0turn1search3

---

# 55. Pattern Recognition Checklist

## Traversal

Ask:

- Do I need every element?
- Can this be done in one pass?
- Do I need previous/next elements?

## Prefix Sum

Ask:

- Are there range-sum queries?
- Is a subarray sum involved?
- Can a condition be written as a difference of prefixes?

## Prefix + HashMap

Ask:

- Is the exact subarray sum important?
- Are negative values allowed?
- Do I need count or longest length?

## Suffix

Ask:

- Do I need information about everything to the right?

## Difference Array

Ask:

- Are there many range updates?

## Kadane

Ask:

- Is the problem asking for maximum/minimum contiguous sum?
- Can I define the best subarray ending at `i`?

## Two Pointers

Ask:

- Is the array sorted?
- Can one pointer move monotonically?
- Is the problem about pairs or in-place filtering?

## Sliding Window

Ask:

- Is the answer a contiguous window?
- Can the window validity be maintained incrementally?
- Is the window fixed or variable?
- Are the value constraints sufficient for monotonic shrinking?

## Sorting + Scanning

Ask:

- Would ordering expose duplicates/interval relationships?
- Can I trade O(n) expected hashing for O(n log n) deterministic sorting?

## Frequency Counting

Ask:

- Does only occurrence count matter?
- Is the value range small enough for an array frequency table?

## Intervals

Ask:

- Can I sort by start?
- What exactly defines overlap?

## In-Place Rearrangement

Ask:

- Can values be placed into their target positions?
- Can two pointers partition the array?
- Is O(1) extra space required?

---

# 56. Mastery Checklist

## Core Arrays

- [ ] Traverse arrays confidently.
- [ ] Modify arrays in-place.
- [ ] Reverse an array.
- [ ] Swap elements.
- [ ] Build prefix sums.
- [ ] Build suffix information.
- [ ] Use difference arrays.
- [ ] Implement Kadane's algorithm.
- [ ] Rotate an array in-place.
- [ ] Perform frequency counting.

## Two Pointers

- [ ] Opposite-direction pointers.
- [ ] Same-direction pointers.
- [ ] Slow/fast pointers.
- [ ] Remove duplicates.
- [ ] Partition arrays.
- [ ] Solve sorted 2Sum.
- [ ] Understand why pointer movement is safe.

## Sliding Window

- [ ] Fixed-size window.
- [ ] Variable-size window.
- [ ] Know when shrinking is valid.
- [ ] Know when sliding window fails.
- [ ] Distinguish sliding window from prefix + hashmap.

## Prefix/Suffix

- [ ] Range sum in O(1) after preprocessing.
- [ ] Prefix + hashmap for exact sums.
- [ ] Longest subarray with sum K.
- [ ] Product except self.
- [ ] Prefix/suffix maxima.

## Rearrangement

- [ ] Move zeroes.
- [ ] Array rotation.
- [ ] Cyclic placement.
- [ ] Partitioning.
- [ ] First missing positive.

## Intervals

- [ ] Sort intervals.
- [ ] Merge intervals.
- [ ] Detect overlap.
- [ ] Understand inclusive endpoints.
- [ ] Solve interval scheduling variants.

## Important Problems

- [ ] Two Sum.
- [ ] Best Time to Buy and Sell Stock.
- [ ] Maximum Subarray.
- [ ] Rotate Array.
- [ ] Product of Array Except Self.
- [ ] 3Sum.
- [ ] Two Sum II.
- [ ] Subarray Sum Equals K.
- [ ] Majority Element.
- [ ] Merge Intervals.
- [ ] Longest Consecutive Sequence.
- [ ] First Missing Positive.
- [ ] Trapping Rain Water.

## Interview Standard

You should be able to:

- [ ] identify the pattern within a few minutes,
- [ ] state the brute-force solution,
- [ ] explain why it is too slow when applicable,
- [ ] derive the optimized solution,
- [ ] prove the invariant,
- [ ] implement it in Java,
- [ ] analyze time and space,
- [ ] handle edge cases,
- [ ] explain the solution verbally.

---

# 57. What to Study Next

The next topics should build naturally from arrays.

Recommended order:

1. **Strings**
2. **Hashing / HashMap / HashSet**
3. **Binary Search**
4. **Linked Lists**
5. **Stacks and Queues**
6. **Intervals / Greedy**
7. **Advanced Sliding Window**
8. **Bit Manipulation**
9. **Recursion and Backtracking**

Do not treat arrays as "finished" after completing this file.

Arrays recur everywhere.

The real mastery target is pattern recognition:

```text
Array
 |
 +-- contiguous range?
 |     |
 |     +-- max/min sum -> Kadane
 |     +-- exact sum -> prefix + hashmap
 |     +-- valid window -> sliding window
 |
 +-- sorted?
 |     |
 |     +-- pair -> two pointers
 |     +-- merge -> two pointers
 |
 +-- many range queries?
 |     |
 |     +-- prefix sum
 |
 +-- many range updates?
 |     |
 |     +-- difference array
 |
 +-- repeated values?
 |     |
 |     +-- frequency/hashmap/sort
 |
 +-- intervals?
 |     |
 |     +-- sort + scan
 |
 +-- constrained values?
 |     |
 |     +-- cyclic placement/index marking
 |
 +-- O(1) space?
       |
       +-- two pointers / in-place / voting / cycles
```

---

# 58. Final Array Cheat Sheet

## Traversal

```java
for (int i = 0; i < n; i++) {
    // nums[i]
}
```

## Prefix Sum

```java
prefix[i + 1] = prefix[i] + nums[i];
```

Range:

```java
prefix[r + 1] - prefix[l]
```

## Difference Array

Range update:

```java
diff[l] += value;
diff[r + 1] -= value;
```

## Kadane

```java
current = Math.max(nums[i], current + nums[i]);
best = Math.max(best, current);
```

## Rotation

```text
reverse all
reverse first k
reverse remaining
```

## Two Pointers

```java
while (left < right) {
    // move one or both pointers
}
```

## Fixed Window

```java
window += nums[right];
window -= nums[right - k];
```

## Prefix + HashMap

```java
prefix += x;
answer += map.getOrDefault(prefix - k, 0);
map.put(prefix, map.getOrDefault(prefix, 0) + 1);
```

## Majority

```text
candidate + count
```

## Product Except Self

```text
prefix pass
+
suffix pass
```

## Intervals

```text
sort by start
scan and merge
```

## Cyclic Placement

```text
value x belongs near index x - 1
```

## Circular Maximum Subarray

```text
max(normal Kadane,
    total - minimum subarray)
```

with an all-negative check.

---

# 59. Final Interview Mental Model

When you see an array problem, do not immediately start coding.

First classify it:

```text
1. What does the problem ask for?
   value / index / count / subarray / modified array

2. Is the range contiguous?
   yes -> subarray techniques

3. Is the array sorted?
   yes -> two pointers / binary search / merge

4. Is there a sum?
   range queries -> prefix sum
   exact subarray sum -> prefix + hashmap
   maximum contiguous sum -> Kadane

5. Is there a window?
   fixed -> fixed sliding window
   variable -> variable sliding window

6. Are there duplicates/frequencies?
   hashmap / hashset / frequency array / sorting

7. Are there intervals?
   sort + scan

8. Are values constrained by their indices?
   cyclic placement / marking

9. Is O(1) extra space required?
   in-place / pointers / cycles / voting

10. Can I state the invariant?
    if not, the solution probably is not fully understood.
```

The goal is not to memorize 100 array solutions.

The goal is to recognize that many array problems are variations of a small set of powerful patterns.
