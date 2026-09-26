---
description: "Work with binary search trees: the ordering invariant, validating with bounds, search, insert, delete, kth smallest, and why balance decides speed."
---

# 02 · Binary Search Trees

A binary search tree (BST) is a binary tree with one extra rule, the **BST invariant**:
for every node, all values in its left subtree are smaller than the node's value, and
all values in its right subtree are larger. (Conventions for duplicates vary — ask.)

That invariant gives you two things for free: **search by walking one path** (like
binary search), and **sorted order via inorder traversal**.

```python
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val, self.left, self.right = val, left, right


def insert(root, val):
    if not root:
        return TreeNode(val)
    if val < root.val:
        root.left = insert(root.left, val)
    elif val > root.val:
        root.right = insert(root.right, val)
    return root                                    # duplicates ignored


def from_values(values):
    root = None
    for v in values:
        root = insert(root, v)
    return root


def inorder(root, out=None):
    out = [] if out is None else out
    if root:
        inorder(root.left, out)
        out.append(root.val)
        inorder(root.right, out)
    return out


root = from_values([5, 3, 8, 1, 4, 9])
assert inorder(root) == [1, 3, 4, 5, 8, 9]
```

## Search

```python
def search(root, target):
    node = root
    while node and node.val != target:
        node = node.left if target < node.val else node.right
    return node


assert search(root, 4).val == 4
assert search(root, 7) is None
assert search(None, 1) is None
```

O(h) time, O(1) space iteratively.

## Worked problem 1: validate a BST

**Problem.** Decide whether a binary tree satisfies the BST invariant (strictly
increasing inorder).

**The trap.** Checking only `left.val < node.val < right.val` at each node is **wrong**.
In the tree below, every parent-child pair looks fine, but 6 sits in the left subtree of
5 even though it is larger:

```text
      5
     / \
    3   8
     \
      6      <- violates: must be < 5
```

**Approach.** Pass down the valid range `(low, high)` for each subtree. Going left
tightens the upper bound to the parent's value; going right tightens the lower bound.

```python
def is_valid_bst(root, low=float("-inf"), high=float("inf")):
    if not root:
        return True
    if not (low < root.val < high):
        return False
    return (is_valid_bst(root.left, low, root.val) and
            is_valid_bst(root.right, root.val, high))


bad = TreeNode(5, TreeNode(3, None, TreeNode(6)), TreeNode(8))
assert not is_valid_bst(bad)
assert is_valid_bst(from_values([5, 3, 8, 1, 4, 9]))
assert is_valid_bst(None)                          # edge: empty tree is valid
assert not is_valid_bst(TreeNode(2, TreeNode(2)))  # edge: duplicate (strict rule)
```

An equivalent approach: do an inorder traversal and check each value is greater than the
previous one. **Complexity:** O(n) time, O(h) space.

## Worked problem 2: kth smallest element

**Approach.** Inorder visits values in sorted order, so stop at the k-th visit. The
iterative traversal lets you stop early without visiting the whole tree.

```python
def kth_smallest(root, k):
    stack, node = [], root
    while stack or node:
        while node:
            stack.append(node)
            node = node.left
        node = stack.pop()
        k -= 1
        if k == 0:
            return node.val
        node = node.right
    raise ValueError("k is larger than the tree size")


r = from_values([5, 3, 6, 2, 4, 1])
assert kth_smallest(r, 1) == 1
assert kth_smallest(r, 3) == 3
assert kth_smallest(r, 6) == 6                     # edge: largest
```

**Complexity:** O(h + k) time. If the tree changes often and you need this repeatedly,
store subtree sizes in each node so you can pick left or right in O(h) — an *order
statistic tree*.

## Worked problem 3: delete a node

Deletion has three cases:

1. **Leaf** — remove it.
2. **One child** — replace the node with its child.
3. **Two children** — replace the node's value with its **inorder successor** (the
   smallest value in its right subtree), then delete that successor from the right
   subtree. The successor has no left child, so its deletion falls into case 1 or 2.

