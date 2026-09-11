# 5. Linked Lists

## 1. What You Must Master

Linked Lists are pointer/reference-based linear data structures. They are heavily tested in coding interviews because the problems are usually less about syntax and more about **reference manipulation, invariants, and pointer movement**.

For FAANG/top product-company interviews, you should be able to:

- Build and traverse linked lists.
- Insert and delete nodes safely.
- Reverse a list iteratively and recursively.
- Reverse nodes in groups.
- Use fast/slow pointers.
- Detect a cycle and find its entry.
- Merge multiple sorted lists.
- Sort a linked list efficiently.
- Remove the nth node from the end.
- Find the intersection of two lists.
- Check whether a linked list is a palindrome.
- Reorder a linked list.
- Deep-copy a list containing random pointers.
- Analyze pointer-based solutions for time and space complexity.

The central skill is:

> **Change links without losing access to the rest of the list.**

---

# 2. Singly Linked List

## 2.1 Structure

A singly linked-list node contains:

```text
data
next
```

Example:

```text
10 → 20 → 30 → null
```

Java:

```java
class ListNode {
    int val;
    ListNode next;

    ListNode(int val) {
        this.val = val;
    }

    ListNode(int val, ListNode next) {
        this.val = val;
        this.next = next;
    }
}
```

The `next` reference points to the next node.

The last node points to:

```text
null
```

---

## 2.2 Head

The first node is usually referenced by:

```java
ListNode head;
```

Example:

```text
head
 ↓
10 → 20 → 30 → null
```

If:

```java
head == null
```

the list is empty.

---

# 3. Doubly Linked List

A doubly linked-list node contains:

```text
prev
data
next
```

Example:

```text
null ← 10 ⇄ 20 ⇄ 30 → null
```

Java:

```java
class DoublyNode {
    int val;
    DoublyNode prev;
    DoublyNode next;

    DoublyNode(int val) {
        this.val = val;
    }
}
```

Advantages:

- Can move forward and backward.
- Deleting a known node can be `O(1)` if its neighbors are available.
- Useful for LRU Cache implementations.
- Useful when both predecessor and successor relationships matter.

Disadvantage:

- More memory.
- More references must be updated correctly.

---

# 4. Circular Linked List

In a circular linked list, the final node does not point to `null`.

Instead:

```text
10 → 20 → 30
↑         ↓
└─────────┘
```

For a singly circular list:

```text
30.next == 10
```

There is no natural `null` termination.

Therefore traversal must use a stopping condition such as:

```java
do {
    // process current
    current = current.next;
} while (current != head);
```

Do not write:

```java
while (current != null)
```

for a circular list.

---

# 5. Traversal

## 5.1 Basic Traversal

```java
public void traverse(ListNode head) {
    ListNode current = head;

    while (current != null) {
        System.out.println(current.val);
        current = current.next;
    }
}
```

Complexity:

```text
Time  : O(n)
Space : O(1)
```

assuming no recursive call stack.

---

## 5.2 Traversal with an Index

```java
int index = 0;

for (ListNode cur = head; cur != null; cur = cur.next) {
    System.out.println(index + " " + cur.val);
    index++;
}
```

---

# 6. Insertion

## 6.1 Insert at Head

Before:

```text
head → 20 → 30
```

Create:

```text
10
```

Then:

```java
newNode.next = head;
head = newNode;
```

After:

```text
head → 10 → 20 → 30
```

Complexity:

```text
O(1)
```

---

## 6.2 Insert After a Known Node

Suppose:

```text
10 → 20 → 40
```

Insert `30` after `20`.

Correct order:

```java
newNode.next = current.next;
current.next = newNode;
```

Result:

```text
10 → 20 → 30 → 40
```

### Critical rule

Do not overwrite:

```text
current.next
```

before saving/accessing the old successor.

---

## 6.3 Insert at Tail

Without a tail pointer:

```text
O(n)
```

With a tail pointer:

```text
O(1)
```

provided the implementation maintains the tail correctly.

---

# 7. Deletion

## 7.1 Delete Head

```java
if (head != null) {
    head = head.next;
}
```

Complexity:

```text
O(1)
```

---

## 7.2 Delete a Node After a Known Predecessor

```java
previous.next = previous.next.next;
```

If:

```text
10 → 20 → 30
```

and `previous` is `10`:

```text
10.next = 30
```

result:

```text
10 → 30
```

---

## 7.3 Delete by Value

