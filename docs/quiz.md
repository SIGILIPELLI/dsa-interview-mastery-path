---
title: "Final Quiz"
description: "Test what you have learned across all four levels of DSA \u0026 Coding Interviews Mastery Path with a 18-question final quiz."
---

# Final Quiz

18 questions drawn from all four levels of this course. You need **70% (13 out of 18)** to pass —
you can retake it as many times as you like. Your best result is saved in this browser.

<div id="mp-quiz-root"></div>

<script type="application/json" id="mp-quiz-data">
[
  {
    "question": "What does this print?\n```python\ndef f(n):\n    c=0\n    for i in range(1,n):\n        j=1\n        while j<n: j*=2; c+=1\n    return c\nprint(f(16))\n```\n",
    "options": [
      "256",
      "16",
      "60",
      "30"
    ],
    "correctIndex": 2,
    "explanation": "An outer loop of n-1 iterations times log n inner steps: O(n log n). Verified by running it."
  },
  {
    "question": "What does this print?\n```python\ndef two_sum(a,t):\n    seen={}\n    for i,x in enumerate(a):\n        if t-x in seen: return [seen[t-x], i]\n        seen[x]=i\nprint(two_sum([3,2,4],6))\n```\n",
    "options": [
      "[0, 1]",
      "[1, 2]",
      "[0, 2]",
      "None"
    ],
    "correctIndex": 1,
    "explanation": "A hash map of complements gives O(n). Verified by running it."
  },
  {
    "question": "What does this print?\n```python\ndef maxsub(a,k):\n    w=sum(a[:k]); b=w\n    for i in range(k,len(a)): w+=a[i]-a[i-k]; b=max(b,w)\n    return b\nprint(maxsub([1,4,2,10,2,3,1,0,20],4))\n```\n",
    "options": [
      "16",
      "20",
      "19",
      "24"
    ],
    "correctIndex": 3,
    "explanation": "Fixed-size sliding window of 4. Verified by running it."
  },
  {
    "question": "Given a sorted array, which technique finds a pair with a given sum in O(n) time and O(1) space?",
    "options": [
      "Sorting again",
      "Nested loops",
      "Hash map",
      "Two pointers from both ends"
    ],
    "correctIndex": 3,
    "explanation": "Move left or right based on the sum."
  },
  {
    "question": "What does this print?\n```python\nfrom collections import deque\nq=deque()\nfor x in [1,2,3]: q.append(x)\nq.appendleft(0)\nprint(q.pop(), q.popleft(), list(q))\n```\n",
    "options": [
      "1 3 [0, 2]",
      "0 3 [1, 2]",
      "3 1 [2]",
      "3 0 [1, 2]"
    ],
    "correctIndex": 3,
    "explanation": "deque works at both ends in O(1). Verified by running it."
  },
  {
    "question": "What does this print (reverse linked list)?\n```python\nclass N:\n    def __init__(s,v,n=None): s.v=v; s.next=n\nh=N(1,N(2,N(3,N(4))))\np=None\nwhile h:\n    nxt=h.next; h.next=p; p=h; h=nxt\nout=[]\nwhile p: out.append(p.v); p=p.next\nprint(out)\n```\n",
    "options": [
      "[4, 3, 2, 1]",
      "[1, 2, 3, 4]",
      "Error",
      "[4]"
    ],
    "correctIndex": 0,
    "explanation": "Save next before re-pointing. Verified by running it."
  },
  {
    "question": "What does this print?\n```python\nimport heapq\nh=[]\nfor x in [7,2,9,4,1]:\n    heapq.heappush(h,x)\n    if len(h)>2: heapq.heappop(h)\nprint(sorted(h))\n```\n",
    "options": [
      "[1, 2]",
      "[7, 9]",
      "[1, 9]",
      "[2, 4]"
    ],
    "correctIndex": 1,
    "explanation": "A size-2 min-heap retains the two largest values. Verified by running it."
  },
  {
    "question": "What does this print (BFS shortest path length in a grid)?\n```python\nfrom collections import deque\ng=['S.#','..#','.E.']\nq=deque([(0,0,0)]); seen={(0,0)}; ans=-1\nwhile q:\n    r,c,d=q.popleft()\n    if g[r][c]=='E': ans=d; break\n    for dr,dc in((1,0),(-1,0),(0,1),(0,-1)):\n        nr,nc=r+dr,c+dc\n        if 0<=nr<3 and 0<=nc<3 and g[nr][nc]!='#' and (nr,nc) not in seen:\n            seen.add((nr,nc)); q.append((nr,nc,d+1))\nprint(ans)\n```\n",
    "options": [
      "3",
      "-1",
      "4",
      "2"
    ],
    "correctIndex": 0,
    "explanation": "BFS finds shortest paths in unweighted graphs. Verified by running it."
  },
  {
    "question": "What does this print (coin change min coins)?\n```python\ndef coins(cs,t):\n    dp=[0]+[10**9]*t\n    for a in range(1,t+1):\n        for c in cs:\n            if c<=a: dp[a]=min(dp[a],dp[a-c]+1)\n    return dp[t]\nprint(coins([1,3,4],6))\n```\n",
    "options": [
      "6",
      "3",
      "1",
      "2"
    ],
    "correctIndex": 3,
    "explanation": "Greedy (4+1+1) gives 3, DP finds 3+3 = 2. Verified by running it."
  },
  {
    "question": "What does this print (merge intervals)?\n```python\niv=sorted([[1,4],[4,5],[7,9],[8,10]])\nout=[iv[0]]\nfor s,e in iv[1:]:\n    if s<=out[-1][1]: out[-1][1]=max(out[-1][1],e)\n    else: out.append([s,e])\nprint(out)\n```\n",
    "options": [
      "[[1,10]]",
      "[[1,4],[4,5],[7,9],[8,10]]",
      "[[1,5],[7,9],[8,10]]",
      "[[1, 5], [7, 10]]"
    ],
    "correctIndex": 3,
    "explanation": "Touching intervals merge when s <= previous end. Verified by running it."
  },
  {
    "question": "What does this print (Dijkstra distances)?\n```python\nimport heapq\ng={'A':[('B',2),('C',5)],'B':[('C',1),('D',7)],'C':[('D',2)],'D':[]}\nd={k:99 for k in g}; d['A']=0; h=[(0,'A')]\nwhile h:\n    du,u=heapq.heappop(h)\n    if du>d[u]: continue\n    for v,w in g[u]:\n        if du+w<d[v]: d[v]=du+w; heapq.heappush(h,(d[v],v))\nprint(d['D'])\n```\n",
    "options": [
      "9",
      "5",
      "8",
      "7"
    ],
    "correctIndex": 1,
    "explanation": "A->B->C->D = 2+1+2. Verified by running it."
  },
  {
    "question": "What does this print (union-find components)?\n```python\np=list(range(7))\ndef find(x):\n    while p[x]!=x: p[x]=p[p[x]]; x=p[x]\n    return x\nfor a,b in [(0,1),(2,3),(1,3),(4,5)]: p[find(a)]=find(b)\nprint(len({find(i) for i in range(7)}))\n```\n",
    "options": [
      "4",
      "2",
      "3",
      "5"
    ],
    "correctIndex": 2,
    "explanation": "Components: {0,1,2,3}, {4,5}, {6}. Verified by running it."
  },
  {
    "question": "What does this print (single number)?\n```python\nr=0\nfor x in [7,3,5,3,7]: r^=x\nprint(r)\n```\n",
    "options": [
      "3",
      "5",
      "7",
      "0"
    ],
    "correctIndex": 1,
    "explanation": "XOR cancels duplicate pairs. Verified by running it."
  },
  {
    "question": "A problem asks for the number of ways to climb stairs taking 1 or 2 steps. Which approach fits?",
    "options": [
      "Greedy",
      "BFS",
      "Dynamic programming (Fibonacci recurrence)",
      "Binary search"
    ],
    "correctIndex": 2,
    "explanation": "Overlapping subproblems."
  },
  {
    "question": "You need all permutations of a list. Which technique fits?",
    "options": [
      "Union-find",
      "Two pointers",
      "Greedy",
      "Backtracking: choose, explore, unchoose"
    ],
    "correctIndex": 3,
    "explanation": "Generate candidates and undo."
  },
  {
    "question": "You need autocomplete suggestions for a prefix over 1M words. Which structure?",
    "options": [
      "A stack",
      "A heap",
      "A list scan",
      "A trie"
    ],
    "correctIndex": 3,
    "explanation": "Prefix queries in O(L)."
  },
  {
    "question": "Edges have negative weights. Which shortest path algorithm do you use?",
    "options": [
      "Dijkstra",
      "BFS",
      "Kruskal",
      "Bellman-Ford"
    ],
    "correctIndex": 3,
    "explanation": "Dijkstra assumes non-negative weights."
  },
  {
    "question": "In an interview you notice your first idea is O(n^2) with n = 10^5. What do you do?",
    "options": [
      "Ignore",
      "Hardcode",
      "Code it",
      "Say so and look for a better pattern (hash map, two pointers, sorting, heap) before coding"
    ],
    "correctIndex": 3,
    "explanation": "Check constraints against complexity."
  }
]
</script>

