---
title: "DSA & Coding Interviews Interview Questions and Answers (178 Questions)"
description: "178 DSA & Coding Interviews interview questions with answers and explanations, from beginner to advanced, organised by level. Free, from the DSA & Coding Interviews Mastery Path."
---

# DSA & Coding Interviews Interview Questions and Answers

**178 questions** with the correct answer and a short explanation, organised from beginner (Level 1) to advanced (Level 4). Every question comes from a lesson in the free DSA & Coding Interviews Mastery Path, so you can open the lesson to go deeper.

!!! tip "Practise, don't just read"
    Each lesson also ends with an interactive quiz, and the [final quiz](quiz.md) tests all four levels. Try to answer each question below before reading the answer.

## Level 1 · Entry

### [01 · Big-O &amp; Complexity Analysis](level-1/01-big-o-complexity.md)

**Q1.** What does this print?

```python
def f(n):
    c=0
    for i in range(n):
        for j in range(i, n):
            c+=1
    return c
print(f(4), f(10))
```

**Answer:** 10 55

n(n+1)/2 iterations: O(n^2). Verified by running it.

**Q2.** What is the time complexity of binary search on a sorted array?

**Answer:** O(log n)

Each step halves the range.

**Q3.** What does Big-O describe?

**Answer:** How running time or space grows with input size, ignoring constants and lower-order terms

Compare growth rates.

**Q4.** What does this print?

```python
def f(n):
    c=0
    while n>1:
        n//=2; c+=1
    return c
print(f(8), f(1000))
```

**Answer:** 3 9

Halving loops run about log2(n) times. Verified by running it.

### [02 · Arrays &amp; Strings](level-1/02-arrays-strings.md)

**Q5.** What does this print?

```python
a=[1,2,3,4,5]
a[1:4]=a[1:4][::-1]
print(a, 'abc'[::-1], 'a-b-c'.split('-'))
```

**Answer:** [1, 4, 3, 2, 5] cba ['a', 'b', 'c']

Slices create copies; assigning a slice replaces a range. Verified by running it.

**Q6.** What does this print?

```python
def prefix(a):
    p=[0]
    for x in a: p.append(p[-1]+x)
    return p
print(prefix([3,1,4,1]))
```

**Answer:** [0, 3, 4, 8, 9]

Prefix sums make range sums O(1). Verified by running it.

**Q7.** Why are string concatenations in a loop O(n^2) in many languages?

**Answer:** Each concatenation copies the whole string; build a list and join

Use ''.join().

**Q8.** What does this print?

```python
s='hello'
print(s[1:3], s[-2:], len(s), s.upper().find('L'))
```

**Answer:** el lo 5 2

Slice end is exclusive. Verified by running it.

### [03 · Hash Maps &amp; Sets](level-1/03-hash-maps-sets.md)

**Q9.** What does this print?

```python
def two_sum(a,t):
    seen={}
    for i,x in enumerate(a):
        if t-x in seen: return seen[t-x], i
        seen[x]=i
print(two_sum([2,7,11,15],9), two_sum([3,2,4],6))
```

**Answer:** (0, 1) (1, 2)

One pass with a complement map is O(n). Verified by running it.

**Q10.** What does this print?

```python
from collections import Counter
c=Counter('mississippi')
print(c.most_common(2), c['z'])
```

**Answer:** [('i', 4), ('s', 4)] 0

Counter returns 0 for missing keys; ties keep insertion order. Verified by running it.

**Q11.** What's the average time of dict/set lookup and insert?

**Answer:** O(1)

Worst case degrades with collisions.

**Q12.** Which structure checks for duplicates in O(n) total?

**Answer:** A set of seen elements

Trade space for time.

### [04 · Two Pointers](level-1/04-two-pointers.md)

**Q13.** What does this print?

```python
def pair_sum(a,t):
    l,r=0,len(a)-1
    while l<r:
        s=a[l]+a[r]
        if s==t: return (l,r)
        if s<t: l+=1
        else: r-=1
print(pair_sum([1,2,4,7,11,15],15), pair_sum([1,2,3],10))
```

**Answer:** (2, 4) None

Two pointers need a sorted array. Verified by running it.

**Q14.** What does this print?

```python
def is_pal(s):
    t=[c.lower() for c in s if c.isalnum()]
    l,r=0,len(t)-1
    while l<r:
        if t[l]!=t[r]: return False
        l+=1; r-=1
    return True
print(is_pal('A man, a plan, a canal: Panama'), is_pal('race a car'))
```

**Answer:** True False

Skip non-alphanumerics. Verified by running it.

**Q15.** What precondition does the opposite-ends two-pointer sum technique need?

**Answer:** The array is sorted

Else sort first or use a hash map.

**Q16.** What's the time and space of two pointers over a sorted array?

**Answer:** O(n) time, O(1) extra space

Each pointer moves at most n steps.

### [05 · Sliding Window](level-1/05-sliding-window.md)

**Q17.** What does this print?

```python
def longest(s):
    seen={}; l=best=0
    for r,c in enumerate(s):
        if c in seen and seen[c]>=l: l=seen[c]+1
        seen[c]=r
        best=max(best,r-l+1)
    return best
print(longest('abcabcbb'), longest('bbbb'), longest('pwwkew'))
```

**Answer:** 3 1 3

Variable-size window with a last-seen index. Verified by running it.

**Q18.** What does this print?

```python
def max_sum(a,k):
    w=sum(a[:k]); best=w
    for i in range(k,len(a)):
        w+=a[i]-a[i-k]; best=max(best,w)
    return best
print(max_sum([2,1,5,1,3,2],3))
```

**Answer:** 9

Fixed window: add the new, drop the old. Verified by running it.

**Q19.** What makes a sliding window efficient?

**Answer:** Each element enters and leaves the window once: O(n) total

Avoid recomputing from scratch.