For a singly linked list, normally track:

```text
previous
current
```

Then bypass the target:

```java
previous.next = current.next;
```

Special case:

```text
target == head
```

must be handled separately or through a dummy node.

---

# 8. Dummy/Sentinel Node

A dummy node simplifies many linked-list problems.

```java
ListNode dummy = new ListNode(0);
dummy.next = head;
```

Now the list conceptually becomes:

```text
dummy → head → ...
```

This is particularly useful when:

- The head might be deleted.
- The insertion/deletion position can be at the beginning.
- Multiple nodes may be removed.
- You want uniform predecessor logic.

At the end:

```java
return dummy.next;
```

### Interview tip

When a linked-list problem has annoying head edge cases, try a dummy node.

---

# 9. Reverse Linked List

This is one of the most important linked-list problems.

Given:

```text
1 → 2 → 3 → 4 → null
```

produce:

```text
4 → 3 → 2 → 1 → null
```

---

## 9.1 Iterative Reversal

Maintain:

```text
prev
curr
next
```

At each step:

```text
next = curr.next
curr.next = prev
prev = curr
curr = next
```

Java:

```java
public ListNode reverseList(ListNode head) {
    ListNode prev = null;
    ListNode curr = head;

    while (curr != null) {
        ListNode next = curr.next;

        curr.next = prev;
        prev = curr;
        curr = next;
    }

    return prev;
}
```

---

## 9.2 Why Save `next` First?

Suppose:

```text
curr → next
```

If you immediately do:

```java
curr.next = prev;
```

the original successor is no longer reachable through `curr.next`.

Therefore:

```java
ListNode next = curr.next;
```

must happen before changing the link.

This is one of the most important pointer-manipulation rules.

---

## 9.3 Complexity

```text
Time  : O(n)
Space : O(1)
```

---

# 10. Recursive Reversal

The recursive idea:

1. Reverse everything after `head`.
2. Put `head` at the end.
3. Fix the final link.

Java:

```java
public ListNode reverseList(ListNode head) {
    if (head == null || head.next == null) {
        return head;
    }

    ListNode newHead = reverseList(head.next);

    head.next.next = head;
    head.next = null;

    return newHead;
}
```

Complexity:

```text
Time  : O(n)
Space : O(n)
```

The extra `O(n)` is the recursion stack.

For interviews, know both versions, but the iterative solution is generally preferable when constant auxiliary space is desired.

---

# 11. Reverse in Groups

A classic hard linked-list pattern is:

> Reverse every group of `k` nodes.

Example:

```text
1 → 2 → 3 → 4 → 5 → 6
k = 3
```

Result:

```text
3 → 2 → 1 → 6 → 5 → 4
```

---

## 11.1 Core Strategy

For each group:

1. Find the kth node.
2. Save the node after the group.
3. Reverse the group.
4. Connect the previous group to the new group head.
5. Connect the reversed group's tail to the remaining list.
6. Continue.

A dummy node greatly simplifies the implementation.

---

## 11.2 Key Invariant

At the beginning of each iteration:

```text
groupPrev
    ↓
first node of next group
```

Find:

```text
kth
```

Then:

```text
groupNext = kth.next
```

Reverse only the group.

---

## 11.3 Complexity

```text
Time  : O(n)
Space : O(1)
```

for the standard iterative solution.

---

# 12. Fast/Slow Pointer

Maintain two references:

```text
slow
fast
```

Typically:

```text
slow = slow.next
fast = fast.next.next
```

Fast moves twice as quickly.

This allows important structural information to be found without knowing the list length.

---

## 12.1 Common Uses

Fast/slow pointers solve or help solve:

- Find middle node.
- Detect cycles.
- Find cycle entry.
- Check palindrome.
- Split a linked list.
- Find certain relative positions.

---

# 13. Find Middle of Linked List

Example:

```text
1 → 2 → 3 → 4 → 5
```

The middle is:

```text
3
```

Java:

```java
public ListNode middleNode(ListNode head) {
    ListNode slow = head;
    ListNode fast = head;

    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }

    return slow;
}
```

For an even-length list:

```text
1 → 2 → 3 → 4
```

this standard version returns:

```text
3
```

the second middle.

---

# 14. Cycle Detection

Use **Floyd's Tortoise and Hare algorithm**.

```text
slow → one step
fast → two steps
```

If a cycle exists, eventually:

```text
slow == fast
```

---

## 14.1 Java

