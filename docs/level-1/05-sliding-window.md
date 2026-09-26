---
description: "Solve contiguous subarray and substring problems with fixed and variable sliding windows, and learn the amortized argument that makes them linear."
---

# 05 · Sliding Window

A sliding window is a contiguous range `[left, right]` that moves across an array or
string. Instead of recomputing something about every subarray from scratch (O(n²) or
worse), you update the answer incrementally as one element enters on the right and
another leaves on the left.

**Signals that a problem wants a sliding window:** the words *contiguous*, *subarray*,
*substring*, combined with *longest*, *shortest*, *at most k*, *exactly k*, or *sum ≥ s*.

## Fixed-size window

**Problem.** Return the maximum average of any contiguous subarray of length `k`.

**Approach.** Sum the first `k` elements. Then slide: add the element entering on the
right, subtract the element leaving on the left.

```python
def max_average(nums, k):
    window = sum(nums[:k])
    best = window
    for right in range(k, len(nums)):
        window += nums[right] - nums[right - k]
        best = max(best, window)
    return best / k


assert max_average([1, 12, -5, -6, 50, 3], 4) == 12.75
assert max_average([5], 1) == 5.0                    # edge: single element
assert max_average([-1, -2, -3], 2) == -1.5          # all negative
```

O(n) time, O(1) space. The naive version (sum every window) is O(n·k).

## Variable-size window: the template

Most variable-window problems fit one template:

```text
left = 0
for right in range(n):
    add nums[right] to the window state
    while the window is invalid:
        remove nums[left] from the window state
        left += 1
    the window [left, right] is now valid: update the answer
```

For "longest valid window" problems you update the answer after shrinking. For "shortest
window that satisfies a condition" you update *inside* the shrink loop, while the window
is still valid.

## Worked problem 1: longest substring without repeating characters

**Problem.** Return the length of the longest substring of `s` with all distinct
characters.

**Approach.** Keep the last index where each character appeared. When `s[right]` was
already seen *inside the current window*, jump `left` past that earlier occurrence.

```python
def length_of_longest_substring(s):
    last_seen = {}
    left = best = 0
    for right, ch in enumerate(s):
        if ch in last_seen and last_seen[ch] >= left:
            left = last_seen[ch] + 1
        last_seen[ch] = right
        best = max(best, right - left + 1)
    return best


assert length_of_longest_substring("abcabcbb") == 3      # "abc"
assert length_of_longest_substring("bbbbb") == 1
assert length_of_longest_substring("pwwkew") == 3        # "wke"
assert length_of_longest_substring("") == 0              # edge: empty
assert length_of_longest_substring("abba") == 2          # stale index must be ignored
```

The `"abba"` case is the classic trap. At the final `a`, `last_seen["a"] = 0`, but
`left` is already 2. Without the `>= left` check, `left` would jump *backward* to 1 and
the answer would be wrong (3 instead of 2).

## Worked problem 2: minimum size subarray sum

**Problem.** Given positive integers `nums` and `target`, return the length of the
shortest contiguous subarray whose sum is ≥ `target`, or 0 if none exists.

```python
def min_subarray_len(target, nums):
    left = total = 0
    best = float("inf")
    for right, x in enumerate(nums):
        total += x
        while total >= target:                 # valid: try to shrink
            best = min(best, right - left + 1)
            total -= nums[left]
            left += 1
    return 0 if best == float("inf") else best


assert min_subarray_len(7, [2, 3, 1, 2, 4, 3]) == 2       # [4, 3]
assert min_subarray_len(4, [1, 4, 4]) == 1
assert min_subarray_len(11, [1, 1, 1, 1, 1, 1, 1, 1]) == 0   # edge: impossible
assert min_subarray_len(3, [3]) == 1
```

**Important restriction:** this works because all numbers are **positive**, so growing
the window always increases the sum and shrinking always decreases it. With negative
numbers the monotonicity breaks and you need prefix sums (Level 2, lesson 7) or a
monotonic deque (Level 3, lesson 5).

## Worked problem 3: longest repeating character replacement

**Problem.** You may replace at most `k` characters in `s` (uppercase letters). Return
the length of the longest substring containing a single repeated letter afterwards.

**Insight.** A window is valid if `window_length - count_of_most_frequent_letter ≤ k`:
you replace everything that is not the majority letter.

```python
from collections import defaultdict

def character_replacement(s, k):
    counts = defaultdict(int)
    left = max_freq = best = 0
    for right, ch in enumerate(s):
        counts[ch] += 1
        max_freq = max(max_freq, counts[ch])
        while (right - left + 1) - max_freq > k:
            counts[s[left]] -= 1
            left += 1
        best = max(best, right - left + 1)
    return best


assert character_replacement("ABAB", 2) == 4
assert character_replacement("AABABBA", 1) == 4
assert character_replacement("", 3) == 0
assert character_replacement("AAAA", 0) == 4
```

Notice `max_freq` is never decreased when the window shrinks. That is deliberate: the
answer can only improve when a window achieves a *higher* max frequency than before, so a
stale (too-high) `max_freq` never produces a wrong answer — it only means the window
stops growing until a genuinely better one appears. This subtlety is worth being able to
explain.

## How It Actually Works

The nested `while` inside the `for` loop looks quadratic, but it is not. The key is an
**amortized** (aggregate) argument: `right` moves forward n times, and `left` also only
moves forward, never back, and never past `right + 1`. So across the whole run `left`
increments at most n times in total. The total work is O(n + n) = O(n), no matter how
the increments are distributed across iterations.

The pattern is correct because of **monotonicity**: if a window `[l, r]` is invalid,
then every larger window `[l', r]` with `l' < l` is also invalid (for "at most k
distinct", a superset window has at least as many distinct characters; for positive
sums, a superset has a larger sum). That is what makes it safe to never move `left`
backward. When you are unsure whether a sliding window applies, ask: "if a window is
bad, is every window containing it also bad?" If yes, slide. If not (e.g. sums with
negative numbers), you need another tool.

## Common mistakes

- Using a window on problems with negative numbers where the validity condition is not
  monotonic.
- Forgetting to update the answer at the right moment (after shrinking for "longest",
  during shrinking for "shortest").
- Recomputing `len(set(window))` or `sum(window)` each step — that reintroduces O(k)
  work per move. Maintain counts incrementally.
- Off-by-one in window length: it is `right - left + 1`.
- Letting `left` move backward (the `"abba"` bug).

## Variations to practice

- Permutation in string / find all anagrams (fixed window + 26-count array).
- Longest substring with at most k distinct characters.
- Minimum window substring (variable window with a "needed" counter).
- Max consecutive ones with at most k flips.
- Count subarrays with exactly k distinct values = atMost(k) − atMost(k−1).

## Exercise

Write `find_anagrams(s, p)` returning every start index in `s` where an anagram of `p`
begins, in O(len(s)) time. Use a fixed window of size `len(p)` and compare count arrays
(or track a "matches" counter so each step is O(1)). Test with `s="cbaebabacd", p="abc"`
→ `[0, 6]`, with `p` longer than `s`, and with `s="abab", p="ab"` → `[0, 1, 2]`.
