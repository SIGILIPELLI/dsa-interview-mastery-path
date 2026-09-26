---
description: "Handle object-design coding rounds: design APIs and classes for an LRU cache, a rate limiter and a time-based key-value store, with complexity guarantees."
---

# 06 · System-Design-Lite in Coding Rounds

Some coding rounds ask you to **design and implement a small component** rather than
solve an algorithmic puzzle: "implement an LRU cache", "design a rate limiter", "build a
key-value store with timestamps". These sit between algorithms and full system design.
There are no servers or databases to draw — but you must choose an interface, choose data
structures that meet stated complexity targets, and write clean, testable classes.

(Full distributed system design — load balancers, replication, sharding — is a separate
kind of round and a separate course. This lesson stays inside one process.)

## A process for design-flavoured problems

1. **Pin down the API.** Method names, arguments, return values, and what happens on
   invalid input (missing key? capacity 0?).
2. **Ask for complexity targets.** "Should `get` and `put` both be O(1)?" The target
   usually dictates the data structures.
3. **Choose structures by operation.** List each operation and what it needs: lookup by
   key → hash map; ordering by recency → linked list; ordering by time → sorted list +
   binary search; minimum → heap.
4. **Write the class with small helpers.** Private methods for pointer surgery keep public
   methods readable.
5. **Test through the public API**, including capacity limits and repeated keys.

## Worked problem 1: LRU cache

**Spec.** `LRUCache(capacity)`, `get(key)` returns the value or -1, `put(key, value)`
inserts or updates; when over capacity, evict the **least recently used** key. Both
operations O(1).

**Structures.** A hash map gives O(1) lookup, but not recency order. A doubly linked list
gives O(1) move-to-front and remove-from-back, but not lookup. Combine them: the map
stores key → node, and the list orders nodes by recency. Sentinel head and tail nodes
remove edge cases.

```python
class _Node:
    __slots__ = ("key", "val", "prev", "next")

    def __init__(self, key=0, val=0):
        self.key, self.val = key, val
        self.prev = self.next = None


class LRUCache:
    def __init__(self, capacity):
        self.capacity = capacity
        self.map = {}
        self.head, self.tail = _Node(), _Node()      # head.next = most recent
        self.head.next, self.tail.prev = self.tail, self.head

    def _unlink(self, node):
        node.prev.next, node.next.prev = node.next, node.prev

    def _push_front(self, node):
        node.prev, node.next = self.head, self.head.next
        self.head.next.prev = node
        self.head.next = node

    def get(self, key):
        node = self.map.get(key)
        if node is None:
            return -1
        self._unlink(node)
        self._push_front(node)
        return node.val

    def put(self, key, value):
        if self.capacity <= 0:
            return
        node = self.map.get(key)
        if node:
            node.val = value
            self._unlink(node)
        else:
            if len(self.map) == self.capacity:
                lru = self.tail.prev
                self._unlink(lru)
                del self.map[lru.key]
            node = _Node(key, value)
            self.map[key] = node
        self._push_front(node)


c = LRUCache(2)
c.put(1, 1); c.put(2, 2)
assert c.get(1) == 1            # 1 is now most recent
c.put(3, 3)                     # evicts 2
assert c.get(2) == -1
c.put(4, 4)                     # evicts 1
assert c.get(1) == -1 and c.get(3) == 3 and c.get(4) == 4
c.put(3, 30)                    # update refreshes recency
c.put(5, 5)                     # evicts 4
assert c.get(4) == -1 and c.get(3) == 30
z = LRUCache(0)
z.put(1, 1)
assert z.get(1) == -1           # edge: zero capacity
```

The node stores its key so eviction can delete the map entry. In Python you could use
`collections.OrderedDict` with `move_to_end` and `popitem(last=False)` — mention it, but
be ready to build the linked list if asked, since that is usually the point.

## Worked problem 2: rate limiter (sliding window log)

**Spec.** `allow(user, timestamp)` returns `True` if the user has made fewer than `limit`
allowed requests in the last `window` seconds (`(timestamp - window, timestamp]`), and
records the request if allowed. Timestamps per user are non-decreasing.

