---
description: "Write correct recursive solutions: base cases, trusting the recursive call, recursion trees, memoization, and Python's recursion limit and call-stack cost."
---

# 08 · Recursion Fundamentals

Recursion is a function solving a problem by calling itself on smaller instances. It is
the foundation for trees, graphs, backtracking and dynamic programming, so being fluent
with it now pays off across all of Level 2.

Every recursive function has two parts:

1. **Base case(s)** — inputs small enough to answer directly, without recursing.
2. **Recursive case** — reduce the problem to one or more smaller problems, call yourself,
   and combine the results.

## The "leap of faith"

The most useful mental model: when writing the recursive case, **assume the recursive
call already works** for smaller inputs, and only ask "given that, how do I finish the
job for this input?" Do not try to trace every level in your head.

**Problem.** Compute `x` to the power `n` (integer `n ≥ 0`) in O(log n) multiplications.

**Approach.** If you trust `power(x, n // 2)` to be correct, then
`xⁿ = (x^(n//2))²` when `n` is even, and `x · (x^(n//2))²` when `n` is odd.

```python
def power(x, n):
    if n == 0:
        return 1
    half = power(x, n // 2)          # computed once, not twice
    if n % 2 == 0:
        return half * half
    return half * half * x


assert power(2, 10) == 1024
assert power(3, 0) == 1              # edge: base case
assert power(5, 1) == 5
assert power(2, 31) == 2 ** 31
```

Calling `power(x, n // 2)` twice instead of storing it in `half` would make two calls per
level, turning O(log n) into O(n) — a very common slip.

## Recursion trees and complexity

To analyze a recursive function, draw the **recursion tree**: one node per call, with
children for the calls it makes. Total work = sum of work over all nodes.

```python
def fib_naive(n):
    if n < 2:
        return n
    return fib_naive(n - 1) + fib_naive(n - 2)


assert [fib_naive(i) for i in range(8)] == [0, 1, 1, 2, 3, 5, 8, 13]
```

Each call spawns two more, and the tree is about `n` levels deep, so there are roughly
2ⁿ calls (the exact growth rate is about 1.618ⁿ). `fib_naive(40)` makes hundreds of
millions of calls. But the tree is full of *repeated* subproblems — `fib(n-2)` is computed
from both `fib(n)` and `fib(n-1)`. Caching results (**memoization**) collapses the tree
to n distinct calls:

```python
from functools import lru_cache

@lru_cache(maxsize=None)
def fib(n):
    if n < 2:
        return n
    return fib(n - 1) + fib(n - 2)


assert fib(80) == 23416728348467685
assert fib(0) == 0 and fib(1) == 1
```

That single idea — recursion plus a cache — *is* top-down dynamic programming, which
Level 2, lesson 6 builds on.

## Worked problem: generate all subsets

**Problem.** Return every subset of a list of distinct integers.

**Approach.** For each element, there are two choices: include it or not. Recurse on the
index; the base case is having decided about every element.

```python
def subsets(nums):
    result, current = [], []

    def build(i):
        if i == len(nums):
            result.append(current.copy())      # copy: current keeps changing
            return
        build(i + 1)                           # exclude nums[i]
        current.append(nums[i])
        build(i + 1)                           # include nums[i]
        current.pop()                          # undo the choice

    build(0)
    return result


assert sorted(subsets([1, 2, 3])) == sorted(
    [[], [1], [2], [3], [1, 2], [1, 3], [2, 3], [1, 2, 3]])
assert subsets([]) == [[]]                     # edge: one subset, the empty one
```

**Complexity:** 2ⁿ subsets, each copied in O(n): **O(n · 2ⁿ) time**, O(n) recursion
depth (plus the output). The append/recurse/pop shape is the heart of backtracking
(Level 2, lesson 5).

## Worked problem: flatten a nested list

Recursion shines on self-similar data.

```python
def flatten(items):
    out = []
    for item in items:
        if isinstance(item, list):
            out.extend(flatten(item))          # trust the recursive call
        else:
            out.append(item)
    return out


assert flatten([1, [2, [3, [4]], 5]]) == [1, 2, 3, 4, 5]
assert flatten([]) == []
assert flatten([[[]]]) == []
```

## Converting recursion to iteration

When depth can exceed Python's limit, simulate the call stack yourself:

```python
def flatten_iter(items):
    out, stack = [], [iter(items)]
    while stack:
        for item in stack[-1]:
            if isinstance(item, list):
                stack.append(iter(item))       # "call" into the sublist
                break
            out.append(item)
        else:
            stack.pop()                        # this level is exhausted: "return"
    return out


deep = current = []
for _ in range(5000):                          # nesting deeper than the recursion limit
    nxt = []
    current.append(nxt)
    current = nxt
current.append(42)
assert flatten_iter(deep) == [42]
assert flatten_iter([1, [2, [3]], 4]) == [1, 2, 3, 4]
```

The `for ... else` runs the `else` only when the loop finishes without `break` — here
meaning "no more items at this level".

## How It Actually Works

Each call creates a **frame** holding its arguments, local variables, and where to
resume in the caller. Frames live on the call stack: a call pushes a frame, a return pops
it. That is why recursion uses O(depth) space even when it allocates nothing else, and
why an infinite recursion eventually fails.

CPython caps the depth with a recursion limit (`sys.getrecursionlimit()`, 1000 by
default) and raises `RecursionError` when it is exceeded. You can raise it with
`sys.setrecursionlimit`, but very deep recursion can then overflow the underlying C
stack and crash the interpreter, so in interviews it is better to mention the limit and,
for inputs that could be deep (a degenerate tree with 10⁵ nodes, a linked list), offer an
iterative version. Python also does **not** perform tail-call optimization, so writing a
function in tail-recursive form does not save stack space the way it can in some
functional languages.

`functools.lru_cache` works by building a dictionary keyed on the call's arguments.
Before running the function body, the wrapper looks up the arguments; on a hit it returns
the stored result. That is why arguments must be hashable (no lists) and why the cache
turns an exponential tree into one call per distinct argument tuple.

## Common mistakes

- Missing or unreachable base case → `RecursionError`.
- Recursive call that does not shrink the problem (`f(n)` calling `f(n)`).
- Appending the shared `current` list instead of a copy — every stored subset ends up
  identical (and empty).
- Forgetting to undo a choice (`current.pop()`) after exploring it.
- Passing slices (`nums[1:]`) down the recursion — O(n) copy per call; pass an index.
- Using `@lru_cache` on a function that takes a list argument (unhashable).

## Variations to practice

- Reverse a string recursively; check a palindrome recursively.
- Merge sort (lesson 9) — two recursive calls plus a linear merge.
- Tower of Hanoi; count its moves (2ⁿ − 1).
- Letter combinations of a phone number.
- Climbing stairs (1 or 2 steps) with memoization.

## Exercise

Write `permutations(nums)` that returns every ordering of a list of distinct integers
using the choose/recurse/undo shape (track which indices are used with a boolean list).
Assert the output for `[1, 2, 3]` has 6 unique permutations, that `[]` gives `[[]]`, and
that `[1]` gives `[[1]]`. Then draw the recursion tree for `[1, 2, 3]` and use it to
explain the O(n · n!) time bound.
