---
description: "A timed set of six Level 2 problems — trees, heaps, graphs, backtracking, DP, intervals — with full Python walkthroughs and a self-review rubric."
---

# 10 · Project — Timed Practice Set 2

This set mixes every Level 2 pattern, and deliberately does **not** tell you which
pattern each problem uses. Recognizing the pattern from the problem statement is half of
what an interview tests.

## Rules

1. **120 minutes** for six problems (about 20 each). If stuck at 25 minutes, write your
   best partial idea as comments and move on.
2. Before coding each one, write a short plan: the pattern, the state or invariant, and
   the target time and space complexity.
3. Test each solution with at least three asserts, including one edge case.
4. Speak your reasoning out loud (or record yourself). You will review it afterwards.

## Self-review rubric

Score each problem 0–2 on each row (max 10 per problem, 60 total):

| Criterion | 0 | 1 | 2 |
|---|---|---|---|
| Pattern | Did not identify | Identified after coding started | Identified during planning |
| Correctness | Fails tests | Passes with bugs fixed late | Passes first run |
| Complexity | Not stated / wrong | Stated, minor errors | Correct time and space |
| Edge cases | None considered | Some | Empty/single/duplicate/extreme all considered |
| Communication | Silent coding | Explained after the fact | Explained plan before coding |

## The problems

1. **Level Averages.** Given a binary tree, return the average value of the nodes on each
   level.
2. **Course Order Possible?** There are `n` courses and prerequisite pairs `[a, b]`
   ("take b before a"). Can all courses be finished?
3. **Last Stone Weight.** Repeatedly smash the two heaviest stones; if they differ, the
   difference remains. Return the last stone's weight, or 0.
4. **Combination Sum III.** Find all combinations of `k` distinct numbers from 1–9 that
   sum to `n`.
5. **Unique Paths With Obstacles.** Count paths from top-left to bottom-right of a grid
   moving only right or down, avoiding cells marked 1.
6. **Employee Free Time (simplified).** Given each employee's busy intervals (each list
   sorted), return the finite intervals when everyone is free.

Stop here until the timer ends.

---

## Walkthrough 1 — level averages

**Pattern:** BFS by level (lesson 1). Snapshot the queue length to process one level at a
time.

```python
from collections import deque

class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val, self.left, self.right = val, left, right


def level_averages(root):
    if not root:
        return []
    out, queue = [], deque([root])
    while queue:
        size, total = len(queue), 0
        for _ in range(size):
            node = queue.popleft()
            total += node.val
            if node.left:
                queue.append(node.left)
            if node.right:
                queue.append(node.right)
        out.append(total / size)
    return out


root = TreeNode(3, TreeNode(9), TreeNode(20, TreeNode(15), TreeNode(7)))
assert level_averages(root) == [3.0, 14.5, 11.0]
assert level_averages(None) == []
assert level_averages(TreeNode(-5)) == [-5.0]
```

O(n) time, O(w) space for the widest level.

## Walkthrough 2 — can all courses be finished?

**Pattern:** cycle detection in a **directed** graph (lesson 4, previewing Level 3's
topological sort). Courses can all be taken exactly when the prerequisite graph has no
cycle. Use three colours: 0 = unvisited, 1 = on the current DFS path, 2 = finished. Meeting
a node coloured 1 means a back edge — a cycle.

```python
def can_finish(n, prerequisites):
    graph = [[] for _ in range(n)]
    for a, b in prerequisites:
        graph[b].append(a)
    state = [0] * n

    def has_cycle(u):
        state[u] = 1
        for v in graph[u]:
            if state[v] == 1:
                return True
            if state[v] == 0 and has_cycle(v):
                return True
        state[u] = 2
        return False

    return not any(state[u] == 0 and has_cycle(u) for u in range(n))


assert can_finish(2, [[1, 0]])
assert not can_finish(2, [[1, 0], [0, 1]])
assert can_finish(3, [])                          # edge: no prerequisites
assert not can_finish(1, [[0, 0]])                # edge: self-loop
assert can_finish(4, [[1, 0], [2, 0], [3, 1], [3, 2]])   # diamond, no cycle
```

