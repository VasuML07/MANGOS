# FAANG DSA Mastery

## Overview
Markdown-first navigation and progress tracking for the DSA portion of the repository.

The repository is designed around FAANG/top-product interview preparation with Java, emphasizing pattern recognition, problem solving, and repeatable revision.

## DSA Dashboard

| Metric | Value |
|---|---:|
| Total Topics | 23 |
| Total Subtopics / H2 Sections | 1407 |
| Unique H2 Labels | 1184 |
| Source Pattern Labels | 82 |
| Unique Tracked LeetCode Problem Units | 230 |
| Easy | 55 |
| Medium | 72 |
| Hard | 23 |
| Difficulty TBD | 80 |
| CORE | 77 |
| IMPORTANT | 36 |
| OPTIONAL | 1 |
| Priority TBD | 116 |
| Not Started | 230* |
| Attempted | 0* |
| Solved Independently | 0* |
| Mastered | 0* |

> `*` Initial tracker state. Update these manually as problems are attempted.

## Goal

- Build reliable DSA problem-solving ability for FAANG and top product-company interviews.
- Become strong enough to transfer patterns to unfamiliar Medium/Hard problems.
- Maintain enough theory for GATE-style reasoning.
- Prefer understanding, derivation, invariants, and complexity analysis over memorization.

## Preparation Philosophy

1. Learn the underlying data structure or algorithm.
2. Identify the recurring problem-solving pattern.
3. Solve Easy problems to establish the template.
4. Spend most practice time on Medium problems.
5. Use Hard problems to test transfer and pattern composition.
6. Record mistakes rather than merely recording accepted submissions.
7. Treat mastery as the ability to solve a new variation independently.

## How This Repository Works

- `NN-topic.md` files contain the source learning material.
- `README.md` is the navigation layer.
- `TRACKING/dsa-progress.md` is the manually maintained progress database.
- The topic files are the source of truth for what has been generated.
- `TBD` is used where the source material does not provide enough metadata.

## DSA Roadmap

```text
01 Bit Manipulation Basics
02 Arrays
03 Strings
04 Hashing
05 Linked Lists
06 Stack
07 Queue and Deque
08 Binary Search
09 Sorting
10 Intervals
11 Heap / Priority Queue
12 Trees
13 Binary Search Trees
14 Trie
15 Graphs
16 Disjoint Set Union
17 Greedy Algorithms
18 Recursion and Backtracking
19 Dynamic Programming
20 Bit Manipulation
21 Math / Number Theory
22 Advanced Data Structures
23 Advanced Patterns
```

## Topic Index

