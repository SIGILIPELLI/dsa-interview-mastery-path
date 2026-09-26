---
description: "Manipulate singly linked lists in Python: dummy heads, in-place reversal, merging, Floyd's cycle detection, and removing the nth node from the end."
---

# 07 · Linked Lists

Linked lists rarely appear in production Python, but they are an interview staple
because they test careful pointer manipulation: can you rewire references without losing
part of the structure or creating a cycle?

A singly linked list is a chain of nodes, each holding a value and a reference to the
next node. The last node points to `None`.

```python
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next


def from_list(values):
    dummy = ListNode()
    tail = dummy
    for v in values:
        tail.next = ListNode(v)
        tail = tail.next
    return dummy.next


def to_list(head):
    out = []
    while head:
        out.append(head.val)
        head = head.next
    return out


assert to_list(from_list([1, 2, 3])) == [1, 2, 3]
assert from_list([]) is None
```

The two helpers are what you would write on the side in a real interview to test your
code. Notice `from_list` already uses the most important linked-list trick: the
**dummy head**.

## Technique 1: the dummy (sentinel) node

Many operations need special handling when the head itself changes (deleting the first
node, inserting before it). A dummy node placed before the head removes the special case:
every real node now has a predecessor.

**Problem.** Remove all nodes with value `val`.

```python
def remove_elements(head, val):
    dummy = ListNode(0, head)
    prev = dummy
    while prev.next:
        if prev.next.val == val:
            prev.next = prev.next.next      # unlink; do not advance prev
        else:
            prev = prev.next
    return dummy.next


assert to_list(remove_elements(from_list([1, 2, 6, 3, 6]), 6)) == [1, 2, 3]
assert to_list(remove_elements(from_list([7, 7, 7]), 7)) == []     # edge: all removed
assert to_list(remove_elements(from_list([]), 1)) == []            # edge: empty
```

## Technique 2: in-place reversal

**Problem.** Reverse a linked list.

**Approach.** Walk the list with `prev` and `curr`. At each node, remember `next`, point
`curr.next` back to `prev`, then advance both.

```python
def reverse_list(head):
    prev, curr = None, head
    while curr:
        nxt = curr.next       # 1. save the rest of the list
        curr.next = prev      # 2. reverse this link
        prev = curr           # 3. advance prev
        curr = nxt            # 4. advance curr
    return prev


assert to_list(reverse_list(from_list([1, 2, 3, 4]))) == [4, 3, 2, 1]
assert to_list(reverse_list(from_list([1]))) == [1]
assert reverse_list(None) is None
```

O(n) time, O(1) space. Draw three boxes and arrows the first few times; the order of the
four assignments matters, and step 1 must come first or you lose the rest of the list.

## Technique 3: merging

**Problem.** Merge two sorted lists into one sorted list by splicing nodes.

```python
def merge_two_lists(a, b):
    dummy = ListNode()
    tail = dummy
    while a and b:
        if a.val <= b.val:
            tail.next, a = a, a.next
        else:
            tail.next, b = b, b.next
        tail = tail.next
    tail.next = a or b            # attach whatever remains
    return dummy.next


assert to_list(merge_two_lists(from_list([1, 2, 4]), from_list([1, 3, 4]))) == [1, 1, 2, 3, 4, 4]
assert to_list(merge_two_lists(None, from_list([0]))) == [0]
assert merge_two_lists(None, None) is None
```

Using `<=` keeps the merge **stable** (equal values from `a` come first), which matters
when merge sort is built on top of this.

## Technique 4: fast and slow pointers

A slow pointer moves one step, a fast pointer two. This finds the middle in one pass and
detects cycles.

```python
def middle_node(head):
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
    return slow                       # second middle for even length


def has_cycle(head):
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow is fast:
            return True
    return False


assert middle_node(from_list([1, 2, 3, 4, 5])).val == 3
assert middle_node(from_list([1, 2, 3, 4])).val == 3
loop = from_list([3, 2, 0, -4])
loop.next.next.next.next = loop.next          # -4 -> 2
assert has_cycle(loop)
assert not has_cycle(from_list([1, 2]))
assert not has_cycle(None)
```

## Worked problem: remove the nth node from the end

**Problem.** Remove the n-th node from the end in one pass (1 ≤ n ≤ length).

**Approach.** Move a `lead` pointer n+1 steps ahead of `trail` (both starting at a
dummy). Then move both until `lead` falls off the end. `trail` is now just before the
node to delete.

```python
def remove_nth_from_end(head, n):
    dummy = ListNode(0, head)
    lead = trail = dummy
    for _ in range(n + 1):
        lead = lead.next
    while lead:
        lead = lead.next
        trail = trail.next
    trail.next = trail.next.next
    return dummy.next


assert to_list(remove_nth_from_end(from_list([1, 2, 3, 4, 5]), 2)) == [1, 2, 3, 5]
assert to_list(remove_nth_from_end(from_list([1]), 1)) == []           # edge: only node
assert to_list(remove_nth_from_end(from_list([1, 2]), 2)) == [2]       # edge: remove head
```

The dummy is what makes "remove the head" work without a special case.

## How It Actually Works

**Memory layout.** Each `ListNode` is a separate Python object somewhere on the heap; a
node's `next` attribute holds a reference (a pointer) to another object. Unlike a list,
there is no contiguous block, so reaching the k-th node means following k references:
O(k). In exchange, inserting or deleting *at a node you already hold* is O(1) — just
rewire one or two references, with no shifting.

**Why Floyd's cycle detection works.** If there is no cycle, `fast` reaches `None`. If
there is a cycle, both pointers eventually enter it. Once both are inside, look at the
distance from `fast` to `slow` along the cycle direction: each step, `fast` gains exactly
one node on `slow` (it moves 2, `slow` moves 1), so the gap shrinks by one each step and
must hit 0 — they cannot jump over each other. The meeting happens within one lap of
`slow` after it enters the cycle, so the algorithm is O(n) time and O(1) space.

A further fact lets you find the *start* of the cycle: after they meet, reset one pointer
to the head and move both one step at a time; they meet again exactly at the cycle's
entry. (If the tail before the cycle has length `a` and they meet `b` steps into a cycle
of length `c`, then `2(a + b) = a + b + kc`, so `a = kc − b`: walking `a` steps from the
meeting point lands on the entry.)

**Garbage collection.** When you unlink a node (`prev.next = prev.next.next`), nothing
references it any more, so CPython's reference counting frees it immediately. In C or C++
you would have to free it yourself — interviewers in those languages may ask about it.

## Common mistakes

- Losing the rest of the list by overwriting `curr.next` before saving it.
- Advancing `prev` after a deletion, which skips checking the next node.
- Dereferencing `None`: in loops, check `fast and fast.next` before `fast.next.next`.
- Forgetting to return `dummy.next` (returning `head`, which may have been deleted).
- Comparing nodes with `==` when you mean identity; use `is`.

## Variations to practice

- Reverse a sublist between positions `left` and `right`.
- Palindrome linked list (find middle, reverse second half, compare).
- Reorder list L0→Ln→L1→Ln-1… (middle + reverse + merge).
- Add two numbers stored as digit lists.
- Merge k sorted lists with a heap (Level 2, lesson 3).
- LRU cache (hash map + doubly linked list).

## Exercise

Write `is_palindrome(head)` in O(n) time and O(1) extra space: find the middle with fast
and slow pointers, reverse the second half, compare the halves, and (for good manners)
reverse the second half back. Test with `[1,2,2,1]`, `[1,2,3,2,1]`, `[1,2]`, `[1]`, and
the empty list.
