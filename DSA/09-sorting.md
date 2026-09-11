# 09 — Sorting

> **Goal:** Master sorting algorithms and sorting-based problem-solving for FAANG/top product-company interviews, competitive programming, and GATE-level CS fundamentals. Focus on understanding *why* each algorithm works, when to use it, its complexity, stability, memory behavior, and how sorting becomes a tool for solving harder problems.

---

## 1. What Is Sorting?

Sorting rearranges elements according to an ordering rule.

Example:

```text
Input:  [5, 2, 8, 1, 3]

Ascending:
[1, 2, 3, 5, 8]

Descending:
[8, 5, 3, 2, 1]
```

Sorting is important because many problems become simpler after ordering the data.

Common consequences of sorting:

- Binary search becomes possible.
- Duplicate values become adjacent.
- Two-pointer techniques become easier.
- Interval merging becomes easier.
- Greedy decisions become easier.
- Median and order statistics become easier.
- Pair/triplet problems become easier.
- Custom ordering becomes possible.

---

# 2. Sorting Algorithm Comparison

| Algorithm | Best | Average | Worst | Extra Space | Stable | In-place |
|---|---:|---:|---:|---:|---|---|
| Bubble Sort | O(n) | O(n²) | O(n²) | O(1) | Yes | Yes |
| Selection Sort | O(n²) | O(n²) | O(n²) | O(1) | Usually No | Yes |
| Insertion Sort | O(n) | O(n²) | O(n²) | O(1) | Yes | Yes |
| Merge Sort | O(n log n) | O(n log n) | O(n log n) | O(n) for arrays | Yes | No for typical array implementation |
| Quick Sort | O(n log n) | O(n log n) | O(n²) | O(log n) average recursion | Usually No | Yes |
| Heap Sort | O(n log n) | O(n log n) | O(n log n) | O(1) auxiliary | No | Yes |
| Counting Sort | O(n + k) | O(n + k) | O(n + k) | O(n + k) | Can be | No |
| Radix Sort | O(d(n + k)) | O(d(n + k)) | O(d(n + k)) | O(n + k) | Yes with stable digit sort | No |
| Bucket Sort | O(n + k) expected* | O(n + k) expected* | O(n²) typical worst | O(n + k) | Depends | No |

Where:

- `n` = number of elements.
- `k` = value/range/bucket-related parameter depending on the algorithm.
- `d` = number of digits/passes.
- `*` Bucket sort's expected performance depends heavily on how uniformly elements are distributed.

### Interview priority

Know deeply:

1. Merge Sort
2. Quick Sort
3. Heap Sort
4. Counting Sort
5. Radix Sort
6. Bucket Sort

Know conceptually and be able to implement:

7. Bubble Sort
8. Selection Sort
9. Insertion Sort

---

# 3. Basic Sorting Algorithms

# 3.1 Bubble Sort

## Idea

Repeatedly compare adjacent elements and swap them when they are in the wrong order.

After one complete pass, the largest remaining element moves to the end.

Example:

```text
[5, 1, 4, 2, 8]

Pass 1:
5 > 1 → swap
[1, 5, 4, 2, 8]

5 > 4 → swap
[1, 4, 5, 2, 8]

5 > 2 → swap
[1, 4, 2, 5, 8]

5 < 8 → no swap

Largest element 8 is now fixed.
```

## Java

```java
static void bubbleSort(int[] a) {
    int n = a.length;

    for (int i = 0; i < n - 1; i++) {
        boolean swapped = false;

        for (int j = 0; j < n - 1 - i; j++) {
            if (a[j] > a[j + 1]) {
                int temp = a[j];
                a[j] = a[j + 1];
                a[j + 1] = temp;
                swapped = true;
            }
        }

        if (!swapped) {
            break;
        }
    }
}
```

## Complexity

| Case | Time |
|---|---:|
| Best | O(n) with early-exit optimization |
| Average | O(n²) |
| Worst | O(n²) |

Space:

```text
O(1)
```

## Properties

- Stable.
- In-place.
- Simple.
- Rarely appropriate for production/interview optimization.

### Interview takeaway

Do not choose Bubble Sort when an O(n log n) solution is available.

Its main value is understanding:

- adjacent swaps,
- inversion reduction,
- stable in-place sorting.

---

# 3.2 Selection Sort

## Idea

Repeatedly find the minimum element in the unsorted suffix and place it at the current position.

Example:

```text
[5, 3, 4, 1, 2]

Find minimum = 1
[1, 3, 4, 5, 2]

Find minimum of remaining = 2
[1, 2, 4, 5, 3]

Find minimum = 3
[1, 2, 3, 5, 4]

Find minimum = 4
[1, 2, 3, 4, 5]
```

## Java

```java
static void selectionSort(int[] a) {
    int n = a.length;

    for (int i = 0; i < n - 1; i++) {
        int minIndex = i;

        for (int j = i + 1; j < n; j++) {
            if (a[j] < a[minIndex]) {
                minIndex = j;
            }
        }

        int temp = a[i];
        a[i] = a[minIndex];
        a[minIndex] = temp;
    }
}
```

## Complexity

```text
Best    O(n²)
Average O(n²)
Worst   O(n²)
Space   O(1)
```

## Properties

- In-place.
- Usually unstable.
- Performs relatively few swaps: at most O(n) swaps.

### Why is ordinary Selection Sort unstable?

Consider:

```text
[(2,A), (2,B), (1,C)]
```

Selecting `1` and swapping it with the first element:

```text
[(1,C), (2,B), (2,A)]
```

The relative order of equal `2`s changed:

```text
A before B → B before A
```

Therefore ordinary selection sort is not stable.

---

# 3.3 Insertion Sort

## Idea

Maintain a sorted prefix.

Take the next element and insert it into its correct position inside the sorted prefix.

Example:

```text
[5, 2, 4, 6, 1, 3]

5
5,2 → insert 2
2,5

4 → insert between 2 and 5
2,4,5

6
2,4,5,6

1
1,2,4,5,6

3
1,2,3,4,5,6
```

## Java

```java
static void insertionSort(int[] a) {
    for (int i = 1; i < a.length; i++) {
        int key = a[i];
        int j = i - 1;

        while (j >= 0 && a[j] > key) {
            a[j + 1] = a[j];
            j--;
        }

        a[j + 1] = key;
    }
}
```

## Complexity

| Case | Time |
|---|---:|
| Best | O(n) |
| Average | O(n²) |
| Worst | O(n²) |

Space:

```text
O(1)
```

## Properties

- Stable.
- In-place.
- Adaptive.
- Excellent for small or nearly sorted arrays.

