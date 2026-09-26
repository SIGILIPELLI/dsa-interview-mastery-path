---
description: "Solve array and string problems in Python: in-place writes, product-except-self, string building with join, and the real cost of slicing and copies."
---

# 02 · Arrays & Strings

Arrays and strings are the raw material of most interview problems. The techniques here
are not glamorous, but they come up inside nearly every harder pattern: scanning with an
index, writing results in place, precomputing from the left and the right, and building
strings efficiently.

In Python, "array" means `list`. Strings (`str`) are **immutable** sequences — you cannot
change a character in place; every modification creates a new string.

## Pattern 1: the read/write pointer (in-place compaction)

**Problem.** Move all zeros in a list to the end, keeping the relative order of the
non-zero elements. Do it in place.

**Brute force.** Build a new list of non-zeros, then append zeros: O(n) time but O(n)
extra space, and it is not in place.

**Approach.** Keep a `write` index marking where the next non-zero belongs. Scan with a
`read` index; each time you see a non-zero, copy it to `write` and advance `write`.
After the scan, fill the rest with zeros.

```python
def move_zeroes(nums):
    write = 0
    for read in range(len(nums)):
        if nums[read] != 0:
            nums[write] = nums[read]
            write += 1
    for i in range(write, len(nums)):
        nums[i] = 0


a = [0, 1, 0, 3, 12]
move_zeroes(a)
assert a == [1, 3, 12, 0, 0]
b = [0, 0]
move_zeroes(b)
assert b == [0, 0]          # edge: all zeros
c = []
move_zeroes(c)
assert c == []              # edge: empty
d = [1, 2]
move_zeroes(d)
assert d == [1, 2]          # edge: no zeros
```

**Complexity:** O(n) time, O(1) extra space.

**Invariant:** at every step, `nums[0:write]` holds exactly the non-zeros seen so far,
in their original order. Stating an invariant like this is the fastest way to convince
an interviewer (and yourself) that the code is right.

This read/write pattern also solves "remove duplicates from a sorted array", "remove all
occurrences of a value", and "compress a string in place".

## Pattern 2: precompute from both sides

**Problem.** Given `nums`, return `answer` where `answer[i]` is the product of every
element except `nums[i]`. Do not use division. Target O(n).

**Why not divide?** Division fails when there is a zero, and the problem forbids it.

**Approach.** The product of everything except `i` equals
(product of everything left of `i`) × (product of everything right of `i`). Compute the
left products in one pass into the output list, then sweep from the right with a running
product.

```python
def product_except_self(nums):
    n = len(nums)
    answer = [1] * n
    left = 1
    for i in range(n):
        answer[i] = left        # product of nums[0..i-1]
        left *= nums[i]
    right = 1
    for i in range(n - 1, -1, -1):
        answer[i] *= right      # times product of nums[i+1..n-1]
        right *= nums[i]
    return answer


assert product_except_self([1, 2, 3, 4]) == [24, 12, 8, 6]
assert product_except_self([-1, 1, 0, -3, 3]) == [0, 0, 9, 0, 0]
assert product_except_self([0, 0]) == [0, 0]       # edge: two zeros
assert product_except_self([5, 2]) == [2, 5]       # edge: minimum length
```

Trace `[1, 2, 3, 4]`: after the left pass `answer = [1, 1, 2, 6]`. The right pass
walks from the end with `right` taking the values 1, 4, 12, 24, so index 3 becomes
6·1, index 2 becomes 2·4, index 1 becomes 1·12, and index 0 becomes 1·24, giving
`[24, 12, 8, 6]`.

**Complexity:** O(n) time, O(1) extra space if the output list is not counted.

The "left pass + right pass" idea reappears in *trapping rain water*, *candy
distribution*, and *best time to buy and sell stock* variants.

## Pattern 3: building strings

**Problem.** Reverse the order of words in a sentence. Words are separated by one or
more spaces; the result should have single spaces and no leading/trailing space.

```python
def reverse_words(s):
    return " ".join(reversed(s.split()))


assert reverse_words("the sky  is blue") == "blue is sky the"
assert reverse_words("  hello world  ") == "world hello"
assert reverse_words("   ") == ""          # edge: only spaces
assert reverse_words("a") == "a"
```

