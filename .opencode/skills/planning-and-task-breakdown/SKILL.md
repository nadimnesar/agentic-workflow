---
name: planning-and-task-breakdown
description: Decompose an approved spec into small, independently verifiable tasks ordered for maximum parallelism and minimum risk. Each task is a thin vertical slice that delivers testable behavior.
license: MIT
compatibility: opencode
metadata:
  phase: plan
  agent: plan
---

# Planning and Task Breakdown

## What I Do

I decompose a spec into the smallest tasks that still deliver working, testable behavior. I turn "build a feature" into a sequenced list of verifiable steps that Build can execute without ambiguity.

## When to Use Me

Use me after Define has produced an approved spec with acceptance criteria. I am the sole skill of the Plan agent.

## What Makes a Good Task

A task is good when:
- **It fits in a session.** An engineer could start and finish it in one focused sitting.
- **It is vertical.** It delivers end-to-end behavior — not "just the model", not "just the view". The thinnest possible slice through all layers.
- **It is verifiable.** There is a concrete way to confirm it is done — a test passes, a UI renders, a command outputs the right value.
- **It is independent.** It does not depend on another task being "almost done."

A task is bad when:
- It says "implement [big thing]" without scoping a specific behavior
- It is purely horizontal ("write all the database models")
- Its only acceptance check is "the code compiles"
- It cannot be demonstrated without N other tasks being complete

## The Decomposition Protocol

### Step 1: Map ACs to Work Units

For every acceptance criterion in the spec, identify the minimal work that makes it pass. Group related ACs when they naturally belong to the same slice.

### Step 2: Identify the Walking Skeleton

Before any other task: what is the smallest end-to-end path through the system that proves the architecture works? This is always Slice 1. It should be a stub — real enough to exercise the full stack, thin enough to be done in an hour.

### Step 3: Order by Risk and Dependency

- High-risk tasks (external integrations, new data models, auth) come early
- Tasks that block others come before the tasks they block
- Optional enhancements come last
- Never order tasks so that Slice N is "all the hard parts"

### Step 4: Write Each Task

```markdown
## Slice N: [Name — what behavior it delivers]

**Delivers:** [One sentence — the user-observable or test-observable outcome]
**AC covered:** [Which spec AC(s) this satisfies]
**Files likely affected:** [list — Plan's best guess, Build will adjust]
**Acceptance check:**
  - [ ] [Specific, runnable verification]
  - [ ] [Specific, runnable verification]
**Dependencies:** [Slice numbers that must be complete first]
**Estimated complexity:** S / M / L
**Notes:** [Anything Build needs to know: edge cases, existing patterns to follow, pitfalls]
```

### Step 5: Risk Register

For every meaningful risk identified during planning:

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| External API might have different schema | Medium | High | Verify in Slice 1 before building against it |
| Migration may require downtime | Low | High | Wrap in feature flag |

### Step 6: Open Technical Questions

Questions that Build must answer before or during a specific slice:
- [ ] Q1: [Question] — blocks: [Slice N]
- [ ] Q2: [Question] — blocks: [Slice N]

## Task Sizing Guide

| Size | Time | Characteristics |
|------|------|-----------------|
| S (Small) | < 30 min | Single function, config change, single component |
| M (Medium) | 30–90 min | Feature slice with 2–3 files, one integration point |
| L (Large) | 90–180 min | Multiple interacting systems; should be split if possible |
| XL | > 3 hrs | **Always split.** If you can't split it, the spec needs more definition. |

## Anti-Patterns to Avoid

- **"Backend slice / Frontend slice" split**: This creates integration problems at the seam. Build vertically.
- **"Refactor first" slice**: Unless the refactor is required to do the feature, it belongs in Review, not Plan.
- **Catch-all slice**: "Handle edge cases" is not a slice. Name the edge cases.
- **Sequential by layer**: "DB → service → API → UI" creates 4 partial things before anything works. Build the full path thinly first.

## Output Format

```markdown
# Task Plan: [Feature Name]
**Spec:** [link or summary]
**Total slices:** N
**Estimated total:** [sum of estimates]

---

[Slice 1]
[Slice 2]
...

---

## Risk Register
[table]

## Open Technical Questions
[list]

## Notes for Build
[anything that doesn't fit in individual slices]
```
