---
description: "Apply bit manipulation in interviews: XOR tricks, counting and clearing set bits, subset masks, bitmask DP, and how Python's unbounded ints behave."
---

# 06 · Bit Manipulation

Bit manipulation problems are short, and they reward knowing a handful of identities.
They also show up inside other techniques: bitmasks as compact sets for DP over subsets,
and XOR for pairing and cancellation.

## The operators

| Operator | Meaning | Example (4-bit) |
|---|---|---|
| `a & b` | AND — 1 where both are 1 | `1100 & 1010 = 1000` |
| `a \| b` | OR — 1 where either is 1 | `1100 \| 1010 = 1110` |
| `a ^ b` | XOR — 1 where they differ | `1100 ^ 1010 = 0110` |
| `~a` | NOT — in Python, `-a - 1` | `~5 == -6` |
| `a << k` | shift left, multiply by 2ᵏ | `3 << 2 == 12` |
| `a >> k` | shift right, floor-divide by 2ᵏ | `13 >> 2 == 3` |

## Identities worth memorizing

```python
x = 0b101100                        # 44

assert (x >> 3) & 1 == 1            # test bit 3
assert x | (1 << 1) == 0b101110     # set bit 1
assert x & ~(1 << 2) == 0b101000    # clear bit 2
assert x ^ (1 << 0) == 0b101101     # toggle bit 0
assert x & (x - 1) == 0b101000      # clear the lowest set bit
assert x & -x == 0b000100           # isolate the lowest set bit
assert (16 & 15) == 0 and (12 & 11) != 0   # n & (n-1) == 0  <=> power of two (n > 0)

# XOR properties: a ^ a == 0, a ^ 0 == a, commutative and associative
assert 7 ^ 7 == 0 and 7 ^ 0 == 7 and (3 ^ 5) ^ 3 == 5
```

## Worked problem 1: single number

**Problem.** Every element appears twice except one. Find it in O(n) time, O(1) space.

**Approach.** XOR everything. Pairs cancel (`a ^ a = 0`) regardless of order, leaving the
single element.

```python
from functools import reduce
from operator import xor

def single_number(nums):
    return reduce(xor, nums, 0)


assert single_number([2, 2, 1]) == 1
assert single_number([4, 1, 2, 1, 2]) == 4
assert single_number([-3]) == -3                  # negatives work too
```

### Follow-up: every element appears three times except one

Count, for each bit position, how many numbers have it set. Positions where the count is
not a multiple of 3 belong to the single number. Python integers are unbounded, so handle
the sign by working in 32-bit two's complement explicitly:

```python
def single_number_thrice(nums):
    result = 0
    for bit in range(32):
        count = sum((n >> bit) & 1 for n in nums)
        if count % 3:
            result |= 1 << bit
    if result >= 1 << 31:                   # reinterpret as signed 32-bit
        result -= 1 << 32
    return result


assert single_number_thrice([2, 2, 3, 2]) == 3
assert single_number_thrice([0, 1, 0, 1, 0, 1, 99]) == 99
assert single_number_thrice([-2, -2, 1, 1, 4, 1, 4, 4, -4, -2]) == -4
```

O(32 · n) time, O(1) space.

## Worked problem 2: counting bits

**Problem.** For every `i` in `0..n`, return the number of 1 bits in `i`.

**DP with bits.** `i >> 1` has the same bits as `i` except the last, so
`bits[i] = bits[i >> 1] + (i & 1)`.

```python
def count_bits(n):
    bits = [0] * (n + 1)
    for i in range(1, n + 1):
        bits[i] = bits[i >> 1] + (i & 1)
    return bits


assert count_bits(5) == [0, 1, 1, 2, 1, 2]
assert count_bits(0) == [0]
assert all(count_bits(64)[i] == bin(i).count("1") for i in range(65))
```

(`int.bit_count()` exists from Python 3.10 for a single number; the DP shows the idea.)

## Worked problem 3: missing number

**Problem.** `nums` contains `n` distinct numbers from `0..n`. Which one is missing?

```python
def missing_number(nums):
    result = len(nums)
    for i, x in enumerate(nums):
        result ^= i ^ x                      # every present number cancels its index
    return result


assert missing_number([3, 0, 1]) == 2
assert missing_number([0, 1]) == 2          # edge: missing the top value
assert missing_number([1]) == 0             # edge: missing zero
```

The sum formula `n(n+1)/2 - sum(nums)` also works; XOR is the answer when an interviewer
asks for one that cannot overflow in fixed-width languages.

## Masks as sets: subset enumeration

With `n ≤ ~20` items, a subset is an `n`-bit integer: bit `i` set means item `i` is in.
Iterating `mask` from 0 to `2ⁿ - 1` enumerates every subset.