### Important insight

Insertion Sort performs work proportional to the number of inversions.

An inversion is a pair:

```text
i < j
and
a[i] > a[j]
```

A nearly sorted array has few inversions, so insertion sort can be very efficient.

---

# 4. Merge Sort

## Core idea

Merge Sort uses **divide and conquer**.

1. Divide the array into halves.
2. Recursively sort both halves.
3. Merge the two sorted halves.

Example:

```text
[8, 3, 5, 4, 7, 6, 1, 2]

          divide
             ↓
[8,3,5,4]       [7,6,1,2]

      ↓                ↓

[3,8] [4,5]       [6,7] [1,2]

      ↓                ↓

[3,4,5,8]         [1,2,6,7]

             ↓

[1,2,3,4,5,6,7,8]
```

## Why O(n log n)?

There are:

```text
log₂ n
```

levels of recursion.

At each level, merging processes:

```text
O(n)
```

elements.

Therefore:

```text
O(n) × O(log n)
= O(n log n)
```

## Java Implementation

```java
static void mergeSort(int[] a) {
    mergeSort(a, 0, a.length - 1);
}

static void mergeSort(int[] a, int left, int right) {
    if (left >= right) {
        return;
    }

    int mid = left + (right - left) / 2;

    mergeSort(a, left, mid);
    mergeSort(a, mid + 1, right);

    merge(a, left, mid, right);
}

static void merge(int[] a, int left, int mid, int right) {
    int[] temp = new int[right - left + 1];

    int i = left;
    int j = mid + 1;
    int k = 0;

    while (i <= mid && j <= right) {
        if (a[i] <= a[j]) {
            temp[k++] = a[i++];
        } else {
            temp[k++] = a[j++];
        }
    }

    while (i <= mid) {
        temp[k++] = a[i++];
    }

    while (j <= right) {
        temp[k++] = a[j++];
    }

    for (int p = 0; p < temp.length; p++) {
        a[left + p] = temp[p];
    }
}
```

## Complexity

```text
Best    O(n log n)
Average O(n log n)
Worst   O(n log n)
```

Typical array implementation:

```text
Auxiliary space = O(n)
```

Recursion stack:

```text
O(log n)
```

## Stability

Merge Sort can be stable.

Critical detail:

```java
if (a[i] <= a[j])
```

Choosing the left element when equal preserves the relative order of equal elements.

## Applications

Merge Sort is particularly useful for:

- Linked lists.
- Counting inversions.
- External sorting.
- Stable sorting.
- Divide-and-conquer problems.

---

# 5. Counting Inversions Using Merge Sort

An inversion is:

```text
i < j
and
a[i] > a[j]
```

Example:

```text
[2, 4, 1, 3, 5]
```

Inversions:

```text
(2,1)
(4,1)
(4,3)
```

Answer:

```text
3
```

A brute-force solution takes:

```text
O(n²)
```

Merge Sort can count inversions in:

```text
O(n log n)
```

## Key observation

During merge, suppose:

```text
left half  = [2, 4]
right half = [1, 3]
```

When `1` is selected before remaining left elements:

```text
2 > 1
4 > 1
```

Therefore all remaining elements in the left half form inversions with `1`.

Add:

```text
mid - i + 1
```

## Java

```java
static long countInversions(int[] a) {
    return sortAndCount(a, 0, a.length - 1);
}

static long sortAndCount(int[] a, int left, int right) {
    if (left >= right) {
        return 0;
    }

    int mid = left + (right - left) / 2;

    long count = 0;

    count += sortAndCount(a, left, mid);
    count += sortAndCount(a, mid + 1, right);
    count += mergeAndCount(a, left, mid, right);

    return count;
}

static long mergeAndCount(int[] a, int left, int mid, int right) {
    int[] temp = new int[right - left + 1];

    int i = left;
    int j = mid + 1;
    int k = 0;

    long count = 0;

    while (i <= mid && j <= right) {
        if (a[i] <= a[j]) {
            temp[k++] = a[i++];
        } else {
            temp[k++] = a[j++];
            count += mid - i + 1L;
        }
    }

    while (i <= mid) {
        temp[k++] = a[i++];
    }

    while (j <= right) {
        temp[k++] = a[j++];
    }

    for (int p = 0; p < temp.length; p++) {
        a[left + p] = temp[p];
    }

    return count;
}
```

Use `long` because the number of inversions can be:

```text
n(n - 1) / 2
```

which may exceed the range of `int`.

---

# 6. Quick Sort

## Core idea

Quick Sort uses divide and conquer.

1. Choose a pivot.
2. Partition the array around the pivot.
3. Recursively sort the two sides.

After partitioning:

```text
elements <= pivot | pivot | elements >= pivot
```

The exact partition invariant depends on the implementation.

---

# 6.1 Lomuto Partition

Common form:

```text
pivot = a[right]
```

Move elements smaller than or equal to the pivot to the left.

## Java

```java
static void quickSort(int[] a) {
    quickSort(a, 0, a.length - 1);
}

static void quickSort(int[] a, int low, int high) {
    if (low >= high) {
        return;
    }

    int pivotIndex = partition(a, low, high);

    quickSort(a, low, pivotIndex - 1);
    quickSort(a, pivotIndex + 1, high);
}

static int partition(int[] a, int low, int high) {
    int pivot = a[high];
    int i = low;

    for (int j = low; j < high; j++) {
        if (a[j] <= pivot) {
            int temp = a[i];
            a[i] = a[j];
            a[j] = temp;
            i++;
        }
    }

    int temp = a[i];
    a[i] = a[high];
    a[high] = temp;

    return i;
}
```

---

# 6.2 Complexity

Average:

```text
O(n log n)
```

Worst case:

```text
O(n²)
```

The worst case can occur when partitions are extremely unbalanced.

Example:

```text
[1,2,3,4,5,6]
```

if the pivot repeatedly becomes the smallest/largest element.

## Space

Recursive stack:

```text
Average: O(log n)
Worst:   O(n)
```

Quick Sort is generally considered in-place when the partition itself uses O(1) auxiliary space, ignoring recursion stack.

---

# 6.3 Pivot Selection

Possible strategies:

- First element.
- Last element.
- Middle element.
- Random pivot.
- Median-of-three.

Randomization helps reduce the likelihood of repeatedly getting pathological partitions on adversarial input.

---

# 6.4 Quick Sort vs Merge Sort

