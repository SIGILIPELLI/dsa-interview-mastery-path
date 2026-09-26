---
description: "Recognize when greedy choices are safe: jump game, interval scheduling, gas station and task assignment, with exchange arguments proving each strategy."
---

# 08 · Greedy Algorithms

A greedy algorithm makes the locally best choice at each step and never reconsiders. When
it works, it is usually the simplest and fastest solution. The catch: it often *seems*
to work and does not. The skill is proving (or at least convincingly arguing) that the
greedy choice is safe — and knowing a counterexample when it is not.

**Signals:** optimization over a sequence where some ordering (by end time, by ratio, by
deadline) makes one choice obviously dominant; "minimum number of X to cover Y"; problems
where DP would work but the constraints are too large for it.

## How to argue a greedy choice is correct

The standard tool is the **exchange argument**: take any optimal solution that does *not*
make the greedy choice, and show you can swap in the greedy choice without making it
worse. Then there is always an optimal solution that agrees with greedy, and by induction
greedy is optimal.

Before investing in a proof, **try to break it**: test two or three tiny cases, including
adversarial ones. Coin change with `[1, 3, 4]` and amount 6 breaks "take the largest
coin" in seconds (lesson 6).

## Worked problem 1: jump game

**Problem.** `nums[i]` is your maximum jump length from index `i`. Starting at index 0,
can you reach the last index?

**Greedy idea.** Track the furthest index reachable so far. Walk forward; if you ever
stand on an index beyond that reach, you are stuck.

```python
def can_jump(nums):
    reach = 0
    for i, jump in enumerate(nums):
        if i > reach:
            return False
        reach = max(reach, i + jump)
    return True


assert can_jump([2, 3, 1, 1, 4])
assert not can_jump([3, 2, 1, 0, 4])        # stuck at index 3
assert can_jump([0])                        # edge: already at the end
assert not can_jump([0, 1])
```

**Why it is correct:** every index ≤ `reach` is reachable (you can always jump *less*
than the maximum), so the set of reachable indices is a prefix `[0, reach]`. Tracking only
its right end loses nothing. O(n) time, O(1) space — versus O(n²) for the obvious DP.

## Worked problem 2: minimum jumps

**Problem.** Same setup, reaching the end is guaranteed; return the minimum number of
jumps.

**Greedy idea (implicit BFS).** Treat indices reachable with `j` jumps as "level j". The
current level ends at `current_end`; while scanning it, compute the furthest point the
next level can reach.

```python
def min_jumps(nums):
    jumps = current_end = furthest = 0
    for i in range(len(nums) - 1):          # no jump needed from the last index
        furthest = max(furthest, i + nums[i])
        if i == current_end:                 # finished this level
            jumps += 1
            current_end = furthest
    return jumps


assert min_jumps([2, 3, 1, 1, 4]) == 2
assert min_jumps([2, 3, 0, 1, 4]) == 2
assert min_jumps([0]) == 0                  # edge: single element
assert min_jumps([1, 1, 1, 1]) == 3
```

## Worked problem 3: maximum non-overlapping intervals

**Problem.** Given intervals, remove the minimum number so the rest do not overlap.
Equivalently: keep as many non-overlapping intervals as possible.

**Greedy choice:** sort by **end time**; repeatedly keep the interval that ends earliest
among those compatible with what you have kept.

```python
def erase_overlap_intervals(intervals):
    kept, last_end = 0, float("-inf")
    for start, end in sorted(intervals, key=lambda iv: iv[1]):
        if start >= last_end:                # compatible (touching is fine)
            kept += 1
            last_end = end
    return len(intervals) - kept


assert erase_overlap_intervals([[1, 2], [2, 3], [3, 4], [1, 3]]) == 1
assert erase_overlap_intervals([[1, 2], [1, 2], [1, 2]]) == 2
assert erase_overlap_intervals([[1, 2], [2, 3]]) == 0
assert erase_overlap_intervals([]) == 0
assert erase_overlap_intervals([[1, 100], [2, 3], [4, 5]]) == 1   # sorting by start would keep [1,100]
```

