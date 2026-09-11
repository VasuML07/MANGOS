# Strings

## 1. Why This Topic Matters

Strings are one of the highest-frequency interview topics because they combine:

- arrays,
- hashing,
- two pointers,
- sliding windows,
- prefix/suffix reasoning,
- sorting,
- pattern matching,
- tries,
- rolling hash.

In Java, strings also have language-specific behavior that matters in interviews:

- `String` is immutable,
- `StringBuilder` is mutable,
- character indexing is O(1) by UTF-16 code unit,
- Unicode is larger than simple ASCII,
- repeated string concatenation can become unnecessarily expensive.

For serious interview preparation, do not treat strings as a separate universe from arrays.

A useful mental model is:

```text
String
 |
 +-- traversal
 +-- frequency
 +-- palindrome
 +-- anagram
 +-- substring
 +-- subsequence
 +-- prefix/suffix
 +-- two pointers
 +-- sliding window
 +-- hashing
 +-- pattern matching
 +-- Trie
```

---

# 2. Prerequisites

You should already know:

- arrays,
- loops,
- `HashMap`,
- `HashSet`,
- sorting,
- two pointers,
- sliding window,
- basic Big-O analysis.

You should understand:

```text
substring != subsequence
```

and:

```text
character != byte
```

especially when Unicode is involved.

---

# 3. String Traversal

Basic traversal:

```java
String s = "hello";

for (int i = 0; i < s.length(); i++) {
    char c = s.charAt(i);
}
```

Enhanced traversal:

```java
for (char c : s.toCharArray()) {
    // process c
}
```

Prefer indexed traversal when:

- you need indices,
- you compare neighboring characters,
- you use two pointers,
- you want to avoid creating another character array.

---

# 4. Java String Fundamentals

## 4.1 Strings Are Immutable

Once created:

```java
String s = "hello";
```

you cannot modify a character inside the same `String` object.

This:

```java
s.charAt(0) = 'H';
```

does not compile.

Instead:

```java
s = "Hello";
```

creates/references another string value.

---

## 4.2 StringBuilder

Use `StringBuilder` when repeatedly constructing a string.

Bad for repeated concatenation:

```java
String result = "";

for (char c : chars) {
    result += c;
}
```

Repeated concatenation can cause repeated object creation.

Prefer:

```java
StringBuilder result = new StringBuilder();

for (char c : chars) {
    result.append(c);
}

return result.toString();
```

### Useful Methods

```java
append()
deleteCharAt()
insert()
reverse()
setCharAt()
toString()
```

---

# 5. Character Frequency

Frequency counting is one of the most reusable string patterns.

For lowercase English letters:

```java
int[] freq = new int[26];

for (int i = 0; i < s.length(); i++) {
    freq[s.charAt(i) - 'a']++;
}
```

This is preferable to a `HashMap` when the character domain is known and small.

### Complexity

```text
Time: O(n)
Space: O(1)
```

because `26` is constant.

---

# 6. Frequency With HashMap

For a broader character domain:

```java
Map<Character, Integer> freq = new HashMap<>();

for (int i = 0; i < s.length(); i++) {
    char c = s.charAt(i);
    freq.put(c, freq.getOrDefault(c, 0) + 1);
}
```

Use this when:

- the character set is not limited to lowercase English,
- you need a flexible mapping,
- the problem's alphabet is large or unspecified.

### Complexity

Expected:

```text
Time: O(n)
Space: O(k)
```

where `k` is the number of distinct characters.

---

# 7. ASCII Concepts

ASCII is a character encoding standard using 7-bit values.

The traditional ASCII character set has:

```text
128 values
0 through 127
```

Important ranges include:

```text
'A' to 'Z'
'a' to 'z'
'0' to '9'
```

For English lowercase letters:

```java
c - 'a'
```

maps:

```text
'a' -> 0
'b' -> 1
...
'z' -> 25
```

For uppercase:

```java
c - 'A'
```

maps:

```text
'A' -> 0
...
'Z' -> 25
```

---

# 8. ASCII vs Unicode

Do not assume:

```text
character = ASCII
```

Unicode represents characters from many writing systems.

Java's `char` is a UTF-16 code unit, not necessarily a complete Unicode code point.

For ordinary interview problems restricted to:

```text
'a'..'z'
```

or ASCII, `char` is usually sufficient.

For general Unicode-aware processing, use code points:

```java
int codePoint = s.codePointAt(index);
```

and iterate with:

```java
for (int i = 0; i < s.length(); ) {
    int cp = s.codePointAt(i);
    i += Character.charCount(cp);
}
```

### Interview Rule

Read the constraints.

If the problem says:

```text
lowercase English letters
```

use:

```java
int[26]
```

Do not over-engineer it with Unicode machinery.

---

# 9. Character Classification

Useful Java methods:

```java
Character.isLetter(c)
Character.isDigit(c)
Character.isWhitespace(c)
Character.isUpperCase(c)
Character.isLowerCase(c)
```

For ASCII-specific checks, direct comparisons can also be useful:

```java
if (c >= 'a' && c <= 'z') {
    // lowercase ASCII
}
```

---

# 10. String Manipulation

Common operations:

```java
s.length()
s.charAt(i)
s.substring(left, right)
s.indexOf(...)
s.lastIndexOf(...)
s.contains(...)
s.startsWith(...)
s.endsWith(...)
s.equals(...)
s.compareTo(...)
```

### Important

`substring(left, right)` uses:

```text
[left, right)
```

So `right` is excluded.

Example:

```java
"abcdef".substring(1, 4)
```

returns:

```text
"bcd"
```

---

# 11. String Equality

Do not use:

```java
s1 == s2
```

to compare string contents.

Use:

```java
s1.equals(s2)
```

`==` checks whether the references point to the same object.

`equals()` checks content equality.

This is a common Java interview trap.

---

# 12. Palindromes

A palindrome reads the same forward and backward.

Examples:

```text
"racecar"
"madam"
"abba"
```

Non-example:

```text
"hello"
```

---

# 13. Palindrome With Two Pointers

Use:

```text
left = 0
right = n - 1
```

Compare:

```text
s[left]
s[right]
```

If different:

```text
not a palindrome
```

Otherwise:

```text
left++
right--
```

### Java

```java
static boolean isPalindrome(String s) {
    int left = 0;
    int right = s.length() - 1;

    while (left < right) {
        if (s.charAt(left) != s.charAt(right)) {
            return false;
        }

        left++;
        right--;
    }

    return true;
}
```

### Complexity

```text
Time: O(n)
Space: O(1)
```

---

# 14. Palindrome Ignoring Case and Non-Alphanumeric Characters

A common interview variation is:

```text
"A man, a plan, a canal: Panama"
```

which should be considered a palindrome after normalization.

Do not necessarily build a new string.

Use two pointers and skip invalid characters.

### Java

```java
static boolean isValidPalindrome(String s) {
    int left = 0;
    int right = s.length() - 1;

    while (left < right) {
        while (left < right
                && !Character.isLetterOrDigit(s.charAt(left))) {
            left++;
        }

        while (left < right
                && !Character.isLetterOrDigit(s.charAt(right))) {
            right--;
        }

        char a = Character.toLowerCase(s.charAt(left));
        char b = Character.toLowerCase(s.charAt(right));

        if (a != b) {
            return false;
        }

        left++;
        right--;
    }

    return true;
}
```

---

# 15. Anagrams

Two strings are anagrams if they contain the same character frequencies.

Example:

```text
"listen"
"silent"
```

Both contain:

```text
e:1
i:1
l:1
n:1
s:1
t:1
```

---

# 16. Anagram Using Frequency

If lowercase English letters are guaranteed:

```java
static boolean isAnagram(String s, String t) {
    if (s.length() != t.length()) {
        return false;
    }

    int[] freq = new int[26];

    for (int i = 0; i < s.length(); i++) {
        freq[s.charAt(i) - 'a']++;
        freq[t.charAt(i) - 'a']--;
    }

    for (int count : freq) {
        if (count != 0) {
            return false;
        }
    }

    return true;
}
```

