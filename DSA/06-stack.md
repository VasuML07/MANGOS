# 6. Stack

## 1. What You Must Master

This topic is a high-frequency DSA interview topic because stacks appear directly in parsing, expression evaluation, monotonic structures, range queries, and simulation.

### Core topics
- Stack implementation
- Monotonic stack
- Parentheses problems
- Expression problems
- Next greater element
- Next smaller element
- Previous greater/smaller
- Histogram problems
- Stock span
- Temperature problems
- Stack + HashMap
- Stack simulation

### Interview priority

| Topic | Priority | Typical difficulty |
|---|---|---|
| Stack basics | High | Easy |
| Parentheses | Very High | Easy–Medium |
| Next Greater Element | Very High | Medium |
| Monotonic stack | Very High | Medium |
| Previous/Next smaller/greater | Very High | Medium |
| Stock span | High | Medium |
| Daily Temperatures | Very High | Medium |
| Largest Rectangle in Histogram | Very High | Hard/Medium |
| Expression evaluation | High | Medium |
| Stack + HashMap | High | Medium |
| Simulation | Medium–High | Easy–Medium |

---

# 2. Stack Fundamentals

## 2.1 What is a Stack?

A stack is a linear data structure following:

> **LIFO — Last In, First Out**

The last inserted element is the first removed.

Example:

```text
push(10)
push(20)
push(30)

Top
 ↓
30
20
10
```

`pop()` removes `30`.

## 2.2 Core operations

| Operation | Meaning | Typical time |
|---|---|---:|
| `push(x)` | Insert at top | O(1) |
| `pop()` | Remove top | O(1) |
| `peek()` | Read top | O(1) |
| `isEmpty()` | Check empty | O(1) |
| `size()` | Number of elements | O(1) |

---

# 3. Stack Implementation in Java

## 3.1 Preferred Java choices

For interviews, prefer:

```java
Deque<Integer> stack = new ArrayDeque<>();
```

Use:

```java
stack.push(x);
stack.pop();
stack.peek();
stack.isEmpty();
```

Avoid using the legacy `Stack` class unless the interviewer specifically asks for it.

## 3.2 Basic template

```java
Deque<Integer> stack = new ArrayDeque<>();

stack.push(10);
stack.push(20);
stack.push(30);

System.out.println(stack.peek()); // 30

System.out.println(stack.pop());  // 30
System.out.println(stack.pop());  // 20

System.out.println(stack.isEmpty()); // false
```

`ArrayDeque` does not permit `null`.

---

# 4. Implement a Stack Using an Array

```java
class MyStack {
    private int[] arr;
    private int top;

    MyStack(int capacity) {
        arr = new int[capacity];
        top = -1;
    }

    void push(int x) {
        if (top == arr.length - 1) {
            throw new IllegalStateException("Stack overflow");
        }
        arr[++top] = x;
    }

    int pop() {
        if (top == -1) {
            throw new IllegalStateException("Stack underflow");
        }
        return arr[top--];
    }

    int peek() {
        if (top == -1) {
            throw new IllegalStateException("Stack is empty");
        }
        return arr[top];
    }

    boolean isEmpty() {
        return top == -1;
    }
}
```

### Invariant

```text
top = index of current top element
top = -1  → empty
```

---

# 5. Implement a Stack Using a Linked List

The head acts as the stack top.

```java
class Node {
    int val;
    Node next;

    Node(int val) {
        this.val = val;
    }
}

class LinkedStack {
    private Node head;

    void push(int x) {
        Node node = new Node(x);
        node.next = head;
        head = node;
    }

    int pop() {
        if (head == null) {
            throw new IllegalStateException("Stack is empty");
        }

        int value = head.val;
        head = head.next;
        return value;
    }

    int peek() {
        if (head == null) {
            throw new IllegalStateException("Stack is empty");
        }

        return head.val;
    }

    boolean isEmpty() {
        return head == null;
    }
}
```

---

# 6. Stack Applications

A stack is useful when the problem contains:

- Nested structures
- Matching pairs
- Undo/backtracking
- "Most recent unresolved item"
- Expression evaluation
- Previous/next greater or smaller
- Removing elements while maintaining an order
- One-pass range information
- Browser/history-like behavior
- Simulation of nested operations

A strong signal is:

> "For each element, find the first element to its left/right satisfying a comparison."

This frequently suggests a **monotonic stack**.

---

# 7. Parentheses Problems

## 7.1 Valid Parentheses

Given:

```text
"()[]{}"
```

return `true`.

Given:

```text
"(]"
```

return `false`.

### Idea

When opening bracket appears:

```text
push it
```

When closing bracket appears:

```text
top must be the matching opening bracket
```

### Java

```java
class Solution {
    public boolean isValid(String s) {
        Deque<Character> stack = new ArrayDeque<>();

        for (char c : s.toCharArray()) {
            if (c == '(' || c == '[' || c == '{') {
                stack.push(c);
            } else {
                if (stack.isEmpty()) {
                    return false;
                }

                char open = stack.pop();

                if ((c == ')' && open != '(') ||
                    (c == ']' && open != '[') ||
                    (c == '}' && open != '{')) {
                    return false;
                }
            }
        }

        return stack.isEmpty();
    }
}
```

### Complexity

- Time: `O(n)`
- Space: `O(n)`

---

# 8. Parentheses: Minimum Additions

Example:

```text
"())"
```

Need one additional `(`.

Track unmatched opening brackets.

