---
description: "Search strings in linear time with KMP's prefix function and Rabin-Karp rolling hashes, and find palindromic substrings by expanding around centers."
---

# 08 · String Algorithms

Python's `in`, `str.find`, and slicing hide most string work, and for interviews that is
usually fine. But several classic problems need you to **reuse information** across
positions rather than restart comparisons: finding a pattern in linear time, detecting
repeated substrings, and palindromes. This lesson covers the three techniques that come
up most: the **prefix function (KMP)**, **rolling hashes (Rabin-Karp)**, and **expanding
around centers**.

## The naive search and its problem

```python
def naive_find(text, pattern):
    n, m = len(text), len(pattern)
    for i in range(n - m + 1):
        if text[i:i + m] == pattern:
            return i
    return -1


assert naive_find("hello", "ll") == 2
assert naive_find("aaaaa", "bba") == -1
assert naive_find("abc", "") == 0
```

Worst case O(n · m): for `text = "aaaa...ab"` and `pattern = "aaa...ab"`, almost every
alignment compares nearly all of the pattern before failing, then shifts by just one and
repeats the same comparisons.

## KMP and the prefix function

**Prefix function.** For each position `i` of a string `s`, `pi[i]` is the length of the
longest proper prefix of `s[:i+1]` that is also a suffix of it.

For `s = "abacaba"`: `pi = [0, 0, 1, 0, 1, 2, 3]` — at the end, `"aba"` is both a prefix
and a suffix of the whole string.

```python
def prefix_function(s):
    pi = [0] * len(s)
    k = 0                                   # length of the current matched border
    for i in range(1, len(s)):
        while k > 0 and s[i] != s[k]:
            k = pi[k - 1]                   # fall back to the next shorter border
        if s[i] == s[k]:
            k += 1
        pi[i] = k
    return pi


assert prefix_function("abacaba") == [0, 0, 1, 0, 1, 2, 3]
assert prefix_function("aaaa") == [0, 1, 2, 3]
assert prefix_function("abcd") == [0, 0, 0, 0]
assert prefix_function("") == []
```

**Search with it.** Run the prefix function on `pattern + "\x00" + text` (a separator
that appears in neither). Any position where `pi` equals `len(pattern)` ends a match.

```python
def kmp_find_all(text, pattern):
    if not pattern:
        return list(range(len(text) + 1))
    combined = pattern + "\x00" + text
    pi = prefix_function(combined)
    m = len(pattern)
    return [i - 2 * m for i in range(len(combined)) if pi[i] == m]


assert kmp_find_all("abababa", "aba") == [0, 2, 4]      # overlapping matches
assert kmp_find_all("hello", "ll") == [2]
assert kmp_find_all("aaa", "b") == []
assert kmp_find_all("ab", "abc") == []                  # pattern longer than text
```

(The index arithmetic: a match ending at combined index `i` starts at text index
`i - m + 1 - (m + 1) = i - 2m`.)

### Worked problem: shortest palindrome

**Problem.** Add the fewest characters to the **front** of `s` to make it a palindrome.

**Insight.** You need the longest prefix of `s` that is a palindrome; everything after
it must be mirrored onto the front. A prefix of `s` is a palindrome exactly when it
equals a suffix of `reversed(s)` — a border question, answered by the prefix function of
`s + "#" + s[::-1]`.

```python
def shortest_palindrome(s):
    combined = s + "#" + s[::-1]
    longest_pal_prefix = prefix_function(combined)[-1]
    return s[longest_pal_prefix:][::-1] + s


assert shortest_palindrome("aacecaaa") == "aaacecaaa"
assert shortest_palindrome("abcd") == "dcbabcd"
assert shortest_palindrome("") == ""
assert shortest_palindrome("aba") == "aba"               # already a palindrome
```

O(n) time and space.

## Rabin-Karp: rolling hashes

**Idea.** Treat a window of characters as a number in base `B`, modulo a large prime `M`.
Sliding the window by one character updates the hash in O(1): remove the leading
character's contribution, shift, add the new one. Compare hashes first; only if they
match do you compare the strings (to rule out collisions).

### Worked problem: longest duplicate substring

**Problem.** Find the longest substring that occurs at least twice (occurrences may
overlap).

**Approach.** If a duplicate of length `L` exists, one of length `L - 1` does too — the
predicate is monotonic, so **binary search on the length** (Level 1, lesson 9). For each
candidate length, roll a hash across the string and look for a repeat.

