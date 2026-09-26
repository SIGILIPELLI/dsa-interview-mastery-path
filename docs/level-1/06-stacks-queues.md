---
description: "Use stacks and queues to solve bracket matching, Reverse Polish evaluation, min-stack and queue-via-stacks problems, with deque for O(1) operations."
---

# 06 · Stacks & Queues

A **stack** is last-in, first-out (LIFO): the most recent item is the first to leave.
A **queue** is first-in, first-out (FIFO). Both restrict access on purpose, and that
restriction is exactly what makes them fit certain problems.

- Use a stack when the most recent unfinished thing must be resolved first: matching
  brackets, undo, nested structures, evaluating expressions, depth-first search.
- Use a queue when things must be processed in arrival order: breadth-first search, task
  scheduling, streaming buffers.

In Python, a `list` is a good stack (`append` and `pop` at the end are O(1)). For a
queue, use `collections.deque`, which gives O(1) `append`, `appendleft`, `pop`, and
`popleft`. Do **not** use `list.pop(0)` as a queue: it is O(n).

## Worked problem 1: valid parentheses

**Problem.** Given a string of `()[]{}` characters, decide whether every bracket is
closed by the right type in the right order.

**Approach.** Push openers. On a closer, the top of the stack must be its matching
opener. At the end, the stack must be empty.

```python
def is_valid(s):
    pairs = {")": "(", "]": "[", "}": "{"}
    stack = []
    for ch in s:
        if ch in pairs:
            if not stack or stack[-1] != pairs[ch]:
                return False
            stack.pop()
        else:
            stack.append(ch)
    return not stack


assert is_valid("()[]{}")
assert is_valid("{[()]}")
assert not is_valid("(]")
assert not is_valid("([)]")
assert not is_valid("(((")          # edge: unclosed openers
assert not is_valid(")")            # edge: closer with empty stack
assert is_valid("")                 # edge: empty string is balanced
```

O(n) time, O(n) space in the worst case (all openers).

## Worked problem 2: evaluate Reverse Polish Notation

**Problem.** Evaluate an expression in postfix form, e.g. `["2","1","+","3","*"]` means
`(2 + 1) * 3 = 9`. Division truncates toward zero.

```python
def eval_rpn(tokens):
    stack = []
    for tok in tokens:
        if tok in {"+", "-", "*", "/"}:
            b = stack.pop()
            a = stack.pop()          # order matters for - and /
            if tok == "+":
                stack.append(a + b)
            elif tok == "-":
                stack.append(a - b)
            elif tok == "*":
                stack.append(a * b)
            else:
                stack.append(int(a / b))   # truncate toward zero
        else:
            stack.append(int(tok))
    return stack[0]


assert eval_rpn(["2", "1", "+", "3", "*"]) == 9
assert eval_rpn(["4", "13", "5", "/", "+"]) == 6
assert eval_rpn(["-7", "2", "/"]) == -3          # truncation, not floor
assert eval_rpn(["42"]) == 42                    # edge: single number
```

**Python trap:** `-7 // 2` is `-4` because `//` floors. Many problems specify truncation
toward zero, which is `int(a / b)` (safe for the magnitudes in typical problems; for very
large integers, compute with `abs` and fix the sign to avoid float precision issues).

## Worked problem 3: min stack

**Problem.** Design a stack supporting `push`, `pop`, `top`, and `get_min`, all in O(1).

**Approach.** Store, alongside each value, the minimum of the stack *at the time it was
pushed*. When you pop, the previous minimum is automatically restored.

```python
class MinStack:
    def __init__(self):
        self._items = []                 # (value, min_so_far)

    def push(self, x):
        current_min = min(x, self._items[-1][1]) if self._items else x
        self._items.append((x, current_min))

    def pop(self):
        return self._items.pop()[0]

    def top(self):
        return self._items[-1][0]

    def get_min(self):
        return self._items[-1][1]


ms = MinStack()
ms.push(-2); ms.push(0); ms.push(-3)
assert ms.get_min() == -3
ms.pop()
assert ms.top() == 0 and ms.get_min() == -2
ms.push(-2)                              # duplicate of the minimum
ms.pop()
assert ms.get_min() == -2
```

