---
description: "Support range queries with point updates in O(log n) using Fenwick trees and segment trees, including range minimum and counting inversions."
---

# 07 · Segment Trees & Fenwick Trees

Prefix sums answer range-sum queries in O(1) — until the array changes. One update
invalidates every later prefix, costing O(n). When a problem mixes **updates** and
**range queries**, you need a structure that makes both O(log n):

| Need | Structure |
|---|---|
| Range sum, point update | Fenwick tree (simplest) or segment tree |
| Range min/max, point update | Segment tree |
| Range update and range query | Segment tree with lazy propagation |
| Static range min, no updates | Sparse table (O(1) query) |

These appear less often in interviews than the Level 1–2 patterns, but they are common in
online assessments with large constraints, and "now support updates" is a classic
follow-up to prefix-sum problems.

## Fenwick tree (binary indexed tree)

A Fenwick tree stores, at 1-based index `i`, the sum of a block of elements ending at `i`
whose length is the **lowest set bit** of `i` (`i & -i`, from lesson 6). Prefix queries
walk downward by stripping the lowest bit; updates walk upward by adding it.

```python
class Fenwick:
    def __init__(self, n):
        self.tree = [0] * (n + 1)          # 1-based

    def add(self, i, delta):               # i is 0-based
        i += 1
        while i < len(self.tree):
            self.tree[i] += delta
            i += i & -i

    def prefix(self, i):                   # sum of elements [0, i)
        total = 0
        while i > 0:
            total += self.tree[i]
            i -= i & -i
        return total

    def range_sum(self, left, right):      # inclusive, 0-based
        return self.prefix(right + 1) - self.prefix(left)


class NumArray:
    """Range sum query with point assignment updates."""
    def __init__(self, nums):
        self.nums = list(nums)
        self.fw = Fenwick(len(nums))
        for i, x in enumerate(nums):
            self.fw.add(i, x)

    def update(self, i, val):
        self.fw.add(i, val - self.nums[i])
        self.nums[i] = val

    def sum_range(self, left, right):
        return self.fw.range_sum(left, right)


na = NumArray([1, 3, 5])
assert na.sum_range(0, 2) == 9
na.update(1, 2)
assert na.sum_range(0, 2) == 8
assert na.sum_range(1, 1) == 2                  # edge: single element
na.update(0, -10)
assert na.sum_range(0, 0) == -10 and na.sum_range(0, 2) == -3
```

Both operations are O(log n). Building by repeated `add` is O(n log n); an O(n) build
exists (propagate each node to its parent once) but is rarely needed.

### Worked problem: count inversions

**Problem.** Count pairs `i < j` with `nums[i] > nums[j]`.

**Approach.** Scan from left to right. For each value, the number of **earlier** values
greater than it is `(count so far) - (count of earlier values ≤ it)`. A Fenwick tree over
value ranks answers "how many earlier values ≤ x" in O(log n). Compress values to ranks
first so the tree size is n, not the value range.

```python
def count_inversions(nums):
    ranks = {v: r for r, v in enumerate(sorted(set(nums)))}
    fw = Fenwick(len(ranks))
    inversions = 0
    for seen, x in enumerate(nums):
        r = ranks[x]
        inversions += seen - fw.prefix(r + 1)    # earlier values with rank > r
        fw.add(r, 1)
    return inversions


assert count_inversions([2, 4, 1, 3, 5]) == 3      # (2,1), (4,1), (4,3)
assert count_inversions([5, 4, 3, 2, 1]) == 10
assert count_inversions([1, 1, 1]) == 0            # equal values are not inversions
assert count_inversions([]) == 0
```

O(n log n). (Merge sort can count inversions in the same bound; the Fenwick version
generalizes to "count of smaller numbers after self" and similar queries.)

## Segment tree

A segment tree is a binary tree where each node stores an aggregate (sum, min, max, gcd —
any **associative** operation) over a contiguous range; leaves are single elements. An
iterative array-based version is compact and fast:

```python
class SegmentTree:
    def __init__(self, nums, op=min, identity=float("inf")):
        self.n = len(nums)
        self.op, self.identity = op, identity
        self.tree = [identity] * (2 * self.n)
        self.tree[self.n:] = nums                  # leaves at n..2n-1
        for i in range(self.n - 1, 0, -1):
            self.tree[i] = op(self.tree[2 * i], self.tree[2 * i + 1])

    def update(self, i, value):
        i += self.n
        self.tree[i] = value
        while i > 1:
            i //= 2
            self.tree[i] = self.op(self.tree[2 * i], self.tree[2 * i + 1])

    def query(self, left, right):                  # inclusive, 0-based
        res = self.identity
        lo, hi = left + self.n, right + self.n + 1
        while lo < hi:
            if lo & 1:
                res = self.op(res, self.tree[lo]); lo += 1
            if hi & 1:
                hi -= 1; res = self.op(res, self.tree[hi])
            lo //= 2; hi //= 2
        return res


st = SegmentTree([5, 2, 8, 6, 3, 7])
assert st.query(0, 5) == 2
assert st.query(2, 4) == 3
st.update(1, 9)
assert st.query(0, 2) == 5
assert st.query(3, 3) == 6                         # single element
sums = SegmentTree([1, 2, 3, 4], op=lambda a, b: a + b, identity=0)
assert sums.query(1, 3) == 9
sums.update(0, 10)
assert sums.query(0, 3) == 19
```

This version works for any `n` (not just powers of two) with a commutative operation
such as min, max or sum. For non-commutative operations, keep separate left and right
accumulators.

### Checking against brute force

When implementing these structures under pressure, a quick randomized comparison with a
naive version catches off-by-one errors fast:

```python
import random

random.seed(7)
arr = [random.randint(-50, 50) for _ in range(40)]
st = SegmentTree(arr, op=max, identity=float("-inf"))
fw = Fenwick(len(arr))
for i, x in enumerate(arr):
    fw.add(i, x)
for _ in range(300):
    if random.random() < 0.4:
        i, v = random.randrange(len(arr)), random.randint(-50, 50)
        fw.add(i, v - arr[i])
        arr[i] = v
        st.update(i, v)
    else:
        l = random.randrange(len(arr)); r = random.randrange(l, len(arr))
        assert st.query(l, r) == max(arr[l:r + 1])
        assert fw.range_sum(l, r) == sum(arr[l:r + 1])
```

## How It Actually Works

**Fenwick trees and binary decomposition.** Any prefix length `i` can be written as a sum
of distinct powers of two (its binary digits). The Fenwick tree stores one precomputed
block for each: stripping the lowest set bit of `i` repeatedly visits at most log₂ n
blocks that exactly tile `[1, i]`. For example, `prefix(13)` with `13 = 1101₂` visits
indices 13 (block of length 1), 12 (length 4) and 8 (length 8): 1 + 4 + 8 = 13 elements.
An update to position `p` must touch every block that *contains* `p`; adding the lowest
set bit walks exactly those, again at most log₂ n of them. The structure only needs
invertible operations for range queries (it computes `prefix(r) - prefix(l)`), which is
why it handles sums but not range minimum.

**Segment trees and range decomposition.** Any range `[l, r]` can be split into at most
about 2 log₂ n **canonical** nodes — at most two per level of the tree. The iterative
query finds them bottom-up: when `lo` is a right child, its node is fully inside the
range and cannot be merged with its sibling, so it is taken and `lo` steps right; the
same logic applies mirrored for `hi`. Then both move up a level. An update changes one
leaf and recomputes its log₂ n ancestors. Because the segment tree combines disjoint
pieces rather than subtracting prefixes, it works for **any associative** operation —
min, max, gcd, even matrix products — at the cost of about 2n storage.

**Lazy propagation** (not coded here) extends segment trees to range *updates*: instead
of updating every leaf in a range, you tag the O(log n) canonical nodes with a pending
update and push it to children only when a later query needs to look inside.

## Common mistakes

- Mixing 0-based and 1-based indices in a Fenwick tree (the most common bug).
- Using a Fenwick tree for range min/max with arbitrary updates — it does not support it.
- Forgetting to update the stored original array when converting "assign" into "add
  delta".
- Not compressing coordinates when values are large (10⁹) — the tree would be enormous.
- Choosing a segment tree when there are no updates (prefix sums are simpler and O(1)).

## Variations to practice

- Count of smaller numbers after self (Fenwick over ranks, scanning right to left).
- Range sum query 2-D, mutable (2-D Fenwick).
- Reverse pairs (`nums[i] > 2 * nums[j]`).
- My calendar with k bookings (segment tree with lazy propagation, or a sweep map).
- Longest increasing subsequence with values as indices (segment tree of max).

## Exercise

Solve **Count of Smaller Numbers After Self**: for each index `i`, return how many
elements to its right are smaller than `nums[i]`. Scan from right to left with a Fenwick
tree over compressed ranks. Test with `[5,2,6,1]` → `[2,1,1,0]`, `[-1]` → `[0]`,
`[-1,-1]` → `[0,0]`, and add a randomized check against the O(n²) brute force.