```java
class Solution {
    public int minAddToMakeValid(String s) {
        int open = 0;
        int additions = 0;

        for (char c : s.toCharArray()) {
            if (c == '(') {
                open++;
            } else {
                if (open > 0) {
                    open--;
                } else {
                    additions++;
                }
            }
        }

        return additions + open;
    }
}
```

This problem does not require an actual stack because only the count matters.

### Important interview lesson

Do not use a stack merely because the topic is "stack."

Ask:

> Do I need the actual previous elements, or only their count/state?

---

# 9. Remove Outermost Parentheses

For primitive parentheses groups, track nesting depth.

```java
class Solution {
    public String removeOuterParentheses(String s) {
        StringBuilder ans = new StringBuilder();
        int depth = 0;

        for (char c : s.toCharArray()) {
            if (c == '(') {
                if (depth > 0) {
                    ans.append(c);
                }
                depth++;
            } else {
                depth--;
                if (depth > 0) {
                    ans.append(c);
                }
            }
        }

        return ans.toString();
    }
}
```

Time: `O(n)`  
Space: `O(n)` for output.

---

# 10. Longest Valid Parentheses

Example:

```text
")()())"
```

Answer:

```text
4
```

The valid substring is:

```text
()()
```

## Stack approach

Store indices.

Initialize:

```text
stack = [-1]
```

When seeing `(`:

```text
push(index)
```

When seeing `)`:

```text
pop()
```

If empty after popping:

```text
push(current index)
```

Otherwise:

```text
length = currentIndex - stack.peek()
```

### Java

```java
class Solution {
    public int longestValidParentheses(String s) {
        Deque<Integer> stack = new ArrayDeque<>();
        stack.push(-1);

        int best = 0;

        for (int i = 0; i < s.length(); i++) {
            if (s.charAt(i) == '(') {
                stack.push(i);
            } else {
                stack.pop();

                if (stack.isEmpty()) {
                    stack.push(i);
                } else {
                    best = Math.max(best, i - stack.peek());
                }
            }
        }

        return best;
    }
}
```

### Why `-1`?

It acts as a sentinel boundary before the string.

---

# 11. Expression Problems

Expression problems typically involve:

- Operators
- Operands
- Precedence
- Parentheses
- Unary/binary operators
- Conversion between expression forms

Common forms:

```text
Infix:    A + B
Prefix:   + A B
Postfix:  A B +
```

---

# 12. Operator Precedence

Typical arithmetic precedence:

```text
()
↑
* /
↑
+ -
```

Example:

```text
2 + 3 * 4
```

is:

```text
2 + (3 * 4)
```

not:

```text
(2 + 3) * 4
```

---

# 13. Evaluate Postfix Expression

Example:

```text
"2 3 + 4 *"
```

Evaluation:

```text
2 3 + → 5
5 4 * → 20
```

## Algorithm

For every token:

- Number → push
- Operator → pop `b`, pop `a`, calculate `a op b`, push result

### Java

```java
class Solution {
    public int evalPostfix(String[] tokens) {
        Deque<Integer> stack = new ArrayDeque<>();

        for (String token : tokens) {
            if (token.equals("+") ||
                token.equals("-") ||
                token.equals("*") ||
                token.equals("/")) {

                int b = stack.pop();
                int a = stack.pop();

                int result;

                switch (token) {
                    case "+" -> result = a + b;
                    case "-" -> result = a - b;
                    case "*" -> result = a * b;
                    default -> result = a / b;
                }

                stack.push(result);
            } else {
                stack.push(Integer.parseInt(token));
            }
        }

        return stack.pop();
    }
}
```

### Critical point

For subtraction/division:

```text
a = second popped
b = first popped
```

Do NOT reverse them.

---

# 14. Evaluate Reverse Polish Notation

This is essentially postfix evaluation.

Example:

```text
["2", "1", "+", "3", "*"]
```

Process:

```text
2
2 1
3
3 3
9
```

Answer:

```text
9
```

Pattern:

> Operand → push  
> Operator → pop two, calculate, push result

---

# 15. Infix Evaluation

A standard two-stack approach uses:

1. Operand stack
2. Operator stack

For:

```text
2 + 3 * 4
```

`*` must be evaluated before `+`.

The key operation is:

```text
while top operator has >= precedence:
    evaluate it
```

### Conceptual template

```java
while (!operators.isEmpty()
        && precedence(operators.peek()) >= precedence(current)) {
    applyTopOperator();
}

operators.push(current);
```

Parentheses override precedence.

---

# 16. Monotonic Stack

This is one of the most important stack patterns for interviews.

A monotonic stack maintains elements in either:

- Increasing order
- Decreasing order

depending on the problem.

## Why?

It efficiently finds:

- Next greater
- Next smaller
- Previous greater
- Previous smaller
- Nearest boundary
- Histogram widths
- Stock span
- Temperature waits

---

# 17. The Core Monotonic Stack Rule

Suppose we want the **next greater element**.

For current value `x`:

```text
while stack is not empty AND stack.top <= x:
    pop
```

After this:

- Anything popped has found its next greater element.
- The remaining top is a candidate for the current element.

Then:

```text
push x
```

This gives amortized `O(n)`.

---

# 18. Why Is It O(n)?

At first glance:

```java
while (...) stack.pop();
```

looks like `O(n²)`.

But every element:

- enters the stack once
- leaves the stack at most once

Therefore total stack operations are `O(n)`.

This is called **amortized analysis**.

---

# 19. Next Greater Element

For each element, find the first greater element to its right.

Example:

```text
nums = [2, 1, 2, 4, 3]

answer = [4, 2, 4, -1, -1]
```