**Exchange argument:** let `g` be the interval with the earliest end. Take any optimal
set; let `o` be its first interval. Since `g` ends no later than `o`, replacing `o` with
`g` cannot create an overlap with the rest of the set. So some optimal solution starts
with `g`; remove everything overlapping `g` and repeat. Sorting by *start* fails (last
test: the long early interval blocks two short ones), and so does sorting by *length*
(a short interval can straddle two compatible longer ones).

## Worked problem 4: gas station

**Problem.** Stations in a circle; `gas[i]` fuel available, `cost[i]` to drive to the
next station. Return the starting index that lets you complete the circuit, or -1. The
answer is unique if it exists.

**Two facts:**

1. If `sum(gas) < sum(cost)`, no start works.
2. If you start at `s` and first run dry trying to reach station `k + 1`, then **no
   station between `s` and `k` can be a valid start** either: you arrived at each of them
   with a non-negative tank, and starting there with an empty tank can only be worse. So
   jump the candidate to `k + 1`.

```python
def can_complete_circuit(gas, cost):
    if sum(gas) < sum(cost):
        return -1
    start = tank = 0
    for i in range(len(gas)):
        tank += gas[i] - cost[i]
        if tank < 0:
            start = i + 1
            tank = 0
    return start


assert can_complete_circuit([1, 2, 3, 4, 5], [3, 4, 5, 1, 2]) == 3
assert can_complete_circuit([2, 3, 4], [3, 4, 3]) == -1
assert can_complete_circuit([5], [4]) == 0                 # edge: one station
assert can_complete_circuit([3, 1, 1], [1, 2, 2]) == 0
```

O(n), one pass. Fact 2 is the greedy insight; fact 1 guarantees that the surviving
candidate actually works.

## How It Actually Works

Greedy algorithms work on problems with two properties:

1. **Greedy-choice property** — some optimal solution contains the greedy choice.
2. **Optimal substructure** — after making that choice, what remains is a smaller
   instance of the same problem.

DP also needs optimal substructure, but it *tries every choice* and takes the best. Greedy
commits to one choice without looking back, which is only valid when property 1 holds.
That is why greedy runs in O(n) or O(n log n) (usually dominated by a sort) where the
corresponding DP might be O(n²) or worse.

There is deeper theory behind which problems admit greedy solutions (for example,
*matroids* characterize a class where "sort by weight, add if it keeps the set valid" is
always optimal — Kruskal's minimum spanning tree algorithm in Level 3 is the classic
instance). For interviews you do not need the theory, but you do need the habit: state
the greedy rule, try to break it on small inputs, then give the exchange argument in two
or three sentences.

## Common mistakes

- Choosing a plausible greedy rule and not testing it against a counterexample.
- Sorting by the wrong key (start vs end vs length) for interval problems.
- Treating "touching" intervals inconsistently with the problem statement.
- Forgetting the global feasibility check (like `sum(gas) < sum(cost)`) that makes the
  greedy candidate valid.
- Using greedy for coin change with arbitrary denominations.

## Variations to practice

- Minimum number of arrows to burst balloons (sort by end).
- Partition labels (last occurrence of each character).
- Assign cookies / boats to save people (sort + two pointers).
- Task scheduler with cooldown (counting argument).
- Candy distribution (two passes, left and right).
- Queue reconstruction by height.

## Exercise

Solve **Partition Labels**: split a string into as many parts as possible so each letter
appears in at most one part, returning the part sizes (`"ababcbacadefegdehijhklij"` →
`[9, 7, 8]`). First compute each letter's last index; then scan, extending the current
part's end to the furthest last-index seen, and cut when `i` reaches it. Add asserts for a
single-character string and a string of all distinct letters, and write the two-sentence
argument for why cutting at the earliest possible point is optimal.