**Q20.** When does a variable-size window shrink?

**Answer:** When the window violates the constraint

Move left until valid.

### [06 · Stacks &amp; Queues](level-1/06-stacks-queues.md)

**Q21.** What does this print?

```python
def valid(s):
    st=[]; m={')':'(',']':'[','}':'{'}
    for c in s:
        if c in m:
            if not st or st.pop()!=m[c]: return False
        else: st.append(c)
    return not st
print(valid('()[]{}'), valid('(]'), valid('(('))
```

**Answer:** True False False

A stack matches nested brackets. Verified by running it.

**Q22.** What does this print?

```python
from collections import deque
q=deque([1,2,3]); q.append(4); x=q.popleft(); q.appendleft(0)
print(x, list(q))
```

**Answer:** 1 [0, 2, 3, 4]

deque supports O(1) at both ends. Verified by running it.

**Q23.** Why not use list.pop(0) as a queue?

**Answer:** It's O(n) because elements shift; use collections.deque

deque.popleft is O(1).

**Q24.** What problem class suits a stack?

**Answer:** Matching/nesting, undo, DFS and 'next greater' patterns

LIFO.

### [07 · Linked Lists](level-1/07-linked-lists.md)

**Q25.** What does this print?

```python
class N:
    def __init__(s,v,n=None): s.v=v; s.next=n
def rev(h):
    p=None
    while h:
        h.next,p,h=p,h,h.next
    return p
h=N(1,N(2,N(3)))
r=rev(h); out=[]
while r: out.append(r.v); r=r.next
print(out)
```

**Answer:** [3, 2, 1]

Re-point next pointers while walking. Verified by running it.

**Q26.** What does this print (fast/slow pointers)?

```python
class N:
    def __init__(s,v,n=None): s.v=v; s.next=n
h=N(1,N(2,N(3,N(4,N(5)))))
s=f=h
while f and f.next:
    s=s.next; f=f.next.next
print(s.v)
```

**Answer:** 3

Slow stops at the middle. Verified by running it.

**Q27.** How do you detect a cycle in a linked list in O(1) space?

**Answer:** Floyd's tortoise and hare: fast and slow pointers meet if there's a cycle

Hash set works but uses O(n).

**Q28.** Why use a dummy head node?

**Answer:** To handle edge cases like removing the head uniformly

Simplifies pointer code.

### [08 · Recursion Fundamentals](level-1/08-recursion.md)

**Q29.** What does this print?

```python
def fib(n, memo={}):
    if n<2: return n
    if n not in memo: memo[n]=fib(n-1)+fib(n-2)
    return memo[n]
print(fib(10), fib(50))
```

**Answer:** 55 12586269025

Memoization turns exponential into O(n). Verified by running it.

**Q30.** What does this print?

```python
def power(x,n):
    if n==0: return 1
    h=power(x,n//2)
    return h*h if n%2==0 else h*h*x
print(power(2,10), power(3,5))
```

**Answer:** 1024 243

Fast exponentiation is O(log n). Verified by running it.

**Q31.** What are the two essential parts of any recursion?

**Answer:** A base case and a recursive step that moves toward it

Without the base case you overflow.

**Q32.** What happens on infinite recursion in Python?

**Answer:** RecursionError (stack depth limit)

Default limit ~1000.

### [09 · Sorting &amp; Binary Search](level-1/09-sorting-binary-search.md)

**Q33.** What does this print?

```python
def bs(a,t):
    l,r=0,len(a)-1
    while l<=r:
        m=(l+r)//2
        if a[m]==t: return m
        if a[m]<t: l=m+1
        else: r=m-1
    return -1
print(bs([1,3,5,7,9],7), bs([1,3,5],4))
```

**Answer:** 3 -1

Classic binary search. Verified by running it.

**Q34.** What does this print?

```python
import bisect
a=[1,2,2,2,5]
print(bisect.bisect_left(a,2), bisect.bisect_right(a,2), bisect.bisect_left(a,3))
```

**Answer:** 1 4 4

Left insertion point versus right insertion point. Verified by running it.

**Q35.** What does this print?

```python
print(sorted([3,1,2], reverse=True), sorted(['bb','a','ccc'], key=len), sorted([(2,'b'),(1,'z'),(2,'a')]))
```

**Answer:** [3, 2, 1] ['a', 'bb', 'ccc'] [(1, 'z'), (2, 'a'), (2, 'b')]

key= and tuple ordering. Verified by running it.

**Q36.** What's a common binary-search bug?

**Answer:** Off-by-one in bounds and an infinite loop from not moving past mid

Check l&lt;=r and l=m+1.

### [10 · Project — Timed Practice Set 1](level-1/10-project-timed-practice-set.md)

**Q37.** What is the goal of the timed practice set?

**Answer:** Find which patterns you can recognize and execute under pressure

Simulate the interview.

**Q38.** How should you use a time limit on each problem?

**Answer:** Plan, code and test within it, then review your gaps

Practice pacing.

**Q39.** What do you do first on an interview problem?

**Answer:** Clarify the problem and examples, then discuss approach before coding

Communication matters.

**Q40.** How should you grade yourself?

**Answer:** Against walkthroughs, noting missed patterns and bugs

Reflect.

## Level 2 · Intermediate

### [01 · Binary Trees &amp; Traversals](level-2/01-trees-traversals.md)

**Q41.** What does this print?

```python
class T:
    def __init__(s,v,l=None,r=None): s.v=v; s.l=l; s.r=r
t=T(1,T(2,T(4),T(5)),T(3))
def pre(n): return [n.v]+pre(n.l)+pre(n.r) if n else []
def ino(n): return ino(n.l)+[n.v]+ino(n.r) if n else []
def post(n): return post(n.l)+post(n.r)+[n.v] if n else []
print(pre(t), ino(t), post(t))
```

