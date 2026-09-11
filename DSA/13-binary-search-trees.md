# 13 — Binary Search Trees

> **Goal:** Master Binary Search Trees (BSTs) for FAANG/top-product-company interviews and CS/GATE-level theory. Focus on recognizing when the BST ordering property lets you replace a general-tree traversal with directed search, and understand the exact invariants behind every operation.

---

## 1. What Is a Binary Search Tree?

A **Binary Search Tree (BST)** is a binary tree with an ordering property.

For every node `x`:

- all keys in the left subtree are smaller than `x.key`
- all keys in the right subtree are larger than `x.key`

For the standard **distinct-key BST**:

```text
            8
          /   \
         3     10
        / \      \
       1   6      14
          / \     /
         4   7   13
```

At node `8`:

```text
left subtree  < 8
right subtree > 8
```

At node `3`:

```text
left subtree  < 3
right subtree > 3
```

The ordering property is what makes a BST useful.

### BST vs Binary Tree

| Property | Binary Tree | BST |
|---|---|---|
| At most 2 children | Yes | Yes |
| Ordered keys | Not required | Yes |
| Search can discard half-like subtree | No | Yes |
| Inorder traversal sorted | No | Yes, for valid distinct-key BST |
| Search complexity | O(n) | O(h) |
| Average search in balanced BST | — | O(log n) |
| Worst-case search | O(n) | O(n) |

`h` = height of the tree.

---

# 2. BST Properties

## 2.1 Local vs Global Ordering

A common mistake is checking only the immediate children.

This is **not enough**:

```text
       10
      /  \
     5    15
         /
        7
```

`7 < 15`, so it is locally valid at node `15`.

But `7` is in the right subtree of `10`, so it must satisfy:

```text
7 > 10
```

It does not.

Therefore the tree is invalid.

### Correct rule

A node is constrained by **all ancestors**, not just its parent.

---

## 2.2 Range Interpretation

Every recursive call should conceptually carry:

```text
(minAllowed, maxAllowed)
```

Example:

```text
        8
       / \
      3   10
```

For node `3`:

```text
(-∞, 8)
```

For node `10`:

```text
(8, +∞)
```

For node `6` under `3`:

```text
(3, 8)
```

This idea is the foundation of robust BST validation.

---

## 2.3 Inorder Traversal Property

For a valid BST with distinct keys:

```text
inorder traversal = strictly increasing sequence
```

Example:

```text
       8
      / \
     3   10
    / \
   1   6
```

Inorder:

```text
1 3 6 8 10
```

This gives another way to validate a BST.

---

## 2.4 Duplicate Keys

There are multiple valid duplicate policies.

### Policy A — No duplicates

```text
left < root < right
```

### Policy B — Duplicates allowed on the left

```text
left <= root < right
```

### Policy C — Duplicates allowed on the right

```text
left < root <= right
```

Interview problems normally state the policy or implicitly assume distinct values.

Do not silently mix policies.

---

# 3. BST Height and Complexity

For a BST:

```text
time = O(h)
```

where `h` is tree height.

## Balanced BST

```text
h = O(log n)
```

Therefore:

```text
search = O(log n)
insert = O(log n)
delete = O(log n)
```

## Skewed BST

If insertion order is:

```text
1, 2, 3, 4, 5
```

the tree becomes:

```text
1
 \
  2
   \
    3
     \
      4
       \
        5
```

Then:

```text
h = O(n)
```

and operations become:

```text
O(n)
```

### Important interview statement

A BST is **not automatically O(log n)**.

It is O(log n) only when its height is O(log n), typically because it is balanced.

---

# 4. BST Node in Java

```java
static class TreeNode {
    int val;
    TreeNode left;
    TreeNode right;

    TreeNode(int val) {
        this.val = val;
    }
}
```

---

# 5. Search in BST

## 5.1 Idea

At every node:

- if `target == node.val`, found
- if `target < node.val`, go left
- if `target > node.val`, go right

You never need to inspect the other subtree.

---

## 5.2 Recursive Search

```java
static TreeNode search(TreeNode root, int target) {
    if (root == null || root.val == target) {
        return root;
    }

    if (target < root.val) {
        return search(root.left, target);
    }

    return search(root.right, target);
}
```

### Complexity

```text
Time: O(h)
Space: O(h) recursion
```

Balanced:

```text
O(log n)
```

Worst case:

```text
O(n)
```

---

## 5.3 Iterative Search

Usually preferable when you only need to search.

```java
static TreeNode searchIterative(TreeNode root, int target) {
    TreeNode current = root;

    while (current != null) {
        if (current.val == target) {
            return current;
        }

        if (target < current.val) {
            current = current.left;
        } else {
            current = current.right;
        }
    }

    return null;
}
```

### Complexity

```text
Time: O(h)
Extra space: O(1)
```

---

## 5.4 Search Recognition Pattern

Whenever you see:

> "Search for a value in a BST"

think:

```text
compare
→ left or right
→ repeat
```

Do **not** automatically use DFS over the entire tree.

---

# 6. Insert in BST

## 6.1 Idea

Follow the same search path.

For a new key:

- smaller → left
- larger → right
- empty position → insert there

Example:

Insert `5`:

```text
        8
       /
      3
     / \
    1   6
       /
      4
```