### Complexity

```text
Time: O(n)
Space: O(1)
```

---

# 17. Anagram Using Sorting

Another approach:

```text
sort characters of s
sort characters of t
compare
```

Complexity:

```text
O(n log n)
```

This is usually less efficient than frequency counting for a fixed alphabet.

But sorting is still useful when:

- you need canonical forms,
- you need to group anagrams,
- alphabet constraints are not convenient.

---

# 18. Substrings

A substring is contiguous.

For:

```text
s = "abcde"
```

these are substrings:

```text
"abc"
"bcd"
"de"
"abcde"
```

but:

```text
"ace"
```

is not a substring.

---

# 19. Number of Substrings

For a string of length `n`, the number of non-empty substrings is:

```text
n(n + 1) / 2
```

Why?

There are:

```text
n
```

substrings of length 1,

```text
n - 1
```

of length 2,

and so on:

```text
n + (n-1) + ... + 1
```

which equals:

```text
n(n+1)/2
```

This is O(n²) possible substrings.

Therefore problems asking you to explicitly enumerate all substrings cannot generally be solved in O(n) time if the output itself is quadratic.

---

# 20. Subsequences

A subsequence preserves relative order but does not require contiguity.

Example:

```text
s = "abcde"
```

Valid subsequences:

```text
"ace"
"bd"
"abcde"
""
```

`"aec"` is not a subsequence because order changed.

---

# 21. Number of Subsequences

For a string of length `n`, there are:

```text
2^n
```

subsequences including the empty subsequence.

Therefore enumerating all subsequences is exponential.

Typical techniques:

- recursion,
- backtracking,
- dynamic programming,
- greedy,
- two pointers for checking.

---

# 22. Checking Whether One String Is a Subsequence

Given:

```text
s = "abc"
t = "ahbgdc"
```

check whether `s` is a subsequence of `t`.

Use two pointers:

```text
i -> s
j -> t
```

When:

```text
s[i] == t[j]
```

advance `i`.

Always advance `j`.

### Java

```java
static boolean isSubsequence(String s, String t) {
    int i = 0;

    for (int j = 0; j < t.length() && i < s.length(); j++) {
        if (s.charAt(i) == t.charAt(j)) {
            i++;
        }
    }

    return i == s.length();
}
```

### Complexity

```text
Time: O(n)
Space: O(1)
```

---

# 23. Prefix and Suffix

A prefix starts at index `0`.

Examples for:

```text
"abcdef"
```

prefixes:

```text
""
"a"
"ab"
"abc"
"abcd"
"abcde"
"abcdef"
```

A suffix ends at index `n-1`.

Examples:

```text
""
"f"
"ef"
"def"
"cdef"
"bcdef"
"abcdef"
```

---

# 24. Proper Prefix and Proper Suffix

A proper prefix cannot be the entire string.

For:

```text
"abc"
```

proper prefixes:

```text
""
"a"
"ab"
```

Proper suffixes:

```text
""
"c"
"bc"
```

This terminology matters heavily in:

- KMP,
- Z algorithm,
- prefix-function problems.

---

# 25. Prefix Function / LPS

KMP uses an array commonly called:

```text
LPS
```

meaning:

```text
Longest Proper Prefix which is also Suffix
```

For:

```text
"ababaca"
```

the LPS array is:

```text
[0,0,1,2,3,0,1]
```

The LPS array tells us how far we can fall back after a mismatch without restarting from zero.

---

# 26. String Hashing

A string hash maps a string to an integer.

A simple conceptual polynomial hash is:

```text
hash(s) =
s[0] * B^(n-1)
+ s[1] * B^(n-2)
+ ...
+ s[n-1]
```

Usually calculated modulo some integer.

A rolling version lets us update a window efficiently.

---

# 27. Why Hash Strings?

Hashing can help with:

- comparing substrings,
- detecting repeated substrings,
- palindrome checks,
- pattern matching,
- duplicate-string detection.

Instead of comparing two length-`L` strings character by character:

```text
O(L)
```

we can often compare their hashes in:

```text
O(1)
```

after preprocessing.

### Important

Hash equality does **not** mathematically guarantee string equality unless collisions are impossible.

For practical algorithms, collision probability is reduced with:

- large moduli,
- two independent hashes,
- carefully chosen bases,
- sometimes 64-bit overflow hashing.

---

# 28. Rolling Hash

For a window of fixed length, a polynomial hash can be updated as the window moves.

Conceptually:

```text
ABC
BCD
CDE
```

Instead of recomputing the entire hash every time, remove the outgoing contribution and add the incoming contribution.

This is the central idea behind:

```text
Rabin-Karp
```

and many substring hashing techniques.

---

# 29. Basic Rolling Hash Formula

One common form is:

```text
H(s[0..n-1])
=
(s[0] * B^(n-1)
+ ...
+ s[n-1]) mod M
```

For a substring hash using prefix hashes, powers of `B` are precomputed.

A normalized substring hash can then be obtained in O(1) after O(n) preprocessing.

---

# 30. Prefix Hash

Let:

```text
prefix[i+1] = prefix[i] * B + value(s[i])
```

modulo `M`.

Then a substring hash can be derived from two prefix values.

Conceptually:

```text
hash(l, r)
=
prefix[r]
-
prefix[l] * B^(r-l)
```

modulo `M`.

The exact indexing depends on the implementation convention.

### Important

Always normalize modulo correctly:

```java
x %= mod;

if (x < 0) {
    x += mod;
}
```

---

# 31. Pattern Matching

Pattern matching asks whether a pattern occurs inside a text.

Example:

```text
text    = "ababcabc"
pattern = "abc"
```

The pattern occurs starting at indices:

```text
2
5
```

depending on zero-based indexing.

Major algorithms:

```text
Naive
KMP
Z algorithm
Rabin-Karp
```

---

# 32. Naive Pattern Matching

Compare the pattern against every possible starting position.

Worst case:

```text
O(nm)
```

where:

```text
n = text length
m = pattern length
```

It is simple and sometimes perfectly adequate.

Do not use an advanced algorithm merely because it exists.

---

# 33. Two Pointers on Strings

Two pointers are useful for:

- palindromes,
- comparing from both ends,
- removing/skipping characters,
- subsequence checking,
- partitioning.

Generic form:

```java
int left = 0;
int right = s.length() - 1;

while (left < right) {
    // process s.charAt(left)
    // process s.charAt(right)

    left++;
    right--;
}
```

---

# 34. Sliding Window on Strings

Sliding window is one of the most important string patterns.

Typical problems:

- longest substring without repeating characters,
- longest substring with at most K distinct characters,
- minimum window substring,
- permutation in string,
- find all anagrams.

The window usually maintains:

```text
[left, right]
```

plus some state:

```text
frequency array
HashMap
HashSet
distinct count
```

---

# 35. Fixed-Size String Window

Suppose we need every substring of length `k`.

Maintain a window:

```text
[left, right]
```

where:

```text
right - left + 1 = k
```

When `right` moves:

```text
add new character
remove old character
```

This gives O(n) scanning rather than rebuilding every substring.

---

# 36. Variable-Size String Window

Generic form:

```java
int left = 0;

for (int right = 0; right < s.length(); right++) {
    // add s.charAt(right)

    while (/* invalid */) {
        // remove s.charAt(left)
        left++;
    }

    // [left, right] is valid
}
```

The crucial question is:

> What state makes the current window valid?

---

# 37. Longest Substring Without Repeating Characters

Example:

```text
"abcabcbb"
```

Answer:

```text
3
```

from:

```text
"abc"
```

### Approach

Maintain the last index of each character.

When a repeated character occurs inside the current window:

```text
left
```

must jump beyond its previous occurrence.

### Java

```java
static int lengthOfLongestSubstring(String s) {
    int[] last = new int[128];
    Arrays.fill(last, -1);

    int left = 0;
    int best = 0;

    for (int right = 0; right < s.length(); right++) {
        char c = s.charAt(right);

        if (last[c] >= left) {
            left = last[c] + 1;
        }

        last[c] = right;

        best = Math.max(best, right - left + 1);
    }

    return best;
}
```