**Answer:** [1, 2, 4, 5, 3] [4, 2, 5, 1, 3] [4, 5, 2, 3, 1]

Root position differs among traversals. Verified by running it.

**Q42.** What does this print?

```python
class T:
    def __init__(s,v,l=None,r=None): s.v=v; s.l=l; s.r=r
t=T(3,T(9),T(20,T(15),T(7)))
def h(n): return 0 if not n else 1+max(h(n.l),h(n.r))
from collections import deque
q=deque([t]); lv=[]
while q:
    lv.append([n.v for n in q]); q=deque(c for n in q for c in (n.l,n.r) if c)
print(h(t), lv)
```

**Answer:** 3 [[3], [9, 20], [15, 7]]

BFS by levels. Verified by running it.

**Q43.** What's the time complexity of a tree traversal?

**Answer:** O(n): each node is visited once

Recursion depth is the height.

**Q44.** Which traversal uses a queue?

**Answer:** Level-order (BFS)

DFS uses a stack/recursion.

### [02 · Binary Search Trees](level-2/02-binary-search-trees.md)

**Q45.** What does this print?

```python
class T:
    def __init__(s,v): s.v=v; s.l=s.r=None
def ins(n,v):
    if not n: return T(v)
    if v<n.v: n.l=ins(n.l,v)
    else: n.r=ins(n.r,v)
    return n
def ino(n): return ino(n.l)+[n.v]+ino(n.r) if n else []
r=None
for v in [5,3,8,1,4,9]: r=ins(r,v)
print(ino(r))
```

**Answer:** [1, 3, 4, 5, 8, 9]

In-order traversal of a BST is sorted. Verified by running it.

**Q46.** What does validating a BST require?

**Answer:** Checking each node lies within the (min, max) bounds inherited from ancestors, not just against its children

Local checks miss violations.

**Q47.** When does a BST degrade to O(n) operations?

**Answer:** When built from sorted input, producing a chain

Balanced trees avoid it.

**Q48.** What's the average search time in a balanced BST?

**Answer:** O(log n)

Height is log n.

### [03 · Heaps &amp; Priority Queues](level-2/03-heaps-priority-queues.md)

**Q49.** What does this print?

```python
import heapq
h=[5,1,8,3]
heapq.heapify(h)
print(heapq.heappop(h), heapq.heappop(h), heapq.nlargest(2,[5,1,8,3]))
```

**Answer:** 1 3 [8, 5]

heapq is a min-heap. Verified by running it.

**Q50.** What does this print?

```python
import heapq
h=[]
for x in [5,1,8,3,9,2]:
    heapq.heappush(h,x)
    if len(h)>3: heapq.heappop(h)
print(sorted(h))
```

**Answer:** [5, 8, 9]

A size-k min-heap keeps the k largest. Verified by running it.

**Q51.** What are push and pop costs in a binary heap?

**Answer:** O(log n) each

heapify is O(n).

**Q52.** How do you simulate a max-heap with heapq?

**Answer:** Push negated values

heappush(h, -x).

### [04 · Graphs: BFS &amp; DFS](level-2/04-graphs-bfs-dfs.md)

**Q53.** What does this print?

```python
from collections import deque
g={1:[2,3],2:[4],3:[4],4:[]}
def bfs(s):
    seen={s}; q=deque([s]); order=[]
    while q:
        n=q.popleft(); order.append(n)
        for m in g[n]:
            if m not in seen: seen.add(m); q.append(m)
    return order
print(bfs(1))
```

**Answer:** [1, 2, 3, 4]

BFS visits by distance. Verified by running it.

**Q54.** What does this print (count islands)?

```python
g=[list('110'),list('010'),list('001')]
def dfs(r,c):
    if 0<=r<3 and 0<=c<3 and g[r][c]=='1':
        g[r][c]='0'
        for dr,dc in((1,0),(-1,0),(0,1),(0,-1)): dfs(r+dr,c+dc)
n=0
for r in range(3):
    for c in range(3):
        if g[r][c]=='1': n+=1; dfs(r,c)
print(n)
```

**Answer:** 2

Flood fill each unvisited land cell. Verified by running it.

**Q55.** Which finds the shortest path in an unweighted graph?

**Answer:** BFS

Layer by layer.

**Q56.** Why track visited nodes?

**Answer:** To avoid infinite loops in cyclic graphs

Mark on enqueue.

### [05 · Backtracking](level-2/05-backtracking.md)

**Q57.** What does this print?

```python
def subsets(a):
    res=[]
    def go(i,cur):
        if i==len(a): res.append(cur[:]); return
        go(i+1,cur)
        cur.append(a[i]); go(i+1,cur); cur.pop()
    go(0,[]); return res
print(subsets([1,2,3]))
```

**Answer:** [[], [3], [2], [2, 3], [1], [1, 3], [1, 2], [1, 2, 3]]

Choose or skip each element; undo after. Verified by running it.

**Q58.** What does this print?

```python
def perms(a):
    res=[]
    def go(cur,left):
        if not left: res.append(cur); return
        for i in range(len(left)): go(cur+[left[i]], left[:i]+left[i+1:])
    go([],a); return res
print(len(perms([1,2,3,4])), perms([1,2,3])[:2])
```

**Answer:** 24 [[1, 2, 3], [1, 3, 2]]

n! permutations. Verified by running it.

**Q59.** What's the core move in backtracking?

**Answer:** Choose, explore, then undo (unchoose)

Prune early.

**Q60.** What does pruning do?

**Answer:** Abandons a partial candidate that can't lead to a solution

Reduces the search.

### [06 · Intro to Dynamic Programming](level-2/06-intro-dynamic-programming.md)

**Q61.** What does this print?