| # | Topic | Problems | Easy | Medium | Hard | Core | Progress | File |
|---:|---|---:|---:|---:|---:|---:|---:|---|
| 1 | Bit Manipulation Basics | 7 | 6 | 1 | 0 | 4 | 0% | [01-bit-manipulation-basics.md](01-bit-manipulation-basics.md) |
| 2 | Arrays | 13 | 3 | 8 | 2 | 11 | 0% | [02-arrays.md](02-arrays.md) |
| 3 | Strings | 12 | 4 | 6 | 2 | 9 | 0% | [03-strings.md](03-strings.md) |
| 4 | Hashing | 16 | 4 | 9 | 3 | 9 | 0% | [04-hashing.md](04-hashing.md) |
| 5 | Linked Lists | 17 | 6 | 9 | 2 | 14 | 0% | [05-linked-lists.md](05-linked-lists.md) |
| 6 | Stack | 5 | 5 | 0 | 0 | 3 | 0% | [06-stack.md](06-stack.md) |
| 7 | Queue & Deque | 4 | 4 | 0 | 0 | 1 | 0% | [07-queue-and-deque.md](07-queue-and-deque.md) |
| 8 | Binary Search | 5 | 5 | 0 | 0 | 3 | 0% | [08-binary-search.md](08-binary-search.md) |
| 9 | 09 — Sorting | 12 | 1 | 8 | 3 | 2 | 0% | [09-sorting.md](09-sorting.md) |
| 10 | 10 — Intervals | 10 | 5 | 5 | 0 | 1 | 0% | [10-intervals.md](10-intervals.md) |
| 11 | 11 — Heap / Priority Queue | 0 | 0 | 0 | 0 | 0 | 0% | [11-heap-priority-queue.md](11-heap-priority-queue.md) |
| 12 | 12 — Trees | 25 | 10 | 10 | 5 | 0 | 0% | [12-trees.md](12-trees.md) |
| 13 | 13 — Binary Search Trees | 15 | 0 | 0 | 0 | 8 | 0% | [13-binary-search-trees.md](13-binary-search-trees.md) |
| 14 | 14 — Trie | 7 | 0 | 0 | 1 | 4 | 0% | [14-trie.md](14-trie.md) |
| 15 | 15 — Graphs | 16 | 0 | 2 | 0 | 6 | 0% | [15-graphs.md](15-graphs.md) |
| 16 | 16 — Disjoint Set Union (DSU) | 2 | 0 | 1 | 0 | 2 | 0% | [16-disjoint-set-union.md](16-disjoint-set-union.md) |
| 17 | Greedy Algorithms | 31 | 6 | 17 | 8 | 1 | 0% | [17-greedy-algorithms.md](17-greedy-algorithms.md) |
| 18 | Recursion & Backtracking | 8 | 5 | 0 | 0 | 2 | 0% | [18-recursion-and-backtracking.md](18-recursion-and-backtracking.md) |
| 19 | Dynamic Programming | 9 | 0 | 1 | 0 | 1 | 0% | [19-dynamic-programming.md](19-dynamic-programming.md) |
| 20 | Bit Manipulation | 10 | 6 | 0 | 0 | 4 | 0% | [20-bit-manipulation.md](20-bit-manipulation.md) |
| 21 | Math / Number Theory | 4 | 0 | 0 | 0 | 0 | 0% | [21-math-number-theory.md](21-math-number-theory.md) |
| 22 | Advanced Data Structures | 16 | 0 | 1 | 0 | 2 | 0% | [22-advanced-data-structures.md](22-advanced-data-structures.md) |
| 23 | Advanced Problem-Solving Patterns | 35 | 4 | 15 | 3 | 27 | 0% | [23-advanced-patterns.md](23-advanced-patterns.md) |

## Pattern Index

> Pattern labels are taken from the generated source files. Labels are preserved rather than silently merged, because the source uses different granularities (for example, `two pointers`, `opposite-direction two pointers`, and `sorting + two pointers`).

