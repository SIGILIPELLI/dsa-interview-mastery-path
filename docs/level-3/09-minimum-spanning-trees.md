---
description: "Connect all nodes at minimum total cost with Kruskal's algorithm and union-find or Prim's algorithm and a heap, and prove both with the cut property."
---

# 09 · Minimum Spanning Trees

A **spanning tree** of a connected, undirected graph is a subset of its edges that
connects every vertex with no cycles — exactly `V - 1` edges. A **minimum spanning tree**
(MST) is a spanning tree with the smallest possible total edge weight.

**Signals:** "connect all points/cities/computers at minimum total cost", "minimum cost
to make the network connected", "cheapest way to lay cable". Contrast with shortest paths:
an MST minimizes the **total** cost of the network, not the distance between any
particular pair. The path between two vertices inside an MST is not, in general, their
shortest path.

## Kruskal's algorithm

**Idea.** Sort edges by weight. Walk through them, adding an edge whenever it connects two
different components (i.e., does not create a cycle). Union-find (lesson 4) answers "are
these already connected?" in near-constant time.

```python
class DSU:
    def __init__(self, n):
        self.parent, self.size = list(range(n)), [1] * n

    def find(self, x):
        while self.parent[x] != x:
            self.parent[x] = self.parent[self.parent[x]]   # path halving
            x = self.parent[x]
        return x

    def union(self, a, b):
        ra, rb = self.find(a), self.find(b)
        if ra == rb:
            return False
        if self.size[ra] < self.size[rb]:
            ra, rb = rb, ra
        self.parent[rb] = ra
        self.size[ra] += self.size[rb]
        return True


def kruskal(n, edges):
    """edges: (weight, u, v). Returns (total, chosen edges) or (None, []) if disconnected."""
    dsu, total, chosen = DSU(n), 0, []
    for w, u, v in sorted(edges):
        if dsu.union(u, v):
            total += w
            chosen.append((u, v, w))
            if len(chosen) == n - 1:
                break
    if len(chosen) != n - 1:
        return None, []
    return total, chosen


edges = [(1, 0, 1), (4, 0, 2), (3, 1, 2), (2, 1, 3), (5, 2, 3)]
total, chosen = kruskal(4, edges)
assert total == 6 and len(chosen) == 3               # 0-1 (1), 1-3 (2), 1-2 (3)
assert kruskal(3, [(1, 0, 1)]) == (None, [])         # edge: disconnected graph
assert kruskal(1, []) == (0, [])                     # edge: single vertex
```

Path halving (`parent[x] = parent[parent[x]]`) is a one-line variant of path compression
with the same practical effect.

**Complexity:** sorting dominates: **O(E log E)**. The union-find work is O(E · α(V)).

## Prim's algorithm

**Idea.** Grow one tree from a start vertex. Repeatedly add the cheapest edge that leaves
the tree. A min-heap of candidate edges (with lazy deletion, as in Dijkstra) supplies it.

### Worked problem: min cost to connect all points

**Problem.** Points on a plane; connecting two costs their Manhattan distance. Return the
minimum cost to connect all points.

The graph is **complete** (every pair is an edge), so E ≈ V²/2. Kruskal would sort all
those edges. Prim with a simple array — no heap — is O(V²), which is optimal for a
dense graph because it must look at every pair anyway.

```python
def min_cost_connect_points(points):
    n = len(points)
    if n <= 1:
        return 0
    in_tree = [False] * n
    best = [float("inf")] * n          # cheapest edge from the tree to each vertex
    best[0] = 0
    total = 0
    for _ in range(n):
        u = min((i for i in range(n) if not in_tree[i]), key=best.__getitem__)
        in_tree[u] = True
        total += best[u]
        ux, uy = points[u]
        for v in range(n):
            if not in_tree[v]:
                d = abs(ux - points[v][0]) + abs(uy - points[v][1])
                if d < best[v]:
                    best[v] = d
    return total


assert min_cost_connect_points([[0, 0], [2, 2], [3, 10], [5, 2], [7, 0]]) == 20
assert min_cost_connect_points([[3, 12], [-2, 5], [-4, 1]]) == 18
assert min_cost_connect_points([[0, 0]]) == 0                  # edge: one point
assert min_cost_connect_points([[0, 0], [0, 0]]) == 0          # duplicate points
```

