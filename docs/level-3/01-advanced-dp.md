---
description: "Advanced dynamic programming in Python: 0/1 knapsack, longest increasing subsequence in O(n log n), edit distance, grid paths and interval DP."
---

# 01 · Advanced Dynamic Programming

Level 2 introduced DP with one-dimensional states. Harder DP problems differ mainly in
**how the state is chosen**. This lesson covers the families that recur most often; for
each, focus on the sentence that defines the state — the code follows from it.

| Family | Typical state | Examples |
|---|---|---|
| Knapsack | items considered × capacity used | subset sum, partition equal subset, target sum |
| Subsequence | index (+ last value) | longest increasing subsequence, number of LIS |
| Two sequences | prefix of A × prefix of B | edit distance, LCS, interleaving strings |
| Grid | row × column | minimum path sum, maximal square |
| Interval | left end × right end | burst balloons, palindrome partitioning, matrix-chain |

## 0/1 knapsack

**Problem.** Items have weights and values; choose a subset with total weight ≤ `W`
maximizing total value. Each item is used at most once.

- **State:** `dp[w]` = best value achievable with capacity `w` using the items processed
  so far.
- **Transition (item with weight `wt`, value `val`):** `dp[w] = max(dp[w], dp[w - wt] + val)`.
- **Order:** iterate `w` **downward** so `dp[w - wt]` still refers to the previous item
  set — each item used at most once.

```python
def knapsack(weights, values, W):
    dp = [0] * (W + 1)
    for wt, val in zip(weights, values):
        for w in range(W, wt - 1, -1):         # downward: 0/1 semantics
            dp[w] = max(dp[w], dp[w - wt] + val)
    return dp[W]


assert knapsack([1, 3, 4, 5], [1, 4, 5, 7], 7) == 9       # items of weight 3 and 4
assert knapsack([5], [10], 4) == 0                         # edge: nothing fits
assert knapsack([], [], 10) == 0
assert knapsack([2, 2, 2], [3, 3, 3], 4) == 6
```

Iterating **upward** instead gives the *unbounded* knapsack (unlimited copies) — the
same loop order as coin change. That one-word difference is a favourite follow-up.

**Complexity:** O(n · W) time, O(W) space. This is *pseudo-polynomial*: polynomial in the
numeric value `W`, not in the number of bits needed to write it. Knapsack is NP-hard in
general; the DP is fast only when `W` is modest.

### Knapsack in disguise: partition equal subset sum

```python
def can_partition(nums):
    total = sum(nums)
    if total % 2:
        return False
    target = total // 2
    reachable = [True] + [False] * target
    for x in nums:
        for s in range(target, x - 1, -1):
            reachable[s] = reachable[s] or reachable[s - x]
    return reachable[target]


assert can_partition([1, 5, 11, 5])            # [1, 5, 5] and [11]
assert not can_partition([1, 2, 3, 5])
assert can_partition([])                       # edge: two empty halves
assert not can_partition([2])
```

## Longest increasing subsequence

**O(n²) DP.** `dp[i]` = length of the longest strictly increasing subsequence ending at
`i` = `1 + max(dp[j])` over `j < i` with `nums[j] < nums[i]`.

**O(n log n) with patience sorting.** Maintain `tails`, where `tails[L]` is the
**smallest possible tail value** of an increasing subsequence of length `L + 1` seen so
far. For each number, binary-search the first tail ≥ it and replace it (or append if
none).

```python
from bisect import bisect_left

def length_of_lis(nums):
    tails = []
    for x in nums:
        i = bisect_left(tails, x)       # first tail >= x (strictly increasing)
        if i == len(tails):
            tails.append(x)
        else:
            tails[i] = x
    return len(tails)


assert length_of_lis([10, 9, 2, 5, 3, 7, 101, 18]) == 4    # 2, 3, 7, 18
assert length_of_lis([0, 1, 0, 3, 2, 3]) == 4
assert length_of_lis([7, 7, 7]) == 1                       # strictly increasing
assert length_of_lis([]) == 0
```

`tails` is always sorted, which is what makes the binary search valid. It is **not** an
actual subsequence — only its length is meaningful. Use `bisect_right` for
*non-decreasing* subsequences.

## Edit distance (two sequences)

**Problem.** Minimum insertions, deletions and substitutions to turn `a` into `b`.

- **State:** `dp[i][j]` = edit distance between `a[:i]` and `b[:j]`.
- **Transition:** if `a[i-1] == b[j-1]`, `dp[i-1][j-1]`. Otherwise
  `1 + min(dp[i-1][j] (delete), dp[i][j-1] (insert), dp[i-1][j-1] (substitute))`.
- **Base:** `dp[i][0] = i`, `dp[0][j] = j`.

```python
def edit_distance(a, b):
    m, n = len(a), len(b)
    prev = list(range(n + 1))                  # row i-1
    for i in range(1, m + 1):
        cur = [i] + [0] * n
        for j in range(1, n + 1):
            if a[i - 1] == b[j - 1]:
                cur[j] = prev[j - 1]
            else:
                cur[j] = 1 + min(prev[j], cur[j - 1], prev[j - 1])
        prev = cur
    return prev[n]


assert edit_distance("horse", "ros") == 3
assert edit_distance("intention", "execution") == 5
assert edit_distance("", "abc") == 3                       # edge: all inserts
assert edit_distance("same", "same") == 0
```