## Right-to-left approach

For `nums[i]`:

```text
while stack not empty and stack.top <= nums[i]:
    pop

if stack empty:
    answer[i] = -1
else:
    answer[i] = stack.top

push nums[i]
```

### Java

```java
class Solution {
    public int[] nextGreaterElements(int[] nums) {
        int n = nums.length;
        int[] ans = new int[n];

        Deque<Integer> stack = new ArrayDeque<>();

        for (int i = n - 1; i >= 0; i--) {
            while (!stack.isEmpty() && stack.peek() <= nums[i]) {
                stack.pop();
            }

            ans[i] = stack.isEmpty() ? -1 : stack.peek();
            stack.push(nums[i]);
        }

        return ans;
    }
}
```

Time: `O(n)`  
Space: `O(n)`

---

# 20. Next Greater Element: Index Stack

Sometimes storing values is not enough.

If you need:

- indices
- distances
- positions

store indices.

Example:

```java
while (!stack.isEmpty() && nums[stack.peek()] <= nums[i]) {
    stack.pop();
}
```

Then:

```java
ans[i] = stack.isEmpty() ? -1 : nums[stack.peek()];
```

### Rule

Use a **value stack** when you only need values.

Use an **index stack** when you need:

- Distance
- Width
- Position
- Later access to the array

---

# 21. Next Smaller Element

For each element, find the first smaller element to its right.

Replace:

```text
<=
```

with the comparison appropriate to your duplicate policy.

Basic strict version:

```java
while (!stack.isEmpty() && nums[stack.peek()] >= nums[i]) {
    stack.pop();
}
```

Then:

```java
ans[i] = stack.isEmpty() ? -1 : nums[stack.peek()];
```

---

# 22. Previous Greater Element

Scan left to right.

For each `i`:

```java
while (!stack.isEmpty() && nums[stack.peek()] <= nums[i]) {
    stack.pop();
}
```

Remaining top is previous greater.

Then:

```java
stack.push(i);
```

---

# 23. Previous Smaller Element

Again scan left to right.

```java
while (!stack.isEmpty() && nums[stack.peek()] >= nums[i]) {
    stack.pop();
}
```

Remaining top is previous smaller.

---

# 24. Monotonic Stack Decision Table

| Goal | Direction | Pop condition |
|---|---|---|
| Next greater | Right → left | `<= current` |
| Next smaller | Right → left | `>= current` |
| Previous greater | Left → right | `<= current` |
| Previous smaller | Left → right | `>= current` |

For duplicates, carefully decide whether equality should be popped.

---

# 25. Daily Temperatures

Given temperatures, find how many days until a warmer temperature.

Example:

```text
[73,74,75,71,69,72,76,73]

[1,1,4,2,1,1,0,0]
```

This is a classic monotonic decreasing stack of indices.

### Algorithm

Scan left → right.

For current day `i`:

```java
while (!stack.isEmpty()
       && temperatures[i] > temperatures[stack.peek()]) {

    int previous = stack.pop();
    ans[previous] = i - previous;
}

stack.push(i);
```

### Java

```java
class Solution {
    public int[] dailyTemperatures(int[] temperatures) {
        int n = temperatures.length;
        int[] ans = new int[n];

        Deque<Integer> stack = new ArrayDeque<>();

        for (int i = 0; i < n; i++) {
            while (!stack.isEmpty()
                    && temperatures[i] > temperatures[stack.peek()]) {

                int j = stack.pop();
                ans[j] = i - j;
            }

            stack.push(i);
        }

        return ans;
    }
}
```

Time: `O(n)`  
Space: `O(n)`

---

# 26. Stock Span

For each day, find the number of consecutive previous days with price less than or equal to today's price.

Example:

```text
[100,80,60,70,60,75,85]

[1,1,1,2,1,4,6]
```

Use a decreasing stack of indices.

For current `i`:

```text
while price[stack.top] <= price[i]:
    pop
```

Then:

```text
if empty:
    span = i + 1
else:
    span = i - stack.top
```

### Java

```java
class StockSpanner {
    private Deque<int[]> stack = new ArrayDeque<>();

    public int next(int price) {
        int span = 1;

        while (!stack.isEmpty() && stack.peek()[0] <= price) {
            span += stack.pop()[1];
        }

        stack.push(new int[]{price, span});

        return span;
    }
}
```

This version stores:

```text
price + accumulated span
```

and is `O(1)` amortized per call.

---

# 27. Largest Rectangle in Histogram

This is one of the most important monotonic-stack problems.

Example:

```text
heights = [2,1,5,6,2,3]
```

Answer:

```text
10
```

The rectangle:

```text
5 × 2 = 10
```

uses heights `5` and `6`.

---

# 28. Histogram Insight

For every bar, determine:

```text
nearest smaller element on left
nearest smaller element on right
```

Then:

```text
width = rightSmallerIndex - leftSmallerIndex - 1
```

Area:

```text
height[i] * width
```

Doing this explicitly with separate arrays works.

But we can compute everything in one pass using a monotonic stack.

---

# 29. Histogram One-Pass Algorithm

Maintain increasing heights.

When current height is smaller than stack top:

```text
the current index is the right boundary
```

The popped bar determines its maximum width.

For popped index:

```text
height = heights[mid]
right = i
left = stack.peek() after popping

width = right - left - 1
```

### Java