```java
public boolean hasCycle(ListNode head) {
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

---

## 14.2 Why Does It Work?

If the list has a cycle, once both pointers enter the cycle, the faster pointer gains one node per iteration relative to the slower pointer.

Because the cycle is finite, fast eventually catches slow.

If there is no cycle:

```text
fast → null
```

and the loop ends.

---

## 14.3 Complexity

```text
Time  : O(n)
Space : O(1)
```

This is better than using a HashSet, which needs `O(n)` space.

---

# 15. Cycle Entry

Detecting a cycle is not the same as finding where the cycle begins.

Example:

```text
1 → 2 → 3 → 4 → 5
        ↑       ↓
        └───────┘
```

Cycle entry:

```text
3
```

---

## 15.1 Floyd's Method

### Step 1

Run slow/fast pointers until they meet.

### Step 2

Set one pointer to `head`.

### Step 3

Move both one step at a time.

```text
pointer1 = head
pointer2 = meeting point

pointer1 = pointer1.next
pointer2 = pointer2.next
```

Their next meeting point is the cycle entry.

---

## 15.2 Java

```java
public ListNode detectCycle(ListNode head) {
    ListNode slow = head;
    ListNode fast = head;

    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;

        if (slow == fast) {
            ListNode entry = head;

            while (entry != slow) {
                entry = entry.next;
                slow = slow.next;
            }

            return entry;
        }
    }

    return null;
}
```

Complexity:

```text
Time  : O(n)
Space : O(1)
```

---

# 16. Merge Two Sorted Linked Lists

Given:

```text
1 → 3 → 5
2 → 4 → 6
```

Result:

```text
1 → 2 → 3 → 4 → 5 → 6
```

---

## 16.1 Dummy Node Solution

```java
public ListNode mergeTwoLists(ListNode list1, ListNode list2) {
    ListNode dummy = new ListNode(0);
    ListNode tail = dummy;

    while (list1 != null && list2 != null) {
        if (list1.val <= list2.val) {
            tail.next = list1;
            list1 = list1.next;
        } else {
            tail.next = list2;
            list2 = list2.next;
        }

        tail = tail.next;
    }

    tail.next = (list1 != null) ? list1 : list2;

    return dummy.next;
}
```

---

## 16.2 Complexity

If:

```text
n = length(list1)
m = length(list2)
```

then:

```text
Time  : O(n + m)
Space : O(1)
```

assuming existing nodes are reused.

---

# 17. Merge Multiple Sorted Lists

For `k` sorted linked lists, common approaches include:

### Sequential merging

Merge one list at a time.

Complexity can be approximately:

```text
O(kN)
```

depending on distribution of lengths.

### Divide and conquer

Merge pairs:

```text
k lists
 ↓
k/2 lists
 ↓
k/4 lists
 ↓
...
 ↓
1 list
```

Typical complexity:

```text
O(N log k)
```

### Min-heap

Store the current smallest node from every non-empty list.

Each extraction:

```text
remove minimum
add its next node
```

Typical complexity:

```text
O(N log k)
```

Know both divide-and-conquer and heap approaches.

---

# 18. Sort Linked List

A linked list should generally be sorted using **merge sort**.

Why?

Arrays can efficiently use random access, but linked lists cannot.

Merge sort only needs sequential traversal and node-link manipulation.

---

## 18.1 Strategy

```text
Find middle
    ↓
Split list
    ↓
Sort left half
    ↓
Sort right half
    ↓
Merge
```

---

## 18.2 Complexity

```text
Time  : O(n log n)
Space : O(log n) recursion stack
```

A bottom-up iterative merge sort can achieve:

```text
O(n log n) time
O(1) auxiliary space
```

with a more complicated implementation.

---

# 19. Remove Nth Node from End

Example:

```text
1 → 2 → 3 → 4 → 5
n = 2
```

Remove:

```text
4
```

Result:

```text
1 → 2 → 3 → 5
```

---

## 19.1 Two-Pointer Technique

Use:

```text
fast
slow
```

Maintain a gap of `n` nodes.

With a dummy node:

```java
public ListNode removeNthFromEnd(ListNode head, int n) {
    ListNode dummy = new ListNode(0);
    dummy.next = head;

    ListNode slow = dummy;
    ListNode fast = dummy;

    for (int i = 0; i < n; i++) {
        fast = fast.next;
    }

    while (fast.next != null) {
        fast = fast.next;
        slow = slow.next;
    }

    slow.next = slow.next.next;

    return dummy.next;
}
```

---

## 19.2 Why Dummy?

If `n` equals the list length, the head itself must be removed.

The dummy node gives the head a predecessor:

```text
dummy → head
```

so the same deletion logic works.

---

# 20. Intersection of Two Linked Lists

Two linked lists may eventually share the **same node object**.

Important distinction:

```text
same value
```

is not the same as:

```text
same node/reference
```

Example:

```text
A: 1 → 2 ┐
         ├→ 8 → 9
