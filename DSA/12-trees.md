# 12 — Trees

> **Goal:** Master binary-tree problems for FAANG and top product-company interviews. Focus on tree structure, DFS/BFS, recursive and iterative traversals, height/diameter/balance, symmetry, path problems, LCA, serialization, tree construction, flattening, boundary/vertical traversal, and left/right views.

---

# 1. What Is a Tree?

A tree is a hierarchical, connected, acyclic data structure.

A binary tree is a tree where each node has at most:

```text
2 children
```

usually called:

```text
left
right
```

Typical Java representation:

```java
class TreeNode {
    int val;
    TreeNode left;
    TreeNode right;

    TreeNode(int val) {
        this.val = val;
    }
}
```

---

# 2. Tree Terminology

Understand these terms precisely.

### Root

The topmost node.

```text
        1
       / \
      2   3
```

Root:

```text
1
```

### Parent

A node directly above another node.

```text
1
|
2
```

`1` is the parent of `2`.

### Child

A node directly below another node.

`2` is a child of `1`.

### Siblings

Nodes with the same parent.

```text
    1
   / \
  2   3
```

`2` and `3` are siblings.

### Leaf

A node with no children.

```text
2
```

is a leaf if:

```text
left == null
right == null
```

### Internal Node

A node with at least one child.

### Edge

A connection between two nodes.

### Path

A sequence of connected nodes.

### Subtree

A node together with all of its descendants.

### Depth

Distance from the root to a node.

Depending on convention:

```text
root depth = 0
```

is the most common algorithmic convention.

### Height

Maximum distance from a node down to a leaf.

Common convention:

```text
leaf height = 0
empty tree height = -1
```

Some problems instead define:

```text
leaf height = 1
empty tree height = 0
```

Always inspect the required convention.

---

# 3. Example Tree

Consider:

```text
             1
           /   \
          2     3
         / \   / \
        4   5 6   7
```

Root:

```text
1
```

Leaves:

```text
4, 5, 6, 7
```

Depth:

```text
depth(1) = 0
depth(2) = 1
depth(4) = 2
```

Height:

```text
height(4) = 0
height(2) = 1
height(1) = 2
```

---

# 4. Binary Tree vs Binary Search Tree

Do not confuse these.

## Binary Tree

Only constraint:

```text
at most two children
```

No ordering requirement.

## Binary Search Tree

Typically:

```text
left subtree < node < right subtree
```

or a problem-specific duplicate policy.

Therefore:

```text
Every BST is a binary tree.
Not every binary tree is a BST.
```

The current topic focuses primarily on **binary trees**.

---

# 5. DFS

Depth-First Search explores a branch deeply before moving to another branch.

For binary trees, the three fundamental DFS traversals are:

```text
Preorder
Inorder
Postorder
```

They differ only in when the current node is processed.

---

# 6. Preorder Traversal

Order:

```text
Root → Left → Right
```

Mnemonic:

```text
N L R
```

For:

```text
        1
       / \
      2   3
     / \
    4   5
```

preorder:

```text
1 2 4 5 3
```

---

# 6.1 Recursive Preorder

```java
static void preorder(TreeNode root, List<Integer> result) {
    if (root == null) {
        return;
    }

    result.add(root.val);

    preorder(root.left, result);
    preorder(root.right, result);
}
```

Complexity:

```text
Time  = O(n)
Space = O(h)
```

where `h` is tree height due to recursion.

Worst-case skewed tree:

```text
h = n
```

Balanced tree:

```text
h = O(log n)
```

---

# 7. Inorder Traversal

Order:

```text
Left → Root → Right
```

Mnemonic:

```text
L N R
```

Example:

```text
        1
       / \
      2   3
     / \
    4   5
```

inorder:

```text
4 2 5 1 3
```

Important:

> Inorder traversal of a BST produces values in sorted order when the BST ordering convention is standard.

---

# 7.1 Recursive Inorder

```java
static void inorder(TreeNode root, List<Integer> result) {
    if (root == null) {
        return;
    }

    inorder(root.left, result);

    result.add(root.val);

    inorder(root.right, result);
}
```

---

# 8. Postorder Traversal

Order:

```text
Left → Right → Root
```

Mnemonic:

```text
L R N
```

Example:

```text
        1
       / \
      2   3
     / \
    4   5
```

postorder:

```text
4 5 2 3 1
```

---

# 8.1 Recursive Postorder

```java
static void postorder(TreeNode root, List<Integer> result) {
    if (root == null) {
        return;
    }

    postorder(root.left, result);
    postorder(root.right, result);

    result.add(root.val);
}
```

---

# 9. BFS

Breadth-First Search processes nodes level by level.

For a tree:

```text
             1
           /   \
          2     3
         / \   / \
        4   5 6   7
```

BFS order:

```text
1 2 3 4 5 6 7
```

Use:

```text
Queue
```

This is one of the most important connections between the Queue topic and Trees.

---

# 10. Level-Order Traversal

Level order is BFS where the result is grouped by depth.

Example:

```text
[
    [1],
    [2,3],
    [4,5,6,7]
]
```

---

# 10.1 Java

```java
static List<List<Integer>> levelOrder(TreeNode root) {
    List<List<Integer>> result = new ArrayList<>();

    if (root == null) {
        return result;
    }

    Queue<TreeNode> queue = new ArrayDeque<>();
    queue.offer(root);

    while (!queue.isEmpty()) {
        int size = queue.size();

        List<Integer> level = new ArrayList<>();

        for (int i = 0; i < size; i++) {
            TreeNode node = queue.poll();

            level.add(node.val);

            if (node.left != null) {
                queue.offer(node.left);
            }

            if (node.right != null) {
                queue.offer(node.right);
            }
        }

        result.add(level);
    }

    return result;
}
```

Complexity:

```text
Time  = O(n)
Space = O(w)
```

where:

```text
w = maximum width of the tree
```

Worst case:

```text
w = O(n)
```

---

# 11. DFS vs BFS

| Property | DFS | BFS |
|---|---|---|
| Main structure | Stack / recursion | Queue |
| Traversal style | Deep first | Level first |
| Typical space | O(h) | O(w) |
| Shortest depth in unweighted tree | Not directly | Natural |
| Path problems | Very common | Also useful |
| Level-based problems | Less direct | Natural |
| Views by level | Possible | Natural |

---

# 12. Recursive vs Iterative Traversal

Recursive traversal uses the call stack.

Iterative traversal explicitly uses:

```text
Stack
```

Why learn both?

- Interviews may ask for iterative traversal.
- Deep trees can cause recursion stack overflow.
- Iterative traversal reveals the actual traversal mechanism.
- Postorder iteration requires careful stack handling.

---

# 13. Iterative Preorder

Preorder:

```text
Root → Left → Right
```

Use a stack.

Because stack is LIFO:

```text
push right first
push left second
```

so that left is processed first.

```java
static List<Integer> preorderIterative(TreeNode root) {
    List<Integer> result = new ArrayList<>();

    if (root == null) {
        return result;
    }

    Deque<TreeNode> stack = new ArrayDeque<>();
    stack.push(root);

    while (!stack.isEmpty()) {
        TreeNode node = stack.pop();

        result.add(node.val);

        if (node.right != null) {
            stack.push(node.right);
        }

        if (node.left != null) {
            stack.push(node.left);
        }
    }

    return result;
}
```

Complexity:

```text
O(n) time
O(h) to O(n) stack space
```

---

# 14. Iterative Inorder

Use a stack to simulate:

```text
go left
process node
go right
```

```java
static List<Integer> inorderIterative(TreeNode root) {
    List<Integer> result = new ArrayList<>();

    Deque<TreeNode> stack = new ArrayDeque<>();
    TreeNode current = root;

    while (current != null || !stack.isEmpty()) {

        while (current != null) {
            stack.push(current);
            current = current.left;
        }

        current = stack.pop();

        result.add(current.val);

        current = current.right;
    }

    return result;
}
```

This pattern is extremely important.

---

# 15. Iterative Postorder

Postorder:

```text
Left → Right → Root
```

One approach uses two stacks.

```java
static List<Integer> postorderTwoStacks(TreeNode root) {
    List<Integer> result = new ArrayList<>();

    if (root == null) {
        return result;
    }

    Deque<TreeNode> s1 = new ArrayDeque<>();
    Deque<TreeNode> s2 = new ArrayDeque<>();

    s1.push(root);

    while (!s1.isEmpty()) {
        TreeNode node = s1.pop();

        s2.push(node);

        if (node.left != null) {
            s1.push(node.left);
        }

        if (node.right != null) {
            s1.push(node.right);
        }
    }

    while (!s2.isEmpty()) {
        result.add(s2.pop().val);
    }

    return result;
}
```

This works because the first stack generates a root-right-left ordering and the second reverses it.

---

# 16. Iterative Postorder With One Stack

A more advanced method uses:

```text
stack
lastVisited
current
```

```java
static List<Integer> postorderOneStack(TreeNode root) {
    List<Integer> result = new ArrayList<>();

    Deque<TreeNode> stack = new ArrayDeque<>();
    TreeNode current = root;
    TreeNode lastVisited = null;

    while (current != null || !stack.isEmpty()) {

        if (current != null) {
            stack.push(current);
            current = current.left;
        } else {
            TreeNode peek = stack.peek();

            if (peek.right != null &&
                lastVisited != peek.right) {

                current = peek.right;

            } else {
                result.add(peek.val);
                lastVisited = stack.pop();
            }
        }
    }

    return result;
}
```

Know the two-stack version first. Master the one-stack version afterward.

---

# 17. Height / Maximum Depth

Height can be computed recursively.

```java
static int height(TreeNode root) {
    if (root == null) {
        return -1;
    }

    return 1 + Math.max(
        height(root.left),
        height(root.right)
    );
}
```

Under the convention:

```text
empty tree = -1
leaf = 0
```

If the problem uses node count instead:

```java
if (root == null) {
    return 0;
}

return 1 + Math.max(...);
```

---

# 18. Maximum Depth

LeetCode-style maximum depth commonly counts nodes:

```text
empty = 0
leaf = 1
```

```java
static int maxDepth(TreeNode root) {
    if (root == null) {
        return 0;
    }

    return 1 + Math.max(
        maxDepth(root.left),
        maxDepth(root.right)
    );
}
```

Do not mix the two height conventions.

---

# 19. Diameter of a Binary Tree

The diameter is the longest path between two nodes.

Depending on the problem:

```text
diameter measured in edges
```

or:

```text
diameter measured in nodes
```

The most common algorithmic definition uses edges.

Example:

```text
             1
           /   \
          2     3
         / \
        4   5
```

Longest path:

```text
4 → 2 → 1 → 3
```

Number of edges:

```text
3
```

---

# 19.1 Key Observation

At node `root`:

```text
leftHeight
rightHeight
```

A path passing through this node has:

```text
leftHeight + rightHeight
```

edges.

Therefore:

```text
diameter =
max(
    leftHeight + rightHeight,
    diameter(left),
    diameter(right)
)
```

Compute height once while updating the answer.

---

# 19.2 Java

```java
static int diameter = 0;

static int diameter(TreeNode root) {
    diameter = 0;
    heightForDiameter(root);
    return diameter;
}

static int heightForDiameter(TreeNode root) {
    if (root == null) {
        return 0;
    }

    int left = heightForDiameter(root.left);
    int right = heightForDiameter(root.right);

    diameter = Math.max(
        diameter,
        left + right
    );

    return 1 + Math.max(left, right);
}
```

