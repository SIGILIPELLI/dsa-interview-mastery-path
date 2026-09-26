---
description: "Test and debug code during a live interview: choosing test cases, tracing with a variable table, spotting common bug classes, and fixing calmly."
---

# 07 · Testing & Debugging Live

In many live interviews you cannot run your code, or running it is allowed but there is
no debugger. Either way, the interviewer is watching *how* you verify your work. "I think
it's correct" is the weakest possible ending; a methodical trace that catches your own
bug is one of the strongest.

## Choosing test cases

You cannot test everything, so pick cases that each target a different risk:

1. **The given example** — confirms the main logic.
2. **Smallest inputs** — empty, one element, two elements. Catches index errors and
   missing base cases.
3. **Boundary structure** — all equal values, already sorted, reverse sorted, a tree that
   is a single path, a graph with no edges.
4. **Values at extremes** — negatives, zeros, very large numbers, duplicates.
5. **"No answer" cases** — target not present, impossible configuration.
6. **The case you designed around** — if your code has a special branch (like the
   `"abba"` stale-index check), exercise it.

Say *why* you chose each one: "Now an input with all duplicates, because my skip-logic
is the part I'm least sure of."

## Tracing with a variable table

Trace the **code you wrote**, not the algorithm you intended. Write a small table with
one column per variable and one row per iteration. It is slower than eyeballing, and far
more reliable.

Take this (buggy) function meant to return the index of the first element ≥ `target`:

```text
def lower_bound(nums, target):
    lo, hi = 0, len(nums) - 1
    while lo < hi:
        mid = (lo + hi) // 2
        if nums[mid] >= target:
            hi = mid
        else:
            lo = mid + 1
    return lo
```

Trace `nums = [1, 3, 5]`, `target = 9` (the "not present, larger than everything" case):

| step | lo | hi | mid | nums[mid] | branch |
|---|---|---|---|---|---|
| start | 0 | 2 | – | – | – |
| 1 | 0 | 2 | 1 | 3 | 3 < 9 → lo = 2 |
| end | 2 | 2 | – | – | loop exits |

It returns 2, but the correct answer is 3 (past the end). The bug: `hi` starts at
`len(nums) - 1`, so "past the end" is not in the search space. Fix: `hi = len(nums)`.

```python
def lower_bound(nums, target):
    lo, hi = 0, len(nums)
    while lo < hi:
        mid = (lo + hi) // 2
        if nums[mid] >= target:
            hi = mid
        else:
            lo = mid + 1
    return lo


assert lower_bound([1, 3, 5], 9) == 3
assert lower_bound([1, 3, 5], 3) == 1
assert lower_bound([1, 3, 5], 0) == 0
assert lower_bound([], 4) == 0
```

The given example (`target = 3`) would have passed the buggy version. Only the boundary
case exposed the bug — which is why case selection matters more than the number of cases.

## Bug classes to check deliberately

Before tracing, scan your code for the usual suspects. Most interview bugs come from a
short list:

| Bug class | Where it hides | Quick check |
|---|---|---|
| Off-by-one | loop bounds, slice ends, `mid` updates | trace size 0, 1, 2 |
| Uninitialized / wrong initial value | `best = 0` when all values are negative | test all-negative input |
| Aliasing | `result.append(path)` without copy; `[[0]*n]*m` | test output with 2+ items |
| Missing state reset | counters not cleared between iterations | trace two iterations |
| Wrong comparison | `<` vs `<=`, strict vs non-strict | test duplicates / touching intervals |
| Mutating while iterating | removing from a list or dict in a `for` loop | reason about the iterator |
| Unhandled "no answer" | returning `None` implicitly | test impossible input |

Here is an example of the "wrong initial value" class, and its fix:

```python
def max_subarray_buggy(nums):
    best = cur = 0                       # bug: assumes some sum >= 0
    for x in nums:
        cur = max(x, cur + x)
        best = max(best, cur)
    return best


def max_subarray(nums):
    best = cur = nums[0]                 # start from a real element
    for x in nums[1:]:
        cur = max(x, cur + x)
        best = max(best, cur)
    return best


assert max_subarray_buggy([-3, -1, -2]) == 0          # wrong: empty subarray not allowed
assert max_subarray([-3, -1, -2]) == -1
assert max_subarray([-2, 1, -3, 4, -1, 2, 1, -5, 4]) == 6
assert max_subarray([5]) == 5
```

## When you find a bug

1. **Say it out loud:** "This returns 2 but should return 3 — the search space excludes
   the end."
2. **Explain the cause**, not just the symptom.
3. **Make the smallest fix** that addresses the cause; avoid rewriting the whole function.
4. **Re-trace the failing case**, then quickly re-check one case that previously passed,
   to make sure the fix did not break it.

Patching symptoms (adding `if target > nums[-1]: return len(nums)`) might pass the test
but signals that you did not understand the bug. Interviewers notice the difference.

## If you are allowed to run code

Still write the tests first, as `assert` statements, so you are checking against expected
values you decided in advance rather than just eyeballing printed output. If a test fails,
add a targeted `print` of the few variables that matter at the suspect point, rather than
printing everything.

## How It Actually Works

Debugging is a **search problem**: somewhere between the input and the wrong output, the
program's state diverged from what you intended. Tracing with a variable table compares
actual state to intended state step by step, so the first row where they differ localizes
the bug. That is why tracing works even without a computer, and why it is more reliable
than re-reading code (re-reading tends to check what you *meant*, which is exactly where
the bug is not visible).

Test-case selection works on the idea of **equivalence classes**: inputs that exercise the
same code paths tend to fail or pass together, so one representative per class gives most
of the value. Boundaries between classes — empty vs one element, strictly less vs equal —
are where bugs concentrate, because that is where the code's conditions switch. A good
small test suite is one case per class plus one per boundary.

## Common mistakes

- Tracing the idea instead of the code.
- Only testing the example from the problem statement.
- Silent fixing: changing code without explaining what was wrong.
- Adding special-case `if` statements for failing inputs instead of fixing the cause.
- Running out of time because testing was left until the last minute — reserve roughly
  five minutes.

## Exercise

Take three of your own solutions from earlier lessons. For each, (1) write a list of six
test cases using the categories above, with a one-line reason for each; (2) deliberately
introduce one bug from the bug-class table; (3) ask a friend (or wait a day) and then find
it by tracing only, with a variable table, without running the code. Record how long each
took and which test case exposed it.
