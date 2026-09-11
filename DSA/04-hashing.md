# 4. Hashing

## 1. What You Must Master

Hashing is one of the highest-value interview techniques because it converts many repeated searches from `O(n)` into expected `O(1)` lookups.

For FAANG/top product-company interviews, you should be able to recognize these patterns quickly:

- Store a value → retrieve information about that value.
- Count occurrences.
- Detect whether something has appeared before.
- Find a complement needed to reach a target.
- Store prefix information and query it later.
- Group elements by a computed key.
- Represent a compound/custom object as a hashable key.
- Hash arrays/strings when direct comparison is expensive.
- Understand collisions and why hash tables are still expected `O(1)`.

---

# 2. HashMap

## 2.1 What Is a HashMap?

A `HashMap<K, V>` stores key-value pairs.

Example:

```text
key       value
"apple"     3
"banana"    5
"mango"     2
```

The key is used to locate the associated value.

### Java

```java
Map<String, Integer> map = new HashMap<>();

map.put("apple", 3);
map.put("banana", 5);

System.out.println(map.get("apple")); // 3
```

### Core operations

| Operation | Average | Worst-case discussion |
|---|---:|---:|
| `put` | O(1) expected | O(n) in pathological cases |
| `get` | O(1) expected | O(n) in pathological cases |
| `containsKey` | O(1) expected | O(n) |
| `remove` | O(1) expected | O(n) |
| `size` | O(1) | O(1) |

For interview analysis, normally state **expected O(1)** for hash-table operations.

---

## 2.2 Important Java Methods

```java
Map<Integer, Integer> map = new HashMap<>();

map.put(10, 100);

map.get(10);
map.containsKey(10);
map.remove(10);
map.size();
map.isEmpty();
map.clear();
```

### `getOrDefault`

Very useful for frequency counting:

```java
map.put(x, map.getOrDefault(x, 0) + 1);
```

Equivalent logic:

```java
if (!map.containsKey(x)) {
    map.put(x, 1);
} else {
    map.put(x, map.get(x) + 1);
}
```

Prefer `getOrDefault` in most interview solutions.

---

## 2.3 Iterating Through a HashMap

### Keys

```java
for (int key : map.keySet()) {
    System.out.println(key);
}
```

### Values

```java
for (int value : map.values()) {
    System.out.println(value);
}
```

### Key-value pairs

```java
for (Map.Entry<Integer, Integer> entry : map.entrySet()) {
    int key = entry.getKey();
    int value = entry.getValue();
}
```

---

# 3. HashSet

## 3.1 What Is a HashSet?

A `HashSet` stores unique values.

```java
Set<Integer> set = new HashSet<>();

set.add(10);
set.add(20);
set.add(10);
```

The set contains:

```text
10
20
```

The duplicate `10` is ignored.

---

## 3.2 Core Operations

| Operation | Expected |
|---|---:|
| `add` | O(1) |
| `contains` | O(1) |
| `remove` | O(1) |
| `size` | O(1) |

Use `HashSet` when you care primarily about **membership/uniqueness**, not associated values.

### Typical decision

```text
Need only "have I seen this?" → HashSet

Need "how many times?" → HashMap

Need "what information belongs to this value?" → HashMap
```

---

# 4. Frequency Map

Frequency maps are one of the most common hashing patterns.

## 4.1 Basic Frequency Counting

Given:

```text
[2, 3, 2, 5, 3, 2]
```

Frequency:

```text
2 → 3
3 → 2
5 → 1
```

### Java

```java
Map<Integer, Integer> freq = new HashMap<>();

for (int x : nums) {
    freq.put(x, freq.getOrDefault(x, 0) + 1);
}
```

---

## 4.2 Character Frequency

```java
Map<Character, Integer> freq = new HashMap<>();

for (char c : s.toCharArray()) {
    freq.put(c, freq.getOrDefault(c, 0) + 1);
}
```

For lowercase English letters, an array is often faster and simpler:

```java
int[] freq = new int[26];

for (char c : s.toCharArray()) {
    freq[c - 'a']++;
}
```

### Interview rule

If the domain is small and fixed:

```text
'a'–'z' → int[26]
'0'–'9' → int[10]
```

If the domain is unknown/general:

```text
HashMap
```

---

# 5. Counting

Hashing can count many different categories.

## 5.1 Count Distinct Values

```java
Set<Integer> set = new HashSet<>();

for (int x : nums) {
    set.add(x);
}

int distinct = set.size();
```

---

## 5.2 Count Values Meeting a Frequency Condition

Example:

> Count elements appearing at least twice.

```java
Map<Integer, Integer> freq = new HashMap<>();

for (int x : nums) {
    freq.put(x, freq.getOrDefault(x, 0) + 1);
}

int answer = 0;

for (int count : freq.values()) {
    if (count >= 2) {
        answer++;
    }
}
```

Be careful about what the question asks:

- number of distinct values appearing twice
- number of duplicate occurrences
- number of pairs
- total elements belonging to repeated groups

These are different quantities.

---

# 6. Duplicate Detection

## 6.1 Basic Duplicate Detection

Given an array, determine whether any value occurs more than once.

```java
Set<Integer> seen = new HashSet<>();

for (int x : nums) {
    if (!seen.add(x)) {
        return true;
    }
}

return false;
```

`HashSet.add()` returns:

```text
true  → value was not present
false → value already existed
```