| Property | Quick Sort | Merge Sort |
|---|---|---|
| Average time | O(n log n) | O(n log n) |
| Worst time | O(n²) | O(n log n) |
| Typical array auxiliary space | O(log n) average | O(n) |
| Stable | Usually No | Yes |
| In-place | Yes, typical implementation | No, typical array implementation |
| Cache locality | Often good | Good |
| Linked lists | Less natural | Excellent |

### Interview rule

If asked:

> Which sorting algorithm guarantees O(n log n) worst-case time?

Answer:

```text
Merge Sort or Heap Sort
```

not ordinary Quick Sort.

---

# 7. Heap Sort

Heap Sort uses a binary heap.

For ascending order:

```text
Build a max heap.
Repeatedly move the maximum element to the end.
Restore heap property.
```

## Max Heap Property

For every node:

```text
parent >= children
```

For a zero-indexed array:

```text
left child  = 2*i + 1
right child = 2*i + 2
parent      = (i - 1) / 2
```

---

# 7.1 Heap Sort Java

```java
static void heapSort(int[] a) {
    int n = a.length;

    // Build max heap.
    for (int i = n / 2 - 1; i >= 0; i--) {
        heapify(a, n, i);
    }

    // Extract maximum repeatedly.
    for (int end = n - 1; end > 0; end--) {
        int temp = a[0];
        a[0] = a[end];
        a[end] = temp;

        heapify(a, end, 0);
    }
}

static void heapify(int[] a, int heapSize, int root) {
    while (true) {
        int largest = root;
        int left = 2 * root + 1;
        int right = 2 * root + 2;

        if (left < heapSize && a[left] > a[largest]) {
            largest = left;
        }

        if (right < heapSize && a[right] > a[largest]) {
            largest = right;
        }

        if (largest == root) {
            break;
        }

        int temp = a[root];
        a[root] = a[largest];
        a[largest] = temp;

        root = largest;
    }
}
```

## Complexity

```text
Build heap: O(n)
Each extraction: O(log n)
n extractions: O(n log n)

Total: O(n log n)
```

Best, average, worst:

```text
O(n log n)
```

Auxiliary space:

```text
O(1)
```

Typical Heap Sort is:

- In-place.
- Unstable.
- Worst-case O(n log n).

---

# 8. Counting Sort

Counting Sort is a **non-comparison sorting algorithm**.

It is useful when values belong to a relatively small integer range.

Example:

```text
[4, 2, 2, 8, 3, 3, 1]
```

Count frequencies:

```text
1 → 1
2 → 2
3 → 2
4 → 1
8 → 1
```

Then reconstruct the sorted array.

---

# 8.1 Basic Counting Sort

For non-negative integers:

```java
static void countingSort(int[] a) {
    if (a.length == 0) {
        return;
    }

    int max = a[0];

    for (int x : a) {
        max = Math.max(max, x);
    }

    int[] count = new int[max + 1];

    for (int x : a) {
        count[x]++;
    }

    int index = 0;

    for (int value = 0; value <= max; value++) {
        while (count[value] > 0) {
            a[index++] = value;
            count[value]--;
        }
    }
}
```

Complexity:

```text
O(n + k)
```

where:

```text
k = maxValue + 1
```

This version is not appropriate when `k` is enormous relative to `n`.

Example:

```text
[1, 2, 1000000000]
```

Creating an array of size one billion is wasteful.

---

# 8.2 Counting Sort With Negative Numbers

Use:

```text
min
max
range = max - min + 1
```

Map:

```text
value → value - min
```

Example:

```text
values = [-3, -1, -3, 2]

min = -3

-3 → 0
-1 → 2
 2 → 5
```

## Java

```java
static void countingSortWithNegatives(int[] a) {
    if (a.length == 0) {
        return;
    }

    int min = a[0];
    int max = a[0];

    for (int x : a) {
        min = Math.min(min, x);
        max = Math.max(max, x);
    }

    long rangeLong = (long) max - min + 1;

    if (rangeLong > Integer.MAX_VALUE) {
        throw new IllegalArgumentException("Range too large");
    }

    int range = (int) rangeLong;
    int[] count = new int[range];

    for (int x : a) {
        count[x - min]++;
    }

    int index = 0;

    for (int i = 0; i < range; i++) {
        while (count[i]-- > 0) {
            a[index++] = i + min;
        }
    }
}
```

---

# 8.3 Stable Counting Sort

A stable counting sort uses cumulative counts and places elements into an output array.

For an element `x`, the cumulative count tells the final region where `x` belongs.

To preserve stability, process the original array from **right to left**.

General structure:

```java
static void stableCountingSort(int[] a) {
    if (a.length == 0) {
        return;
    }

    int max = a[0];

    for (int x : a) {
        max = Math.max(max, x);
    }

    int[] count = new int[max + 1];

    for (int x : a) {
        count[x]++;
    }

    for (int i = 1; i < count.length; i++) {
        count[i] += count[i - 1];
    }

    int[] output = new int[a.length];

    for (int i = a.length - 1; i >= 0; i--) {
        int x = a[i];
        output[count[x] - 1] = x;
        count[x]--;
    }

    System.arraycopy(output, 0, a, 0, a.length);
}
```

Stable counting sort is important because Radix Sort relies on a stable digit-sorting operation.

---

# 9. Radix Sort

Radix Sort sorts numbers digit by digit.

For decimal numbers:

```text
ones
tens
hundreds
thousands
...
```

A stable sorting algorithm is applied at every digit.

Example:

```text
[170, 45, 75, 90, 802, 24, 2, 66]

After ones:
[170, 90, 802, 2, 24, 45, 75, 66]

After tens:
...

After hundreds:
[2, 24, 45, 66, 75, 90, 170, 802]
```

---

# 9.1 LSD Radix Sort

LSD = Least Significant Digit first.

For base 10:

```text
ones → tens → hundreds → ...
```

Each pass must be stable.

## Java for non-negative integers

```java
static void radixSort(int[] a) {
    if (a.length == 0) {
        return;
    }

    int max = a[0];

    for (int x : a) {
        if (x < 0) {
            throw new IllegalArgumentException(
                "This implementation expects non-negative integers."
            );
        }
        max = Math.max(max, x);
    }

    for (int exp = 1; max / exp > 0; exp *= 10) {
        countingSortByDigit(a, exp);

        if (exp > Integer.MAX_VALUE / 10) {
            break;
        }
    }
}

static void countingSortByDigit(int[] a, int exp) {
    int[] count = new int[10];
    int[] output = new int[a.length];

    for (int x : a) {
        int digit = (x / exp) % 10;
        count[digit]++;
    }

    for (int i = 1; i < 10; i++) {
        count[i] += count[i - 1];
    }

    // Right to left preserves stability.
    for (int i = a.length - 1; i >= 0; i--) {
        int x = a[i];
        int digit = (x / exp) % 10;

        output[count[digit] - 1] = x;
        count[digit]--;
    }

    System.arraycopy(output, 0, a, 0, a.length);
}
```

