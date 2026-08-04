---
name: premortem
description: >
  Run a premortem on a plan before executing it — imagine the plan has
  already failed, surface the failure modes, and harden the plan against
  them. Use when the user says "premortem", "pre-mortem", "poke holes in
  this plan", "imagine this failed", "stress-test this plan", or "what
  could go wrong with this plan". Requires an actual plan, spec, or written
  approach on the table AND an explicit request to stress it. Do NOT
  trigger on a rhetorical "what could go wrong", on general risk chat with
  no plan, or on work already finished (that's postmortem). This skill can
  spawn multiple subagents — it is expensive; never trigger it
  speculatively.
---

# Premortem

Turn a plan that's about to be executed into a ranked register of the ways it
fails — before any of them get the chance. The method is Klein's prospective
hindsight ("it's months later and the plan failed — write the history of the
failure"), run as a small agent team. The output is a hardened plan: the kill
risk named, tripwires set, the top mitigations folded in.

## The one rule

**Blind first pass.** Each lens writes its failure history before seeing any
other's, so the second and third lists aren't anchored on the first. Be honest
about what this buys: de-anchoring, not independence. The agents share a model
and a brief, so agreement across lenses is one opinion stated three times —
never treat cross-lens frequency as corroboration.

## Step 0 — Choose the shape

First, decide the scale, and state the chosen shape in one line before doing
anything else. Launch the team only if ALL three hold:

- The plan is **hard to reverse or wide** — a migration, deploy, release,
  schema change, or 3+ files.
- Executing it **costs clearly more than the premortem** — rough line: 30+
  minutes of work.
- A **written plan, spec, or plan-mode plan exists** to point the agents at.

If any fail: write 3–5 failure modes inline in a single paragraph and stop.
No agents. A casual ask about a small change gets the inline answer.

If the plan already went through a review or an ExitPlanMode approval this
session, say so and ask what the premortem should add before spawning
anything.

## Step 1 — Assemble the brief (you, not the agents)

State the plan verbatim (or give the plan file's absolute path), the goal and
definition of success, the key assumptions it rests on, and its constraints
(deadline, budget, irreversibility). Vague brief → generic failure modes.

Point each lens at **different primary material**, at most three paths each —
technical → the code; assumptions → dependency manifests, API docs, README
claims; process → the conversation and how the result will be used. What
independence exists comes from inputs, not adjectives.

Hard precondition: if the brief contains no verbatim plan text and no paths,
do not launch agents. Write the plan into the brief in enough detail that a
stranger could critique it, confirm it with the user, then proceed. A
premortem with nothing concrete to read returns platitudes.

## Step 2 — Round 1: blind imagination

Launch 3 one-shot agents **in parallel** (one message, three Agent calls),
`subagent_type: general-purpose` — prefer a *generic* read-only subagent type
if the install has one, but not `reasoning:distinguished-engineer`: its
plan-reviewer persona and verdict format would override the lens framing.
Each brief carries, verbatim:

- **Read-only contract**: "You are read-only. Do not create, edit, or delete
  any file; run no state-changing command; do not use AskUserQuestion. Your
  final message is your entire output."
- **The framing**: "It is [timeframe] later. The plan was executed and it
  failed badly. Write the history of the failure." — plus ONE lens:
  *technical/implementation* (bugs, integration breaks, things that don't
  work as documented), *assumptions/external* (premises that turn out false,
  dependencies that shift, estimates that don't survive contact), or
  *human/process* (miscommunication, sequencing failures, "done" that isn't
  done).
- **Grounding rule**: "Every failure mode cites a specific line of the plan
  or a file:line you actually read, quoted — otherwise label it SPECULATIVE.
  Never invent file names, symbols, or schemas. Reject any failure mode that
  would read the same for a different plan."
- **Return format**: 4–6 failure modes, most damaging first, each as WHAT
  BROKE / TRIGGER / EARLY WARNING / MITIGATION.
- **Reading cap**: "Read the brief and the listed paths only — do not explore
  the codebase."

If the Agent tool is missing or a call errors, do not retry with different
parameters: run the lenses yourself in sequence, writing each lens's full
list before starting the next, and disclose that in the deliverable.

## Step 3 — Round 2: cross-examination

Merge and dedupe **by mechanism**: two findings are the same if the same
event triggers them, whatever the vocabulary. Carry at most 8 forward.

Then launch three fresh one-shot agents — this is the primary path; it works
everywhere, because round 2 needs only the merged text — each handed the
merged register with the same three asks: (a) ATTACK what should be dropped
as unlikely or low-impact, (b) RANK by likelihood × impact, (c) ADD at most
one new failure mode the collision of perspectives suggests. (If the
environment has named agents and SendMessage, resuming the round-1 agents is
an allowed optimization — same instructions either way.)

Synthesize: rank by median position across the three replies; where they
disagree by more than three places, say so in the register instead of picking
a winner; drop findings whose cited evidence no attacker could locate. An
agent can return nothing (killed, skipped, errored) — synthesize from
whatever came back, and never block the deliverable on a missing reply.

## Step 4 — Deliver and offer to apply

Relay in this order — not the raw agent output:

1. A **provenance line**: lenses launched vs. returned, whether round 2 ran,
   what was read, how many findings were unique to a single lens. A degraded
   run says "single-context run" in plain words.
2. The **single risk most likely to kill the plan** — one short paragraph,
   before everything else.
3. The **register**, capped at 7 rows: failure mode, trigger, likelihood,
   impact, early warning, mitigation. The rest as one-line names.
4. At most 3 **tripwires** — early-warning signs to actively watch during
   execution, each paired with its pre-agreed response.
5. The **full text of every proposed edit**, shown before any checklist — a
   checklist label is never the only thing the user sees before an edit
   lands. Quote-and-diff only when a plan file exists; otherwise the amended
   passage stated in full.

Then ONE `AskUserQuestion` (`multiSelect: true`) holding at most the top 4
mitigations; everything below that stays in the writeup as proposals. Never
paginate across multiple calls — more than 4 worth offering means the ranking
isn't done. Name the ONE artifact you will edit, with its path, before
asking. Apply branches:

- **Plan mode active** — edit nothing but the plan file. Fold accepted
  mitigations into it and let ExitPlanMode carry them to approval. Never
  leave plan mode to apply premortem edits.
- **Plan already approved** — don't silently amend it; re-present the
  amended plan for approval.
- **No plan file** — "apply" means restating the amended plan in full,
  mitigations folded in.
- **No interactive user** (headless, hooks) — output the proposed edits as a
  diff and stop. Never auto-apply.

## Keep it honest

If the plan is genuinely solid, say so in two lines and stop — don't
manufacture risks to justify the ceremony. Name what the plan already gets
right, so the mitigations don't trample it. The point is a plan that
survives contact, not a longer document.
