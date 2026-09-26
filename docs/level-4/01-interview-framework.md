---
description: "A six-step framework for live coding interviews: clarify, examples, approach, complexity, code, test — with phrases to use and a worked example."
---

# 01 · The Interview Communication Framework

In a live coding round, the interviewer cannot see inside your head. A correct solution
reached silently gives them little evidence about how you would work on their team; a
slightly imperfect solution reached with clear, structured reasoning often gives them
much more. A framework makes that reasoning visible — and, just as importantly, it keeps
*you* from panicking, because you always know what the next step is.

## The six steps

| Step | Goal | Rough time in a 45-minute round |
|---|---|---|
| 1. Clarify | Pin down inputs, outputs, constraints | 2–4 min |
| 2. Examples | Build and walk through small cases, including edge cases | 2–4 min |
| 3. Approach | Brute force first, then improve; agree before coding | 5–10 min |
| 4. Complexity | State time and space of the chosen approach | 1 min |
| 5. Code | Write clean code, narrating the non-obvious parts | 10–15 min |
| 6. Test | Trace an example, run edge cases, fix bugs | 5 min |

These times are a guide, not a rule; some problems need more thinking and less code.

### Step 1: clarify

Restate the problem in your own words, then ask about anything that would change the
solution. Useful questions:

- **Input shape and size.** "How large can `n` be?" (This tells you the target
  complexity — Level 1, lesson 1.) "Can the array be empty?"
- **Value ranges.** Negatives? Zeros? Duplicates? Very large numbers?
- **Output details.** Indices or values? Any order? What if there is no answer — return
  -1, empty list, raise?
- **Mutability.** "May I modify the input?"

Avoid asking questions whose answers do not matter; it can look like stalling.

### Step 2: examples

Write one normal example and work it by hand. Then list edge cases: empty, one element,
all equal, already sorted/reverse sorted, negative values, no valid answer. Working an
example by hand is often where the approach reveals itself.

### Step 3: approach

State a brute force in one or two sentences with its complexity, even if it is obviously
too slow. It proves you can solve the problem, gives you a fallback, and gives the
interviewer something to react to. Then look for the bottleneck — the repeated work — and
name the pattern that removes it. **Get agreement before coding:** "Does this approach
sound reasonable, or would you like me to consider something else?"

### Step 4: complexity

"This is O(n log n) time because of the sort, and O(n) extra space for the hash map."
Stating it before coding shows you are choosing deliberately.

### Step 5: code

- Use clear names (`left`, `right`, `seen`, `window_sum`), not `a`, `b`, `x2`.
- Write helper functions for self-contained pieces; it is fine to say "I'll write
  `is_valid` as a helper and fill it in after."
- Narrate decisions, not keystrokes: "I check the complement before inserting so an
  element can't pair with itself" is useful; "now I'm typing a for loop" is noise.
- If you realize a bug mid-way, say so and fix it. Hiding it is worse.

### Step 6: test

Do not say "I think it works." **Trace** your code on the small example line by line,
tracking variables. Then run through the edge cases you listed in step 2. If you find a
bug, fix it and re-trace the affected part. Lesson 7 covers this step in depth.

## A worked example (condensed)

**Problem as given:** "Find the length of the longest substring without repeating
characters."

> **Candidate:** So given a string, I return the length — not the substring itself — of
> the longest contiguous run with all distinct characters. What characters can appear —
> just lowercase letters, or any Unicode? And how long can the string be?
>
> **Interviewer:** Any characters. Up to 10⁵.
>
> **Candidate:** Then O(n²) is probably too slow. Examples: `"abcabcbb"` → 3 for `"abc"`;
> `"bbbb"` → 1; empty string → 0. A tricky one is `"abba"`: when I reach the second `a`,
> the earlier `a` is already outside my window.
>
> Brute force: check every substring for uniqueness, O(n²) with a set if I extend
> incrementally. The repeated work is re-scanning characters I've seen. Since I want the
> longest *contiguous* window where a condition holds, and shrinking always fixes a
> duplicate, this is a sliding window. I'll store the last index of each character and
> jump the left edge past a duplicate — but only forward, which handles `"abba"`. O(n)
> time, O(k) space for k distinct characters. Shall I code that?
>
> **Interviewer:** Go ahead.

The code is the one from Level 1, lesson 5. The candidate then traces `"abba"`
explicitly, showing that `left` stays at 2.

## Handling hints

A hint is information, not a judgement. Acknowledge it, connect it to what you were
thinking, and use it: "Good point — if the array is sorted, then I don't need the hash
map; two pointers would work in O(1) space." Arguing with a hint, or ignoring it, is one
of the most avoidable ways to do poorly.

## When you are stuck

Say what you are trying, out loud: "I'm looking for a way to avoid recomputing the sum
for every window..." Then use these unsticking moves in order:

1. Work a second, different example by hand.
2. Solve a simpler version (sorted input, no duplicates, k = 1).
3. Run through the pattern list: hash map? two pointers? sliding window? sort first?
   binary search on the answer? BFS? DP? heap?
4. Ask what the brute force repeats, and what structure would remember it.

## How It Actually Works

Interviews are an evaluation under **uncertainty**. An interviewer has roughly 45 minutes
to gather evidence about several distinct things — problem solving, coding fluency,
communication, testing habits — and must later justify a hiring recommendation in
writing. Many organizations use structured feedback forms with separate dimensions for
those skills; the exact rubric varies, but the principle is common because structured
interviews are more consistent than unstructured impressions.

The framework works because each step produces **observable evidence** for one of those
dimensions: clarifying questions and examples show problem understanding; the brute force
and optimization show analytical skill; complexity statements show fundamentals; narrated
code shows fluency and communication; tracing shows verification habits. A silent
candidate who writes perfect code produces evidence for only one or two of them.

There is also a cognitive reason. Talking through small examples and naming the brute
force offloads working memory onto the page, which is exactly what you need under stress.
The structure means you never face a blank "what now?" moment — the next step is always
defined.

## Common mistakes

- Coding immediately after reading the problem.
- Skipping the brute force because "it's obviously too slow".
- Asking whether to use a language feature rather than just using it sensibly.
- Monologuing without checking in; pause after the approach and ask for agreement.
- Declaring done without tracing the code.
- Treating the interviewer as an adversary instead of a collaborator.

## Exercise

Pick three problems you have already solved from Levels 1–2. For each, record yourself
(audio is enough) going through all six steps as if in an interview, aiming for 25–30
minutes each. Listen back and note: how long you spent on each step, whether you stated
complexity before coding, whether you traced the code, and every moment of silence
longer than about 30 seconds. Write one concrete change for the next attempt.