## Complexity

If:

```text
d = number of digits
k = radix/base
```

then:

```text
O(d(n + k))
```

For fixed-width integers, this can behave close to linear time.

## Important limitation

Radix Sort is not universally superior to comparison sorting.

It depends on:

- Representation of keys.
- Number of digits.
- Chosen base.
- Stable digit sort.
- Memory requirements.

---

# 10. Bucket Sort

Bucket Sort distributes elements into buckets.

General idea:

```text
1. Create buckets.
2. Place each element into an appropriate bucket.
3. Sort each bucket.
4. Concatenate buckets.
```

Example for values in `[0, 1)`:

```text
Input:
[0.42, 0.32, 0.23, 0.52, 0.25, 0.47, 0.51]

Buckets:
0.0–0.1
0.1–0.2
0.2–0.3 → 0.23, 0.25
0.3–0.4 → 0.32
0.4–0.5 → 0.42, 0.47
0.5–0.6 → 0.52, 0.51

Sort each bucket and concatenate.
```

## Java Example

```java
static void bucketSort(double[] a) {
    int n = a.length;

    if (n <= 1) {
        return;
    }

    @SuppressWarnings("unchecked")
    ArrayList<Double>[] buckets = new ArrayList[n];

    for (int i = 0; i < n; i++) {
        buckets[i] = new ArrayList<>();
    }

    for (double x : a) {
        int index = (int) (x * n);

        // Assumes 0 <= x < 1.
        index = Math.min(index, n - 1);

        buckets[index].add(x);
    }

    for (ArrayList<Double> bucket : buckets) {
        bucket.sort(Double::compare);
    }

    int index = 0;

    for (ArrayList<Double> bucket : buckets) {
        for (double x : bucket) {
            a[index++] = x;
        }
    }
}
```

## Complexity

Under a favorable/uniform distribution:

```text
Expected: O(n + k)
```

But poor distribution can produce large buckets.

If all elements go into one bucket and the bucket is sorted using a quadratic algorithm, the total can approach:

```text
O(n²)
```

### Key idea

Bucket Sort's performance depends strongly on the distribution of values.

---

# 11. Stable vs Unstable Sorting

## Stable sorting

A sorting algorithm is stable if equal-key elements retain their relative order.

Example:

```text
[(90,A), (80,B), (90,C)]
```

Sort by score:

```text
[(80,B), (90,A), (90,C)]
```

`A` remains before `C`.

## Common stable algorithms

- Bubble Sort.
- Insertion Sort.
- Merge Sort.
- Counting Sort, when implemented stably.
- Radix Sort, when each digit pass is stable.

## Common unstable algorithms

- Selection Sort.
- Quick Sort.
- Heap Sort.

### Why stability matters

Suppose employees are already sorted by:

```text
name
```

Then we stable-sort by:

```text
department
```

Employees within the same department retain their previous name ordering.

This makes multi-stage sorting possible.

---

# 12. In-Place Sorting

An algorithm is commonly called in-place when it uses O(1) or small constant auxiliary memory apart from recursion/system stack considerations.

Examples:

```text
Bubble Sort      → O(1)
Selection Sort   → O(1)
Insertion Sort   → O(1)
Heap Sort        → O(1)
Quick Sort       → O(log n) recursion on average
```

Merge Sort on arrays usually requires:

```text
O(n)
```

auxiliary memory.

### Important interview nuance

Do not blindly say:

> Quick Sort is O(1) space.

Recursive calls require stack space.

More accurate:

```text
Auxiliary partition memory: O(1)
Average recursion stack:    O(log n)
Worst recursion stack:      O(n)
```

---

# 13. Comparison vs Non-Comparison Sorting

## Comparison sorting

Ordering decisions are based on comparisons such as:

```text
a < b
a > b
a <= b
```

Examples:

- Bubble Sort.
- Selection Sort.
- Insertion Sort.
- Merge Sort.
- Quick Sort.
- Heap Sort.

For general comparison sorting, there is a lower bound:

```text
Ω(n log n)
```

in the worst/decision-tree sense for sorting arbitrary distinct elements.

Therefore a comparison-based algorithm cannot generally guarantee linear time for arbitrary keys.

---

# 13.1 Non-Comparison Sorting

Non-comparison algorithms exploit properties of the keys.

Examples:

- Counting Sort.
- Radix Sort.
- Bucket Sort.

They can beat the comparison-sorting lower bound under appropriate assumptions.

Example:

```text
Counting Sort:
O(n + k)
```

If `k = O(n)`, this becomes:

```text
O(n)
```

But this does not contradict the comparison lower bound because Counting Sort is not comparison-based.

---

# 14. Custom Comparator

In Java, custom ordering is usually expressed with `Comparator`.

Example:

```java
Integer[] a = {5, 2, 8, 1};

Arrays.sort(a, (x, y) -> Integer.compare(y, x));
```

Result:

```text
[8, 5, 2, 1]
```

Prefer:

```java
Integer.compare(y, x)
```

over:

```java
y - x
```

because subtraction can overflow.

---

# 14.1 Multiple Criteria

Sort employees:

1. Higher salary first.
2. If salary is equal, lower age first.
3. If age is equal, name alphabetically.

```java
employees.sort(
    Comparator.comparingInt(Employee::getSalary)
              .reversed()
              .thenComparingInt(Employee::getAge)
              .thenComparing(Employee::getName)
);
```

The exact API depends on the object's fields and types, but the pattern is:

```text
primary key
→ secondary key
→ tertiary key
```

---

# 14.2 Comparator Using Arrays

Suppose:

```java
int[][] intervals = {
    {5, 7},
    {1, 3},
    {2, 4}
};
```

Sort by start:

```java
Arrays.sort(intervals, (a, b) -> Integer.compare(a[0], b[0]));
```

Sort by end:

```java
Arrays.sort(intervals, (a, b) -> Integer.compare(a[1], b[1]));
```

---

# 15. Sorting Objects

Java provides different APIs depending on the collection/data type.

## Arrays of objects

```java
Student[] students = ...;

Arrays.sort(
    students,
    Comparator.comparingInt(Student::getMarks)
);
```

## Lists

```java
List<Student> students = ...;

students.sort(
    Comparator.comparingInt(Student::getMarks)
);
```

## Reverse order

```java
students.sort(
    Comparator.comparingInt(Student::getMarks).reversed()
);
```