### Complexity

For ASCII:

```text
Time: O(n)
Space: O(1)
```

---

# 38. Why the Window Jump Is Safe

Suppose:

```text
s = "abcad"
```

At the second `a`:

```text
left = 0
previous a = 0
```

The current window:

```text
"abca"
```

is invalid.

Set:

```text
left = 1
```

Now:

```text
"bca"
```

contains no duplicate.

We do not need to test:

```text
left = 1, 2, 3...
```

one by one.

That is the optimization.

---

# 39. Minimum Window Substring

This is a classic hard sliding-window problem.

Given:

```text
s = "ADOBECODEBANC"
t = "ABC"
```

answer:

```text
"BANC"
```

### General Strategy

Maintain:

```text
need = required character frequencies
have = current window frequencies
```

Expand `right` until the window contains everything required.

Then shrink `left` while it remains valid.

The answer is the shortest valid window encountered.

### Key State

Track:

```text
required number of distinct characters
formed number currently satisfied
```

This avoids repeatedly checking the entire frequency table.

---

# 40. Anagrams Using Sliding Window

To find all anagrams of:

```text
p
```

inside:

```text
s
```

both must have the same character frequencies.

Since the anagram length is fixed:

```text
window size = p.length()
```

This becomes a fixed-size sliding window.

### Pattern

```text
build frequency of p

for every character in s:
    add character
    if window too large:
        remove left character

    if frequencies match:
        record window
```

This is an important bridge between:

```text
anagrams
```

and:

```text
sliding window
```

---

# 41. Pattern Matching: KMP

KMP stands for:

```text
Knuth-Morris-Pratt
```

It finds occurrences of a pattern in a text in:

```text
O(n + m)
```

where:

```text
n = text length
m = pattern length
```

---

# 42. Why KMP Is Needed

Naive matching may repeatedly compare the same characters.

Example:

```text
text    = aaaaaaaaaab
pattern = aaaab
```

A mismatch near the end does not mean all previous matching information is useless.

KMP stores that information using the LPS array.

---

# 43. KMP LPS Construction

For each pattern index `i`, LPS stores:

```text
length of the longest proper prefix
that is also a suffix of pattern[0..i]
```

Example:

```text
pattern = "ababaca"
```

LPS:

```text
[0,0,1,2,3,0,1]
```

---

# 44. KMP Search

Maintain:

```text
i = text index
j = pattern index
```

If:

```text
text[i] == pattern[j]
```

advance both.

If mismatch:

```text
j = lps[j - 1]
```

instead of resetting `j` to zero.

Only when:

```text
j == 0
```

do we move the text pointer after a mismatch.

### Java

```java
static int kmpSearch(String text, String pattern) {
    if (pattern.isEmpty()) {
        return 0;
    }

    int[] lps = buildLps(pattern);

    int i = 0;
    int j = 0;

    while (i < text.length()) {
        if (text.charAt(i) == pattern.charAt(j)) {
            i++;
            j++;

            if (j == pattern.length()) {
                return i - j;
            }
        } else if (j > 0) {
            j = lps[j - 1];
        } else {
            i++;
        }
    }

    return -1;
}

static int[] buildLps(String pattern) {
    int[] lps = new int[pattern.length()];

    int length = 0;
    int i = 1;

    while (i < pattern.length()) {
        if (pattern.charAt(i) == pattern.charAt(length)) {
            lps[i] = ++length;
            i++;
        } else if (length > 0) {
            length = lps[length - 1];
        } else {
            lps[i] = 0;
            i++;
        }
    }

    return lps;
}
```

### Complexity

```text
LPS construction: O(m)
Search:            O(n)
Total:             O(n + m)
Space:             O(m)
```

---

# 45. KMP Recognition

Think KMP when:

- exact pattern matching matters,
- worst-case O(n + m) is required,
- the pattern contains repeated prefixes/suffixes,
- you need to search the same pattern efficiently.

For ordinary Java coding interviews, know the idea and implementation, but do not force KMP into every substring problem.

---

# 46. Z Algorithm

The Z algorithm constructs a Z array.

For each position `i`:

```text
Z[i] =
length of the longest substring starting at i
that matches the prefix of the entire string
```

Example:

```text
S = "aabxaab"
```

The Z array captures prefix matches at every position.

---

# 47. Pattern Matching With Z Algorithm

To search:

```text
pattern in text
```

construct:

```text
pattern + separator + text
```

For example:

```text
pattern = "abc"
text = "xabcabc"
```

construct:

```text
"abc#xabcabc"
```

Choose a separator not occurring in either string.

Whenever:

```text
Z[i] == pattern.length()
```

the pattern occurs there.

### Complexity

```text
Time: O(n + m)
Space: O(n + m)
```

---

# 48. Z Algorithm Core

Maintain a Z-box:

```text
[L, R]
```

where:

```text
s[L..R]
```

matches the prefix.

For a new index `i` inside the box:

```text
Z[i] >= min(Z[i-L], R-i+1)
```

Then extend beyond `R` when possible.

This is what makes the algorithm linear.

---

# 49. Rabin-Karp

Rabin-Karp uses hashing for pattern matching.

Compute:

```text
hash(pattern)
```

and rolling hashes of text windows of the same length.

If:

```text
windowHash == patternHash
```

then verify characters if collision safety is required.

### Complexity

Expected:

```text
O(n + m)
```

Typical worst case with collisions:

```text
O(nm)
```

### Strength

It is especially useful when:

- searching for multiple patterns,
- hashing is already being used,
- comparing fixed-length windows.

---

# 50. Rabin-Karp Rolling Update

For a base `B` and modulus `M`, a fixed-length hash can be updated by:

1. removing the outgoing character,
2. shifting the remaining hash,
3. adding the incoming character.

Conceptually:

```text
newHash =
(oldHash - outgoingContribution)
* B
+ incoming
```

then reduce modulo `M`.

The exact formula depends on the chosen polynomial convention.

---

# 51. KMP vs Z vs Rabin-Karp

| Algorithm | Main idea | Typical time | Extra space |
|---|---|---:|---:|
| Naive | direct comparison | O(nm) | O(1) |
| KMP | prefix/LPS fallback | O(n+m) | O(m) |
| Z | prefix matches everywhere | O(n+m) | O(n+m) |
| Rabin-Karp | rolling hash | expected O(n+m) | O(1) to O(m) depending on implementation |

### Practical Interview Rule

Learn in this order:

```text
Naive
→ KMP
→ Z
→ Rabin-Karp
→ rolling hash applications
```

KMP is the most important exact linear-time pattern-matching algorithm to know deeply.

---

# 52. Trie

A Trie is a prefix tree.

It is useful when the problem involves:

```text
words
prefixes
dictionary lookup
autocomplete
word search
prefix constraints
```

Example words:

```text
cat
car
care
```

share the prefix:

```text
ca
```

A Trie stores that shared structure.

---

# 53. Trie Node

Basic lowercase-English Trie node:

```java
static class TrieNode {
    TrieNode[] children = new TrieNode[26];
    boolean isWord;
}
```

---

# 54. Trie Insert

```java
static void insert(TrieNode root, String word) {
    TrieNode node = root;

    for (int i = 0; i < word.length(); i++) {
        int index = word.charAt(i) - 'a';

        if (node.children[index] == null) {
            node.children[index] = new TrieNode();
        }

        node = node.children[index];
    }

    node.isWord = true;
}
```

---

# 55. Trie Search

```java
static boolean search(TrieNode root, String word) {
    TrieNode node = root;

    for (int i = 0; i < word.length(); i++) {
        int index = word.charAt(i) - 'a';

        if (node.children[index] == null) {
            return false;
        }

        node = node.children[index];
    }

    return node.isWord;
}
```

---

# 56. Trie Prefix Search

