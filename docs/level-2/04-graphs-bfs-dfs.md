---
description: "Model problems as graphs and traverse them with BFS and DFS: adjacency lists, grids, connected components, shortest unweighted paths and cycle checks."
---

# 04 · Graphs: BFS & DFS

A graph is a set of **vertices** (nodes) connected by **edges**. Trees are a special case
(connected, no cycles). Many interview problems are graph problems in disguise: a grid
of islands, a word ladder, course prerequisites, friends of friends, states of a puzzle.
The first step is always to name the vertices and edges.

## Representing a graph

The standard representation is an **adjacency list**: a dict mapping each vertex to its
neighbours.

```python
from collections import defaultdict, deque

def build_graph(n, edges, directed=False):
    graph = defaultdict(list)
    for u, v in edges:
        graph[u].append(v)
        if not directed:
            graph[v].append(u)
    for node in range(n):
        graph[node]                     # make sure isolated nodes exist
    return graph


g = build_graph(4, [(0, 1), (1, 2)])
assert sorted(g[1]) == [0, 2] and g[3] == []
```

An adjacency list uses O(V + E) space. An **adjacency matrix** (V × V booleans) uses
O(V²) space but answers "is there an edge u–v?" in O(1); it only makes sense for small,
dense graphs.

**Grids are implicit graphs.** Cell `(r, c)` is a vertex; its neighbours are the up to
four adjacent cells. You never build the adjacency list — you generate neighbours on the
fly.

## BFS vs DFS

- **Breadth-first search** explores in rings of increasing distance using a **queue**. In
  an unweighted graph, the first time BFS reaches a vertex is along a **shortest path**
  (fewest edges).
- **Depth-first search** follows one path as deep as possible before backtracking, using
  recursion or a **stack**. It is the natural fit for connectivity, cycle detection,
  path existence, and topological ordering.

Both are O(V + E) time: each vertex is enqueued/visited once and each edge examined once
(twice if undirected). Both need a **visited** set — without it, cycles cause infinite
loops.

## Worked problem 1: number of islands (DFS on a grid)

**Problem.** A grid of `"1"` (land) and `"0"` (water). Count islands (groups of land
connected horizontally or vertically).

**Approach.** Scan every cell. When you find unvisited land, that is a new island;
flood-fill it so its cells are not counted again.

```python
def num_islands(grid):
    if not grid:
        return 0
    rows, cols = len(grid), len(grid[0])
    seen = set()

    def fill(r, c):
        stack = [(r, c)]                      # iterative DFS: safe for large islands
        seen.add((r, c))
        while stack:
            cr, cc = stack.pop()
            for nr, nc in ((cr + 1, cc), (cr - 1, cc), (cr, cc + 1), (cr, cc - 1)):
                if (0 <= nr < rows and 0 <= nc < cols and
                        grid[nr][nc] == "1" and (nr, nc) not in seen):
                    seen.add((nr, nc))
                    stack.append((nr, nc))

    count = 0
    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == "1" and (r, c) not in seen:
                count += 1
                fill(r, c)
    return count


grid = [list("11000"),
        list("11000"),
        list("00100"),
        list("00011")]
assert num_islands(grid) == 3
assert num_islands([list("000")]) == 0          # edge: no land
assert num_islands([list("1")]) == 1            # edge: single cell
assert num_islands([]) == 0
```

**Complexity:** O(R · C) time and space. Marking cells visited **when pushing** (not when
popping) prevents the same cell from being pushed many times.

## Worked problem 2: shortest path in a binary grid (BFS)

**Problem.** In an n × n grid of 0s (open) and 1s (blocked), find the length (number of
cells) of the shortest path from top-left to bottom-right moving in 8 directions, or -1.

```python
from collections import deque

def shortest_path_binary_matrix(grid):
    n = len(grid)
    if grid[0][0] or grid[n - 1][n - 1]:
        return -1
    queue = deque([(0, 0, 1)])                  # row, col, path length so far
    seen = {(0, 0)}
    while queue:
        r, c, dist = queue.popleft()
        if (r, c) == (n - 1, n - 1):
            return dist
        for dr in (-1, 0, 1):
            for dc in (-1, 0, 1):
                nr, nc = r + dr, c + dc
                if (0 <= nr < n and 0 <= nc < n and not grid[nr][nc]
                        and (nr, nc) not in seen):
                    seen.add((nr, nc))
                    queue.append((nr, nc, dist + 1))
    return -1


assert shortest_path_binary_matrix([[0, 1], [1, 0]]) == 2
assert shortest_path_binary_matrix([[0, 0, 0], [1, 1, 0], [1, 1, 0]]) == 4
assert shortest_path_binary_matrix([[1, 0], [0, 0]]) == -1    # edge: start blocked
assert shortest_path_binary_matrix([[0]]) == 1                # edge: 1x1
```

DFS would also find *a* path, but not necessarily the shortest. When the question says
"minimum number of steps" in an unweighted setting, reach for BFS.

## Worked problem 3: connected components and cycle detection