### Heap-based Prim for sparse graphs

```python
import heapq
from collections import defaultdict

def prim(n, edges):
    graph = defaultdict(list)
    for w, u, v in edges:
        graph[u].append((w, v))
        graph[v].append((w, u))
    in_tree, total = set(), 0
    heap = [(0, 0)]                          # (edge weight, vertex)
    while heap and len(in_tree) < n:
        w, u = heapq.heappop(heap)
        if u in in_tree:
            continue                          # stale: already connected more cheaply
        in_tree.add(u)
        total += w
        for w2, v in graph[u]:
            if v not in in_tree:
                heapq.heappush(heap, (w2, v))
    return total if len(in_tree) == n else None


assert prim(4, [(1, 0, 1), (4, 0, 2), (3, 1, 2), (2, 1, 3), (5, 2, 3)]) == 6
assert prim(3, [(1, 0, 1)]) is None
```

O(E log V) — preferable to the O(V²) array version when the graph is sparse.

## Choosing between them

| Graph | Prefer |
|---|---|
| Given as an edge list, sparse | Kruskal (sort + union-find, very little code) |
| Dense or complete (points on a plane) | Prim with an array, O(V²) |
| Adjacency lists, sparse | Prim with a heap, or Kruskal |

## How It Actually Works

Both algorithms are justified by one fact, the **cut property**: split the vertices into
any two non-empty groups (a *cut*). Among the edges crossing between the groups, a
lightest one belongs to some minimum spanning tree.

**Proof sketch (exchange argument).** Take an MST `T` that does not contain the lightest
crossing edge `e`. Adding `e` to `T` creates a cycle, and that cycle must cross the cut
somewhere else, at an edge `f`. Remove `f`: the result is still a spanning tree, and its
weight is `weight(T) - w(f) + w(e) ≤ weight(T)`. So some MST contains `e`.

- **Prim** applies the cut property directly: the cut is (tree so far, everything else),
  and it adds the lightest crossing edge.
- **Kruskal** processes edges in increasing order. When it accepts edge `u–v`, consider
  the cut (u's current component, everything else). Every lighter edge was already
  examined and either merged components or was internal, so `u–v` is a lightest edge
  crossing that cut.

This is the matroid-style greedy mentioned in Level 2, lesson 8: sort by weight, add if it
keeps the structure valid (acyclic), and the result is optimal. If all edge weights are
distinct, the MST is unique.

## Common mistakes

- Treating an MST as a shortest-path tree (they answer different questions).
- Forgetting to check that the result has `V - 1` edges (the graph might be disconnected).
- Building all O(V²) edges and sorting them for dense point sets when O(V²) Prim with no
  heap is simpler and faster.
- In heap-based Prim, marking vertices as in the tree when pushed rather than when popped.
- Using directed-graph input without clarifying — MSTs are defined for undirected graphs
  (the directed analogue, the minimum arborescence, needs a different algorithm).

## Variations to practice

- Connecting cities with minimum cost (edge list — Kruskal).
- Optimize water distribution (add a virtual "well" vertex connected to every house).
- Find critical and pseudo-critical edges in an MST (re-run Kruskal excluding/forcing each
  edge).
- Minimum spanning tree in a grid where edges are height differences.

## Exercise

Solve **Optimize Water Distribution in a Village**: each house `i` can build a well at
cost `wells[i]`, or connect to another house via pipes `(house1, house2, cost)`. Model
"build a well at house i" as an edge from a new virtual vertex 0 to house i with weight
`wells[i]`, then run Kruskal. Test with `n = 3, wells = [1, 2, 2]`,
`pipes = [(1, 2, 1), (2, 3, 1)]` → 3, and with a case where every house should build its
own well. Explain in two sentences why the virtual vertex makes the reduction correct.