B: 4 → 5 ┘
```

The intersection is the actual node containing `8`.

---

## 20.1 Two-Pointer Solution

```java
public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
    ListNode a = headA;
    ListNode b = headB;

    while (a != b) {
        a = (a == null) ? headB : a.next;
        b = (b == null) ? headA : b.next;
    }

    return a;
}
```

---

## 20.2 Why Does It Work?

Suppose:

```text
A = uniqueA + common
B = uniqueB + common
```

Pointer `a` traverses:

```text
uniqueA + common + uniqueB + common
```

Pointer `b` traverses:

```text
uniqueB + common + uniqueA + common
```

After switching heads, both traverse the same total number of nodes before reaching the intersection.

If there is no intersection, both eventually become:

```text
null
```

---

## 20.3 Complexity

```text
Time  : O(n + m)
Space : O(1)
```

---

# 21. Palindrome Linked List

Example:

```text
1 → 2 → 3 → 2 → 1
```

is a palindrome.

---

## 21.1 O(n) Extra Space

Copy values into an array/list:

```text
[1, 2, 3, 2, 1]
```

Then use two pointers.

Easy, but not optimal in auxiliary space.

---

## 21.2 O(1) Extra Space

Use:

```text
1. Find middle.
2. Reverse second half.
3. Compare both halves.
4. Optionally restore the second half.
```

Example:

```text
1 → 2 → 3 → 2 → 1
          ↓
reverse second half
```

Then compare:

```text
1 ↔ 1
2 ↔ 2
```

---

## 21.3 Complexity

```text
Time  : O(n)
Space : O(1)
```

This is the preferred interview solution when constant auxiliary space is expected.

---

# 22. Reordering Linked List

Given:

```text
1 → 2 → 3 → 4 → 5
```

reorder to:

```text
1 → 5 → 2 → 4 → 3
```

For:

```text
1 → 2 → 3 → 4
```

result:

```text
1 → 4 → 2 → 3
```

---

## 22.1 Three-Part Strategy

This problem is best understood as three standard techniques combined:

### Step 1 — Find middle

Use fast/slow pointers.

### Step 2 — Reverse second half

Use standard linked-list reversal.

### Step 3 — Merge alternately

```text
first → second → first → second → ...
```

This is a high-value interview pattern because it combines multiple fundamental linked-list techniques.

---

# 23. Copy List with Random Pointer

Each node contains:

```text
val
next
random
```

`random` may point to:

- Any node in the list.
- Itself.
- `null`.

---

## 23.1 Why Is This Hard?

For every original node:

```text
original → copied node
```

must be established.

Then:

```text
copy.next
copy.random
```

must point to the corresponding copied nodes.

---

# 24. Copy Random List — HashMap Solution

Store:

```text
original node → cloned node
```

### Pass 1

Create all cloned nodes.

```java
Map<Node, Node> map = new HashMap<>();

Node cur = head;

while (cur != null) {
    map.put(cur, new Node(cur.val));
    cur = cur.next;
}
```

### Pass 2

Set references:

```java
cur = head;

while (cur != null) {
    Node copy = map.get(cur);

    copy.next = map.get(cur.next);
    copy.random = map.get(cur.random);

    cur = cur.next;
}
```

Return:

```java
return map.get(head);
```

---

## 24.1 Complexity

```text
Time  : O(n)
Space : O(n)
```

This is usually the easiest correct solution to explain.

---

# 25. Copy Random List — O(1) Extra Space

There is a more advanced solution.

### Step 1

Insert each copy directly after its original:

```text
A → B → C
```

becomes:

```text
A → A' → B → B' → C → C'
```

### Step 2

Set random pointers.

If:

```text
A.random = C
```

then:

```text
A'.random = C'
```

which is accessible as:

```text
A.random.next
```

### Step 3

Separate the original and copied lists.

Final:

```text
Original:
A → B → C