## Coding challenges

Try these on your own computer, then compare your output. Nothing you write is sent anywhere.

### Top K and longest unique substring

Implement top_k_frequent(words, k) with a heap of (-count, word) so ties are alphabetical, and longest_unique(s) with a sliding window and a last-seen map.

**Starter code**

```python
import heapq
from collections import Counter

def top_k_frequent(words, k):
    # k most frequent words, ties broken alphabetically (use a heap of (-count, word))
    return []

def longest_unique(s):
    # length of the longest substring without repeating characters (sliding window)
    return 0

print(top_k_frequent("the day is sunny the the sunny is is".split(), 3))
print(top_k_frequent(["b", "a", "b", "a", "c"], 2))
print(longest_unique("abcabcbb"), longest_unique("pwwkew"), longest_unique(""))
```

**Expected output**

```text
['is', 'the', 'sunny']
['a', 'b']
3 3 0
```

<details><summary>Show solution and explanation</summary>

<pre><code>import heapq
from collections import Counter

def top_k_frequent(words, k):
    counts = Counter(words)
    # heap of (-count, word): most frequent first, ties alphabetical
    heap = [(-c, w) for w, c in counts.items()]
    heapq.heapify(heap)
    return [heapq.heappop(heap)[1] for _ in range(min(k, len(heap)))]