```java
class Solution {
    public int largestRectangleArea(int[] heights) {
        Deque<Integer> stack = new ArrayDeque<>();
        int best = 0;

        for (int i = 0; i <= heights.length; i++) {
            int current = (i == heights.length) ? 0 : heights[i];

            while (!stack.isEmpty() && heights[stack.peek()] > current) {
                int mid = stack.pop();

                int left = stack.isEmpty() ? -1 : stack.peek();
                int width = i - left - 1;

                best = Math.max(best, heights[mid] * width);
            }

            stack.push(i);
        }

        return best;
    }
}
```

There is a subtle indexing issue with the sentinel iteration because `i == heights.length` is pushed after processing. A safer interview implementation is to use a sentinel array or handle the final cleanup separately.

### Safer implementation

```java
class Solution {
    public int largestRectangleArea(int[] heights) {
        Deque<Integer> stack = new ArrayDeque<>();
        int best = 0;

        for (int i = 0; i < heights.length; i++) {
            while (!stack.isEmpty()
                    && heights[stack.peek()] > heights[i]) {

                int h = heights[stack.pop()];
                int left = stack.isEmpty() ? -1 : stack.peek();
                int width = i - left - 1;

                best = Math.max(best, h * width);
            }

            stack.push(i);
        }

        int n = heights.length;

        while (!stack.isEmpty()) {
            int h = heights[stack.pop()];
            int left = stack.isEmpty() ? -1 : stack.peek();
            int width = n - left - 1;

            best = Math.max(best, h * width);
        }

        return best;
    }
}
```

Time: `O(n)`  
Space: `O(n)`

---

# 30. Histogram Duplicate Heights

A common implementation choice is whether to pop on:

```java
heights[stack.peek()] > current
```

or:

```java
heights[stack.peek()] >= current
```

Both can work if the invariant and width calculation are consistent.

For interviews, explain your duplicate policy.

A robust rule:

> Decide whether equal heights should be merged before writing the loop.

---

# 31. Maximal Rectangle

A 2D matrix problem can be converted into repeated histogram problems.

For every row:

```text
height[j] =
    0                  if matrix[row][j] == '0'
    height[j] + 1      if matrix[row][j] == '1'
```

Then:

```text
largest rectangle in histogram
```

on each row.

Complexity:

```text
O(rows × cols)
```

This is a major example of **reducing a 2D problem to a 1D monotonic-stack problem**.

---

# 32. Stack + HashMap

Some problems need:

```text
stack + HashMap
```

A common pattern is:

1. Build frequency/count information with a HashMap.
2. Use a stack to maintain unresolved elements.

---

# 33. Remove Duplicate Letters

Goal:

Return the smallest lexicographical string containing every distinct character exactly once.

Example:

```text
bcabc
```

Answer:

```text
abc
```

Useful state:

```text
frequency[character]
inStack[character]
stack
```

For current character `c`:

```text
decrement frequency[c]
```

If already in stack:

```text
continue
```

Otherwise:

```text
while stack not empty
      and top > c
      and frequency[top] > 0:

    pop top
```

Then push `c`.

This is a powerful example of:

> Monotonic decision + future availability information.

---

# 34. Frequency + Stack Template

```java
int[] freq = new int[26];
boolean[] inStack = new boolean[26];

for (char c : s.toCharArray()) {
    freq[c - 'a']++;
}

Deque<Character> stack = new ArrayDeque<>();

for (char c : s.toCharArray()) {
    int idx = c - 'a';
    freq[idx]--;

    if (inStack[idx]) {
        continue;
    }

    while (!stack.isEmpty()
            && stack.peek() > c
            && freq[stack.peek() - 'a'] > 0) {

        char removed = stack.pop();
        inStack[removed - 'a'] = false;
    }

    stack.push(c);
    inStack[idx] = true;
}
```

---

# 35. Stack Simulation

Stacks are frequently used to simulate processes.

Examples:

- Backspace operations
- Browser navigation
- File paths
- Collision systems
- Undo operations
- Removing adjacent elements
- Nested decoding
- Simplifying expressions

---

# 36. Backspace String Compare

Example:

```text
ab#c
ad#c
```

Both become:

```text
ac
```

A stack can simulate `#` as backspace.

### Java

```java
class Solution {
    private String build(String s) {
        StringBuilder stack = new StringBuilder();

        for (char c : s.toCharArray()) {
            if (c == '#') {
                if (stack.length() > 0) {
                    stack.deleteCharAt(stack.length() - 1);
                }
            } else {
                stack.append(c);
            }
        }

        return stack.toString();
    }

    public boolean backspaceCompare(String s, String t) {
        return build(s).equals(build(t));
    }
}
```

---

# 37. Remove All Adjacent Duplicates

Example:

```text
abbaca
```

Process:

```text
a
ab
abb → remove bb
a
ac
```

Answer:

```text
ca
```

### Java

```java
class Solution {
    public String removeDuplicates(String s) {
        StringBuilder stack = new StringBuilder();

        for (char c : s.toCharArray()) {
            if (stack.length() > 0
                    && stack.charAt(stack.length() - 1) == c) {
                stack.deleteCharAt(stack.length() - 1);
            } else {
                stack.append(c);
            }
        }

        return stack.toString();
    }
}
```

Time: `O(n)` amortized.

---

# 38. Decode String

Example:

```text
3[a2[c]]
```

Answer:

```text
accaccacc
```

Need to preserve:

- Previous string
- Repeat count

Use two stacks:

```text
countStack
stringStack
```

When `[` occurs:

```text
push current state
```

When `]` occurs:

```text
restore previous state
```

### Java