Copy:
A' → B' → C'
```

Complexity:

```text
Time  : O(n)
Space : O(1) auxiliary
```

This is an important advanced linked-list technique.

---

# 26. Pointer Manipulation Invariant

Before changing a pointer, ask:

> After this assignment, can I still reach every part of the list that I need?

For example:

```java
curr.next = prev;
```

is dangerous if the original successor has not been saved.

Correct:

```java
ListNode next = curr.next;
curr.next = prev;
```

Then continue with:

```java
curr = next;
```

This mental check prevents many linked-list bugs.

---

# 27. Common Edge Cases

Always test:

### Empty list

```text
head = null
```

### One node

```text
1 → null
```

### Two nodes

```text
1 → 2
```

### Deleting head

```text
1 → 2 → 3
```

### Deleting tail

```text
1 → 2 → 3
```

### `n` equals list length

For remove-nth-node problems, this means deleting the head.

### Cycle begins at head

```text
1 → 2 → 3
↑       ↓
└───────┘
```

### No cycle

```text
1 → 2 → 3 → null
```

### No intersection

Two lists eventually terminate independently.

### Complete intersection

Both heads are the same node.

---

# 28. Linked List Complexity

| Operation | Singly Linked List |
|---|---:|
| Access by index | O(n) |
| Search | O(n) |
| Insert at head | O(1) |
| Delete head | O(1) |
| Insert after known node | O(1) |
| Delete after known predecessor | O(1) |
| Insert at tail without tail | O(n) |
| Insert at tail with tail | O(1) |
| Traverse | O(n) |
| Reverse | O(n) |

Remember:

> A linked list does not provide O(1) random access.

---

# 29. Linked List vs Array

| Property | Array | Linked List |
|---|---|---|
| Random access | O(1) | O(n) |
| Sequential traversal | O(n) | O(n) |
| Insert at beginning | O(n) normally | O(1) |
| Delete known node/link | O(n) shifting normally | O(1) if predecessor/reference is available |
| Cache locality | Good | Usually worse |
| Extra pointer memory | No | Yes |
| Dynamic node structure | No | Yes |

In modern interview problems, do not claim linked lists are automatically faster for every insertion/deletion. The complexity depends on whether the relevant node/position is already known.

---

# 30. Fast/Slow Pointer Pattern Recognition

When you see:

```text
"middle"
"cycle"
"cycle start"
"nth from end"
"split list"
"palindrome"
```

consider:

```text
fast + slow
```

---

# 31. Reverse + Merge Pattern

Several hard problems are combinations of basic operations.

For example:

```text
Reorder List
```

is:

```text
middle
  +
reverse
  +
merge
```

Similarly:

```text
Palindrome
```

is:

```text
middle
  +
reverse
  +