This is a very useful Java trick.

---

## 6.2 Complexity

For `n` elements:

```text
Time  : O(n) expected
Space : O(n)
```

---

## 6.3 Common Variations

Duplicate detection appears in:

- Contains Duplicate
- Nearby duplicates
- Duplicate within distance `k`
- Duplicate strings
- Duplicate rows
- Duplicate states in graph/search problems
- Detecting repeated prefix states

---

# 7. Complement Lookup

This is the classic **Two Sum pattern**.

Suppose:

```text
nums = [2, 7, 11, 15]
target = 9
```

For each number `x`, calculate:

```text
complement = target - x
```

If the complement has already been seen, the answer is found.

---

## 7.1 Java

```java
public int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> map = new HashMap<>();

    for (int i = 0; i < nums.length; i++) {
        int complement = target - nums[i];

        if (map.containsKey(complement)) {
            return new int[] {map.get(complement), i};
        }

        map.put(nums[i], i);
    }

    return new int[] {};
}
```

---

## 7.2 Why It Works

At index `i`, we need:

```text
nums[j] + nums[i] = target
```

Therefore:

```text
nums[j] = target - nums[i]
```

The HashMap answers:

> Have we already seen `target - nums[i]`?

in expected `O(1)`.

---

## 7.3 General Complement Pattern

Do not memorize only Two Sum.

Think:

```text
Current value
      ↓
What value/state would complete the requirement?
      ↓
Have I seen that complement before?
      ↓
HashMap/HashSet lookup
```

This generalizes to:

- Two Sum
- Four Sum variants
- Difference equals `k`
- Pair with a target difference
- Frequency-based pair counting
- String complement patterns

---

# 8. Prefix Sum + HashMap

This is one of the most important advanced hashing patterns.

## 8.1 Core Idea

Suppose:

```text
prefix[i] = sum of elements from 0 through i
```

For a subarray `(j + 1 ... i)`:

```text
sum = prefix[i] - prefix[j]
```

If we want:

```text
sum = k
```

then:

```text
prefix[i] - prefix[j] = k
```

Rearrange:

```text
prefix[j] = prefix[i] - k
```

Therefore, while scanning the array:

```text
currentPrefix = prefix[i]

neededPrefix = currentPrefix - k
```

If `neededPrefix` has appeared before, a subarray summing to `k` exists.

That is the central pattern.

---

# 9. Subarray Sum Equals K

## 9.1 Example

```text
nums = [1, 1, 1]
k = 2
```

Prefix sums:

```text
index       0  1  2
value       1  1  1
prefix      1  2  3
```

At prefix `2`:

```text
2 - 2 = 0
```

So the initial prefix sum `0` is required.

At prefix `3`:

```text
3 - 2 = 1
```

Prefix sum `1` occurred earlier, giving another valid subarray.

Answer:

```text
2
```

---

## 9.2 Java

```java
public int subarraySum(int[] nums, int k) {
    Map<Integer, Integer> prefixCount = new HashMap<>();

    prefixCount.put(0, 1);

    int prefix = 0;
    int answer = 0;

    for (int x : nums) {
        prefix += x;

        int needed = prefix - k;

        answer += prefixCount.getOrDefault(needed, 0);

        prefixCount.put(
            prefix,
            prefixCount.getOrDefault(prefix, 0) + 1
        );
    }

    return answer;
}
```

---

## 9.3 Why `prefixCount.put(0, 1)`?

It represents the empty prefix before the array starts.

For:

```text
[5]
k = 5
```

At the first element:

```text
prefix = 5
needed = 5 - 5 = 0
```

The initial zero must already exist so the algorithm recognizes `[5]`.

---

## 9.4 Why Store Counts, Not Just Presence?

Because the same prefix sum can occur multiple times.

If:

```text
prefix = 7
```

has appeared three times, then a later prefix requiring `7` gives **three different subarrays**.

Therefore:

```text
HashMap<prefixSum, frequency>
```

is required for counting all subarrays.

---

# 10. Prefix Sum + Hashing Variations

This pattern appears in many forms.

### Target sum

```text
prefix - k
```

### Sum divisible by k

Store:

```text
prefix % k
```

If the same remainder appears twice, their difference is divisible by `k`.

### Binary array with target

Convert:

```text
0 → -1
1 → +1
```

Then equal prefix sums identify subarrays with equal numbers of zeros and ones.

### Longest subarray

Instead of storing frequency, often store the **first index** at which a prefix state appeared.

Why?

If the same prefix state appears at indices `j` and `i`, then:

```text
length = i - j
```

To maximize the length, keep the earliest index.

---

# 11. Grouping

Hashing is extremely useful when several objects need to be grouped according to a property.

Examples:

- Group Anagrams
- Group equal values
- Group strings by character frequency
- Group intervals/states by a key
- Group records by category
- Group numbers by remainder
- Group coordinates by a normalized representation

---

## 11.1 Group Anagrams

Input:

```text
["eat", "tea", "tan", "ate", "nat", "bat"]
```

Anagrams have the same character-frequency signature.

For example:

```text
eat → a:1,e:1,t:1
tea → a:1,e:1,t:1
ate → a:1,e:1,t:1
```

Therefore they get the same key.

---

## 11.2 HashMap of Lists

```java
Map<String, List<String>> groups = new HashMap<>();

for (String s : strs) {
    char[] chars = s.toCharArray();
    Arrays.sort(chars);

    String key = new String(chars);

    groups
        .computeIfAbsent(key, k -> new ArrayList<>())
        .add(s);
}
```

