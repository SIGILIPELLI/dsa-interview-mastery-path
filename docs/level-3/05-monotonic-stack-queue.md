---
description: "Use monotonic stacks and deques for next-greater-element, daily temperatures, largest rectangle in a histogram and sliding-window maximum in linear time."
---

# 05 · Monotonic Stack & Queue

A **monotonic stack** is an ordinary stack with one rule: its contents stay sorted
(increasing or decreasing) from bottom to top. Before pushing a new element, you pop
everything that would break the order — and **the moment an element is popped is the
moment you learn its answer**.

**Signals:** "next greater / next smaller element", "previous smaller", "how many days
until warmer", "span", "largest rectangle", "remove k digits to make the smallest
number", "sliding window maximum". Any time a brute force scans left or right from each
element looking for the first element that beats it.

## Worked problem 1: next greater element

**Problem.** For each element, find the next element to its right that is strictly
greater, or -1.

**Approach.** Scan left to right, keeping a stack of **indices whose answer is still
unknown**. Their values are decreasing from bottom to top (if a bigger element had come
after a smaller one, the smaller one would already have its answer). When a new value
arrives, it is the answer for every stacked index with a smaller value.

```python
def next_greater(nums):
    result = [-1] * len(nums)
    stack = []                               # indices, values decreasing
    for i, x in enumerate(nums):
        while stack and nums[stack[-1]] < x:
            result[stack.pop()] = x
        stack.append(i)
    return result


assert next_greater([2, 1, 2, 4, 3]) == [4, 2, 4, -1, -1]
assert next_greater([5, 4, 3]) == [-1, -1, -1]           # decreasing: no answers
assert next_greater([1, 2, 3]) == [2, 3, -1]
assert next_greater([]) == []
assert next_greater([2, 2]) == [-1, -1]                  # equal is not greater
```

**Complexity:** O(n) — each index is pushed once and popped at most once.

### Daily temperatures

Same pattern, but the answer is the **distance** to the next warmer day:

```python
def daily_temperatures(temps):
    result = [0] * len(temps)
    stack = []
    for i, t in enumerate(temps):
        while stack and temps[stack[-1]] < t:
            j = stack.pop()
            result[j] = i - j
        stack.append(i)
    return result


assert daily_temperatures([73, 74, 75, 71, 69, 72, 76, 73]) == [1, 1, 4, 2, 1, 1, 0, 0]
assert daily_temperatures([30]) == [0]
```

### Circular arrays

For "next greater in a circular array", loop over indices `0 .. 2n-1` using `i % n`, and
only push during the first pass. Each element gets a second chance to see elements that
come before it.

```python
def next_greater_circular(nums):
    n = len(nums)
    result = [-1] * n
    stack = []
    for i in range(2 * n):
        x = nums[i % n]
        while stack and nums[stack[-1]] < x:
            result[stack.pop()] = x
        if i < n:
            stack.append(i)
    return result


assert next_greater_circular([1, 2, 1]) == [2, -1, 2]
assert next_greater_circular([3, 3]) == [-1, -1]
```

## Worked problem 2: largest rectangle in a histogram

**Problem.** Bars of width 1 with heights `h`. Find the area of the largest rectangle
inside the histogram.

**Insight.** For each bar, the widest rectangle using that bar's full height extends left
and right until the first **shorter** bar on each side. An increasing stack gives both
boundaries: when bar `j` is popped by a shorter bar `i`, the right boundary is `i`, and
the left boundary is whatever is below `j` on the stack.

```python
def largest_rectangle(heights):
    stack = []                           # indices, heights increasing
    best = 0
    for i, h in enumerate(heights + [0]):     # sentinel 0 flushes the stack at the end
        while stack and heights[stack[-1]] > h:
            height = heights[stack.pop()]
            left = stack[-1] if stack else -1
            best = max(best, height * (i - left - 1))
        stack.append(i)
    return best


assert largest_rectangle([2, 1, 5, 6, 2, 3]) == 10     # heights 5 and 6, width 2
assert largest_rectangle([2, 4]) == 4
assert largest_rectangle([]) == 0
assert largest_rectangle([3, 3, 3]) == 9               # equal heights
assert largest_rectangle([1, 2, 3, 4, 5]) == 9         # 3 * 3
```

`heights + [0]` creates a copy with the sentinel; `heights[stack[-1]]` never reads the
sentinel's index while it is on the stack, because the loop ends right after pushing it.

**Complexity:** O(n). This problem is the core of "maximal rectangle of 1s in a matrix":
treat each row as the base of a histogram of consecutive-1 heights.