```python
def coins(cs,t):
    dp=[0]+[float('inf')]*t
    for a in range(1,t+1):
        for c in cs:
            if c<=a: dp[a]=min(dp[a],dp[a-c]+1)
    return dp[t] if dp[t]!=float('inf') else -1
print(coins([1,2,5],11), coins([2],3))
```

**Answer:** 3 -1

Bottom-up min coins. Verified by running it.

**Q62.** What does this print?

```python
def climb(n):
    a,b=1,1
    for _ in range(n-1): a,b=b,a+b
    return b
print(climb(5), climb(10))
```

**Answer:** 8 89

Ways to climb with 1 or 2 steps (Fibonacci). Verified by running it.

**Q63.** What two properties signal dynamic programming?

**Answer:** Overlapping subproblems and optimal substructure

Cache repeated work.

**Q64.** How do you define a DP solution?

**Answer:** State, transition, base cases and order of computation

Define the state first.

### [07 · Prefix Sums &amp; Intervals](level-2/07-prefix-sums-intervals.md)

**Q65.** What does this print?

```python
from itertools import accumulate
a=[2,4,6,8,10]
p=[0]+list(accumulate(a))
print(p, p[4]-p[1])
```

**Answer:** [0, 2, 6, 12, 20, 30] 18

Range sum = p[r+1]-p[l]. Verified by running it.

**Q66.** What does this print (merge intervals)?

```python
iv=sorted([[8,10],[1,3],[2,6],[15,18]])
out=[iv[0]]
for s,e in iv[1:]:
    if s<=out[-1][1]: out[-1][1]=max(out[-1][1],e)
    else: out.append([s,e])
print(out)
```

**Answer:** [[1, 6], [8, 10], [15, 18]]

Sort by start, then merge overlaps. Verified by running it.

**Q67.** Why sort intervals by start first?

**Answer:** So overlaps are adjacent and a single pass suffices

O(n log n).

**Q68.** How do prefix sums help subarray-sum-equals-k?

**Answer:** Store counts of prefix sums in a hash map and look up p-k

Works with negatives.

### [08 · Greedy Algorithms](level-2/08-greedy.md)

**Q69.** What does this print?

```python
def jump(a):
    far=cur=jumps=0
    for i in range(len(a)-1):
        far=max(far,i+a[i])
        if i==cur: jumps+=1; cur=far
    return jumps
print(jump([2,3,1,1,4]), jump([1,1,1,1]))
```

**Answer:** 2 3

Greedy: extend the farthest reach. Verified by running it.

**Q70.** What does this print (max non-overlapping intervals)?

```python
iv=sorted([[1,3],[2,4],[3,5],[7,8]], key=lambda x:x[1])
end=-1; c=0
for s,e in iv:
    if s>=end: c+=1; end=e
print(c)
```

**Answer:** 3

Earliest finishing time first. Verified by running it.

**Q71.** Why can greedy fail on coin change with arbitrary coins?

**Answer:** Locally largest coin may not lead to the fewest total coins

E.g. coins {1,3,4} for 6.

**Q72.** How do you justify a greedy choice?

**Answer:** Show an exchange argument or that a local optimum extends to a global one

Prove it.

### [09 · Tries](level-2/09-tries.md)

**Q73.** What does this print?

```python
class Trie:
    def __init__(s): s.c={}; s.end=False
    def add(s,w):
        n=s
        for ch in w: n=n.c.setdefault(ch,Trie())
        n.end=True
    def starts(s,p):
        n=s
        for ch in p:
            if ch not in n.c: return False
            n=n.c[ch]
        return True
t=Trie()
for w in ['apple','app','bat']: t.add(w)
print(t.starts('ap'), t.starts('ba'), t.starts('cat'))
```

**Answer:** True True False

Shared prefixes share nodes. Verified by running it.

**Q74.** What's the time to insert or search a word of length L in a trie?

**Answer:** O(L)

Independent of how many words.

**Q75.** When is a trie better than a hash set of words?

**Answer:** For prefix queries and autocomplete

Hash sets can't do prefixes.

**Q76.** What marks a complete word in a trie node?

**Answer:** An end-of-word flag

Prefix is not a word.

### [10 · Project — Timed Practice Set 2](level-2/10-project-practice-set-2.md)

**Q77.** What does the mixed practice set test?

**Answer:** Recognizing the right pattern without being told

Half the interview.

**Q78.** How do you pick a pattern?

**Answer:** Match constraints and keywords (sorted, subarray, shortest path, count ways) to techniques

Pattern recognition.

**Q79.** What should you do after solving?

**Answer:** State time/space complexity and test edge cases

Interviewers expect it.

**Q80.** What's a good brute-force-first habit?

**Answer:** State the brute force, then improve with a pattern

Shows reasoning.

## Level 3 · Advanced

### [01 · Advanced Dynamic Programming](level-3/01-advanced-dp.md)

**Q81.** What does this print (LCS length)?

```python
def lcs(a,b):
    dp=[[0]*(len(b)+1) for _ in a+' ']
    for i in range(1,len(a)+1):
        for j in range(1,len(b)+1):
            dp[i][j]=dp[i-1][j-1]+1 if a[i-1]==b[j-1] else max(dp[i-1][j],dp[i][j-1])
    return dp[-1][-1]
print(lcs('abcde','ace'), lcs('abc','def'))
```

**Answer:** 3 0

2-D DP over prefixes. Verified by running it.

**Q82.** What does this print (0/1 knapsack)?

```python
def knap(w,v,C):
    dp=[0]*(C+1)
    for wi,vi in zip(w,v):
        for c in range(C,wi-1,-1): dp[c]=max(dp[c],dp[c-wi]+vi)
    return dp[C]
print(knap([1,3,4,5],[1,4,5,7],7))
```

**Answer:** 9

Iterate capacity downward to use each item once. Verified by running it.

**Q83.** Why iterate capacity in decreasing order for 0/1 knapsack?

**Answer:** So each item is used at most once

