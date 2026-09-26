---
description: "Capstone: a complete annotated 45-minute mock coding interview transcript, the tested solution, a scoring rubric, and a self-review template."
---

# 10 · Capstone — Full Mock Interview

This capstone brings the whole course together. First, read a complete 45-minute mock
interview transcript with commentary. Then score it with the rubric. Finally, run your own
full mock and review it with the same rubric and template.

The transcript is a fictional teaching example. The problem is a well-known style of
medium-to-hard question; the interviewer's behaviour reflects common practice, not any
specific company's process.

## Part 1 — The transcript

**Problem statement (as spoken):** "You're given a list of flight tickets as
`[from, to]` pairs. They form a single trip that uses every ticket exactly once, starting
from `JFK`. Reconstruct the itinerary. If there are several valid itineraries, return the
one that is smallest in lexicographic order when read as a single sequence."

### 0:00–0:05 — Clarify

> **Candidate:** Let me restate: each ticket is a directed edge from one airport to
> another. I need an ordering of airports, starting at JFK, that uses every ticket once —
> so the output has one more airport than there are tickets. Among valid orderings I
> return the lexicographically smallest. A few questions: is a valid itinerary guaranteed
> to exist?
>
> **Interviewer:** Yes.
>
> **Candidate:** Can there be duplicate tickets — the same from/to twice?
>
> **Interviewer:** Yes, and each must be used.
>
> **Candidate:** And roughly how many tickets?
>
> **Interviewer:** Up to a few hundred.

*Commentary: the restatement already translates the problem into graph language
("directed edge", "uses every ticket once"). Asking about duplicates matters: it rules
out storing destinations in a set.*

### 0:05–0:10 — Examples

> **Candidate:** Example: `[[JFK,SFO],[JFK,ATL],[SFO,ATL],[ATL,JFK],[ATL,SFO]]`. From JFK
> I can go to ATL or SFO; ATL is smaller. JFK→ATL, then from ATL: JFK or SFO; JFK is
> smaller → JFK, then SFO, ATL, SFO. That gives JFK, ATL, JFK, SFO, ATL, SFO — uses all 5
> tickets. Good.
>
> But greedy "always take the smallest" can fail. Consider
> `[[JFK,KUL],[JFK,NRT],[NRT,JFK]]`. Greedy goes JFK→KUL and then KUL has no outgoing
> tickets — stuck, with two tickets unused. The right answer is JFK, NRT, JFK, KUL.

*Commentary: constructing a counterexample to the obvious greedy **before** coding is the
single most valuable move in this interview. It shows the candidate tests ideas rather
than trusting them.*

### 0:10–0:20 — Approach

> **Candidate:** Brute force: backtracking — try destinations in sorted order, recurse,
> and undo if I get stuck before using every ticket. The first complete path found is the
> smallest, because I try options in sorted order. Worst case that's exponential, though
> in practice it's often fine for a few hundred tickets.
>
> Can I do better? Using every edge exactly once is an **Eulerian path**. There's a
> classic linear-time method, Hierholzer's algorithm: walk from the start, always taking
> an unused edge, and when you reach a node with no unused edges, add it to the route
> *as you backtrack*. The route comes out reversed. The stuck node, KUL in my example,
> gets appended first — so it ends up last in the final order, which is exactly where a
> dead end must go.
>
> To get the lexicographically smallest, I take the smallest available destination each
> time. I'll keep each airport's destinations in a min-heap, or sort them in reverse and
> pop from the end of a list.
>
> Complexity: sorting edges is O(E log E); the traversal visits each edge once, so O(E)
> after that. Space O(E). Shall I go with that?
>
> **Interviewer:** Why does taking the smallest destination still give the smallest
> itinerary, given greedy failed earlier?
>
> **Candidate:** Because now greedy isn't committing to the *order of output*. When the
> smallest choice leads to a dead end — like KUL — that dead end is emitted first in
> post-order, which puts it at the end of the final route. The walk then continues with
> the next choice from where it left off, and those edges get placed before the dead end.
> So a smaller branch only goes "first" if it can be continued; otherwise it's pushed to
> the end, which is the only place it can go anyway.
>
> **Interviewer:** Okay, go ahead.

*Commentary: the interviewer probed the key insight. The candidate's answer connects
back to the earlier counterexample — strong evidence of understanding, not memorization.
Note the candidate offered backtracking first; had they not known Hierholzer's
algorithm, a well-implemented backtracking solution would still have been a reasonable
outcome.*

### 0:20–0:32 — Code

```python
from collections import defaultdict

def find_itinerary(tickets):
    graph = defaultdict(list)
    for src, dst in sorted(tickets, reverse=True):
        graph[src].append(dst)            # reverse-sorted: pop() gives the smallest
    route, stack = [], ["JFK"]
    while stack:
        airport = stack[-1]
        if graph[airport]:
            stack.append(graph[airport].pop())   # take smallest unused ticket
        else:
            route.append(stack.pop())            # dead end: emit in post-order
    return route[::-1]
```

> **Candidate:** I wrote it iteratively so a long itinerary can't hit the recursion limit.
> `stack` is the current walk; when the top airport has no tickets left, it's finished and
> goes to `route`. At the end I reverse.

### 0:32–0:40 — Test

