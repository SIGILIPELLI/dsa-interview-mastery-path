---
description: "Learn dynamic programming from scratch: define states and transitions, memoize then tabulate, and solve climbing stairs, house robber, coin change and LCS."
---

# 06 · Intro to Dynamic Programming

Dynamic programming (DP) has a reputation for being hard, mostly because it is often
taught as a bag of tricks. It is really one idea: **if a problem can be solved from
answers to smaller versions of itself, and those smaller versions repeat, compute each
one once and reuse it.**

Two conditions tell you DP applies:

1. **Optimal substructure** — the answer for a state can be built from answers for
   smaller states.
2. **Overlapping subproblems** — the same smaller states are needed again and again
   (otherwise plain recursion, or divide and conquer, is enough).

**Signals:** "number of ways", "minimum/maximum cost", "is it possible", "longest ...",
over a sequence, grid, or budget — especially when greedy choices can be shown to fail.

## A four-step recipe

1. **State:** what does `dp[i]` (or `dp[i][j]`) mean, in words? Write the sentence down.
2. **Transition:** how is `dp[i]` computed from smaller states?
3. **Base cases:** the smallest states you can answer directly.
4. **Order and answer:** compute states so dependencies are ready; say which state holds
   the final answer.

Most DP bugs come from skipping step 1. If you cannot say what a state means in one
sentence, the transition will be wrong.

## Worked problem 1: climbing stairs

**Problem.** You climb 1 or 2 steps at a time. How many distinct ways reach step `n`?

- **State:** `ways(i)` = number of ways to reach step `i`.
- **Transition:** the last move was 1 step (from `i-1`) or 2 steps (from `i-2`), so
  `ways(i) = ways(i-1) + ways(i-2)`.
- **Base:** `ways(0) = 1` (one way: do nothing), `ways(1) = 1`.

Top-down (memoization) is the recursion written straight from the recurrence:

```python
from functools import lru_cache

def climb_stairs_memo(n):
    @lru_cache(maxsize=None)
    def ways(i):
        if i <= 1:
            return 1
        return ways(i - 1) + ways(i - 2)
    return ways(n)


assert climb_stairs_memo(2) == 2
assert climb_stairs_memo(5) == 8
assert climb_stairs_memo(0) == 1
```

Bottom-up (tabulation) fills the table from small to large. Since each state needs only
the previous two, keep two variables — O(1) space:

```python
def climb_stairs(n):
    prev2, prev1 = 1, 1                  # ways(i-2), ways(i-1)
    for _ in range(2, n + 1):
        prev2, prev1 = prev1, prev1 + prev2
    return prev1


assert [climb_stairs(n) for n in range(6)] == [1, 1, 2, 3, 5, 8]
```

## Worked problem 2: house robber

**Problem.** Houses in a row hold `nums[i]` money. You cannot rob two adjacent houses.
Maximize the total.

- **State:** `best(i)` = max money from houses `0..i`.
- **Transition:** either skip house `i` (`best(i-1)`) or rob it (`nums[i] + best(i-2)`).
  `best(i) = max(best(i-1), nums[i] + best(i-2))`.
- **Base:** `best(-1) = best(-2) = 0` (no houses).

```python
def rob(nums):
    prev2 = prev1 = 0                       # best(i-2), best(i-1)
    for x in nums:
        prev2, prev1 = prev1, max(prev1, x + prev2)
    return prev1


assert rob([1, 2, 3, 1]) == 4               # 1 + 3
assert rob([2, 7, 9, 3, 1]) == 12           # 2 + 9 + 1
assert rob([]) == 0                         # edge: no houses
assert rob([5]) == 5
assert rob([2, 1, 1, 2]) == 4               # greedy "take every other" fails here
```

The last test is why greedy fails: taking indices 0 and 3 (non-alternating) is optimal.

## Worked problem 3: coin change (minimum coins)

**Problem.** Given coin denominations and an `amount`, return the fewest coins that make
the amount, or -1. Coins can be reused.

**Why not greedy?** With coins `[1, 3, 4]` and amount 6, greedy takes 4 + 1 + 1 (3
coins), but 3 + 3 uses 2.

- **State:** `dp[a]` = fewest coins to make amount `a`.
- **Transition:** `dp[a] = 1 + min(dp[a - c])` over coins `c ≤ a`.
- **Base:** `dp[0] = 0`. Unreachable amounts stay at infinity.