Increasing order allows reuse (unbounded).

**Q84.** How do you reduce a 2-D DP table's space?

**Answer:** Keep only the previous row when transitions need only it

O(m) instead of O(mn).

### [02 · Shortest Paths: Dijkstra &amp; Bellman-Ford](level-3/02-shortest-paths.md)

**Q85.** What does this print (Dijkstra)?

```python
import heapq
g={'A':[('B',1),('C',4)],'B':[('C',2),('D',5)],'C':[('D',1)],'D':[]}
d={k:float('inf') for k in g}; d['A']=0; h=[(0,'A')]
while h:
    du,u=heapq.heappop(h)
    if du>d[u]: continue
    for v,w in g[u]:
        if du+w<d[v]: d[v]=du+w; heapq.heappush(h,(d[v],v))
print(d)
```

**Answer:** {'A': 0, 'B': 1, 'C': 3, 'D': 4}

Greedy settling with a min-heap. Verified by running it.

**Q86.** When does Dijkstra fail?

**Answer:** With negative edge weights (use Bellman-Ford)

Needs non-negative weights.

**Q87.** What's Dijkstra's complexity with a heap?

**Answer:** O((V+E) log V)

Dense graphs can use arrays.

**Q88.** Which finds shortest paths with negative edges?

**Answer:** Bellman-Ford

O(VE).

### [03 · Topological Sort](level-3/03-topological-sort.md)

**Q89.** What does this print (topological order, Kahn)?

```python
from collections import deque
g={'a':['b','c'],'b':['d'],'c':['d'],'d':[]}
indeg={k:0 for k in g}
for u in g:
    for v in g[u]: indeg[v]+=1
q=deque(k for k in g if indeg[k]==0); out=[]
while q:
    u=q.popleft(); out.append(u)
    for v in g[u]:
        indeg[v]-=1
        if indeg[v]==0: q.append(v)
print(out, len(out)==len(g))
```

**Answer:** ['a', 'b', 'c', 'd'] True

If not all nodes are output, there's a cycle. Verified by running it.

**Q90.** What does topological sort apply to?

**Answer:** Directed acyclic graphs such as dependency ordering

Course schedule problems.

**Q91.** How do you detect a cycle with Kahn's algorithm?

**Answer:** If the output has fewer nodes than the graph, a cycle exists

Leftover in-degree &gt; 0.

**Q92.** What's the complexity?

**Answer:** O(V+E)

Each node and edge once.

### [04 · Union-Find (Disjoint Set Union)](level-3/04-union-find.md)

**Q93.** What does this print (union-find)?

```python
p=list(range(6))
def find(x):
    while p[x]!=x: p[x]=p[p[x]]; x=p[x]
    return x
def union(a,b):
    ra,rb=find(a),find(b)
    if ra==rb: return False
    p[ra]=rb; return True
print(union(0,1), union(1,2), union(0,2), len({find(i) for i in range(6)}))
```

**Answer:** True True False 4

The third union is redundant: same component. Verified by running it.

**Q94.** What do path compression and union by rank give?

**Answer:** Nearly O(1) amortized operations (inverse Ackermann)

Practical speed.

**Q95.** What are union-find use cases?

**Answer:** Connected components, cycle detection in undirected graphs and Kruskal's MST

Dynamic connectivity.

**Q96.** What does find() return?

**Answer:** The representative (root) of the element's set

Compare roots.

### [05 · Monotonic Stack &amp; Queue](level-3/05-monotonic-stack-queue.md)

**Q97.** What does this print (next greater element)?

```python
a=[2,1,2,4,3]
res=[-1]*len(a); st=[]
for i,x in enumerate(a):
    while st and a[st[-1]]<x: res[st.pop()]=x
    st.append(i)
print(res)
```

**Answer:** [4, 2, 4, -1, -1]

A monotonic decreasing stack of indices. Verified by running it.

**Q98.** What does this print (sliding window max)?

```python
from collections import deque
a=[1,3,-1,-3,5,3,6,7]; k=3
dq=deque(); out=[]
for i,x in enumerate(a):
    while dq and a[dq[-1]]<=x: dq.pop()
    dq.append(i)
    if dq[0]<=i-k: dq.popleft()
    if i>=k-1: out.append(a[dq[0]])
print(out)
```

**Answer:** [3, 3, 5, 5, 6, 7]

A monotonic deque keeps candidates in order. Verified by running it.

**Q99.** Why is monotonic stack O(n)?

**Answer:** Each element is pushed and popped at most once

Amortized.

**Q100.** What does a monotonic stack solve?

**Answer:** Next/previous greater or smaller element problems

Also histogram areas.

### [06 · Bit Manipulation](level-3/06-bit-manipulation.md)

**Q101.** What does this print?

```python
x=0b101101
print(x&-x, bin(x&(x-1)), bin(x).count('1'), 7^7, 1<<4, 20>>2)
```

**Answer:** 1 0b101100 4 0 16 5

x&amp;-x isolates the lowest set bit; x&amp;(x-1) clears it. Verified by running it.

**Q102.** What does this print (single number via XOR)?

```python
a=[4,1,2,1,2]
r=0
for x in a: r^=x
print(r)
```

**Answer:** 4

Pairs cancel under XOR. Verified by running it.

**Q103.** What does this print (power of two)?

```python
print([n for n in range(1,20) if n&(n-1)==0])
```

**Answer:** [1, 2, 4, 8, 16]

Exactly one bit set. Verified by running it.

**Q104.** What does `x & (x-1)` do?

**Answer:** Clears the lowest set bit

Counts bits in a loop.

### [07 · Segment Trees &amp; Fenwick Trees](level-3/07-segment-fenwick-trees.md)

**Q105.** What does this print (Fenwick tree)?

