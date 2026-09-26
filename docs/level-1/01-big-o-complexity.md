---
description: "Analyze time and space complexity with Big-O, spot dominant terms, understand amortized cost, and use input constraints to pick a target algorithm."
---

# 01 · Big-O & Complexity Analysis

Every solution in this course is written in **Python 3**. The ideas — and the complexity
arguments — transfer unchanged to any other language; only the syntax differs.

Before you can compare two solutions you need a way to talk about how much work each
does as the input grows. Big-O notation is that vocabulary. In an interview it is also a
signal: stating the complexity of your approach *before* you code it shows you know
where you are headed.

## What Big-O measures

Big-O describes an **upper bound on growth**, ignoring constant factors and lower-order
terms. If a function performs `3n² + 10n + 7` basic steps on an input of size `n`, we
say it is **O(n²)**: once `n` is large, the `n²` term dominates everything else, and the
constant 3 depends on the machine and the language anyway.

The classes you will meet constantly, from fastest to slowest:

| Class | Name | Typical source |
|-------|------|----------------|
| O(1) | constant | index into a list, dict lookup (average) |
| O(log n) | logarithmic | binary search, heap push/pop, balanced-tree operations |
| O(n) | linear | one pass over the input |
| O(n log n) | linearithmic | comparison sorting, n heap operations |
| O(n²) | quadratic | all pairs, nested loops over the same input |
| O(2ⁿ) | exponential | all subsets |
| O(n!) | factorial | all permutations |

## Counting work: a worked problem

**Problem.** Given a list of integers, return `True` if any value appears at least twice.

### Approach 1 — compare every pair

```python
def has_duplicate_pairs(nums):
    n = len(nums)
    for i in range(n):
        for j in range(i + 1, n):
            if nums[i] == nums[j]:
                return True
    return False
```

The inner loop runs `n-1`, then `n-2`, ..., then 0 times: a total of `n(n-1)/2`
comparisons in the worst case (no duplicates). That is **O(n²) time, O(1) extra space**.

### Approach 2 — sort first

```python
def has_duplicate_sort(nums):
    s = sorted(nums)               # O(n log n) time, O(n) space for the copy
    for i in range(1, len(s)):     # O(n)
        if s[i] == s[i - 1]:
            return True
    return False
```

After sorting, equal values are adjacent. Total: O(n log n) + O(n) = **O(n log n)**.
Sequential steps *add*; the larger term wins.

### Approach 3 — remember what you have seen

```python
def has_duplicate(nums):
    seen = set()
    for x in nums:
        if x in seen:              # average O(1)
            return True
        seen.add(x)                # average O(1)
    return False


assert has_duplicate([1, 2, 3, 1]) is True
assert has_duplicate([1, 2, 3]) is False
assert has_duplicate([]) is False           # edge case: empty input
assert has_duplicate([7]) is False          # edge case: single element
for f in (has_duplicate_pairs, has_duplicate_sort):
    assert f([4, 4]) is True and f([]) is False
```

**O(n) time, O(n) space.** This is the classic trade: spend memory to save time.
Which one is "best" depends on constraints — if memory is extremely tight, the sorting
version (in place with `nums.sort()`) may be preferred.

## Rules of thumb for analysis

- **Nested loops multiply** when the inner loop's length depends on the input:
  `for i in range(n): for j in range(m)` is O(n·m).
- **Sequential blocks add**, and you keep the largest term.
- **Halving the problem each step** gives O(log n). Doubling a counter until it reaches
  `n` also takes O(log n) steps.
- **Recursion**: multiply the number of calls by the work per call. A recursive function
  that makes two calls on `n-1` makes about 2ⁿ calls.
- **Hidden costs in Python**: `x in some_list` is O(n); `lst.pop(0)` and
  `lst.insert(0, x)` are O(n); slicing `s[a:b]` copies `b-a` elements; `s1 + s2` builds a
  new string. These show up inside loops and silently turn O(n) into O(n²).

```python
# Looks linear, is quadratic: each 'in' scans the list.
def common_slow(a, b):
    return [x for x in a if x in b]          # O(len(a) * len(b))

def common_fast(a, b):
    bset = set(b)                             # O(len(b))
    return [x for x in a if x in bset]        # O(len(a)) average

assert common_slow([1, 2, 3], [3, 1]) == common_fast([1, 2, 3], [3, 1]) == [1, 3]
```

## Space complexity

Space complexity counts **extra** memory as a function of input size: new lists, dicts,
sets, and the **recursion call stack**. A recursive function that goes `n` levels deep
uses O(n) stack space even if it allocates nothing else. Most interviewers do not count
the output itself (e.g. the list you must return) unless asked; say which convention you
are using.