Here height is measured in nodes below the null boundary, so:

```text
left + right
```

produces diameter in edges.

Complexity:

```text
O(n)
```

---

# 20. Balanced Binary Tree

A binary tree is height-balanced if at every node:

```text
|height(left) - height(right)| <= 1
```

The naive approach computes height separately at every node.

Worst case:

```text
O(n²)
```

Better:

> Compute height bottom-up and return an invalid marker as soon as imbalance is detected.

---

# 20.1 Java

```java
static boolean isBalanced(TreeNode root) {
    return checkHeight(root) != -1;
}

static int checkHeight(TreeNode root) {
    if (root == null) {
        return 0;
    }

    int left = checkHeight(root.left);

    if (left == -1) {
        return -1;
    }

    int right = checkHeight(root.right);

    if (right == -1) {
        return -1;
    }

    if (Math.abs(left - right) > 1) {
        return -1;
    }

    return 1 + Math.max(left, right);
}
```

Complexity:

```text
O(n)
```

This is an important tree pattern:

```text
return useful information
+
return failure marker
```

---

# 21. Symmetric Tree

A tree is symmetric if its left and right subtrees are mirror images.

Example:

```text
        1
       / \
      2   2
     / \ / \
    3  4 4  3
```

is symmetric.

---

# 21.1 Mirror Condition

Two trees are mirrors if:

```text
left.left  ↔ right.right
left.right ↔ right.left
```

and:

```text
values are equal
```

---

# 21.2 Recursive Java

```java
static boolean isSymmetric(TreeNode root) {
    if (root == null) {
        return true;
    }

    return isMirror(root.left, root.right);
}

static boolean isMirror(TreeNode a, TreeNode b) {
    if (a == null && b == null) {
        return true;
    }

    if (a == null || b == null) {
        return false;
    }

    if (a.val != b.val) {
        return false;
    }

    return isMirror(a.left, b.right)
        && isMirror(a.right, b.left);
}
```

Complexity:

```text
O(n)
```

---

# 22. Path Sum

A common question:

> Does there exist a root-to-leaf path whose values sum to `targetSum`?

At each node:

```text
remaining = target - node.val
```

At a leaf:

```text
remaining == node.val
```

or equivalently:

```text
target == 0
```

after subtracting the current node.

---

# 22.1 Java

```java
static boolean hasPathSum(
        TreeNode root,
        int targetSum) {

    if (root == null) {
        return false;
    }

    if (root.left == null && root.right == null) {
        return targetSum == root.val;
    }

    int remaining = targetSum - root.val;

    return hasPathSum(root.left, remaining)
        || hasPathSum(root.right, remaining);
}
```

Important:

> A root-to-leaf path must actually end at a leaf.

Do not return true at an arbitrary node just because the running sum matches.

---

# 23. All Root-to-Leaf Paths

If you need all paths:

```text
DFS
+
current path
+
backtracking
```

Pattern:

```java
path.add(node.val);

dfs(node.left);
dfs(node.right);

path.remove(path.size() - 1);
```

This is the same backtracking principle used in arrays, graphs, and combinatorial problems.

---

# 24. Maximum Path Sum

This is more subtle than root-to-leaf path sum.

A path may:

- Start anywhere.
- End anywhere.
- Pass through a node.
- Use at most one left branch and one right branch at a node.

Example:

```text
       -10
       /  \
      9   20
          / \
         15  7
```

Maximum path:

```text
15 → 20 → 7
```

sum:

```text
42
```

---

# 24.1 Key Insight

At each node calculate:

### Gain returned to parent

Only one side can be continued upward:

```text
node.val + max(leftGain, rightGain, 0)
```

### Best path through current node

Both sides may be used:

```text
leftGain + node.val + rightGain
```

Update global answer.

---

# 24.2 Java

```java
static int maxPathSum(TreeNode root) {
    int[] answer = {Integer.MIN_VALUE};

    maxGain(root, answer);

    return answer[0];
}

static int maxGain(
        TreeNode root,
        int[] answer) {

    if (root == null) {
        return 0;
    }

    int left = Math.max(
        0,
        maxGain(root.left, answer)
    );

    int right = Math.max(
        0,
        maxGain(root.right, answer)
    );

    int throughRoot =
        left + root.val + right;

    answer[0] = Math.max(
        answer[0],
        throughRoot
    );

    return root.val + Math.max(left, right);
}
```

The distinction is critical:

```text
returned value
=
best one-sided gain

global answer
=
best two-sided path
```

---

# 25. Lowest Common Ancestor

The Lowest Common Ancestor (LCA) of two nodes is the deepest node that is an ancestor of both.

Example:

```text
             1
           /   \
          2     3
         / \
        4   5
```

LCA of:

```text
4 and 5
```

is:

```text
2
```

LCA of:

```text
4 and 3
```

is:

```text
1
```

---

# 26. LCA in a General Binary Tree

For a general binary tree:

```text
if root == null
or root == p
or root == q
    return root
```

Then search both subtrees.

If both sides return non-null:

```text
root is LCA
```

Otherwise return the non-null side.

---

# 26.1 Java

```java
static TreeNode lowestCommonAncestor(
        TreeNode root,
        TreeNode p,
        TreeNode q) {

    if (root == null ||
        root == p ||
        root == q) {
        return root;
    }

    TreeNode left =
        lowestCommonAncestor(root.left, p, q);

    TreeNode right =
        lowestCommonAncestor(root.right, p, q);

    if (left != null && right != null) {
        return root;
    }

    return left != null ? left : right;
}
```

Complexity:

```text
O(n)
```

Space:

```text
O(h)
```

assuming recursive call stack.

---

# 27. LCA in a BST

A BST provides ordering information.

For:

```text
p < root < q
```

