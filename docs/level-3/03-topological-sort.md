---
description: "Order tasks with dependencies using Kahn's algorithm and DFS postorder, detect cycles, and run dynamic programming over a DAG such as longest path."
---

# 03 · Topological Sort

A **topological order** of a directed graph lists its vertices so that every edge
`u → v` has `u` before `v`. It exists exactly when the graph has **no directed cycle** —
a directed acyclic graph (DAG).

**Signals:** prerequisites, build orders, dependency resolution, "order of tasks",
spreadsheet recalculation, "is there a valid ordering?", and any DP whose states have a
dependency structure that is not a simple line or grid.

## Kahn's algorithm (BFS on in-degrees)

**Idea.** A vertex with no incoming edges can go first. Output it, remove its outgoing
edges (decrementing neighbours' in-degrees), and repeat with any vertex whose in-degree
has dropped to zero. If you output fewer than V vertices, the rest are stuck on a cycle.

### Worked problem: course schedule order

**Problem.** `n` courses, prerequisite pairs `[a, b]` meaning "take `b` before `a`".
Return a valid order, or an empty list if impossible.

```python
from collections import deque

def find_order(n, prerequisites):
    graph = [[] for _ in range(n)]
    indegree = [0] * n
    for a, b in prerequisites:
        graph[b].append(a)
        indegree[a] += 1
    queue = deque(i for i in range(n) if indegree[i] == 0)
    order = []
    while queue:
        u = queue.popleft()
        order.append(u)
        for v in graph[u]:
            indegree[v] -= 1
            if indegree[v] == 0:
                queue.append(v)
    return order if len(order) == n else []


def is_valid_order(order, prerequisites):
    pos = {c: i for i, c in enumerate(order)}
    return all(pos[b] < pos[a] for a, b in prerequisites)


pre = [[1, 0], [2, 0], [3, 1], [3, 2]]
order = find_order(4, pre)
assert len(order) == 4 and is_valid_order(order, pre)
assert find_order(2, [[1, 0], [0, 1]]) == []             # cycle
assert find_order(1, []) == [0]                           # edge: single course
assert sorted(find_order(3, [])) == [0, 1, 2]             # no constraints
```

**Complexity:** O(V + E) time and space. Kahn's algorithm is iterative, so there is no
recursion-depth concern, and cycle detection comes for free.

## DFS postorder

**Idea.** In a DFS, a vertex *finishes* only after everything reachable from it has
finished. So listing vertices in **reverse finishing order** puts each vertex before
everything it points to. A back edge to a vertex still "in progress" is a cycle.

```python
def topo_sort_dfs(n, edges):
    graph = [[] for _ in range(n)]
    for u, v in edges:
        graph[u].append(v)
    state = [0] * n                    # 0 new, 1 in progress, 2 done
    finished = []

    def visit(u):
        state[u] = 1
        for v in graph[u]:
            if state[v] == 1:
                raise ValueError("cycle")
            if state[v] == 0:
                visit(v)
        state[u] = 2
        finished.append(u)

    for u in range(n):
        if state[u] == 0:
            visit(u)
    return finished[::-1]


order = topo_sort_dfs(6, [(5, 2), (5, 0), (4, 0), (4, 1), (2, 3), (3, 1)])
pos = {v: i for i, v in enumerate(order)}
assert all(pos[u] < pos[v] for u, v in [(5, 2), (5, 0), (4, 0), (4, 1), (2, 3), (3, 1)])
try:
    topo_sort_dfs(2, [(0, 1), (1, 0)])
    assert False, "expected a cycle"
except ValueError:
    pass
```

Both algorithms are O(V + E). Prefer Kahn's in Python for large inputs (no recursion
limit), and when you need to process vertices "level by level" (e.g. minimum number of
semesters).

## Worked problem: alien dictionary

**Problem.** Words are sorted according to an unknown alphabet. Derive a valid letter
order, or return `""` if the input is inconsistent.

**Approach.** Compare each adjacent pair of words: the first position where they differ
gives one edge `a → b`. Then topologically sort the letters. One subtle invalid case: a
word followed by its own proper prefix (`"abc"` before `"ab"`) is impossible in any
alphabet.

