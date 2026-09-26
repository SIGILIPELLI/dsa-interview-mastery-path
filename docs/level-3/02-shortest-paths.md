---
description: "Find shortest paths in weighted graphs with Dijkstra's algorithm and a heap, handle negative edges with Bellman-Ford, and solve limited-stops routing."
---

# 02 · Shortest Paths: Dijkstra & Bellman-Ford

BFS finds shortest paths when every edge costs the same. Once edges have **weights**
(travel times, prices, latencies), a path with more edges can be cheaper, and BFS's
level-by-level order no longer means anything. This lesson covers the two algorithms you
need, and how to choose:

| Situation | Algorithm | Time |
|---|---|---|
| Unweighted | BFS | O(V + E) |
| Weights 0 or 1 only | 0-1 BFS with a deque | O(V + E) |
| Non-negative weights | Dijkstra with a binary heap | O((V + E) log V) |
| Negative weights, or a limit on the number of edges | Bellman-Ford | O(V · E) |
| All pairs, small V | Floyd-Warshall | O(V³) |

## Dijkstra's algorithm

**Idea.** Keep a tentative distance for every vertex. Repeatedly take the unfinished
vertex with the smallest tentative distance, declare it final, and **relax** its outgoing
edges: if going through it gives a neighbour a shorter distance, update the neighbour.

A min-heap supplies "smallest tentative distance". Python's `heapq` cannot decrease a key
in place, so we use **lazy deletion**: push a new entry whenever a distance improves, and
skip stale entries when they are popped.

### Worked problem: network delay time

**Problem.** `times[i] = (u, v, w)`: a signal from `u` reaches `v` after `w`. Starting
at node `k` (nodes numbered 1..n), how long until all nodes receive it? Return -1 if some
node never does.

```python
import heapq
from collections import defaultdict

def dijkstra(n, edges, source):
    graph = defaultdict(list)
    for u, v, w in edges:
        graph[u].append((v, w))
    dist = {source: 0}
    heap = [(0, source)]
    while heap:
        d, u = heapq.heappop(heap)
        if d > dist.get(u, float("inf")):
            continue                              # stale entry
        for v, w in graph[u]:
            nd = d + w
            if nd < dist.get(v, float("inf")):
                dist[v] = nd
                heapq.heappush(heap, (nd, v))
    return dist


def network_delay_time(times, n, k):
    dist = dijkstra(n, times, k)
    if len(dist) < n:
        return -1
    return max(dist.values())


assert network_delay_time([(2, 1, 1), (2, 3, 1), (3, 4, 1)], 4, 2) == 2
assert network_delay_time([(1, 2, 1)], 2, 2) == -1         # node 1 unreachable from 2
assert network_delay_time([], 1, 1) == 0                   # edge: single node
# a longer path in edges can be shorter in weight:
assert network_delay_time([(1, 2, 10), (1, 3, 1), (3, 2, 2)], 3, 1) == 3
```

Trace the last case: pop 1 (d=0) → tentative 2:10, 3:1. Pop 3 (d=1) → relax 3→2 gives
3 < 10, so 2:3 and push `(3, 2)`. Pop `(3, 2)` — final. Later pop `(10, 2)` is stale and
skipped. Maximum distance: 3.

### Recovering the path

Store `parent[v] = u` on each successful relaxation, then walk backward from the target.

```python
import heapq

def shortest_path(graph, source, target):
    dist, parent = {source: 0}, {source: None}
    heap = [(0, source)]
    while heap:
        d, u = heapq.heappop(heap)
        if u == target:
            break                                 # target is final: stop early
        if d > dist[u]:
            continue
        for v, w in graph.get(u, []):
            if d + w < dist.get(v, float("inf")):
                dist[v], parent[v] = d + w, u
                heapq.heappush(heap, (d + w, v))
    if target not in dist:
        return None, []
    path, node = [], target
    while node is not None:
        path.append(node)
        node = parent[node]
    return dist[target], path[::-1]


g = {"A": [("B", 4), ("C", 1)], "C": [("B", 2), ("D", 7)], "B": [("D", 1)]}
assert shortest_path(g, "A", "D") == (4, ["A", "C", "B", "D"])
assert shortest_path(g, "A", "A") == (0, ["A"])
assert shortest_path(g, "D", "A") == (None, [])
```

## Bellman-Ford

Dijkstra's greedy finalization breaks with negative edges: a vertex declared final might
later be reachable more cheaply through a negative edge. Bellman-Ford makes no such
commitment. It relaxes **every edge**, `V - 1` times. After round `i`, every shortest path
using at most `i` edges is correct.

That "at most i edges" property is exactly what the next problem needs.

### Worked problem: cheapest flights within k stops