Path:

```text
5 < 8 → left
5 > 3 → right
5 < 6 → left
5 > 4 → right
```

Insert:

```text
        8
       /
      3
     / \
    1   6
       /
      4
       \
        5
```

---

## 6.2 Recursive Insert

```java
static TreeNode insert(TreeNode root, int key) {
    if (root == null) {
        return new TreeNode(key);
    }

    if (key < root.val) {
        root.left = insert(root.left, key);
    } else if (key > root.val) {
        root.right = insert(root.right, key);
    }

    return root;
}
```

This version ignores duplicate keys.

### Complexity

```text
Time: O(h)
Space: O(h) recursion
```

---

## 6.3 Iterative Insert

```java
static TreeNode insertIterative(TreeNode root, int key) {
    if (root == null) {
        return new TreeNode(key);
    }

    TreeNode current = root;

    while (true) {
        if (key < current.val) {
            if (current.left == null) {
                current.left = new TreeNode(key);
                break;
            }
            current = current.left;
        } else if (key > current.val) {
            if (current.right == null) {
                current.right = new TreeNode(key);
                break;
            }
            current = current.right;
        } else {
            // Duplicate policy: ignore.
            break;
        }
    }

    return root;
}
```

---

# 7. Delete in BST

Deletion is the most important BST operation to understand structurally.

There are three cases.

## Case 1 — Leaf

```text
    5
   / \
  3   7
```

Delete `3`:

```text
    5
     \
      7
```

Simply remove it.

---

## Case 2 — One Child

```text
    5
     \
      7
       \
        8
```

Delete `7`.

Replace `7` with its only child:

```text
    5
     \
      8
```

---

## Case 3 — Two Children

This is the important case.

Example:

```text
       8
      / \
     3   10
        /
       9
```

Suppose deleting `8`.

We need a replacement that preserves BST ordering.

Use either:

- **inorder successor** = smallest value in right subtree
- **inorder predecessor** = largest value in left subtree

Using successor:

```text
successor(8) = 9
```

Replace `8` with `9`, then delete the original `9`.

---

# 8. Inorder Successor During Deletion

The successor of a node with a right subtree is:

```text
go right once
then go left as far as possible
```

```java
static TreeNode minNode(TreeNode root) {
    TreeNode current = root;

    while (current.left != null) {
        current = current.left;
    }

    return current;
}
```

---

# 9. Complete BST Delete

```java
static TreeNode delete(TreeNode root, int key) {
    if (root == null) {
        return null;
    }

    if (key < root.val) {
        root.left = delete(root.left, key);
    } else if (key > root.val) {
        root.right = delete(root.right, key);
    } else {
        // Case 1: no child
        if (root.left == null && root.right == null) {
            return null;
        }

        // Case 2: one child
        if (root.left == null) {
            return root.right;
        }

        if (root.right == null) {
            return root.left;
        }

        // Case 3: two children
        TreeNode successor = minNode(root.right);
        root.val = successor.val;
        root.right = delete(root.right, successor.val);
    }

    return root;
}
```

### Complexity

```text
Time: O(h)
Space: O(h) recursion
```

Balanced:

```text
O(log n)
```

Worst case:

```text
O(n)
```

---

# 10. Why Successor Works

Suppose:

```text
        8
       / \
      3   12
          /
         10
        /
       9
```

For node `8`, the right subtree contains:

```text
9, 10, 12
```

The smallest value is:

```text
9
```

So:

```text
all left-subtree values < 9
all remaining right-subtree values > 9
```

Thus `9` can replace `8`.

This is exactly why the **minimum of the right subtree** is safe.

Symmetrically, the **maximum of the left subtree** is also safe.

---

# 11. Validate BST

This is one of the most frequently tested BST problems.

## 11.1 Incorrect Approach

Checking only:

```java
node.left.val < node.val
node.right.val > node.val
```

is insufficient.

Example:

```text
        10
       /  \
      5    15
          /
         7
```

Node `7` is less than its parent `15`, but it is still in the right subtree of `10`.

Therefore invalid.

---

# 12. Validation Using Bounds

The cleanest recursive approach carries valid bounds.

```java
static boolean isValidBST(TreeNode root) {
    return validate(root, Long.MIN_VALUE, Long.MAX_VALUE);
}

static boolean validate(TreeNode node, long low, long high) {
    if (node == null) {
        return true;
    }

    if (node.val <= low || node.val >= high) {
        return false;
    }

    return validate(node.left, low, node.val)
        && validate(node.right, node.val, high);
}
```

### Why `long`?

If node values are `Integer.MIN_VALUE` or `Integer.MAX_VALUE`, using integer sentinels can create boundary problems.

Using:

```java
long low
long high
```

avoids that issue for `int` node values.

### Complexity

```text
Time: O(n)
Space: O(h)
```

Every node must potentially be examined.

---

# 13. Validation Using Inorder Traversal

Because a valid distinct-key BST has strictly increasing inorder traversal:

```text
inorder = sorted strictly increasing
```