```java
class Solution {
    public String decodeString(String s) {
        Deque<Integer> counts = new ArrayDeque<>();
        Deque<StringBuilder> strings = new ArrayDeque<>();

        StringBuilder current = new StringBuilder();
        int number = 0;

        for (char c : s.toCharArray()) {
            if (Character.isDigit(c)) {
                number = number * 10 + (c - '0');
            } else if (c == '[') {
                counts.push(number);
                strings.push(current);

                number = 0;
                current = new StringBuilder();
            } else if (c == ']') {
                int times = counts.pop();
                StringBuilder previous = strings.pop();

                for (int k = 0; k < times; k++) {
                    previous.append(current);
                }

                current = previous;
            } else {
                current.append(c);
            }
        }

        return current.toString();
    }
}
```

---

# 39. Asteroid Collision

Example:

```text
[5,10,-5]
```

Result:

```text
[5,10]
```

Positive asteroid moves right.

Negative asteroid moves left.

A collision can happen when:

```text
stack.top > 0
current < 0
```

Resolve repeatedly.

### Java

```java
class Solution {
    public int[] asteroidCollision(int[] asteroids) {
        Deque<Integer> stack = new ArrayDeque<>();

        for (int x : asteroids) {
            boolean alive = true;

            while (alive && x < 0
                    && !stack.isEmpty()
                    && stack.peek() > 0) {

                int top = stack.peek();

                if (top < -x) {
                    stack.pop();
                } else if (top == -x) {
                    stack.pop();
                    alive = false;
                } else {
                    alive = false;
                }
            }

            if (alive) {
                stack.push(x);
            }
        }

        int[] ans = new int[stack.size()];

        for (int i = ans.length - 1; i >= 0; i--) {
            ans[i] = stack.pop();
        }

        return ans;
    }
}
```

---

# 40. Make the String Great

Repeatedly remove adjacent pairs where the same letter has opposite case.

Example:

```text
leEeetcode
```

Answer:

```text
leetcode
```

Stack simulation solves this in `O(n)`.

The general pattern is:

```text
if current conflicts with stack.top:
    pop
else:
    push
```

---

# 41. Stack Simulation: Key Pattern

When the problem says:

- "remove adjacent..."
- "undo..."
- "cancel..."
- "collide..."
- "match..."
- "process nested..."
- "most recent operation..."

Try a stack.

---

# 42. Monotonic Stack: The Master Pattern

## 42.1 Generic next-greater template

```java
Deque<Integer> stack = new ArrayDeque<>();

for (int i = n - 1; i >= 0; i--) {

    while (!stack.isEmpty()
            && nums[stack.peek()] <= nums[i]) {
        stack.pop();
    }

    // stack.peek() is next greater candidate

    stack.push(i);
}
```

## 42.2 Generic previous-greater template

```java
Deque<Integer> stack = new ArrayDeque<>();

for (int i = 0; i < n; i++) {

    while (!stack.isEmpty()
            && nums[stack.peek()] <= nums[i]) {
        stack.pop();
    }

    // stack.peek() is previous greater candidate

    stack.push(i);
}
```

---

# 43. How to Derive the Correct Pop Condition

Do not memorize four unrelated algorithms.

Ask:

### What do I want?

If I want the next greater element:

```text
smaller/equal elements cannot be the answer
```

Therefore remove:

```text
<= current
```

If I want next smaller:

```text
greater/equal elements cannot be the answer
```

Therefore remove:

```text
>= current
```

This reasoning is more reliable than memorization.

---

# 44. Strict vs Non-Strict Comparisons

The difference between:

```text
<
```

and:

```text
<=
```

matters when duplicates exist.

Example:

```text
[2, 2, 3]
```

Before coding, decide:

- Does "greater" mean strictly greater?
- Does "smaller" mean strictly smaller?
- Can equal values remain in the stack?
- Should equal values be merged?

Many monotonic-stack bugs are caused by inconsistent equality handling.

---

# 45. Sentinel Technique

A sentinel can force remaining stack elements to be processed.

For increasing stack:

```text
append -∞
```

For decreasing stack:

```text
append +∞
```

Histogram commonly uses:

```text
current = 0
```

after all real heights.

This eliminates a separate cleanup loop, although a cleanup loop can sometimes be clearer in interviews.

---

# 46. Stack of Indices vs Stack of Values

## Use values when:

You only need:

```text
next greater VALUE
```

## Use indices when:

You need:

```text
distance
width
position
original array access
```

Examples:

| Problem | Stack stores |
|---|---|
| Next greater value | Values or indices |
| Daily Temperatures | Indices |
| Stock Span | Indices or `(price, span)` |
| Histogram | Indices |
| Maximal Rectangle | Indices |
| Parentheses matching | Characters |
| Expression evaluation | Numbers |
| Decode String | State |

---

# 47. Common Stack Complexity

| Pattern | Time | Space |
|---|---:|---:|
| Push/pop/peek | O(1) | O(1) each |
| Valid parentheses | O(n) | O(n) |
| Postfix evaluation | O(n) | O(n) |
| Next greater | O(n) | O(n) |
| Daily temperatures | O(n) | O(n) |
| Stock span | O(n) total | O(n) |
| Histogram | O(n) | O(n) |
| Stack simulation | Usually O(n) | O(n) |

---

# 48. Why Monotonic Stack Beats Brute Force

Suppose:

```text
[2, 1, 5, 3, 4]
```

Brute force next greater:

```text
For every i:
    scan right until greater
```

Worst case:

```text
O(n²)
```

Monotonic stack:

```text
Each element pushed once
Each element popped once
```

Therefore:

```text
O(n)
```

---

# 49. Common Mistakes

