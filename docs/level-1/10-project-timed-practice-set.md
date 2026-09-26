---
description: "A timed six-problem practice set covering Level 1, with a scoring sheet and full walkthroughs: approach, tested Python code, complexity, edge cases."
---

# 10 · Project — Timed Practice Set 1

This project is a simulation, not a reading exercise. You will solve six problems under
a clock, then grade yourself against the walkthroughs. The goal is to find out which
Level 1 patterns you can *recognize and execute under pressure*, not just follow when
someone else explains them.

## Rules

1. Set a timer for **90 minutes** total. Aim for about 15 minutes per problem; if you are
   stuck at 20 minutes, write down your best idea and move on.
2. Work in a plain editor. No searching, no autocomplete-driven guessing.
3. For each problem, before coding, write a comment block with: the pattern you think
   applies, the approach in two or three sentences, and the target complexity.
4. Write at least three `assert` tests per problem, including one edge case, and run them.
5. Only after the timer ends, read the walkthroughs below.

## Scoring sheet

Copy this table and fill it in honestly. It is your study plan for the next week.

| # | Pattern recognized? | Correct & tested? | Complexity stated correctly? | Time taken | Notes |
|---|---|---|---|---|---|
| 1 | | | | | |
| 2 | | | | | |
| 3 | | | | | |
| 4 | | | | | |
| 5 | | | | | |
| 6 | | | | | |

Score 1 point for each "yes". 15+ out of 18 means Level 1 is solid; below 12, revisit the
lessons for whichever patterns you missed before moving to Level 2.

## The problems

1. **Anagram Groups Count.** Given a list of lowercase words, return the number of
   distinct anagram groups. `["eat","tea","tan","ate","nat","bat"]` → 3.
2. **Longest Subarray With Sum At Most K.** Given a list of **non-negative** integers and
   `k`, return the length of the longest contiguous subarray with sum ≤ `k`.
3. **Merge Sorted Arrays In Place.** `a` has length `m + n`, where the first `m` values
   are sorted and the last `n` are zeros (spare slots); `b` has `n` sorted values. Merge
   `b` into `a` in place, sorted.
4. **Maximum Nesting Depth.** Given a valid parentheses string of `(` and `)`,
   return its maximum nesting depth. `"(()(()))"` → 3.
5. **Kth Smallest Pair Distance (simplified).** Given a sorted list and a distance `d`,
   count the pairs `i < j` with `nums[j] - nums[i] <= d`, in O(n).
6. **First Bad Version.** Versions `1..n`; `is_bad(v)` is false for all versions before
   some first bad one and true from there on. Find the first bad version with the
   fewest calls.

Stop reading here until your timer is done.

---

## Walkthrough 1 — anagram groups count

**Pattern:** hashing with a canonical key (lesson 3). Two words are in the same group
exactly when their sorted letters (or letter counts) match, so count distinct keys.

```python
def count_anagram_groups(words):
    return len({"".join(sorted(w)) for w in words})


assert count_anagram_groups(["eat", "tea", "tan", "ate", "nat", "bat"]) == 3
assert count_anagram_groups([]) == 0
assert count_anagram_groups(["a", "a"]) == 1
assert count_anagram_groups(["ab", "ba", "abc"]) == 2
```

**Complexity:** O(n · k log k) with sorted keys; switch to 26-count tuples for O(n · k).
**Edge cases:** empty list, duplicate words, words of different lengths (never anagrams —
the keys differ automatically).

## Walkthrough 2 — longest subarray with sum at most k

**Pattern:** variable sliding window (lesson 5). Non-negative values make it valid: adding
an element never decreases the sum, so if a window is too big, every window containing
it is too big.

```python
def longest_at_most_k(nums, k):
    left = total = best = 0
    for right, x in enumerate(nums):
        total += x
        while total > k and left <= right:
            total -= nums[left]
            left += 1
        best = max(best, right - left + 1)
    return best


assert longest_at_most_k([1, 2, 1, 0, 1, 1, 0], 4) == 5     # [1, 0, 1, 1, 0]
assert longest_at_most_k([5, 6], 4) == 0                    # edge: every element too big
assert longest_at_most_k([], 3) == 0
assert longest_at_most_k([0, 0, 0], 0) == 3
```

In the `[5, 6], k=4` case, the window shrinks past `right` (`left = right + 1`), making
the length 0 — which is exactly what we want. The `left <= right` guard stops `left` from
running further.

**Complexity:** O(n) time (each index enters and leaves once), O(1) space.
**If values could be negative**, the window no longer works; you would need prefix sums
with a sorted structure.

## Walkthrough 3 — merge sorted arrays in place

**Pattern:** two pointers, but filling from the **back** (lesson 4). Filling from the
front would overwrite values of `a` you still need. The end of `a` is free space, so
write the largest remaining value there.