```python
def coin_change(coins, amount):
    INF = float("inf")
    dp = [0] + [INF] * amount
    for a in range(1, amount + 1):
        for c in coins:
            if c <= a and dp[a - c] + 1 < dp[a]:
                dp[a] = dp[a - c] + 1
    return dp[amount] if dp[amount] != INF else -1


assert coin_change([1, 3, 4], 6) == 2
assert coin_change([1, 2, 5], 11) == 3            # 5 + 5 + 1
assert coin_change([2], 3) == -1                  # edge: impossible
assert coin_change([1], 0) == 0                   # edge: zero amount
```

**Complexity:** O(amount × len(coins)) time, O(amount) space.

## Worked problem 4: longest common subsequence (2-D DP)

**Problem.** Length of the longest subsequence common to strings `a` and `b` (characters
in order, not necessarily contiguous). `"abcde"`, `"ace"` → 3.

- **State:** `dp[i][j]` = LCS length of prefixes `a[:i]` and `b[:j]`.
- **Transition:** if `a[i-1] == b[j-1]`, extend: `dp[i-1][j-1] + 1`. Otherwise drop one
  character from either string: `max(dp[i-1][j], dp[i][j-1])`.
- **Base:** `dp[0][*] = dp[*][0] = 0` (empty prefix).

```python
def lcs(a, b):
    m, n = len(a), len(b)
    dp = [[0] * (n + 1) for _ in range(m + 1)]
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if a[i - 1] == b[j - 1]:
                dp[i][j] = dp[i - 1][j - 1] + 1
            else:
                dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])
    return dp[m][n]


assert lcs("abcde", "ace") == 3
assert lcs("abc", "def") == 0
assert lcs("", "abc") == 0                       # edge: empty string
assert lcs("aaaa", "aa") == 2
```

O(m · n) time and space; since row `i` only needs row `i-1`, space can drop to O(n).
Using `(m+1) × (n+1)` with an empty-prefix row and column removes all boundary special
cases — a habit worth keeping.

## How It Actually Works

**Why DP turns exponential into polynomial.** The naive recursion for climbing stairs
explores a binary tree of calls with about φⁿ nodes (φ ≈ 1.618), but there are only n+1
*distinct* states. DP's running time is simply **(number of states) × (work per
transition)**. For coin change that is `amount` states × `len(coins)` work; for LCS it is
`m·n` states × O(1). Counting states and transitions this way is how you state the
complexity of any DP solution.

**Memoization vs tabulation.** Both compute each state once.

- Top-down recursion only visits states that are actually reachable from the start,
  which can be far fewer than the full table, and the code mirrors the recurrence. But
  it pays function-call overhead and, in Python, risks `RecursionError` when the state
  chain is deep (e.g. `amount = 10⁴`).
- Bottom-up iterates in an order where dependencies are already filled (increasing `i`
  here). No recursion limit, usually faster, and it exposes which previous states are
  needed — which is how you spot the rolling-variable space optimizations above.

**The dependency graph.** States and transitions form a directed acyclic graph: an edge
from each state to the states it depends on. Tabulation is just evaluating that DAG in
topological order. If the "dependencies" ever form a cycle, the problem is not a DP as
posed — you need a different state definition (or a shortest-path algorithm).

## Common mistakes

- Vague state definitions ("dp[i] is the answer for i") — say *exactly* what i covers.
- Off-by-one between the table index and string index (`dp[i]` for prefix length `i`
  uses `a[i-1]`).
- Initializing a minimization table with 0 instead of infinity.
- Building a 2-D list with `[[0] * n] * m` — all rows are the same list object.
- Assuming a greedy choice works without a proof; try a small counterexample first.

## Variations to practice

- Min cost climbing stairs.
- House robber II (houses in a circle: run the line version twice).
- Decode ways (`"226"` → 3).
- Unique paths in a grid (with and without obstacles).
- Word break (can a string be segmented into dictionary words?).
- Edit distance (Level 3, lesson 1 extends LCS-style tables).

## Exercise

Solve **Word Break**: given a string `s` and a list of words, can `s` be split into a
sequence of dictionary words? Define `dp[i]` = "can the prefix `s[:i]` be segmented",
write the transition, and implement it bottom-up with a set for the dictionary. Test
`"leetcode"` with `["leet","code"]`, `"applepenapple"` with `["apple","pen"]`,
`"catsandog"` with `["cats","dog","sand","and","cat"]` (False), and the empty string.
State the complexity in terms of `len(s)` and the maximum word length.
