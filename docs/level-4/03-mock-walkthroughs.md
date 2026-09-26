---
description: "Two annotated mock coding interviews — a clean solve of top-k frequent words and a recovery from a wrong first approach to a meeting-rooms problem."
---

# 03 · Mock Interview Walkthroughs

Reading about the framework is not the same as seeing it applied. Below are two
annotated mock sessions, written as transcripts with commentary. The first is a clean
solve. The second starts in the wrong direction and recovers — which is far more common
in real interviews, and far more instructive.

Both are fictional sessions written for teaching. The interviewer lines show typical
behaviour, not any particular company's process.

## Session 1: top k frequent words

> **Interviewer:** Given a list of words and an integer k, return the k most frequent
> words. Sort by frequency, highest first; break ties alphabetically.
>
> **Candidate:** To confirm: output is a list of k words, ordered by count descending, and
> equal counts go in alphabetical order. Is k always valid — between 1 and the number of
> distinct words?
>
> **Interviewer:** Yes.
>
> **Candidate:** And how many words might there be?
>
> **Interviewer:** Up to about 10⁴, and k can be anything in range.

*Commentary: two clarifying questions, both of which matter (k validity removes an edge
case; size tells us O(n log n) is fine).*

> **Candidate:** Example: `["i","love","code","i","love","coding"]`, k = 2. Counts: i:2,
> love:2, code:1, coding:1. Top two by count are "i" and "love", tied at 2, so
> alphabetical: `["i","love"]`.
>
> The simplest approach: count with a hash map, O(n), then sort the distinct words by the
> key (negative count, word) and take the first k. That's O(n + m log m) for m distinct
> words. Given the size, that's fine. If k were much smaller than m and we cared, I could
> use a size-k heap for O(m log k), but the tie-breaking makes the heap comparisons
> fiddlier — I'd start with sorting unless you'd like the heap version.
>
> **Interviewer:** Sorting is fine. Go ahead.

*Commentary: proposes the simple version, names the optimization and its trade-off,
and lets the interviewer choose. Good use of time.*

```python
from collections import Counter

def top_k_frequent_words(words, k):
    counts = Counter(words)
    ordered = sorted(counts, key=lambda w: (-counts[w], w))
    return ordered[:k]


assert top_k_frequent_words(["i", "love", "code", "i", "love", "coding"], 2) == ["i", "love"]
assert top_k_frequent_words(["the", "day", "is", "sunny", "the", "the", "the",
                             "sunny", "is", "is"], 4) == ["the", "is", "sunny", "day"]
assert top_k_frequent_words(["b", "a"], 2) == ["a", "b"]     # all tied: alphabetical
assert top_k_frequent_words(["x"], 1) == ["x"]
```

> **Candidate:** Tracing the first example: counts gives {i:2, love:2, code:1, coding:1}.
> Sort keys: i → (-2,"i"), love → (-2,"love"), code → (-1,"code"), coding → (-1,"coding").
> Sorted: i, love, code, coding. First two: i, love. Correct. Edge case: all words tied —
> they come out alphabetical, which matches the spec.
>
> **Interviewer:** What would the heap version look like?
>
> **Candidate:** I'd push `(count, reversed-order key)` into a min-heap of size k so the
> *worst* candidate is on top. The awkward part is that among equal counts, the word that
> should be evicted first is the alphabetically *later* one, so the heap needs to order
> words in reverse. In Python I'd wrap words in a small class with an inverted `__lt__`.
> Then pop everything and reverse. O(m log k).

*Commentary: a follow-up is not a trap; it is a chance to show depth. Explaining the
difficulty honestly ("the awkward part is...") is better than hand-waving.*

## Session 2: recovering from a wrong start

> **Interviewer:** You're given meeting intervals `[start, end)`. Return the minimum number
> of conference rooms needed.
>
> **Candidate:** So intervals are half-open — a meeting ending at 10 and one starting at 10
> can share a room?
>
> **Interviewer:** Correct.
>
> **Candidate:** Example: `[[0,30],[5,10],[15,20]]` → 2, since [0,30] overlaps both
> others, but those two don't overlap each other.
>
> My first idea: sort by start, and count how many intervals overlap the previous one.
> Each overlap means another room.

*Commentary: this idea is wrong, but the candidate has not coded it yet — which is exactly
why stating the approach first is valuable.*