```python
def delete(root, key):
    if not root:
        return None
    if key < root.val:
        root.left = delete(root.left, key)
    elif key > root.val:
        root.right = delete(root.right, key)
    else:
        if not root.left:
            return root.right          # covers leaf and right-only
        if not root.right:
            return root.left
        succ = root.right
        while succ.left:
            succ = succ.left
        root.val = succ.val
        root.right = delete(root.right, succ.val)
    return root


r = from_values([5, 3, 6, 2, 4, 7])
r = delete(r, 3)                                   # two children
assert inorder(r) == [2, 4, 5, 6, 7] and is_valid_bst(r)
r = delete(r, 7)                                   # leaf
assert inorder(r) == [2, 4, 5, 6]
r = delete(r, 42)                                  # edge: key not present
assert inorder(r) == [2, 4, 5, 6]
assert delete(TreeNode(1), 1) is None              # edge: delete the only node
```

## Worked problem 4: lowest common ancestor in a BST

In a BST you do not need a full traversal: if both values are smaller than the current
node, the LCA is on the left; if both are larger, on the right; otherwise the current
node is where they split.

```python
def lca_bst(root, p, q):
    node = root
    while node:
        if p < node.val and q < node.val:
            node = node.left
        elif p > node.val and q > node.val:
            node = node.right
        else:
            return node.val
    return None


r = from_values([6, 2, 8, 0, 4, 7, 9, 3, 5])
assert lca_bst(r, 2, 8) == 6
assert lca_bst(r, 2, 4) == 2          # edge: one is the ancestor of the other
assert lca_bst(r, 3, 5) == 4
```

O(h) time, O(1) space.

## How It Actually Works

**Everything is O(h), and h depends on insertion order.** Each comparison moves one
level down, so search, insert and delete cost O(h). Insert random values and the
expected height is O(log n). Insert *sorted* values — `1, 2, 3, ..., n` — and every new
value goes to the right of the last one: the tree becomes a linked list with height n,
and every operation is O(n).

**Self-balancing trees** fix this by restructuring after updates. An AVL tree keeps the
heights of every node's two subtrees within 1 of each other; a red-black tree enforces
colouring rules guaranteeing that no root-to-leaf path is more than twice as long as any
other. Both restore balance with **rotations**: a local rewiring of three nodes that
changes the shape while preserving inorder order. With height O(log n) guaranteed, all
operations become worst-case O(log n). Java's `TreeMap` and C++'s `std::map` are
red-black trees.

**Python has no built-in balanced BST.** In interviews, options are: a sorted list with
`bisect` (O(log n) search, but O(n) insert because of shifting — often fine for
moderate n), a heap if you only need the minimum, or the third-party
`sortedcontainers.SortedList` (not available in most interview environments, so do not
depend on it without asking). Knowing *why* you would want a balanced BST, and which
trade-off you are making without one, is what interviewers look for.

## Common mistakes

- Validating only parent-child pairs instead of full ranges.
- Using `0` or a fixed number as the initial bounds instead of infinities (fails for
  negative values or large values).
- Forgetting to reassign the returned subtree (`root.left = delete(...)`) — the deletion
  silently does nothing.
- Stating O(log n) without noting the tree must be balanced.
- Ignoring duplicate-handling conventions.

## Variations to practice

- Convert a sorted array to a height-balanced BST (pick the middle as root).
- Inorder successor of a node.
- Two Sum in a BST (inorder to a list, or a hash set during traversal).
- Range sum of BST (prune subtrees outside the range).
- Recover a BST where two nodes were swapped.

## Exercise

Write `sorted_array_to_bst(nums)` that builds a height-balanced BST from a sorted list
by recursively choosing the middle element as the root (pass indices, not slices).
Verify with `inorder`, `is_valid_bst`, and a height check that the height is at most
`ceil(log2(n + 1))` for n = 1, 2, 7, and 100. Explain why picking the middle guarantees
balance.