```java
static boolean startsWith(TrieNode root, String prefix) {
    TrieNode node = root;

    for (int i = 0; i < prefix.length(); i++) {
        int index = prefix.charAt(i) - 'a';

        if (node.children[index] == null) {
            return false;
        }

        node = node.children[index];
    }

    return true;
}
```

### Complexity

For word/prefix length `L`:

```text
Insert:     O(L)
Search:     O(L)
StartsWith: O(L)
```

Space depends on the number of nodes and alphabet representation.

---

# 57. Trie Recognition

Think Trie when you see:

```text
many words
prefix
starts with
dictionary
autocomplete
word lookup
replace words using prefixes
```

Do not use a Trie just because the problem contains strings.

If the task is simply:

```text
count character frequencies
```

a Trie is unnecessary.

---

# 58. Rolling Hash in More Detail

A common polynomial rolling hash is:

```text
H = (((c1 * B + c2) * B + c3) * B + ...)
```

modulo `M`.

For a substring:

```text
s[l..r]
```

we want to calculate its hash without scanning all characters.

Precompute:

```text
prefixHash
power
```

where:

```text
power[i] = B^i mod M
```

Then substring hashes can be extracted in O(1).

---

# 59. Double Hashing

One hash can theoretically collide.

A stronger practical approach uses:

```text
hash1 modulo M1
hash2 modulo M2
```

A substring is considered equal only when both hashes match.

This drastically reduces practical collision probability.

For interviews, understand the principle rather than memorizing arbitrary constants.

---

# 60. Rolling Hash vs KMP

### KMP

Deterministic pattern matching.

```text
O(n + m)
```

with guaranteed worst-case complexity.

### Rolling Hash

Probabilistic equality/matching.

Very useful for:

- substring equality,
- repeated substrings,
- palindrome comparison,
- duplicate detection,
- binary-search + hashing problems.

Choose based on the task.

---

# 61. String Hashing for Palindromes

A useful advanced technique is to compare:

```text
forward hash
```

against:

```text
reverse hash
```

for a substring.

If the normalized hashes match, the substring is very likely a palindrome.

This enables problems such as:

```text
longest palindromic substring
```

to be attacked with hashing, although Manacher's algorithm is the deterministic linear-time advanced solution for that specific problem.

---

# 62. String Problem Decision Framework

When you receive a string problem:

## Step 1 — What is being asked?

```text
frequency?
substring?
subsequence?
prefix?
pattern occurrence?
palindrome?
anagram?
minimum/maximum window?
```

## Step 2 — Is the character set constrained?

If:

```text
lowercase English
```

use:

```text
int[26]
```

If arbitrary:

```text
HashMap
```

may be appropriate.

## Step 3 — Is the problem about both ends?

Think:

```text
two pointers
```

## Step 4 — Is it a contiguous window?

Think:

```text
sliding window
```

## Step 5 — Is the window fixed?

Think:

```text
fixed-size sliding window
```

## Step 6 — Does the window size vary?

Think:

```text
variable-size sliding window
```

## Step 7 — Exact pattern matching?

Think:

```text
KMP
Z
Rabin-Karp
```

## Step 8 — Prefix dictionary?

Think:

```text
Trie
```

## Step 9 — Substring equality/repetition?

Think:

```text
rolling hash
```

---

# 63. Important String Patterns

## Pattern 1 — Frequency Array

Signal:

```text
lowercase English letters
anagram
character count
frequency
```

Use:

```java
int[26]
```

---

## Pattern 2 — Two Pointers

Signal:

```text
palindrome
compare ends
subsequence
```

Use:

```text
left/right
```

---

## Pattern 3 — Sliding Window

Signal:

```text
longest substring
shortest substring
at most K
exactly K with appropriate transformation
all anagrams
permutation in string
```

Use:

```text
left/right + frequency state
```

---

## Pattern 4 — Prefix + HashMap

This is more common with numeric arrays, but analogous prefix-state ideas can appear in strings.

Look for a cumulative state where:

```text
currentState - previousState
```

represents the desired substring property.

---

## Pattern 5 — KMP

Signal:

```text
exact pattern matching
linear worst-case requirement
```

---

## Pattern 6 — Z Algorithm

Signal:

```text
prefix matching at every position
pattern search
string borders
```

---

## Pattern 7 — Rabin-Karp / Rolling Hash

Signal:

```text
substring equality
duplicate substrings
many fixed-length comparisons
hash-based matching
```

---

## Pattern 8 — Trie

Signal:

```text
prefix
dictionary
many words
startsWith
autocomplete
```

---

# 64. Common Mistakes

## Mistake 1: Using `==` for String Content

Wrong:

```java
if (a == b)
```

Correct:

```java
if (a.equals(b))
```

---

## Mistake 2: Confusing Substring and Subsequence

Substring:

```text
contiguous
```

Subsequence:

```text
order preserved
not necessarily contiguous
```

---

## Mistake 3: Assuming `char` Means Unicode Character

Java `char` is a UTF-16 code unit.

For general Unicode code points, use code-point APIs.

---

## Mistake 4: Rebuilding Every Window

If the window moves by one position, update its state incrementally.

Do not repeatedly construct:

```java
s.substring(left, right)
```

inside a large loop unless required.

---

## Mistake 5: Using HashMap When Alphabet Is Tiny

For:

```text
'a'..'z'
```

an `int[26]` is usually cleaner and faster.

---

## Mistake 6: Ignoring Hash Collisions

A hash match is not a mathematical proof of equality.

Use:

- direct verification,
- double hashing,
- or a deterministic algorithm where required.

---

## Mistake 7: Mishandling Sliding-Window Shrinking

When removing a character:

```java
freq[s.charAt(left)]--;
left++;
```

Make sure the condition becomes valid before recording the answer.

---

## Mistake 8: Forgetting Empty Strings

Always check what the problem defines for:

```text
""
```

especially:

- substring search,
- palindrome,
- prefix,
- KMP,
- Trie operations.

---

## Mistake 9: Creating Excessive Temporary Strings

Repeated:

```java
substring()
+
```

or concatenation can create unnecessary allocations.

Use indices and `StringBuilder` where appropriate.

---

# 65. Complexity Patterns

| Technique | Time | Extra Space |
|---|---:|---:|
| String traversal | O(n) | O(1) |
| Frequency array | O(n) | O(1) for fixed alphabet |
| HashMap frequency | O(n) expected | O(k) |
| Palindrome two pointers | O(n) | O(1) |
| Anagram frequency | O(n) | O(1) for fixed alphabet |
| Anagram sorting | O(n log n) | depends |
| Subsequence check | O(n) | O(1) |
| Fixed sliding window | O(n) | O(k) or O(1) fixed alphabet |
| Variable sliding window | O(n) expected | O(k) |
| Naive pattern matching | O(nm) | O(1) |
| KMP | O(n+m) | O(m) |
| Z algorithm | O(n+m) | O(n+m) |
| Rabin-Karp | expected O(n+m) | O(1) to O(m) |
| Trie insert/search | O(L) | O(total trie nodes) |
| Rolling hash preprocessing | O(n) | O(n) |
| Rolling hash substring query | O(1) | O(1) after preprocessing |

---

# 66. Easy → Medium → Hard Progression

## Easy

Master:

1. string traversal
2. character frequency
3. valid palindrome
4. anagram check
5. subsequence check
6. string reversal
7. first unique character
8. basic string manipulation
9. common prefix

## Medium

Then master:

1. longest substring without repeating characters
2. permutation in string
3. find all anagrams
4. group anagrams
5. longest palindromic substring
6. string compression
7. minimum window-style problems
8. KMP
9. Trie
10. rolling hash basics

## Hard

Then:

1. Minimum Window Substring
2. Word Search II
3. Palindrome Pairs
4. advanced Trie problems
5. Z algorithm applications
6. Rabin-Karp applications
7. rolling hash + binary search
8. advanced palindrome hashing
9. repeated substring problems

Medium should dominate interview practice.

---

# 67. Selected LeetCode Problems

The following problems are selected to cover the major string patterns without turning the file into an arbitrary list.

