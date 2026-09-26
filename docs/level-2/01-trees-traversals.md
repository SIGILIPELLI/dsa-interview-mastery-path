---
description: "Traverse binary trees recursively and iteratively, do level-order BFS, and solve depth, diameter and path-sum problems by returning values up the tree."
---

# 01 · Binary Trees & Traversals

A binary tree is a node with a value and up to two children, each of which is itself a
binary tree (or `None`). That recursive definition is the key to nearly every tree
problem: solve it for the children, then combine at the root.

```python
from collections import deque


class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right


def build(values):
    """Build a tree from level-order values, with None for missing nodes."""
    if not values or values[0] is None:
        return None
    root = TreeNode(values[0])
    queue, i = deque([root]), 1
    while queue and i < len(values):
        node = queue.popleft()
        for side in ("left", "right"):
            if i < len(values) and values[i] is not None:
                child = TreeNode(values[i])
                setattr(node, side, child)
                queue.append(child)
            i += 1
    return root


t = build([1, 2, 3, None, 4])
assert t.left.right.val == 4 and t.right.val == 3
```

## The three depth-first orders

For each node, "visit" means process its value. The orders differ in *when*:

- **Preorder** — node, left, right. Useful for copying/serializing a tree.
- **Inorder** — left, node, right. On a binary *search* tree it yields sorted order.
- **Postorder** — left, right, node. Use when a node needs its children's results first
  (heights, sizes, deleting a tree).

```python
def preorder(root):
    return [root.val] + preorder(root.left) + preorder(root.right) if root else []

def inorder(root):
    return inorder(root.left) + [root.val] + inorder(root.right) if root else []

def postorder(root):
    return postorder(root.left) + postorder(root.right) + [root.val] if root else []


t = build([1, 2, 3, 4, 5])
assert preorder(t) == [1, 2, 4, 5, 3]
assert inorder(t) == [4, 2, 5, 1, 3]
assert postorder(t) == [4, 5, 2, 3, 1]
assert inorder(None) == []
```

These one-liners are fine for explaining, but list concatenation copies at every level
(O(n²) worst case on a skewed tree). In an interview, collect into one shared list:

```python
def inorder_iterative(root):
    out, stack, node = [], [], root
    while stack or node:
        while node:                 # go as far left as possible
            stack.append(node)
            node = node.left
        node = stack.pop()          # leftmost unvisited node
        out.append(node.val)
        node = node.right           # then its right subtree
    return out


assert inorder_iterative(build([1, 2, 3, 4, 5])) == [4, 2, 5, 1, 3]
assert inorder_iterative(None) == []
```

The iterative version is also the answer when the interviewer says "the tree can be
100,000 nodes deep".

## Level-order traversal (BFS)

**Problem.** Return the values level by level: `[[3], [9, 20], [15, 7]]`.

```python
def level_order(root):
    if not root:
        return []
    result, queue = [], deque([root])
    while queue:
        level = []
        for _ in range(len(queue)):      # exactly the nodes of this level
            node = queue.popleft()
            level.append(node.val)
            if node.left:
                queue.append(node.left)
            if node.right:
                queue.append(node.right)
        result.append(level)
    return result


assert level_order(build([3, 9, 20, None, None, 15, 7])) == [[3], [9, 20], [15, 7]]
assert level_order(None) == []
assert level_order(build([1])) == [[1]]
```

Snapshotting `len(queue)` at the start of each iteration is the trick that separates
levels. The same loop gives the right-side view (last of each level), zigzag order, and
level averages.

## Returning values up the tree

Most tree problems are postorder in disguise: compute something for each subtree and
return it to the parent.

### Worked problem 1: maximum depth

```python
def max_depth(root):
    if not root:
        return 0
    return 1 + max(max_depth(root.left), max_depth(root.right))


assert max_depth(build([3, 9, 20, None, None, 15, 7])) == 3
assert max_depth(None) == 0
```

### Worked problem 2: diameter of a binary tree