Result conceptually:

```text
aet → [eat, tea, ate]
ant → [tan, nat]
abt → [bat]
```

---

## 11.3 Frequency Signature Instead of Sorting

Sorting each string costs:

```text
O(L log L)
```

A frequency signature can reduce this to:

```text
O(L)
```

for a fixed alphabet.

Example:

```text
a1#b0#c0#...#e1#...#t1
```

Then use that signature as the key.

This is often preferable when strings are long.

---

# 12. Custom Keys

Many interview problems require a key made from multiple properties.

Examples:

```text
(row, column)
(x, y)
(word length, first character)
(value, index)
(character counts)
```

The key must correctly implement:

```text
equals()
hashCode()
```

---

## 12.1 Java Record as a Custom Key

Modern Java provides a clean solution:

```java
record Pair(int x, int y) {}
```

Then:

```java
Set<Pair> seen = new HashSet<>();

seen.add(new Pair(2, 3));

if (seen.contains(new Pair(2, 3))) {
    // found
}
```

Records automatically provide appropriate `equals()` and `hashCode()` implementations based on their components.

---

## 12.2 Custom Class

If using a normal class:

```java
class Pair {
    int x;
    int y;

    Pair(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (!(obj instanceof Pair)) return false;

        Pair other = (Pair) obj;
        return x == other.x && y == other.y;
    }

    @Override
    public int hashCode() {
        return Objects.hash(x, y);
    }
}
```

---

## 12.3 Critical Rule

If two objects are considered equal:

```text
a.equals(b) == true
```

they must have:

```text
a.hashCode() == b.hashCode()
```

The reverse is not required.

Different objects may have the same hash code because of collisions.

---

# 13. Hashing Arrays

Arrays in Java do not behave like value objects for `HashMap`/`HashSet`.

This is dangerous:

```java
int[] a = {1, 2, 3};
int[] b = {1, 2, 3};

System.out.println(a.equals(b)); // false
```

`a` and `b` are different array objects.

---

## 13.1 Correct Array Content Comparison

Use:

```java
Arrays.equals(a, b)
```

For nested arrays:

```java
Arrays.deepEquals(a, b)
```

---

## 13.2 Correct Array Hash

Use:

```java
Arrays.hashCode(a)
```

For nested arrays:

```java
Arrays.deepHashCode(array)
```

---

## 13.3 Arrays as HashMap Keys

Avoid directly using mutable arrays as keys unless you understand the equality/hash behavior.

Better approaches:

### Convert to a list

```java
List<Integer> key = Arrays.stream(arr)
                           .boxed()
                           .toList();
```

### Convert to a canonical string

```java
String key = Arrays.toString(arr);
```

### Use a custom immutable key

Best when performance and correctness matter.

---

# 14. Hashing Strings

Strings make excellent hash keys because Java `String` has content-based `equals()` and `hashCode()`.

```java
Map<String, Integer> map = new HashMap<>();

map.put("hello", 10);

System.out.println(map.get("hello")); // 10
```

Two distinct `String` objects with the same characters compare equal.

---

## 14.1 String as a Canonical Key

Examples:

```text
sorted characters
frequency signature
normalized string
lowercase string
removed punctuation
```

The key should represent exactly the property by which objects are grouped.

---

# 15. Hashing vs Sorting

Suppose you need to detect duplicates.

### Sorting

```text
sort → compare neighbors
```

Complexity:

```text
O(n log n)
```

### HashSet

```text
insert → membership check
```

Expected:

```text
O(n)
```

but uses:

```text
O(n)
```

extra space.

### Decision

| Requirement | Preferred |
|---|---|
| Need expected linear time | Hashing |
| Need constant/low extra space | Sorting |
| Need counts | HashMap |
| Need membership only | HashSet |
| Need ordered output | Sorting / TreeMap / TreeSet |
| Need grouping | HashMap |
| Need repeated lookup | HashMap/HashSet |

---

# 16. Collision Concept

## 16.1 What Is a Collision?

A collision occurs when different keys map to the same hash-table bucket/index.

Conceptually:

```text
key A ──hash──→ bucket 5
key B ──hash──→ bucket 5
```

Even though:

```text
A != B
```

they can have the same hash value or ultimately map to the same bucket.

---

## 16.2 Why Do Collisions Exist?

Hash tables usually have a finite number of buckets, while the possible keys may be enormous.

Therefore, different keys can map to the same bucket.

This follows from the pigeonhole principle.

---

## 16.3 Collision Handling

Common strategies include:

### Separate chaining

Each bucket stores multiple entries.

Conceptually:

```text
bucket 5 → entry A → entry B → entry C
```

### Open addressing

If a bucket is occupied, another location is searched according to a probing strategy.

Examples:

- Linear probing
- Quadratic probing
- Double hashing

---

# 17. Why HashMap Is Expected O(1)

A hash table computes something like:

```text
index = hash(key) → bucket
```

Ideally, keys are distributed across buckets.

Therefore lookup usually examines only a small amount of data.

Hence:

```text
Expected:
get    → O(1)
put    → O(1)
remove → O(1)
```

But poor hashing/distribution can create many collisions.

Therefore, do not blindly claim that hashing is always `O(1)`.

Say:

> Expected/amortized O(1), depending on the implementation and hash distribution.

