---
name: idea-refine
description: Refine ideas through structured divergent and convergent thinking. Generates multiple approaches, evaluates trade-offs explicitly, and converges on the most viable path before committing to a spec.
license: MIT
compatibility: opencode
metadata:
  phase: define
  agent: define
---

# Idea Refine

## What I Do

I prevent premature convergence on the first plausible solution. Before committing to an approach, I force explicit exploration of the solution space — then converge with a documented rationale.

## When to Use Me

Use me after `interview-me` has established the problem, and before `spec-driven-development` writes the spec. Especially important when:

- The problem has multiple non-obvious solutions
- Trade-offs exist between simplicity, performance, maintainability, or time-to-build
- You or the user are attached to a specific solution that hasn't been challenged
- The problem involves a new domain, library, or architecture pattern

## The Refinement Protocol

### Phase 1: Problem Restatement

Before generating options, restate the problem in one sentence that everyone agrees on. This is the anchor. If you can't state it in one sentence, the interview isn't done.

```
Problem: [one sentence]
Constraints (hard): [list]
Constraints (soft/preferred): [list]
```

### Phase 2: Diverge — Generate Options (3 minimum)

Generate at least 3 meaningfully different approaches. "Meaningfully different" means different trade-offs, not just different libraries for the same approach.

For each option:

```
### Option N: [Name]
**Summary:** [1 sentence — what it does]
**How it works:** [2–4 sentences]
**Strengths:**
- ...
**Weaknesses:**
- ...
**Time to build:** [rough estimate]
**Risk:** [what could go wrong]
```

Anti-pattern: Do not generate "Option 1: The correct approach. Option 2: A bad version. Option 3: A worse version." Make each option genuinely defensible.

### Phase 3: Evaluate Against Constraints

Build a comparison table:

| Criterion              | Option 1 | Option 2 | Option 3 |
| ---------------------- | -------- | -------- | -------- |
| Meets hard constraints | ✓/✗      | ✓/✗      | ✓/✗      |
| Simplest to build      | ★★★      | ★★       | ★        |
| Simplest to maintain   | ...      | ...      | ...      |
| Performance at scale   | ...      | ...      | ...      |
| Reversibility          | ...      | ...      | ...      |

Immediately eliminate any option that fails a hard constraint.

### Phase 4: Converge — Choose and Justify

Pick one option. State why. Be explicit about what you are trading away:

```
## Chosen Approach: [Name]

**Why this one:**
[2–4 sentences that would convince a skeptical senior engineer]

**What we're trading away:**
- We are not getting [strength of Option X] because [reason]
- We are accepting [weakness of chosen option] because [rationale]

**Assumptions this choice depends on:**
- [If X is true about the system/team/constraints, this holds]
```

### Phase 5: Challenge the Choice

Before locking in, ask: _"What would have to be true for this to be the wrong choice?"_

Document the answer. If any of those conditions might actually be true, revisit.

## Output

Pass the "Chosen Approach" section to `spec-driven-development` as the approach definition.

## Rules

- Never skip to Phase 4 from Phase 1. The divergence step is not optional.
- "We'll just use [technology X]" is a solution, not a justification. Justify it.
- If the user strongly prefers an option, still generate alternatives. If their preferred option wins the comparison, great — now everyone knows why.
- "It depends" is not a convergence answer. Pick one and state the conditions.