A plain visited set is not enough: in the diamond case, node 3 is reached twice without
any cycle. The "on current path" state is what distinguishes a cycle from a re-merge.
O(V + E). (For very deep graphs, switch to Kahn's iterative algorithm from Level 3.)

## Walkthrough 3 — last stone weight

**Pattern:** max-heap (lesson 3), simulated with negated values.

```python
import heapq

def last_stone_weight(stones):
    heap = [-s for s in stones]
    heapq.heapify(heap)
    while len(heap) > 1:
        a = -heapq.heappop(heap)
        b = -heapq.heappop(heap)
        if a != b:
            heapq.heappush(heap, -(a - b))
    return -heap[0] if heap else 0


assert last_stone_weight([2, 7, 4, 1, 8, 1]) == 1
assert last_stone_weight([1]) == 1
assert last_stone_weight([3, 3]) == 0              # edge: all destroyed
assert last_stone_weight([]) == 0
```

O(n log n). Sorting the list after every smash would be O(n² log n).

## Walkthrough 4 — combination sum III

**Pattern:** backtracking with increasing choices (lesson 5). Prune when the running sum
exceeds `n` or the combination is already `k` long.

```python
def combination_sum3(k, n):
    result, path = [], []

    def backtrack(start, remaining):
        if len(path) == k:
            if remaining == 0:
                result.append(path.copy())
            return
        for x in range(start, 10):
            if x > remaining:
                break
            path.append(x)
            backtrack(x + 1, remaining - x)
            path.pop()

    backtrack(1, n)
    return result


assert combination_sum3(3, 7) == [[1, 2, 4]]
assert combination_sum3(3, 9) == [[1, 2, 6], [1, 3, 5], [2, 3, 4]]
assert combination_sum3(4, 1) == []                # edge: impossible
assert combination_sum3(9, 45) == [[1, 2, 3, 4, 5, 6, 7, 8, 9]]
```

The search space is at most C(9, k) ≤ 126 combinations, so this is effectively constant
time — worth saying, since the interviewer may ask about complexity.

## Walkthrough 5 — unique paths with obstacles

**Pattern:** 2-D DP (lesson 6). `dp[c]` = number of ways to reach the current row's cell
`c`. A cell's count is the sum of ways from above (the old `dp[c]`) and from the left
(`dp[c-1]`), or 0 if blocked. One row of storage suffices.

```python
def unique_paths_with_obstacles(grid):
    cols = len(grid[0])
    dp = [0] * cols
    dp[0] = 1
    for row in grid:
        for c in range(cols):
            if row[c] == 1:
                dp[c] = 0
            elif c > 0:
                dp[c] += dp[c - 1]
    return dp[-1]


assert unique_paths_with_obstacles([[0, 0, 0], [0, 1, 0], [0, 0, 0]]) == 2
assert unique_paths_with_obstacles([[0, 1], [0, 0]]) == 1
assert unique_paths_with_obstacles([[1]]) == 0                  # edge: start blocked
assert unique_paths_with_obstacles([[0, 0], [0, 1]]) == 0       # edge: end blocked
assert unique_paths_with_obstacles([[0] * 3 for _ in range(3)]) == 6
```

O(R · C) time, O(C) space. The start-blocked case works because the first row's
`dp[0] = 0` propagates.

## Walkthrough 6 — employee free time

**Pattern:** intervals (lesson 7). Flatten all busy intervals, merge them, and report the
gaps between consecutive merged blocks.

```python
def employee_free_time(schedules):
    busy = sorted(iv for person in schedules for iv in person)
    free, end = [], None
    for s, e in busy:
        if end is not None and s > end:
            free.append([end, s])
        end = e if end is None else max(end, e)
    return free


assert employee_free_time([[[1, 2], [5, 6]], [[1, 3]], [[4, 10]]]) == [[3, 4]]
assert employee_free_time([[[1, 3], [6, 7]], [[2, 4]], [[2, 5], [9, 12]]]) == \
    [[5, 6], [7, 9]]
assert employee_free_time([[[1, 5]]]) == []           # edge: one block, no gaps
assert employee_free_time([[[1, 2]], [[2, 3]]]) == [] # touching: no free time
```

O(N log N) for N total intervals. Because each person's list is already sorted, a k-way
heap merge (lesson 3) gives O(N log k) — a good follow-up to offer.

## After the set

Add up your rubric scores. Below 40 of 60: repeat the weakest two lessons and re-attempt
those problems in a week. 40–50: move on, but add the missed patterns to a spaced review
list. Above 50: you are ready for Level 3.

Look especially at the **Pattern** row. If you only recognized the pattern after coding
had started, practice "pattern-only drills": read ten problem statements and, for each,
write only the pattern and the key state or invariant in under two minutes, without
coding.

## How It Actually Works

This set interleaves six patterns on purpose. When you practise one topic at a time, you
already know which tool to use before you read the problem, so the hardest interview
skill — **choosing** the approach — never gets exercised. Mixed sets make practice feel
harder and slower, but that difficulty is exactly the selection step you need to train.

Each walkthrough also shows how a Level 2 pattern reduces to a small invariant: BFS by
levels depends on the queue holding exactly one level at the start of each loop; the
three-colour DFS depends on "on the current path" meaning "an ancestor in the recursion";
the one-row DP depends on `dp[c]` still holding the previous row's value until it is
overwritten. When a solution fails, checking which invariant broke is usually faster
than re-reading the code line by line. The rubric turns a vague "that went okay" into
per-dimension scores you can compare from week to week.

## Exercise

Pick the problem where you scored lowest. Write a **variation** of it that breaks your
solution (for example, unique paths where you may also move diagonally, or course
scheduling where you must output an order). Solve the variation, test it with at least
four asserts, and write two sentences on what changed in the state, invariant, or
complexity.
