---
description: "Level 2 of DSA & Coding Interviews: trees, BSTs, heaps, BFS and DFS, backtracking, first dynamic programming, prefix sums, greedy and tries."
---

# Level 2 · Intermediate <span class="level-badge">Core patterns</span>

Level 1 was mostly about linear structures. Level 2 moves into **non-linear** ones —
trees and graphs — and into the three problem-solving strategies that cover a large
share of medium-difficulty interview questions: **search** (BFS, DFS, backtracking),
**dynamic programming**, and **greedy choice**.

The unifying skill here is thinking recursively about structure. A tree is a root plus
smaller trees. A graph search is "visit this node, then do the same for its unvisited
neighbours". A DP solution is "the answer for this state, given the answers for smaller
states". Once you are comfortable with the leap of faith from Level 1, lesson 8, most of
this level becomes variations on one theme.

## Modules

1. [Binary Trees & Traversals](01-trees-traversals.md) — preorder/inorder/postorder, level order, and returning values up the tree
2. [Binary Search Trees](02-binary-search-trees.md) — the ordering invariant, validation, insertion, deletion, and why balance matters
3. [Heaps & Priority Queues](03-heaps-priority-queues.md) — `heapq`, top-k, merging k sorted lists, and the two-heap median
4. [Graphs: BFS & DFS](04-graphs-bfs-dfs.md) — adjacency lists, grids as graphs, shortest unweighted paths, connected components
5. [Backtracking](05-backtracking.md) — choose/explore/undo, pruning, combinations, permutations, N-Queens
6. [Intro to Dynamic Programming](06-intro-dynamic-programming.md) — overlapping subproblems, memoization vs tabulation, 1-D DP
7. [Prefix Sums & Intervals](07-prefix-sums-intervals.md) — range sums in O(1), subarray-sum counting, merging and scheduling intervals
8. [Greedy Algorithms](08-greedy.md) — when a local choice is provably safe, and the exchange argument
9. [Tries](09-tries.md) — prefix trees for autocomplete, word search, and prefix counting
10. [Project — Timed Practice Set 2](10-project-practice-set-2.md) — six mixed problems with full walkthroughs

## What you need before starting

- Level 1 — especially recursion (lesson 8), hash maps (lesson 3), and `deque` (lesson 6).
- Comfort writing a small class (`TreeNode`, `TrieNode`).
- Patience with drawing: sketch trees, graphs and DP tables on paper. It is faster than
  debugging in your head.

By the end of this level you should be able to handle the majority of problems labelled
"medium" on common practice platforms, and explain your choice of approach.
