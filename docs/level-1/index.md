---
description: "Level 1 of DSA & Coding Interviews: Big-O, arrays, hashing, two pointers, sliding window, stacks, linked lists, recursion and binary search."
---

# Level 1 · Entry <span class="level-badge">Foundations</span>

Goal: build the toolkit that the majority of coding-interview problems are made from.
By the end of this level you should be able to look at a problem about an array or a
string and quickly list two or three candidate approaches, estimate the complexity of
each, and code the best one without hesitating over the basics.

Every topic here is small on its own. The skill being trained is *recognition*: noticing
that "longest substring with at most k distinct characters" is a sliding window, that
"pair summing to a target in a sorted array" is two pointers, and that "have I seen this
before?" is a hash set.

## Modules

1. [Big-O & Complexity Analysis](01-big-o-complexity.md) — counting work, dominant terms, amortized cost, and reading constraints to guess the target complexity
2. [Arrays & Strings](02-arrays-strings.md) — in-place updates, the cost of slicing and concatenation, and the "build a list then join" habit
3. [Hash Maps & Sets](03-hash-maps-sets.md) — trading memory for time, frequency counting, and how hash tables really store keys
4. [Two Pointers](04-two-pointers.md) — opposite-end and fast/slow pointers, and the invariant that makes them correct
5. [Sliding Window](05-sliding-window.md) — fixed and variable windows, and why each element enters and leaves once
6. [Stacks & Queues](06-stacks-queues.md) — matching brackets, evaluating expressions, and `deque` for O(1) queues
7. [Linked Lists](07-linked-lists.md) — pointer surgery, dummy heads, reversal, and cycle detection
8. [Recursion Fundamentals](08-recursion.md) — base cases, the call stack, recursion trees, and Python's recursion limit
9. [Sorting & Binary Search](09-sorting-binary-search.md) — what `sorted` guarantees, and binary search on answers, not just arrays
10. [Project — Timed Practice Set 1](10-project-timed-practice-set.md) — six problems under a clock, with full walkthroughs

## What you need before starting

- Working Python 3: lists, dicts, sets, loops, functions, and slicing. You do not need
  classes beyond a simple node class, which lesson 7 introduces.
- A way to run a Python file (`python3 file.py`). An editor with no autocomplete is
  closer to interview conditions, but not required.
- Comfort with high-school algebra (logarithms appear in lesson 1; the lesson explains
  what you need).

By the end of this level you will have solved roughly thirty problems across the worked
examples and exercises, and you will have timed yourself on a mixed set.
