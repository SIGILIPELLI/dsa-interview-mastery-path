---
description: "Master the two-pointer pattern: pair sums in sorted arrays, 3Sum without duplicates, container with most water, and the invariant proving it correct."
---

# 04 · Two Pointers

Two pointers replace a nested loop over pairs with a single coordinated scan. Instead of
checking all O(n²) pairs, you move two indices according to a rule that provably never
skips the answer. The result is usually O(n) time and O(1) space.

There are two main shapes:

- **Opposite ends** — `left` starts at 0, `right` at `n-1`, and they move toward each
  other. Works when the input is sorted or has some monotonic structure.
- **Same direction** — a slow and a fast pointer move forward at different speeds
  (the read/write compaction in lesson 2 is one; sliding window in lesson 5 is another;
  Floyd's cycle detection in lesson 7 is a third).

## Worked problem 1: pair sum in a sorted array

**Problem.** `nums` is sorted ascending. Return the 1-based indices of two numbers that
add up to `target` (exactly one answer exists). Use O(1) extra space.

**Approach.** Look at `nums[left] + nums[right]`:

- too small → the only way to grow the sum is to move `left` right;
- too large → move `right` left;
- equal → done.

```python
def two_sum_sorted(nums, target):
    left, right = 0, len(nums) - 1
    while left < right:
        s = nums[left] + nums[right]
        if s == target:
            return [left + 1, right + 1]
        if s < target:
            left += 1
        else:
            right -= 1
    return []


assert two_sum_sorted([2, 7, 11, 15], 9) == [1, 2]
assert two_sum_sorted([2, 3, 4], 6) == [1, 3]
assert two_sum_sorted([-1, 0], -1) == [1, 2]       # edge: two elements, negatives
assert two_sum_sorted([1, 1, 1, 5], 6) == [1, 4]   # duplicates
```

**Why is it safe to discard?** When `s < target`, pairing `nums[left]` with *any*
index `≤ right` gives a sum ≤ `s` (the array is sorted), so `nums[left]` cannot be part
of the answer with any remaining partner. Discarding it loses nothing. The symmetric
argument covers `s > target`. That is the whole correctness proof — say it out loud in an
interview.

**Complexity:** each step moves one pointer inward, so at most n-1 steps: O(n) time,
O(1) space.

## Worked problem 2: 3Sum

**Problem.** Return all unique triplets `[a, b, c]` from `nums` with `a + b + c == 0`.
The output must not contain duplicate triplets.

**Approach.** Sort. Fix the first element `nums[i]`, then run the sorted pair-sum on the
rest for target `-nums[i]`. Skip duplicate values at both levels so each triplet is
produced once.

```python
def three_sum(nums):
    nums = sorted(nums)
    n = len(nums)
    result = []
    for i in range(n - 2):
        if nums[i] > 0:
            break                                  # smallest is positive: no zero sum
        if i > 0 and nums[i] == nums[i - 1]:
            continue                               # same first value as before
        left, right = i + 1, n - 1
        while left < right:
            s = nums[i] + nums[left] + nums[right]
            if s < 0:
                left += 1
            elif s > 0:
                right -= 1
            else:
                result.append([nums[i], nums[left], nums[right]])
                left += 1
                right -= 1
                while left < right and nums[left] == nums[left - 1]:
                    left += 1                      # skip duplicate second values
    return result


assert three_sum([-1, 0, 1, 2, -1, -4]) == [[-1, -1, 2], [-1, 0, 1]]
assert three_sum([0, 0, 0, 0]) == [[0, 0, 0]]     # edge: many duplicates
assert three_sum([0, 1, 1]) == []
assert three_sum([]) == []
```

Trace `[-4, -1, -1, 0, 1, 2]` (sorted): `i=0` (-4) finds nothing. `i=1` (-1): the pair
search finds `(-1, 2)` then `(0, 1)`. `i=2` is skipped because it repeats -1.

**Complexity:** sorting is O(n log n); the outer loop times the inner scan is O(n²).
Total **O(n²) time**, O(1) extra space beyond the sort (Python's `sorted` makes a copy:
O(n)).

## Worked problem 3: container with most water

**Problem.** `height[i]` is the height of a vertical line at position `i`. Choose two
lines that, with the x-axis, hold the most water. Return that area.

**Approach.** Start with the widest container (`left=0`, `right=n-1`). The area is
`min(h[left], h[right]) × (right - left)`. Move the pointer at the **shorter** line.

```python
def max_area(height):
    left, right = 0, len(height) - 1
    best = 0
    while left < right:
        h = min(height[left], height[right])
        best = max(best, h * (right - left))
        if height[left] < height[right]:
            left += 1
        else:
            right -= 1
    return best


assert max_area([1, 8, 6, 2, 5, 4, 8, 3, 7]) == 49
assert max_area([1, 1]) == 1                      # edge: two lines
assert max_area([4, 3, 2, 1, 4]) == 16
assert max_area([0, 0, 0]) == 0
```

**Why move the shorter one?** Suppose `height[left] < height[right]`. Any container that
keeps `left` and uses some `right' < right` is narrower, and its height is still capped
by `height[left]`. So none of those can beat the current area — the line at `left` is
finished, and discarding it is safe.

## How It Actually Works

Two-pointer algorithms are really a **search over a 2-D grid of pairs** `(i, j)`. Picture
an n × n table where cell `(i, j)` holds `nums[i] + nums[j]`. For a sorted array, values
increase going right along a row and going down a column. Starting at the top-right
corner `(0, n-1)`, each comparison with the target eliminates an entire row (move `left`
down) or an entire column (move `right` left). Each elimination is justified by the
monotonic ordering, and since there are only n rows and n columns, you finish in O(n)
steps instead of visiting n² cells.

That picture explains when the pattern applies: you need a rule that, from a comparison
at the current pair, can prove a whole row or column is useless. Sortedness provides
this for sums; the "shorter wall caps the height" argument provides it for container
with most water. When no such rule exists, two pointers do not apply and you should
reach for hashing instead (plain Two Sum on an unsorted array, where sorting would lose
the original indices).

## Common mistakes

- Using `while left <= right` for pair problems, which lets an element pair with itself.
- In 3Sum, deduplicating with a `set` of tuples after the fact: it works, but it hides
  that your loop is producing duplicates and still costs extra time and memory. Skip
  duplicates structurally.
- Forgetting to move *both* pointers after recording a match, causing an infinite loop.
- Applying opposite-end pointers to unsorted input without sorting (or without a proof
  that the move rule is safe).
- Sorting when the problem asks for original indices — sort `(value, index)` pairs or use
  a hash map instead.

## Variations to practice

- Valid palindrome (skip non-alphanumerics from both ends).
- 3Sum closest; 4Sum (one more outer loop: O(n³)).
- Remove duplicates from a sorted array in place (same direction).
- Trapping rain water with two pointers and running maxima from each side.
- Sort colors / Dutch national flag (three pointers: low, mid, high).

## Exercise

Solve **Sort Colors**: given a list of 0s, 1s and 2s, sort it in place in one pass
using O(1) space (no counting then rewriting). Use three pointers `low`, `mid`, `high`
and write down the invariant for the four regions they define before coding. Test on
`[2,0,2,1,1,0]`, `[]`, `[1]`, `[2,2,2]`, and `[0,2,1]`.