**Problem.** Flights `(from, to, price)`. Find the cheapest price from `src` to `dst`
using at most `k` stops (i.e. at most `k + 1` flights), or -1.

```python
def find_cheapest_price(n, flights, src, dst, k):
    INF = float("inf")
    cost = [INF] * n
    cost[src] = 0
    for _ in range(k + 1):                   # at most k+1 edges
        nxt = cost.copy()                    # read old round, write new round
        for u, v, p in flights:
            if cost[u] + p < nxt[v]:
                nxt[v] = cost[u] + p
        cost = nxt
    return cost[dst] if cost[dst] != INF else -1


flights = [(0, 1, 100), (1, 2, 100), (2, 0, 100), (1, 3, 600), (2, 3, 200)]
assert find_cheapest_price(4, flights, 0, 3, 1) == 700     # 0->1->3
assert find_cheapest_price(4, flights, 0, 3, 2) == 400     # 0->1->2->3
assert find_cheapest_price(3, [(0, 1, 100)], 0, 2, 5) == -1
assert find_cheapest_price(2, [(0, 1, 5)], 0, 0, 0) == 0   # edge: src == dst
```

The `cost.copy()` is essential: without it, one round could chain several relaxations
and use more edges than allowed. **Complexity:** O(k · E).

### Detecting negative cycles

If a V-th round of relaxation still improves some distance, the graph contains a
negative cycle reachable from the source (shortest paths are then undefined, since you
could loop forever to reduce cost).

```python
def has_negative_cycle(n, edges, source=0):
    dist = [float("inf")] * n
    dist[source] = 0
    for _ in range(n - 1):
        for u, v, w in edges:
            if dist[u] + w < dist[v]:
                dist[v] = dist[u] + w
    return any(dist[u] + w < dist[v] for u, v, w in edges)


assert has_negative_cycle(3, [(0, 1, 1), (1, 2, -1), (2, 0, -1)])
assert not has_negative_cycle(3, [(0, 1, 1), (1, 2, -1), (0, 2, 5)])
```

## How It Actually Works

**Why Dijkstra's greedy choice is safe.** Claim: when vertex `u` is popped with distance
`d`, `d` is its true shortest distance. Suppose a shorter path existed. It starts at the
source (finalized) and at some point first leaves the set of finalized vertices, at an
edge `x → y`. `y`'s tentative distance is at most the cost of that path prefix, which
(because all remaining edge weights are **non-negative**) is at most the whole path's
cost, which is less than `d`. Then `y` would have been popped before `u` — a
contradiction. Every step of that argument uses non-negativity; with a negative edge
later on the path, the prefix could cost *more* than the whole path, and the proof
collapses.

**Complexity of the heap version.** Each successful relaxation pushes one entry, so the
heap sees at most E pushes and E pops, each O(log E) = O(log V) (since E ≤ V²). Scanning
adjacency lists is O(V + E). Total O((V + E) log V). Lazy deletion wastes some heap
space (up to E entries) but avoids needing a decrease-key operation, which `heapq` does
not provide. (Theoretical variants with Fibonacci heaps reach O(E + V log V), but they
are rarely faster in practice and never expected in interviews.)

**Why Bellman-Ford needs V − 1 rounds.** A shortest path without cycles has at most
V − 1 edges. Relaxing every edge once guarantees the first edge of every shortest path
is settled; the second round settles the second edge, and so on. After V − 1 rounds all
are settled — unless a negative cycle keeps offering improvements, which the extra round
detects.

## Common mistakes

- Using Dijkstra with negative edge weights.
- Forgetting the stale-entry check (`if d > dist[u]: continue`) — still correct, but can
  redo a lot of work.
- Marking vertices visited when **pushed** (correct for BFS, wrong for Dijkstra — a
  better path may arrive later).
- Bellman-Ford with a stop limit but without copying the distance array per round.
- Returning `inf` instead of the required sentinel (-1) for unreachable targets.

## Variations to practice

- Path with maximum probability (Dijkstra with a max-heap on products).
- Minimum effort path in a grid (Dijkstra where path cost is the max edge).
- Swim in rising water (Dijkstra or binary search + BFS).
- 0-1 BFS for a grid where some moves are free.
- Floyd-Warshall for all-pairs distances in a small city graph.

## Exercise

Solve **Path With Minimum Effort**: in a grid of heights, a path's effort is the
**maximum** absolute height difference between consecutive cells. Find the minimum
effort from top-left to bottom-right. Adapt Dijkstra so a path's "distance" is
`max(current effort, |h1 - h2|)` instead of a sum, and explain why the greedy argument
still holds (hint: extending a path can never decrease its max). Test on
`[[1,2,2],[3,8,2],[5,3,5]]` → 2, a 1×1 grid → 0, and a single row.