```python
def longest_dup_substring(s):
    n = len(s)
    B, M = 256, (1 << 61) - 1                        # large Mersenne prime modulus

    def find_dup(length):
        if length == 0:
            return ""
        h, power = 0, pow(B, length, M)
        seen = {}
        for i, ch in enumerate(s):
            h = (h * B + ord(ch)) % M
            if i >= length:
                h = (h - ord(s[i - length]) * power) % M
            if i >= length - 1:
                start = i - length + 1
                for prev in seen.get(h, []):
                    if s[prev:prev + length] == s[start:start + length]:
                        return s[start:start + length]      # verified, not a collision
                seen.setdefault(h, []).append(start)
        return None

    lo, hi, best = 1, n - 1, ""
    while lo <= hi:
        mid = (lo + hi) // 2
        found = find_dup(mid)
        if found is not None:
            best, lo = found, mid + 1
        else:
            hi = mid - 1
    return best


assert longest_dup_substring("banana") == "ana"
assert longest_dup_substring("abcd") == ""
assert longest_dup_substring("aaaa") == "aaa"             # overlapping occurrences
assert longest_dup_substring("a") == ""
```

Expected O(n log n) time: O(log n) lengths, O(n) hashing each, with rare collisions
verified by direct comparison. Always verify (or state that you are accepting a tiny
collision probability) — interviewers ask.

## Palindromes: expand around centers

**Problem.** Find the longest palindromic substring.

**Approach.** Every palindrome has a center: a character (odd length) or a gap between
two characters (even length). There are `2n - 1` centers; expand outward from each while
the ends match.

```python
def longest_palindrome(s):
    if not s:
        return ""
    best_lo, best_hi = 0, 0
    for center in range(2 * len(s) - 1):
        lo, hi = center // 2, (center + 1) // 2      # same index, or adjacent pair
        while lo >= 0 and hi < len(s) and s[lo] == s[hi]:
            lo -= 1
            hi += 1
        if (hi - 1) - (lo + 1) > best_hi - best_lo:
            best_lo, best_hi = lo + 1, hi - 1
    return s[best_lo:best_hi + 1]


assert longest_palindrome("babad") in ("bab", "aba")
assert longest_palindrome("cbbd") == "bb"                # even-length center
assert longest_palindrome("a") == "a"
assert longest_palindrome("") == ""
assert longest_palindrome("forgeeksskeegfor") == "geeksskeeg"
```

O(n²) time, O(1) space — simpler than the DP table and usually what interviewers expect.
Manacher's algorithm reaches O(n) by reusing mirror information inside a known
palindrome; it is worth knowing it exists, rarely worth coding in an interview.

## How It Actually Works

**Why KMP is linear.** In `prefix_function`, `k` increases by at most 1 per iteration of
the `for` loop, and every step of the inner `while` strictly decreases it (since
`pi[k-1] < k`). `k` never goes below 0, so the total number of decreases across the whole
run is bounded by the total number of increases, which is at most n. Total work: O(n).
The fallback `k = pi[k - 1]` is the key idea: when the next character does not extend the
current border, the next candidate is the longest border *of that border* — a shorter
prefix that is still a suffix — so no earlier comparison has to be redone.

**Why rolling hashes work.** The polynomial hash of `s[i..i+L-1]` is
`s[i]·B^(L-1) + ... + s[i+L-1]` mod M. Moving the window subtracts `s[i]·B^L`
(after the shift), multiplies by B, and adds the new character — all O(1) with modular
arithmetic. Two different strings of length L collide with probability roughly L/M for a
random-looking base and a prime modulus, which is negligible for a 61-bit prime. But
**adversarial inputs** can be built against a fixed, known base and modulus, which is why
production code randomizes the base or uses two moduli, and why correctness-critical code
verifies matches.

**Why center expansion is O(n²) and correct.** Every palindrome is determined by its
center and radius, and expanding from a center finds the maximal palindrome there. With
2n − 1 centers and up to n/2 expansion steps each, the worst case (e.g. `"aaaa...a"`) is
quadratic.

## Common mistakes

- Using a separator in KMP that can appear in the input.
- Off-by-one in converting match end indices to start indices.
- Rolling-hash subtraction without `% M` producing negative values (Python's `%` keeps
  it non-negative, but in Java/C++ you must add M).
- Trusting a hash match without verification (or without saying you accept the risk).
- Forgetting even-length palindromes when expanding around centers.

## Variations to practice

- Repeated substring pattern (`s` is `k` copies of a block iff `n % (n - pi[-1]) == 0`
  and `pi[-1] > 0`).
- Count palindromic substrings (center expansion, counting instead of maximizing).
- Repeated DNA sequences (rolling hash or a set of fixed-length slices).
- Minimum window substring (Level 1 sliding window).
- Z-function (an alternative to the prefix function).

## Exercise

Solve **Repeated Substring Pattern**: decide whether `s` can be built by repeating one of
its proper substrings. Implement it twice — once with the prefix-function identity above
and once with the trick `s in (s + s)[1:-1]` — and write a one-paragraph explanation of
why the doubled-string trick works. Test with `"abab"`, `"aba"`, `"abcabcabc"`, `"a"`,
and `"aa"`.