compare
```

Learning the primitive operations makes these problems much easier.

---

# 32. Merge Pattern Recognition

When lists are sorted:

```text
merge rather than concatenate + sort
```

For two sorted lists:

```text
two pointers
```

For many sorted lists:

```text
divide and conquer
```

or:

```text
min-heap
```

---

# 33. Sorting a Linked List

Do not use an algorithm that assumes efficient random access.

Preferred:

```text
Merge Sort
```

because:

```text
split → recursively sort → merge
```

fits linked-list structure naturally.

---

# 34. Recursive vs Iterative

| Problem | Preferred/important approach |
|---|---|
| Basic traversal | Iterative |
| Reverse list | Iterative + recursive understanding |
| Merge lists | Iterative |
| Merge sort | Recursive understanding |
| Reverse in K groups | Iterative |
| Cycle detection | Iterative |
| Palindrome | Iterative pointer technique |
| Reorder | Iterative |
| Copy random pointer | HashMap first; O(1) technique as advanced |

The important point is not to avoid recursion. It is to understand its stack-space cost.

---

# 35. Advanced Pointer Technique: Previous Node

For many singly linked-list problems, maintain:

```text
prev
curr
next
```

The basic structure:

```java
while (curr != null) {
    // save information
    ListNode next = curr.next;

    // manipulate curr

    // advance
    prev = curr;
    curr = next;
}
```

This template appears repeatedly in:

- Reverse list
- Reverse groups
- List transformations
- Partitioning
- In-place manipulation

---

# 36. Advanced Pointer Technique: Dummy + Tail

For building a new linked structure:

```java
ListNode dummy = new ListNode(0);
ListNode tail = dummy;
```

Then:

```java
tail.next = node;
tail = tail.next;
```

At the end:

```java
return dummy.next;
```

This removes special handling for the first inserted node.

---

# 37. Advanced Pointer Technique: Split

To split a list:

```text
1 → 2 → 3 → 4 → 5
```

use fast/slow pointers to find the midpoint.

Then disconnect:

```java
slow.next = null;
```

Be careful about exactly which half you want.

The definition of "middle" can differ depending on the problem.

---

# 38. Advanced Pattern: In-Place Transformation

Many linked-list interview problems deliberately require:

```text
O(1) auxiliary space
```

That means you should modify links rather than creating another list.

Typical operations:

```text
reverse
split
merge
reconnect
```

The nodes themselves can be reused.

---

# 39. Common Mistakes

## Mistake 1: Losing the next node

Wrong:

```java
curr.next = prev;
curr = curr.next;
```

This moves `curr` backward rather than to the original successor.

Correct:

```java
ListNode next = curr.next;
curr.next = prev;
curr = next;
```

---

## Mistake 2: Incorrect head handling

Deleting or inserting at the head often breaks code.

Use a dummy node when appropriate.

---

## Mistake 3: Comparing values instead of nodes

Intersection requires:

```java
a == b
```

not:

```java
a.val == b.val
```

Cycle detection also compares references:

```java
slow == fast
```

not values.

---

## Mistake 4: Incorrect cycle loop condition

Use:

```java
while (fast != null && fast.next != null)
```

before accessing:

```java
fast.next.next
```

---

## Mistake 5: Off-by-one in remove nth from end

The exact gap between `fast` and `slow` matters.

Use a dummy node and verify with:

```text
n = 1
n = list length
```

---

## Mistake 6: Forgetting to disconnect reversed segments

For recursive reversal:

```java
head.next = null;
```

is important.

Otherwise an old link can create a cycle.

---

## Mistake 7: Forgetting the remaining nodes

After merging or reversing a portion, reconnect the untouched suffix.

---

## Mistake 8: Treating a linked list like an array

There is no efficient:

```text
list[i]
```

operation.

Repeatedly walking from the head can turn an apparently simple algorithm into `O(n²)`.

---

# 40. Interview Problem Templates

## Template 1 — Reverse

```java
ListNode prev = null;
ListNode curr = head;

while (curr != null) {
    ListNode next = curr.next;
    curr.next = prev;
    prev = curr;
    curr = next;
}

return prev;
```

---

## Template 2 — Fast/Slow

```java
ListNode slow = head;
ListNode fast = head;

while (fast != null && fast.next != null) {
    slow = slow.next;
    fast = fast.next.next;
}
```

---

## Template 3 — Dummy Node

```java
ListNode dummy = new ListNode(0);
dummy.next = head;
```

Then:

```java
return dummy.next;
```

---

## Template 4 — Merge

```java
ListNode dummy = new ListNode(0);
ListNode tail = dummy;

while (a != null && b != null) {
    if (a.val <= b.val) {
        tail.next = a;
        a = a.next;
    } else {
        tail.next = b;
        b = b.next;
    }

    tail = tail.next;
}

tail.next = (a != null) ? a : b;

