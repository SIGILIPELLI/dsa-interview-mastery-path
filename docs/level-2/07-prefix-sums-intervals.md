---
description: "Answer range-sum queries in O(1) with prefix sums, count subarrays summing to k with a hash map, and merge, insert and schedule intervals after sorting."
---

# 07 · Prefix Sums & Intervals

Two families of problems share a theme: turning questions about **ranges** into cheap
operations on precomputed or sorted data.

- **Prefix sums** precompute running totals so any range sum is one subtraction.
- **Interval problems** sort by start (or end) so overlapping ranges become adjacent.

## Prefix sums

Define `prefix[i]` as the sum of the first `i` elements (`prefix[0] = 0`). Then the sum
of `nums[l..r]` inclusive is `prefix[r + 1] - prefix[l]`.

```python
from itertools import accumulate

class RangeSum:
    def __init__(self, nums):
        self.prefix = [0] + list(accumulate(nums))

    def query(self, left, right):             # inclusive
        return self.prefix[right + 1] - self.prefix[left]


rs = RangeSum([-2, 0, 3, -5, 2, -1])
assert rs.query(0, 2) == 1
assert rs.query(2, 5) == -1
assert rs.query(0, 5) == -3
assert rs.query(3, 3) == -5                  # edge: single element
```

O(n) to build, O(1) per query. The leading 0 is what makes `left = 0` work without a
special case.

### Worked problem 1: subarray sum equals k

**Problem.** Count contiguous subarrays whose sum is exactly `k`. Values may be
**negative**, so sliding window does not apply.

**Insight.** A subarray `(j, i]` sums to `k` exactly when
`prefix[i] - prefix[j] = k`, i.e. `prefix[j] = prefix[i] - k`. As you scan, keep a
count of how many times each prefix sum has occurred so far; each earlier occurrence of
`prefix[i] - k` is one valid subarray ending at `i`.

```python
from collections import defaultdict

def subarray_sum(nums, k):
    seen = defaultdict(int)
    seen[0] = 1                        # the empty prefix
    running = count = 0
    for x in nums:
        running += x
        count += seen[running - k]
        seen[running] += 1
    return count


assert subarray_sum([1, 1, 1], 2) == 2
assert subarray_sum([1, 2, 3], 3) == 2           # [1, 2] and [3]
assert subarray_sum([1, -1, 0], 0) == 3          # [1,-1], [0], [1,-1,0]
assert subarray_sum([], 0) == 0
assert subarray_sum([3], 3) == 1
```

**Why `seen[0] = 1`?** It represents the prefix before any element, so subarrays starting
at index 0 are counted. Forgetting it is the most common bug in this problem.

**Complexity:** O(n) time, O(n) space. The same trick handles "longest subarray with sum
k" (store the *first* index of each prefix sum) and "subarray sum divisible by k" (store
prefix sums modulo k).

### 2-D prefix sums

For a matrix, `P[r][c]` = sum of the rectangle from `(0,0)` to `(r-1, c-1)`. Inclusion–
exclusion gives any sub-rectangle in O(1):

```python
def build_2d(matrix):
    rows, cols = len(matrix), len(matrix[0])
    P = [[0] * (cols + 1) for _ in range(rows + 1)]
    for r in range(rows):
        for c in range(cols):
            P[r + 1][c + 1] = matrix[r][c] + P[r][c + 1] + P[r + 1][c] - P[r][c]
    return P


def rect_sum(P, r1, c1, r2, c2):              # inclusive corners
    return P[r2 + 1][c2 + 1] - P[r1][c2 + 1] - P[r2 + 1][c1] + P[r1][c1]


P = build_2d([[3, 0, 1], [5, 6, 3], [1, 2, 0]])
assert rect_sum(P, 1, 1, 2, 2) == 11          # 6 + 3 + 2 + 0
assert rect_sum(P, 0, 0, 2, 2) == 21
assert rect_sum(P, 0, 0, 0, 0) == 3
```

## Intervals

Represent intervals as `[start, end]` pairs. Nearly every interval problem starts with
**sorting by start** (or by end, for scheduling).

### Worked problem 2: merge overlapping intervals

```python
def merge_intervals(intervals):
    merged = []
    for start, end in sorted(intervals):
        if merged and start <= merged[-1][1]:          # overlaps the last one
            merged[-1][1] = max(merged[-1][1], end)
        else:
            merged.append([start, end])
    return merged


assert merge_intervals([[1, 3], [2, 6], [8, 10], [15, 18]]) == [[1, 6], [8, 10], [15, 18]]
assert merge_intervals([[1, 4], [4, 5]]) == [[1, 5]]        # touching counts as overlap
assert merge_intervals([[1, 4], [2, 3]]) == [[1, 4]]        # contained interval
assert merge_intervals([]) == []
```

`max(...)` in the merge handles the contained-interval case — a frequent miss.
**Complexity:** O(n log n) for the sort, O(n) for the scan.