```java
static boolean isValidBSTInorder(TreeNode root) {
    Deque<TreeNode> stack = new ArrayDeque<>();
    TreeNode current = root;
    Long previous = null;

    while (current != null || !stack.isEmpty()) {
        while (current != null) {
            stack.push(current);
            current = current.left;
        }

        current = stack.pop();

        if (previous != null && current.val <= previous) {
            return false;
        }

        previous = (long) current.val;
        current = current.right;
    }

    return true;
}
```

### Complexity

```text
Time: O(n)
Space: O(h)
```

### Interview choice

Use **bounds** when explaining the BST invariant.

Use **inorder** when the problem naturally involves sorted order or kth elements.

---

# 14. K-th Smallest Element in BST

This is a major interview pattern.

## Key Observation

Inorder traversal of a BST gives sorted order.

Example:

```text
       5
      / \
     3   7
    / \
   2   4
```

Inorder:

```text
2 3 4 5 7
```

Therefore:

```text
1st smallest = 2
2nd smallest = 3
3rd smallest = 4
...
```

---

# 15. K-th Smallest — Iterative Inorder

```java
static int kthSmallest(TreeNode root, int k) {
    Deque<TreeNode> stack = new ArrayDeque<>();
    TreeNode current = root;

    while (current != null || !stack.isEmpty()) {
        while (current != null) {
            stack.push(current);
            current = current.left;
        }

        current = stack.pop();

        k--;

        if (k == 0) {
            return current.val;
        }

        current = current.right;
    }

    throw new IllegalArgumentException("k is outside the number of nodes");
}
```

### Complexity

```text
Time: O(h + k) in the usual traversal-bound analysis
Space: O(h)
```

More precisely, only nodes needed to reach the kth inorder element are visited.

---

# 16. K-th Largest Element

Use reverse inorder:

```text
right → root → left
```

because this produces descending order.

```java
static int kthLargest(TreeNode root, int k) {
    Deque<TreeNode> stack = new ArrayDeque<>();
    TreeNode current = root;

    while (current != null || !stack.isEmpty()) {
        while (current != null) {
            stack.push(current);
            current = current.right;
        }

        current = stack.pop();

        k--;

        if (k == 0) {
            return current.val;
        }

        current = current.left;
    }

    throw new IllegalArgumentException("k is outside the number of nodes");
}
```

---

# 17. K-th Smallest — Recursive Pattern

```java
static int kthSmallestRecursive(TreeNode root, int k) {
    int[] count = {0};
    int[] answer = {0};

    inorderFind(root, k, count, answer);
    return answer[0];
}

static boolean inorderFind(
        TreeNode node,
        int k,
        int[] count,
        int[] answer) {

    if (node == null) {
        return false;
    }

    if (inorderFind(node.left, k, count, answer)) {
        return true;
    }

    count[0]++;

    if (count[0] == k) {
        answer[0] = node.val;
        return true;
    }

    return inorderFind(node.right, k, count, answer);
}
```

The iterative version is generally cleaner for interviews.

---

# 18. LCA in BST

For a general binary tree, LCA may require searching both subtrees.

A BST gives more information.

Suppose:

```text
          6
        /   \
       2     8
      / \   / \
     0   4 7   9
        / \
       3   5
```

Find LCA of `2` and `8`.

At `6`:

```text
2 < 6
8 > 6
```

They split at `6`.

Therefore:

```text
LCA = 6
```

---

# 19. BST LCA Rules

At node `root`:

### Both values smaller

```text
p < root.val
q < root.val
```

Go left.

### Both values larger

```text
p > root.val
q > root.val
```

Go right.

### Otherwise

They split at the current node, so current node is the LCA.

---

# 20. LCA Java Implementation

```java
static TreeNode lowestCommonAncestor(
        TreeNode root,
        TreeNode p,
        TreeNode q) {

    TreeNode current = root;

    while (current != null) {
        if (p.val < current.val && q.val < current.val) {
            current = current.left;
        } else if (p.val > current.val && q.val > current.val) {
            current = current.right;
        } else {
            return current;
        }
    }

    return null;
}
```

### Complexity

```text
Time: O(h)
Space: O(1)
```

---

# 21. Successor and Predecessor

These are important because they connect BST structure with sorted order.

## Inorder Order

For:

```text
       8
      / \
     3   10
    / \
   1   6
      / \
     4   7
```

Inorder:

```text
1, 3, 4, 6, 7, 8, 10
```

For node `6`:

```text
predecessor = 4
successor   = 7
```

---

# 22. Inorder Successor

The successor is the smallest key greater than the current key.

There are two structural cases.

## Case 1 — Right subtree exists

Go:

```text
right
→ left as far as possible
```

Example:

```text
    6
     \
      10
      /
     8
```

Successor of `6`:

```text
8
```

---

## Case 2 — No right subtree

Move upward until you come from a **left child**.

Example:

```text
        8
       /
      3
       \
        6
```

For `6`, there is no right subtree.

Its first ancestor where `6` lies in the left subtree is:

```text
8
```

So:

```text
successor(6) = 8
```

---

# 23. Successor Java Implementation

Assume values are distinct and `root` is the BST root.

```java
static TreeNode successor(TreeNode root, TreeNode node) {
    TreeNode answer = null;
    TreeNode current = root;

    while (current != null) {
        if (node.val < current.val) {
            answer = current;
            current = current.left;
        } else if (node.val > current.val) {
            current = current.right;
        } else {
            break;
        }
    }

    if (current == null) {
        return null;
    }

    if (current.right != null) {
        return minNode(current.right);
    }

    return answer;
}
```

