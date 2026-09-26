---
description: "Prepare for behavioral interviews: build a story bank from real experience, structure answers with STAR, handle follow-ups, and avoid common pitfalls."
---

# 02 · Behavioral Interview Prep

Many engineering interview loops include one or more **behavioral** conversations:
questions about how you have handled real situations — a conflict, a mistake, an
ambiguous project, a deadline. They are sometimes treated as a formality by candidates,
which is a mistake. They are usually evaluated with the same seriousness as the coding
rounds, and they are the round you can prepare for most reliably, because the material is
your own experience.

## What is being assessed

The questions vary, but they tend to probe a small set of themes:

- **Ownership** — did you take responsibility for outcomes, including problems you did
  not cause?
- **Collaboration and conflict** — how do you work with people who disagree with you?
- **Dealing with ambiguity** — what do you do when requirements are unclear?
- **Learning from failure** — can you describe a real mistake and what changed after it?
- **Impact and prioritization** — how do you decide what matters, and what difference did
  your work make?
- **Growth** — how have you learned new skills or helped others learn?

Early-career candidates are not expected to have led large projects. School projects,
internships, open-source contributions, part-time jobs and volunteer work are all
legitimate sources — the reasoning and behaviour matter more than the scale.

## Step 1: build a story bank

Before any interview, write down **8–12 real situations** from your experience. For each,
note a one-line summary and which themes it could illustrate. One good story can often
answer several questions from different angles.

| Story | One-line summary | Themes |
|---|---|---|
| Flaky test suite | Tracked down intermittent CI failures nobody owned | ownership, ambiguity, impact |
| API disagreement | Disagreed with a teammate about an interface; resolved with a prototype | conflict, collaboration |
| Missed deadline | Underestimated a migration; communicated late; changed how I estimate | failure, learning |
| Onboarding doc | Wrote setup docs after struggling myself | growth, helping others |

(These rows are illustrative *formats* — use your own experiences, never invented ones.
Interviewers ask follow-up questions, and fabricated details fall apart quickly; it is
also simply dishonest.)

## Step 2: structure with STAR

**STAR** is a simple structure that keeps answers focused:

- **Situation** — the context, in two or three sentences.
- **Task** — what you specifically were responsible for, or the problem to solve.
- **Action** — what **you** did, step by step. This is the heart of the answer and should
  be the longest part.
- **Result** — what happened, ideally with something concrete (time saved, bug rate
  reduced, feature shipped), and what you learned.

A common refinement is to add a short **reflection** at the end: what you would do
differently now.

### A worked answer

**Question:** "Tell me about a time you disagreed with a teammate."

> **Situation:** On a four-person student project we were building a course-scheduling
> app. Two weeks before the demo, a teammate proposed switching our data layer to a new
> library he'd been reading about.
>
> **Task:** I owned the scheduling logic, which depended heavily on the data layer, and I
> was worried the switch would put the demo at risk.
>
> **Action:** Rather than argue in the group chat, I asked him to walk me through what
> problems the new library would solve. Two of them were real — our queries were slow.
> I suggested we time-box a one-day spike: he would port one screen, and I would measure
> query times before and after. I also wrote down what "done" would mean for the demo so
> we could compare risk honestly.
>
> **Result:** The spike showed a real speed-up but also two days of porting work we
> couldn't afford. We agreed to fix the slow queries with an index for the demo and plan
> the migration afterwards. The demo went smoothly. What I took away was that
> disagreements get easier when you turn opinions into a small experiment.

Notice: specific, first-person actions ("I asked", "I suggested", "I wrote"), a real
trade-off, no villain, and a concrete lesson.

## Step 3: prepare for follow-ups

Interviewers commonly dig into one part of an answer. Rehearse answers to:

- "What would you do differently?"
- "What did your teammate think of that?"
- "How did you measure that result?"
- "What was the hardest part for you personally?"
- "What happened afterwards?"

If you do not remember a detail, say so plainly rather than inventing one.

## Common question shapes

Most behavioral questions are a variation of a few shapes. Map each to stories in your
bank:

- "Tell me about a time you failed / made a mistake."
- "...had to deliver with incomplete information."
- "...disagreed with someone / received critical feedback."
- "...went beyond what was asked."
- "...had to learn something quickly."
- "...had to prioritize between competing tasks."
- "Why do you want to work here / on this team?" (Research the team's actual work;
  connect it honestly to your interests.)
- "Tell me about yourself." (A 60–90 second arc: current focus, one or two highlights,
  what you are looking for next.)

## How It Actually Works

Behavioral interviews rest on a simple premise from industrial-organizational psychology:
**past behaviour in specific situations is a better guide to future behaviour than
hypothetical answers** ("what would you do if...?"). Asking for a specific past example
forces concrete detail, which is harder to fake and easier to evaluate consistently.
That is why interviewers push for specifics and redirect "we" answers to "what did *you*
do?".

STAR works because it mirrors what an evaluator needs to write in feedback: enough
context to understand the stakes, a clear account of the candidate's own contribution,
and evidence of outcome and reflection. Answers that skip Situation are confusing;
answers that skip Action give no evidence; answers that skip Result leave the story
without a point.

## Common mistakes

- Saying "we" throughout, so your contribution is invisible.
- Choosing a "failure" that is really a disguised strength ("I work too hard").
- Blaming others in conflict stories.
- Rambling through a long Situation and rushing the Action.
- Memorizing scripts word for word — you will sound rehearsed and struggle with
  follow-ups. Memorize the **bullet points**, not the sentences.
- Inventing or exaggerating stories.

## Exercise

Build your story bank: write at least eight real situations in the table format above.
Then pick four common questions from the list and write a STAR outline (bullets, not
prose) for each. Finally, say each answer aloud with a timer — aim for about two minutes —
and write down three likely follow-up questions for each, with your honest answers.
