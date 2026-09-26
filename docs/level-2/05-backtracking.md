---
description: "Solve search problems with backtracking: the choose-explore-undo template, combination sum, permutations with duplicates, word search and N-Queens."
---

# 05 · Backtracking

Backtracking is organized brute force. You build a candidate solution one decision at a
time; after each decision you recurse; when you return, you **undo** the decision and try
the next option. When a partial candidate can no longer lead to a valid solution, you
stop exploring that branch early — that is **pruning**, and it is where all the
performance comes from.

**Signals:** "return all ...", "generate every ...", "find any arrangement that ...",
combinations, permutations, subsets, partitions, placing pieces on a board, and
constraints small enough (n ≲ 15–20) that exponential time is acceptable.

## The template

```text
def backtrack(state):
    if state is a complete solution:
        record a copy of it
        return
    for choice in available choices:
        if choice is not allowed: continue      # prune
        make the choice (modify state)
        backtrack(state)
        undo the choice (restore state)
```

Three questions define any backtracking solution: **what is the state**, **what are the
choices at each step**, and **when is a candidate complete**?

## Worked problem 1: combination sum

**Problem.** Given distinct positive `candidates` and a `target`, return all unique
combinations that sum to `target`. Each candidate may be used unlimited times.
`[2,3,6,7], 7` → `[[2,2,3],[7]]`.

**Avoiding duplicates.** `[2,2,3]` and `[3,2,2]` are the same combination. Enforce a
canonical order: only choose candidates at index ≥ the last one chosen.

```python
def combination_sum(candidates, target):
    candidates = sorted(candidates)
    result, path = [], []

    def backtrack(start, remaining):
        if remaining == 0:
            result.append(path.copy())
            return
        for i in range(start, len(candidates)):
            c = candidates[i]
            if c > remaining:
                break                        # prune: sorted, so all later c are too big
            path.append(c)
            backtrack(i, remaining - c)      # i, not i+1: reuse allowed
            path.pop()

    backtrack(0, target)
    return result


assert combination_sum([2, 3, 6, 7], 7) == [[2, 2, 3], [7]]
assert combination_sum([2, 3, 5], 8) == [[2, 2, 2, 2], [2, 3, 3], [3, 5]]
assert combination_sum([2], 1) == []                  # edge: impossible
assert combination_sum([1], 2) == [[1, 1]]
```

Sorting enables the `break`: once one candidate is too large, every later one is too.

## Worked problem 2: permutations with duplicates

**Problem.** Return all **unique** permutations of a list that may contain duplicates.

**Approach.** Sort, track used indices, and apply the classic rule: skip `nums[i]` if it
equals `nums[i-1]` and `nums[i-1]` is **not** currently used. This forces equal values to
be placed in their original relative order, so each distinct arrangement is produced
once.

```python
def permute_unique(nums):
    nums = sorted(nums)
    used = [False] * len(nums)
    result, path = [], []

    def backtrack():
        if len(path) == len(nums):
            result.append(path.copy())
            return
        for i in range(len(nums)):
            if used[i]:
                continue
            if i > 0 and nums[i] == nums[i - 1] and not used[i - 1]:
                continue                                  # duplicate branch
            used[i] = True
            path.append(nums[i])
            backtrack()
            path.pop()
            used[i] = False

    backtrack()
    return result


assert permute_unique([1, 1, 2]) == [[1, 1, 2], [1, 2, 1], [2, 1, 1]]
assert len(permute_unique([1, 2, 3])) == 6
assert permute_unique([]) == [[]]                   # edge: one empty permutation
assert permute_unique([5, 5, 5]) == [[5, 5, 5]]
```

## Worked problem 3: word search on a grid

**Problem.** Does `word` exist in a grid of letters, formed by adjacent (not diagonal)
cells, each used at most once?

