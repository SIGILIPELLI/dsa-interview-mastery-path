---
description: "Tactics for timed online coding assessments: triage, reading constraints, fast input parsing, partial credit, stress testing, and avoiding common timeouts."
---

# 04 · Online Assessment Tactics

An **online assessment (OA)** is a timed coding test taken alone on a web platform,
usually as an early screening step. Formats vary: commonly two to four problems in 60–120
minutes, with hidden test cases that score your submission automatically. Some include
multiple-choice questions or debugging tasks. Always read the instructions the platform
gives you — they are the only authoritative description of your particular test.

OAs reward different habits from live interviews. Nobody hears your reasoning; what
counts is passing test cases within time limits. This lesson covers the tactics that
matter in that setting.

## Before the test

- **Practise in the same conditions:** a browser editor, a timer, no autocomplete, and
  problems you have not seen.
- **Know your language's I/O.** Some platforms give you a function signature to fill in;
  others require reading from standard input and printing to standard output. Be ready
  for both.
- **Prepare your environment** (quiet room, stable connection, charged laptop), and read
  the platform's rules about what is allowed. Do not use prohibited resources — beyond
  the ethics, assessments may include plagiarism detection and follow-up interviews where
  you will be asked about your code.

## During the test: triage first

1. **Read every problem** before starting any (spend 3–5 minutes). Difficulty is not
   always in order.
2. **Start with the one you are most confident about.** Early full marks reduce pressure.
3. **Budget time per problem** and set checkpoints. If a problem is eating double its
   budget, submit the best partial solution and move on.

## Read the constraints like a spec

Constraints tell you the intended complexity (Level 1, lesson 1). In a timed test this is
the fastest way to rule out approaches.

- `n ≤ 20` → exponential search is intended.
- `n ≤ 2000` → O(n²) is fine.
- `n ≤ 2 · 10⁵` → O(n log n) or O(n); O(n²) will time out.
- Values up to `10⁹` with `n` small → think math, binary search on the answer, or
  coordinate compression.
- "Answer modulo 10⁹+7" → counting problem, usually DP or combinatorics; the numbers get
  huge.

## Input parsing in Python

When a problem reads from standard input, reading line by line with `input()` can be slow
for large inputs. Read everything at once and split:

```python
import io
import sys


def solve(data):
    tokens = data.split()
    n = int(tokens[0])
    nums = list(map(int, tokens[1:1 + n]))
    return max(nums) - min(nums)


def main():
    data = sys.stdin.read()
    print(solve(data))


# Local testing without a real stdin:
assert solve("5\n3 9 1 4 7\n") == 8
assert solve("1\n42") == 0                      # edge: single number
sys.stdin = io.StringIO("3\n-1 -5 2\n")
main()                                          # prints 7
sys.stdin = sys.__stdin__
```

Separating `solve(data)` from I/O makes it trivial to test locally with string inputs.
For large outputs, build a list of strings and print `"\n".join(...)` once rather than
calling `print` thousands of times.

## Partial credit strategy

Hidden tests usually include small and large cases. If you cannot find the optimal
solution:

1. **Submit a correct brute force first.** It often passes the small cases and banks
   points.
2. Then optimize, keeping the brute force as a reference to compare against.

A correct O(n²) that passes most tests generally beats an ambitious O(n log n) that is
wrong on all of them. Check your platform's scoring rules — some score per test case,
others only full solutions.

## Stress testing against a brute force

When your optimized solution fails hidden tests and you cannot see why, generate random
small inputs and compare against the brute force. It finds counterexamples in seconds.

```python
import random

def max_subarray_brute(nums):
    return max(sum(nums[i:j + 1]) for i in range(len(nums)) for j in range(i, len(nums)))


def max_subarray_fast(nums):                      # Kadane's algorithm
    best = cur = nums[0]
    for x in nums[1:]:
        cur = max(x, cur + x)
        best = max(best, cur)
    return best


random.seed(0)
for _ in range(500):
    arr = [random.randint(-10, 10) for _ in range(random.randint(1, 8))]
    assert max_subarray_fast(arr) == max_subarray_brute(arr), arr
```

Small value ranges and short lengths make failures easy to read. Including the input in
the assertion message (`, arr`) prints the counterexample directly.

## Common causes of lost points

- **Time limit exceeded** from hidden O(n²): `in` on a list, `list.pop(0)`, string `+=`
  in loops, slicing inside loops, recursion without memoization.
- **Recursion depth** errors on large inputs — use iteration, or raise the limit with
  `sys.setrecursionlimit` knowing it may not be enough.
- **Wrong answer on edge cases**: empty input, n = 1, all equal, maximum values, negative
  numbers, answer does not exist.
- **Modulo mistakes**: forgetting to apply `% MOD` after each multiplication (not a
  correctness problem in Python, but huge integers become slow).
- **Output format**: extra spaces, wrong separators, printing a list instead of
  space-separated values, missing trailing newline requirements.

## How It Actually Works

Automated judges run your program against a set of test inputs, compare its output with
the expected output (exact match, or a checker for problems with multiple valid
answers), and enforce a time and memory limit per test. Time limits are set by the
problem author, typically with some slack over a reference solution; some platforms scale
limits per language because interpreted languages like Python are slower than compiled
ones, but not all do. That is why constraint-based complexity estimation matters: if the
intended solution is O(n log n) and yours is O(n²), no amount of micro-optimization will
save it on the large tests.

Hidden tests are designed to cover the edge cases authors expect people to miss: minimum
sizes, maximum sizes, duplicates, degenerate structures (a tree that is a line, a graph
with no edges). If your solution passes the visible examples but fails hidden ones, the
cause is almost always one of those edge cases or a complexity problem on the largest
inputs — which is exactly what stress testing and constraint reading target.

## Exercise

Simulate an OA: choose three unseen problems (one easy, two medium) from lesson 5, set a
75-minute timer, and follow the triage process. For each problem, write the brute force
first and keep it. After the timer, write a random stress test comparing your final
solution with the brute force for at least 500 random small inputs, and record whether it
found any bugs your hand-written tests missed.