**Problem.** The diameter is the number of **edges** on the longest path between any two
nodes. The path need not pass through the root.

**Approach.** The longest path through a given node goes down its left side and down its
right side: `height(left) + height(right)` edges. Compute heights bottom-up, and while
doing so, update a global best. Each call *returns* one thing (height) but *records*
another (diameter through this node).

```python
def diameter(root):
    best = 0

    def height(node):
        nonlocal best
        if not node:
            return 0
        lh, rh = height(node.left), height(node.right)
        best = max(best, lh + rh)
        return 1 + max(lh, rh)

    height(root)
    return best


assert diameter(build([1, 2, 3, 4, 5])) == 3          # 4 -> 2 -> 1 -> 3
assert diameter(build([1, 2])) == 1
assert diameter(None) == 0
# path that avoids the root:
skewed = build([1, 2, None, 3, 4, 5, None, None, 6])
assert diameter(skewed) == 4                          # 5 -> 3 -> 2 -> 4 -> 6
```

This "return one value, update another" shape also solves *maximum path sum*, *longest
univalue path*, and *binary tree cameras*.

### Worked problem 3: path sum (top-down)

Some problems pass information *down* instead:

```python
def has_path_sum(root, target):
    if not root:
        return False
    remaining = target - root.val
    if not root.left and not root.right:     # leaf
        return remaining == 0
    return has_path_sum(root.left, remaining) or has_path_sum(root.right, remaining)


t = build([5, 4, 8, 11, None, 13, 4, 7, 2, None, None, None, 1])
assert has_path_sum(t, 22)                   # 5 -> 4 -> 11 -> 2
assert not has_path_sum(t, 5)                # 5 alone is not a leaf path
assert not has_path_sum(None, 0)             # edge: empty tree has no paths
```

## How It Actually Works

**Every traversal visits each node once**, so all of them are O(n) time. The space
difference is where the interesting part is:

- DFS (recursive or with an explicit stack) holds one frame per node on the current
  root-to-node path: **O(h)** space, where `h` is the height. For a balanced tree
  `h ≈ log n`; for a degenerate "linked list" tree `h = n`.
- BFS holds one level at a time: **O(w)** space, where `w` is the maximum width. A
  complete tree's last level holds about n/2 nodes, so BFS can use O(n) where DFS uses
  O(log n). Conversely, on a long skinny tree BFS is cheap and DFS is expensive.

The iterative inorder loop is the recursive one with the call stack made explicit: the
inner `while node` is the chain of "recurse left" calls; `stack.pop()` is returning from
one; `node = node.right` is the next recursive call.

Recursion works so naturally on trees because the recursion tree **is** the data tree:
each call handles exactly one node, and the base case `node is None` handles every
missing child. That is also why you should almost always write the `None` base case
rather than checking `if node.left:` before each call — it removes a class of special
cases.

## Common mistakes

- Treating a node with one child as a leaf in path problems (a leaf has *no* children).
- Counting nodes instead of edges for diameter (or vice versa) — clarify with the
  interviewer.
- Computing height inside a loop over nodes (calling `max_depth` from every node) →
  O(n²). Return heights bottom-up in one pass.
- Forgetting `nonlocal` when updating an outer variable from a nested function.
- Assuming the tree is balanced when stating space complexity. Say O(h).

## Variations to practice

- Invert (mirror) a binary tree.
- Same tree / symmetric tree / subtree of another tree.
- Binary tree right side view; zigzag level order.
- Lowest common ancestor of a binary tree.
- Binary tree maximum path sum (values may be negative).
- Serialize and deserialize a binary tree.

## Exercise

Implement `lowest_common_ancestor(root, p, q)` for a general binary tree (not a BST),
where `p` and `q` are node objects guaranteed to be in the tree. Use one postorder pass:
return `p`/`q` if found, the node itself if both sides report a hit, otherwise whichever
side is non-empty. Test the case where one node is the ancestor of the other, and the
case where they are in different subtrees. State time and space in terms of `n` and `h`.
