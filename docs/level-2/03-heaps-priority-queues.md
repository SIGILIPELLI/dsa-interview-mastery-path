---
description: "Use heaps and heapq for top-k, merging k sorted lists and a running median, and see how sift-up, sift-down and O(n) heapify actually work."
---

# 03 · Heaps & Priority Queues

A **priority queue** hands out items in order of priority rather than arrival. The
standard implementation is a **binary heap**, which gives:

| Operation | Cost |
|-----------|------|
| peek at min | O(1) |
| push | O(log n) |
| pop min | O(log n) |
| build from n items | O(n) |

**Signals:** "k largest/smallest", "k closest", "merge k sorted", "schedule by earliest
deadline", "running median", "repeatedly take the best remaining option". Any time you
would re-sort after every change, a heap is probably the answer.

## `heapq` essentials

Python's `heapq` module turns a plain list into a **min-heap**:

```python
import heapq

h = []
for x in [5, 1, 8, 3]:
    heapq.heappush(h, x)
assert h[0] == 1                       # smallest is always at index 0
assert heapq.heappop(h) == 1
assert heapq.heappop(h) == 3

nums = [9, 4, 7, 1]
heapq.heapify(nums)                    # in place, O(n)
assert nums[0] == 1

# Max-heap: negate the keys.
mh = []
for x in [5, 1, 8]:
    heapq.heappush(mh, -x)
assert -mh[0] == 8

# Tuples compare element by element; add a counter to break ties
# between objects that are not comparable themselves.
tasks = []
heapq.heappush(tasks, (2, 0, "write report"))
heapq.heappush(tasks, (1, 1, "fix bug"))
assert heapq.heappop(tasks)[2] == "fix bug"
```

## Worked problem 1: k largest elements (top-k)

**Problem.** Return the k-th largest element of an unsorted list.

**Options:**

- Sort: O(n log n).
- Max-heap of everything, pop k times: O(n + k log n).
- **Min-heap of size k**: keep the k largest seen so far; the root is the smallest of
  them, i.e. the k-th largest. O(n log k) time, O(k) space — best when k ≪ n or the data
  is a stream.

```python
import heapq

def kth_largest(nums, k):
    heap = []
    for x in nums:
        heapq.heappush(heap, x)
        if len(heap) > k:
            heapq.heappop(heap)          # discard the smallest of k+1
    return heap[0]


assert kth_largest([3, 2, 1, 5, 6, 4], 2) == 5
assert kth_largest([3, 2, 3, 1, 2, 4, 5, 5, 6], 4) == 4
assert kth_largest([7], 1) == 7                  # edge: single element
assert kth_largest([2, 2, 2], 3) == 2            # duplicates
```

(`heapq.nlargest(k, nums)[-1]` does the same in one line; know what it does underneath.)

## Worked problem 2: top k frequent elements

```python
import heapq
from collections import Counter

def top_k_frequent(nums, k):
    counts = Counter(nums)
    return [x for x, _ in heapq.nlargest(k, counts.items(), key=lambda p: p[1])]


assert sorted(top_k_frequent([1, 1, 1, 2, 2, 3], 2)) == [1, 2]
assert top_k_frequent([4], 1) == [4]
```

O(n + m log k) for m distinct values. A **bucket sort** by frequency (index = count) gets
O(n), since frequencies are bounded by n — a good follow-up to mention.

## Worked problem 3: merge k sorted lists

**Problem.** Merge k sorted lists into one sorted list.

**Approach.** Keep a heap holding the current front element of each list. Pop the
smallest, output it, and push the next element from the same list.

```python
import heapq

def merge_k_sorted(lists):
    heap = [(lst[0], i, 0) for i, lst in enumerate(lists) if lst]
    heapq.heapify(heap)
    out = []
    while heap:
        val, i, j = heapq.heappop(heap)
        out.append(val)
        if j + 1 < len(lists[i]):
            heapq.heappush(heap, (lists[i][j + 1], i, j + 1))
    return out


assert merge_k_sorted([[1, 4, 5], [1, 3, 4], [2, 6]]) == [1, 1, 2, 3, 4, 4, 5, 6]
assert merge_k_sorted([]) == []
assert merge_k_sorted([[], [1]]) == [1]          # edge: some lists empty
```