## Worked problem 3: sliding window maximum (monotonic deque)

**Problem.** Return the maximum of every window of size `k`.

**Approach.** Keep a deque of indices whose values are **decreasing** from front to
back. The front is always the current window's maximum. When a new value arrives, pop
smaller values from the back (they can never be a maximum while this larger, newer value
is in the window). Pop the front when it slides out of the window.

```python
from collections import deque

def max_sliding_window(nums, k):
    dq, out = deque(), []
    for i, x in enumerate(nums):
        while dq and nums[dq[-1]] <= x:
            dq.pop()                          # dominated: older and not larger
        dq.append(i)
        if dq[0] <= i - k:
            dq.popleft()                      # left the window
        if i >= k - 1:
            out.append(nums[dq[0]])
    return out


assert max_sliding_window([1, 3, -1, -3, 5, 3, 6, 7], 3) == [3, 3, 5, 5, 6, 7]
assert max_sliding_window([1], 1) == [1]
assert max_sliding_window([9, 8, 7], 1) == [9, 8, 7]      # edge: k = 1
assert max_sliding_window([4, 2, 12, 3], 4) == [12]       # edge: k = n
```

O(n) time, O(k) space. A heap would give O(n log n); the deque is strictly better here.

## Worked problem 4: remove k digits

**Problem.** Remove `k` digits from a number string to make the smallest possible number.

**Greedy + monotonic stack.** Scan left to right. While the previous kept digit is
larger than the current one and removals remain, remove it — a larger digit in a more
significant position is always worse.

```python
def remove_k_digits(num, k):
    stack = []
    for d in num:
        while k and stack and stack[-1] > d:
            stack.pop()
            k -= 1
        stack.append(d)
    if k:
        stack = stack[:-k]                  # remaining removals from the end
    return "".join(stack).lstrip("0") or "0"


assert remove_k_digits("1432219", 3) == "1219"
assert remove_k_digits("10200", 1) == "200"
assert remove_k_digits("10", 2) == "0"                  # edge: remove everything
assert remove_k_digits("12345", 2) == "123"             # increasing: trim the end
```

## How It Actually Works

**Amortized O(n).** The `while` loop inside the `for` loop looks quadratic, but every
index is pushed exactly once and popped at most once over the whole run. So the total
number of pops is at most n, and the algorithm is O(n) overall — the same aggregate
argument as the sliding window in Level 1.

**Why popping is safe (the dominance argument).** In the next-greater problem, an index
`j` stays on the stack only while no greater element has appeared after it. When `x`
arrives and `nums[j] < x`, `x` is by definition the *first* greater element after `j`
(anything between them was ≤ `nums[j]`, or `j` would already have been popped). So `j`'s
answer is final and `j` can leave. In the sliding-window maximum, an index is popped
from the back when a newer, larger value arrives: the older value leaves the window
**earlier** and is **smaller**, so it can never be the maximum again. Removing elements
that are provably useless is what keeps the structure small and sorted.

**What the stack encodes.** At any moment, a decreasing stack holds exactly the
"candidates still waiting for a greater element", and they are sorted because each one
survived all the elements pushed after it. For the histogram, the element below a popped
bar in an increasing stack is the nearest shorter bar to its left — which is why one
structure yields both boundaries.

## Common mistakes

- Storing values instead of indices when you need distances or window boundaries.
- Using `<` vs `<=` inconsistently — decide how equal elements should behave (strictly
  greater? equal allowed?) and test with duplicates.
- Forgetting to flush the stack at the end (use a sentinel or a final loop).
- In the sliding-window deque, removing the front with a value comparison instead of an
  index comparison (fails with duplicate values).
- Missing `lstrip("0") or "0"` in remove k digits.

## Variations to practice

- Online stock span (previous greater-or-equal, streaming).
- Sum of subarray minimums (previous smaller and next smaller for each element).
- Maximal rectangle in a binary matrix.
- Trapping rain water with a stack.
- Shortest subarray with sum at least k (monotonic deque over prefix sums; works with
  negatives).
- Remove duplicate letters (smallest subsequence with each letter once).

## Exercise

Solve **Sum of Subarray Minimums**: for every contiguous subarray, take its minimum, and
return the sum of those minimums. For each index `i`, find how many subarrays have
`nums[i]` as their minimum using the distance to the previous *smaller* element and to
the next *smaller-or-equal* element (asymmetric to avoid double-counting duplicates).
Test with `[3,1,2,4]` → 17, `[11,81,94,43,3]` → 444, `[2,2]` → 6, and a single element.
