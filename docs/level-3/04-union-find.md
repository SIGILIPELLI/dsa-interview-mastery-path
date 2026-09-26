---
description: "Implement union-find with path compression and union by size, and apply it to connected components, redundant edges, account merging and grid islands."
---

# 04 · Union-Find (Disjoint Set Union)

Union-find maintains a collection of **disjoint sets** under two operations:

- `find(x)` — which set is `x` in? (returns a representative, the set's "root")
- `union(x, y)` — merge the sets containing `x` and `y`.

With two standard optimizations, both run in nearly constant amortized time. It is the
tool of choice for **dynamic connectivity**: edges arrive one at a time and you keep
asking "are these two connected?" or "how many groups are there now?". BFS/DFS would have
to re-traverse after every addition.

## Implementation

```python
class DSU:
    def __init__(self, n):
        self.parent = list(range(n))       # each element starts as its own root
        self.size = [1] * n
        self.components = n

    def find(self, x):
        root = x
        while self.parent[root] != root:
            root = self.parent[root]
        while self.parent[x] != root:      # path compression
            self.parent[x], x = root, self.parent[x]
        return root

    def union(self, a, b):
        ra, rb = self.find(a), self.find(b)
        if ra == rb:
            return False                   # already connected
        if self.size[ra] < self.size[rb]:  # union by size: small under large
            ra, rb = rb, ra
        self.parent[rb] = ra
        self.size[ra] += self.size[rb]
        self.components -= 1
        return True

    def connected(self, a, b):
        return self.find(a) == self.find(b)


d = DSU(5)
assert d.union(0, 1) and d.union(3, 4)
assert d.connected(0, 1) and not d.connected(1, 3)
assert not d.union(1, 0)                   # redundant
assert d.components == 3
d.union(1, 4)
assert d.connected(0, 3) and d.size[d.find(0)] == 4
```

The iterative `find` avoids recursion limits. `union` returning whether a merge happened
is the detail that makes several problems one-liners.

## Worked problem 1: number of connected components

```python
def count_components(n, edges):
    dsu = DSU(n)
    for a, b in edges:
        dsu.union(a, b)
    return dsu.components


assert count_components(5, [(0, 1), (1, 2), (3, 4)]) == 2
assert count_components(5, [(0, 1), (1, 2), (2, 3), (3, 4)]) == 1
assert count_components(3, []) == 3                   # edge: no edges
```

## Worked problem 2: redundant connection

**Problem.** A tree on nodes 1..n had one extra edge added. Return the edge that can be
removed to make it a tree again (if several, the last one in the input).

**Approach.** Add edges in order; the first edge whose endpoints are already connected
closes a cycle. Because edges are processed in input order, it is the last cycle edge in
the input that we meet — exactly what the problem asks.

```python
def find_redundant_connection(edges):
    dsu = DSU(len(edges) + 1)
    for a, b in edges:
        if not dsu.union(a, b):
            return [a, b]
    return []


assert find_redundant_connection([[1, 2], [1, 3], [2, 3]]) == [2, 3]
assert find_redundant_connection([[1, 2], [2, 3], [3, 4], [1, 4], [1, 5]]) == [1, 4]
```

## Worked problem 3: accounts merge

**Problem.** Each account is `[name, email1, email2, ...]`. Two accounts belong to the
same person if they share any email. Merge them; output each person's name followed by
their sorted emails.

**Approach.** Give each email an id and union all emails within one account. Emails in
the same set belong to the same person.

```python
from collections import defaultdict

def accounts_merge(accounts):
    email_id, owner = {}, {}
    for acc in accounts:
        for email in acc[1:]:
            if email not in email_id:
                email_id[email] = len(email_id)
                owner[email] = acc[0]
    dsu = DSU(len(email_id))
    for acc in accounts:
        first = email_id[acc[1]]
        for email in acc[2:]:
            dsu.union(first, email_id[email])
    groups = defaultdict(list)
    for email, i in email_id.items():
        groups[dsu.find(i)].append(email)
    return sorted([owner[g[0]]] + sorted(g) for g in groups.values())


accounts = [["John", "js@mail.com", "john@mail.com"],
            ["John", "js@mail.com", "jnew@mail.com"],
            ["Mary", "mary@mail.com"],
            ["John", "johnnybravo@mail.com"]]
assert accounts_merge(accounts) == [
    ["John", "jnew@mail.com", "john@mail.com", "js@mail.com"],
    ["John", "johnnybravo@mail.com"],
    ["Mary", "mary@mail.com"]]
assert accounts_merge([["A", "a@x"]]) == [["A", "a@x"]]
```

Two different people can share a name, which is why we union by **email**, not by name.

## Worked problem 4: islands added one at a time

**Problem.** An `m × n` grid starts as water. Positions are turned into land one by one.
After each addition, report the number of islands.

A fresh BFS after each addition costs O(m·n) per step. Union-find handles each addition
in near-constant time.

```python
def num_islands_online(m, n, positions):
    dsu = DSU(m * n)
    land = set()
    islands, out = 0, []
    for r, c in positions:
        if (r, c) not in land:
            land.add((r, c))
            islands += 1
            for nr, nc in ((r + 1, c), (r - 1, c), (r, c + 1), (r, c - 1)):
                if (nr, nc) in land and dsu.union(r * n + c, nr * n + nc):
                    islands -= 1                  # merged two islands into one
        out.append(islands)
    return out


assert num_islands_online(3, 3, [(0, 0), (0, 1), (1, 2), (2, 1)]) == [1, 1, 2, 3]
assert num_islands_online(3, 3, [(0, 0), (0, 2), (0, 1)]) == [1, 2, 1]   # bridge
assert num_islands_online(1, 1, [(0, 0), (0, 0)]) == [1, 1]              # duplicate
```

## How It Actually Works

Each set is stored as a **tree** of parent pointers; the root is the representative.
`find` walks up to the root, so its cost is the depth of the element. The two
optimizations keep depth tiny:

**Union by size (or rank).** Always attach the smaller tree under the larger root. An
element's depth increases only when its tree is attached under another, and that only
happens when the other tree is at least as large — so its tree size at least doubles each
time. A size can double at most log₂ n times, so depth is at most log₂ n. That alone
gives O(log n) per operation.

**Path compression.** After finding the root, point every node on the search path
directly at it. Future finds on those nodes take one step. Paths flatten over time.

**Together:** a sequence of m operations on n elements costs O(m · α(n)), where α is the
inverse Ackermann function. α grows so slowly that it is at most 4 for any input size
that could exist in practice, so the amortized cost is "effectively constant". The proof
(due to Tarjan) is intricate and never asked in interviews; what you should be able to
explain is the doubling argument for union by size and what path compression does.

**What union-find cannot do.** It only merges — there is no efficient "split" or
"delete edge". For deletions, a common trick is to process events **in reverse** (turning
deletions into additions). It also does not give you paths between elements, only
whether they are connected.

## Common mistakes

- Comparing `parent[a] == parent[b]` instead of `find(a) == find(b)`.
- Forgetting union by size/rank, leading to long chains on adversarial inputs.
- Recursive `find` on 10⁵+ elements without compression → recursion depth errors.
- Decrementing the component count even when `union` did not merge anything.
- Mapping 2-D coordinates to ids incorrectly (`r * n + c`, where `n` is the column count).

## Variations to practice

- Number of provinces (adjacency matrix input).
- Satisfiability of equality equations (`a==b`, `b!=c`).
- Smallest string with swaps (union positions, sort each group).
- Most stones removed with same row or column.
- Kruskal's minimum spanning tree (lesson 9).

## Exercise

Solve **Satisfiability of Equality Equations**: given strings like `"a==b"` and `"b!=a"`,
decide whether all can hold simultaneously. Process all `==` first with unions over the
26 letters, then check every `!=` for a conflict. Explain why the two passes must be in
that order. Test `["a==b","b!=a"]` → False, `["b==a","a==b"]` → True, `["a!=a"]` → False,
and a chain `["a==b","b==c","a!=c"]` → False.
