---
title: "DSA & Coding Interviews Cheat Sheet: Quick Reference, Beginner to Advanced"
description: "A free DSA & Coding Interviews cheat sheet: 15 quick-reference tables from every level of the DSA & Coding Interviews Mastery Path, each linked to its full lesson."
---

# DSA & Coding Interviews Cheat Sheet

Quick-reference tables from **15 lessons** of the free DSA & Coding Interviews Mastery Path, in course order from beginner to advanced. Each section links to the full lesson with explanations and runnable examples.

## Level 1 · Entry

### [01 · Big-O &amp; Complexity Analysis](level-1/01-big-o-complexity.md)

*What Big-O measures*

| Class | Name | Typical source |
|-------|------|----------------|
| O(1) | constant | index into a list, dict lookup (average) |
| O(log n) | logarithmic | binary search, heap push/pop, balanced-tree operations |
| O(n) | linear | one pass over the input |
| O(n log n) | linearithmic | comparison sorting, n heap operations |
| O(n²) | quadratic | all pairs, nested loops over the same input |
| O(2ⁿ) | exponential | all subsets |
| O(n!) | factorial | all permutations |

### [10 · Project — Timed Practice Set 1](level-1/10-project-timed-practice-set.md)

*Scoring sheet*

| # | Pattern recognized? | Correct & tested? | Complexity stated correctly? | Time taken | Notes |
|---|---|---|---|---|---|
| 1 | | | | | |
| 2 | | | | | |
| 3 | | | | | |
| 4 | | | | | |
| 5 | | | | | |
| 6 | | | | | |

## Level 2 · Intermediate

### [03 · Heaps &amp; Priority Queues](level-2/03-heaps-priority-queues.md)

| Operation | Cost |
|-----------|------|
| peek at min | O(1) |
| push | O(log n) |
| pop min | O(log n) |
| build from n items | O(n) |

### [10 · Project — Timed Practice Set 2](level-2/10-project-practice-set-2.md)

*Self-review rubric*

| Criterion | 0 | 1 | 2 |
|---|---|---|---|
| Pattern | Did not identify | Identified after coding started | Identified during planning |
| Correctness | Fails tests | Passes with bugs fixed late | Passes first run |
| Complexity | Not stated / wrong | Stated, minor errors | Correct time and space |
| Edge cases | None considered | Some | Empty/single/duplicate/extreme all considered |
| Communication | Silent coding | Explained after the fact | Explained plan before coding |

## Level 3 · Advanced

### [01 · Advanced Dynamic Programming](level-3/01-advanced-dp.md)

| Family | Typical state | Examples |
|---|---|---|
| Knapsack | items considered × capacity used | subset sum, partition equal subset, target sum |
| Subsequence | index (+ last value) | longest increasing subsequence, number of LIS |
| Two sequences | prefix of A × prefix of B | edit distance, LCS, interleaving strings |
| Grid | row × column | minimum path sum, maximal square |
| Interval | left end × right end | burst balloons, palindrome partitioning, matrix-chain |

### [02 · Shortest Paths: Dijkstra &amp; Bellman-Ford](level-3/02-shortest-paths.md)

| Situation | Algorithm | Time |
|---|---|---|
| Unweighted | BFS | O(V + E) |
| Weights 0 or 1 only | 0-1 BFS with a deque | O(V + E) |
| Non-negative weights | Dijkstra with a binary heap | O((V + E) log V) |
| Negative weights, or a limit on the number of edges | Bellman-Ford | O(V · E) |
| All pairs, small V | Floyd-Warshall | O(V³) |

### [06 · Bit Manipulation](level-3/06-bit-manipulation.md)

*The operators*

| Operator | Meaning | Example (4-bit) |
|---|---|---|
| `a & b` | AND — 1 where both are 1 | `1100 & 1010 = 1000` |
| `a \| b` | OR — 1 where either is 1 | `1100 \| 1010 = 1110` |
| `a ^ b` | XOR — 1 where they differ | `1100 ^ 1010 = 0110` |
| `~a` | NOT — in Python, `-a - 1` | `~5 == -6` |
| `a << k` | shift left, multiply by 2ᵏ | `3 << 2 == 12` |
| `a >> k` | shift right, floor-divide by 2ᵏ | `13 >> 2 == 3` |

### [07 · Segment Trees &amp; Fenwick Trees](level-3/07-segment-fenwick-trees.md)

| Need | Structure |
|---|---|
| Range sum, point update | Fenwick tree (simplest) or segment tree |
| Range min/max, point update | Segment tree |
| Range update and range query | Segment tree with lazy propagation |
| Static range min, no updates | Sparse table (O(1) query) |