O(m · n) time, O(n) space with two rows.

## Grid DP: maximal square

**Problem.** Largest square of 1s in a binary matrix; return its area.

**State:** `dp[r][c]` = side of the largest all-1 square whose **bottom-right corner** is
`(r, c)`. It is limited by the squares ending above, left, and diagonally up-left:
`1 + min(up, left, diag)`.

```python
def maximal_square(matrix):
    if not matrix:
        return 0
    cols = len(matrix[0])
    dp = [0] * (cols + 1)
    best = 0
    for row in matrix:
        diag = 0                                  # dp value up-left of current cell
        for c in range(1, cols + 1):
            up = dp[c]
            if row[c - 1] == "1":
                dp[c] = 1 + min(up, dp[c - 1], diag)
                best = max(best, dp[c])
            else:
                dp[c] = 0
            diag = up
    return best * best


m = [list("10100"), list("10111"), list("11111"), list("10010")]
assert maximal_square(m) == 4
assert maximal_square([list("0")]) == 0
assert maximal_square([list("11"), list("11")]) == 4
```

## Interval DP: burst balloons

**Problem.** Balloons `nums[i]`; bursting `i` earns `left * nums[i] * right` using its
current neighbours (outside the array counts as 1). Maximize the total.

**The key reframe.** Choosing which balloon to burst *first* splits badly, because
neighbours change. Instead choose which balloon in the interval `(l, r)` to burst
**last**: when `k` is last, its neighbours are exactly the boundaries `l` and `r`, and
the two sides are independent subproblems.

- **State:** `dp[l][r]` = max coins from bursting all balloons strictly between `l` and
  `r` (with padding 1s at both ends).
- **Transition:** `max over k in (l, r) of dp[l][k] + vals[l]*vals[k]*vals[r] + dp[k][r]`.
- **Order:** by increasing interval length.

```python
def max_coins(nums):
    vals = [1] + nums + [1]
    n = len(vals)
    dp = [[0] * n for _ in range(n)]
    for length in range(2, n):                   # r - l
        for l in range(0, n - length):
            r = l + length
            dp[l][r] = max(dp[l][k] + vals[l] * vals[k] * vals[r] + dp[k][r]
                           for k in range(l + 1, r))
    return dp[0][n - 1]


assert max_coins([3, 1, 5, 8]) == 167
assert max_coins([1, 5]) == 10
assert max_coins([]) == 0                        # edge: nothing to burst
assert max_coins([7]) == 7
```

O(n³) time, O(n²) space.

## How It Actually Works

**Why the downward loop enforces "use once".** In the 1-D knapsack array, cells with
smaller `w` are read to compute larger `w`. Going downward, when you compute `dp[w]`, the
cell `dp[w - wt]` has not yet been updated for the current item — it still describes the
previous item set, so the current item can contribute at most once. Going upward,
`dp[w - wt]` may already include the current item, allowing reuse. The 1-D array is a
compressed 2-D table `dp[item][w]`, and the loop direction decides which row you read.

**Why patience sorting works.** Invariant: `tails[L]` is the smallest tail among all
increasing subsequences of length `L + 1` so far, and `tails` is strictly increasing
(a length-`L+2` subsequence contains a length-`L+1` one with a smaller tail). When `x`
arrives, the first `tails[i] ≥ x` is the longest subsequence that `x` *cannot* extend;
`x` extends the one of length `i` and becomes a better (smaller) tail for length
`i + 1`. Replacing preserves the invariant, and appending happens exactly when `x`
extends the longest subsequence so far. Each step is a binary search: O(n log n).

**Choosing the state is the whole game.** Interval DP works because "last to burst"
makes subproblems independent; "first to burst" does not. Whenever a transition seems to
need information that is not in your state (who the current neighbours are, what the
last value was), either add it to the state or reframe the decision so the dependence
disappears.

## Common mistakes

- Upward iteration in 0/1 knapsack (silently allows reuse).
- Treating `tails` in LIS as the actual subsequence.
- `bisect_left` vs `bisect_right` for strict vs non-strict increase.
- Choosing "first" instead of "last" in interval DP and getting dependent subproblems.
- Iterating interval DP by `l` from 0 upward instead of by length — you read states not
  yet computed.

## Variations to practice

- Target sum (assign + or − to reach a target: knapsack on `(total + target) / 2`).
- Number of longest increasing subsequences; Russian doll envelopes (sort + LIS).
- Longest palindromic subsequence (interval DP, or LCS with the reversed string).
- Minimum path sum; dungeon game (DP from the bottom-right).
- Best time to buy and sell stock with cooldown (state machine DP).

## Exercise

Solve **Target Sum**: given `nums` and `target`, count the ways to assign `+` or `-` to
each number so the expression equals `target`. Show algebraically that it reduces to
counting subsets with sum `(sum(nums) + target) / 2`, handle the cases where that value
is negative or non-integer, and implement the counting knapsack with a downward loop.
Test with `[1,1,1,1,1], 3` → 5, `[1], 1` → 1, `[1], 2` → 0, and `[0,0], 0` → 4 (zeros
double the count — make sure your loop handles them).