return dummy.next;
```

---

## Template 5 — Cycle Detection

```java
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
```

---

# 41. Selected LeetCode Problems

This set focuses on high-value interview patterns rather than trying to list every linked-list problem.

## Easy

### 1. Reverse Linked List — LeetCode 206

**Pattern:** Pointer reversal

**Priority:** Essential

Master:

```text
prev / curr / next
```

---

### 2. Merge Two Sorted Lists — LeetCode 21

**Pattern:** Two-pointer merge + dummy node

**Priority:** Essential

---

### 3. Linked List Cycle — LeetCode 141

**Pattern:** Fast/slow pointer

**Priority:** Essential

---

### 4. Middle of the Linked List — LeetCode 876

**Pattern:** Fast/slow pointer

**Priority:** Essential

---

### 5. Intersection of Two Linked Lists — LeetCode 160

**Pattern:** Two-pointer head switching

**Priority:** Essential

---

### 6. Palindrome Linked List — LeetCode 234

**Pattern:** Middle + reverse + compare

**Priority:** Essential

---

## Medium

### 7. Remove Nth Node From End of List — LeetCode 19

**Pattern:** Dummy + two pointers

**Priority:** Essential

---

### 8. Add Two Numbers — LeetCode 2

**Pattern:** Simultaneous traversal + carry

**Priority:** Essential

---

### 9. Linked List Cycle II — LeetCode 142

**Pattern:** Floyd cycle entry

**Priority:** Essential

---

### 10. Reorder List — LeetCode 143

**Pattern:**

```text
middle + reverse + merge
```

**Priority:** Essential

---

### 11. Sort List — LeetCode 148

**Pattern:** Merge sort

**Priority:** Essential

---

### 12. Copy List with Random Pointer — LeetCode 138

**Pattern:** HashMap / in-place interleaving

**Priority:** Essential

---

### 13. Odd Even Linked List — LeetCode 328

**Pattern:** Pointer rearrangement

**Priority:** High

---

### 14. Partition List — LeetCode 86

**Pattern:** Two lists + dummy nodes

**Priority:** High

---

### 15. Rotate List — LeetCode 61

**Pattern:** Length + circular connection + break

**Priority:** High

---

## Hard

### 16. Reverse Nodes in k-Group — LeetCode 25

**Pattern:** Group reversal

**Priority:** Essential

---

### 17. Merge k Sorted Lists — LeetCode 23

**Pattern:** Heap / divide and conquer

**Priority:** Essential

---

# 42. LeetCode Progression

## Stage 1 — Fundamentals

Solve:

1. Reverse Linked List
2. Merge Two Sorted Lists
3. Linked List Cycle
4. Middle of the Linked List
5. Intersection of Two Linked Lists

Goal:

```text
Manipulate pointers without losing nodes.
```

---

## Stage 2 — Core Interview Problems

Solve:

1. Remove Nth Node From End
2. Add Two Numbers
3. Linked List Cycle II
4. Palindrome Linked List
5. Reorder List
6. Sort List
7. Copy List with Random Pointer
8. Odd Even Linked List
9. Partition List
10. Rotate List

Goal:

```text
Combine two or more linked-list techniques.
```

---

## Stage 3 — Hard

Solve:

1. Reverse Nodes in k-Group
2. Merge k Sorted Lists

Goal:

```text
Handle complicated pointer state while preserving O(n) or O(n log k) performance.
```

---

# 43. GATE / CS Exam Relevance

For GATE and theoretical CS preparation, focus on:

- Singly linked-list representation.
- Doubly linked-list representation.
- Circular linked lists.
- Pointer/reference manipulation.
- Insertion and deletion.
- Complexity of operations.
- Linked-list implementation of stacks.
- Linked-list implementation of queues.
- Polynomial representation using linked lists.
- Sparse representations.
- Traversal.
- Reversal.
- Memory/reference behavior.
- Edge cases involving head and tail.

For placement preparation, emphasize implementation.

For GATE, emphasize:

```text
representation
+
operation complexity
+
pointer behavior
```

---

# 44. GATE PYQ Practice Set

## PYQ 1 — Linked List Insertion/Deletion

**Question type:** Determine the effect of pointer assignments when inserting or deleting a node from a linked list.

### Method

For every assignment:

1. Identify the node on the left side.
2. Identify the reference being changed.
3. Draw the affected links.
4. Check whether any node becomes unreachable.
5. Continue the sequence exactly as written.

### Core principle

For:

```java
newNode.next = current.next;
current.next = newNode;
```

the old successor must be connected to the new node before changing `current.next`.

### Exam trap

Pointer assignment is directional.

Changing:

```text
A.next
```

does not automatically change:

```text
B.next
```

even if `B` is nearby in the diagram.

---

## PYQ 2 — Circular Linked List

**Question type:** Determine traversal behavior, number of nodes visited, or termination conditions for a circular linked list.

### Key idea

Unlike a normal singly linked list:

```text
last.next != null
```

Instead:

```text
last.next = head
```

Therefore:

```java
while (current != null)
```

does not terminate.

A correct traversal must detect a return to the starting node or use another explicitly defined stopping condition.

### Exam trap

Do not apply ordinary null-terminated-list assumptions to circular lists.

---

## PYQ 3 — Linked-List Complexity

**Question type:** Determine the time complexity of operations performed on singly/doubly linked lists.

### Key facts

For a singly linked list:

```text
Access kth node       → O(n)
Search                → O(n)
Insert at head        → O(1)
Delete head           → O(1)
Insert after known node → O(1)
Delete after known predecessor → O(1)
```

For a doubly linked list, if a node reference is already available, deletion can be performed in `O(1)` because both neighbors are directly accessible.

### Exam trap

Do not confuse:

```text
known node/reference
```

with:

```text
known position by index
```

Finding a node by index can still require `O(n)` traversal.

---

# 45. Advanced Theory: Why Linked Lists Have O(1) Local Updates

Suppose:

```text
A → B → C
```

To insert `X` between `A` and `B`:

```text
A → X → B → C
```

Only local references need to change:

```java
X.next = A.next;
A.next = X;
```

No shifting of all later elements is required.

This is the key structural advantage over array insertion in the middle.

---

# 46. Advanced Theory: Why Random Access Is O(n)

To reach the kth node:

```text
head
 ↓