> **Candidate:** Tracing the tricky case `[[JFK,KUL],[JFK,NRT],[NRT,JFK]]`. Graph after
> reverse sort: JFK: [NRT, KUL], NRT: [JFK].
>
> | stack | action | route |
> |---|---|---|
> | [JFK] | pop KUL from JFK's list | [] |
> | [JFK, KUL] | KUL has none → emit KUL | [KUL] |
> | [JFK] | pop NRT | [KUL] |
> | [JFK, NRT] | pop JFK | [KUL] |
> | [JFK, NRT, JFK] | JFK empty → emit | [KUL, JFK] |
> | [JFK, NRT] | NRT empty → emit | [KUL, JFK, NRT] |
> | [JFK] | emit | [KUL, JFK, NRT, JFK] |
>
> Reversed: JFK, NRT, JFK, KUL. Correct. Edge cases: a single ticket gives [JFK, X].
> Duplicate tickets: both copies are in the list, so both get used. Let me also write
> these as asserts.

```python
assert find_itinerary([["MUC", "LHR"], ["JFK", "MUC"], ["SFO", "SJC"], ["LHR", "SFO"]]) == \
    ["JFK", "MUC", "LHR", "SFO", "SJC"]
assert find_itinerary([["JFK", "SFO"], ["JFK", "ATL"], ["SFO", "ATL"],
                       ["ATL", "JFK"], ["ATL", "SFO"]]) == \
    ["JFK", "ATL", "JFK", "SFO", "ATL", "SFO"]
assert find_itinerary([["JFK", "KUL"], ["JFK", "NRT"], ["NRT", "JFK"]]) == \
    ["JFK", "NRT", "JFK", "KUL"]
assert find_itinerary([["JFK", "AAA"]]) == ["JFK", "AAA"]             # single ticket
assert find_itinerary([["JFK", "A"], ["A", "JFK"], ["JFK", "A"]]) == \
    ["JFK", "A", "JFK", "A"]                                          # duplicate tickets
```

*Commentary: the trace uses the actual code and a table. The candidate chose the
counterexample from step 2 as the test — the case most likely to break.*

### 0:40–0:45 — Follow-up and wrap-up

> **Interviewer:** What if a valid itinerary weren't guaranteed?
>
> **Candidate:** I'd check the Eulerian path conditions first: for a directed graph, every
> node has equal in- and out-degree except the start (out = in + 1) and the end
> (in = out + 1), or all are equal; and all edges must be reachable from the start. Or,
> more simply, run the algorithm and check the route length is `len(tickets) + 1`.
>
> **Interviewer:** Great. Any questions for me?
>
> **Candidate:** Yes — what does a typical week look like for new engineers on your team?

*Commentary: a concise, correct follow-up with a cheap practical check. Having a genuine
question ready at the end is part of the conversation too.*

## Part 2 — The rubric

Score each dimension 1–4. The descriptions give the anchors for 2 and 4; 1 and 3 sit
below and between them.

| Dimension | 2 — developing | 4 — strong |
|---|---|---|
| Problem understanding | Restates problem; misses a constraint | Restates precisely; asks questions that change the solution |
| Approach & reasoning | Reaches a working idea with help | Brute force → bottleneck → optimized idea; tests ideas with counterexamples |
| Coding | Works with some messy or redundant parts | Clean, idiomatic, well-named; appropriate structure (e.g. iterative for depth) |
| Complexity | States it when asked, minor errors | States time and space unprompted and correctly |
| Testing | Runs the example only | Chooses targeted cases, traces the real code, fixes bugs by cause |
| Communication | Explains when prompted | Narrates reasoning, checks in, uses hints well |

**Scoring the transcript** (your own judgement may differ — that is part of the
exercise): most readers would score it 4 on understanding, approach, complexity and
testing; coding and communication are also strong. A realistic "good" performance often
has a 3 or two — a bug caught late, a hint needed — and that is fine.

## Part 3 — Your own full mock

1. Find a partner (or an unfamiliar problem and a recorder). Choose an unseen problem
   from lesson 5, Style B or E.
2. Run a strict 45 minutes: 5 minutes of introductions or behavioral questions, 35 of
   coding, 5 of questions for the interviewer.
3. Record it, with permission.
4. Within an hour, fill in this self-review template:

```text
Problem:
Time coding started (mm:ss):
Clarifying questions I asked:
Edge cases I listed before coding:
Brute force stated? (y/n)          Complexity stated unprompted? (y/n)
Hints received and how I used them:
Bugs found — by me / by interviewer — and their causes:
Rubric scores (1-4): understanding _ approach _ coding _ complexity _ testing _ communication _
Longest silence (approx.):
One thing to repeat next time:
One thing to change next time:
```

5. Listen to the recording, then update the template — memory is kinder than recordings.
6. Repeat weekly, rotating problem styles, until your lowest rubric score is at least 3.

## How It Actually Works

A full mock works as a capstone because interview performance is an **integration
skill**: pattern recognition, coding fluency, communication and verification all have to
run at the same time, under time pressure and observation. Practising them separately
builds components; only the full simulation trains the coordination between them — such
as keeping track of an edge case while explaining an approach, or noticing a bug while
someone is watching.

The rubric makes feedback **specific and comparable over time**. "That went okay" is not
actionable; "testing: 2 — only ran the given example" is. Scoring each dimension
separately also stops one strong area (say, fast coding) from masking a weak one (never
stating complexity). Reviewing a recording addresses a well-known limitation of memory:
people tend to recall their performance as a summary impression and forget specific
moments, which are exactly what you need to improve.

## Common mistakes

- Doing capstone mocks only on familiar problems — use unseen ones.
- Skipping the self-review because the mock "felt fine".
- Changing too many things at once; pick one improvement per mock.
- Treating a bad mock as a verdict instead of data.

## Exercise

Complete three full mocks over three weeks, each with a filled self-review template and
rubric scores. Plot (on paper is fine) your six rubric scores across the three mocks.
Write a short final reflection: which dimension improved most, which is still weakest,
and the specific practice you will do for it — then schedule that practice in your study
plan from lesson 9.
