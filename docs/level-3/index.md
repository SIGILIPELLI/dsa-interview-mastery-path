---
description: "Level 3 of DSA & Coding Interviews: knapsack and interval DP, Dijkstra, topological sort, union-find, monotonic stacks, bits, range trees, strings, MSTs."
---

# Level 3 · Advanced <span class="level-badge">Hard rounds</span>

Levels 1 and 2 cover the patterns behind most interview questions. Level 3 covers the
tools that appear in harder rounds, in online assessments with tight time limits, and in
follow-up questions ("now make it work when the array is updated", "now the edges have
weights"). Each lesson here builds on something you already know: advanced DP extends
lesson 2.6, Dijkstra extends BFS, monotonic stacks extend ordinary stacks, and segment
trees extend prefix sums.

Expect these lessons to take longer. Work each algorithm by hand on a small example
before reading its code — for Dijkstra and union-find in particular, a paper trace makes
the invariant obvious in a way the code does not.

## Modules

1. [Advanced Dynamic Programming](01-advanced-dp.md) — 0/1 knapsack, LIS in O(n log n), edit distance, grid DP, interval DP
2. [Shortest Paths: Dijkstra & Bellman-Ford](02-shortest-paths.md) — weighted graphs, lazy-deletion heaps, negative edges
3. [Topological Sort](03-topological-sort.md) — Kahn's algorithm, DFS postorder, and DP over a DAG
4. [Union-Find (Disjoint Set Union)](04-union-find.md) — path compression, union by size, dynamic connectivity
5. [Monotonic Stack & Queue](05-monotonic-stack-queue.md) — next greater element, largest rectangle, sliding-window maximum
6. [Bit Manipulation](06-bit-manipulation.md) — XOR tricks, masks, subset enumeration, bitmask DP
7. [Segment Trees & Fenwick Trees](07-segment-fenwick-trees.md) — range queries with point updates in O(log n)
8. [String Algorithms](08-string-algorithms.md) — KMP, rolling hashes (Rabin-Karp), palindromes by expansion
9. [Minimum Spanning Trees](09-minimum-spanning-trees.md) — Kruskal with union-find, Prim with a heap, the cut property
10. [Project — Hard Problem Set](10-project-hard-problem-set.md) — five hard problems with complete walkthroughs

## What you need before starting

- All of Level 2, especially DP (lesson 6), heaps (lesson 3), and BFS/DFS (lesson 4).
- Comfort proving a small invariant in words ("every item in the stack is ...").
- Familiarity with binary representation of integers for lesson 6.

After this level you should be able to recognize when a problem needs one of these
heavier tools, implement it from memory in 10–15 minutes, and explain its complexity.