the root is the LCA.

If both are smaller:

```text
go left
```

If both are larger:

```text
go right
```

This can reduce the search to tree height.

This is one reason to distinguish:

```text
binary tree
```

from:

```text
BST
```

---

# 28. Serialize / Deserialize

Serialization converts a tree into a storable/transmittable representation.

Example:

```text
        1
       / \
      2   3
```

could become:

```text
1,2,#,#,3,#,#
```

where:

```text
#
=
null
```

The serialization must preserve enough structure to reconstruct the exact tree.

---

# 29. Preorder Serialization

Use:

```text
node
left subtree
right subtree
```

For null:

```text
#
```

Example:

```text
        1
       / \
      2   3
```

serialization:

```text
1,2,#,#,3,#,#
```

---

# 29.1 Java

```java
static void serialize(
        TreeNode root,
        StringBuilder sb) {

    if (root == null) {
        sb.append("#,");
        return;
    }

    sb.append(root.val).append(",");

    serialize(root.left, sb);
    serialize(root.right, sb);
}

static String serialize(TreeNode root) {
    StringBuilder sb = new StringBuilder();

    serialize(root, sb);

    return sb.toString();
}
```

---

# 30. Deserialize

Read tokens in the same preorder order.

If token is:

```text
#
```

return null.

Otherwise:

```text
create node
deserialize left
deserialize right
```

---

# 30.1 Java

```java
static TreeNode deserialize(String data) {
    String[] tokens = data.split(",");
    int[] index = {0};

    return deserialize(tokens, index);
}

static TreeNode deserialize(
        String[] tokens,
        int[] index) {

    String token = tokens[index[0]++];

    if (token.equals("#")) {
        return null;
    }

    TreeNode node =
        new TreeNode(Integer.parseInt(token));

    node.left =
        deserialize(tokens, index);

    node.right =
        deserialize(tokens, index);

    return node;
}
```

The serialization and deserialization procedures must use exactly the same structural convention.

---

# 31. Construct Tree From Preorder + Inorder

Given:

```text
preorder
inorder
```

construct the unique binary tree assuming values are unique.

Key fact:

```text
preorder first element = root
```

In inorder:

```text
left of root → left subtree
right of root → right subtree
```

---

# 31.1 Efficient Strategy

Build:

```text
value → inorder index
```

using a hashmap.

Then recursively split the ranges.

---

# 31.2 Java

```java
static TreeNode buildTree(
        int[] preorder,
        int[] inorder) {

    Map<Integer, Integer> position =
        new HashMap<>();

    for (int i = 0; i < inorder.length; i++) {
        position.put(inorder[i], i);
    }

    return build(
        preorder,
        0,
        preorder.length - 1,
        inorder,
        0,
        inorder.length - 1,
        position
    );
}

static TreeNode build(
        int[] preorder,
        int preLeft,
        int preRight,
        int[] inorder,
        int inLeft,
        int inRight,
        Map<Integer, Integer> position) {

    if (preLeft > preRight ||
        inLeft > inRight) {
        return null;
    }

    int rootValue = preorder[preLeft];

    TreeNode root = new TreeNode(rootValue);

    int rootIndex = position.get(rootValue);

    int leftSize = rootIndex - inLeft;

    root.left = build(
        preorder,
        preLeft + 1,
        preLeft + leftSize,
        inorder,
        inLeft,
        rootIndex - 1,
        position
    );

    root.right = build(
        preorder,
        preLeft + leftSize + 1,
        preRight,
        inorder,
        rootIndex + 1,
        inRight,
        position
    );

    return root;
}
```

Complexity:

```text
O(n) time
O(n) hashmap + O(h) recursion
```

---

# 32. Construct Tree From Inorder + Postorder

Key fact:

```text
postorder last element = root
```

Then locate root in inorder.

The same partitioning idea applies.

The major difference is which side of the postorder range is processed first.

---

# 33. Important Traversal Facts

For a binary tree:

```text
Preorder:
Root Left Right

Inorder:
Left Root Right

Postorder:
Left Right Root
```

Root position:

```text
Preorder  → first
Inorder   → middle
Postorder → last
```

This is the key to reconstruction problems.

---

# 34. Can Any Two Traversals Construct a Tree?

For a general binary tree with distinct values:

```text
preorder + inorder
```

can uniquely determine the tree.

```text
postorder + inorder
```

can uniquely determine the tree.

But:

```text
preorder + postorder
```

does not generally uniquely determine a binary tree unless additional constraints exist, such as a full binary tree.

This is a common theoretical question.

---

# 35. Flatten Binary Tree to Linked List

Given:

```text
        1
       / \
      2   5
     / \   \
    3   4   6
```

flatten into:

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

Required structure:

```text
left = null
right = next node
```

The required ordering is usually preorder.

---

# 35.1 Recursive Strategy

For each node:

1. Flatten left subtree.
2. Flatten right subtree.
3. Move flattened left subtree between node and right subtree.

Need to preserve the original right subtree before modifying pointers.

---

# 35.2 Java

```java
static void flatten(TreeNode root) {
    if (root == null) {
        return;
    }

    flatten(root.left);
    flatten(root.right);

    TreeNode rightSubtree = root.right;

    root.right = root.left;
    root.left = null;

    TreeNode current = root;

    while (current.right != null) {
        current = current.right;
    }

    current.right = rightSubtree;
}
```

This straightforward recursive version can become O(n²) on certain tree shapes because of repeated scans.

A more efficient approach carries the tail or uses reverse preorder.

---

# 36. Flatten With Reverse Preorder

Process:

```text
Right → Left → Root
```

Maintain:

```text
previous
```

Then:

```text
root.right = previous
root.left = null
previous = root
```

---

# 36.1 Java