```python
n=8; t=[0]*(n+1)
def upd(i,v):
    while i<=n: t[i]+=v; i+=i&-i
def qry(i):
    s=0
    while i>0: s+=t[i]; i-=i&-i
    return s
for i,v in enumerate([3,2,-1,6,5,4,-3,3],1): upd(i,v)
print(qry(4), qry(8)-qry(2))
```

**Answer:** 10 14

Prefix sums with point updates in O(log n). Verified by running it.

**Q106.** What do segment trees support?

**Answer:** Range queries and point (or range) updates in O(log n)

Sum, min, max.

**Q107.** When is a prefix-sum array not enough?

**Answer:** When values change between queries

Use Fenwick or segment tree.

**Q108.** What's a Fenwick tree's advantage over a segment tree?

**Answer:** Smaller and simpler code for prefix-sum style queries

Limited operations.

### [08 · String Algorithms](level-3/08-string-algorithms.md)

**Q109.** What does this print (KMP failure function)?

```python
def fail(p):
    f=[0]*len(p); k=0
    for i in range(1,len(p)):
        while k and p[i]!=p[k]: k=f[k-1]
        if p[i]==p[k]: k+=1
        f[i]=k
    return f
print(fail('ababaca'))
```

**Answer:** [0, 0, 1, 2, 3, 0, 1]

Longest proper prefix that's also a suffix. Verified by running it.

**Q110.** What does this print (rolling hash)?

```python
s='abcd'
M=1_000_007; B=31
h=0
for ch in s: h=(h*B+ord(ch))%M
print(h)
```

**Answer:** 987060

Polynomial rolling hash. Verified by running it.

**Q111.** What's the complexity of naive substring search?

**Answer:** O(n*m)

KMP/Z achieve O(n+m).

**Q112.** Why use a rolling hash?

**Answer:** Update the hash of a sliding substring in O(1) (Rabin-Karp)

Verify on match to avoid collisions.

### [09 · Minimum Spanning Trees](level-3/09-minimum-spanning-trees.md)

**Q113.** What does this print (Kruskal MST weight)?

```python
edges=[(1,0,1),(3,1,2),(4,0,2),(2,2,3),(5,1,3)]
p=list(range(4))
def find(x):
    while p[x]!=x: p[x]=p[p[x]]; x=p[x]
    return x
w=0
for c,a,b in sorted(edges):
    ra,rb=find(a),find(b)
    if ra!=rb: p[ra]=rb; w+=c
print(w)
```

**Answer:** 6

Pick cheapest edges that don't form cycles. Verified by running it.

**Q114.** What does Prim's algorithm grow?

**Answer:** A tree from a start vertex, adding the cheapest crossing edge each time

Uses a heap.

**Q115.** How many edges does an MST of V vertices have?

**Answer:** V-1

For connected graphs.

**Q116.** Which structure does Kruskal use to avoid cycles?

**Answer:** Union-find

Sort edges first.

### [10 · Project — Hard Problem Set](level-3/10-project-hard-problem-set.md)

**Q117.** What's the purpose of the hard problem set?

**Answer:** Combine multiple patterns under realistic difficulty

Stretch.

**Q118.** What to do when stuck?

**Answer:** Restate, try small examples and a brute force, then look for a pattern

Show process.

**Q119.** How do you manage time on hard problems?

**Answer:** Get a correct solution first, then optimize

Correctness first.

**Q120.** What do you review after?

**Answer:** Pattern, complexity and mistakes

Reflect.

## Level 4 · Master

### [01 · The Interview Communication Framework](level-4/01-interview-framework.md)

**Q121.** What's a good interview framework order?

**Answer:** Clarify, examples, approach and complexity, code, test, optimize

Communicate throughout.

**Q122.** Why think aloud?

**Answer:** The interviewer evaluates your reasoning, not just the answer

Be collaborative.

**Q123.** What should you do with edge cases?

**Answer:** List and test them (empty, one element, duplicates, large)

Catches bugs.

**Q124.** When should you ask clarifying questions?

**Answer:** Before designing: constraints, input ranges and expected output

Avoid wrong assumptions.

### [02 · Behavioral Interview Prep](level-4/02-behavioral-prep.md)

**Q125.** What does the STAR method structure?

**Answer:** Situation, Task, Action, Result

Concise stories.

**Q126.** What do interviewers look for in behavioral answers?

**Answer:** Ownership, collaboration, learning from failure and impact

Be specific.

**Q127.** How do you answer a question about a failure?

**Answer:** Own it, explain what you learned and what you changed

Show growth.

**Q128.** How should you prepare stories?

**Answer:** Have several reusable examples with measurable results

Map to competencies.

### [03 · Mock Interview Walkthroughs](level-4/03-mock-walkthroughs.md)

**Q129.** What makes a mock interview useful?

**Answer:** Realistic conditions and honest feedback

Simulate pressure.

**Q130.** What should you review after a mock?

**Answer:** Where you were stuck, communication and missed edge cases

Targeted practice.

**Q131.** How do you recover from a wrong approach mid-interview?

**Answer:** Say so, return to examples and pivot calmly

Resilience is evaluated.

**Q132.** What's a good mock cadence?

**Answer:** Regular practice with varied problem types

Consistency.

### [04 · Online Assessment Tactics](level-4/04-online-assessments.md)

**Q133.** What differs about online assessments?

**Answer:** Strict timers, auto-graded tests and hidden cases

Optimize for correctness and speed.

**Q134.** What's a good OA strategy?

**Answer:** Read all problems, solve the easiest first and handle edge cases

Manage time.

**Q135.** Why test your own edge cases in an OA?

**Answer:** Hidden tests include extremes

Large inputs and empties.

**Q136.** How do you avoid time-limit failures?

**Answer:** Check complexity against constraints before coding

n up to 1e5 means O(n log n).

### [05 · Problem Sets by Interview Style](level-4/05-problem-sets-by-style.md)

**Q137.** Why practice problems grouped by style?