### Complexity

```text
Time: O(h)
Space: O(1)
```

---

# 24. Inorder Predecessor

The predecessor is the largest key smaller than the current key.

## Case 1 — Left subtree exists

Go:

```text
left
→ right as far as possible
```

## Case 2 — No left subtree

Move upward until you come from a **right child**.

---

# 25. Predecessor Java Implementation

```java
static TreeNode predecessor(TreeNode root, TreeNode node) {
    TreeNode answer = null;
    TreeNode current = root;

    while (current != null) {
        if (node.val > current.val) {
            answer = current;
            current = current.right;
        } else if (node.val < current.val) {
            current = current.left;
        } else {
            break;
        }
    }

    if (current == null) {
        return null;
    }

    if (current.left != null) {
        return maxNode(current.left);
    }

    return answer;
}

static TreeNode maxNode(TreeNode root) {
    TreeNode current = root;

    while (current.right != null) {
        current = current.right;
    }

    return current;
}
```

### Complexity

```text
Time: O(h)
Space: O(1)
```

---

# 26. Successor / Predecessor Recognition

Remember:

```text
SUCCESSOR:
right → far left
or first ancestor larger than node

PREDECESSOR:
left → far right
or first ancestor smaller than node
```

---

# 27. Convert Sorted Array to BST

Given a sorted array:

```text
[-10, -3, 0, 5, 9]
```

we want a height-balanced BST.

## Core idea

Choose the middle element as the root.

```text
          0
        /   \
      -3     9
      /     /
   -10      5
```

Then recursively repeat.

---

# 28. Why Choose the Middle?

If we choose the first element repeatedly:

```text
-10
  \
  -3
    \
     0
      \
       5
        \
         9
```

the tree is skewed.

Choosing the middle approximately halves the remaining elements.

Therefore:

```text
height = O(log n)
```

---

# 29. Sorted Array → BST Java

```java
static TreeNode sortedArrayToBST(int[] nums) {
    return build(nums, 0, nums.length - 1);
}

static TreeNode build(int[] nums, int left, int right) {
    if (left > right) {
        return null;
    }

    int mid = left + (right - left) / 2;

    TreeNode root = new TreeNode(nums[mid]);

    root.left = build(nums, left, mid - 1);
    root.right = build(nums, mid + 1, right);

    return root;
}
```

### Complexity

```text
Time: O(n)
Space: O(log n) recursion for balanced output
```

Every array element becomes exactly one tree node.

---

# 30. Recover Corrupted BST

A valid BST can become invalid if two node values are swapped.

Example:

Valid inorder:

```text
1 2 3 4 5
```

Corrupted inorder:

```text
1 4 3 2 5
```

The values `2` and `4` were swapped.

## Key observation

A valid BST has increasing inorder order.

A swapped pair creates one or two violations.

---

# 31. Detecting the Two Swapped Nodes

During inorder traversal, maintain:

```text
previous node
first violation
second violation
```

If:

```text
previous.val > current.val
```

we found an ordering violation.

### First violation

Set:

```text
first = previous
second = current
```

### Second violation

Set:

```text
second = current
```

At the end:

```text
swap first.val and second.val
```

---

# 32. Recover BST Java

```java
static class RecoverState {
    TreeNode previous;
    TreeNode first;
    TreeNode second;
}

static void recoverTree(TreeNode root) {
    RecoverState state = new RecoverState();
    recoverInorder(root, state);

    int temp = state.first.val;
    state.first.val = state.second.val;
    state.second.val = temp;
}

static void recoverInorder(TreeNode node, RecoverState state) {
    if (node == null) {
        return;
    }

    recoverInorder(node.left, state);

    if (state.previous != null
            && state.previous.val > node.val) {

        if (state.first == null) {
            state.first = state.previous;
        }

        state.second = node;
    }

    state.previous = node;

    recoverInorder(node.right, state);
}
```

### Complexity

```text
Time: O(n)
Space: O(h)
```

The tree structure is unchanged; only the two incorrect values are swapped back.

---

# 33. Why There Can Be One or Two Violations

## Adjacent swapped values

Valid:

```text
1 2 3 4 5
```

Swap `2` and `3`:

```text
1 3 2 4 5
```

One violation:

```text
3 > 2
```

So:

```text
first = 3
second = 2
```

---

## Non-adjacent swapped values

Swap `2` and `5`:

```text
1 5 3 4 2
```

Violations:

```text
5 > 3
4 > 2
```

First violation identifies:

```text
first = 5
```

The last offending current node identifies:

```text
second = 2
```

Hence the algorithm updates `second` every time a violation occurs.

---

# 34. BST Iterator

A BST iterator usually asks for:

```text
next()
hasNext()
```

with values returned in ascending order.

The key is to avoid storing the entire inorder traversal if the problem expects O(h) auxiliary space.

---

# 35. BST Iterator — Stack Idea

Instead of generating all values first:

1. Push the entire left path from root.
2. `next()` pops the smallest available node.
3. If that node has a right child, push the left path from that right subtree.
4. The stack always represents the next inorder frontier.

---