## Mistake 1: Using `Stack<Integer>`

Prefer:

```java
Deque<Integer> stack = new ArrayDeque<>();
```

## Mistake 2: Wrong pop condition

For next greater:

```java
while (top <= current)
```

For next smaller:

```java
while (top >= current)
```

assuming strict greater/smaller semantics.

## Mistake 3: Forgetting index storage

If the answer is a distance:

```text
i - previousIndex
```

you need indices.

## Mistake 4: Wrong operand order

For:

```text
a - b
```

if popping:

```text
b = stack.pop()
a = stack.pop()
```

then calculate:

```text
a - b
```

## Mistake 5: Forgetting remaining stack elements

Histogram and similar problems often require:

```text
final cleanup
```

or a sentinel.

## Mistake 6: Ignoring duplicates

Always decide whether equality should remain or be removed.

## Mistake 7: Claiming nested while loops are O(n²)

For a monotonic stack, explain amortized analysis.

---

# 50. Interview Decision Framework

When you see a problem, ask these questions in order.

### Step 1 — Is there nesting?

Examples:

```text
()
[]
{}
3[a2[c]]
```

Think:

```text
Stack
```

### Step 2 — Is there an unresolved previous item?

Examples:

```text
next greater
next smaller
daily temperature
stock span
```

Think:

```text
Monotonic stack
```

### Step 3 — Do I need nearest boundaries?

Examples:

```text
nearest smaller left/right
histogram
```

Think:

```text
Index-based monotonic stack
```

### Step 4 — Is there cancellation/removal?

Examples:

```text
adjacent duplicates
asteroids
backspace
undo
```

Think:

```text
Stack simulation
```

### Step 5 — Is there frequency/future availability?

Example:

```text
remove duplicate letters
```

Think:

```text
HashMap/frequency + stack
```

---

# 51. LeetCode Practice Roadmap

The following progression is designed for FAANG/top-product-company interviews.

## Easy

### 1. Valid Parentheses
- Difficulty: Easy
- Pattern: Stack
- Priority: Essential
- Core lesson: Matching nested delimiters

### 2. Baseball Game
- Difficulty: Easy
- Pattern: Stack simulation
- Priority: High
- Core lesson: Undo/recent-state simulation

### 3. Backspace String Compare
- Difficulty: Easy
- Pattern: Stack simulation
- Priority: High
- Core lesson: Process operations against previous state

### 4. Remove All Adjacent Duplicates in String
- Difficulty: Easy
- Pattern: Stack simulation
- Priority: Essential
- Core lesson: Cancel with stack top

### 5. Next Greater Element I
- Difficulty: Easy
- Pattern: Monotonic stack + HashMap
- Priority: Essential
- Core lesson: Map values to next greater values

---

# 52. Medium

### 6. Evaluate Reverse Polish Notation
- Difficulty: Medium
- Pattern: Expression stack
- Priority: Essential
- Core lesson: Operand/operator processing

### 7. Daily Temperatures
- Difficulty: Medium
- Pattern: Monotonic decreasing stack
- Priority: Essential
- Core lesson: Next greater distance

### 8. Online Stock Span
- Difficulty: Medium
- Pattern: Monotonic stack
- Priority: Essential
- Core lesson: Previous greater boundary

### 9. Next Greater Element II
- Difficulty: Medium
- Pattern: Circular monotonic stack
- Priority: Very High
- Core lesson: Simulate circular traversal using `2*n`

### 10. Next Greater Element III
- Difficulty: Medium
- Pattern: Monotonic reasoning + next permutation
- Priority: Medium
- Core lesson: Next greater arrangement

### 11. Remove K Digits
- Difficulty: Medium
- Pattern: Monotonic stack
- Priority: Very High
- Core lesson: Greedy deletion

### 12. Decode String
- Difficulty: Medium
- Pattern: Two stacks / nested simulation
- Priority: High
- Core lesson: Save and restore nested state

### 13. Asteroid Collision
- Difficulty: Medium
- Pattern: Stack simulation
- Priority: High
- Core lesson: Repeated local collision resolution

### 14. Simplify Path
- Difficulty: Medium
- Pattern: Stack
- Priority: High
- Core lesson: Path cancellation

### 15. Min Stack
- Difficulty: Medium
- Pattern: Stack + auxiliary state
- Priority: Essential
- Core lesson: Maintain current minimum

### 16. Evaluate Reverse Polish Notation
- Difficulty: Medium
- Pattern: Stack
- Priority: Essential
- Core lesson: Postfix evaluation

### 17. Minimum Add to Make Parentheses Valid
- Difficulty: Medium
- Pattern: Balance counting
- Priority: High
- Core lesson: Stack can sometimes be reduced to counters

### 18. Remove Duplicate Letters
- Difficulty: Medium
- Pattern: Stack + frequency
- Priority: Very High
- Core lesson: Greedy monotonic stack with future availability

### 19. Car Fleet
- Difficulty: Medium
- Pattern: Monotonic stack / ordering
- Priority: High
- Core lesson: Process events by sorted position

---

# 53. Hard / Advanced

### 20. Largest Rectangle in Histogram
- Difficulty: Hard
- Pattern: Monotonic stack
- Priority: Essential
- Core lesson: Nearest smaller boundaries

### 21. Maximal Rectangle
- Difficulty: Hard
- Pattern: Histogram + monotonic stack
- Priority: Very High
- Core lesson: Reduce 2D to repeated 1D problems

### 22. Trapping Rain Water
- Difficulty: Hard
- Pattern: Monotonic stack / two pointers
- Priority: Very High
- Core lesson: Boundary-based reasoning