```python
def exist(board, word):
    rows, cols = len(board), len(board[0])

    def backtrack(r, c, k):
        if k == len(word):
            return True
        if not (0 <= r < rows and 0 <= c < cols) or board[r][c] != word[k]:
            return False
        saved, board[r][c] = board[r][c], "#"          # mark as used
        found = (backtrack(r + 1, c, k + 1) or backtrack(r - 1, c, k + 1) or
                 backtrack(r, c + 1, k + 1) or backtrack(r, c - 1, k + 1))
        board[r][c] = saved                            # undo
        return found

    return any(backtrack(r, c, 0) for r in range(rows) for c in range(cols))


board = [list("ABCE"), list("SFCS"), list("ADEE")]
assert exist(board, "ABCCED")
assert exist(board, "SEE")
assert not exist(board, "ABCB")                  # would reuse B
assert exist([list("A")], "A")                   # edge: 1x1
assert board == [list("ABCE"), list("SFCS"), list("ADEE")]   # grid restored
```

**Complexity:** O(R · C · 3ᴸ) for word length L — from each start, each step has at most
3 new directions (not back where it came from). Marking cells in place avoids a separate
visited set.

## Worked problem 4: N-Queens

**Problem.** Place n queens on an n × n board so none attack each other. Return the
number of solutions.

**State:** one queen per row, so choose a column for each row in turn. Track attacked
columns and both diagonals in sets: cells on the same `\` diagonal share `r - c`; on the
same `/` diagonal they share `r + c`.

```python
def total_n_queens(n):
    cols, diag1, diag2 = set(), set(), set()

    def place(r):
        if r == n:
            return 1
        count = 0
        for c in range(n):
            if c in cols or (r - c) in diag1 or (r + c) in diag2:
                continue                                 # prune attacked squares
            cols.add(c); diag1.add(r - c); diag2.add(r + c)
            count += place(r + 1)
            cols.remove(c); diag1.remove(r - c); diag2.remove(r + c)
        return count

    return place(0)


assert [total_n_queens(n) for n in range(1, 9)] == [1, 0, 0, 2, 10, 4, 40, 92]
```

The sets make each safety check O(1) instead of scanning previous rows.

## How It Actually Works

Backtracking is a **depth-first traversal of a decision tree** that is never built in
memory. Each node is a partial solution; each edge is a choice; leaves are complete
candidates or dead ends. Only the current root-to-node path exists at any time (in
`path`, `used`, the sets), which is why memory is O(depth) even when the tree has
millions of nodes.

The "undo" step is what makes sharing one mutable `path` safe. Without undo you would
have to copy the state for every child — correct but far slower. With undo, the state
after returning from a child is exactly what it was before entering it, so siblings see a
clean slate.

Time complexity is bounded by the size of the tree: roughly (branching factor)^(depth)
nodes times the work per node. That is why it is exponential, and why pruning matters so
much: cutting a branch near the root removes an entire subtree. In N-Queens, the column
and diagonal checks cut the 8⁸ ≈ 16.7 million naive placements for n = 8 down to a
search that makes 2,057 calls to `place` in total (you can verify this by adding a
counter to the code above).

The duplicate-skipping rule in `permute_unique` works because, among equal values, it
only allows the leftmost unused copy to be placed next. Any two branches that would place
different copies of the same value at the same position are collapsed into one.

## Common mistakes

- Appending `path` instead of `path.copy()` — every stored result aliases the same list.
- Forgetting to undo (pop, unmark, remove from set), corrupting sibling branches.
- Wrong `start` index: `i + 1` means each element once; `i` means reuse allowed; `0`
  gives permutations, not combinations.
- Deduplicating results with a set at the end instead of skipping duplicate branches —
  you still pay for exploring them.
- Not sorting before using `break`-style pruning or duplicate skipping.

## Variations to practice

- Subsets II (with duplicates).
- Combination sum II (each number once, input has duplicates).
- Palindrome partitioning.
- Generate valid parentheses (track open/close counts).
- Sudoku solver.
- Letter combinations of a phone number.

## Exercise

Write `generate_parentheses(n)` that returns all valid strings of `n` pairs of
parentheses. Your choices at each step are `"("` (allowed while fewer than `n` opened)
and `")"` (allowed while fewer closed than opened). Assert the result for `n = 3` has
the 5 expected strings, `n = 1` gives `["()"]`, and `n = 0` gives `[""]`. Explain why
no invalid string is ever generated, so no final validity check is needed.