```python
from collections import defaultdict, deque

class RateLimiter:
    def __init__(self, limit, window):
        self.limit, self.window = limit, window
        self.log = defaultdict(deque)

    def allow(self, user, timestamp):
        q = self.log[user]
        while q and q[0] <= timestamp - self.window:
            q.popleft()                     # outside the window
        if len(q) < self.limit:
            q.append(timestamp)
            return True
        return False


rl = RateLimiter(limit=2, window=10)
assert rl.allow("a", 1) and rl.allow("a", 2)
assert not rl.allow("a", 5)                 # third within 10s
assert rl.allow("b", 5)                     # users are independent
assert rl.allow("a", 11)                    # t=1 has expired (11 - 10 = 1)
assert not rl.allow("a", 11)
```

Amortized O(1) per call; memory O(limit) per user. Discuss alternatives when asked:

- **Fixed window counter** — one counter per window; O(1) memory but allows bursts of up
  to 2× the limit around window boundaries.
- **Token bucket** — tokens refill at a steady rate up to a cap; allows controlled bursts
  and uses O(1) memory per user.

## Worked problem 3: time-based key-value store

**Spec.** `set(key, value, timestamp)` and `get(key, timestamp)` returning the value set
at the largest timestamp ≤ the given one, or `""`. Timestamps for `set` are strictly
increasing per key.

**Structures.** Per key, keep parallel lists of timestamps and values. Because timestamps
arrive in increasing order, the lists stay sorted, and `get` is a binary search.

```python
from bisect import bisect_right
from collections import defaultdict

class TimeMap:
    def __init__(self):
        self.times = defaultdict(list)
        self.values = defaultdict(list)

    def set(self, key, value, timestamp):
        self.times[key].append(timestamp)
        self.values[key].append(value)

    def get(self, key, timestamp):
        i = bisect_right(self.times[key], timestamp)
        return self.values[key][i - 1] if i else ""


tm = TimeMap()
tm.set("foo", "bar", 1)
assert tm.get("foo", 1) == "bar"
assert tm.get("foo", 3) == "bar"
tm.set("foo", "bar2", 4)
assert tm.get("foo", 4) == "bar2" and tm.get("foo", 5) == "bar2"
assert tm.get("foo", 0) == ""               # edge: before the first set
assert tm.get("missing", 10) == ""          # edge: unknown key
```

`set` is O(1) amortized, `get` is O(log n). If timestamps could arrive out of order, you
would need a sorted insertion (O(n) with a list) or a balanced tree — a good follow-up to
raise yourself.

## How It Actually Works

Design-flavoured problems are really about **composing data structures so that every
operation meets its target**. Each structure is good at some operations and bad at
others; no single one gives O(1) lookup *and* O(1) recency updates. Combining two, and
keeping them consistent, is the core skill. The LRU cache is the canonical example: the
hash map indexes into the linked list, so each operation uses the structure that is fast
for it, and every mutation updates both.

Consistency is where bugs live. Every code path that changes one structure must change
the other: eviction must remove from the list **and** the map; updates must refresh
recency **and** the value. Writing small private helpers (`_unlink`, `_push_front`)
makes each invariant-preserving change happen in exactly one place.

The same reasoning scales up. In a real service, "evict least recently used" is what
caches like the ones in front of databases do, and rate limiting algorithms (sliding
logs, token buckets) are used at API gateways. The trade-offs you mention here — memory
per user versus burst accuracy — are the same ones discussed in full system design
rounds.

## Common mistakes

- Starting to code before agreeing on the API and complexity targets.
- Forgetting to delete the evicted key from the map.
- Treating `get` in an LRU cache as read-only (it must update recency).
- Using a list with `pop(0)` for time windows (O(n) per expiry).
- Not handling capacity 0, unknown keys, or queries before the first timestamp.

## Exercise

Implement an **LFU cache** (evict the least *frequently* used key, breaking ties by least
recently used) with O(1) `get` and `put`. Hint: a map from key to (value, frequency), a
map from frequency to an ordered collection of keys (an `OrderedDict` per frequency is
acceptable), and a running `min_freq`. Test eviction ties, updates to existing keys, and
capacity 1. Then write three sentences comparing the invariants you maintain with those
of the LRU cache.