## Multiple fields

```java
students.sort(
    Comparator.comparingInt(Student::getMarks)
              .reversed()
              .thenComparing(Student::getName)
);
```

### Primitive arrays

For primitive arrays:

```java
int[] a = {4, 2, 7, 1};

Arrays.sort(a);
```

A comparator cannot directly be supplied to `Arrays.sort(int[])`.

If custom object ordering is required, use objects such as:

```java
Integer[]
```

or a custom class.

---

# 16. Sorting Intervals

Interval problems are one of the most important uses of sorting in interviews.

Represent:

```text
[start, end]
```

Example:

```text
[1,3]
[2,6]
[8,10]
[15,18]
```

Most interval problems begin by sorting by:

```text
start time
```

---

# 16.1 Merge Intervals

## Problem

Merge overlapping intervals.

Input:

```text
[[1,3], [2,6], [8,10], [9,12]]
```

Output:

```text
[[1,6], [8,12]]
```

## Approach

1. Sort by start.
2. Maintain the current merged interval.
3. If the next interval overlaps, extend the end.
4. Otherwise, save the current interval and start a new one.

## Java

```java
static int[][] mergeIntervals(int[][] intervals) {
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
Sorting: O(n log n)
Scan:    O(n)

Total:   O(n log n)
```

---

# 16.2 Insert Interval

Given sorted non-overlapping intervals, insert a new interval and merge if necessary.

Example:

```text
Intervals:
[[1,3], [6,9]]

New:
[2,5]
```

Result:

```text
[[1,5], [6,9]]
```

Three phases:

```text
1. Intervals completely before new interval.
2. Overlapping intervals.
3. Intervals completely after new interval.
```

This problem does not necessarily require sorting again because the original intervals are already sorted.

---

# 16.3 Meeting Rooms

Given meeting intervals, determine whether a person can attend all meetings.

Example:

```text
[0,30]
[5,10]
[15,20]
```

Sort by start.

If:

```text
current.start < previous.end
```

there is an overlap.

For the example:

```text
[0,30]
[5,10]
```

overlap exists.

Therefore:

```text
false
```

---

# 16.4 Minimum Meeting Rooms

Given intervals, find the minimum number of rooms required.

Approach 1:

- Sort intervals by start.
- Use a min-heap containing current meeting end times.
- Remove meetings that have ended.
- Add current meeting's end.
- Heap size = rooms currently required.