# 36. Java BST Iterator

```java
class BSTIterator {
    private final Deque<TreeNode> stack = new ArrayDeque<>();

    BSTIterator(TreeNode root) {
        pushLeft(root);
    }

    private void pushLeft(TreeNode node) {
        while (node != null) {
            stack.push(node);
            node = node.left;
        }
    }

    int next() {
        TreeNode node = stack.pop();

        if (node.right != null) {
            pushLeft(node.right);
        }

        return node.val;
    }

    boolean hasNext() {
        return !stack.isEmpty();
    }
}
```

### Complexity

Initialization:

```text
O(h)
```

Each node is pushed once and popped once:

```text
O(n) total
```

Amortized:

```text
O(1) per next()
```

Space:

```text
O(h)
```

---

# 37. Why BST Iterator Is Amortized O(1)

A single `next()` may push multiple nodes:

```text
node.right
→ left
→ left
→ left
```

So one call can cost O(h).

But across the entire iteration:

- every node is pushed once
- every node is popped once

Therefore total stack work is:

```text
O(n)
```

Across `n` calls:

```text
O(n) / n = O(1) amortized
```

This is a standard amortized-analysis pattern.

---

# 38. BST Iterator: Mental Model

Think of the stack as:

> "The path to the smallest not-yet-returned node, plus future alternatives."

Example:

```text
        7
       / \
      3   10
     / \
    1   5
```

Initial stack:

```text
7
3
1  ← top
```

`next()`:

```text
1
```

Then:

```text
3
```

Then:

```text
5
```

Then:

```text
7
```

Then:

```text
10
```

No complete inorder array is required.

---

# 39. BST Operations Summary

| Operation | Balanced BST | Worst-case BST |
|---|---:|---:|
| Search | O(log n) | O(n) |
| Insert | O(log n) | O(n) |
| Delete | O(log n) | O(n) |
| Min | O(log n) | O(n) |
| Max | O(log n) | O(n) |
| Successor | O(log n) | O(n) |
| Predecessor | O(log n) | O(n) |
| LCA | O(log n) | O(n) |
| Validate BST | O(n) | O(n) |
| K-th smallest | O(h + k) typical | O(n) worst case |
| Build from sorted array | O(n) | O(n) |
| BST iterator init | O(h) | O(n) |
| Iterator `next()` | O(1) amortized | O(1) amortized |

---

# 40. BST vs Heap

These structures are frequently confused.

| Property | BST | Heap |
|---|---|---|
| Shape | Arbitrary binary tree | Complete binary tree |
| Ordering | Left < root < right | Parent dominates children |
| Search arbitrary key | O(h) | O(n) |
| Minimum | Leftmost | Root in min heap |
| Maximum | Rightmost | Root in max heap |
| Sorted traversal | Inorder | Not sorted |
| Typical use | Ordered set/map | Priority queue |
| Java structure | TreeMap / TreeSet | PriorityQueue |

A heap cannot efficiently answer:

> "Does value 73 exist?"

A BST can, assuming good height.

---

# 41. BST vs Hash Table

| Operation | BST | Hash Table |
|---|---|---|
| Search | O(h) | O(1) average |
| Insert | O(h) | O(1) average |
| Delete | O(h) | O(1) average |
| Sorted order | Yes | No |
| Range queries | Good | Poor without extra structure |
| Min/max | Easy | Not naturally ordered |
| Predecessor/successor | Natural | Not natural |

Use a BST-like ordered structure when **ordering matters**.

Use hashing when you primarily need fast exact-key lookup.

---

# 42. Java Ordered Structures

Java provides balanced-tree implementations through:

```java
TreeSet
TreeMap
```

These are not ordinary unbalanced BSTs.

They are based on a self-balancing search tree implementation and provide logarithmic operations.

Useful operations include:

```java
TreeSet<Integer> set = new TreeSet<>();

set.add(10);
set.add(5);
set.add(20);

set.contains(10);
set.first();
set.last();

set.lower(10);   // greatest element < 10
set.higher(10);  // smallest element > 10

set.floor(10);  // greatest element <= 10
set.ceiling(10); // smallest element >= 10
```

This is directly related to predecessor/successor concepts.

---

# 43. BST Floor and Ceiling

These are common interview variants.

## Floor

Largest key:

```text
<= target
```

## Ceiling

Smallest key:

```text
>= target
```

Example:

```text
keys = 2, 5, 8, 12
target = 7
```

Then:

```text
floor = 5
ceiling = 8
```

---

# 44. Floor in BST

```java
static Integer floor(TreeNode root, int target) {
    Integer answer = null;
    TreeNode current = root;

    while (current != null) {
        if (current.val == target) {
            return current.val;
        }

        if (current.val < target) {
            answer = current.val;
            current = current.right;
        } else {
            current = current.left;
        }
    }

    return answer;
}
```

---

# 45. Ceiling in BST

```java
static Integer ceiling(TreeNode root, int target) {
    Integer answer = null;
    TreeNode current = root;

    while (current != null) {
        if (current.val == target) {
            return current.val;
        }

        if (current.val > target) {
            answer = current.val;
            current = current.left;
        } else {
            current = current.right;
        }
    }

    return answer;
}
```

Both run in:

```text
O(h)
```

---