### Worked problem 3: meeting rooms (minimum rooms needed)

**Problem.** Given meeting time intervals, what is the minimum number of rooms so that no
meetings overlap in the same room?

**Approach (sweep line).** Separate starts and ends, sort both, and walk through time. A
start needs a room; an end frees one. The maximum number simultaneously in use is the
answer. When a meeting ends exactly as another starts, process the end first (a room
freed at 10 can be reused at 10).

```python
def min_meeting_rooms(intervals):
    starts = sorted(s for s, _ in intervals)
    ends = sorted(e for _, e in intervals)
    rooms = best = 0
    j = 0
    for s in starts:
        while j < len(ends) and ends[j] <= s:     # free rooms whose meeting ended
            rooms -= 1
            j += 1
        rooms += 1
        best = max(best, rooms)
    return best


assert min_meeting_rooms([[0, 30], [5, 10], [15, 20]]) == 2
assert min_meeting_rooms([[7, 10], [2, 4]]) == 1
assert min_meeting_rooms([[1, 5], [5, 10]]) == 1           # back-to-back
assert min_meeting_rooms([]) == 0
assert min_meeting_rooms([[1, 10], [2, 10], [3, 10]]) == 3
```

An equivalent approach uses a min-heap of end times (lesson 3): pop when the earliest end
is ≤ the next start, push the new end, and the heap's maximum size is the answer.

### Worked problem 4: insert an interval

**Problem.** Insert a new interval into a sorted list of non-overlapping intervals,
merging as needed.

```python
def insert_interval(intervals, new):
    out, i, n = [], 0, len(intervals)
    start, end = new
    while i < n and intervals[i][1] < start:          # entirely before
        out.append(intervals[i]); i += 1
    while i < n and intervals[i][0] <= end:           # overlapping: absorb
        start = min(start, intervals[i][0])
        end = max(end, intervals[i][1])
        i += 1
    out.append([start, end])
    out.extend(intervals[i:])                          # entirely after
    return out


assert insert_interval([[1, 3], [6, 9]], [2, 5]) == [[1, 5], [6, 9]]
assert insert_interval([[1, 2], [3, 5], [6, 7], [8, 10], [12, 16]], [4, 8]) == \
    [[1, 2], [3, 10], [12, 16]]
assert insert_interval([], [5, 7]) == [[5, 7]]
assert insert_interval([[1, 5]], [6, 8]) == [[1, 5], [6, 8]]
```

O(n) — no sort needed because the input is already sorted.

## How It Actually Works

**Prefix sums are the discrete version of integration.** A range sum is the difference of
two cumulative totals, just like a definite integral is `F(b) - F(a)`. The technique works
for any operation with an **inverse**: sums (subtract), XOR (XOR again), products of
non-zero numbers (divide). It does **not** work for min or max, because you cannot "undo"
a minimum — that is exactly why range-minimum queries need segment trees or sparse
tables (Level 3, lesson 7). The inverse also explains a limitation: if the array changes,
every later prefix changes, so updates cost O(n). Fenwick trees fix that.

**The hash map trick is two-sum on prefix sums.** "Find `j < i` with
`prefix[j] = prefix[i] - k`" is Two Sum from Level 1 applied to the prefix array, with
counts instead of indices.

**Why sorting makes intervals easy.** After sorting by start, if interval B does not
overlap the merged interval A just before it, then no *later* interval can overlap A
either (they all start at or after B's start, which is already past A's end). So each
merged block can be closed and never revisited — a one-pass scan suffices. The sweep line
for meeting rooms works on the same principle: after sorting events by time, the number
of active intervals only changes at start/end points, so checking those 2n points finds
the maximum.

## Common mistakes

- Forgetting `seen[0] = 1` (or `prefix[0] = 0`).
- Using a sliding window for subarray-sum problems with negative numbers.
- Treating touching intervals inconsistently — decide (and confirm) whether `[1,4]` and
  `[4,5]` overlap.
- Replacing the merged end with the new end instead of taking the `max`.
- Sorting intervals by end when merging (by start is correct for merging; by end is the
  greedy choice for "max non-overlapping intervals" — lesson 8).

## Variations to practice

- Continuous subarray sum (multiple of k, length ≥ 2).
- Product of array except self (prefix and suffix products — Level 1, lesson 2).
- Range addition with a difference array (the inverse of a prefix sum).
- Interval list intersections (two pointers over two sorted lists).
- Car pooling (difference array or sweep line).

## Exercise

Implement a **difference array**: given `length` and a list of updates
`(start, end, inc)` meaning "add `inc` to every index in `[start, end]`", return the
final array in O(length + number of updates). Hint: add `inc` at `start`, subtract it at
`end + 1`, then take a prefix sum. Test with `length=5, updates=[(1,3,2),(2,4,3),(0,2,-2)]`
→ `[-2, 0, 3, 5, 3]`, with no updates, and with an update covering the last index.