**Problem.** Given `n` nodes and undirected edges, is the graph a valid tree? (A tree is
connected with exactly n − 1 edges and no cycles.)

```python
def valid_tree(n, edges):
    if len(edges) != n - 1:
        return False                  # too many edges -> cycle; too few -> disconnected
    graph = build_graph(n, edges)
    seen, stack = {0}, [0]
    while stack:
        node = stack.pop()
        for nxt in graph[node]:
            if nxt not in seen:
                seen.add(nxt)
                stack.append(nxt)
    return len(seen) == n             # n-1 edges + connected => no cycle


assert valid_tree(5, [(0, 1), (0, 2), (0, 3), (1, 4)])
assert not valid_tree(5, [(0, 1), (1, 2), (2, 3), (1, 3), (1, 4)])
assert valid_tree(1, [])                          # edge: single node
assert not valid_tree(4, [(0, 1), (2, 3), (1, 0)])  # right count, but disconnected
```

The edge-count check does the heavy lifting: a connected graph on n vertices with exactly
n − 1 edges cannot contain a cycle. For **directed** graphs, cycle detection needs a
three-state visit (unvisited / in progress / done) — see topological sort in Level 3.

## Worked problem 4: multi-source BFS

**Problem.** Rotting oranges: each minute, rotten oranges (2) rot adjacent fresh ones (1).
How many minutes until none are fresh, or -1 if impossible?

**Approach.** Start BFS from **all** rotten oranges at once; each BFS level is one minute.

```python
from collections import deque

def oranges_rotting(grid):
    rows, cols = len(grid), len(grid[0])
    queue, fresh = deque(), 0
    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == 2:
                queue.append((r, c))
            elif grid[r][c] == 1:
                fresh += 1
    minutes = 0
    while queue and fresh:
        for _ in range(len(queue)):
            r, c = queue.popleft()
            for nr, nc in ((r + 1, c), (r - 1, c), (r, c + 1), (r, c - 1)):
                if 0 <= nr < rows and 0 <= nc < cols and grid[nr][nc] == 1:
                    grid[nr][nc] = 2
                    fresh -= 1
                    queue.append((nr, nc))
        minutes += 1
    return minutes if fresh == 0 else -1


assert oranges_rotting([[2, 1, 1], [1, 1, 0], [0, 1, 1]]) == 4
assert oranges_rotting([[2, 1, 1], [0, 1, 1], [1, 0, 1]]) == -1   # isolated fresh
assert oranges_rotting([[0, 2]]) == 0                             # edge: nothing fresh
```

(This version mutates the input grid; mention it, and copy first if that is not allowed.)

## How It Actually Works

**Why BFS finds shortest paths.** BFS maintains the invariant that the queue contains
vertices in non-decreasing order of distance, with at most two distinct distances (`d`
and `d + 1`) at any time. When a vertex at distance `d` is dequeued, its unvisited
neighbours get distance `d + 1` and go to the back. So vertices are discovered in order
of distance, and the first discovery of any vertex is via a shortest path. This breaks
the moment edges have different weights — a two-edge path can be cheaper than a
one-edge path — which is why weighted graphs need Dijkstra (Level 3).

**Why O(V + E).** The visited set guarantees each vertex is processed once. Processing a
vertex scans its adjacency list, and the adjacency lists together contain E entries
(2E for undirected). Summing gives O(V + E). With an adjacency matrix, scanning a
vertex's neighbours costs O(V) regardless of how many there are, so the total becomes
O(V²).

**DFS and the stack.** Recursive DFS uses the call stack; iterative DFS uses an explicit
list. They can visit neighbours in different orders, which does not matter for
connectivity but does matter when a problem depends on the exact visitation order.
Recursive DFS on a 10⁵-cell grid will exceed Python's default recursion limit — use the
iterative version for large inputs.

## Common mistakes

- Forgetting the visited set, or marking visited on pop instead of on push (the same
  node gets queued many times; correctness survives but complexity can blow up).
- Using DFS when the problem asks for a minimum number of steps.
- Bounds checks after indexing (`grid[nr][nc]` with `nr = -1` silently wraps to the last
  row in Python instead of failing!). Check bounds first.
- Building an adjacency list that misses isolated vertices, then concluding the graph is
  connected.
- Using `list.pop(0)` for BFS.

## Variations to practice

- Clone graph (BFS/DFS with a map from old node to new node).
- Pacific Atlantic water flow (reverse BFS from both oceans).
- Word ladder (BFS where neighbours differ by one letter).
- Surrounded regions (flood-fill from the border).
- Number of connected components (DFS, or union-find in Level 3).
- Bipartite check (2-colouring with BFS).

## Exercise

Write `is_bipartite(graph)` where `graph[i]` is the list of neighbours of node `i`. Colour
nodes with BFS, starting a new BFS from every uncoloured node (the graph may be
disconnected). Test on a 4-cycle (bipartite), a triangle (not), a graph with an isolated
vertex, and an empty graph. State why odd cycles are exactly what makes a graph
non-bipartite.