```python
from collections import deque

def alien_order(words):
    letters = {ch for w in words for ch in w}
    graph = {ch: set() for ch in letters}
    indegree = {ch: 0 for ch in letters}
    for w1, w2 in zip(words, words[1:]):
        for a, b in zip(w1, w2):
            if a != b:
                if b not in graph[a]:
                    graph[a].add(b)
                    indegree[b] += 1
                break
        else:
            if len(w1) > len(w2):
                return ""                         # prefix after longer word
    queue = deque(sorted(ch for ch in letters if indegree[ch] == 0))
    out = []
    while queue:
        ch = queue.popleft()
        out.append(ch)
        for nxt in sorted(graph[ch]):
            indegree[nxt] -= 1
            if indegree[nxt] == 0:
                queue.append(nxt)
    return "".join(out) if len(out) == len(letters) else ""


assert alien_order(["wrt", "wrf", "er", "ett", "rftt"]) == "wertf"
assert alien_order(["z", "x", "z"]) == ""              # z < x < z: cycle
assert alien_order(["abc", "ab"]) == ""                # invalid prefix order
assert alien_order(["z"]) == "z"
```

(The `sorted` calls only make the output deterministic for testing; any valid order is
accepted.)

## DP over a DAG: longest path

Longest path is NP-hard in general graphs but easy on a DAG: process vertices in
topological order, and each vertex's best value is final when you reach it.

**Problem.** Longest increasing path in a matrix (moving up/down/left/right to a strictly
larger value). The "strictly larger" rule means edges always go uphill, so there are no
cycles — it is a DAG.

```python
from functools import lru_cache

def longest_increasing_path(matrix):
    if not matrix:
        return 0
    rows, cols = len(matrix), len(matrix[0])

    @lru_cache(maxsize=None)
    def longest_from(r, c):
        best = 1
        for nr, nc in ((r + 1, c), (r - 1, c), (r, c + 1), (r, c - 1)):
            if 0 <= nr < rows and 0 <= nc < cols and matrix[nr][nc] > matrix[r][c]:
                best = max(best, 1 + longest_from(nr, nc))
        return best

    return max(longest_from(r, c) for r in range(rows) for c in range(cols))


assert longest_increasing_path([[9, 9, 4], [6, 6, 8], [2, 1, 1]]) == 4   # 1-2-6-9
assert longest_increasing_path([[3, 4, 5], [3, 2, 6], [2, 2, 1]]) == 4
assert longest_increasing_path([[1]]) == 1
assert longest_increasing_path([[7, 7], [7, 7]]) == 1                    # no uphill moves
```

Memoized DFS is implicitly processing the DAG in reverse topological order: a cell's
answer is computed only after all its uphill neighbours are done. O(R · C) time. (For a
large grid, raise the recursion limit or rewrite with Kahn's algorithm on in-degrees.)

## How It Actually Works

**Why Kahn's algorithm is correct.** Every DAG has at least one vertex with in-degree 0
(otherwise, walking backward along incoming edges forever would eventually revisit a
vertex — a cycle). Removing such a vertex leaves a smaller DAG, so induction gives a
complete ordering. Conversely, vertices on a cycle each keep at least one incoming edge
from another cycle vertex, so their in-degree never reaches zero; that is why
`len(order) < n` exactly detects cycles.

**Why reverse postorder is correct.** For any edge `u → v` in a DAG, when DFS examines
it, `v` is either new (then `v` finishes inside `u`'s visit, before `u`) or already done
(finished earlier). It cannot be "in progress", because that would mean `v` can reach
`u` — a cycle. Either way `v` finishes before `u`, so reversing finishing order places
`u` before `v`.

**Why DP needs a DAG.** DP evaluates each state after the states it depends on. That is
only possible if dependencies have no cycles — i.e., they form a DAG, and a topological
order *is* a valid evaluation order. Every DP table you have filled (left to right, by
increasing length) was a topological order of its dependency DAG chosen by hand.

## Common mistakes

- Reversing edge direction (`a` requires `b` means `b → a`).
- Counting duplicate edges twice in in-degree without adding them twice to the graph
  (or vice versa) — keep them consistent, or deduplicate.
- Forgetting isolated vertices (letters that appear but have no constraints).
- Treating a plain visited set as cycle detection in a directed graph.
- Missing the "longer word before its prefix" invalid case in alien dictionary.

## Variations to practice

- Parallel courses / minimum semesters (Kahn's by levels).
- Sequence reconstruction (is the topological order unique? — queue size must stay 1).
- Build order with package versions.
- Find all recipes from supplies.
- Number of ways to reach a node in a DAG (DP in topological order).

## Exercise

Solve **Minimum Semesters**: given `n` courses and prerequisite pairs, return the minimum
number of semesters to take all courses if you can take any number of courses per
semester (only after their prerequisites), or -1 if impossible. Run Kahn's algorithm
level by level, counting levels. Test with a chain (n semesters), no prerequisites (1
semester), a diamond, and a cycle.
