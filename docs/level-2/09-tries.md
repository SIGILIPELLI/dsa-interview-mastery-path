---
description: "Build a trie (prefix tree) in Python for insert, search and prefix queries, then use it for autocomplete, word replacement and multi-word grid search."
---

# 09 · Tries

A **trie** (prefix tree, pronounced "try") stores a set of strings as a tree of
characters. Each edge is one character; each path from the root spells a prefix; a flag
marks nodes where a complete word ends. Words that share a prefix share the nodes for it.

```text
insert: "car", "cat", "cart", "dog"

        (root)
        /    \
       c      d
       |      |
       a      o
      / \     |
     r*  t*   g*
     |
     t*          * = end of a word
```

**Signals:** "starts with", "prefix", "autocomplete", "dictionary of words" searched many
times, or searching *many* words in a grid at once.

## Implementation

```python
class TrieNode:
    __slots__ = ("children", "is_word")

    def __init__(self):
        self.children = {}          # char -> TrieNode
        self.is_word = False


class Trie:
    def __init__(self):
        self.root = TrieNode()

    def insert(self, word):
        node = self.root
        for ch in word:
            node = node.children.setdefault(ch, TrieNode())
        node.is_word = True

    def _walk(self, s):
        node = self.root
        for ch in s:
            node = node.children.get(ch)
            if node is None:
                return None
        return node

    def search(self, word):
        node = self._walk(word)
        return node is not None and node.is_word

    def starts_with(self, prefix):
        return self._walk(prefix) is not None


t = Trie()
for w in ["car", "cat", "cart", "dog"]:
    t.insert(w)
assert t.search("car") and t.search("cart")
assert not t.search("ca")              # a prefix, not a word
assert t.starts_with("ca")
assert not t.starts_with("cow")
assert t.starts_with("")               # edge: empty prefix matches everything
t.insert("")
assert t.search("")                    # edge: empty word can be stored
```

Every operation is **O(L)** for a string of length L, independent of how many words are
stored.

## Worked problem 1: autocomplete

**Problem.** Return up to `k` stored words that start with `prefix`, in lexicographic
order.

**Approach.** Walk to the prefix node, then DFS below it visiting children in sorted
order, stopping after `k` results.

```python
def autocomplete(trie, prefix, k):
    start = trie._walk(prefix)
    results = []
    if start is None:
        return results

    def dfs(node, path):
        if len(results) == k:
            return
        if node.is_word:
            results.append(prefix + "".join(path))
        for ch in sorted(node.children):
            path.append(ch)
            dfs(node.children[ch], path)
            path.pop()

    dfs(start, [])
    return results


t = Trie()
for w in ["mobile", "mouse", "moneypot", "monitor", "mousepad", "apple"]:
    t.insert(w)
assert autocomplete(t, "mou", 5) == ["mouse", "mousepad"]
assert autocomplete(t, "mo", 3) == ["mobile", "moneypot", "monitor"]
assert autocomplete(t, "x", 3) == []              # edge: no match
assert autocomplete(t, "apple", 2) == ["apple"]   # prefix is itself a word
```

In a production autocomplete you would cache the top suggestions at each node so queries
avoid the DFS, trading memory for latency.

## Worked problem 2: replace words with their shortest root

**Problem.** Given a dictionary of roots and a sentence, replace each word with the
shortest root that is a prefix of it (`"cattle"` → `"cat"` if `"cat"` is a root).

```python
def replace_words(roots, sentence):
    trie = Trie()
    for r in roots:
        trie.insert(r)

    def shortest_root(word):
        node = trie.root
        for i, ch in enumerate(word):
            node = node.children.get(ch)
            if node is None:
                return word
            if node.is_word:
                return word[:i + 1]
        return word

    return " ".join(shortest_root(w) for w in sentence.split())


assert replace_words(["cat", "bat", "rat"], "the cattle was rattled by the battery") == \
    "the cat was rat by the bat"
assert replace_words(["a", "aa"], "aadsfasf absbs") == "a a"    # shortest wins
assert replace_words([], "hello world") == "hello world"         # edge: no roots
```

Checking every root against every word would cost O(roots × words × L). The trie makes it
O(total characters).