# 46. Range Queries in a BST

Suppose we need values in:

```text
[L, R]
```

A naive inorder traversal visits every node:

```text
O(n)
```

But BST ordering lets us prune subtrees.

At node `x`:

- if `x < L`, its left subtree is also `< L` → skip left
- if `x > R`, its right subtree is also `> R` → skip right
- otherwise process `x` and potentially both subtrees

Example:

```java
static void rangeQuery(
        TreeNode root,
        int low,
        int high,
        List<Integer> result) {

    if (root == null) {
        return;
    }

    if (root.val > low) {
        rangeQuery(root.left, low, high, result);
    }

    if (root.val >= low && root.val <= high) {
        result.add(root.val);
    }

    if (root.val < high) {
        rangeQuery(root.right, low, high, result);
    }
}
```

If `k` values are reported, a common bound is:

```text
O(h + k)
```

for a balanced BST under the usual output-sensitive analysis.

---

# 47. Construct BST from Preorder

A preorder sequence alone can determine a BST when the BST ordering property is known and values are distinct.

Example:

```text
preorder = [8, 3, 1, 6, 10, 14]
```

Use bounds to determine where each value belongs.

A typical recursive state is:

```text
(index, lowerBound, upperBound)
```

```java
static TreeNode bstFromPreorder(int[] preorder) {
    int[] index = {0};
    return buildFromPreorder(
            preorder,
            index,
            Long.MIN_VALUE,
            Long.MAX_VALUE
    );
}

static TreeNode buildFromPreorder(
        int[] preorder,
        int[] index,
        long low,
        long high) {

    if (index[0] == preorder.length) {
        return null;
    }

    int value = preorder[index[0]];

    if (value <= low || value >= high) {
        return null;
    }

    index[0]++;

    TreeNode root = new TreeNode(value);

    root.left = buildFromPreorder(
            preorder,
            index,
            low,
            value
    );

    root.right = buildFromPreorder(
            preorder,
            index,
            value,
            high
    );

    return root;
}
```

### Complexity

```text
Time: O(n)
Space: O(h)
```

---

# 48. Important BST Invariants

These are the invariants you should be able to state without hesitation.

### Search

At every node:

```text
target < node → left
target > node → right
```

### Insert

Insert only at a position where the ordering property remains valid.

### Delete

After deletion:

```text
left subtree < root < right subtree
```

must still hold.

### Validate

Every node must satisfy its complete ancestor-imposed range.

### K-th smallest

Inorder gives sorted order.

### LCA

If both keys are on one side, move there.

Otherwise current node is the split point.

### Successor

Next value in inorder.

### Predecessor

Previous value in inorder.

### Recover BST

Repair violations in inorder sortedness.

### Iterator

Maintain the inorder frontier with a stack.

---

# 49. Common BST Bugs

## Bug 1 — Assuming BST is always O(log n)

Wrong.

```text
BST → O(h)
```

Only a balanced BST guarantees:

```text
h = O(log n)
```

---

## Bug 2 — Validate only immediate children

Wrong:

```text
left < node < right
```

must hold for entire subtrees.

Use bounds.

---

## Bug 3 — Wrong deletion replacement

For two children, arbitrary replacement breaks ordering.

Use:

```text
min(right subtree)
```

or:

```text
max(left subtree)
```

---

## Bug 4 — Forgetting to reconnect recursive delete

This is wrong:

```java
delete(root.left, key);
```

You need:

```java
root.left = delete(root.left, key);
```

because deletion may change the subtree root.

---

## Bug 5 — K-th smallest with preorder

Wrong traversal.

Use:

```text
inorder
```

because BST inorder is sorted.

---

## Bug 6 — K-th largest with normal inorder

Use:

```text
reverse inorder:
right → root → left
```

---

## Bug 7 — LCA using general-tree logic

A BST gives ordering information.

Exploit it.

---

## Bug 8 — Successor confused with right child

The successor is **not necessarily the right child**.

If a right subtree exists:

```text
successor = leftmost node of right subtree
```

Otherwise it is an ancestor.

---

## Bug 9 — Recover BST swaps nodes instead of values

Typical interview problem:

> exactly two node values were swapped.

The intended repair is usually:

```text
swap their values
```

not physically rearrange tree nodes.

---

## Bug 10 — Iterator stores all values unnecessarily

An inorder array works functionally, but can use:

```text
O(n)
```

space.

The stack-based iterator uses:

```text
O(h)
```

space with amortized O(1) `next()`.

---

# 50. Interview Pattern Recognition

| Problem wording | Immediate thought |
|---|---|
| Search in BST | Directed traversal |
| Insert into BST | Search for empty position |
| Delete BST node | 0 / 1 / 2 children |
| Validate BST | Bounds or inorder |
| K-th smallest | Inorder |
| K-th largest | Reverse inorder |
| LCA in BST | Split point |
| Next greater key | Successor |
| Previous smaller key | Predecessor |
| Sorted array → balanced BST | Middle element |
| Two values swapped | Inorder violation |
| Iterator in sorted order | Explicit inorder stack |
| Floor | Greatest `<= x` |
| Ceiling | Smallest `>= x` |
| Range values | Prune using ordering |
| Ordered dynamic set | TreeSet / balanced BST |