### [09 · Minimum Spanning Trees](level-3/09-minimum-spanning-trees.md)

*Choosing between them*

| Graph | Prefer |
|---|---|
| Given as an edge list, sparse | Kruskal (sort + union-find, very little code) |
| Dense or complete (points on a plane) | Prim with an array, O(V²) |
| Adjacency lists, sparse | Prim with a heap, or Kruskal |

## Level 4 · Master

### [01 · The Interview Communication Framework](level-4/01-interview-framework.md)

*The six steps*

| Step | Goal | Rough time in a 45-minute round |
|---|---|---|
| 1. Clarify | Pin down inputs, outputs, constraints | 2–4 min |
| 2. Examples | Build and walk through small cases, including edge cases | 2–4 min |
| 3. Approach | Brute force first, then improve; agree before coding | 5–10 min |
| 4. Complexity | State time and space of the chosen approach | 1 min |
| 5. Code | Write clean code, narrating the non-obvious parts | 10–15 min |
| 6. Test | Trace an example, run edge cases, fix bugs | 5 min |

### [02 · Behavioral Interview Prep](level-4/02-behavioral-prep.md)

*Step 1: build a story bank*

| Story | One-line summary | Themes |
|---|---|---|
| Flaky test suite | Tracked down intermittent CI failures nobody owned | ownership, ambiguity, impact |
| API disagreement | Disagreed with a teammate about an interface; resolved with a prototype | conflict, collaboration |
| Missed deadline | Underestimated a migration; communicated late; changed how I estimate | failure, learning |
| Onboarding doc | Wrote setup docs after struggling myself | growth, helping others |

### [07 · Testing &amp; Debugging Live](level-4/07-testing-debugging-live.md)

*Tracing with a variable table*

| step | lo | hi | mid | nums[mid] | branch |
|---|---|---|---|---|---|
| start | 0 | 2 | – | – | – |
| 1 | 0 | 2 | 1 | 3 | 3 < 9 → lo = 2 |
| end | 2 | 2 | – | – | loop exits |

### [08 · Offers &amp; Negotiation Basics](level-4/08-offers-negotiation.md)

*Components of a typical offer*

| Component | What it is | Questions to ask |
|---|---|---|
| Base salary | Fixed regular pay | Is it the same across locations? When are reviews? |
| Signing bonus | One-time payment on joining | Is there a clawback if I leave early? When is it paid? |
| Annual bonus | Variable pay tied to performance/company results | Is there a target percentage? How has it typically paid out? |
| Equity | Stock, RSUs, or options | Type, number, vesting schedule, cliff, and (for options) strike price |
| Benefits | Health cover, retirement contributions, leave | What is covered for dependants? Any matching contributions? |
| Other | Relocation, learning budget, equipment, remote policy | Is it written into the offer? |
| Level/title | Where you sit in the company's ladder | What level is this? What does promotion to the next level require? |

### [09 · Study Plans: 4, 8 and 12 Weeks](level-4/09-study-plans.md)

*4-week plan (intensive refresher)*

| Week | Lessons | Practice focus | Mocks |
|---|---|---|---|
| 1 | L1: 1–9 (skim what you know) | 3 problems/day across arrays, hashing, two pointers, windows, binary search | — |
| 2 | L2: 1–6 | trees, heaps, BFS/DFS, backtracking, 1-D DP | L1 practice set (timed) |
| 3 | L2: 7–9; L4: 1, 5, 7 | intervals, greedy, mixed mediums | 2 mock interviews |
| 4 | L4: 2, 3, 4, 6; review queue | behavioral story bank; weakest patterns | L2 practice set + 2 mocks + capstone |

### [10 · Capstone — Full Mock Interview](level-4/10-capstone-mock-interview.md)

*Part 2 — The rubric*

| Dimension | 2 — developing | 4 — strong |
|---|---|---|
| Problem understanding | Restates problem; misses a constraint | Restates precisely; asks questions that change the solution |
| Approach & reasoning | Reaches a working idea with help | Brute force → bottleneck → optimized idea; tests ideas with counterexamples |
| Coding | Works with some messy or redundant parts | Clean, idiomatic, well-named; appropriate structure (e.g. iterative for depth) |
| Complexity | States it when asked, minor errors | States time and space unprompted and correctly |
| Testing | Runs the example only | Chooses targeted cases, traces the real code, fixes bugs by cause |
| Communication | Explains when prompted | Narrates reasoning, checks in, uses hints well |

---

Go deeper with the full [DSA & Coding Interviews Mastery Path](index.md) curriculum or test yourself with the [interview questions](interview-questions.md).