```java
static TreeNode previous = null;

static void flattenEfficient(TreeNode root) {
    if (root == null) {
        return;
    }

    flattenEfficient(root.right);
    flattenEfficient(root.left);

    root.right = previous;
    root.left = null;

    previous = root;
}
```

Reset `previous` before a new independent call.

Complexity:

```text
O(n)
```

---

# 37. Boundary Traversal

Boundary traversal typically consists of:

```text
1. Root
2. Left boundary
3. All leaves
4. Right boundary in reverse
```

Avoid duplicating leaves.

Example:

```text
             1
           /   \
          2     3
         / \   / \
        4   5 6   7
```

Boundary:

```text
1 2 4 5 6 7 3
```

The exact ordering convention should be checked against the problem statement.

---

# 38. Boundary Traversal Strategy

### Step 1

Add root if it is not a leaf.

### Step 2

Traverse left boundary excluding leaves.

### Step 3

Traverse all leaves from left to right.

### Step 4

Traverse right boundary excluding leaves and reverse it.

Important:

> Leaves should generally be handled separately to prevent duplication.

---

# 39. Boundary Traversal Java

```java
static List<Integer> boundary(TreeNode root) {
    List<Integer> result = new ArrayList<>();

    if (root == null) {
        return result;
    }

    if (!isLeaf(root)) {
        result.add(root.val);
    }

    addLeftBoundary(root.left, result);
    addLeaves(root, result);
    addRightBoundary(root.right, result);

    return result;
}

static boolean isLeaf(TreeNode node) {
    return node != null &&
           node.left == null &&
           node.right == null;
}

static void addLeftBoundary(
        TreeNode root,
        List<Integer> result) {

    TreeNode current = root;

    while (current != null) {
        if (!isLeaf(current)) {
            result.add(current.val);
        }

        if (current.left != null) {
            current = current.left;
        } else {
            current = current.right;
        }
    }
}

static void addLeaves(
        TreeNode root,
        List<Integer> result) {

    if (root == null) {
        return;
    }

    if (isLeaf(root)) {
        result.add(root.val);
        return;
    }

    addLeaves(root.left, result);
    addLeaves(root.right, result);
}

static void addRightBoundary(
        TreeNode root,
        List<Integer> result) {

    List<Integer> temp = new ArrayList<>();

    TreeNode current = root;

    while (current != null) {
        if (!isLeaf(current)) {
            temp.add(current.val);
        }

        if (current.right != null) {
            current = current.right;
        } else {
            current = current.left;
        }
    }

    for (int i = temp.size() - 1; i >= 0; i--) {
        result.add(temp.get(i));
    }
}
```

---

# 40. Vertical Traversal

Vertical traversal groups nodes according to horizontal position.

Assign:

```text
root → column 0
left  → column -1
right → column +1
```

Example:

```text
        1
       / \
      2   3
     / \   \
    4   5   6
```

Coordinates:

```text
1 → column 0
2 → column -1
3 → column 1
4 → column -2
5 → column 0
6 → column 2
```

Therefore columns are grouped:

```text
[-2] → [4]
[-1] → [2]
[ 0] → [1,5]
[ 1] → [3]
[ 2] → [6]
```

---

# 41. Vertical Traversal Tie-Breaking

Different problems define different ordering rules.

A common specification is:

```text
1. column ascending
2. row ascending
3. value ascending when row and column are equal
```

Therefore coordinates alone may not be sufficient.

You may need:

```text
(row, column, value)
```

plus sorting.

---

# 42. Vertical Traversal With BFS

BFS naturally gives row ordering.

Store:

```text
(node, row, column)
```

Then group by column.

A robust general approach is:

```text
Map<column, List<NodeInfo>>
```

followed by sorting each column according to the problem's tie rules.

---

# 43. Vertical Traversal Java Pattern

```java
static class NodeInfo {
    TreeNode node;
    int row;
    int col;

    NodeInfo(TreeNode node, int row, int col) {
        this.node = node;
        this.row = row;
        this.col = col;
    }
}
```

Collect:

```java
List<NodeInfo> nodes = new ArrayList<>();
```

Then sort:

```java
nodes.sort((a, b) -> {
    if (a.col != b.col) {
        return Integer.compare(a.col, b.col);
    }

    if (a.row != b.row) {
        return Integer.compare(a.row, b.row);
    }

    return Integer.compare(a.node.val, b.node.val);
});
```

Group equal columns.

The exact tie-breaking must follow the problem definition.

---

# 44. Right View

The right view contains the node visible from the right side.

Example:

```text
        1
       / \
      2   3
       \   \
        5   6
```

Right view:

```text
1 3 6
```

At each depth, take the rightmost node.

---

# 45. Right View Using BFS

For each level:

```text
process all nodes
take the last node
```

```java
static List<Integer> rightSideView(TreeNode root) {
    List<Integer> result = new ArrayList<>();

    if (root == null) {
        return result;
    }

    Queue<TreeNode> queue = new ArrayDeque<>();
    queue.offer(root);

    while (!queue.isEmpty()) {
        int size = queue.size();

        for (int i = 0; i < size; i++) {
            TreeNode node = queue.poll();

            if (i == size - 1) {
                result.add(node.val);
            }

            if (node.left != null) {
                queue.offer(node.left);
            }

            if (node.right != null) {
                queue.offer(node.right);
            }
        }
    }

    return result;
}
```

---

# 46. Right View Using DFS

Process:

```text
Right → Left
```

The first node encountered at each depth is the visible node.

```java
static void rightView(
        TreeNode root,
        int depth,
        List<Integer> result) {

    if (root == null) {
        return;
    }

    if (depth == result.size()) {
        result.add(root.val);
    }

    rightView(root.right, depth + 1, result);
    rightView(root.left, depth + 1, result);
}
```

Wrapper:

```java
static List<Integer> rightSideViewDFS(TreeNode root) {
    List<Integer> result = new ArrayList<>();

    rightView(root, 0, result);

    return result;
}
```

---

# 47. Left View

Mirror image of right view.

At each depth:

```text
take the leftmost visible node
```

BFS:

```text
take first node of each level
```

DFS:

```text
Left → Right
```

and record the first node encountered at each depth.

---

# 48. Left View Java

```java
static void leftView(
        TreeNode root,
        int depth,
        List<Integer> result) {

    if (root == null) {
        return;
    }

    if (depth == result.size()) {
        result.add(root.val);
    }

    leftView(root.left, depth + 1, result);
    leftView(root.right, depth + 1, result);
}

static List<Integer> leftSideView(TreeNode root) {
    List<Integer> result = new ArrayList<>();

    leftView(root, 0, result);

    return result;
}
```

---

# 49. Tree DP Pattern

Many tree problems can be expressed as:

```text
solve(node)
```

where the recursive function returns information about the subtree.

Examples:

### Height

```text
return height
```

### Diameter

```text
return height
update global diameter
```

### Balanced tree

```text
return height
or -1 if invalid
```

### Maximum path sum

```text
return best one-sided gain
update global two-sided answer
```

This is often called:

```text
Tree DP
```

or:

```text
postorder dynamic programming
```

because children are solved before the parent.

---

# 50. The Most Important Tree Pattern

For many binary-tree problems:

```text
solve(left)
solve(right)
combine
return information
```

Generic form:

```java
static Info solve(TreeNode root) {
    if (root == null) {
        return baseCase;
    }

    Info left = solve(root.left);
    Info right = solve(root.right);

    Info current = combine(
        root,
        left,
        right
    );

    return current;
}
```

Mastering this pattern makes many "hard" tree problems much easier to derive.

---

# 51. Global Answer vs Return Value

A common source of confusion:

The recursive function may return one quantity while the final answer needs another.

Example: maximum path sum.

Return:

```text
best one-sided path that can connect to parent
```

Global:

```text
best path anywhere in tree
```

Similarly, diameter:

Return:

```text
height
```

Global:

```text
maximum leftHeight + rightHeight
```

This distinction is fundamental.

---

# 52. Null as a Base Case

Most recursive tree functions begin with:

```java
if (root == null) {
    return ...;
}
```

The correct return value depends on the problem.

Examples:

### Height in node-count convention

```java
return 0;
```

### Height in edge-count convention

```java
return -1;
```

### Maximum path gain

```java
return 0;
```

### Search

```java
return null;
```

Do not blindly use the same null value for every tree problem.

---

# 53. Path Problems: Root-to-Leaf vs Any-to-Any

This distinction is critical.

## Root-to-leaf

Must:

```text
start at root
end at leaf
```

Example:

```text
Path Sum
```

## Any-to-any

Can:

```text
start anywhere
end anywhere
```

Example:

```text
Maximum Path Sum
```

Any-to-any problems usually require more careful tree DP.

---

# 54. Diameter vs Maximum Path Sum

Both often use:

```text
left contribution
+
right contribution
```

But the quantities differ.

### Diameter

Usually:

```text
length / number of edges
```

### Maximum Path Sum

Uses:

```text
node values
```

and may contain negative values.

The shared structure is:

```text
postorder
→ calculate child contributions
→ combine at current node
```

---

# 55. Serialize / Deserialize Invariant

For preorder serialization:

```text
serialize(node)
=
node
+ serialize(left)
+ serialize(right)
```

The deserializer must consume exactly the same sequence:

```text
read node
→ build left
→ build right
```

The key invariant is:

> The serializer and deserializer agree on both traversal order and null representation.

Without null markers, different tree shapes can produce the same value sequence.

---

# 56. Traversal Reconstruction

For:

```text
preorder + inorder
```

process:

```text
preorder first element
→ root
→ locate root in inorder
→ split left/right
→ recursively construct
```

For:

```text
postorder + inorder
```

process:

```text
postorder last element
→ root
→ locate root in inorder
→ split left/right
→ recursively construct
```

The hashmap eliminates repeated inorder searches.

---

# 57. Boundary vs Vertical vs Views

These are often confused.

### Boundary traversal

Walk the outer boundary:

```text
root
left boundary
leaves
right boundary reversed
```

### Vertical traversal

Group nodes by:

```text
horizontal column
```

### Left/right view

Select:

```text
one visible node per depth
```

They require different coordinate/order logic.

---

# 58. BFS Level Template

Memorize this template:

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

This solves many problems involving:

- Level order.
- Minimum depth.
- Left view.
- Right view.
- Level averages.
- Level maximums.
- Zigzag traversal.
- Per-level aggregation.

---

# 59. DFS Template

Basic recursive template:

```java
static void dfs(TreeNode root) {
    if (root == null) {
        return;
    }

    // preorder work

    dfs(root.left);

    // inorder work

    dfs(root.right);

    // postorder work
}
```

The location of the operation determines the traversal.

```text
Before children → preorder
Between children → inorder
After children → postorder
```

This is one of the most important concepts in tree recursion.

---

# 60. Iterative Traversal Insight

Recursion implicitly stores:

```text
which node
which child is being processed
where to return
```

The explicit stack must reproduce this state.

Therefore iterative traversal is not a completely different algorithm.

It is:

```text
recursive call stack
→ explicitly managed stack
```

Understanding this makes iterative tree traversal much easier.

---

# 61. Complexity of Common Tree Operations

For a tree with `n` nodes and height `h`:

| Problem | Time | Extra Space |
|---|---:|---:|
| Preorder | O(n) | O(h) |
| Inorder | O(n) | O(h) |
| Postorder | O(n) | O(h) |
| BFS | O(n) | O(w) |
| Height | O(n) | O(h) |
| Diameter | O(n) | O(h) |
| Balanced tree | O(n) | O(h) |
| Symmetry | O(n) | O(h) |
| Path sum | O(n) | O(h) |
| Maximum path sum | O(n) | O(h) |
| General-tree LCA | O(n) | O(h) |
| Serialize | O(n) | O(n) output + O(h) stack |
| Deserialize | O(n) | O(n) output/tree + O(h) stack |
| Build from traversals | O(n) | O(n) |
| Flatten | O(n) with efficient method | O(h) |
| Boundary traversal | O(n) | O(h) / O(n) output |
| Vertical traversal | O(n log n) typical sorting approach | O(n) |
| Left/right view | O(n) | O(w) BFS or O(h) DFS |

---

# 62. Common Tree Bugs

## Bug 1 — Confusing height and depth

```text
depth = root → node
height = node → deepest leaf
```

They are not interchangeable.

---

## Bug 2 — Wrong height convention

Some problems use:

```text
null = 0
leaf = 1
```

Others use:

```text
null = -1
leaf = 0
```

Read the problem carefully.

---

## Bug 3 — Forgetting leaf requirement

For root-to-leaf path sum, reaching an internal node with the correct sum is not enough.

---

## Bug 4 — Losing the original right subtree

When flattening:

```java
root.right = root.left;
```

you must preserve the original right subtree first.

---

## Bug 5 — Duplicate leaves in boundary traversal

Do not include leaves again in the left/right boundary lists.

---

## Bug 6 — Wrong vertical tie-breaking

Column ordering alone may not satisfy the problem.

Check:

```text
column
row
value
```

requirements.

---

## Bug 7 — Reusing global state

If using:

```java
static int diameter;
```

reset it before solving a new tree.

---

## Bug 8 — Confusing general binary tree LCA with BST LCA

BST LCA uses ordering.

General binary-tree LCA does not.

---

## Bug 9 — Assuming recursion is always safe

A highly skewed tree can have:

```text
h = n
```

and recursion may overflow the stack.

---

# 63. Tree Problem Recognition

## Type A — Traversal

Keywords:

```text
preorder
inorder
postorder
level order
BFS
DFS
```

Use:

```text
stack / recursion / queue
```

---

## Type B — Tree Property

Keywords:

```text
height
balanced
symmetric
identical
depth
```

Use:

```text
recursive DFS
```

---

## Type C — Tree DP

Keywords:

```text
diameter
maximum path
best subtree
minimum cost
maximum gain
```

Use:

```text
postorder
return child information
combine at node
```

---

## Type D — Path

Keywords:

```text
root-to-leaf
path sum
maximum path
ancestor
```

Use:

```text
DFS
+
path state / subtree DP
```

---

## Type E — Level

Keywords:

```text
level
depth
each row
visible from side
minimum depth
```

Use:

```text
BFS
```

or depth-aware DFS.

---

## Type F — Reconstruction

Keywords:

```text
construct tree
given preorder/inorder
given inorder/postorder
```

Use:

```text
traversal properties
+
hashmap
+
recursion
```

---

## Type G — Serialization

Keywords:

```text
serialize
deserialize
encode
decode
store tree
```

Use:

```text
DFS/BFS
+
null markers
```

---

## Type H — Views / Coordinates

Keywords:

```text
vertical
boundary
left view
right view
```

Use:

```text
BFS/DFS
+
depth/column
+
ordering rules
```

---

# 64. High-Value LeetCode Roadmap

## Easy

Master:

- Maximum Depth of Binary Tree.
- Same Tree.
- Symmetric Tree.
- Invert Binary Tree.
- Minimum Depth of Binary Tree.
- Binary Tree Preorder Traversal.
- Binary Tree Inorder Traversal.
- Binary Tree Postorder Traversal.
- Binary Tree Level Order Traversal.
- Path Sum.

## Medium

Master:

- Binary Tree Right Side View.
- Binary Tree Zigzag Level Order Traversal.
- Lowest Common Ancestor of a Binary Tree.
- Construct Binary Tree from Preorder and Inorder Traversal.
- Construct Binary Tree from Inorder and Postorder Traversal.
- Flatten Binary Tree to Linked List.
- Diameter of Binary Tree.
- Balanced Binary Tree.
- Path Sum II.
- Boundary Traversal variants.
- Vertical traversal variants.
- Serialize and Deserialize Binary Tree.

## Hard / Advanced

Master:

- Binary Tree Maximum Path Sum.
- Advanced vertical traversal.
- Complex tree construction.
- Tree DP variants.
- Serialization with additional constraints.
- Path optimization on trees.
- Advanced view/coordinate problems.

Difficulty can vary by platform/version; use the roadmap by pattern rather than relying only on labels.

---

# 65. GATE / CS Theory Focus

Know these precisely:

### Tree traversal

```text
Preorder = Root Left Right
Inorder  = Left Root Right
Postorder = Left Right Root
```

### BFS

Uses:

```text
Queue
```

### DFS

Uses:

```text
Stack / recursion
```

### Binary tree property

Maximum nodes at level `d`:

```text
2^d
```

assuming root is level `0`.

Maximum nodes in a binary tree of height `h` measured in edges:

```text
2^(h+1) - 1
```

### Minimum possible height for n nodes

Approximately:

```text
ceil(log2(n + 1)) - 1
```

under edge-based height.

### Worst-case height

```text
n - 1
```

for a skewed tree.

### Balanced tree

Subtree heights differ by at most the allowed balance threshold, commonly:

```text
1
```

---

# 66. GATE-Style Solved Questions

> These are **GATE-style practice questions**, not claims about a particular official GATE year.

---

## Question 1 — Traversal

Given:

```text
        1
       / \
      2   3
     / \
    4   5
```

What are the preorder, inorder, and postorder traversals?

### Solution

Preorder:

```text
Root → Left → Right

1 2 4 5 3
```

Inorder:

```text
Left → Root → Right

4 2 5 1 3
```

Postorder:

```text
Left → Right → Root

4 5 2 3 1
```

---

## Question 2 — Maximum Nodes

What is the maximum number of nodes in a binary tree of height `4`, where height is measured in edges?

### Solution

Maximum nodes:

```text
2^(h+1) - 1
```

For:

```text
h = 4
```

there are levels:

```text
0,1,2,3,4
```

Therefore:

```text
1 + 2 + 4 + 8 + 16
= 31
```

Answer:

```text
31
```

---

## Question 3 — Tree DP

Consider:

```text
       -10
       /  \
      9    20
          /  \
         15   7
```

What is the maximum path sum?

### Solution

At node `20`:

```text
left gain = 15
right gain = 7
```

Path through `20`:

```text
15 + 20 + 7
= 42
```

Therefore:

```text
Maximum path sum = 42
```

The path is:

```text
15 → 20 → 7
```

---

# 67. Serious Mastery Checklist

## Fundamentals

- [ ] Tree terminology.
- [ ] Root, parent, child, sibling.
- [ ] Leaf/internal node.
- [ ] Depth vs height.
- [ ] Binary tree vs BST.
- [ ] Complete/full/perfect tree concepts.

## DFS

- [ ] Preorder.
- [ ] Inorder.
- [ ] Postorder.
- [ ] Recursive traversal.
- [ ] Iterative preorder.
- [ ] Iterative inorder.
- [ ] Iterative postorder.
- [ ] Understand recursion stack.

## BFS

- [ ] Queue.
- [ ] Level-order traversal.
- [ ] Per-level processing.
- [ ] Width.
- [ ] Minimum depth.
- [ ] Views.

## Tree Properties

- [ ] Height.
- [ ] Diameter.
- [ ] Balanced tree.
- [ ] Symmetric tree.
- [ ] Same tree.
- [ ] Invert tree.

## Path Problems

- [ ] Path Sum.
- [ ] All root-to-leaf paths.
- [ ] Maximum Path Sum.
- [ ] Root-to-leaf vs any-to-any.
- [ ] One-sided return vs global answer.

## LCA

- [ ] General binary tree LCA.
- [ ] BST LCA.
- [ ] Understand ancestor logic.

## Construction / Encoding

- [ ] Serialize.
- [ ] Deserialize.
- [ ] Preorder + inorder construction.
- [ ] Inorder + postorder construction.
- [ ] Traversal uniqueness theory.

## Transformations

- [ ] Flatten tree.
- [ ] Reverse-preorder flattening.
- [ ] Preserve pointers correctly.

## Views / Traversals

- [ ] Boundary traversal.
- [ ] Vertical traversal.
- [ ] Left view.
- [ ] Right view.
- [ ] Coordinate assignment.
- [ ] Tie-breaking.

## Advanced

- [ ] Tree DP.
- [ ] Postorder state aggregation.
- [ ] Coordinate-based traversal.
- [ ] Iterative traversal under deep trees.
- [ ] Complexity in terms of `n`, `h`, and `w`.

---

# 68. Final Revision Sheet

```text
TREES
│
├── TERMINOLOGY
│   ├── Root
│   ├── Parent / Child
│   ├── Leaf
│   ├── Depth
│   └── Height
│
├── DFS
│   ├── Preorder
│   │   └── Root Left Right
│   ├── Inorder
│   │   └── Left Root Right
│   └── Postorder
│       └── Left Right Root
│
├── BFS
│   ├── Queue
│   └── Level order
│
├── PROPERTIES
│   ├── Height
│   ├── Diameter
│   ├── Balanced
│   └── Symmetric
│
├── PATHS
│   ├── Path Sum
│   ├── Root-to-leaf
│   └── Maximum Path Sum
│
├── ANCESTORS
│   └── Lowest Common Ancestor
│
├── ENCODING
│   ├── Serialize
│   └── Deserialize
│
├── CONSTRUCTION
│   ├── Preorder + Inorder
│   └── Inorder + Postorder
│
├── TRANSFORMATION
│   └── Flatten
│
└── SPECIAL TRAVERSALS
    ├── Boundary
    ├── Vertical
    ├── Right View
    └── Left View
```

### Highest-value templates

```text
PREORDER
node
→ left
→ right

INORDER
left
→ node
→ right

POSTORDER
left
→ right
→ node

BFS
queue
→ process level
→ add children

TREE DP
solve(left)
solve(right)
→ combine
→ return state

DIAMETER
child heights
→ left + right
→ update answer
→ return height

MAX PATH SUM
left gain
right gain
→ two-sided path updates answer
→ one-sided gain returned

BALANCED
left height
right height
→ if difference > 1: invalid
→ otherwise return height

LCA
node found in left
+
node found in right
→ current node is LCA

CONSTRUCT
traversal identifies root
→ inorder identifies split
→ recursively construct

VIEWS
depth/level
→ choose first or last node

VERTICAL
row + column
→ sort/group according to tie rules
```

### Final interview principle

When given a binary-tree problem, first identify **what information must move upward from each subtree**.

Ask:

```text
Do I need height?
Do I need a boolean?
Do I need a path sum?
Do I need a maximum gain?
Do I need an ancestor?
Do I need level information?
Do I need coordinates?
```

Then choose:

```text
DFS recursion
BFS queue
explicit stack
tree DP
hashmap + traversal
coordinate sorting
```

The most reusable tree pattern is:

```text
             node
            /    \
        solve    solve
          ↓        ↓
        left      right
             ↓
          combine
             ↓
        return state
```

Once this postorder state-combination pattern is mastered, height, diameter, balance, maximum path sum, subtree properties, and many advanced tree problems become variations of the same underlying technique.