> **Interviewer:** How would that handle `[[1,5],[2,3],[4,6]]`?
>
> **Candidate:** Let me trace it. Sorted by start: [1,5], [2,3], [4,6]. [2,3] overlaps
> [1,5] — two rooms. [4,6] overlaps... [2,3]? No, 3 ≤ 4. Overlaps [1,5]? Yes. So my rule
> of comparing to the previous interval says no new room, and it would answer... actually
> I realize "count overlaps with the previous" isn't well defined here — the true answer
> is 2, because at time 4 only [1,5] and [4,6] are active, but a different input could
> fool a pairwise rule. The real quantity is the **maximum number of meetings active at
> once**. Comparing to the previous one isn't enough; I need to know when *each* active
> meeting ends.

*Commentary: the interviewer's example was a hint. The candidate traced it honestly,
noticed the rule was ill-defined, and reframed the problem precisely. That reframing is
the turning point.*

> **Candidate:** So: sort by start, and keep a min-heap of end times of meetings currently
> occupying rooms. For each new meeting, if the earliest end is ≤ its start, that room is
> free — pop it. Then push the new end. The heap size is the number of rooms in use; the
> answer is its maximum size. O(n log n) time, O(n) space.
>
> **Interviewer:** Sounds good.

```python
import heapq

def min_meeting_rooms(intervals):
    ends = []
    best = 0
    for start, end in sorted(intervals):
        if ends and ends[0] <= start:
            heapq.heappop(ends)            # reuse the room that frees up earliest
        heapq.heappush(ends, end)
        best = max(best, len(ends))
    return best


assert min_meeting_rooms([[0, 30], [5, 10], [15, 20]]) == 2
assert min_meeting_rooms([[1, 5], [2, 3], [4, 6]]) == 2
assert min_meeting_rooms([[1, 2], [2, 3], [3, 4]]) == 1       # half-open: chain shares
assert min_meeting_rooms([]) == 0
assert min_meeting_rooms([[1, 10], [1, 10], [1, 10]]) == 3
```

> **Candidate:** Tracing `[[1,5],[2,3],[4,6]]`: [1,5] → heap {5}, best 1. [2,3]: 5 > 2, no
> pop; heap {3,5}, best 2. [4,6]: 3 ≤ 4, pop 3; push 6; heap {5,6}, best 2. Answer 2.
> Edge case with back-to-back meetings: each pops the previous end, so one room. Empty
> input: loop doesn't run, returns 0.
>
> Why pop at most once per meeting? The heap holds one entry per room — the end time of
> that room's latest meeting. I only add a room when even the earliest-ending room is still
> busy at this start time, which means *every* room is busy, so a new one really is
> needed. Otherwise I reuse exactly one room. So the heap size never shrinks, and it
> always equals the number of rooms allocated so far.

*Commentary: the last paragraph shows the candidate anticipating a likely follow-up.
Popping *all* ended meetings with a `while` loop also works — then the heap size is "rooms
in use right now" and you take its maximum. Being able to explain why both versions are
correct is a strong signal.*

## What to take from these sessions

- **Say the approach before coding.** Session 2's wrong idea cost two minutes instead of
  fifteen.
- **Treat interviewer examples as hints.** They are almost always chosen to break
  something.
- **Reframe precisely.** "Maximum number active at once" is the real problem; once named,
  the heap is natural.
- **Trace with the actual code**, not with what you meant to write.

## How It Actually Works

A mock interview is useful for the same reason any deliberate practice is: it produces
**feedback on the specific behaviour you want to improve**, under conditions close to the
real ones. Solving problems alone trains pattern recognition; it does not train speaking
while thinking, responding to interruptions, or recovering in front of someone. Those are
separate skills, and they only improve with practice in that format.

Recovery is weighted heavily because it is realistic. Engineers routinely start in the
wrong direction at work; what distinguishes strong ones is noticing quickly, saying so,
and changing course without defensiveness. An interview where the candidate never makes a
mistake gives less evidence about this than one with a well-handled correction.

## Common mistakes in mocks

- Practising only with friends who give hints too freely. Ask your partner to stay close
  to a neutral interviewer role.
- Not recording or writing feedback, so the same issues repeat.
- Only doing problems you already know. Use unseen problems for mocks.
- Stopping the mock to look something up. Treat it as the real thing.

## Exercise

Run two mock interviews with a partner (swap roles), each 45 minutes, using unseen
problems from lesson 5. As the interviewer, write down: every clarifying question asked,
the time when coding started, whether complexity was stated, whether the code was traced,
and how a hint was received. As the candidate, write a short self-review within an hour of
finishing, then compare it with your partner's notes.