---

## 67.1 Valid Palindrome

- Platform: LeetCode
- Difficulty: Easy
- Topic: Strings
- Pattern: two pointers
- Priority: CORE
- Free: Yes

#### Official Link

https://leetcode.com/problems/valid-palindrome/

#### Why This Problem Matters

It teaches the fundamental string two-pointer pattern.

#### Hint 1 — Observation

Compare characters from both ends.

#### Hint 2 — Direction

Skip characters that are not letters or digits.

#### Hint 3 — Pattern / Data Structure

Two pointers.

#### Hint 4 — Algorithm

```text
left = 0
right = n - 1

skip invalid characters
compare normalized characters
move inward
```

#### Java Solution

```java
class Solution {
    public boolean isPalindrome(String s) {
        int left = 0;
        int right = s.length() - 1;

        while (left < right) {
            while (left < right
                    && !Character.isLetterOrDigit(s.charAt(left))) {
                left++;
            }

            while (left < right
                    && !Character.isLetterOrDigit(s.charAt(right))) {
                right--;
            }

            char a = Character.toLowerCase(s.charAt(left));
            char b = Character.toLowerCase(s.charAt(right));

            if (a != b) {
                return false;
            }

            left++;
            right--;
        }

        return true;
    }
}
```

#### Complexity

- Time: O(n)
- Space: O(1)

#### Common Mistakes

- forgetting case normalization,
- failing to skip punctuation,
- using extra strings unnecessarily.

#### Mastery Check

Can you solve it without constructing a cleaned copy of the string?

---

## 67.2 Valid Anagram

- Platform: LeetCode
- Difficulty: Easy
- Topic: Strings
- Pattern: frequency counting
- Priority: CORE
- Free: Yes

#### Official Link

https://leetcode.com/problems/valid-anagram/

#### Java Solution

```java
class Solution {
    public boolean isAnagram(String s, String t) {
        if (s.length() != t.length()) {
            return false;
        }

        int[] freq = new int[26];

        for (int i = 0; i < s.length(); i++) {
            freq[s.charAt(i) - 'a']++;
            freq[t.charAt(i) - 'a']--;
        }

        for (int x : freq) {
            if (x != 0) {
                return false;
            }
        }

        return true;
    }
}
```

#### Complexity

- Time: O(n)
- Space: O(1)

#### Important Constraint

This implementation assumes lowercase English letters.

If the problem allows arbitrary characters, adapt the data structure.

#### Mastery Check

Why can one frequency array track both strings?

---

## 67.3 Longest Common Prefix

- Platform: LeetCode
- Difficulty: Easy
- Topic: Strings
- Pattern: prefix comparison
- Priority: IMPORTANT
- Free: Yes

#### Official Link

https://leetcode.com/problems/longest-common-prefix/

#### Why This Problem Matters

It teaches direct prefix reasoning.

#### Java Solution

```java
class Solution {
    public String longestCommonPrefix(String[] strs) {
        if (strs.length == 0) {
            return "";
        }

        String prefix = strs[0];

        for (int i = 1; i < strs.length; i++) {
            while (!strs[i].startsWith(prefix)) {
                prefix = prefix.substring(0, prefix.length() - 1);

                if (prefix.isEmpty()) {
                    return "";
                }
            }
        }

        return prefix;
    }
}
```

#### Complexity

Worst case depends on total input characters; the key idea is repeated prefix reduction.

#### Common Mistakes

- indexing beyond a shorter string,
- assuming all strings have equal length,
- forgetting the empty-prefix case.

#### Mastery Check

Can you solve it using character-by-character column scanning?

---

## 67.4 Longest Substring Without Repeating Characters

- Platform: LeetCode
- Difficulty: Medium
- Topic: Strings
- Pattern: variable sliding window
- Priority: CORE
- Free: Yes

#### Official Link

https://leetcode.com/problems/longest-substring-without-repeating-characters/

#### Why This Problem Matters

This is one of the most important sliding-window problems.

#### Hint 1 — Observation

The current window must contain unique characters.

#### Hint 2 — Direction

When a duplicate enters, move the left boundary past the previous occurrence.

#### Hint 3 — Pattern / Data Structure

Sliding window + last occurrence.

#### Hint 4 — Algorithm

Maintain:

```text
last[c] = latest index of c
```

When:

```text
last[c] >= left
```

jump:

```text
left = last[c] + 1
```

#### Java Solution

```java
class Solution {
    public int lengthOfLongestSubstring(String s) {
        int[] last = new int[128];
        Arrays.fill(last, -1);

        int left = 0;
        int best = 0;

        for (int right = 0; right < s.length(); right++) {
            char c = s.charAt(right);

            if (last[c] >= left) {
                left = last[c] + 1;
            }

            last[c] = right;

            best = Math.max(best, right - left + 1);
        }

        return best;
    }
}
```

#### Complexity

- Time: O(n)
- Space: O(1) for ASCII

#### Common Mistakes

- setting `left = last[c] + 1` without checking whether the previous occurrence is inside the current window,
- using a `Set` and shrinking one character at a time when a direct jump is possible.

#### Mastery Check

Explain why `last[c] >= left` is necessary.

---

## 67.5 Group Anagrams

- Platform: LeetCode
- Difficulty: Medium
- Topic: Strings / Hashing
- Pattern: canonical representation
- Priority: CORE
- Free: Yes

#### Official Link

https://leetcode.com/problems/group-anagrams/

#### Why This Problem Matters

It teaches how to convert an object into a key representing its structural identity.

#### Common Key Options

Sorted characters:

```text
"eat" -> "aet"
"tea" -> "aet"
```

or frequency encoding.

### Java — Sorted Key

```java
class Solution {
    public List<List<String>> groupAnagrams(String[] strs) {
        Map<String, List<String>> groups = new HashMap<>();

        for (String s : strs) {
            char[] chars = s.toCharArray();
            Arrays.sort(chars);

            String key = new String(chars);

            groups
                .computeIfAbsent(key, k -> new ArrayList<>())
                .add(s);
        }

        return new ArrayList<>(groups.values());
    }
}
```

#### Complexity

If average string length is `k`:

```text
Time: O(n * k log k)
Space: O(nk)
```

where `n` is the number of strings.

#### Follow-Up

For lowercase English letters, use a frequency signature to avoid sorting each string.

#### Mastery Check

Why does the sorted representation uniquely identify an anagram class?

---

## 67.6 Find All Anagrams in a String

- Platform: LeetCode
- Difficulty: Medium
- Topic: Strings
- Pattern: fixed-size sliding window + frequency
- Priority: CORE
- Free: Yes

#### Official Link

https://leetcode.com/problems/find-all-anagrams-in-a-string/

#### Java

```java
class Solution {
    public List<Integer> findAnagrams(String s, String p) {
        List<Integer> result = new ArrayList<>();

        if (p.length() > s.length()) {
            return result;
        }

        int[] need = new int[26];
        int[] window = new int[26];

        for (char c : p.toCharArray()) {
            need[c - 'a']++;
        }

        int k = p.length();

        for (int right = 0; right < s.length(); right++) {
            window[s.charAt(right) - 'a']++;

            if (right >= k) {
                window[s.charAt(right - k) - 'a']--;
            }

            if (right >= k - 1 && Arrays.equals(need, window)) {
                result.add(right - k + 1);
            }
        }

        return result;
    }
}
```

The above `Arrays.equals` costs O(26), which is constant for lowercase English letters.

A more general optimized implementation tracks how many frequency requirements are satisfied.

#### Complexity

For a fixed 26-letter alphabet:

```text
Time: O(n)
Space: O(1)
```

#### Mastery Check

Why is the window size exactly `p.length()`?

---

## 67.7 Permutation in String

- Platform: LeetCode
- Difficulty: Medium
- Topic: Strings
- Pattern: fixed sliding window + frequency
- Priority: CORE
- Free: Yes

#### Official Link

https://leetcode.com/problems/permutation-in-string/

#### Key Idea

A permutation has exactly the same character frequencies.