---

# 51. BST Problem-Solving Decision Tree

When you see a BST problem:

```text
                BST problem
                     |
          +----------+----------+
          |                     |
     Exact key?             Order-related?
          |                     |
       Search              +-----+------+
                           |            |
                       kth/rank      next/prev
                           |            |
                        inorder      successor
                                     predecessor
          |
      Modification?
          |
     +----+----+
     |         |
   insert    delete
               |
        +------+------+ 
        |      |      |
       0      1       2 children
      child  child     |
                     successor/
                    predecessor
```

---

# 52. BST vs Balanced BST

A normal BST does not automatically rebalance.

If input arrives in sorted order:

```text
1, 2, 3, 4, 5, 6
```

an ordinary insertion-based BST can become:

```text
1
 \
  2
   \
    3
     \
      4
       \
        5
         \
          6
```

A self-balancing BST such as an AVL tree or Red-Black tree maintains:

```text
h = O(log n)
```

This distinction matters in system and GATE questions.

---

# 53. GATE / CS Theory

## 53.1 BST Search Complexity

Search takes:

```text
O(h)
```

where `h` is tree height.

Therefore:

```text
best case: O(1)
balanced: O(log n)
worst case: O(n)
```

---

## 53.2 Inorder Traversal

For a BST with distinct keys:

```text
inorder = sorted increasing order
```

This is one of the most important BST properties.

---

## 53.3 Minimum and Maximum

Minimum:

```text
leftmost node
```

Maximum:

```text
rightmost node
```

If a node has no left child, it is not necessarily globally minimum; it may only be locally leftmost from its current position.

---

## 53.4 Number of BSTs with n Distinct Keys

The number of structurally distinct BSTs that can be formed using `n` distinct keys is the **n-th Catalan number**:

```text
C_n = 1/(n+1) * binomial(2n, n)
```

Recurrence:

```text
C_0 = 1

C_n = Σ(C_i × C_(n-1-i))
      for i = 0 to n-1
```

For example:

```text
n = 3

C_3 = 5
```

So three distinct keys can form five structurally distinct BSTs.

---

## 53.5 Height Extremes

For `n` nodes:

Minimum possible height depends on the convention.

If height means number of edges:

```text
minimum ≈ floor(log2 n)
maximum = n - 1
```

A completely skewed BST has:

```text
h = n - 1
```

---

# 54. GATE-Style Solved Question 1 — BST Validation

> **Question:** Consider the tree:
>
> ```text
>         10
>        /  \
>       5    15
>           /
>          7
> ```
>
> Is it a valid BST?

### Solution

At node `15`, the left child `7` satisfies:

```text
7 < 15
```

But `7` lies in the right subtree of `10`.

Therefore it must satisfy:

```text
7 > 10
```

It does not.

### Answer

```text
Invalid BST
```

### Key lesson

BST validity is based on **global ancestor constraints**, not only parent-child comparisons.

---

# 55. GATE-Style Solved Question 2 — Height and Search

> **Question:** A BST contains `n` nodes. What is the worst-case time complexity of searching for a key?

### Solution

BST search follows exactly one root-to-leaf path.

Its complexity is:

```text
O(h)
```

In the worst case, the BST is skewed:

```text
h = O(n)
```

Therefore:

```text
Worst-case search = O(n)
```

### Answer

```text
O(n)
```

A common trap is answering O(log n) without assuming balance.

---

# 56. GATE-Style Solved Question 3 — K-th Smallest

> **Question:** A valid BST has inorder traversal:
>
> ```text
> 2, 4, 5, 7, 9, 12, 15
> ```
>
> What is the 5th smallest key?

### Solution

BST inorder traversal is sorted.

Therefore:

```text
1st = 2
2nd = 4
3rd = 5
4th = 7
5th = 9
```

### Answer

```text
9
```

### Key lesson

For a BST:

```text
k-th smallest = k-th node in inorder
```

---

# 57. Additional GATE-Style Practice Questions

## Question 4

A BST is constructed by inserting:

```text
50, 30, 70, 20, 40, 60, 80
```

What is its inorder traversal?

### Solution

Inorder of any valid BST is sorted.

```text
20 30 40 50 60 70 80
```

### Answer

```text
20 30 40 50 60 70 80
```

---

## Question 5

A BST contains distinct keys. Which traversal produces keys in decreasing order?

### Solution

Normal inorder:

```text
left → root → right
```

produces increasing order.

Reverse inorder:

```text
right → root → left
```

produces decreasing order.

### Answer

```text
Reverse inorder
```

---

## Question 6

A BST has `n` nodes and height `h`. What is the time complexity of finding its minimum element?

### Solution

Follow left pointers:

```text
root → left → left → ...
```

At most `h` edges are traversed.

### Answer

```text
O(h)
```

---

# 58. LeetCode Roadmap

The following problem types should be mastered.

## Level 1 — Core

1. Search in a Binary Search Tree
2. Insert into a Binary Search Tree
3. Minimum / Maximum in BST
4. Validate Binary Search Tree
5. Lowest Common Ancestor of a BST
6. Convert Sorted Array to Binary Search Tree

## Level 2 — Interview Core