1 → 2 → 3 → ... → k
```

you must follow references one by one.

Therefore:

```text
T(k) = O(k)
```

and worst-case:

```text
O(n)
```

There is no direct address calculation like an array index.

---

# 47. Advanced Problem Decomposition

When a linked-list problem looks difficult, break it into primitive operations.

Ask:

```text
Do I need to:
    find middle?
    reverse?
    split?
    merge?
    reconnect?
    detect a cycle?
    maintain a gap?
```

Many hard problems are combinations of these primitives.

Examples:

```text
Palindrome
= middle + reverse + compare

Reorder List
= middle + reverse + merge

Reverse k-Group
= locate group + reverse + reconnect

Merge k Lists
= merge + heap/divide-and-conquer
```

---

# 48. Mastery Checklist

Before marking **Linked Lists** complete, you should be able to:

- [ ] Define a singly linked list.
- [ ] Define a doubly linked list.
- [ ] Define a circular linked list.
- [ ] Implement a Java `ListNode`.
- [ ] Traverse a linked list.
- [ ] Insert at the head.
- [ ] Insert after a known node.
- [ ] Insert at the tail.
- [ ] Delete the head.
- [ ] Delete a node using its predecessor.
- [ ] Explain when a dummy node helps.
- [ ] Reverse a linked list iteratively.
- [ ] Reverse a linked list recursively.
- [ ] Reverse nodes in groups of `k`.
- [ ] Find the middle using fast/slow pointers.
- [ ] Detect a cycle using Floyd's algorithm.
- [ ] Find the cycle entry.
- [ ] Merge two sorted linked lists.
- [ ] Merge `k` sorted lists.
- [ ] Sort a linked list with merge sort.
- [ ] Remove the nth node from the end.
- [ ] Find the intersection of two linked lists.
- [ ] Distinguish node identity from equal values.
- [ ] Check whether a linked list is a palindrome.
- [ ] Reorder a linked list.
- [ ] Copy a list with random pointers using a HashMap.
- [ ] Understand the O(1)-auxiliary-space random-pointer solution.
- [ ] Explain pointer invariants.
- [ ] Handle empty and one-node lists.
- [ ] Handle head/tail edge cases.
- [ ] State linked-list complexities correctly.
- [ ] Solve Easy linked-list problems quickly.
- [ ] Solve Medium linked-list problems without looking at solutions.
- [ ] Attempt Hard linked-list problems using decomposition.
- [ ] Practice GATE linked-list representation and complexity questions.

---

# 49. Final Linked-List Cheat Sheet

| Problem signal | First technique to consider |
|---|---|
| Reverse list | `prev / curr / next` |
| Find middle | Fast/slow |
| Detect cycle | Fast/slow |
| Cycle entry | Floyd + reset one pointer |
| Remove nth from end | Dummy + two pointers |
| Merge sorted lists | Two pointers |
| Sort list | Merge sort |
| Intersection | Head-switching pointers |
| Palindrome | Middle + reverse + compare |
| Reorder | Middle + reverse + merge |
| Reverse k nodes | Group reversal |
| Copy random pointer | HashMap / interleaving |
| Head edge cases | Dummy node |
| Build output list | Dummy + tail |
| Circular traversal | Stop when returning to start |

---

# 50. Final Interview Rule

When you see a linked-list problem, do not immediately start writing code.

First draw:

```text
prev → curr → next
```

or:

```text
slow → 
fast → →
```

Then identify the structural operation:

```text
REVERSE
SPLIT
MERGE
DELETE
INSERT
CONNECT
```

For every pointer update, verify:

```text
1. What link am I changing?
2. What node will become unreachable if I change it now?
3. Have I saved the required reference?
4. Where should the modified node point afterward?
5. Have I handled head/null/one-node cases?
```

The core linked-list skill is not memorizing solutions. It is maintaining correct **references and invariants** while changing the structure in-place.
