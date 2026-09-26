---
description: "Structured 4-, 8- and 12-week coding interview study plans mapped to this course, with daily routines, spaced review, mock schedules and progress checks."
---

# 09 · Study Plans: 4, 8 and 12 Weeks

The biggest predictor of useful preparation is not the number of problems solved but
**consistent, reviewed practice**. Solving 200 problems once and forgetting them is worth
less than solving 80 carefully and revisiting them. This lesson gives three schedules
built on this course, and the routines that make any schedule work.

Choose the plan by the time you have, not by ambition. A 4-week plan followed completely
beats a 12-week plan abandoned in week 3.

## The daily routine (all plans)

A focused session of 60–120 minutes:

1. **Review (10–15 min):** re-solve one problem from your review queue (see below)
   without looking at your old code.
2. **Learn (20–30 min):** read one lesson section or pattern.
3. **Practise (30–60 min):** solve one or two new problems, timed, using the six-step
   framework (lesson 1). Write asserts.
4. **Log (5 min):** record each problem in a spreadsheet: date, pattern, time taken,
   solved alone / with hint / from solution, one-line key insight.

## The review queue (spaced repetition)

Every problem you could not solve alone, or solved slowly, goes into a review queue with
due dates roughly **3 days, 7 days and 21 days** later. On each due date, re-solve it from
scratch. If it goes smoothly, advance it; if not, reset it to 3 days. You can implement
this in a spreadsheet, or with a tiny script:

```python
from datetime import date, timedelta

INTERVALS = [3, 7, 21]

def next_review(last_review, stage, success):
    """Return (next date, next stage). stage indexes INTERVALS; None means retired."""
    if not success:
        return last_review + timedelta(days=INTERVALS[0]), 0
    if stage + 1 >= len(INTERVALS):
        return None, None                                   # learned: retire it
    return last_review + timedelta(days=INTERVALS[stage + 1]), stage + 1


d = date(2026, 1, 1)
assert next_review(d, 0, True) == (date(2026, 1, 8), 1)      # 3-day stage passed -> 7 days
assert next_review(d, 1, False) == (date(2026, 1, 4), 0)     # failed -> back to 3 days
assert next_review(d, 2, True) == (None, None)               # passed final stage
```

The specific intervals are a reasonable starting point, not a scientific constant — adjust
them if problems are coming back too easy or too hard.

## 4-week plan (intensive refresher)

For someone who has studied DSA before and needs to get sharp. About 10–14 hours per week.

| Week | Lessons | Practice focus | Mocks |
|---|---|---|---|
| 1 | L1: 1–9 (skim what you know) | 3 problems/day across arrays, hashing, two pointers, windows, binary search | — |
| 2 | L2: 1–6 | trees, heaps, BFS/DFS, backtracking, 1-D DP | L1 practice set (timed) |
| 3 | L2: 7–9; L4: 1, 5, 7 | intervals, greedy, mixed mediums | 2 mock interviews |
| 4 | L4: 2, 3, 4, 6; review queue | behavioral story bank; weakest patterns | L2 practice set + 2 mocks + capstone |

Skip Level 3 except for topics you know are relevant to your target roles.

## 8-week plan (standard)

For someone with programming experience but rusty or incomplete DSA. About 8–12 hours per
week.

| Week | Lessons | Practice focus |
|---|---|---|
| 1 | L1: 1–3 | complexity, arrays, hashing — 10–12 easy problems |
| 2 | L1: 4–6 | two pointers, sliding window, stacks |
| 3 | L1: 7–10 | linked lists, recursion, binary search; **L1 practice set** |
| 4 | L2: 1–3 | trees, BSTs, heaps |
| 5 | L2: 4–6 | graphs, backtracking, DP intro |
| 6 | L2: 7–10; L4: 1 | prefix sums, greedy, tries; **L2 practice set**; first mock |
| 7 | L3: 1–5 (selectively); L4: 2, 5, 7 | advanced DP, Dijkstra, union-find, monotonic stack; 2 mocks |
| 8 | L4: 3, 4, 6, 8, 10 | review queue, OA simulation, behavioral prep, **capstone** |

## 12-week plan (from foundations)

For someone new to DSA or returning after a long break. About 6–10 hours per week.

| Weeks | Focus |
|---|---|
| 1–4 | All of Level 1, one lesson every 2–3 days, with every exercise. Easy problems only for the first two weeks. L1 practice set at the end of week 4. |
| 5–8 | All of Level 2. Start mediums in week 6. First peer mock in week 8. L2 practice set. |
| 9–10 | Level 3, lessons 1–5 and any others your targets need. L4 lessons 1, 5, 7. Weekly mock. |
| 11 | L4 lessons 2, 4, 6: behavioral story bank, OA simulation, design-lite problems. |
| 12 | Review queue, two mocks, capstone, lesson 8 before offers arrive. |

## Checking progress

At the end of each week, look at your log and answer:

- What share of new problems did I solve alone within the time limit? Is it rising?
- Which pattern appears most often in my "needed a hint" rows? Next week's review targets
  that pattern.
- Am I stating complexity and tracing code by default, or only when I remember?

If a week goes badly, do not try to "catch up" by doubling the next week — extend the
plan by a week instead. Consistency matters more than any particular pace.

## How It Actually Works

Two well-established findings from learning research underpin these plans. The
**spacing effect**: information reviewed at increasing intervals is retained far better
than the same amount of study massed into one session. And **retrieval practice**:
actively recalling (re-solving a problem from scratch) strengthens memory more than
re-reading a solution. The review queue combines both. A related idea, **interleaving**
(mixing problem types rather than drilling one type in a block), makes practice feel
harder but improves the ability to *choose* the right method — which is precisely the
skill an interview tests, since problems do not arrive labelled. That is why the practice
sets and later weeks deliberately mix patterns.

## Common mistakes

- Counting problems solved as the measure of progress, instead of problems solved *alone*.
- Reading solutions too early (give it 20–30 minutes of real effort first).
- Never revisiting solved problems.
- Skipping mocks because they feel uncomfortable — the discomfort is the point.
- Leaving behavioral preparation to the night before.

## Exercise

Choose the plan that fits your available time, copy its table into your own calendar with
specific dates, and set up your problem log with the columns described above. Implement
the review-queue logic in a spreadsheet or extend the script into a small command-line
tool that reads a CSV log and prints today's due problems. Commit to a review date at the
end of week 1 when you will adjust the plan based on your log.