Therefore:

```text
window length = s1.length()
```

and compare the window's frequency vector against `s1`.

This is essentially the boolean version of Find All Anagrams.

#### Mastery Check

Can you explain why the problem is a fixed-size window rather than a variable-size window?

---

## 67.8 Minimum Window Substring

- Platform: LeetCode
- Difficulty: Hard
- Topic: Strings
- Pattern: variable sliding window
- Priority: CORE
- Free: Yes

#### Official Link

https://leetcode.com/problems/minimum-window-substring/

#### Why This Problem Matters

It is the canonical hard variable-window problem.

#### Hint 1 — Observation

The answer must be a contiguous window containing every required character with required multiplicity.

#### Hint 2 — Direction

Expand right until the window becomes valid.

#### Hint 3 — Pattern / Data Structure

Frequency map + formed/required counts.

#### Hint 4 — Algorithm

1. Count required characters.
2. Expand `right`.
3. When a character requirement becomes satisfied, increment `formed`.
4. While `formed == required`, shrink from the left.
5. Record the smallest valid window.

#### Java Solution

```java
class Solution {
    public String minWindow(String s, String t) {
        if (s.length() < t.length() || t.isEmpty()) {
            return "";
        }

        Map<Character, Integer> need = new HashMap<>();

        for (char c : t.toCharArray()) {
            need.put(c, need.getOrDefault(c, 0) + 1);
        }

        Map<Character, Integer> window = new HashMap<>();

        int required = need.size();
        int formed = 0;

        int left = 0;

        int bestLength = Integer.MAX_VALUE;
        int bestLeft = 0;

        for (int right = 0; right < s.length(); right++) {
            char c = s.charAt(right);

            window.put(c, window.getOrDefault(c, 0) + 1);

            if (need.containsKey(c)
                    && window.get(c).intValue() == need.get(c).intValue()) {
                formed++;
            }

            while (formed == required) {
                int length = right - left + 1;

                if (length < bestLength) {
                    bestLength = length;
                    bestLeft = left;
                }

                char removed = s.charAt(left);

                window.put(
                        removed,
                        window.get(removed) - 1
                );

                if (need.containsKey(removed)
                        && window.get(removed) < need.get(removed)) {
                    formed--;
                }

                left++;
            }
        }

        return bestLength == Integer.MAX_VALUE
                ? ""
                : s.substring(bestLeft, bestLeft + bestLength);
    }
}
```

#### Complexity

```text
Time: O(n)
Space: O(k)
```

where `k` is the number of distinct characters in `t`.

#### Common Mistakes

- checking only whether a character exists rather than required multiplicity,
- shrinking before the window is valid,
- forgetting to decrement `formed`,
- returning the first valid window instead of the minimum.

#### Mastery Check

Explain exactly when `formed` increases and decreases.

---

## 67.9 Implement strStr / Find the Index of the First Occurrence

- Platform: LeetCode
- Difficulty: Easy
- Topic: Strings
- Pattern: pattern matching
- Priority: IMPORTANT
- Free: Yes

#### Official Link

https://leetcode.com/problems/find-the-index-of-the-first-occurrence-in-a-string/

#### Core Learning

Start with naive matching:

```text
O(nm)
```

Then implement KMP:

```text
O(n+m)
```

The problem is useful because it provides a direct path from basic string traversal to advanced pattern matching.

#### Mastery Check

Implement KMP without looking at your previous code.

---

## 67.10 Implement Trie (Prefix Tree)

- Platform: LeetCode
- Difficulty: Medium
- Topic: Strings
- Pattern: Trie
- Priority: CORE
- Free: Yes

#### Official Link

https://leetcode.com/problems/implement-trie-prefix-tree/

#### Java

```java
class Trie {
    static class Node {
        Node[] children = new Node[26];
        boolean isWord;
    }

    private final Node root = new Node();

    public void insert(String word) {
        Node node = root;

        for (char c : word.toCharArray()) {
            int index = c - 'a';

            if (node.children[index] == null) {
                node.children[index] = new Node();
            }

            node = node.children[index];
        }

        node.isWord = true;
    }

    public boolean search(String word) {
        Node node = findNode(word);
        return node != null && node.isWord;
    }

    public boolean startsWith(String prefix) {
        return findNode(prefix) != null;
    }

    private Node findNode(String s) {
        Node node = root;

        for (char c : s.toCharArray()) {
            int index = c - 'a';

            if (node.children[index] == null) {
                return null;
            }

            node = node.children[index];
        }

        return node;
    }
}
```

#### Complexity

For word length `L`:

```text
insert: O(L)
search: O(L)
prefix: O(L)
```

#### Mastery Check

Explain why a Trie makes prefix lookup proportional to prefix length rather than the number of stored words.

---

## 67.11 Longest Palindromic Substring

- Platform: LeetCode
- Difficulty: Medium
- Topic: Strings
- Pattern: expand around center
- Priority: CORE
- Free: Yes

#### Official Link

https://leetcode.com/problems/longest-palindromic-substring/

#### Key Idea

Every palindrome has a center.

The center can be:

```text
one character
```

for odd length:

```text
aba
```

or:

```text
between two characters
```

for even length:

```text
abba
```

Expand from every possible center.

### Java

```java
class Solution {
    public String longestPalindrome(String s) {
        if (s.length() < 2) {
            return s;
        }

        int bestLeft = 0;
        int bestRight = 0;

        for (int center = 0; center < s.length(); center++) {
            int len1 = expand(s, center, center);
            int len2 = expand(s, center, center + 1);

            int len = Math.max(len1, len2);

            if (len > bestRight - bestLeft + 1) {
                bestLeft = center - (len - 1) / 2;
                bestRight = center + len / 2;
            }
        }

        return s.substring(bestLeft, bestRight + 1);
    }

    private int expand(String s, int left, int right) {
        while (left >= 0
                && right < s.length()
                && s.charAt(left) == s.charAt(right)) {
            left--;
            right++;
        }

        return right - left - 1;
    }
}
```

#### Complexity

```text
Time: O(n²)
Space: O(1)
```

#### Follow-Up

Know that Manacher's algorithm solves longest palindromic substring in O(n), but it is an advanced algorithm and not necessary before mastering center expansion.

#### Mastery Check

Why are there two center types?

---

## 67.12 Word Search II

- Platform: LeetCode
- Difficulty: Hard
- Topic: Strings / Trie / Backtracking
- Pattern: Trie + DFS
- Priority: ADVANCED
- Free: Yes

#### Official Link

https://leetcode.com/problems/word-search-ii/

#### Why This Problem Matters

It combines:

```text
Trie
+
grid traversal
+
DFS/backtracking
```

This is an important advanced string problem because it shows how a Trie can prune impossible word searches.

#### Mastery Check

Explain why checking every word independently wastes work and how the Trie shares prefixes.

---

# 68. Fully Explained Problems

## Problem 1 — Longest Substring Without Repeating Characters

### Question

Find the length of the longest substring without duplicate characters.

Example:

```text
s = "abcabcbb"
```

Answer:

```text
3
```

### Brute Force

Generate every substring and check whether its characters are unique.

There are O(n²) substrings, and checking each may cost O(n).

Worst case:

```text
O(n³)
```

Too slow.

### Better Direction

Use a sliding window.

Maintain:

```text
[left, right]
```

such that the window contains no duplicates.

When `right` introduces a duplicate:

```text
left
```

must move past the previous occurrence.

### Why Store Last Index?

Instead of:

```text
remove one character at a time
```

we can jump directly:

```text
left = last[c] + 1
```

provided the previous occurrence is inside the current window.

### Invariant

At every iteration:

```text
s[left..right]
```

contains no duplicate characters.

### Complexity

```text
Time: O(n)
Space: O(k)
```

where `k` is the character set size.

---

## Problem 2 — Minimum Window Substring

### Question

Find the minimum substring of `s` containing every character of `t` with the required multiplicity.

Example:

```text
s = "ADOBECODEBANC"
t = "ABC"
```