---

# 18. Load Factor

A hash table becomes less efficient as it becomes too full.

The **load factor** is conceptually:

```text
number of stored entries / number of buckets
```

When the table reaches an implementation-specific threshold, it can resize and rehash entries.

Java's `HashMap` has a default load factor of `0.75`.

For interviews, the important concept is:

```text
More entries relative to buckets
        ↓
More potential collisions
        ↓
Resize when threshold is reached
        ↓
Redistribute entries
```

---

# 19. HashMap vs HashSet

| Feature | HashMap | HashSet |
|---|---|---|
| Stores | key-value pairs | unique values |
| Main question | "What is associated with this key?" | "Have I seen this?" |
| Frequency counting | Excellent | Not enough |
| Duplicate detection | Possible | Excellent |
| Complement lookup | Excellent | Sometimes |
| Grouping | Excellent | Not enough |
| Expected lookup | O(1) | O(1) |

---

# 20. HashMap Pattern: Value → Index

Sometimes you need the location of an earlier element.

```java
Map<Integer, Integer> firstIndex = new HashMap<>();

for (int i = 0; i < nums.length; i++) {
    firstIndex.putIfAbsent(nums[i], i);
}
```

Useful for:

- Longest subarray
- First occurrence
- Distance between equal values
- Prefix-state problems

---

# 21. HashMap Pattern: State → Count

Use:

```text
state → number of occurrences
```

Examples:

```text
value → frequency
prefix sum → frequency
remainder → frequency
string signature → frequency
```

Java:

```java
map.put(state, map.getOrDefault(state, 0) + 1);
```

---

# 22. HashMap Pattern: State → First Index

Use:

```text
state → earliest position
```

This is especially important for longest-subarray problems.

Example:

```java
Map<Integer, Integer> first = new HashMap<>();

first.put(0, -1);

int prefix = 0;

for (int i = 0; i < nums.length; i++) {
    prefix += nums[i];

    if (first.containsKey(prefix)) {
        int length = i - first.get(prefix);
        // update answer
    } else {
        first.put(prefix, i);
    }
}
```

### Why don't we overwrite?

For maximum length, the earliest occurrence produces the largest distance to the current index.

---

# 23. HashMap Pattern: Key → List

Use this for grouping.

```java
Map<String, List<String>> map = new HashMap<>();

map.computeIfAbsent(key, k -> new ArrayList<>())
   .add(value);
```

This pattern is extremely common in interview solutions.

---

# 24. HashMap Pattern: Key → Set

Useful when each key has multiple unique associated values.

```java
Map<String, Set<Integer>> map = new HashMap<>();

map.computeIfAbsent(key, k -> new HashSet<>())
   .add(value);
```

Example applications:

- Unique users per category
- Graph adjacency with duplicate prevention
- Unique indices associated with a value

---

# 25. Canonicalization

A major hashing technique is converting equivalent objects into the same canonical key.

Examples:

### Anagrams

```text
"eat"
"tea"
"ate"
```

Canonical key:

```text
"aet"
```

### Case-insensitive comparison

```text
"Apple"
"apple"
```

Canonical key:

```text
"apple"
```

### Coordinate normalization

Equivalent structures can be shifted/normalized before hashing.

The general pattern:

```text
Object
  ↓
Normalize / canonicalize
  ↓
Hashable key
  ↓
HashMap / HashSet
```

---

# 26. Hashing for Graph/Grid Problems

Hashing is useful for representing visited states.

Example:

```java
record Cell(int r, int c) {}

Set<Cell> visited = new HashSet<>();
```

Then:

```java
Cell cell = new Cell(r, c);

if (visited.contains(cell)) {
    // already visited
}

visited.add(cell);
```

This is cleaner than manually encoding coordinates when using Java records.

---

# 27. String Hashing for Algorithms

There are two different meanings of "string hashing" in interviews.

## Meaning 1: Hash table usage

Use:

```java
HashMap<String, ...>
HashSet<String>
```

This gives expected constant-time lookup for whole strings.

## Meaning 2: Polynomial/Rolling Hash

Represent a string using a numeric hash so substrings can be compared efficiently.

A common form is:

```text
H(s) = s[0] * B^(n-1)
     + s[1] * B^(n-2)
     + ...
     + s[n-1]
```

computed modulo some number.

This is useful for:

- Substring comparison
- Duplicate substring detection
- Pattern matching
- Rabin-Karp
- Longest common substring variants
- Binary search + hashing

---

# 28. Rolling Hash

Suppose:

```text
"abcdef"
```

and we want hashes of consecutive windows.

Instead of recomputing every window from scratch, remove the outgoing character and add the incoming character.

Conceptually:

```text
hash("abc")
      ↓ slide
hash("bcd")
      ↓ slide
hash("cde")
```

This can make each window update `O(1)`.

---

## 28.1 Collision Warning

Two different strings can have the same hash.

Therefore:

```text
same hash ≠ guaranteed same string
```

Hash equality is generally a fast filter, not mathematical proof of equality unless the algorithm provides an appropriate deterministic guarantee.

For high-confidence randomized hashing, techniques such as:

- Two independent moduli
- Different bases
- 64-bit hashing with suitable assumptions

can reduce collision probability.

---

# 29. Hashing and Subarrays

You should recognize the following forms immediately.

### Zero-sum subarray

If the same prefix sum appears twice:

```text
prefix[i] == prefix[j]
```

then:

```text
sum(j+1 ... i) = 0
```

### Sum equals K

Look for:

```text
prefix - K
```

### Divisible by K

Track:

```text
prefix % K
```

### Equal 0s and 1s

Convert:

```text
0 → -1
1 → +1
```

Then look for repeated prefix sums.

---

# 30. Hashing and Differences

Suppose you need:

```text
nums[j] - nums[i] = k
```

Then:

```text
nums[j] = nums[i] + k
```

or:

```text
nums[i] = nums[j] - k
```

Depending on the direction, store previous values in a HashSet/HashMap and perform complement-style lookup.

Always derive the needed value algebraically instead of memorizing a formula.

---

# 31. Counting Pairs with Hashing

Example:

> Count pairs whose sum is `k`.

For each `x`, the required earlier value is:

```text
k - x
```

If its frequency is `f`, then `f` new pairs are formed.

```java
Map<Integer, Integer> freq = new HashMap<>();

long answer = 0;

for (int x : nums) {
    int complement = k - x;

    answer += freq.getOrDefault(complement, 0);

    freq.put(x, freq.getOrDefault(x, 0) + 1);
}
```

### Important

Use `long` for the answer when the number of pairs can be `O(n²)`.

---

# 32. Hashing with Negative Numbers

Java's `HashMap<Integer, Integer>` naturally handles negative integers.

For prefix sums, this is particularly important.

Example:

```text
nums = [1, -1, 2, -2]
```

Prefix sums can move back and forth:

```text
1, 0, 2, 0
```

A HashMap handles these values directly.

Do not assume prefix sums are positive.

---

# 33. Hashing and Sliding Window

Hashing frequently appears inside sliding-window problems.

Example:

> Longest substring without repeating characters.

Maintain:

```text
character → latest index
```

Then when a duplicate appears, jump the left pointer.

```java
Map<Character, Integer> lastSeen = new HashMap<>();

int left = 0;
int answer = 0;

for (int right = 0; right < s.length(); right++) {
    char c = s.charAt(right);

    if (lastSeen.containsKey(c)) {
        left = Math.max(left, lastSeen.get(c) + 1);
    }

    answer = Math.max(answer, right - left + 1);
    lastSeen.put(c, right);
}
```

This gives expected:

```text
Time  : O(n)
Space : O(min(n, alphabet size))
```

---

# 34. Hashing + Sorting

Sometimes the strongest solution combines both.

Examples:

- Sort first, then hash selected values.
- Sort each string to form an anagram key.
- Sort coordinates before generating a canonical key.
- Sort intervals, then use hashing for auxiliary state.

Do not treat patterns as mutually exclusive.

Interview problems often combine techniques.

---

# 35. Hashing vs Two Pointers

A common decision:

### Two pointers

Usually works when the data has useful order, often after sorting.

```text
sort → left/right movement
```

### Hashing

Works without requiring ordering.

```text
store → lookup
```

Example:

```text
Two Sum:
HashMap → O(n) expected
Sort + two pointers → O(n log n)
```

Choose based on the required output and constraints.

---

# 36. Common Interview Mistakes

## Mistake 1: Saying HashMap is always O(1)

Correct:

```text
Expected O(1)
```

because collisions can occur.

---

## Mistake 2: Using HashSet when frequency is required

A set tells you:

```text
exists / does not exist
```

It does not tell you:

```text
how many times
```

Use a HashMap for frequencies.

---

## Mistake 3: Overwriting the first index

For longest-subarray problems:

```java
first.put(prefix, i);
```

every time can destroy the earliest occurrence.

Usually:

```java
first.putIfAbsent(prefix, i);
```

is correct.

---

## Mistake 4: Forgetting the empty prefix

For prefix-sum problems, often initialize:

```java
map.put(0, 1);
```

or:

```java
first.put(0, -1);
```

depending on whether you need counts or indices.

---

## Mistake 5: Incorrect custom key equality

If a custom class is used as a key, implement consistent:

```text
equals()
hashCode()
```

---

## Mistake 6: Mutable keys

Do not mutate fields that participate in equality/hash code after inserting an object into a hash table.

Otherwise the object may become effectively unreachable through normal lookup.

---

## Mistake 7: Using arrays as if they have content equality

Remember:

```java
a.equals(b)
```

does not perform array-content comparison.

Use:

```java
Arrays.equals(a, b)
Arrays.hashCode(a)
```

---

## Mistake 8: Ignoring integer overflow

Prefix sums and pair counts can exceed `int`.

Use:

```java
long
```

when constraints require it.

---

# 37. Complexity Cheat Sheet

| Pattern | Typical Time | Space |
|---|---:|---:|
| Frequency map | O(n) expected | O(n) |
| Duplicate detection | O(n) expected | O(n) |
| Two Sum | O(n) expected | O(n) |
| Count target-sum pairs | O(n) expected | O(n) |
| Subarray sum = K | O(n) expected | O(n) |
| Grouping | O(n) expected + key construction | O(n) |
| Grid coordinate hashing | O(n) expected | O(n) |
| HashSet membership | O(1) expected | O(n) |
| String hash table lookup | O(L) for hashing/comparison considerations | O(1) auxiliary per entry |
| Rolling hash window update | O(1) | O(1) |

Always account for the cost of **constructing the key**.

For example, if a key requires sorting a string of length `L`, grouping is not simply `O(n)`.