**Complexity:** N total elements, heap of size ≤ k: **O(N log k)** time, O(k) extra space.
The list index `i` in the tuple breaks ties so Python never compares beyond it. With
linked-list nodes (which are not comparable), the tie-breaker is essential.

## Worked problem 4: running median (two heaps)

**Problem.** Numbers arrive one at a time; after each, report the median.

**Approach.** Keep the smaller half in a **max-heap** (`low`) and the larger half in a
**min-heap** (`high`), with `low` allowed one extra element. The median is `low`'s top,
or the average of both tops when sizes are equal.

```python
import heapq

class RunningMedian:
    def __init__(self):
        self.low = []    # max-heap via negation
        self.high = []   # min-heap

    def add(self, x):
        heapq.heappush(self.low, -x)
        heapq.heappush(self.high, -heapq.heappop(self.low))    # move low's max up
        if len(self.high) > len(self.low):
            heapq.heappush(self.low, -heapq.heappop(self.high))

    def median(self):
        if len(self.low) > len(self.high):
            return -self.low[0]
        return (-self.low[0] + self.high[0]) / 2


rm, medians = RunningMedian(), []
for x in [5, 15, 1, 3]:
    rm.add(x)
    medians.append(rm.median())
assert medians == [5, 10.0, 5, 4.0]
rm2 = RunningMedian()
for x in [2, 2, 2]:
    rm2.add(x)
assert rm2.median() == 2
```

Every `add` does a constant number of heap operations: O(log n). `median` is O(1).
Routing every new value through `low` then `high` guarantees the "every value in `low` ≤
every value in `high`" invariant without case analysis.

## How It Actually Works

**The shape.** A binary heap is a *complete* binary tree stored in an array with no
pointers: the children of index `i` are `2i + 1` and `2i + 2`, and its parent is
`(i - 1) // 2`. Completeness (every level full except possibly the last, filled left to
right) keeps the height at ⌊log₂ n⌋ and makes the array layout gap-free.

**The heap property.** Every parent is ≤ its children (min-heap). That is a much weaker
condition than being sorted — siblings are unordered — which is why maintaining it is
cheaper than maintaining sorted order.

**Push = sift up.** Append the new item at the end (the next free leaf), then while it is
smaller than its parent, swap them. Each swap moves one level up, so at most ⌊log₂ n⌋
swaps.

**Pop = sift down.** Remove the root, move the last item into the root, then repeatedly
swap it with its smaller child until neither child is smaller. Again at most one swap
per level: O(log n). (CPython's `heapq` uses a variant that moves the hole all the way
to a leaf and then sifts the item back up, which saves comparisons on average.)

**Why `heapify` is O(n), not O(n log n).** It sifts down every non-leaf node, from the
last one back to the root. Half the nodes are leaves (0 work), a quarter sit one level
up (at most 1 swap), an eighth two levels up (at most 2), and so on. The total is
n · (1/4 + 2/8 + 3/16 + ...) which converges to at most n: linear. Pushing items one at
a time, by contrast, can cost O(n log n) because *most* nodes are near the bottom and
each push may sift all the way up.

## Common mistakes

- Forgetting `heapq` is a **min**-heap; negate for max-heap behaviour.
- Pushing tuples whose later elements are not comparable (e.g. nodes) without a unique
  tie-breaker → `TypeError` on equal priorities.
- Assuming `heap` is sorted — only `heap[0]` is guaranteed. The second smallest is
  `heap[1]` or `heap[2]`, and beyond that there is no order. Pop to get sorted output.
- Using a max-heap of all n items for top-k when a size-k min-heap suffices.
- Modifying an item's priority in place (the heap does not know). Push a new entry and
  skip stale ones on pop ("lazy deletion").

## Variations to practice

- K closest points to the origin.
- Task scheduler with cooldown.
- Reorganize string so no two adjacent characters are equal.
- Find median from data stream (done above) and sliding window median.
- Meeting rooms II (min-heap of end times — also in lesson 7).
- Dijkstra's algorithm (Level 3, lesson 2) is a heap-driven graph search.

## Exercise

Implement `k_closest(points, k)` returning the k points closest to the origin, in
O(n log k) using a size-k **max**-heap keyed by negative squared distance. Add asserts
for ties in distance, k equal to the number of points, and points on the axes. Then
explain in two sentences why comparing squared distances is safe (no `sqrt` needed).
