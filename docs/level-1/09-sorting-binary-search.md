---
description: "Use sorting and binary search in interviews: merge sort, custom sort keys, a bug-free lower-bound template, rotated arrays, and searching the answer."
---

# 09 · Sorting & Binary Search

Sorting is often the first move that unlocks a problem: it groups equal items, enables
two pointers, and makes binary search possible. Binary search itself is deceptively
simple — the idea fits in a sentence, but off-by-one errors in the implementation are one
of the most common interview bugs. This lesson gives you one template to trust.

## Sorting: what you need to know

In interviews you almost always call the built-in sort rather than write one. Know its
properties:

- `sorted(iterable)` returns a new list; `lst.sort()` sorts in place and returns `None`.
- Both are **O(n log n)** in the worst case and **stable** (equal keys keep their
  original relative order).
- `key=` transforms each element once for comparison: `sorted(words, key=len)`.
  Sort by several criteria with a tuple: `key=lambda p: (-p.score, p.name)`.

```python
people = [("ana", 31), ("bo", 25), ("cy", 31), ("di", 25)]
by_age_desc_then_name = sorted(people, key=lambda p: (-p[1], p[0]))
assert by_age_desc_then_name == [("ana", 31), ("cy", 31), ("bo", 25), ("di", 25)]
```

You should still be able to write **merge sort**, both because interviewers ask and
because its divide-and-conquer shape reappears (counting inversions, merging k lists).

```python
def merge_sort(nums):
    if len(nums) <= 1:
        return nums
    mid = len(nums) // 2
    left = merge_sort(nums[:mid])
    right = merge_sort(nums[mid:])
    merged, i, j = [], 0, 0
    while i < len(left) and j < len(right):
        if left[i] <= right[j]:            # <= keeps it stable
            merged.append(left[i]); i += 1
        else:
            merged.append(right[j]); j += 1
    merged.extend(left[i:])
    merged.extend(right[j:])
    return merged


assert merge_sort([5, 2, 4, 6, 1, 3]) == [1, 2, 3, 4, 5, 6]
assert merge_sort([]) == []
assert merge_sort([2, 2, 1]) == [1, 2, 2]
```

O(n log n) time: log n levels of splitting, O(n) merging work per level. O(n) extra
space. (The slicing here adds copying but does not change the bound, since each level
already does O(n) work.)

## Binary search: one template

The most robust form finds the **first index where a condition becomes true**, given
that the condition is false for a prefix of the range and true for the rest (it is
*monotonic*).

```python
def first_true(lo, hi, cond):
    """Smallest i in [lo, hi) with cond(i) True, or hi if none."""
    while lo < hi:
        mid = (lo + hi) // 2
        if cond(mid):
            hi = mid          # mid might be the answer; keep it
        else:
            lo = mid + 1      # mid is not the answer; discard it
    return lo


def lower_bound(nums, target):
    return first_true(0, len(nums), lambda i: nums[i] >= target)


nums = [1, 3, 3, 5, 8]
assert lower_bound(nums, 3) == 1          # first 3
assert lower_bound(nums, 4) == 3          # where 4 would be inserted
assert lower_bound(nums, 0) == 0
assert lower_bound(nums, 9) == 5          # edge: past the end
assert lower_bound([], 1) == 0            # edge: empty
```

Why this template is hard to get wrong:

- The search space is the half-open interval `[lo, hi)`, and the loop runs while it is
  non-empty.
- `mid` is always strictly less than `hi`, so `hi = mid` shrinks the range, and
  `lo = mid + 1` shrinks it too. The loop always terminates.
- When it ends, `lo == hi` is the boundary between false and true.

"Find exact target" is `i = lower_bound(...)` followed by
`i < len(nums) and nums[i] == target`. Python's `bisect.bisect_left` is the same
function, and `bisect_right` gives the first index with `nums[i] > target`.

## Worked problem 1: search in a rotated sorted array

**Problem.** A sorted array of distinct values was rotated (e.g. `[4,5,6,7,0,1,2]`).
Find the index of `target` in O(log n), or return -1.

**Approach.** At any `mid`, at least one half — `[lo, mid]` or `[mid, hi]` — is sorted.
Check whether the target lies inside the sorted half; if so, search there, otherwise
search the other half.