7. Kth Smallest Element in a BST
8. Delete Node in a BST
9. BST Iterator
10. Inorder Successor / Predecessor
11. Floor / Ceiling in BST
12. Range queries in BST

## Level 3 — Advanced

13. Recover Binary Search Tree
14. Construct BST from Preorder
15. Trim a Binary Search Tree
16. Two Sum in a BST
17. Balance a BST
18. Convert BST to Greater Tree
19. Greater Sum Tree
20. Count / rank-style BST problems

### Practice order

```text
Properties
→ Search
→ Insert
→ Validate
→ Delete
→ Inorder / Reverse Inorder
→ K-th smallest/largest
→ LCA
→ Successor / predecessor
→ Sorted array → BST
→ Iterator
→ Recover BST
→ Advanced transformations
```

---

# 59. Java BST Cheat Sheet

## Search

```java
while (root != null) {
    if (target == root.val) return root;
    root = target < root.val ? root.left : root.right;
}
```

## Insert

```java
if (key < root.val) {
    root.left = insert(root.left, key);
} else if (key > root.val) {
    root.right = insert(root.right, key);
}
```

## Delete

```text
0 children → null
1 child    → return child
2 children → replace with successor/predecessor
```

## Validate

```text
bounds
or
inorder strictly increasing
```

## K-th smallest

```text
inorder
```

## K-th largest

```text
reverse inorder
```

## LCA

```text
both smaller → left
both larger  → right
otherwise    → current
```

## Successor

```text
right → leftmost
or first larger ancestor
```

## Predecessor

```text
left → rightmost
or first smaller ancestor
```

## Balanced BST from sorted array

```text
middle → root
left half → left
right half → right
```

## Recover BST

```text
inorder violations
→ identify two nodes
→ swap values
```

## Iterator

```text
stack of left paths
```

---

# 60. Mastery Checklist

You should be able to solve these without looking at notes.

### BST fundamentals

- [ ] Define the BST ordering property.
- [ ] Explain why BST search is O(h), not automatically O(log n).
- [ ] Explain balanced vs skewed BST.
- [ ] Explain duplicate-key policies.
- [ ] Prove why inorder traversal is sorted.

### Search and modification

- [ ] Implement recursive search.
- [ ] Implement iterative search.
- [ ] Implement recursive insert.
- [ ] Implement iterative insert.
- [ ] Explain all three BST deletion cases.
- [ ] Implement deletion using inorder successor.
- [ ] Explain why successor replacement works.

### Validation and ordering

- [ ] Validate using lower/upper bounds.
- [ ] Validate using inorder traversal.
- [ ] Find minimum and maximum.
- [ ] Find floor.
- [ ] Find ceiling.
- [ ] Find successor.
- [ ] Find predecessor.

### Order statistics

- [ ] Find k-th smallest.
- [ ] Find k-th largest.
- [ ] Explain why inorder gives sorted order.
- [ ] Explain reverse inorder.

### Structural problems

- [ ] Find LCA in BST.
- [ ] Convert sorted array to balanced BST.
- [ ] Construct BST from preorder.
- [ ] Recover a BST with two swapped values.
- [ ] Implement a BST iterator.

### Theory

- [ ] Derive O(h) operation complexity.
- [ ] Explain worst-case O(n).
- [ ] Explain balanced O(log n).
- [ ] Know Catalan-number relation.
- [ ] Know height extremes.
- [ ] Distinguish BST from heap.
- [ ] Distinguish BST from hash table.
- [ ] Understand TreeSet / TreeMap as balanced ordered structures.

---

# 61. Final Revision Sheet

```text
BST:
left < root < right

Search:
compare → left/right
O(h)

Insert:
search → empty position
O(h)

Delete:
0 children → remove
1 child    → replace by child
2 children → successor/predecessor
O(h)

Validate:
global bounds
OR
inorder strictly increasing

Inorder:
sorted increasing

Reverse inorder:
sorted decreasing

K-th smallest:
k-th inorder node

K-th largest:
k-th reverse-inorder node

LCA:
both smaller → left
both larger  → right
otherwise    → current

Minimum:
leftmost

Maximum:
rightmost

Successor:
right → leftmost
OR first larger ancestor

Predecessor:
left → rightmost
OR first smaller ancestor

Sorted array → BST:
middle as root
recursive halves

Recover BST:
find inorder violations
swap two values

Iterator:
explicit inorder stack
O(h) space
O(1) amortized next()

Complexity:
balanced BST → O(log n)
skewed BST   → O(n)
```

---

# 62. Final Mental Model

The entire topic can be reduced to a few ideas:

```text
                 BST
                  |
          ordering property
                  |
       +----------+----------+
       |          |          |
    Search      Insert     Delete
       |          |          |
    compare     compare    0/1/2
    left/right  left/right  children
                              |
                       successor/predecessor

Ordering property
       |
       +-------------------------+
       |                         |
    inorder                    bounds
       |                         |
 sorted order              validate BST
       |
   +---+---+
   |       |
 kth      iterator
   |
rank/order

BST ordering
       |
       +----------------+
       |                |
      LCA         successor/
                  predecessor
```

The most important transition to internalize is:

> **A BST is a binary tree plus an ordering invariant. Almost every interview problem in this topic is solved by exploiting that invariant instead of traversing unnecessary subtrees.**