### 23. Basic Calculator
- Difficulty: Hard
- Pattern: Stack + expression parsing
- Priority: High
- Core lesson: Nested arithmetic state

### 24. Basic Calculator II
- Difficulty: Medium
- Pattern: Expression stack
- Priority: High
- Core lesson: Operator precedence

### 25. Longest Valid Parentheses
- Difficulty: Hard
- Pattern: Stack / DP
- Priority: Very High
- Core lesson: Boundary indices

### 26. Parsing A Boolean Expression
- Difficulty: Hard
- Pattern: Stack parsing
- Priority: Medium
- Core lesson: Nested expression evaluation

### 27. Word Break II
- Difficulty: Hard
- Pattern: DFS + memoization
- Priority: Medium
- Note: Not primarily a stack problem; useful as a contrast against forcing stack usage.

---

# 54. Recommended Mastery Order

Follow this exact order.

## Phase 1 — Stack fundamentals

1. Stack implementation
2. Array implementation
3. Linked-list implementation
4. `Deque`
5. Push/pop/peek
6. Stack simulation

## Phase 2 — Parentheses

7. Valid Parentheses
8. Minimum Add to Make Parentheses Valid
9. Remove Outermost Parentheses
10. Longest Valid Parentheses

## Phase 3 — Basic monotonic stack

11. Next Greater Element I
12. Next Smaller Element
13. Previous Greater
14. Previous Smaller

## Phase 4 — Applications

15. Daily Temperatures
16. Stock Span
17. Next Greater Element II
18. Remove K Digits
19. Remove Duplicate Letters

## Phase 5 — Advanced

20. Largest Rectangle in Histogram
21. Maximal Rectangle
22. Trapping Rain Water
23. Basic Calculator
24. Nested parsing problems

---

# 55. Pattern Recognition Cheat Sheet

| If the problem says... | Think... |
|---|---|
| Matching brackets | Stack |
| Nested structure | Stack |
| Undo previous operation | Stack |
| Adjacent cancellation | Stack |
| Collision with previous item | Stack |
| Next greater | Monotonic stack |
| Next smaller | Monotonic stack |
| Previous greater | Monotonic stack |
| Previous smaller | Monotonic stack |
| Number of days until greater | Monotonic stack |
| Consecutive previous smaller/equal | Stock span |
| Largest rectangle | Increasing monotonic stack |
| Rectangle boundaries | Previous/next smaller |
| Nested arithmetic | Stack |
| Nested decoding | Stack |
| Frequency + lexicographically smallest subsequence | Frequency + monotonic stack |

---

# 56. Monotonic Stack Formula Sheet

## Next greater to right

```java
for (int i = n - 1; i >= 0; i--) {
    while (!stack.isEmpty() && value(stack.peek()) <= value(i)) {
        stack.pop();
    }

    answer[i] = stack.isEmpty() ? -1 : value(stack.peek());
    stack.push(i);
}
```

## Next smaller to right

```java
for (int i = n - 1; i >= 0; i--) {
    while (!stack.isEmpty() && value(stack.peek()) >= value(i)) {
        stack.pop();
    }

    answer[i] = stack.isEmpty() ? -1 : value(stack.peek());
    stack.push(i);
}
```

## Previous greater to left

```java
for (int i = 0; i < n; i++) {
    while (!stack.isEmpty() && value(stack.peek()) <= value(i)) {
        stack.pop();
    }

    answer[i] = stack.isEmpty() ? -1 : value(stack.peek());
    stack.push(i);
}
```

## Previous smaller to left

```java
for (int i = 0; i < n; i++) {
    while (!stack.isEmpty() && value(stack.peek()) >= value(i)) {
        stack.pop();
    }

    answer[i] = stack.isEmpty() ? -1 : value(stack.peek());
    stack.push(i);
}
```

---

# 57. Histogram Formula Sheet

For bar `i`:

```text
left = index of nearest smaller element on left
right = index of nearest smaller element on right
```

Then:

```text
width = right - left - 1
area = heights[i] × width
```

If no smaller element exists:

```text
left = -1
right = n
```

Therefore:

```text
width = n
```

for a bar spanning the entire histogram.

---

# 58. GATE / CS Theory Focus

For GATE-style preparation, know:

- LIFO principle
- Stack ADT
- Array-based implementation
- Linked-list implementation
- Overflow and underflow
- Stack complexity
- Recursion and call stack
- Infix/prefix/postfix expressions
- Operator precedence
- Parentheses matching
- Expression conversion
- Expression evaluation
- Applications of stacks
- Monotonic-stack reasoning
- Amortized analysis of push/pop sequences

---

# 59. Three GATE-Style Solved PYQs

## PYQ 1 — Stack Permutation

A stack initially receives the elements:

```text
1, 2, 3, 4
```

in that order.

Which output sequence is possible?

A. `2, 1, 4, 3`  
B. `3, 1, 4, 2`  
C. `3, 2, 4, 1`  
D. `4, 1, 2, 3`

### Solution

Input:

```text
1 2 3 4
```

For A:

```text
push 1
push 2
pop 2
pop 1

push 3
push 4
pop 4
pop 3
```

Output:

```text
2 1 4 3
```

Therefore A is possible.

### Answer

**A**

### Concept

Stack permutations must respect LIFO among elements currently present in the stack.

---

## PYQ 2 — Postfix Evaluation

Evaluate:

```text
5 2 3 * + 4 -
```

### Solution

Process left to right.

```text
5
5 2
5 2 3
```