Answer:

```text
"BANC"
```

### Why Brute Force Fails

There are O(n²) substrings.

Checking every substring against `t` makes the solution too slow.

### Window State

We need:

```text
need[c]    = required count
window[c]  = current count
```

We also maintain:

```text
required = number of distinct required characters
formed    = number of distinct requirements currently satisfied
```

### Expand

Move `right`.

When:

```text
window[c] == need[c]
```

that requirement becomes satisfied.

So:

```text
formed++
```

### Shrink

When:

```text
formed == required
```

the window is valid.

Now move `left` forward to minimize it.

If removing a character causes:

```text
window[c] < need[c]
```

the window becomes invalid:

```text
formed--
```

### Invariant

Whenever:

```text
formed == required
```

the current window contains all required characters with sufficient multiplicity.

### Complexity

Each character enters and leaves the window at most once:

```text
Time: O(n)
Space: O(k)
```

---

## Problem 3 — KMP Pattern Matching

### Question

Find the first occurrence of:

```text
pattern
```

inside:

```text
text
```

in O(n + m).

### Why Naive Matching Repeats Work

Suppose the pattern has a repeated prefix.

After a mismatch, some of the characters already matched can still be useful.

KMP captures that information with:

```text
LPS
```

### LPS Meaning

For every pattern position:

```text
LPS[i]
```

is the length of the longest proper prefix of:

```text
pattern[0..i]
```

that is also a suffix.

### Search Transition

On mismatch:

```text
j = lps[j - 1]
```

The text index does not need to move backward.

### Why This Is Linear

The pattern pointer can fall back using already computed information rather than repeatedly comparing the same characters.

The total search is:

```text
O(n)
```

after:

```text
O(m)
```

LPS preprocessing.

Therefore:

```text
O(n + m)
```

overall.

---

# 69. 3 Fully Explained & Solved GATE PYQs

The following are selected GATE CSE previous-year questions that test string algorithms/concepts closely related to this topic.

---

## PYQ 1 — GATE CSE 2003, Question 50

### Question

Consider a pattern-matching problem involving strings and the use of a finite automaton. The question asks about the number of states/transition behavior needed for recognizing occurrences of a pattern.

### What Is Being Tested

- pattern matching,
- finite automata,
- string preprocessing,
- relationship between a pattern and its prefixes.

### Core Idea

Finite-automaton string matching preprocesses the pattern into states representing:

```text
length of the prefix of the pattern
that matches the current suffix of the processed text
```

This is conceptually related to the failure/fallback information used by KMP.

### Step-by-Step Reasoning

For a pattern of length:

```text
m
```

the automaton states represent:

```text
0, 1, 2, ..., m
```

where state `q` means:

```text
q characters of the pattern currently match
```

After reading a new character, the transition determines the longest pattern prefix that is still a suffix of the resulting text.

This is why prefix/suffix relationships are fundamental to efficient pattern matching.

### Generalizable Lesson

If a string problem repeatedly asks:

```text
how much of the pattern has already matched?
```

think about:

```text
prefix-suffix structure
```

which leads naturally to:

```text
KMP / finite automaton matching
```

Source: GATE CSE 2003 string/pattern-matching question. citeturn2search0

---

## PYQ 2 — GATE CSE 2014, Question 37

### Question

A string-processing problem asks about matching prefixes/suffixes of a pattern and determining the behavior of a string-matching algorithm after a mismatch.

### What Is Being Tested

- prefix/suffix,
- pattern matching,
- KMP-style fallback,
- preprocessing.

### Approach

The key question after a mismatch is:

> What is the longest proper prefix of the already matched portion that is also a suffix?

That value tells us where the pattern pointer can safely continue.

For a pattern segment:

```text
P[0..j]
```

the fallback is:

```text
LPS[j - 1]
```

rather than:

```text
0
```

### Why This Matters

Suppose:

```text
pattern = "abab"
```

After matching:

```text
"ab"
```

the prefix:

```text
"ab"
```

is also a suffix of the matched region.

Therefore some matching information survives a mismatch.

### Generalizable Lesson

Do not think of KMP as a magic algorithm.

Its entire purpose is:

```text
reuse prefix/suffix information after mismatch
```

Source: GATE CSE 2014 pattern-matching question. citeturn2search1

---

## PYQ 3 — GATE CSE 2021, String/Pattern Matching

### Question

A GATE question involving a string matching algorithm asks about the number of comparisons or behavior of a pattern-matching procedure under a particular text/pattern arrangement.

### What Is Being Tested

- worst-case pattern matching,
- repeated comparisons,
- algorithmic complexity,
- distinction between naive matching and optimized matching.

### Approach

The naive algorithm may compare the same pattern characters repeatedly.

For:

```text
text length = n
pattern length = m
```

its worst-case behavior can reach:

```text
O(nm)
```

An algorithm such as KMP avoids these repeated comparisons by using prefix-function information.

### Key Lesson

When analyzing string matching, distinguish:

```text
naive worst case: O(nm)
```

from:

```text
KMP: O(n+m)
```

and:

```text
Rabin-Karp: expected O(n+m), worst-case O(nm)
```

### Generalizable Lesson

GATE questions frequently test the actual worst-case complexity rather than the average behavior you may observe on random strings.

Source: GATE CSE string matching question. citeturn2search2

---

# 70. String Pattern Comparison

| Problem signal | Best first pattern |
|---|---|
| Same character frequencies | frequency array / HashMap |
| Same characters in different order | anagram |
| Same from both ends | two pointers |
| Longest contiguous section | sliding window |
| Fixed-length matching | fixed window |
| Minimum valid contiguous section | variable sliding window |
| Exact pattern search | KMP / Z / Rabin-Karp |
| Prefix dictionary | Trie |
| Many substring equality checks | rolling hash |
| Prefix/suffix overlap | KMP / Z |
| Subsequence | two pointers |
| Palindrome | two pointers / center expansion |
| Longest palindrome | center expansion / advanced Manacher |
| Multiple dictionary words | Trie |

---

# 71. KMP vs Z Algorithm — Mental Model

## KMP

Focuses on:

```text
the pattern
```

and computes:

```text
LPS / prefix function
```

to determine how far the pattern can fall back.

Think:

```text
"What part of the pattern can I reuse?"
```

## Z Algorithm

Focuses on:

```text
the entire string
```

and computes:

```text
how much of the prefix matches at every position
```

Think:

```text
"How much does the string starting here match the prefix?"
```

Both exploit prefix matching, but their representations differ.

---

# 72. String Hashing — Interview-Level Implementation

A simple 64-bit rolling hash can be implemented using Java `long` overflow.

One conceptual implementation:

```java
static long hash(String s) {
    long h = 0;
    long base = 911382323L;

    for (int i = 0; i < s.length(); i++) {
        h = h * base + s.charAt(i);
    }

    return h;
}
```

This is convenient but probabilistic.

For stronger control, use modular arithmetic and potentially two moduli.

Do not claim a hash comparison is mathematically collision-free.

---

# 73. Prefix Hash Implementation

One modular implementation:

```java
static class RollingHash {
    private final long mod;
    private final long base;
    private final long[] prefix;
    private final long[] power;

    RollingHash(String s, long base, long mod) {
        this.base = base;
        this.mod = mod;

        int n = s.length();

        prefix = new long[n + 1];
        power = new long[n + 1];

        power[0] = 1;

        for (int i = 0; i < n; i++) {
            prefix[i + 1] =
                    (prefix[i] * base + s.charAt(i)) % mod;

            power[i + 1] =
                    (power[i] * base) % mod;
        }
    }

    long hash(int left, int right) {
        long value =
                prefix[right]
                - (prefix[left] * power[right - left]) % mod;

        if (value < 0) {
            value += mod;
        }

        return value;
    }
}
```

Here:

```text
hash(left, right)
```

represents:

```text
s[left..right-1]
```

under this convention.

### Important

The base/modulus choices matter in practical collision resistance.

For production-quality hashing, carefully choose parameters rather than blindly copying constants.