```python
def search_rotated(nums, target):
    lo, hi = 0, len(nums) - 1
    while lo <= hi:
        mid = (lo + hi) // 2
        if nums[mid] == target:
            return mid
        if nums[lo] <= nums[mid]:                 # left half is sorted
            if nums[lo] <= target < nums[mid]:
                hi = mid - 1
            else:
                lo = mid + 1
        else:                                     # right half is sorted
            if nums[mid] < target <= nums[hi]:
                lo = mid + 1
            else:
                hi = mid - 1
    return -1


assert search_rotated([4, 5, 6, 7, 0, 1, 2], 0) == 4
assert search_rotated([4, 5, 6, 7, 0, 1, 2], 3) == -1
assert search_rotated([1], 1) == 0
assert search_rotated([3, 1], 1) == 1                # edge: two elements
assert search_rotated([], 5) == -1
```

This uses the closed-interval form `[lo, hi]` because it is checking for an exact match
at `mid`. Both forms are fine; the danger is mixing them.

## Worked problem 2: binary search on the answer

**Problem.** Koko has piles of bananas and `h` hours. Each hour she picks one pile and
eats up to `k` bananas from it. Find the minimum integer speed `k` that finishes all
piles within `h` hours.

**Insight.** You are not searching an array — you are searching the *answer space*
`k ∈ [1, max(piles)]`. The condition "speed `k` finishes in time" is monotonic: if `k`
works, any faster speed also works. So find the first `k` where it becomes true.

```python
def min_eating_speed(piles, h):
    def finishes(k):
        hours = sum((p + k - 1) // k for p in piles)   # ceil(p / k)
        return hours <= h

    return first_true(1, max(piles) + 1, finishes)


assert min_eating_speed([3, 6, 7, 11], 8) == 4
assert min_eating_speed([30, 11, 23, 4, 20], 5) == 30   # one pile per hour
assert min_eating_speed([30, 11, 23, 4, 20], 6) == 23
assert min_eating_speed([1], 1) == 1
```

**Complexity:** O(n · log M) where M is the largest pile. This "binary search on the
answer" idea solves shipping capacity, splitting arrays to minimize the largest sum,
minimum days to make bouquets, and many optimization problems where checking a candidate
is easy but constructing the optimum directly is hard.

## How It Actually Works

**Why binary search is O(log n).** Each iteration halves the interval, so after `k`
iterations at most `n / 2ᵏ` candidates remain; the loop ends after about `log₂ n`
iterations. For 10⁹ candidates that is about 30 checks.

**Why comparison sorting cannot beat O(n log n).** A comparison sort must distinguish
all n! possible input orderings, and each comparison has two outcomes, so the decision
tree needs at least n! leaves and therefore a depth of at least log₂(n!), which is on the
order of n log n. Algorithms like counting sort escape the bound only because they do
not compare — they index by value, which requires a small key range.

**What Python's sort does.** CPython's `list.sort` has used Timsort for most of its
history and switched to a closely related variant (Powersort's merge policy) in Python
3.11. It scans the input for already-sorted "runs", extends short runs with binary
insertion sort, then merges runs using a policy that keeps merges balanced. On random
data it behaves like a merge sort, O(n log n); on data that is already mostly sorted it
can approach O(n). It is stable because merges never reorder equal elements. It uses up
to O(n) temporary memory for merging.

**Integer overflow.** In languages with fixed-width integers, `(lo + hi) / 2` can
overflow; the standard fix is `lo + (hi - lo) / 2`. Python integers are arbitrary
precision, so this is not an issue here, but mention it if interviewing in Java or C++.

## Common mistakes

- Mixing `[lo, hi)` and `[lo, hi]` conventions within one function → infinite loops or
  skipped elements.
- `lo = mid` (without `+ 1`) in a `lo < hi` loop when `mid` rounds down → infinite loop.
- Applying binary search when the condition is not monotonic.
- Forgetting that `lst.sort()` returns `None` (`x = lst.sort()` is a bug).
- Using float division for the ceiling: `math.ceil(p / k)` works but integer
  `(p + k - 1) // k` avoids floating-point issues on large values.

## Variations to practice

- Find the first and last position of a target (two lower-bound calls).
- Find the minimum in a rotated sorted array.
- Square root of an integer (first `x` with `x*x > n`, minus one).
- Capacity to ship packages within D days.
- Merge intervals (sort by start — Level 2, lesson 7).
- Kth largest element (sort vs heap vs quickselect).

## Exercise

Solve **Capacity to Ship Packages Within D Days**: given package weights (shipped in
order) and `days`, find the least ship capacity that ships everything in time. Identify
the lowest and highest possible answers, write the `can_ship(capacity)` check, and reuse
`first_true`. Test with `[1..10], days=5` → 15, `[3,2,2,4,1,4], days=3` → 6, and a
single package.