## Reading constraints to choose a target

Problem statements usually give input limits. Roughly, a well-written Python solution can
perform on the order of ten million simple operations per second — treat that as a loose
guide, not a benchmark. From that you can back out what complexity is expected:

| Constraint on n | Complexity that usually fits |
|-----------------|------------------------------|
| n ≤ 10–12 | O(n!) or O(2ⁿ · n) — brute force permutations/subsets |
| n ≤ 20–25 | O(2ⁿ) — subsets, bitmask DP |
| n ≤ 500 | O(n³) |
| n ≤ 5,000 | O(n²) |
| n ≤ 10⁵ – 10⁶ | O(n log n) or O(n) |
| n ≥ 10⁸, or values up to 10¹⁸ | O(log n) or O(1) — math or binary search |

This table is one of the most useful tools in an online assessment: if `n` can be 10⁵,
an O(n²) idea will not pass, and you should keep looking before you code.

## How It Actually Works

**Why constants are dropped.** Big-O is defined formally: `f(n) = O(g(n))` if there
exist constants `c > 0` and `n₀` such that `f(n) ≤ c · g(n)` for all `n ≥ n₀`. For
`3n² + 10n + 7`, pick `c = 4`; for `n ≥ 11`, `10n + 7 ≤ n²`, so the whole thing is at
most `4n²`. The definition is deliberately insensitive to constant factors because those
change with hardware, interpreter, and implementation, while the growth rate does not.
Related notations: **Ω** (lower bound) and **Θ** (tight bound — both upper and lower).
Interviewers say "Big-O" but usually mean the tight bound.

**Why O(log n) shows up.** If each step discards half the remaining input, after `k`
steps `n / 2ᵏ` items remain. The loop ends when that reaches 1, i.e. `k = log₂ n`. For a
million items that is about 20 steps; for a billion, about 30. The base of the logarithm
does not matter in Big-O because `log_a n = log_b n / log_b a` — a constant factor.

**Amortized analysis: why `list.append` is O(1).** A Python list is a dynamic array: a
contiguous block of slots with some spare capacity. When the block is full, CPython
allocates a larger block (growing by a proportional factor, roughly 1.125× plus a small
constant in current CPython) and copies everything over. That copy is O(n) — so how can
append be O(1)?

Consider a simplified array that *doubles* when full. Appending n items triggers copies
of sizes 1, 2, 4, ..., up to n, which sum to less than 2n. Spread across n appends, that
is fewer than 2 copies per append — constant *on average over the sequence*. That is what
**amortized O(1)** means: any single append might be slow, but the total cost of n
appends is O(n). Any constant growth factor greater than 1 gives the same result (a
smaller factor just raises the constant). Growing by a *fixed amount* (say +10 slots)
would not: total copying becomes O(n²).

**Average vs worst case.** Dict and set operations are O(1) on average but O(n) in the
worst case (lesson 3 explains collisions). Quicksort-style algorithms are O(n log n)
average but O(n²) worst case. State which one you mean; "average O(1)" is the standard
claim for hash lookups.

## Common mistakes

- Calling a solution O(n) when it has `in list`, `.index()`, `.pop(0)`, or string
  concatenation inside the loop.
- Forgetting the recursion stack when stating space complexity.
- Treating two *different* inputs as one: iterating over `a` and looking up in `b` is
  O(|a| + |b|), not O(n). Name your variables in the complexity.
- Saying "O(2n)" — drop the constant; say O(n).
- Over-optimizing: for n ≤ 1,000 an O(n²) solution is fine and often clearer. Match the
  constraints, then optimize only if needed.

## Exercise

1. State the time and space complexity of each function, then check yourself by
   reasoning about the worst-case input:

    ```python
    def f1(n):
        i, total = 1, 0
        while i < n:
            total += i
            i *= 3
        return total

    def f2(nums):
        out = []
        for x in nums:
            out = out + [x * 2]
        return out

    def f3(s):
        return len(set(s)) == len(s)

    assert f1(10) == 1 + 3 + 9
    assert f2([1, 2]) == [2, 4]
    assert f3("abc") and not f3("aba")
    ```

2. Rewrite `f2` so it is O(n).
3. A problem says `1 ≤ n ≤ 2 · 10⁵`. Your idea is O(n²). Estimate how many operations
   that is and decide whether it will pass.

*(Answers: f1 is O(log n) time — `i` triples each step; f2 is O(n²) because
`out + [...]` copies the list every iteration; f3 is O(n) average time and O(n) space.)*
