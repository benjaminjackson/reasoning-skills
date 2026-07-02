---
name: atom-of-thought
description: Use when facing a complex, multi-part problem that would benefit from being broken into independent and dependent sub-questions before answering, rather than reasoned about as one tangled whole. Trigger on "atom of thought", "AoT", "break this down systematically", "decompose this problem", "atomic decomposition", or any complex analytical, planning, or reasoning task where a direct one-shot answer risks missing a sub-part or an implicit dependency between parts.
---

When facing a complex problem, use this approach:

## Step 1: Direct Assessment
Attempt to solve the original problem directly to establish a baseline understanding.

## Step 2: Atomic Decomposition
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

## Step 3: Parallel Resolution
Solve all independent atomic questions first, in parallel:

```
Independent Resolutions:
Q1: [Question] → [Answer + Evidence]
Q2: [Question] → [Answer + Evidence]
...
```

## Step 4: Dependency Resolution  
Solve dependent questions using resolved atoms as known conditions:

```
Dependent Resolutions:
Q3 (needs Q1, Q2): Given [Q1 answer] and [Q2 answer], [modified question] → [Answer]
...
```

## Step 5: Contraction & Integration
Reformulate the original problem using all resolved atoms as known conditions:

```
Contracted Problem: Given [all resolved atoms], [simplified original question]
Final Answer: [Integrated solution]
```

## Step 6: Ensemble Validation
Compare direct solution vs. AoT solution:
- Which approach provides more confidence?
- Are results consistent?
- Final decision with reasoning

## Meta-Guidelines:
- **Markov Property**: Each step depends only on current state, not history
- **Iterative Depth**: If contracted problem still complex, repeat AoT process  
- **Atomic Validation**: Ensure each atom is truly independent or has clear dependencies
- **Efficiency Focus**: Avoid redundant reasoning paths

This framework transforms complex reasoning into manageable atomic operations while maintaining logical dependencies.
