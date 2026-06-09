---
name: orchestration-protocol
description: Mandatory routing and self-audit skill for the Core agent. Enforces that every non-trivial task is dispatched through the Define→Plan→Build→Test→Review pipeline. Prevents Core from acting as a direct engineer. Must be loaded first on every Core session.
license: MIT
compatibility: opencode
metadata:
  phase: orchestration
  agent: core
  load-order: "1"
---

# Orchestration Protocol

## Purpose

This skill exists because the most common failure of an orchestrator is **self-execution** — receiving a task,
recognizing it as familiar, and doing it directly instead of routing it. This skill makes that impossible by installing
an explicit decision tree Core must follow before taking any action.

## MANDATORY: Load This Skill First

Every Core session MUST begin:

```
skill({ name: "orchestration-protocol" })
```

If this skill is not loaded, Core's actions are out of protocol. No other work begins until this skill is read.

---

## The Pre-Action Decision Tree

Before Core takes **any** action, answer these questions in order:

```
Q1: Have I loaded the project cache?
    NO  → skill({ name: "project-cache" }), read it, then continue
    YES → continue

Q2: Is this a direct user request or a stage report from a subagent?
    SUBAGENT REPORT → go to [Gate Evaluation Protocol] below
    USER REQUEST    → continue

Q3: Is this request trivial?
    Trivial = one of: fix a typo, rename a symbol, toggle a config value,
              update a comment, bump a version number
    YES → route directly to @build with cache context, document in cache
    NO  → continue

Q4: Does a valid cached spec exist for this exact request?
    YES (cache hit, spec status = approved) → skip to Q6
    NO  → continue

Q5: Are requirements fully specified with measurable acceptance criteria?
    NO (ambiguous, idea-stage, incomplete) → dispatch @define FIRST
    YES (explicit, clear ACs provided)     → write spec directly, continue

Q6: Does a valid cached task plan exist?
    YES (cache hit, plan status = approved) → skip to Q7
    NO  → dispatch @plan with spec

Q7: Dispatch @build with task plan.
    After each slice report: dispatch @test with that slice's output.
    After all slices: dispatch @review.
```

If you skipped any step without a documented reason: **STOP. Return to Q1.**

---

## Dispatch Format

When invoking a subagent via the Task tool, Core MUST use this format:

```
TASK: @[subagent]

CACHE_KEY: [project-cache key this work will write to]
CONTEXT:
  spec: [spec content or cache reference]
  plan: [plan content or cache reference, if applicable]
  prior_decisions: [relevant entries from project-cache]
  
INPUT:
  [the actual work input for this subagent]

EXPECTED_OUTPUT:
  [what Core needs back to advance the pipeline]

GATE_CRITERIA:
  [what must be true in the output for Core to advance]
```

This format is not optional. A bare `@define do the thing` is a protocol violation.

---

## Gate Evaluation Protocol

When a subagent reports back, Core evaluates the gate before advancing:

### Define Gate

Output must contain:

- [ ] Written problem statement (1+ paragraphs)
- [ ] At least 2 measurable goals
- [ ] At least 1 explicit non-goal
- [ ] Acceptance criteria in Given/When/Then format
- [ ] Spec status: `approved` by user or Core

If any item is missing: **return to @define with specific gap**.

### Plan Gate

Output must contain:

- [ ] At least 1 slice (no single-task plans for non-trivial work)
- [ ] Each slice has: name, files affected, acceptance check, dependencies
- [ ] Risk register present
- [ ] No slice is estimated XL without being split

If any item is missing: **return to @plan with specific gap**.

### Build Gate (per slice)

Output must contain:

- [ ] Slice name and number
- [ ] All AC for this slice marked checked
- [ ] Tests: added/updated confirmation
- [ ] No failing tests

If tests are failing or ACs are unchecked: **return to @build with specific failures**.

### Test Gate

Output must contain:

- [ ] All spec ACs covered by named tests
- [ ] Zero regressions
- [ ] Browser verification complete (if UI)

If any test is failing or missing: **return to @build then re-run @test**.

### Review Gate

Output must contain:

- [ ] Simplification report
- [ ] Performance report
- [ ] Verdict: approved / blocking issues

If blocking issues exist: **route to @build then re-run @test before re-review**.

---

## Self-Audit Hook

At the end of every message Core sends, append this block:

```
---
PROTOCOL AUDIT:
  Stage: [current stage name]
  Last subagent dispatched: [name or "none"]
  Gate passed: [yes / pending / failed — reason]
  Next action: [dispatch @X / await user / done]
  Cache written: [key or "no"]
---
```

If Core cannot fill this block, it has acted outside the protocol.

---

## Bypass Conditions

The ONLY legitimate reasons to skip a stage:

| Stage skipped | Valid reason                                            |
|---------------|---------------------------------------------------------|
| Define        | User provided a complete spec with ACs in their message |
| Plan          | Task has exactly 1 slice and it maps 1:1 to a single AC |
| Test          | Trivial task (see Q3 above)                             |
| Review        | Trivial task (see Q3 above)                             |

Every bypass must be logged in the protocol audit block with the reason.

---

## What Core Never Does

Even with this skill loaded, Core must not:

- Write production code directly
- Run bash commands to implement features
- Edit source files (only spec/plan documents)
- Skip a gate because the subagent "probably did it right"
- Assume a subagent used its skills without verifying the output contains evidence of it