**Answer:** To build pattern recognition within a family

Then mix.

**Q138.** What's a good problem-set progression?

**Answer:** Easy to medium to hard within each pattern, then mixed review

Build confidence.

**Q139.** What should you track?

**Answer:** Patterns you missed and time to solve

Data-driven study.

**Q140.** What do you do with a solved problem?

**Answer:** Revisit it later to check retention

Spaced repetition.

### [06 · System-Design-Lite in Coding Rounds](level-4/06-system-design-lite.md)

**Q141.** What's system design 'lite' in interviews?

**Answer:** A short high-level design discussion of components, data and trade-offs

Clarify requirements first.

**Q142.** What should you clarify first?

**Answer:** Functional requirements, scale and constraints

Drive the design.

**Q143.** What does estimating scale help with?

**Answer:** Choosing storage, caching and partitioning approaches

Back-of-envelope.

**Q144.** How do you handle trade-offs?

**Answer:** State options and pick one with reasons

Reasoned choices.

### [07 · Testing &amp; Debugging Live](level-4/07-testing-debugging-live.md)

**Q145.** What's a useful debugging approach live?

**Answer:** Trace with a small example and narrate your reasoning

Systematic.

**Q146.** What do you do when tests fail?

**Answer:** Reproduce with the failing input and isolate the bug

Calm debugging.

**Q147.** Why write tests for your solution?

**Answer:** To show rigor and catch mistakes

Dry-run by hand.

**Q148.** Where do bugs often hide in DSA code?

**Answer:** Off-by-one, empty inputs and boundary conditions

Check edges.

### [08 · Offers &amp; Negotiation Basics](level-4/08-offers-negotiation.md)

**Q149.** What should you do before negotiating an offer?

**Answer:** Know your priorities, market data and alternatives

Prepare.

**Q150.** How do you negotiate respectfully?

**Answer:** Express enthusiasm, ask about flexibility and present reasons

Collaborative.

**Q151.** What's included in total compensation?

**Answer:** Salary, bonus, equity, benefits and growth

Compare holistically.

**Q152.** When do you decide?

**Answer:** After getting details in writing and weighing options

Take time.

### [09 · Study Plans: 4, 8 and 12 Weeks](level-4/09-study-plans.md)

**Q153.** What makes a study plan effective?

**Answer:** Consistent practice, spaced review and tracking weaknesses

Plan backward from the date.

**Q154.** How should you split your time?

**Answer:** Learn patterns, practice problems, mock interviews and review

Balanced.

**Q155.** How many problems do you need?

**Answer:** Quality over quantity: master patterns rather than counting

Understand why.

**Q156.** How do you avoid burnout?

**Answer:** Sustainable pace with rest and variety

Marathon.

### [10 · Capstone — Full Mock Interview](level-4/10-capstone-mock-interview.md)

**Q157.** What does the capstone mock interview combine?

**Answer:** A full simulated interview: problem solving, communication and review

Everything together.

**Q158.** How do you evaluate your performance?

**Answer:** Against a rubric: correctness, complexity, communication and testing

Honest assessment.

**Q159.** What's the next step after the capstone?

**Answer:** Targeted practice on gaps, then real interviews

Iterate.

**Q160.** What mindset helps?

**Answer:** Curious, collaborative and calm under pressure

Demonstrate how you think.

## Mixed questions across all levels

**Q161.** What does this print?

```python
def f(n):
    c=0
    for i in range(1,n):
        j=1
        while j<n: j*=2; c+=1
    return c
print(f(16))
```

**Answer:** 60

An outer loop of n-1 iterations times log n inner steps: O(n log n). Verified by running it.

**Q162.** What does this print?

```python
def two_sum(a,t):
    seen={}
    for i,x in enumerate(a):
        if t-x in seen: return [seen[t-x], i]
        seen[x]=i
print(two_sum([3,2,4],6))
```

**Answer:** [1, 2]

A hash map of complements gives O(n). Verified by running it.

**Q163.** What does this print?

```python
def maxsub(a,k):
    w=sum(a[:k]); b=w
    for i in range(k,len(a)): w+=a[i]-a[i-k]; b=max(b,w)
    return b
print(maxsub([1,4,2,10,2,3,1,0,20],4))
```

**Answer:** 24

Fixed-size sliding window of 4. Verified by running it.

**Q164.** Given a sorted array, which technique finds a pair with a given sum in O(n) time and O(1) space?

**Answer:** Two pointers from both ends

Move left or right based on the sum.

**Q165.** What does this print?

```python
from collections import deque
q=deque()
for x in [1,2,3]: q.append(x)
q.appendleft(0)
print(q.pop(), q.popleft(), list(q))
```

**Answer:** 3 0 [1, 2]

deque works at both ends in O(1). Verified by running it.

**Q166.** What does this print (reverse linked list)?

```python
class N:
    def __init__(s,v,n=None): s.v=v; s.next=n
h=N(1,N(2,N(3,N(4))))
p=None
while h:
    nxt=h.next; h.next=p; p=h; h=nxt
out=[]
while p: out.append(p.v); p=p.next
print(out)
```

**Answer:** [4, 3, 2, 1]

Save next before re-pointing. Verified by running it.

**Q167.** What does this print?

```python
import heapq
h=[]
for x in [7,2,9,4,1]:
    heapq.heappush(h,x)
    if len(h)>2: heapq.heappop(h)
print(sorted(h))
```

**Answer:** [7, 9]

A size-2 min-heap retains the two largest values. Verified by running it.

**Q168.** What does this print (BFS shortest path length in a grid)?