---

# 38. Interview Pattern Recognition

When you see:

### "Have we seen this before?"

Think:

```text
HashSet
```

### "How many times?"

Think:

```text
HashMap frequency
```

### "Find the other value needed"

Think:

```text
Complement lookup
```

### "Find a subarray with sum K"

Think:

```text
Prefix sum + HashMap
```

### "Longest subarray satisfying a prefix-state condition"

Think:

```text
Prefix state → first index
```

### "Count all subarrays satisfying a prefix condition"

Think:

```text
Prefix state → frequency
```

### "Group objects with the same property"

Think:

```text
Canonical key → List
```

### "Unique compound state"

Think:

```text
Custom key + HashSet
```

### "Fast substring comparison"

Think:

```text
Rolling hash
```

---

# 39. Must-Know Java Templates

## Frequency Map

```java
Map<Integer, Integer> freq = new HashMap<>();

for (int x : nums) {
    freq.put(x, freq.getOrDefault(x, 0) + 1);
}
```

---

## Duplicate Detection

```java
Set<Integer> seen = new HashSet<>();

for (int x : nums) {
    if (!seen.add(x)) {
        return true;
    }
}

return false;
```

---

## Complement Lookup

```java
Map<Integer, Integer> map = new HashMap<>();

for (int i = 0; i < nums.length; i++) {
    int need = target - nums[i];

    if (map.containsKey(need)) {
        // answer
    }

    map.put(nums[i], i);
}
```

---

## Prefix Sum + Count

```java
Map<Integer, Integer> count = new HashMap<>();

count.put(0, 1);

int prefix = 0;
int answer = 0;

for (int x : nums) {
    prefix += x;

    answer += count.getOrDefault(prefix - k, 0);

    count.put(prefix, count.getOrDefault(prefix, 0) + 1);
}
```

---

## Prefix State + First Index

```java
Map<Integer, Integer> first = new HashMap<>();

first.put(0, -1);

int prefix = 0;

for (int i = 0; i < nums.length; i++) {
    prefix += nums[i];

    if (first.containsKey(prefix)) {
        int length = i - first.get(prefix);
        // update maximum
    } else {
        first.put(prefix, i);
    }
}
```

---

## Grouping

```java
Map<String, List<String>> groups = new HashMap<>();

for (String s : strs) {
    String key = buildKey(s);

    groups.computeIfAbsent(key, k -> new ArrayList<>())
          .add(s);
}
```

---

## Custom Coordinate Key

```java
record Cell(int row, int col) {}

Set<Cell> visited = new HashSet<>();
```

---

# 40. Selected LeetCode Problems

The following set is intentionally focused on high-value interview patterns rather than listing every problem.

## Easy

### 1. Two Sum — LeetCode 1

**Pattern:** Complement lookup

**Core lesson:**

```text
target - current
```

Use a HashMap to find the required earlier value.

**Priority:** Essential

---

### 2. Contains Duplicate — LeetCode 217

**Pattern:** HashSet / duplicate detection

**Core lesson:**

```java
if (!seen.add(x))
```

**Priority:** Essential

---

### 3. Valid Anagram — LeetCode 242

**Pattern:** Frequency counting

**Core lesson:**

Two strings are anagrams if their character frequencies match.

**Priority:** Essential

---

### 4. Majority Element — LeetCode 169

**Pattern:** Frequency map / counting

Hashing solution:

```java
Map<Integer, Integer> freq = new HashMap<>();

for (int x : nums) {
    int count = freq.getOrDefault(x, 0) + 1;

    if (count > nums.length / 2) {
        return x;
    }

    freq.put(x, count);
}

return -1;
```

**Priority:** High

Note: Boyer-Moore is the optimal constant-space technique and should also be learned separately.

---

## Medium

### 5. Group Anagrams — LeetCode 49

**Pattern:** Canonical key + grouping

**Core lesson:**

```text
string → canonical signature → HashMap
```

**Priority:** Essential

---

### 6. Subarray Sum Equals K — LeetCode 560

**Pattern:** Prefix sum + frequency map

**Core lesson:**

```text
needed = currentPrefix - k
```

**Priority:** Essential

---

### 7. Longest Substring Without Repeating Characters — LeetCode 3

**Pattern:** Sliding window + HashMap

**Core lesson:**

```text
character → latest index
```

**Priority:** Essential

---

### 8. Longest Consecutive Sequence — LeetCode 128

**Pattern:** HashSet

**Core lesson:**

Only start a sequence at:

```text
x where x - 1 is absent
```

This avoids sorting and gives expected `O(n)`.

**Priority:** Essential

---

### 9. Top K Frequent Elements — LeetCode 347

**Pattern:** Frequency map + bucket/heap

**Core lesson:**

Separate:

```text
counting
```

from:

```text
top-k extraction
```

**Priority:** Essential

---

### 10. 4Sum II — LeetCode 454

**Pattern:** Complement lookup + frequency counting

**Core lesson:**

Split the four arrays into two pairs:

```text
A + B
C + D
```

Store frequencies of one side and search for the complement.

**Priority:** High

---

### 11. Continuous Subarray Sum — LeetCode 523

**Pattern:** Prefix remainder + HashMap

**Core lesson:**

For divisibility by `k`, equal prefix remainders imply a subarray whose sum is divisible by `k`.

**Priority:** High

---

### 12. Subarray Sums Divisible by K — LeetCode 974