## Worked problem 4: queue using two stacks

**Problem.** Implement a FIFO queue using only stack operations, with amortized O(1) per
operation.

```python
class QueueViaStacks:
    def __init__(self):
        self._in, self._out = [], []

    def push(self, x):
        self._in.append(x)

    def _shift(self):
        if not self._out:
            while self._in:
                self._out.append(self._in.pop())

    def pop(self):
        self._shift()
        return self._out.pop()

    def peek(self):
        self._shift()
        return self._out[-1]

    def empty(self):
        return not self._in and not self._out


q = QueueViaStacks()
q.push(1); q.push(2)
assert q.peek() == 1
assert q.pop() == 1
q.push(3)
assert q.pop() == 2 and q.pop() == 3 and q.empty()
```

Each element is moved from `_in` to `_out` **at most once**, so n operations cost O(n)
in total: amortized O(1) each, even though a single `pop` can take O(n).

## Queues in practice: `deque`

```python
from collections import deque

def recent_calls(timestamps, window=3000):
    """For each ping time t, count pings in [t - window, t]. Times are increasing."""
    q, out = deque(), []
    for t in timestamps:
        q.append(t)
        while q[0] < t - window:
            q.popleft()
        out.append(len(q))
    return out


assert recent_calls([1, 100, 3001, 3002]) == [1, 2, 3, 3]
assert recent_calls([]) == []
```

This is a queue acting as a sliding window over time. Breadth-first search (Level 2)
uses a `deque` in exactly the same way.

## How It Actually Works

**List as a stack.** Appending and popping at the end of a dynamic array touch only the
last slot, so both are O(1) (append amortized). Popping from the front must shift every
remaining reference left by one: O(n).

**`deque` as a queue.** CPython's `deque` is a **doubly linked list of fixed-size
blocks** (each block holds dozens of item pointers). The deque tracks the left and right
block plus an index within each. Adding on either end writes into the current end block,
or links a new block when it fills up; removing works symmetrically. No shifting ever
happens, so both ends are O(1). The trade-off: indexing into the middle (`d[i]`) must
walk blocks, so it is O(n) — use a deque for the ends, a list for random access.

**Why stacks model recursion.** Every function call pushes a frame (local variables and
return address) on the call stack; returning pops it. Any recursive algorithm can be
rewritten iteratively by managing an explicit stack yourself — useful in Python, where
the default recursion limit is about 1,000 frames.

## Common mistakes

- Popping from an empty stack: check `if not stack` first, especially for closers.
- Returning `True` at the end of bracket matching without checking the stack is empty.
- Wrong operand order for `-` and `/` in RPN (the first pop is the *right* operand).
- Using `list.pop(0)` or `list.insert(0, x)` as a queue — quadratic over n operations.
- Using `//` when the problem says "truncate toward zero".

## Variations to practice

- Decode string (`"3[a2[c]]"` → `"accaccacc"`) with a stack of (string, count).
- Simplify a Unix path (`"/a/./b/../../c/"` → `"/c"`).
- Basic calculator with `+`, `-` and parentheses.
- Asteroid collision.
- Daily temperatures — the gateway to monotonic stacks (Level 3, lesson 5).

## Exercise

Implement `decode_string(s)` for inputs like `"3[a]2[bc]"` → `"aaabcbc"` and
`"3[a2[c]]"` → `"accaccacc"`. Use a stack that stores `(previous_string, repeat_count)`
when you see `[`, and multi-digit counts like `"10[a]"`. Add asserts for an input with no
brackets and one with adjacent groups, and state the complexity in terms of the *output*
length.