---

# 74. Rolling Hash Applications

Once substring hashes are O(1), many problems become possible.

## Compare Two Substrings

```text
hash(l1,r1) == hash(l2,r2)
```

after normalization.

## Find Repeated Substrings

Hash every substring of a fixed length.

## Binary Search the Answer

For problems such as:

```text
longest repeated substring
```

you can sometimes:

```text
binary search length
+
rolling hash feasibility check
```

giving approximately:

```text
O(n log n)
```

depending on the problem.

## Palindrome Detection

Compare a substring against the corresponding reversed substring hash.

---

# 75. Advanced Trie Problems

After basic Trie operations, learn:

## Word Search II

```text
Trie + DFS
```

## Replace Words

```text
Trie + shortest prefix
```

## Design Add and Search Words

```text
Trie + DFS/backtracking
```

## Maximum XOR

Not a traditional character Trie, but the same tree structure can be built over bits.

The deeper lesson:

> A Trie is a tree representation of shared prefixes.

---

# 76. String-Specific Edge Cases

Always test:

```text
""
"a"
"aa"
"ab"
all identical characters
all unique characters
mixed case
spaces
punctuation
digits
Unicode if allowed
very long strings
pattern longer than text
pattern equals text
pattern occurs at index 0
pattern occurs at the end
pattern occurs multiple times
```

For frequency problems:

```text
empty frequency
missing character
duplicate character
```

For sliding windows:

```text
window becomes valid
window becomes invalid after removal
required multiplicity > 1
```

---

# 77. Interview Optimization Ladder

For many string problems, think in this order:

### Level 1 — Brute Force

```text
Generate substrings
compare directly
```

### Level 2 — Frequency / Hashing

```text
avoid repeated character work
```

### Level 3 — Two Pointers

```text
avoid rebuilding ranges
```

### Level 4 — Sliding Window

```text
maintain a valid contiguous range
```

### Level 5 — Prefix/Suffix State

```text
reuse cumulative information
```

### Level 6 — KMP / Z / Rabin-Karp

```text
optimize exact pattern matching
```

### Level 7 — Trie / Rolling Hash

```text
solve structural prefix or substring-equality problems
```

This progression is more useful than memorizing isolated solutions.

---

# 78. Mastery Checklist

## Core Strings

- [ ] Traverse strings.
- [ ] Use `charAt`.
- [ ] Understand `String` immutability.
- [ ] Use `StringBuilder`.
- [ ] Count character frequencies.
- [ ] Understand ASCII.
- [ ] Understand basic Unicode/UTF-16 concepts.
- [ ] Manipulate strings safely.
- [ ] Compare strings with `equals`.

## Palindromes

- [ ] Basic palindrome.
- [ ] Case-insensitive palindrome.
- [ ] Ignore punctuation.
- [ ] Two-pointer palindrome.
- [ ] Expand around center.
- [ ] Understand palindrome hashing conceptually.

## Anagrams

- [ ] Frequency-array anagram.
- [ ] HashMap anagram.
- [ ] Sorting-based anagram.
- [ ] Group anagrams.
- [ ] Find all anagrams using sliding window.

## Substrings

- [ ] Define substring correctly.
- [ ] Count substrings.
- [ ] Enumerate substrings when required.
- [ ] Longest substring.
- [ ] Minimum substring.
- [ ] Fixed-size substring windows.
- [ ] Variable-size windows.

## Subsequences

- [ ] Define subsequence correctly.
- [ ] Check subsequence with two pointers.
- [ ] Understand exponential number of subsequences.
- [ ] Recognize when DP/backtracking is needed.

## Prefix/Suffix

- [ ] Prefix definition.
- [ ] Suffix definition.
- [ ] Proper prefix.
- [ ] Proper suffix.
- [ ] Prefix/suffix overlap.
- [ ] LPS array.

## Hashing

- [ ] Understand string hashing.
- [ ] Polynomial hash.
- [ ] Rolling hash.
- [ ] Prefix hash.
- [ ] Substring hash.
- [ ] Collision awareness.
- [ ] Double hashing concept.

## Pattern Matching

- [ ] Naive matching.
- [ ] KMP.
- [ ] LPS construction.
- [ ] KMP search.
- [ ] Z algorithm.
- [ ] Z-based pattern search.
- [ ] Rabin-Karp.
- [ ] Rolling hash.

## Trie

- [ ] Trie node.
- [ ] Insert.
- [ ] Search.
- [ ] StartsWith.
- [ ] Prefix queries.
- [ ] Trie + DFS.
- [ ] Word Search II.

## Sliding Window

- [ ] Fixed-size window.
- [ ] Variable-size window.
- [ ] Frequency state.
- [ ] Last occurrence state.
- [ ] Longest unique substring.
- [ ] Minimum window.
- [ ] All anagrams.
- [ ] Permutation in string.

---

# 79. Must-Know String Problems

## Easy

- [ ] Valid Palindrome
- [ ] Valid Anagram
- [ ] Longest Common Prefix
- [ ] Is Subsequence
- [ ] First Unique Character
- [ ] Reverse String
- [ ] Find the Index of the First Occurrence

## Medium

- [ ] Longest Substring Without Repeating Characters
- [ ] Group Anagrams
- [ ] Find All Anagrams in a String
- [ ] Permutation in String
- [ ] Longest Palindromic Substring
- [ ] String Compression
- [ ] Implement Trie
- [ ] Add and Search Word

## Hard

- [ ] Minimum Window Substring
- [ ] Word Search II
- [ ] Palindrome Pairs
- [ ] Advanced Trie problems
- [ ] Rolling-hash substring problems
- [ ] Advanced KMP/Z problems

---

# 80. Final String Cheat Sheet

## Frequency

```java
int[] freq = new int[26];

for (char c : s.toCharArray()) {
    freq[c - 'a']++;
}
```

## Palindrome

```java
int left = 0;
int right = s.length() - 1;

while (left < right) {
    if (s.charAt(left) != s.charAt(right)) {
        return false;
    }

    left++;
    right--;
}
```

## Anagram

```text
same character frequencies
```

## Subsequence

```text
advance target pointer every iteration
advance source pointer only on match
```

## Fixed Window

```text
window size = k
add right
remove left
```

## Variable Window

```java
for (int right = 0; right < n; right++) {
    // add right

    while (invalid) {
        // remove left
        left++;
    }

    // process valid window
}
```

## Last Occurrence

```java
left = Math.max(left, last[c] + 1);
```

## KMP

```text
build LPS
match
fallback using LPS
```

## Z Algorithm

```text
Z[i] = longest prefix match starting at i
```

## Rabin-Karp

```text
pattern hash
+
rolling text-window hash
```

## Trie

```text
character -> child
word end -> boolean
```

## Rolling Hash

```text
prefix hash
+
base powers
=
O(1) substring hash
```

---

# 81. Final Interview Mental Model

When you see a string problem:

```text
1. What exactly is being asked?
   frequency / palindrome / anagram / substring /
   subsequence / pattern / prefix / window

2. Is the alphabet constrained?
   lowercase English -> int[26]
   otherwise -> HashMap / appropriate representation

3. Is the problem about both ends?
   two pointers

4. Is it about a contiguous range?
   sliding window

5. Is the window fixed?
   fixed-size sliding window

6. Does the window size vary?
   variable-size sliding window

7. Is it exact pattern matching?
   KMP / Z / Rabin-Karp

8. Are prefix relationships important?
   KMP / Z

9. Are many prefix-based dictionary queries involved?
   Trie

10. Are many substring equality checks involved?
    rolling hash

11. Is the answer a subsequence?
    two pointers / DP depending on the problem

12. Can I state the invariant?
    If not, do not code yet.
```

The main objective is pattern recognition.

A large fraction of interview string questions reduce to:

```text
Frequency
Two Pointers
Sliding Window
Prefix/Suffix
Hashing
KMP
Z Algorithm
Rabin-Karp
Trie
Rolling Hash
```

Master those patterns and then practice variations instead of memorizing individual answers.