`*`:

```text
2 × 3 = 6
```

Stack:

```text
5 6
```

`+`:

```text
5 + 6 = 11
```

Stack:

```text
11
```

Push `4`:

```text
11 4
```

`-`:

```text
11 - 4 = 7
```

### Answer

```text
7
```

---

## PYQ 3 — Amortized Stack Operations

An algorithm pushes every array element onto a stack and may pop elements inside a while-loop. Each element is pushed once and popped at most once.

What is the total worst-case time complexity?

A. `O(log n)`  
B. `O(n)`  
C. `O(n log n)`  
D. `O(n²)`

### Solution

Although a `while` loop is nested inside a `for` loop, each element can be:

```text
pushed once
popped once
```

Therefore the total number of stack operations is bounded by a constant multiple of `n`.

```text
Total = O(n)
```

### Answer

**B — O(n)**

### Key GATE/interview concept

This is **amortized analysis**.

Nested loops do not automatically imply `O(n²)`.

---

# 60. Interview-Level Variations You Must Be Able to Handle

After solving the standard problems, modify them yourself.

### Next Greater

Be able to solve:

- Next greater to right
- Next greater to left
- Circular next greater
- Next greater distance
- Next greater index
- Next greater among a subset

### Histogram

Be able to solve:

- Largest rectangle
- Previous smaller
- Next smaller
- Maximal rectangle
- Histogram with duplicate heights

### Parentheses

Be able to solve:

- Valid parentheses
- Minimum removals
- Minimum additions
- Longest valid substring
- Score of parentheses
- Nested depth

### Simulation

Be able to solve:

- Backspace
- Collision
- Adjacent removal
- Undo
- Nested decoding
- Path simplification

---

# 61. Master Templates

## Basic stack

```java
Deque<Integer> stack = new ArrayDeque<>();

stack.push(x);
int top = stack.peek();
int removed = stack.pop();
```

## Character stack

```java
Deque<Character> stack = new ArrayDeque<>();
```

## Index stack

```java
Deque<Integer> stack = new ArrayDeque<>();
```

## Monotonic stack

```java
while (!stack.isEmpty() && condition(stack.peek(), current)) {
    stack.pop();
}

stack.push(current);
```

## Stack simulation

```java
for (item : input) {
    if (conflict(stack.peek(), item)) {
        stack.pop();
    } else {
        stack.push(item);
    }
}
```

---

# 62. Final Mastery Checklist

You are ready to move on only when you can solve these without looking up the pattern:

### Fundamentals
- [ ] Implement stack with array
- [ ] Implement stack with linked list
- [ ] Use `ArrayDeque`
- [ ] Explain LIFO
- [ ] Explain overflow/underflow

### Parentheses
- [ ] Valid Parentheses
- [ ] Minimum Add to Make Parentheses Valid
- [ ] Longest Valid Parentheses

### Monotonic stack
- [ ] Next greater
- [ ] Next smaller
- [ ] Previous greater
- [ ] Previous smaller
- [ ] Circular next greater
- [ ] Daily Temperatures
- [ ] Stock Span

### Advanced
- [ ] Remove K Digits
- [ ] Remove Duplicate Letters
- [ ] Largest Rectangle in Histogram
- [ ] Maximal Rectangle
- [ ] Trapping Rain Water

### Expression problems
- [ ] Postfix evaluation
- [ ] RPN
- [ ] Infix evaluation
- [ ] Operator precedence
- [ ] Nested expressions

### Simulation
- [ ] Backspace
- [ ] Adjacent duplicate removal
- [ ] Asteroid Collision
- [ ] Decode String
- [ ] Simplify Path

### Interview explanation
- [ ] Explain why monotonic stack is O(n)
- [ ] Explain amortized analysis
- [ ] Explain index vs value stack
- [ ] Explain duplicate handling
- [ ] Derive pop condition instead of memorizing it
- [ ] Explain sentinel technique
- [ ] State time and space complexity

---

# 63. One-Page Revision Sheet

```text
STACK
│
├── Basics
│   ├── push
│   ├── pop
│   ├── peek
│   └── ArrayDeque
│
├── Parentheses
│   ├── Valid Parentheses
│   ├── Minimum Additions
│   └── Longest Valid Parentheses
│
├── Expression
│   ├── Postfix
│   ├── RPN
│   ├── Infix
│   └── Precedence
│
├── Monotonic Stack
│   ├── Next Greater
│   ├── Next Smaller
│   ├── Previous Greater
│   ├── Previous Smaller
│   ├── Daily Temperatures
│   ├── Stock Span
│   └── Circular NGE
│
├── Histogram
│   ├── Previous Smaller
│   ├── Next Smaller
│   ├── Largest Rectangle
│   └── Maximal Rectangle
│
├── Stack + HashMap
│   └── Frequency + future availability
│
└── Simulation
    ├── Backspace
    ├── Adjacent duplicates
    ├── Asteroids
    ├── Decode String
    └── Simplify Path
```

## Core rule

> **If an element is unresolved until a later element arrives, and each element should be processed only a constant number of times, look for a monotonic stack.**

## Core complexity

```text
Normal stack operation: O(1)
Monotonic stack overall: O(n)
Stack space: O(n)
```

## Most important problems

```text
1. Valid Parentheses
2. Next Greater Element I
3. Daily Temperatures
4. Stock Span
5. Remove K Digits
6. Remove Duplicate Letters
7. Largest Rectangle in Histogram
8. Maximal Rectangle
9. Trapping Rain Water
10. Basic Calculator
```