def longest_unique(s):
    last, left, best = {}, 0, 0
    for right, ch in enumerate(s):
        if ch in last and last[ch] &gt;= left:
            left = last[ch] + 1
        last[ch] = right
        best = max(best, right - left + 1)
    return best

print(top_k_frequent(&quot;the day is sunny the the sunny is is&quot;.split(), 3))
print(top_k_frequent([&quot;b&quot;, &quot;a&quot;, &quot;b&quot;, &quot;a&quot;, &quot;c&quot;], 2))
print(longest_unique(&quot;abcabcbb&quot;), longest_unique(&quot;pwwkew&quot;), longest_unique(&quot;&quot;))
</code></pre>

<p>The heap orders by count descending then word ascending; the sliding window moves `left` past the previous occurrence of a repeated character, so each element is processed once: O(n). Verified by running the solution.</p>
</details>

### Course schedule and island count

Implement can_finish(n, prereqs) with Kahn's algorithm (return whether all courses can be taken and the order) and count_islands(grid) with an iterative DFS.

**Starter code**

```python
from collections import deque

def can_finish(n, prereqs):
    # return (possible, order) using Kahn topological sort; prereqs are (course, prerequisite)
    return (False, [])

def count_islands(grid):
    # number of connected groups of "1" cells (4-directional), without recursion
    return 0

print(can_finish(4, [(1, 0), (2, 1), (3, 2)]))
print(can_finish(2, [(1, 0), (0, 1)]))
print(count_islands(["11000", "11000", "00100", "00011"]))
```

**Expected output**

```text
(True, [0, 1, 2, 3])
(False, [])
3
```

<details><summary>Show solution and explanation</summary>

<pre><code>from collections import deque

def can_finish(n, prereqs):
    &quot;&quot;&quot;Course schedule: topological sort with cycle detection (Kahn).&quot;&quot;&quot;
    g = {i: [] for i in range(n)}
    indeg = [0] * n
    for a, b in prereqs:          # b must come before a
        g[b].append(a)
        indeg[a] += 1
    q = deque(i for i in range(n) if indeg[i] == 0)
    order = []
    while q:
        u = q.popleft()
        order.append(u)
        for v in g[u]:
            indeg[v] -= 1
            if indeg[v] == 0:
                q.append(v)
    return (len(order) == n), order

def count_islands(grid):
    rows, cols = len(grid), len(grid[0])
    seen = set()
    def dfs(r, c):
        stack = [(r, c)]
        while stack:
            r, c = stack.pop()
            if (r, c) in seen or not (0 &lt;= r &lt; rows and 0 &lt;= c &lt; cols) or grid[r][c] != &quot;1&quot;:
                continue
            seen.add((r, c))
            stack += [(r + 1, c), (r - 1, c), (r, c + 1), (r, c - 1)]
    n = 0
    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == &quot;1&quot; and (r, c) not in seen:
                n += 1
                dfs(r, c)
    return n

print(can_finish(4, [(1, 0), (2, 1), (3, 2)]))
print(can_finish(2, [(1, 0), (0, 1)]))
print(count_islands([&quot;11000&quot;, &quot;11000&quot;, &quot;00100&quot;, &quot;00011&quot;]))
</code></pre>

<p>If Kahn&#x27;s algorithm outputs fewer nodes than exist, the remaining nodes form a cycle. Counting islands launches one flood fill per unvisited land cell. Verified by running the solution.</p>
</details>
