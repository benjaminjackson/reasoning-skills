# reasoning-skills

A [Claude Code plugin marketplace](https://docs.claude.com/en/docs/claude-code/plugins) with one plugin, **reasoning**, shipping four skills: **atom-of-thought**, which decomposes complex problems before answering; **confess**, which audits Claude's own work afterward; **postmortem**, which turns a finished session into a root-cause analysis and proposed fixes; and **premortem**, which stress-tests a plan before execution by imagining it already failed.

## Installation

```
claude plugin marketplace add benjaminjackson/reasoning-skills
claude plugin install reasoning@reasoning-skills
```

## atom-of-thought

Helps Claude catch the sub-parts and hidden dependencies that a one-shot answer misses, by decomposing a multi-part problem into sub-questions, solving them in dependency order, and recombining them into one answer, checked at the end against a direct, one-shot attempt.

Each step needs only the current state, not the full history, so you never have to hold the whole problem in your head at once.

The skill is based on ["Atom of Thoughts for Markov LLM Test-Time Scaling"](#paper) (Teng et al., NeurIPS 2025), adapted here from a test-time-scaling technique into a prompting skill Claude runs during a conversation.

### Usage

- **Explicitly**, as a slash command:
  ```
  /reasoning:atom-of-thought <your problem>
  ```
- **Automatically**, when Claude judges a prompt warrants it: phrases like "atom of thought," "AoT," "break this down systematically," "decompose this problem," or "atomic decomposition" will trigger it, as will any tangled analytical, planning, or reasoning task.

It's built for problems with structure to decompose: multiple sub-questions, some of which depend on others. For a single-part question it's the wrong tool: you'll get the overhead of decomposition with nothing to decompose.

### What to expect

Output is longer and more structured than a normal reply. You'll see six sections, roughly in this shape:

```
## Step 1: Direct Assessment
[a quick baseline answer]

## Step 2: Atomic Decomposition
Q1: ... — DEPENDENCIES: none
Q2: ... — DEPENDENCIES: none
Q3: ... — DEPENDENCIES: [Q1, Q2]
Completeness check: ...

## Step 3: Parallel Resolution
Q1 → ...
Q2 → ...

## Step 4: Dependency Resolution
Q3 (needs Q1, Q2): ...

## Step 5: Contraction & Integration
Contracted Problem: ...
Final Answer: ...

## Step 6: Ensemble Validation
[direct vs. decomposed answer, and which one wins]
```

Expect a trade: more tokens and more time per answer, for fewer missed sub-parts and dependencies on complex problems. On easy ones, that trade isn't worth it, which is why the skill triggers only on complex, multi-part problems.

### Paper

Fei Teng, Quan Shi, Zhaoyang Yu, Jiayi Zhang, Yuyu Luo, Chenglin Wu, Zhijiang Guo.
**"Atom of Thoughts for Markov LLM Test-Time Scaling."** NeurIPS 2025.
[arXiv:2502.12018](https://arxiv.org/abs/2502.12018) · [official code](https://github.com/qixucen/atom)

The paper's core move: existing test-time scaling methods burn extra compute re-processing the whole history at every step. The authors instead treat reasoning as a Markov process, where each step depends only on the current state, not the full history, and scale the approach with tree search and reflective refinement. Doing so surfaces an atomic structure: reasoning decomposes cleanly into self-contained, low-complexity units called Atom of Thoughts (AoT). In their experiments, AoT consistently beats baselines as compute budget grows, and composes with reasoning and non-reasoning LLMs.

This skill is a prompting-level adaptation of that idea, not a port of the paper's code. The original is a multi-agent, tool-driven pipeline; it's the same decompose → resolve → contract shape, run as instructions for Claude in a single conversation.

## confess

A self-audit of work Claude just finished (not a re-check of whether the work is correct, but a report on how confident Claude is in it), so shortcuts and unverified assumptions surface before they cause problems. When triggered, Claude reports:

1. A confidence score (1-10) per component, with reasons.
2. Every shortcut or "good enough" call it made, why, and the risk.
3. Every assumption it made without verifying, and how to verify it.
4. Issues it noticed but didn't handle.
5. What would raise its confidence, as a concrete checklist.
6. The weakest part of the work, named plainly.

Once you run it, the work's quality is already locked in, so there's nothing to gain by hedging: a thorough confession is the whole point.

### Usage

- **Explicitly**, as a slash command, right after Claude finishes a task:
  ```
  /reasoning:confess
  ```
- **Automatically**, on phrases like "confess," "be honest about this," "what are you unsure about," "what did you cut corners on," "what's the weakest part," or "audit your own work."

### What to expect

A confidence table, then plain-text lists for shortcuts, unverified assumptions, and unhandled issues, plus a confidence-boosting checklist, and it names the weakest part. No code or file changes: it's a report, not a fix. If it comes back empty, push back and ask Claude to look harder. That's not proof the work is flawless.

### Paper

Manas Joglekar, Jeremy Chen, Gabriel Wu, Jason Yosinski, Jasmine Wang, Boaz Barak, Amelia Glaese.
**"Training LLMs for Honesty via Confessions."** OpenAI, 2025.
[arXiv:2512.08093](https://arxiv.org/abs/2512.08093) · [blog post](https://openai.com/index/how-confessions-can-keep-language-models-honest/)

The paper's core move: have the model produce a second output after its main response — a "confession" — and reward that confession on whether it's truthful, independent of how well the main response scored. Once honesty is the only way to maximize the confession's reward, it becomes the dominant strategy regardless of how the main response turned out. Trained into GPT-5-Thinking, this reward scheme got the model to reliably own up to hallucinations, rule-breaking, and reward hacking in its confessions even when the main response buried or denied them.

This skill is a prompting-level adaptation of that idea, not the RL training. The paper trains honesty in with a separate reward signal; this skill runs the same after-the-fact, separately-judged confession as a self-audit prompt inside a conversation.

## postmortem

An engineering postmortem on a just-finished task or session: what was learned, where time was wasted, and what would make it go faster next time. Claude reconstructs a chronological account of the session (every user correction and wasted round-trip included), hands the facts to one well-briefed reviewer subagent, and relays back:

1. A root cause, driven to ground with a 5-Whys chain, separated from contributing paper-cuts.
2. A minimum-round-trip replay — the ideal sequence that reaches the same end state in the fewest actions.
3. Proposed before→after edits to the skill, config, or code the friction lives in, each tied to a moment in the timeline.

It ends with a checklist of the proposed fixes — pick the ones you want and Claude applies just those; the rest stay as proposals in the writeup.

### Usage

- **Explicitly**, as a slash command, after finishing (or abandoning) a piece of work:
  ```
  /reasoning:postmortem
  ```
- **Automatically**, on phrases like "postmortem," "retro," "what did we learn," "how could that have gone better," or "how do we do this more efficiently next time."

### What to expect

One reviewer subagent, not a panel — multiple review formats converge on the same findings, so the budget goes into the brief instead. Output is the root cause, the replay, and the proposed edits, ending with the single highest-leverage fix. Effort scales with the session: a ten-minute task gets a paragraph, not a subagent.

## premortem

A premortem on a plan that hasn't been executed yet: imagine it already failed, write the history of that failure, and harden the plan against what turns up. Claude first decides the shape — a small plan gets a one-paragraph premortem inline, no agents. For a plan that's expensive, wide, or hard to reverse, it launches three read-only agents in parallel (technical, assumptions, and process lenses), each writing its failure history blind to the others, then cross-examines: each agent attacks, ranks, and extends the merged list. You get back:

1. A provenance line — what actually ran, so a degraded run can't dress up as a full one.
2. The single risk most likely to kill the plan, first.
3. A ranked failure register (capped at 7 rows) and up to 3 tripwires to watch during execution.
4. Proposed plan edits, shown in full, then offered as a single checklist — pick the ones you want and Claude folds just those into the plan.

### Usage

- **Explicitly**, as a slash command, with a plan on the table:
  ```
  /reasoning:premortem
  ```
- **Automatically**, on phrases like "premortem," "pre-mortem," "poke holes in this plan," "imagine this failed," or "stress-test this plan." It requires an actual written plan plus an explicit ask — a rhetorical "what could go wrong?" won't (and shouldn't) fire it.

### What to expect

The blind first pass buys de-anchoring, not true independence — the agents share a model and a brief, so the skill treats agreement between lenses as one opinion stated three times, never as corroboration. Finished work is the postmortem's job, not this one's. Effort scales with stakes: the team only launches when the plan is hard to reverse and costs more than the premortem does.

### Sources

Gary Klein. **"Performing a Project Premortem."** Harvard Business Review, September 2007.
[hbr.org](https://hbr.org/2007/09/performing-a-project-premortem)

Deborah J. Mitchell, J. Edward Russo, Nancy Pennington. **"Back to the future: Temporal perspective in the explanation of events."** Journal of Behavioral Decision Making, 1989.

The 1989 study found that prospective hindsight — imagining an outcome as if it had already occurred — increased the number and specificity of reasons participants generated for it; Klein's HBR article summarizes this as a ~30% improvement in identifying reasons for future outcomes, and his premortem turns it into a team exercise. This skill is a prompting-level adaptation, not a reproduction: it keeps prospective hindsight and the blind-then-share ordering, but a team of Claude agents can't reproduce a real room's diversity of information, and the technique's empirical support is thinner than the papers behind atom-of-thought and confess — it's a procedure, not a validated result.

## Author

Benjamin Jackson ([@benjaminjackson](https://github.com/benjaminjackson))
