---
description: "Five hard interview problems combining Level 3 techniques — DP, Dijkstra, stacks, heaps, greedy — each with a full walkthrough and tested solution."
---

# 10 · Project — Hard Problem Set

Hard problems are rarely a single new trick. They usually **combine** two ideas you
already know — a heap inside a graph search, a monotonic stack inside a DP, binary search
wrapped around a BFS. This set is designed to practice that combination step.

## Rules

1. Allow **45 minutes per problem**; do them on separate days if you prefer.
2. Spend the first 10 minutes without code: write a brute force, its complexity, and why
   it is too slow for the stated constraints. Then look for the bottleneck.
3. When you have an idea, test it by hand on the example before coding.
4. After solving (or at 45 minutes), read the walkthrough and write a short note: *which
   combination of techniques was it, and what was the clue?*

## The problems

1. **Trapping Rain Water.** Given bar heights, compute how much water is trapped after
   rain. `n ≤ 2·10⁴`.
2. **Swim in Rising Water.** An `n × n` grid of distinct elevations; at time `t` you can
   swim between adjacent cells if both elevations are ≤ `t`. Find the least `t` to get
   from top-left to bottom-right. `n ≤ 50`.
3. **Longest Valid Parentheses.** Length of the longest well-formed parentheses
   substring. `n ≤ 3·10⁴`.
4. **Sliding Window Median.** Return the median of every window of size `k`.
   `n ≤ 10⁵`.
5. **Minimum Number of Refueling Stops.** A car starts with `start_fuel` and must travel
   `target` miles; stations `[position, fuel]` are sorted by position. Find the minimum
   stops, or -1.

---

## Walkthrough 1 — trapping rain water

**Brute force:** for each index, water = `min(max left, max right) - height`, scanning
both sides: O(n²).