## Java

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
        if (!minHeap.isEmpty() &&
            minHeap.peek() <= interval[0]) {
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
Heap operations: O(n log n)

Total: O(n log n)
Space: O(n)
```

---

# 16.5 Alternative: Sweep Line

Separate starts and ends:

```text
starts = sorted start times
ends   = sorted end times
```

Use two pointers.

If:

```text
starts[i] < ends[j]
```

a new room is needed.

Otherwise a room becomes free.

This produces an O(n log n) solution with sorting and can be useful when only event counts matter.

---

# 17. Sorting + Two Pointers

Sorting is frequently used before two-pointer algorithms.

Example:

> Find whether an array contains two values whose sum is `target`.

After sorting:

```text
left = 0
right = n - 1
```

If:

```text
a[left] + a[right] < target
```

move:

```text
left++
```

If:

```text
sum > target
```

move:

```text
right--
```

If equal:

```text
found
```

Complexity:

```text
Sort: O(n log n)
Two pointers: O(n)

Total: O(n log n)
```

---

# 18. Sorting + Greedy

Sorting often reveals the order in which greedy decisions should be made.

Classic examples:

- Activity selection.
- Interval scheduling.
- Meeting scheduling.
- Minimum arrows to burst balloons.
- Assign cookies.
- Job sequencing variants.
- Fractional knapsack.
- Merge intervals.

The key question is:

> What ordering makes the greedy choice locally optimal?

---

# 19. Sorting + Hashing

A problem may combine:

```text
sorting + hashmap
```

Example:

> Find duplicate values.

Possible approaches:

### HashSet

```text
O(n) expected time
O(n) space
```

### Sort first

```text
O(n log n)
O(1) or O(log n) extra depending on sorting
```

Then duplicates become adjacent.

This trade-off is important in interviews:

```text
Time vs space
```

---

# 20. Sorting + Binary Search

After sorting:

```text
O(n log n)
```

binary searches can often answer repeated queries in:

```text
O(log n)
```

This pattern is common in:

- Count values in a range.
- Find insertion positions.
- Closest values.
- Pair-sum variants.
- Scheduling.
- Coordinate-based problems.

---

# 21. Sorting by End Time

Many interval scheduling problems use:

```text
sort by end time
```

rather than start time.

For maximum number of non-overlapping intervals:

```text
Choose the interval that finishes earliest.
```

Why?

Finishing early leaves the maximum remaining space for future intervals.

This is a core greedy pattern.

---

# 22. Sorting by Custom Derived Key

Sometimes the required order is not a direct field.

Example:

> Sort numbers so their concatenation forms the largest possible number.

For:

```text
[3, 30, 34, 5, 9]
```

compare:

```text
"3" + "30" = "330"
"30" + "3" = "303"
```

Since:

```text
330 > 303
```

`3` should come before `30`.

Comparator:

```java
Arrays.sort(
    nums,
    (a, b) -> (b + a).compareTo(a + b)
);
```

When numbers are represented as strings:

```java
Arrays.sort(
    numbers,
    (a, b) -> (b + a).compareTo(a + b)
);
```

This is a classic custom-comparator interview pattern.

---

# 23. Sorting Strings

Java:

```java
String[] words = {"banana", "apple", "cat"};

Arrays.sort(words);
```

Lexicographic order:

```text
apple
banana
cat
```

Case-sensitive ordering depends on Java's `String.compareTo`.

Custom length ordering:

```java
Arrays.sort(
    words,
    Comparator.comparingInt(String::length)
);
```

Length descending:

```java
Arrays.sort(
    words,
    Comparator.comparingInt(String::length).reversed()
);
```

Length, then lexicographic:

```java
Arrays.sort(
    words,
    Comparator.comparingInt(String::length)
          .thenComparing(String::compareTo)
);
```

---

# 24. Common Sorting Bugs

## Bug 1: Comparator subtraction

Bad:

```java
(a, b) -> a[0] - b[0]
```

Potential integer overflow.

Prefer:

```java
(a, b) -> Integer.compare(a[0], b[0])
```

---

## Bug 2: Quick Sort recursion boundaries

After partitioning at `p`:

```java
quickSort(a, low, p - 1);
quickSort(a, p + 1, high);
```

Do not include the pivot again.

---

## Bug 3: Merge boundaries

Be precise about:

```text
left
mid
right
```

If using inclusive boundaries:

```text
[left, mid]
[mid + 1, right]
```

---

## Bug 4: Forgetting stability in Radix Sort

Every digit pass must preserve the relative ordering established by previous passes.

---

## Bug 5: Counting Sort range explosion

Do not allocate:

```text
count[max + 1]
```

when:

```text
max - min
```

is enormous.

---

## Bug 6: Assuming Bucket Sort is always O(n)

Bucket Sort's good performance depends on assumptions about distribution and bucket sorting.

---

## Bug 7: Using int for inversion count

Maximum inversions:

```text
n(n - 1) / 2
```

Use:

```java
long
```

when necessary.

---

# 25. Java Sorting Cheat Sheet

## Primitive array

```java
int[] a = {5, 2, 8, 1};

Arrays.sort(a);
```

## Object array

```java
Integer[] a = {5, 2, 8, 1};

Arrays.sort(a, Comparator.reverseOrder());
```

## List

```java
List<Integer> list = new ArrayList<>();

Collections.sort(list);
```

or:

```java
list.sort(Integer::compareTo);
```

## Reverse list

```java
list.sort(Comparator.reverseOrder());
```

## 2D array by first column

```java
Arrays.sort(
    intervals,
    (a, b) -> Integer.compare(a[0], b[0])
);
```

## 2D array by second column descending

```java
Arrays.sort(
    intervals,
    (a, b) -> Integer.compare(b[1], a[1])
);
```

## Multiple criteria

```java
Arrays.sort(
    students,
    Comparator.comparingInt(Student::getMarks)
              .reversed()
              .thenComparing(Student::getName)
);
```

---

# 26. Choosing the Right Sorting Algorithm

| Situation | Good Choice |
|---|---|
| Tiny/nearly sorted data | Insertion Sort |
| Need stable O(n log n) | Merge Sort |
| Need worst-case O(n log n) + O(1) auxiliary | Heap Sort |
| Average-case fast array sorting | Quick Sort / library sort |
| Small integer range | Counting Sort |
| Fixed-length integer/string keys | Radix Sort |
| Uniformly distributed values | Bucket Sort |
| Linked list | Merge Sort |
| Need few swaps | Selection Sort |
| Teaching adjacent swaps | Bubble Sort |

Do not memorize this table without understanding the assumptions.

---

# 27. Decision Framework

When given a sorting problem, ask:

### Question 1: Do I actually need to sort?

Maybe a:

- HashMap.
- HashSet.
- Heap.
- Counting array.
- Monotonic structure.
- Binary search over an answer.

is better.

### Question 2: What are the keys?

Are they:

- General comparable objects?
- Small integers?
- Fixed-width integers?
- Floating-point values?
- Strings?
- Intervals?

### Question 3: Does stability matter?

If equal keys must preserve previous order:

```text
Use a stable algorithm.
```

### Question 4: Is extra memory restricted?

If yes:

```text
Heap Sort / Quick Sort / Insertion Sort
```

may be relevant.

### Question 5: Is worst-case complexity important?

Need guaranteed:

```text
O(n log n)
```

Consider:

```text
Merge Sort
Heap Sort
```

### Question 6: Can the key domain be exploited?

If yes:

```text
Counting / Radix / Bucket
```

may beat comparison sorting.

---

# 28. Important Interview Patterns

## Pattern 1: Sort → Two Pointers

```text
sort
left/right pointers
```

Used for:

- Two Sum variants.
- 3Sum.
- 4Sum.
- Closest pair.
- Duplicate handling.

---

## Pattern 2: Sort → Merge Intervals

```text
sort by start
scan
merge overlaps
```

---

## Pattern 3: Sort → Greedy

```text
sort by the correct criterion
take locally optimal choices
```

---

## Pattern 4: Sort → Binary Search

```text
sort
binary search repeatedly
```

Useful for many queries.

---

## Pattern 5: Sort → Count Frequencies

After sorting:

```text
equal values become adjacent
```

This can simplify:

- Frequency counting.
- Duplicate detection.
- Grouping.
- Run-length processing.

---

## Pattern 6: Custom Comparator

Transform the problem into:

```text
Define exactly what it means for A to come before B.
```

Then encode that relation in a comparator.

---

# 29. Comparator Contract

A comparator should define a consistent ordering.

For two objects:

```text
compare(a, b) < 0
```

means:

```text
a comes before b
```

```text
compare(a, b) == 0
```

means they are considered equivalent under the comparator.

```text
compare(a, b) > 0
```

means:

```text
a comes after b
```

A comparator should behave consistently and should not violate the ordering assumptions expected by the sorting implementation.

### Important

Do not write arbitrary comparators that are not transitive.

Bad comparator logic can cause incorrect behavior or runtime failures in sorting algorithms.

---

# 30. Stability Example With Objects

Suppose:

```text
Student:
(name, marks)
```

Initial order:

```text
(Alice, 90)
(Bob, 80)
(Charlie, 90)
```

Stable sort by marks ascending:

```text
(Bob, 80)
(Alice, 90)
(Charlie, 90)
```

The two students with `90` remain:

```text
Alice before Charlie
```

If the sorting algorithm is unstable, their relative order is not guaranteed.

---

# 31. Sorting and the Lower Bound

For comparison-based sorting, there are:

```text
n!
```

possible permutations of `n` distinct elements.

A comparison decision tree must distinguish among these possibilities.

A binary decision tree of height `h` has at most:

```text
2^h
```

leaves.

Therefore:

```text
2^h >= n!
```

Taking logarithms:

```text
h >= log₂(n!)
```

Using:

```text
log(n!) = Θ(n log n)
```

we obtain:

```text
h = Ω(n log n)
```

Therefore comparison-based sorting has a lower bound of:

```text
Ω(n log n)
```

in the general case.

This is a major GATE/theory concept.

---

# 32. Why Non-Comparison Sorting Can Beat O(n log n)

The lower bound applies to comparison-based sorting.

Counting Sort does not ask:

```text
Is x < y?
```

Instead it directly indexes by key:

```text
count[x]
```

Radix Sort examines digits.

Bucket Sort uses ranges/buckets.

Therefore they exploit additional assumptions about the input domain.

---

# 33. Merge Sort Recurrence

For Merge Sort:

```text
T(n) = 2T(n/2) + O(n)
```

By the Master Theorem:

```text
a = 2
b = 2
f(n) = O(n)

n^(log_b a)
= n^(log₂2)
= n
```

Therefore:

```text
T(n) = O(n log n)
```

---

# 34. Quick Sort Recurrences

Balanced partition:

```text
T(n) = 2T(n/2) + O(n)
```

Therefore:

```text
O(n log n)
```

Worst partition:

```text
T(n) = T(n - 1) + O(n)
```

Therefore:

```text
O(n²)
```

This is why pivot quality matters.

---

# 35. Why Build Heap Is O(n), Not O(n log n)

A common interview question.

Naively, one might think:

```text
n nodes × O(log n)
= O(n log n)
```

But most nodes are near the leaves and require very little heapification.

The total work is bounded by:

```text
O(n)
```

This is why bottom-up heap construction is linear.

---

# 36. Sorting Intervals: Start vs End

Memorize the purpose, not just the comparator.

### Merge overlapping intervals

Usually:

```text
sort by start
```

### Maximum non-overlapping intervals

Usually:

```text
sort by end
```

### Meeting room allocation

Usually:

```text
sort by start + min heap
```

or:

```text
sort starts and ends separately
```

The sorting key is determined by the invariant required by the problem.

---

# 37. Advanced Sorting-Based Problems

Once the core algorithms are mastered, practice:

- Count inversions.
- Reverse pairs.
- 3Sum.
- 4Sum.
- Largest number.
- Merge intervals.
- Insert interval.
- Non-overlapping intervals.
- Meeting rooms.
- Minimum meeting rooms.
- Maximum number of events.
- Sort colors.
- Kth largest/smallest.
- Top K frequent elements.
- Wiggle sort.
- Relative sort.
- Largest number.
- Custom object sorting.
- Coordinate compression.
- Sweep line.
- Counting-based frequency problems.

---

# 38. Coordinate Compression

Coordinate compression replaces large values with their rank/order.

Example:

```text
values:
[1000, 500000, 1000, 7000]
```

Unique sorted values:

```text
[1000, 7000, 500000]
```

Compressed:

```text
1000   → 0
7000   → 1
500000 → 2
```

This is useful when:

- Values are huge.
- Only relative ordering matters.
- Fenwick Trees/Segment Trees need a manageable index range.
- Intervals or points have large coordinates.

Typical complexity:

```text
O(n log n)
```

due to sorting.

---

# 39. Dutch National Flag / 3-Way Partition

A major sorting-related pattern is partitioning an array into three regions.

Example:

```text
[2,0,2,1,1,0]
```

Goal:

```text
[0,0,1,1,2,2]
```

Maintain:

```text
low
mid
high
```

Invariant:

```text
[0 ... low-1]   → 0
[low ... mid-1] → 1
[mid ... high]  → unknown
[high+1 ... n-1]→ 2
```

## Java

```java
static void sortColors(int[] nums) {
    int low = 0;
    int mid = 0;
    int high = nums.length - 1;

    while (mid <= high) {
        if (nums[mid] == 0) {
            swap(nums, low, mid);
            low++;
            mid++;
        } else if (nums[mid] == 1) {
            mid++;
        } else {
            swap(nums, mid, high);
            high--;
        }
    }
}

static void swap(int[] a, int i, int j) {
    int temp = a[i];
    a[i] = a[j];
    a[j] = temp;
}
```

Complexity:

```text
O(n) time
O(1) space
```

This is not a general-purpose replacement for sorting; it exploits exactly three known categories.

---

# 40. Selection vs Sorting

Sometimes the problem asks for:

```text
k-th smallest
k-th largest
```

Full sorting costs:

```text
O(n log n)
```

Alternatives:

- Heap → O(n log k)
- Quickselect → O(n) average
- Counting → O(n + k) under suitable constraints

This illustrates an important interview principle:

> Do not automatically sort when the problem only asks for partial ordering.

---

# 41. Java's Built-In Sort

In interviews, using:

```java
Arrays.sort(...)
```

is usually appropriate unless the interviewer explicitly asks you to implement sorting.

Know the distinction between:

```java
Arrays.sort(primitiveArray);
```

and:

```java
Arrays.sort(objectArray, comparator);
```

Also know that library sorting implementations are optimized production algorithms; their exact implementation details can vary by Java version and data type.

For interview questions requiring algorithm analysis, discuss the algorithm you are intentionally using rather than assuming every `Arrays.sort` call represents the same algorithm.

---

# 42. Sorting Problem-Solving Checklist

Before coding:

```text
1. Do I actually need full sorting?
2. What is n?
3. What is the key domain?
4. Is the data already partially sorted?
5. Does stability matter?
6. Is extra memory restricted?
7. Do I need guaranteed O(n log n)?
8. Can counting/radix/bucket exploit the keys?
9. Can sorting simplify the problem into two pointers?
10. Can sorting simplify interval processing?
11. Is a custom comparator required?
12. Could a heap/quickselect be better than full sorting?
```

---

# 43. Common Interview Questions

## Basic

1. Implement Bubble Sort.
2. Implement Selection Sort.
3. Implement Insertion Sort.
4. Compare their complexities.
5. Which is stable?
6. Which works well on nearly sorted data?

## Merge Sort

7. Implement Merge Sort.
8. Explain its O(n log n) complexity.
9. Why does Merge Sort need extra memory for arrays?
10. Count inversions using Merge Sort.
11. Why is Merge Sort useful for linked lists?

## Quick Sort

12. Implement partition.
13. Implement Quick Sort.
14. Explain worst-case O(n²).
15. How can pivot selection be improved?
16. Compare Quick Sort and Merge Sort.
17. Explain recursion-stack complexity.

## Heap Sort

18. Build a max heap.
19. Explain why build-heap is O(n).
20. Implement Heap Sort.
21. Compare Heap Sort and Quick Sort.

## Non-comparison

22. Implement Counting Sort.
23. Handle negative numbers.
24. Explain when Counting Sort becomes impractical.
25. Implement Radix Sort.
26. Why must Radix Sort use stable digit sorting?
27. Explain Bucket Sort and its distribution assumption.

## Concepts

28. Stable vs unstable.
29. In-place vs out-of-place.
30. Comparison vs non-comparison.
31. Custom comparator.
32. Sort objects by multiple criteria.
33. Sort intervals.
34. Merge intervals.
35. Minimum meeting rooms.
36. Sort + two pointers.
37. Sort + greedy.

---

# 44. LeetCode Roadmap

## Easy

Focus on:

- Basic sorting implementation.
- Sort array.
- Sort colors.
- Merge sorted structures.
- Relative ordering.
- Simple interval sorting.
- Custom comparator basics.

## Medium

Prioritize:

- Merge Intervals.
- Insert Interval.
- Non-overlapping Intervals.
- Meeting Rooms II-style problems.
- 3Sum.
- 4Sum.
- Sort Colors.
- Kth Largest Element.
- Top K Frequent Elements.
- Largest Number.
- Count inversions/reverse-pair style problems.
- Sorting with custom objects.
- Greedy problems where sorting reveals the correct order.

## Hard

Practice:

- Reverse Pairs.
- Count of Smaller Numbers After Self.
- Advanced interval scheduling.
- Complex sweep-line problems.
- Advanced custom comparator problems.
- Sorting combined with Fenwick Tree/Segment Tree.
- Advanced k-th/order-statistic problems.

---

# 45. GATE / CS Theory Focus

Know these precisely:

### Complexity

```text
Bubble:
Best O(n), Average/Worst O(n²)

Selection:
O(n²) all cases

Insertion:
Best O(n), Average/Worst O(n²)

Merge:
O(n log n) all cases

Quick:
Average O(n log n)
Worst O(n²)

Heap:
O(n log n) all cases

Counting:
O(n + k)

Radix:
O(d(n + k))

Bucket:
Expected linear under appropriate distribution assumptions
```

### Properties

Know:

```text
Stable
In-place
Adaptive
Comparison-based
Non-comparison-based
```

### Theory

Know:

```text
Ω(n log n) comparison-sorting lower bound
Merge Sort recurrence
Quick Sort balanced/worst recurrences
Heap construction O(n)
Counting Sort assumptions
Radix Sort stability requirement
```

---

# 46. GATE-Style Solved Questions

> These are **GATE-style practice questions**, not claims about a particular official GATE year.

---

## Question 1 — Comparison Sorting Lower Bound

For sorting `n` distinct elements using only comparisons, the asymptotic lower bound is:

A. Ω(n)

B. Ω(log n)

C. Ω(n log n)

D. Ω(n²)

### Solution

There are:

```text
n!
```

possible permutations.

A comparison decision tree must distinguish among all of them.

Thus:

```text
2^h >= n!
```

Taking logarithms:

```text
h >= log₂(n!)
```

and:

```text
log(n!) = Θ(n log n)
```

Therefore:

```text
Answer = C
```

---

## Question 2 — Stable Sorting

Consider:

```text
[(4,A), (2,B), (4,C), (1,D)]
```

The array is sorted by the first component.

Which output demonstrates stability?

A.

```text
[(1,D), (2,B), (4,A), (4,C)]
```

B.

```text
[(1,D), (2,B), (4,C), (4,A)]
```

C.

Both are equally stable.

D.

Neither is stable.

### Solution

The equal-key elements are:

```text
(4,A)
(4,C)
```

Their original order is:

```text
A before C
```

A stable sort must preserve that order.

Therefore:

```text
[(1,D), (2,B), (4,A), (4,C)]
```

is stable.

```text
Answer = A
```

---

## Question 3 — Merge Sort Recurrence

Consider:

```text
T(n) = 2T(n/2) + cn
```

where `c` is a positive constant.

What is the asymptotic complexity?

A. O(n)

B. O(log n)

C. O(n log n)

D. O(n²)

### Solution

Using the Master Theorem:

```text
a = 2
b = 2
f(n) = cn
```

Calculate:

```text
n^(log₂2) = n
```

Therefore:

```text
f(n) = Θ(n^(log_b a))
```

This is the balanced case.

Hence:

```text
T(n) = Θ(n log n)
```

```text
Answer = C
```

---

# 47. Serious Mastery Checklist

You should be able to explain and implement without looking up:

## Basic

- [ ] Bubble Sort
- [ ] Selection Sort
- [ ] Insertion Sort
- [ ] Best/average/worst cases
- [ ] Stable vs unstable
- [ ] In-place property

## Core Interview Algorithms

- [ ] Merge Sort
- [ ] Merge operation
- [ ] Merge Sort recurrence
- [ ] Count inversions
- [ ] Quick Sort
- [ ] Partition
- [ ] Pivot selection
- [ ] Quick Sort worst case
- [ ] Heap Sort
- [ ] Heapify
- [ ] Build heap in O(n)

## Non-comparison

- [ ] Counting Sort
- [ ] Counting Sort with negative values
- [ ] Stable Counting Sort
- [ ] Radix Sort
- [ ] Stable digit sorting
- [ ] Bucket Sort
- [ ] Distribution assumptions

## Java

- [ ] Arrays.sort
- [ ] List.sort
- [ ] Comparator
- [ ] Multiple criteria
- [ ] Primitive vs object arrays
- [ ] Comparator overflow avoidance

## Problem Patterns

- [ ] Sort + two pointers
- [ ] Sort + greedy
- [ ] Sort + binary search
- [ ] Sort + hashing
- [ ] Merge intervals
- [ ] Insert interval
- [ ] Meeting rooms
- [ ] Sweep line
- [ ] Coordinate compression
- [ ] Custom comparator
- [ ] Partial sorting / selection

---

# 48. Final Revision Sheet

```text
SORTING
│
├── Basic
│   ├── Bubble
│   ├── Selection
│   └── Insertion
│
├── O(n log n)
│   ├── Merge
│   ├── Quick (average)
│   └── Heap
│
├── Non-comparison
│   ├── Counting
│   ├── Radix
│   └── Bucket
│
├── Properties
│   ├── Stable
│   ├── Unstable
│   ├── In-place
│   └── Out-of-place
│
├── Java
│   ├── Arrays.sort
│   ├── List.sort
│   └── Comparator
│
└── Patterns
    ├── Sort + Two Pointers
    ├── Sort + Greedy
    ├── Sort + Binary Search
    ├── Sort + Hashing
    ├── Intervals
    ├── Sweep Line
    └── Coordinate Compression
```

### Complexity memory

```text
Bubble       → O(n²), stable, in-place
Selection    → O(n²), usually unstable, in-place
Insertion    → O(n²), stable, in-place
Merge        → O(n log n), stable, O(n) array space
Quick        → O(n log n) average, O(n²) worst, usually unstable
Heap         → O(n log n), unstable, in-place
Counting     → O(n + k)
Radix        → O(d(n + k))
Bucket       → expected O(n + k) under suitable distribution
```

### Highest-value interview lesson

Sorting is often not the final algorithm.

It is frequently the **transformation that exposes structure**:

```text
unsorted data
      ↓
    sort
      ↓
ordered structure
      ↓
two pointers / greedy / binary search / intervals / sweep line
      ↓
efficient solution
```

Master sorting not only as a collection of algorithms, but as a problem-solving primitive.