```python
def subsets_bitmask(nums):
    n = len(nums)
    return [[nums[i] for i in range(n) if mask >> i & 1] for mask in range(1 << n)]


assert sorted(map(sorted, subsets_bitmask([1, 2, 3]))) == sorted(
    [[], [1], [2], [3], [1, 2], [1, 3], [2, 3], [1, 2, 3]])
assert subsets_bitmask([]) == [[]]
```

## Bitmask DP: shortest route visiting every node

**Problem.** In a small undirected graph (n ≤ 12), return the length of the shortest walk
that visits every node (you may start anywhere, revisit nodes, and reuse edges).

**State:** `(node, visited_mask)`. BFS over these states, because every move costs 1.
There are `n · 2ⁿ` states — about 49,000 for n = 12 — which is small.

```python
from collections import deque

def shortest_path_all_nodes(graph):
    n = len(graph)
    full = (1 << n) - 1
    queue = deque((i, 1 << i, 0) for i in range(n))
    seen = {(i, 1 << i) for i in range(n)}
    while queue:
        node, mask, dist = queue.popleft()
        if mask == full:
            return dist
        for nxt in graph[node]:
            state = (nxt, mask | (1 << nxt))
            if state not in seen:
                seen.add(state)
                queue.append((nxt, state[1], dist + 1))
    return -1


assert shortest_path_all_nodes([[1, 2, 3], [0], [0], [0]]) == 4
assert shortest_path_all_nodes([[1], [0, 2, 4], [1, 3, 4], [2], [1, 2]]) == 4
assert shortest_path_all_nodes([[]]) == 0                    # edge: one node
```

## How It Actually Works

**Two's complement.** Fixed-width machines represent a negative number `-x` as
`2ʷ - x` in `w` bits, which is the same as "invert all bits of `x`, then add 1". That is
why `~x == -x - 1` and why `x & -x` isolates the lowest set bit: `-x` flips every bit
above the lowest 1 and keeps that 1 and the zeros below it, so AND leaves only that bit.

**`x & (x - 1)`.** Subtracting 1 turns the lowest set bit into 0 and all zeros below it
into 1s; bits above are unchanged. AND-ing with `x` therefore clears exactly the lowest
set bit. Repeating until zero counts set bits in O(number of set bits) — Kernighan's
method.

**Python's integers are unbounded.** Python behaves as if negative numbers had an
infinite number of leading 1 bits. So `-1 >> 5` is still `-1`, `~0` is `-1`, and a loop
like `while n: n >>= 1` never terminates for negative `n`. When a problem assumes 32-bit
integers, mask explicitly with `& 0xFFFFFFFF` and convert back to signed as in
`single_number_thrice`. In Java/C++ you would instead get overflow and need to know the
difference between arithmetic (`>>`) and logical (`>>>` in Java) right shifts.

**Why XOR cancels.** XOR is addition modulo 2 on each bit independently. It is
associative and commutative, every value is its own inverse, and 0 is the identity. So
XOR-ing a multiset is order-independent, and every value that appears an even number of
times contributes nothing.

## Common mistakes

- Operator precedence. In Python, arithmetic (`+`, `-`) binds tighter than shifts,
  shifts bind tighter than `&`, then `^`, then `|`, and all of them bind tighter than
  comparisons. So in Python `n & n - 1 == 0` parses as `(n & (n - 1)) == 0`, but the
  same expression means something different in C and Java, where `==` binds tighter
  than `&`. Write the
  parentheses every time: `(x >> i) & 1`, `(n & (n - 1)) == 0`.
- Infinite loops shifting negative numbers right in Python.
- Forgetting `n > 0` in the power-of-two check (`0 & -1 == 0`).
- Using bitmasks when `n` is too large (2³⁰ states is not feasible).
- Assuming `~x` gives the unsigned complement in Python.

## Variations to practice

- Reverse bits of a 32-bit unsigned integer.
- Sum of two integers without `+` (carry with AND and shift; mask to 32 bits in Python).
- Single number III (two unique numbers: split by the lowest set bit of their XOR).
- Maximum XOR of two numbers (binary trie, or greedy prefix sets).
- Traveling salesman on ≤ 15 cities (bitmask DP).

## Exercise

Solve **Single Number III**: exactly two elements appear once and all others twice; find
both in O(n) time and O(1) space. XOR everything to get `a ^ b`, isolate any set bit
(`diff & -diff`) — `a` and `b` differ there — and partition the numbers by that bit,
XOR-ing each group. Test with `[1,2,1,3,2,5]` → `{3,5}`, `[-1,0]`, and `[0,1]`.