**Pattern:** Prefix remainder frequency

**Core lesson:**

```text
same remainder → divisible subarray
```

**Priority:** High

---

### 13. Longest Arithmetic Subsequence of Given Difference — LeetCode 1218

**Pattern:** Value/state → best length

**Core lesson:**

For current `x`, look for:

```text
x - difference
```

**Priority:** High

---

## Hard

### 14. Minimum Window Substring — LeetCode 76

**Pattern:** Sliding window + frequency maps

**Core lesson:**

Maintain:

```text
required frequencies
current-window frequencies
```

and shrink whenever the window remains valid.

**Priority:** Essential

---

### 15. Substring with Concatenation of All Words — LeetCode 30

**Pattern:** HashMap + fixed-step sliding window

**Core lesson:**

Combine:

```text
word frequency map
+
sliding window
```

**Priority:** High

---

### 16. First Missing Positive — LeetCode 41

**Pattern:** Hashing/array-index state

**Core lesson:**

This problem is especially valuable because the expected constant-extra-space solution requires moving away from a normal HashSet and using the input array itself as a presence structure.

**Priority:** High

---

# 41. LeetCode Progression

## Stage 1 — Easy

Master:

1. Contains Duplicate
2. Two Sum
3. Valid Anagram
4. Majority Element

Goal:

```text
Recognize basic hashing immediately.
```

---

## Stage 2 — Medium

Master:

1. Group Anagrams
2. Subarray Sum Equals K
3. Longest Substring Without Repeating Characters
4. Longest Consecutive Sequence
5. Top K Frequent Elements
6. 4Sum II
7. Continuous Subarray Sum
8. Subarray Sums Divisible by K
9. Longest Arithmetic Subsequence of Given Difference

Goal:

```text
Combine hashing with another technique.
```

---

## Stage 3 — Hard

Master:

1. Minimum Window Substring
2. Substring with Concatenation of All Words
3. First Missing Positive

Goal:

```text
Choose the right state representation under constraints.
```

---

# 42. GATE / CS Exam Relevance

Hashing is also important for theoretical computer-science questions.

Focus on:

- Hash functions
- Collision
- Collision resolution
- Separate chaining
- Open addressing
- Linear probing
- Quadratic probing
- Double hashing
- Load factor
- Expected search complexity
- Worst-case search complexity
- Successful vs unsuccessful search
- Hash-table insertion/deletion
- Primary clustering
- Secondary clustering
- Universal hashing concepts

For GATE, do not study hashing only as a coding technique. You need both:

```text
DSA implementation
+
data-structure theory
```

---

# 43. GATE PYQ Practice Set

## PYQ 1 — Hashing / Collision Resolution

**Question type:** Hash-table insertion and collision resolution.

Given a hash table and a specified collision-resolution method, determine the final table after inserting a sequence of keys.

### Method

For each key:

1. Compute the initial hash position.
2. Check whether it is occupied.
3. Apply the specified probing rule.
4. Continue until an allowed empty position is found.
5. Insert the key.

### Key formulas

Linear probing:

```text
h_i(k) = (h(k) + i) mod m
```

Quadratic probing commonly uses:

```text
h_i(k) = (h(k) + c1*i + c2*i²) mod m
```

Double hashing:

```text
h_i(k) = (h1(k) + i*h2(k)) mod m
```

### Exam trap

Do not substitute a collision-resolution method from memory. Use exactly the method specified by the question.

---

## PYQ 2 — Hashing / Load Factor

**Question type:** Determine the number of probes or expected search cost based on the load factor and collision-resolution strategy.

### Key idea

Load factor:

```text
α = n / m
```

where:

```text
n = number of stored keys
m = number of table slots
```

As `α` increases, collisions become more likely.

### Exam trap

Distinguish:

```text
successful search
```

from:

```text
unsuccessful search
```

and distinguish:

```text
separate chaining
```

from:

```text
open addressing
```

because their expected-cost formulas differ.

---

## PYQ 3 — Hashing / Probe Sequence

**Question type:** Given a hash function and collision strategy, identify the probe sequence or determine whether a key can be inserted.

### Method

Write every probe explicitly:

```text
h0
h1
h2
...
```

Reduce each value modulo table size.

For example, with linear probing:

```text
h_i(k) = (h(k) + i) mod m
```

Do not skip occupied positions.

### Exam trap

A correct first hash position does not mean the key is inserted there. Collision handling must be applied exactly as specified.

---

# 44. Important Hashing Theory

## Hash Function Requirements

A useful hash function should distribute keys reasonably uniformly across the table.

Desirable properties:

- Fast to compute.
- Deterministic for a given key.
- Good distribution.
- Low unnecessary clustering.

---

## Collision Is Not Equality

This is critical:

```text
same hash
```

does **not** imply:

```text
same key
```

A hash table still needs equality comparison to determine whether two keys are actually equal.

Conceptually:

```text
hash(key)
    ↓
bucket
    ↓
compare candidate keys using equality
```

---

# 45. Advanced Pattern: Frequency of Prefix States

Many difficult problems can be reduced to:

```text
state → frequency
```

Examples:

```text
prefix sum → count
prefix remainder → count
balance → count
character mask → count
parity state → count
```

The hardest part is often not the HashMap.

The hardest part is finding the correct **state**.

---

# 46. Advanced Pattern: Prefix State / Balance

Suppose:

```text
A → +1
B → -1
```

