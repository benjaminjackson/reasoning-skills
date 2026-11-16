---
name: atom-of-thought
description: Default reasoning framework for any question with more than one moving part. Use it by default, without being asked, whenever a question, decision, diagnosis, plan, comparison, trade-off, estimate, or "why/how/should" question could be split into separate sub-questions, especially when some answers depend on others. Trigger on any request to reason carefully, even when atom of thought is never named, including "think it through", "think this through", "think through", "think carefully", "think hard", "step by step", "reason through", "work through", "walk me through", "talk me through", "figure out", "weigh", "pros and cons", "help me decide", "what am I missing", "break down", "break this down", "unpack", "untangle", "decompose", "atom of thought", "AoT", or "atomic decomposition". When unsure, use it; skip it only for single-fact lookups, one-step edits, or casual chat.
---

When facing a complex problem, use this approach:

## Step 1: Atomic Decomposition
Discover sub-questions in four passes — don't target a count:

**Pass 1 — Generate freely:** List every conceptually separable question embedded in the problem. Don't filter yet.

**Pass 2 — Atomize:** For each question, ask "does this embed multiple independent questions?" If yes, split it. Keep splitting until each one is indivisible.

**Pass 3 — Mark dependencies:** A dependency is only information that must come from a *previous sub-question's answer* — not from the problem statement itself. Don't mark known things as dependencies.

**Pass 4 — Completeness & prune:**
- Completeness: "If I resolved every question on this list, would the original problem be fully solved?" If not, add what's missing.
- Prune: Remove any sub-question whose answer is already explicit in the problem statement.

**Decomposition Template:**
```
Main Problem: [Original question/task]

Atomic Sub-questions:
Q1: [Question] — DEPENDENCIES: none
Q2: [Question] — DEPENDENCIES: none
...
Qn: [Question] — DEPENDENCIES: [Q1, Q3]

Completeness check: Resolving all Qs above fully answers the original problem? [Yes / No — if No, add missing atoms]

Dependency Graph: [Visualize relationships]
```

## Step 2: Parallel Resolution
Solve all independent atomic questions first, in parallel:

```
Independent Resolutions:
Q1: [Question] → [Answer + Evidence]
Q2: [Question] → [Answer + Evidence]
...
```

## Step 3: Dependency Resolution
Solve dependent questions using resolved atoms as known conditions:

```
Dependent Resolutions:
Q3 (needs Q1, Q2): Given [Q1 answer] and [Q2 answer], [modified question] → [Answer]
...
```

## Step 4: Contraction & Integration
Reformulate the original problem using all resolved atoms as known conditions:

```
Contracted Problem: Given [all resolved atoms], [simplified original question]
Final Answer: [Integrated solution]
```

## Meta-Guidelines:
- **Markov Property**: Each step depends only on current state, not history
- **Iterative Depth**: If contracted problem still complex, repeat AoT process  
- **Atomic Validation**: Ensure each atom is truly independent or has clear dependencies
- **Efficiency Focus**: Avoid redundant reasoning paths

This framework transforms complex reasoning into manageable atomic operations while maintaining logical dependencies.