| Pattern | Source Mentions | Tracked Problems | Mastery |
|---|---:|---:|---:|
| Aggressive placement | 1 | 0 | 0% |
| Balance counting | 1 | 0 | 0% |
| Basic binary search | 1 | 1 | 0% |
| BFS | 4 | 0 | 0% |
| BFS + state | 1 | 0 | 0% |
| BFS / shortest path interpretation | 1 | 0 | 0% |
| BFS on implicit graph | 1 | 0 | 0% |
| BFS/DFS | 1 | 0 | 0% |
| Binary search | 2 | 1 | 0% |
| Binary search + counting | 1 | 0 | 0% |
| Binary search + feasibility | 1 | 0 | 0% |
| Binary search + greedy | 2 | 0 | 0% |
| Binary search + window | 1 | 0 | 0% |
| Binary search on answer | 3 | 1 | 0% |
| Binary search on slope | 1 | 0 | 0% |
| Binary search on speed | 1 | 0 | 0% |
| Binary search on time | 1 | 0 | 0% |
| Binary search partition | 1 | 0 | 0% |
| Boundary binary search | 1 | 0 | 0% |
| Boyer-Moore voting | 1 | 1 | 0% |
| canonical representation | 1 | 1 | 0% |
| Circular monotonic stack | 1 | 0 | 0% |
| Circular queue | 1 | 1 | 0% |
| cyclic placement / index mapping | 1 | 1 | 0% |
| DFS + memoization | 1 | 0 | 0% |
| DP + monotonic deque | 1 | 0 | 0% |
| expand around center | 1 | 1 | 0% |
| Expression stack | 2 | 0 | 0% |
| extract and rebuild bits | 1 | 1 | 0% |
| First true | 1 | 1 | 0% |
| fixed sliding window + frequency | 1 | 1 | 0% |
| fixed-size sliding window + frequency | 1 | 1 | 0% |
| Flattened binary search | 1 | 0 | 0% |
| Frequency + queue-style reasoning | 1 | 1 | 0% |
| frequency counting | 1 | 1 | 0% |
| Frequency/lookup map | 1 | 1 | 0% |
| HashSet + sequence starts | 1 | 1 | 0% |
| Histogram + monotonic stack | 1 | 0 | 0% |
| Kadane's algorithm | 1 | 1 | 0% |
| Lower bound | 1 | 1 | 0% |
| Lower/upper boundary | 1 | 0 | 0% |
| Monotonic decreasing stack | 1 | 0 | 0% |
| Monotonic deque | 2 | 0 | 0% |
| Monotonic reasoning + next permutation | 1 | 0 | 0% |
| Monotonic stack | 3 | 0 | 0% |
| Monotonic stack + HashMap | 1 | 1 | 0% |
| Monotonic stack / ordering | 1 | 0 | 0% |
| Monotonic stack / two pointers | 1 | 0 | 0% |
| Multi-source BFS | 2 | 0 | 0% |
| n & (n - 1) | 1 | 0 | 0% |
| one-set-bit detection | 1 | 1 | 0% |
| opposite-direction two pointers | 1 | 1 | 0% |
| pattern matching | 1 | 1 | 0% |
| per-bit counting | 1 | 1 | 0% |
| prefix + suffix | 1 | 1 | 0% |
| prefix comparison | 1 | 1 | 0% |
| prefix sum + hashmap | 1 | 1 | 0% |
| Queue | 1 | 1 | 0% |
| Queue / timestamps | 1 | 0 | 0% |
| Queue simulation | 3 | 1 | 0% |
| Queue/deque simulation | 1 | 0 | 0% |
| remove lowest set bit / recurrence | 1 | 1 | 0% |
| reversal / in-place | 1 | 1 | 0% |
| Rotated binary search | 1 | 0 | 0% |
| running minimum / greedy scan | 1 | 1 | 0% |
| sorting + scanning | 1 | 1 | 0% |
| sorting + two pointers | 1 | 1 | 0% |
| Stack | 3 | 1 | 0% |
| Stack + auxiliary state | 1 | 0 | 0% |
| Stack + expression parsing | 1 | 0 | 0% |
| Stack + frequency | 1 | 0 | 0% |
| Stack / DP | 1 | 0 | 0% |
| Stack parsing | 1 | 0 | 0% |
| Stack simulation | 4 | 3 | 0% |
| Trie | 1 | 1 | 0% |
| Trie + DFS | 1 | 1 | 0% |
| two pointers | 1 | 1 | 0% |
| two pointers / prefix-suffix maxima | 1 | 1 | 0% |
| Two stacks / nested simulation | 1 | 0 | 0% |
| Value-space binary search | 1 | 0 | 0% |
| variable sliding window | 2 | 2 | 0% |
| XOR cancellation | 2 | 2 | 0% |

## Problem Progress

- Global problem database: **230 unique problem units**.
- All initial statuses are `Not Started` / mastery `0` unless the source explicitly contained completion information. The generated source files contain no personal completion history, so no problem is marked solved.

## Difficulty Progress

| Difficulty | Total | Not Started | Attempted | Solved | Mastered |
|---|---:|---:|---:|---:|---:|
| Easy | 55 | 55 | 0 | 0 | 0 |
| Medium | 72 | 72 | 0 | 0 | 0 |
| Hard | 23 | 23 | 0 | 0 | 0 |
| TBD | 80 | 80 | 0 | 0 | 0 |

## Priority System

### CORE
Problems every serious candidate should know or that the source explicitly identifies as essential/core.

### IMPORTANT
Strongly recommended problems or problems explicitly marked high/priority in the source.

### OPTIONAL
Additional depth. Only explicitly marked optional entries are counted here.

### TBD
The source names the problem but does not provide a reliable priority label. Do not invent one.

Priority and difficulty are independent. A Hard problem can be CORE; an Easy problem can be IMPORTANT.