Then a prefix sum represents the balance between A and B.

If the same balance occurs twice:

```text
prefix[i] == prefix[j]
```

the segment between them has equal numbers of A and B.

This technique generalizes to:

- Equal 0s and 1s
- Balanced parentheses variants
- Character-balance problems
- Binary-state problems

---

# 47. Advanced Pattern: Bitmask as Hash Key

For small alphabets, a bitmask can represent a set of characters.

For example:

```text
a → bit 0
b → bit 1
c → bit 2
...
```

Then:

```java
mask |= 1 << (c - 'a');
```

A mask can become a compact hashable state.

This appears in problems involving:

- Character parity
- Unique-character sets
- Palindrome permutations
- Even/odd character counts

The state can then be stored in:

```java
HashMap<Integer, ...>
HashSet<Integer>
```

---

# 48. Advanced Pattern: Tuple State

When one value is not enough, hash the full state.

Examples:

```text
(value, remainder)
(row, column)
(left, right)
(countA, countB)
(mask, index)
```

Use:

```java
record State(int a, int b) {}
```

or another immutable representation.

The key design question is:

> What information completely determines whether two states are equivalent?

That information belongs in the key.

---

# 49. Hashing Decision Tree

When solving a new problem:

```text
Do I repeatedly need to ask whether something exists?
        │
        ├── Yes → HashSet / HashMap
        │
        └── No
             ↓
Do I need a frequency?
        │
        ├── Yes → HashMap
        │
        └── No
             ↓
Do I need the complement of the current value?
        │
        ├── Yes → HashMap / HashSet
        │
        └── No
             ↓
Is there a subarray/prefix-state condition?
        │
        ├── Yes → Prefix state + HashMap
        │
        └── No
             ↓
Do objects need grouping?
        │
        ├── Yes → Canonical key + HashMap
        │
        └── No
             ↓
Do I need fast substring comparison?
        │
        ├── Yes → Rolling hash
        │
        └── No → Consider other techniques
```

---

# 50. What to Memorize

Do not memorize hundreds of individual solutions.

Memorize these core patterns:

```text
1. value → frequency
2. value → index
3. state → first index
4. state → frequency
5. key → list
6. key → set
7. current → complement
8. prefix → required prefix
9. object → canonical key
10. coordinate → custom key
```

And these Java tools:

```java
HashMap
HashSet
getOrDefault
putIfAbsent
computeIfAbsent
equals
hashCode
Arrays.equals
Arrays.hashCode
record
```

---

# 51. Final Hashing Cheat Sheet

| Problem signal | Pattern |
|---|---|
| Duplicate? | HashSet |
| Seen before? | HashSet |
| Count frequency | HashMap |
| Find complement | HashMap |
| Two Sum | Complement HashMap |
| Count target-sum pairs | Frequency HashMap |
| Subarray sum = K | Prefix sum + frequency |
| Longest zero-sum subarray | Prefix sum + first index |
| Divisible by K | Prefix remainder + frequency |
| Equal 0s and 1s | Balance + first index/frequency |
| Group anagrams | Canonical key + HashMap |
| Group objects | Key → List |
| Unique associated values | Key → Set |
| Grid state | Custom key + HashSet |
| Complex state | Tuple/record key |
| Fast substring comparison | Rolling hash |
| Collision theory | Hash functions + resolution |
| Ordered hashing needed | TreeMap/TreeSet may be preferable |

---

# 52. Mastery Checklist

Before marking **Hashing** complete, you should be able to:

- [ ] Explain how a HashMap works conceptually.
- [ ] Explain HashSet.
- [ ] Implement frequency counting.
- [ ] Detect duplicates in expected O(n).
- [ ] Solve Two Sum using complement lookup.
- [ ] Count target-sum pairs.
- [ ] Derive prefix-sum + HashMap instead of memorizing it.
- [ ] Explain why `map.put(0, 1)` is needed.
- [ ] Distinguish prefix frequency from prefix first-index storage.
- [ ] Solve subarray-sum problems.
- [ ] Solve remainder-based subarray problems.
- [ ] Group values using canonical keys.
- [ ] Use `computeIfAbsent`.
- [ ] Create correct custom hash keys.
- [ ] Explain `equals()` and `hashCode()`.
- [ ] Explain why collisions happen.
- [ ] Explain separate chaining.
- [ ] Explain open addressing.
- [ ] Explain load factor.
- [ ] State expected vs worst-case complexity correctly.
- [ ] Understand Java array equality/hash behavior.
- [ ] Use records for compound keys.
- [ ] Understand rolling hash at a conceptual level.
- [ ] Recognize hashing + sliding-window combinations.
- [ ] Recognize hashing + prefix-sum combinations.
- [ ] Solve Easy hashing problems quickly.
- [ ] Solve Medium hashing problems without looking at solutions.
- [ ] Attempt Hard problems involving multiple combined patterns.
- [ ] Practice GATE hashing theory and numerical/probing questions.

---

# 53. Final Interview Rule

When a problem repeatedly asks:

```text
"Have I seen this?"
"How many times?"
"What was associated with this?"
"What value do I need?"
"Have I seen this prefix state?"
"Which objects belong to the same group?"
```

your first instinct should be:

```text
HASH IT.
```

Then determine what should be stored:

```text
value
frequency
index
prefix state
remainder
canonical signature
custom state
```

That choice is the actual skill being tested.