**Bottleneck:** recomputing maxima. **Idea 1:** precompute prefix and suffix maxima
(Level 1's two-pass technique): O(n) time, O(n) space. **Idea 2:** two pointers with
running maxima, O(1) space. At each step, the side with the smaller running max is the
limiting side, so its water is fully determined.

```python
def trap(height):
    left, right = 0, len(height) - 1
    left_max = right_max = water = 0
    while left < right:
        if height[left] < height[right]:
            left_max = max(left_max, height[left])
            water += left_max - height[left]
            left += 1
        else:
            right_max = max(right_max, height[right])
            water += right_max - height[right]
            right -= 1
    return water


assert trap([0, 1, 0, 2, 1, 0, 1, 3, 2, 1, 2, 1]) == 6
assert trap([4, 2, 0, 3, 2, 5]) == 9
assert trap([]) == 0
assert trap([3, 2, 1]) == 0                     # descending: nothing trapped
```

**Why it works:** the water above index `left` is `min(true max on its left, true max
on its right) - height[left]`. The left side's true max is exactly `left_max`. For the
right side, notice that `left_max` was set by some bar `p ≤ left`, and the left pointer
only ever moves when its bar is **strictly shorter** than the current right bar. So
`height[p]` is less than some bar at or right of the current `right` — meaning the true
right max is at least `left_max`, and the minimum is `left_max`. The mirror argument covers
the right pointer. **Combination:** two pointers + prefix/suffix maxima.

## Walkthrough 2 — swim in rising water

**Reframe:** minimize the **maximum** elevation along a path. That is a "minimax path",
solvable by Dijkstra where a path's cost is the max of its cells rather than a sum
(Level 3, lesson 2 exercise). Alternatives: binary search on `t` with a BFS check, or
union-find adding cells in elevation order until the corners connect.

```python
import heapq

def swim_in_water(grid):
    n = len(grid)
    best = [[float("inf")] * n for _ in range(n)]
    best[0][0] = grid[0][0]
    heap = [(grid[0][0], 0, 0)]
    while heap:
        t, r, c = heapq.heappop(heap)
        if (r, c) == (n - 1, n - 1):
            return t
        if t > best[r][c]:
            continue
        for nr, nc in ((r + 1, c), (r - 1, c), (r, c + 1), (r, c - 1)):
            if 0 <= nr < n and 0 <= nc < n:
                nt = max(t, grid[nr][nc])
                if nt < best[nr][nc]:
                    best[nr][nc] = nt
                    heapq.heappush(heap, (nt, nr, nc))
    return -1


assert swim_in_water([[0, 2], [1, 3]]) == 3
assert swim_in_water([[0, 1, 2, 3, 4], [24, 23, 22, 21, 5], [12, 13, 14, 15, 16],
                      [11, 17, 18, 19, 20], [10, 9, 8, 7, 6]]) == 16
assert swim_in_water([[5]]) == 5                         # edge: 1x1
```

O(n² log n). **Combination:** Dijkstra + a non-additive path cost. The greedy argument
still holds because extending a path never lowers its maximum.

## Walkthrough 3 — longest valid parentheses

**Brute force:** check every substring: O(n³), or O(n²) with incremental balance.

**Stack idea:** keep a stack of indices, seeded with `-1` as the base "last unmatched
position". Push `(` indices. On `)`, pop; if the stack becomes empty, this `)` is
unmatched — push its index as the new base. Otherwise the current valid run extends from
the new top + 1 to here.

```python
def longest_valid_parentheses(s):
    stack = [-1]
    best = 0
    for i, ch in enumerate(s):
        if ch == "(":
            stack.append(i)
        else:
            stack.pop()
            if not stack:
                stack.append(i)                  # new base after an unmatched ')'
            else:
                best = max(best, i - stack[-1])
    return best


assert longest_valid_parentheses("(()") == 2
assert longest_valid_parentheses(")()())") == 4
assert longest_valid_parentheses("") == 0
assert longest_valid_parentheses("()(())") == 6          # adjacent groups join
assert longest_valid_parentheses("((((") == 0
```

O(n). **Combination:** stack of indices + a sentinel base (the same "boundary below the
popped element" idea as the histogram in lesson 5). A DP solution also exists:
`dp[i]` = longest valid substring ending at `i`.

## Walkthrough 4 — sliding window median

**Brute force:** sort each window: O(n · k log k).

**Idea:** extend the two-heap running median (Level 2, lesson 3) with **removals**. Heaps
cannot delete arbitrary elements, so use **lazy deletion**: record values that should be
removed in a counter, track each heap's *valid* size separately, and discard invalid tops
only when they surface.

```python
import heapq
from collections import defaultdict

def median_sliding_window(nums, k):
    low, high = [], []              # max-heap (negated), min-heap
    delayed = defaultdict(int)      # value -> pending removals
    sizes = [0, 0]                  # valid sizes of low, high

    def prune(heap, sign):
        while heap and delayed[sign * heap[0]]:
            delayed[sign * heap[0]] -= 1
            heapq.heappop(heap)

    def rebalance():
        if sizes[0] > sizes[1] + 1:
            heapq.heappush(high, -heapq.heappop(low))
            sizes[0] -= 1; sizes[1] += 1
            prune(low, -1)
        elif sizes[0] < sizes[1]:
            heapq.heappush(low, -heapq.heappop(high))
            sizes[1] -= 1; sizes[0] += 1
            prune(high, 1)

    def add(x):
        if not low or x <= -low[0]:
            heapq.heappush(low, -x); sizes[0] += 1
        else:
            heapq.heappush(high, x); sizes[1] += 1
        rebalance()

    def remove(x):
        delayed[x] += 1
        if x <= -low[0]:
            sizes[0] -= 1
            if x == -low[0]:
                prune(low, -1)
        else:
            sizes[1] -= 1
            if high and x == high[0]:
                prune(high, 1)
        rebalance()

    def median():
        return float(-low[0]) if k % 2 else (-low[0] + high[0]) / 2

    out = []
    for i, x in enumerate(nums):
        add(x)
        if i >= k:
            remove(nums[i - k])
        if i >= k - 1:
            out.append(median())
    return out


def brute(nums, k):
    res = []
    for i in range(len(nums) - k + 1):
        w = sorted(nums[i:i + k])
        res.append(float(w[k // 2]) if k % 2 else (w[k // 2 - 1] + w[k // 2]) / 2)
    return res


assert median_sliding_window([1, 3, -1, -3, 5, 3, 6, 7], 3) == [1, -1, -1, 3, 5, 6]
assert median_sliding_window([1, 2, 3, 4, 2, 3, 1, 4, 2], 3) == brute([1, 2, 3, 4, 2, 3, 1, 4, 2], 3)
assert median_sliding_window([5], 1) == [5.0]
import random
random.seed(1)
for _ in range(200):
    arr = [random.randint(-5, 5) for _ in range(random.randint(1, 25))]
    kk = random.randint(1, len(arr))
    assert median_sliding_window(arr, kk) == brute(arr, kk)
```

O(n log n). This is the hardest implementation in the set; the randomized comparison with
a brute force is exactly how you should gain confidence in code like this. **Combination:**
two heaps + lazy deletion + balanced-size bookkeeping.

## Walkthrough 5 — minimum refueling stops

**Brute force / DP:** `dp[j]` = furthest distance reachable with `j` stops, O(n²).

**Greedy + heap:** drive as far as current fuel allows, remembering (in a max-heap) the
fuel of every station passed. When you cannot reach the next station or the target,
retroactively "stop" at the passed station with the most fuel. Taking the largest
available fuel is always at least as good as any other single stop.

```python
import heapq

def min_refuel_stops(target, start_fuel, stations):
    heap, fuel, stops, i = [], start_fuel, 0, 0
    while fuel < target:
        while i < len(stations) and stations[i][0] <= fuel:
            heapq.heappush(heap, -stations[i][1])     # reachable, remember it
            i += 1
        if not heap:
            return -1
        fuel += -heapq.heappop(heap)                  # stop at the best one
        stops += 1
    return stops


assert min_refuel_stops(1, 1, []) == 0                       # edge: enough already
assert min_refuel_stops(100, 1, [[10, 100]]) == -1           # cannot reach station
assert min_refuel_stops(100, 10, [[10, 60], [20, 30], [30, 30], [60, 40]]) == 2
assert min_refuel_stops(100, 50, [[25, 25], [50, 50]]) == 1
```

Here `fuel` doubles as "furthest reachable position" since the car starts at 0. O(n log n).
**Combination:** greedy with deferred decisions + max-heap.

## Reflection

Write your notes table: problem, techniques combined, clue. Typical clues from this set:

- "minimize the maximum along a path" → Dijkstra variant / binary search + BFS / DSU;
- "longest valid ..." with nesting → stack of indices with a base sentinel;
- "median" + "window" → two heaps + lazy deletion;
- "minimum number of stops" where you can decide later → greedy + heap of options.

## How It Actually Works

Hard problems are usually hard because **one known technique is not quite enough**, and
you must notice which extra property lets two techniques fit together. Dijkstra alone
sums edge weights; swim-in-water needs it to track a maximum instead, which works because
"max" never decreases as a path grows — the same property Dijkstra's greedy proof uses
for non-negative sums. A heap alone cannot delete arbitrary values; lazy deletion makes
it possible by postponing removals until they surface at the top, while separate valid-size
counters keep the balancing logic honest. Greedy alone would commit to refuelling
decisions too early; the heap lets you defer each decision until it is forced, at which
point taking the largest option is provably best.

The practical lesson is to ask, after writing a brute force: *what is the expensive
repeated operation, and what structure makes that exact operation cheap?* In each
walkthrough, the answer names the second technique. Randomized comparison against a
brute force (as in walkthrough 4) is the standard way to gain confidence in combined
solutions, because their bugs tend to appear only in specific interleavings of
operations that hand-written tests miss.

## Exercise

Re-derive the invariant for Walkthrough 1 in your own words, then trace the two-pointer
version on `[4, 2, 0, 3, 2, 5]` by hand, writing `left_max`, `right_max` and the water added
at each step. Then solve
**Trapping Rain Water II** (a 2-D elevation map): use a min-heap seeded with all boundary
cells and expand inward, always from the lowest boundary. Test on the 3×6 example
`[[1,4,3,1,3,2],[3,2,1,3,2,4],[2,3,3,2,3,1]]` → 4 and on a grid too small to hold water.