```python
def merge_in_place(a, m, b, n):
    i, j, write = m - 1, n - 1, m + n - 1
    while j >= 0:                         # once b is exhausted, a's rest is in place
        if i >= 0 and a[i] > b[j]:
            a[write] = a[i]
            i -= 1
        else:
            a[write] = b[j]
            j -= 1
        write -= 1


a = [1, 2, 3, 0, 0, 0]
merge_in_place(a, 3, [2, 5, 6], 3)
assert a == [1, 2, 2, 3, 5, 6]
a = [0]
merge_in_place(a, 0, [1], 1)
assert a == [1]                           # edge: a has no real values
a = [4, 5, 0, 0]
merge_in_place(a, 2, [1, 2], 2)
assert a == [1, 2, 4, 5]                  # all of b goes in front
a = [1]
merge_in_place(a, 1, [], 0)
assert a == [1]                           # edge: b empty
```

**Complexity:** O(m + n) time, O(1) space. **Why the loop only checks `j`:** if `b` runs
out first, the remaining `a[0..i]` values are already sorted and already in position.

## Walkthrough 4 — maximum nesting depth

**Pattern:** a stack — but you only need its *size*, so a counter replaces it (lesson 6).
This simplification is worth pointing out in an interview: it shows you understand what
the stack was doing.

```python
def max_depth(s):
    depth = best = 0
    for ch in s:
        if ch == "(":
            depth += 1
            best = max(best, depth)
        elif ch == ")":
            depth -= 1
    return best


assert max_depth("(()(()))") == 3
assert max_depth("()()") == 1
assert max_depth("") == 0
assert max_depth("((()))") == 3
```

**Complexity:** O(n) time, O(1) space. If the string could be *invalid*, you would also
check that `depth` never goes negative and ends at 0.

## Walkthrough 5 — count pairs within distance d

**Pattern:** same-direction two pointers on a sorted array. For each `right`, move `left`
forward until `nums[right] - nums[left] <= d`; then every index in `[left, right)` pairs
validly with `right`.

```python
def count_pairs_within(nums, d):
    left = count = 0
    for right in range(len(nums)):
        while nums[right] - nums[left] > d:
            left += 1
        count += right - left
    return count


assert count_pairs_within([1, 3, 4, 7], 2) == 2          # (1,3), (3,4)
assert count_pairs_within([1, 1, 1], 0) == 3             # duplicates pair with each other
assert count_pairs_within([], 5) == 0
assert count_pairs_within([1, 100], 1000) == 1
```

**Complexity:** O(n) — `left` only advances. This counting function is exactly the
`cond` you would binary search over in the full "k-th smallest pair distance" problem:
binary search on `d` for the first distance whose count reaches k (lesson 9).

## Walkthrough 6 — first bad version

**Pattern:** binary search for the first true (lesson 9). The predicate `is_bad` is
monotonic by definition.

```python
def first_bad_version(n, is_bad):
    lo, hi = 1, n                 # answer is guaranteed to exist in [1, n]
    while lo < hi:
        mid = (lo + hi) // 2
        if is_bad(mid):
            hi = mid
        else:
            lo = mid + 1
    return lo


calls = []
def make_checker(first_bad):
    def is_bad(v):
        calls.append(v)
        return v >= first_bad
    return is_bad

assert first_bad_version(5, make_checker(4)) == 4
assert first_bad_version(1, make_checker(1)) == 1         # edge: n = 1
assert first_bad_version(10**9, make_checker(1)) == 1     # edge: first version bad
calls.clear()
assert first_bad_version(2**20, make_checker(777_777)) == 777_777
assert len(calls) <= 21                                   # ~log2(n) calls
```

**Complexity:** O(log n) calls to `is_bad`, O(1) space.

## Reviewing your attempt

For each problem you missed, write one sentence answering: *what in the problem
statement should have pointed me to the pattern?* For example: "non-negative values +
contiguous + longest" → sliding window; "sorted + pairs" → two pointers; "monotonic
yes/no over a range" → binary search. Those trigger phrases are what you are really
training.

Also review *how* you spent the time. Common patterns in self-review:

- Coding before having an approach → long debugging. Fix: always write the plan comment.
- Correct idea, wrong boundaries → practice the binary search template until it is
  automatic.
- Forgot edge cases → make "empty, one element, all same, negatives" a reflexive checklist.

## How It Actually Works

Why practise under a clock instead of just solving more problems? Untimed practice lets
you recognize a pattern slowly, by trial and error; interviews and assessments require
recognizing it quickly, from the wording of the problem alone. A timed set forces
**retrieval** — pulling the right technique from memory with no hints about which lesson
it came from — and retrieval under mild pressure is what strengthens that recognition.

The six problems were chosen so that each one hinges on a single property: non-negative
values make the window monotonic (problem 2), free space at the end makes back-to-front
merging safe (problem 3), sortedness makes the pair-counting pointer only move forward
(problem 5), and a monotonic predicate makes binary search valid (problem 6). If you
notice the property, the pattern follows. The scoring sheet separates *recognizing* the
pattern from *executing* it because they fail for different reasons and need different
practice: recognition failures call for more varied problem reading; execution failures
call for drilling templates such as the binary search in lesson 9.

## Exercise

A week after this attempt, redo the set **from scratch** in 60 minutes without looking at
your previous code. Then write three new problems of your own — one each for hashing,
sliding window, and binary search on the answer — with a statement, a solution, and at
least four asserts each. Writing problems forces you to understand exactly which
property (sortedness, non-negativity, monotonicity) a pattern depends on.
