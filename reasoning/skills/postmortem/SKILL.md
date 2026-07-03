---
name: postmortem
description: >
  Run an engineering postmortem on a just-finished task or session — what was
  learned, where time was wasted, and the concrete fixes so it goes faster next
  time. Reconstructs the facts, hands them to ONE well-briefed reviewer
  subagent, and returns a root-cause analysis, a minimum-round-trip replay, and
  proposed edits to the skill/config/code that caused the friction. Use
  when the user says "postmortem", "retro", "what did we learn", "how could that
  have gone better", "how do we do this more efficiently next time", or asks to
  review how a piece of work went.
---

# Postmortem

Turn a finished (or stuck) piece of work into a short, honest postmortem that
ends in concrete fixes — not vibes. The output is for next time: cheaper
round-trips, fewer user corrections, a skill/config/code edit that bakes the
lesson in.

## The one rule

**One well-briefed reviewer, not a panel.** It is tempting to run several agents
in different "formats" (5-Whys, timeline, start/stop/continue) for thoroughness.
Don't — they converge on the same findings and you pay N× the tokens for one
diagnosis. A single agent, given the full facts and asked for all three
artifacts in one pass, produces the same conclusions. (Spend the budget on the
*brief*, not on copies of the agent.)

Exception — run a **second, independent** agent only when the diagnosis is
genuinely uncertain or the stakes are high and you want a confidence check that
the root cause isn't an artifact of how you framed the brief. That's a
deliberate adversarial second opinion, not a format reshuffle.

## Step 1 — Reconstruct the facts (you do this, not the agent)

The reviewer was not present, so its analysis is only as good as the facts you
hand it. From the conversation/session, write a tight **chronological** account:

- Each step taken, in order — searches, writes, tool calls, decisions.
- **Every user correction or redirect** ("no, do it this way", "trust me",
  "actually find X first"). These are the postmortem's gold — each one marks a
  place the default path was wrong.
- **Every wasted round-trip** — retries, wrong paths, blown token caps, dead-end
  plans, things done in two passes that could've been one.
- The **end state** reached, and whether it was correct.

Be specific and name real things (files, fields, queries, names). Vague facts →
vague fixes.

## Step 2 — Point at the artifacts

List the files the friction actually lives in, with absolute paths, and tell the
agent to **read them** so its fixes quote real text instead of guessing:

- The skill / script / config / code that drove the work.
- The **source of truth** it should have matched (schema doc, API reference,
  spec). Mismatches between an instruction file and the real schema are a
  classic root cause — make the agent reconcile them.

## Step 3 — Launch ONE reviewer subagent

Use `subagent_type: distinguished-engineer` (model `opus`, high effort) — it does
critical chain-of-thought review. Tell it **not** to use AskUserQuestion (it's a
background agent; it should make reasonable assumptions and just write the
postmortem). Hand it this brief:

```
You are doing an engineering postmortem of a just-finished work session. You
were NOT present — here are the facts. Read the artifact files listed so your
fixes quote the real text. Do not use AskUserQuestion; make reasonable
assumptions and write the postmortem. Be concrete and honest; the end state was
[correct/...], so focus on cost, not blame.

## What we were trying to do
[goal in one or two sentences]

## What actually happened (chronological)
[the numbered account from Step 1]

## Artifacts to read (quote real text in any proposed edit)
[absolute paths from Step 2, with what each is]

## Produce, in one pass:
1. ROOT CAUSE — drive a 5-Whys chain on the single most expensive
   inefficiency to its root. Separate the root cause from contributing
   paper-cuts.
2. MINIMUM-ROUND-TRIP REPLAY — the ideal sequence that reaches the same end
   state in the fewest actions, with each user redirect designed out.
3. CONCRETE FIXES — before→after edits to the artifact file(s) that make the
   ideal path the default. Quote the real "before" text. Tie each fix to a
   moment in the timeline.
End with the SINGLE highest-leverage fix — if only one ships, which one.
```

## Step 4 — Deliver and offer to apply

Relay the postmortem (don't dump the raw agent output — surface the root cause,
the replay, and the proposed edits as before→after diffs). Then offer the fixes
via `AskUserQuestion` with `multiSelect: true`, one option per proposed fix
(label = the file + a few words; description = what the edit changes and why).

Group the fixes logically into pages — one question per group (e.g. by artifact
file, or root-cause fixes vs. paper-cuts), with the group named in the question
`header`. A single AskUserQuestion call holds up to 4 questions of 4 options
each; if the fixes overflow that, make additional calls, one page at a time.
Don't split a coherent group just to fill pages — a lone group is one question.

Apply the selected fixes; leave the rest as proposals in the writeup.

## Keep it honest

- The point is the *next run*, not a writeup. If the only real lesson is "one
  tooling slip, nothing structural", say that in two lines and stop — don't
  manufacture a framework.
- Name what went *right* too, briefly — the discipline worth keeping (e.g. never
  shipped bad data, recovered cleanly) so the fixes don't trample it.
- Match effort to the work: a 10-minute task gets a paragraph, not a subagent.
  Reach for the reviewer when the session had real rework worth designing out.