## Worked problem 3: word search II (many words in a grid)

**Problem.** Given a letter grid and a list of words, return all words that can be
formed by adjacent cells (each cell once per word).

Running Level 2 lesson 5's single-word search for each word repeats a lot of work. With a
trie, one backtracking pass from each cell explores **all words at once**, and abandons a
path the moment its prefix is not in the trie.

```python
def find_words(board, words):
    root = TrieNode()
    for w in words:
        node = root
        for ch in w:
            node = node.children.setdefault(ch, TrieNode())
        node.is_word = True

    rows, cols = len(board), len(board[0])
    found = set()

    def dfs(r, c, node, path):
        ch = board[r][c]
        nxt = node.children.get(ch)
        if nxt is None:
            return                                  # prefix not in any word: prune
        path.append(ch)
        if nxt.is_word:
            found.add("".join(path))
        board[r][c] = "#"
        for nr, nc in ((r + 1, c), (r - 1, c), (r, c + 1), (r, c - 1)):
            if 0 <= nr < rows and 0 <= nc < cols and board[nr][nc] != "#":
                dfs(nr, nc, nxt, path)
        board[r][c] = ch
        path.pop()

    for r in range(rows):
        for c in range(cols):
            dfs(r, c, root, [])
    return sorted(found)


board = [list("oaan"), list("etae"), list("ihkr"), list("iflv")]
assert find_words(board, ["oath", "pea", "eat", "rain"]) == ["eat", "oath"]
assert find_words([list("a")], ["a", "b"]) == ["a"]
assert find_words([list("ab")], ["aba"]) == []          # cannot reuse a cell
```

A further optimization used in practice: remove a word's end flag after finding it and
prune childless nodes, so the trie shrinks as words are found.

## How It Actually Works

**Why operations are O(L).** Each character moves one level down by a dictionary lookup
(average O(1)), so the number of stored words never enters the cost. Compare a hash set
of words: `word in set` is also O(L) on average (hashing reads the whole string), but a
set cannot answer "does any word start with `ca`?" without scanning everything. The
trie's advantage is **prefix structure**, not raw lookup speed.

**Memory.** A trie stores at most one node per character inserted, but each node is a
Python object with a dict — far heavier than the characters themselves. Two common
representations:

- **Dict children** (used here): memory proportional to actual branches; flexible
  alphabet.
- **Fixed array of 26 children**: O(1) indexing without hashing, but 26 slots per node
  even when most are empty. Common in C++/Java solutions.

`__slots__` in `TrieNode` tells Python not to create a per-instance `__dict__`, which
noticeably reduces memory when there are many nodes. Compressed tries (**radix trees**)
merge chains of single-child nodes into one edge labelled with a string, cutting the
node count; they are used in routing tables and some key-value stores.

**Why the grid search is faster with a trie.** Without it, each of W words costs a full
grid search. With it, every DFS path is checked against all words simultaneously, and a
path dies as soon as its prefix matches no word, so the work depends on how much of the
grid's path space overlaps with the dictionary's prefixes — typically far less than W
separate searches.

## Common mistakes

- Returning `True` from `search` for a prefix that is not a complete word (forgetting
  `is_word`).
- Creating nodes during `search`/`starts_with` (using `setdefault` there pollutes the
  trie).
- Not restoring the grid cell after backtracking in word search.
- Adding the same word multiple times to the results (use a set, or clear `is_word`).
- Overusing tries: for a single lookup or no prefix queries, a set is simpler.

## Variations to practice

- Design an "add and search word" structure where `.` matches any letter.
- Longest word in a dictionary buildable one letter at a time.
- Count words with a given prefix (store a counter in each node).
- Maximum XOR of two numbers (a binary trie over bits — Level 3, lesson 6).
- Palindrome pairs.

## Exercise

Extend `Trie` with `count_prefix(prefix)` (how many inserted words start with `prefix`)
and `erase(word)` (remove one occurrence), both in O(L). Store a `pass_count` in every
node, incremented on insert and decremented on erase, plus an `end_count` for duplicates.
Assert behaviour after inserting `"apple"` twice and `"app"` once, erasing one
`"apple"`, and querying `"app"`, `"appl"`, and `"b"`.