`str.split()` with no argument splits on runs of whitespace and drops empty strings,
which handles the messy spacing for free. An interviewer may ask you to do it "without
built-ins"; the manual version scans for word boundaries and collects words into a list:

```python
def reverse_words_manual(s):
    words, i, n = [], 0, len(s)
    while i < n:
        while i < n and s[i] == " ":
            i += 1
        j = i
        while j < n and s[j] != " ":
            j += 1
        if i < j:
            words.append(s[i:j])
        i = j
    words.reverse()
    return " ".join(words)


assert reverse_words_manual("the sky  is blue") == "blue is sky the"
assert reverse_words_manual("") == ""
```

## Pattern 4: character counting with a fixed alphabet

**Problem.** Are two strings anagrams of each other?

```python
def is_anagram(s, t):
    if len(s) != len(t):
        return False
    counts = [0] * 26                  # assumes lowercase a-z
    for a, b in zip(s, t):
        counts[ord(a) - ord("a")] += 1
        counts[ord(b) - ord("a")] -= 1
    return all(c == 0 for c in counts)


assert is_anagram("anagram", "nagaram")
assert not is_anagram("rat", "car")
assert is_anagram("", "")
assert not is_anagram("a", "ab")
```

O(n) time and O(1) space — the count array has a fixed size of 26 regardless of `n`.
If the alphabet is arbitrary Unicode, use `collections.Counter(s) == Counter(t)`
instead; space becomes O(k) for k distinct characters. Ask which one applies.

## How It Actually Works

**A Python list is an array of pointers.** CPython stores a list as a contiguous block
of references to objects, plus a length and a capacity. Indexing `lst[i]` computes an
address (`base + i × pointer_size`) — that is why it is O(1). Inserting at the front has
to shift every reference one slot right, which is O(n). Appending at the end usually
writes into spare capacity (amortized O(1), see lesson 1).

**Slices copy.** `nums[1:]` allocates a new list and copies n-1 references. Recursion
that passes `nums[1:]` down each level does O(n) copying per call, turning an O(n)
algorithm into O(n²) time and space. Pass indices (`lo`, `hi`) instead.

**Strings are immutable, so `+=` in a loop can be quadratic.** `s += c` conceptually
creates a new string and copies the old contents. CPython has an optimization that can
sometimes resize the string in place when nothing else references it, but it is an
implementation detail you should not rely on — and it does not exist in every Python.
The portable, always-linear idiom is to append pieces to a list and call `"".join(parts)`
once at the end; `join` computes the total length first, allocates once, and copies each
piece exactly once.

**`ord` and fixed-size counting.** Characters map to integer code points, so a letter
can index directly into an array. This is a direct-addressed table: no hashing, no
collisions, guaranteed O(1). It only works when the key range is small and known.

## Common mistakes

- Modifying a list while iterating over it with `for x in lst` (skips elements). Iterate
  over indices or build a new list.
- Off-by-one on inclusive/exclusive bounds. Python slices are half-open: `s[i:j]` has
  `j - i` characters and excludes `s[j]`.
- Treating strings as mutable: `s[0] = "x"` raises `TypeError`. Convert to a list of
  characters, modify, then `join`.
- Forgetting the empty input, a single element, all-identical elements, and negative
  numbers.
- Using `sorted(s) == sorted(t)` for anagrams and then claiming O(n) — it is O(n log n).

## Variations to practice

- Rotate an array right by `k` steps in place (hint: reverse the whole array, then
  reverse the first `k` and the rest; handle `k > n` with `k %= n`).
- Longest common prefix of a list of strings.
- Check if a string is a palindrome considering only alphanumeric characters.
- Spiral traversal of a matrix.

## Exercise

Write `rotate(nums, k)` that rotates `nums` right by `k` steps **in place** using O(1)
extra space. Test it on `[1,2,3,4,5,6,7], k=3` → `[5,6,7,1,2,3,4]`, on `k=0`, on
`k = len(nums)`, on `k` larger than the length, and on a single-element list. Then state
its time and space complexity and explain why the triple-reversal trick is correct
(what does each reversal accomplish?).