```python
from collections import deque
g=['S.#','..#','.E.']
q=deque([(0,0,0)]); seen={(0,0)}; ans=-1
while q:
    r,c,d=q.popleft()
    if g[r][c]=='E': ans=d; break
    for dr,dc in((1,0),(-1,0),(0,1),(0,-1)):
        nr,nc=r+dr,c+dc
        if 0<=nr<3 and 0<=nc<3 and g[nr][nc]!='#' and (nr,nc) not in seen:
            seen.add((nr,nc)); q.append((nr,nc,d+1))
print(ans)
```

**Answer:** 3

BFS finds shortest paths in unweighted graphs. Verified by running it.

**Q169.** What does this print (coin change min coins)?

```python
def coins(cs,t):
    dp=[0]+[10**9]*t
    for a in range(1,t+1):
        for c in cs:
            if c<=a: dp[a]=min(dp[a],dp[a-c]+1)
    return dp[t]
print(coins([1,3,4],6))
```

**Answer:** 2

Greedy (4+1+1) gives 3, DP finds 3+3 = 2. Verified by running it.

**Q170.** What does this print (merge intervals)?

```python
iv=sorted([[1,4],[4,5],[7,9],[8,10]])
out=[iv[0]]
for s,e in iv[1:]:
    if s<=out[-1][1]: out[-1][1]=max(out[-1][1],e)
    else: out.append([s,e])
print(out)
```

**Answer:** [[1, 5], [7, 10]]

Touching intervals merge when s &lt;= previous end. Verified by running it.

**Q171.** What does this print (Dijkstra distances)?

```python
import heapq
g={'A':[('B',2),('C',5)],'B':[('C',1),('D',7)],'C':[('D',2)],'D':[]}
d={k:99 for k in g}; d['A']=0; h=[(0,'A')]
while h:
    du,u=heapq.heappop(h)
    if du>d[u]: continue
    for v,w in g[u]:
        if du+w<d[v]: d[v]=du+w; heapq.heappush(h,(d[v],v))
print(d['D'])
```

**Answer:** 5

A-&gt;B-&gt;C-&gt;D = 2+1+2. Verified by running it.

**Q172.** What does this print (union-find components)?

```python
p=list(range(7))
def find(x):
    while p[x]!=x: p[x]=p[p[x]]; x=p[x]
    return x
for a,b in [(0,1),(2,3),(1,3),(4,5)]: p[find(a)]=find(b)
print(len({find(i) for i in range(7)}))
```

**Answer:** 3

Components: {0,1,2,3}, {4,5}, {6}. Verified by running it.

**Q173.** What does this print (single number)?

```python
r=0
for x in [7,3,5,3,7]: r^=x
print(r)
```

**Answer:** 5

XOR cancels duplicate pairs. Verified by running it.

**Q174.** A problem asks for the number of ways to climb stairs taking 1 or 2 steps. Which approach fits?

**Answer:** Dynamic programming (Fibonacci recurrence)

Overlapping subproblems.

**Q175.** You need all permutations of a list. Which technique fits?

**Answer:** Backtracking: choose, explore, unchoose

Generate candidates and undo.

**Q176.** You need autocomplete suggestions for a prefix over 1M words. Which structure?

**Answer:** A trie

Prefix queries in O(L).

**Q177.** Edges have negative weights. Which shortest path algorithm do you use?

**Answer:** Bellman-Ford

Dijkstra assumes non-negative weights.

**Q178.** In an interview you notice your first idea is O(n^2) with n = 10^5. What do you do?

**Answer:** Say so and look for a better pattern (hash map, two pointers, sorting, heap) before coding

Check constraints against complexity.

## Practical coding questions

### Top K and longest unique substring

Implement top_k_frequent(words, k) with a heap of (-count, word) so ties are alphabetical, and longest_unique(s) with a sliding window and a last-seen map.

Starter code:

```
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

Expected output:

```text
['is', 'the', 'sunny']
['a', 'b']
3 3 0
```

??? success "Show a solution"

    ```
    import heapq
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
            if ch in last and last[ch] >= left:
                left = last[ch] + 1
            last[ch] = right
            best = max(best, right - left + 1)
        return best

    print(top_k_frequent("the day is sunny the the sunny is is".split(), 3))
    print(top_k_frequent(["b", "a", "b", "a", "c"], 2))
    print(longest_unique("abcabcbb"), longest_unique("pwwkew"), longest_unique(""))
    ```

    The heap orders by count descending then word ascending; the sliding window moves `left` past the previous occurrence of a repeated character, so each element is processed once: O(n). Verified by running the solution.

### Course schedule and island count

Implement can_finish(n, prereqs) with Kahn's algorithm (return whether all courses can be taken and the order) and count_islands(grid) with an iterative DFS.

Starter code:

```
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

Expected output:

```text
(True, [0, 1, 2, 3])
(False, [])
3
```

??? success "Show a solution"

    ```
    from collections import deque

    def can_finish(n, prereqs):
        """Course schedule: topological sort with cycle detection (Kahn)."""
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
                if (r, c) in seen or not (0 <= r < rows and 0 <= c < cols) or grid[r][c] != "1":
                    continue
                seen.add((r, c))
                stack += [(r + 1, c), (r - 1, c), (r, c + 1), (r, c - 1)]
        n = 0
        for r in range(rows):
            for c in range(cols):
                if grid[r][c] == "1" and (r, c) not in seen:
                    n += 1
                    dfs(r, c)
        return n

    print(can_finish(4, [(1, 0), (2, 1), (3, 2)]))
    print(can_finish(2, [(1, 0), (0, 1)]))
    print(count_islands(["11000", "11000", "00100", "00011"]))
    ```

    If Kahn's algorithm outputs fewer nodes than exist, the remaining nodes form a cycle. Counting islands launches one flood fill per unvisited land cell. Verified by running the solution.

---

More free practice: the [DSA & Coding Interviews Mastery Path](index.md) curriculum, the [Interview Prep Pack](https://learnmasterypath.com/interview-pack/) and the [free resume ATS check](https://learnmasterypath.com/resume/).
