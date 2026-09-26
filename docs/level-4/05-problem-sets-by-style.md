---
description: "Practice sets grouped by common coding-round styles — warm-up screens, pattern mediums, follow-up ladders, implementation-heavy and hard rounds."
---

# 05 · Problem Sets by Interview Style

Coding rounds differ less by company than by **style**. The same organization might run a
short phone screen, a medium pattern problem on site, and an implementation-heavy round
for another team. This lesson groups practice by those styles. It deliberately does not
say which companies ask which questions: processes vary by team and change over time,
and lists claiming otherwise are unreliable.

For each style you get a description, what tends to be valued, a set of practice
problems (described in plain words so you can find them on any practice platform or
simply solve them here), and one fully solved example.

## Style A — The warm-up screen (20–30 minutes)

A shorter round, often by phone or video, with one easy-to-medium problem. Speed and
clean code matter; there may be a small follow-up.

**Practice set:** valid anagram; two sum; best time to buy and sell a stock once;
merge two sorted lists; valid palindrome ignoring punctuation; first unique character;
move zeroes; majority element.

**Solved example — best time to buy and sell once.** Track the lowest price so far; the
best profit selling today is `price - lowest`.

```python
def max_profit(prices):
    lowest, best = float("inf"), 0
    for p in prices:
        lowest = min(lowest, p)
        best = max(best, p - lowest)
    return best


assert max_profit([7, 1, 5, 3, 6, 4]) == 5
assert max_profit([7, 6, 4, 3, 1]) == 0             # never profitable
assert max_profit([]) == 0
assert max_profit([2, 4, 1]) == 2                   # the later minimum doesn't help
```

## Style B — The pattern medium (40–45 minutes)

The most common format: one medium problem that maps to a known pattern, possibly with a
follow-up. Evaluated on approach, code quality, complexity and testing.

**Practice set:** longest substring without repeats; group anagrams; top k frequent
elements; number of islands; course schedule; coin change; merge intervals; kth smallest
in a BST; product of array except self; permutations; word break; rotting oranges.

**Solved example — decode ways.** A digit string maps letters A=1..Z=26. Count decodings.
`dp[i]` = number of ways to decode `s[:i]`.

```python
def num_decodings(s):
    if not s:
        return 0
    prev2, prev1 = 1, 1                  # dp[i-2], dp[i-1]; dp[0] = 1
    for i in range(1, len(s) + 1):
        cur = 0
        if s[i - 1] != "0":
            cur += prev1                 # single digit
        if i >= 2 and "10" <= s[i - 2:i] <= "26":
            cur += prev2                 # two digits
        prev2, prev1 = prev1, cur
    return prev1


assert num_decodings("12") == 2          # AB, L
assert num_decodings("226") == 3
assert num_decodings("06") == 0          # leading zero is invalid
assert num_decodings("10") == 1
assert num_decodings("2101") == 1        # 2 10 1
```

String comparison `"10" <= s[i-2:i] <= "26"` works because both sides are two-character
digit strings.

## Style C — The follow-up ladder

One problem that the interviewer extends step by step: "now the input is a stream", "now
it doesn't fit in memory", "now support deletions". Evaluated on adaptability and on
knowing *why* each change forces a different structure.

**Practice ladders:**

- Two sum → sorted input (two pointers) → data stream with `add`/`find` (hash map of
  counts) → many `find` calls, few `add` calls (precompute pair sums).
- Kth largest → in a stream (size-k heap) → with deletions (two heaps with lazy deletion,
  or a sorted structure).
- Range sum → with updates (Fenwick tree) → 2-D.
- Merge intervals → insert one interval → a stream of intervals (sorted container or a
  balanced BST).

**Solved example — two sum on a stream.**

```python
from collections import Counter

class TwoSumStream:
    def __init__(self):
        self.counts = Counter()

    def add(self, x):
        self.counts[x] += 1

    def find(self, target):
        for x in self.counts:
            y = target - x
            if y in self.counts and (y != x or self.counts[x] > 1):
                return True
        return False


ts = TwoSumStream()
for x in (1, 3, 5):
    ts.add(x)
assert ts.find(4) and not ts.find(7)
assert not ts.find(2)                       # 1 + 1 needs two 1s
ts.add(1)
assert ts.find(2)
```

`add` is O(1), `find` is O(m) for m distinct values. If `find` dominated, you would
flip the trade-off and precompute all pair sums on `add` (O(m) per add, O(1) per find).
Stating that trade-off is usually what the ladder is testing.

## Style D — Implementation-heavy

Less algorithmic cleverness, more careful code: simulate a process, parse a format,
implement a small data structure. Evaluated on correctness, structure, naming and edge
cases. Lesson 6 covers the object-design variant.

**Practice set:** spiral matrix; game of life (in place); text justification; basic
calculator; design a hit counter; string compression; valid sudoku; robot bounded in a
circle.

**Solved example — spiral order.**

```python
def spiral_order(matrix):
    out = []
    if not matrix:
        return out
    top, bottom, left, right = 0, len(matrix) - 1, 0, len(matrix[0]) - 1
    while top <= bottom and left <= right:
        for c in range(left, right + 1):
            out.append(matrix[top][c])
        top += 1
        for r in range(top, bottom + 1):
            out.append(matrix[r][right])
        right -= 1
        if top <= bottom:
            for c in range(right, left - 1, -1):
                out.append(matrix[bottom][c])
            bottom -= 1
        if left <= right:
            for r in range(bottom, top - 1, -1):
                out.append(matrix[r][left])
            left += 1
    return out


assert spiral_order([[1, 2, 3], [4, 5, 6], [7, 8, 9]]) == [1, 2, 3, 6, 9, 8, 7, 4, 5]
assert spiral_order([[1, 2, 3, 4], [5, 6, 7, 8], [9, 10, 11, 12]]) == \
    [1, 2, 3, 4, 8, 12, 11, 10, 9, 5, 6, 7]
assert spiral_order([[1], [2], [3]]) == [1, 2, 3]      # single column
assert spiral_order([]) == []
```

The two `if` guards prevent re-reading a row or column when the remaining region is a
single line — the classic bug in this problem.

## Style E — The hard round

Fewer candidates see these, and interviewers often expect partial progress rather than a
perfect solution. Evaluated on how far you get and how clearly you reason.

**Practice set:** trapping rain water; median of two sorted arrays; minimum window
substring; word ladder; largest rectangle in histogram; merge k sorted lists; edit
distance; serialize/deserialize a binary tree; sliding window maximum. Level 3's project
has full walkthroughs for several of these.

## How It Actually Works

Grouping by style rather than by company works because the **skills being sampled** are
what stay stable. Pattern mediums sample recognition and execution; follow-up ladders
sample understanding of trade-offs; implementation rounds sample care and structure; hard
rounds sample reasoning under difficulty. A candidate who prepares for these skills is
prepared for whatever specific question appears, while one who memorizes rumoured
question lists is exposed as soon as the question changes — and it usually does.

Practising by style also teaches **time calibration**. A warm-up screen punishes long
deliberation; a hard round punishes rushing into code. Knowing which kind of round you
are in tells you how to spend your minutes.

## Exercise

Build a personal practice plan: choose two problems from each practice set above that
you have not solved (ten total). Solve each under the time limit of its style, recording
time taken, whether you needed a hint, and which pattern it was. Then write the full
follow-up ladder for one Style C problem, solving every rung with tested code.
