---
description: "Use Python dicts and sets for O(1) lookups: two-sum, grouping anagrams, frequency counting, and how open addressing and resizing make hashing fast."
---

# 03 · Hash Maps & Sets

If one data structure earns you the most interview points per minute of study, it is the
hash map. The question "have I seen this before, and where?" appears in a huge share of
array and string problems, and a hash map answers it in average O(1) time.

In Python: `dict` is the hash map, `set` is the hash set, and `collections` adds
`Counter` (a dict of counts) and `defaultdict` (a dict that creates missing values).

## Worked problem 1: Two Sum

**Problem.** Given `nums` and `target`, return indices `i != j` with
`nums[i] + nums[j] == target`. Exactly one solution exists.

**Brute force.** Try all pairs: O(n²).

**Key insight.** For each `x`, the partner you need is `target - x`. If you have stored
every earlier value with its index, you can check for the partner in O(1).

```python
def two_sum(nums, target):
    index_of = {}                       # value -> index where we saw it
    for i, x in enumerate(nums):
        need = target - x
        if need in index_of:
            return [index_of[need], i]
        index_of[x] = i
    return []                           # not reached if a solution is guaranteed


assert two_sum([2, 7, 11, 15], 9) == [0, 1]
assert two_sum([3, 2, 4], 6) == [1, 2]
assert two_sum([3, 3], 6) == [0, 1]     # edge: duplicate values
assert two_sum([-1, -2, -3, -4], -7) == [2, 3]   # negatives
```

**Why check before inserting?** For `[3, 2, 4], target = 6`, checking after inserting 3
would find `6 - 3 = 3` — the same element — and return `[0, 0]`. Checking first
guarantees `i != j`.

**Complexity:** O(n) time, O(n) space.

## Worked problem 2: Group Anagrams

**Problem.** Group a list of words so that anagrams end up together.

**Approach.** Anagrams share a *canonical key*. Two choices:

- the sorted characters, `"".join(sorted(w))` — O(k log k) per word of length k;
- a 26-length count tuple — O(k) per word.

Dict keys must be hashable, so use a `tuple` (not a list).

```python
from collections import defaultdict

def group_anagrams(words):
    groups = defaultdict(list)
    for w in words:
        counts = [0] * 26
        for ch in w:
            counts[ord(ch) - ord("a")] += 1
        groups[tuple(counts)].append(w)
    return list(groups.values())


result = group_anagrams(["eat", "tea", "tan", "ate", "nat", "bat"])
assert sorted(sorted(g) for g in result) == [["ate", "eat", "tea"], ["bat"], ["nat", "tan"]]
assert group_anagrams([""]) == [[""]]            # edge: empty word
assert group_anagrams([]) == []                  # edge: no words
```

**Complexity:** O(n · k) time for n words of max length k; O(n · k) space.

## Worked problem 3: Longest Consecutive Sequence

**Problem.** Given an unsorted list, return the length of the longest run of consecutive
integers (e.g. `[100, 4, 200, 1, 3, 2]` → 4, for 1-2-3-4). Target O(n).

**Approach.** Sorting gives O(n log n). For O(n), put everything in a set. Only *start*
counting from a number `x` when `x - 1` is not in the set — that is, when `x` begins a
run. Then walk upward.

```python
def longest_consecutive(nums):
    values = set(nums)
    best = 0
    for x in values:
        if x - 1 not in values:           # x starts a run
            length = 1
            while x + length in values:
                length += 1
            best = max(best, length)
    return best


assert longest_consecutive([100, 4, 200, 1, 3, 2]) == 4
assert longest_consecutive([0, 3, 7, 2, 5, 8, 4, 6, 0, 1]) == 9
assert longest_consecutive([]) == 0               # edge: empty
assert longest_consecutive([5, 5, 5]) == 1        # edge: duplicates
```

**Why is this O(n) despite the nested `while`?** The inner loop only runs from run
starts, and each number is visited by the inner loop at most once overall (it belongs to
exactly one run). Total inner iterations ≤ n. This "each element is touched a bounded
number of times" argument is the same one used for sliding windows.

## Counting with `Counter`

```python
from collections import Counter

def first_unique_char(s):
    counts = Counter(s)
    for i, ch in enumerate(s):
        if counts[ch] == 1:
            return i
    return -1


assert first_unique_char("leetcode") == 0
assert first_unique_char("loveleetcode") == 2
assert first_unique_char("aabb") == -1
assert first_unique_char("") == -1
```

`Counter.most_common(k)` returns the k most frequent items, which is handy for
"top k frequent elements" (lesson 3 of Level 2 shows the heap-based alternative).

## How It Actually Works

**Hashing.** `hash(key)` turns a key into a large integer. The table takes that integer
modulo its size (in CPython, by masking low bits, since the size is a power of two) to
pick a slot. If keys were spread evenly, each slot would hold about one key, and lookup
would be one probe: O(1).

**Collisions.** Different keys can land in the same slot. There are two classic fixes:

- **Separate chaining** — each slot holds a small list of entries; a lookup scans that
  list. Java's `HashMap` uses chaining (and converts long chains into balanced trees).
- **Open addressing** — if the slot is taken, probe other slots in a deterministic
  sequence until you find the key or an empty slot. CPython's `dict` and `set` use open
  addressing with a perturbed probe sequence that mixes in higher bits of the hash so
  that keys with similar low bits spread out.

**Load factor and resizing.** As the table fills, probe sequences get longer. CPython
resizes a dict when it is about two-thirds full, allocating a bigger table and
re-inserting every key. That rehash is O(n), but because the table grows
geometrically, the cost amortizes to O(1) per insertion — the same argument as list
appends.

**Why the worst case is O(n).** If many keys collide (by bad luck or a deliberately
crafted input), every lookup degrades to scanning them. To make that attack hard, Python
randomizes string and bytes hashes per process (`PYTHONHASHSEED`). Small integers hash to
themselves, which is fine for typical interview inputs.

**Why keys must be immutable.** The slot is chosen from the hash at insertion time. If a
key could change after insertion, its hash would change, and a later lookup would probe
the wrong place. That is why `list` and `dict` are unhashable, while `tuple` (of hashable
items) and `frozenset` are allowed.

**Insertion order.** Since Python 3.7, dicts preserve insertion order as a language
guarantee: CPython keeps entries in a compact array in insertion order and uses a
separate sparse index table for hashing. Sets make no ordering promise.

## Common mistakes

- Using a list as a dict key (`TypeError: unhashable type: 'list'`) — convert to a
  tuple.
- Adding to the map before checking for the complement in Two Sum.
- `counts[x] += 1` on a plain dict raises `KeyError` for new keys — use `Counter`,
  `defaultdict(int)`, or `counts.get(x, 0) + 1`.
- Forgetting that a set drops duplicates when the problem cares about multiplicity.
- Claiming worst-case O(1). Say "average O(1)".

## Variations to practice

- Subarray sum equals k (hash map of prefix sums — Level 2, lesson 7).
- Isomorphic strings (two maps, one each direction).
- Contains duplicate within distance k (map value → last index).
- Four-sum count across four lists (pair sums into a Counter: O(n²)).

## Exercise

Implement `word_pattern(pattern, s)`: return `True` if the words in `s` follow the same
pattern as the letters in `pattern` with a **bijection** — e.g. `"abba"`,
`"dog cat cat dog"` → `True`; `"abba"`, `"dog dog dog dog"` → `False`. Write at least
five asserts, including mismatched lengths and a case that fails only because the
mapping is not one-to-one in the reverse direction. State the complexity.