## Mastery System

| Level | Meaning |
|---:|---|
| 0 | Unseen / Not Started |
| 1 | Attempted |
| 2 | Solved With Help |
| 3 | Independently Solved |
| 4 | Can Explain |
| 5 | Pattern Mastery |

An accepted submission is not automatically mastery. Level 5 requires recognizing and applying the underlying pattern to a new variation.

## Daily 1-Hour Workflow

```text
5–10 min  → Concept / pattern review
30–40 min → Serious problem attempt
10–15 min → Solution analysis + mistake capture
```

Do not optimize for raw problem count. Optimize for independent reasoning and transfer.

## How to Use the Topic Files

1. Read the concept sections.
2. Study the Java template.
3. Attempt the selected problems without looking at the solution.
4. Record the outcome in `TRACKING/dsa-progress.md`.
5. Re-attempt problems that required hints or solutions.
6. Mark Level 5 only after successful transfer to a variation.

## How to Use the Tracker

- Update one problem row after every meaningful attempt.
- Record the main mistake in one sentence.
- Record time honestly.
- Increase mastery only when the corresponding definition is satisfied.
- Use the weak-area section to choose the next study target.
- Keep the revision queue manual; do not turn this into an automated scheduling system.

## Problem-Solving Rules

- Spend an initial period deriving the approach before opening a solution.
- State the invariant before implementation when the problem uses a maintained state.
- Check edge cases before coding.
- Derive complexity after the algorithm is fixed.
- If a pattern depends on an assumption, write the assumption down.
- Prefer the simplest data structure that satisfies the required operations.

## Revision System

Revise problems when they are forgotten, required help, contain a repeated mistake, or represent a high-value interview pattern.

Suggested sequence:

```text
Attempt → record mistake → re-solve → explain → solve variation → mastery
```

## Weak-Area System

Rank weaknesses using:
- repeated mistakes
- low confidence
- dependence on hints
- dependence on complete solutions
- inability to derive an approach
- poor Medium performance
- inability to recognize the pattern

## Progress Dashboard

All completion metrics begin at zero because the source files do not contain personal completion history. Update the tracker manually.

## Repository Structure

```text
01-bit-manipulation-basics.md
02-arrays.md
03-strings.md
04-hashing.md
05-linked-lists.md
06-stack.md
07-queue-and-deque.md
08-binary-search.md
09-sorting.md
10-intervals.md
11-heap-priority-queue.md
12-trees.md
13-binary-search-trees.md
14-trie.md
15-graphs.md
16-disjoint-set-union.md
17-greedy-algorithms.md
18-recursion-and-backtracking.md
19-dynamic-programming.md
20-bit-manipulation.md
21-math-number-theory.md
22-advanced-data-structures.md
23-advanced-patterns.md

README.md
TRACKING/
└── dsa-progress.md
```

## Recommended Study Order

Follow the numeric topic order initially. After the first pass, use weak-area and pattern data to revisit prerequisite topics.

Recommended dependency flow:

```text
Arrays / Strings / Hashing
        ↓
Linked Lists / Stack / Queue / Binary Search / Sorting / Intervals
        ↓
Heap / Trees / BST / Trie
        ↓
Graphs / DSU
        ↓
Greedy / Recursion / Backtracking / DP
        ↓
Bit Manipulation / Math
        ↓
Advanced Data Structures
        ↓
Advanced Patterns
```

## Mastery Definition

You are ready to move on when you can:
- identify the pattern quickly
- derive the approach
- implement it in Java without a template
- explain correctness
- state time and space complexity
- handle edge cases
- solve at least one unfamiliar variation

## Data Notes

- The current source set contains 23 generated DSA topic files. Duplicate generated snapshots with the same filename were treated as duplicate snapshots, not separate topics.
- The problem database contains 230 unique named problem units extracted from the LeetCode/practice-roadmap material. Cross-topic duplicates are counted once globally.
- 80 problems have no explicit difficulty in the source.
- 116 problems have no explicit priority in the source.
- Problem links are `TBD` where the source did not provide an official link.
- The source contains 82 distinct exact `Pattern:` labels. They are preserved at source granularity.
