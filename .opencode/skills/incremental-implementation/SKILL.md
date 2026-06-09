---
name: incremental-implementation
description: Build in thin vertical slices — implement the minimum that makes a test pass, verify it works end-to-end, then expand. Never accumulate unverified changes across multiple slices.
license: MIT
compatibility: opencode
metadata:
  phase: build
  agent: build
---

# Incremental Implementation

## What I Do

I enforce a discipline of building and verifying in small, committed increments. The goal is that at every step, the codebase is in a working state — no half-implemented features, no "it'll work once X is done."

## Core Principle

**The red-green-commit loop:**

```
1. RED    → Write a test (or identify the AC) that currently fails
2. GREEN  → Write the minimum code to make it pass
3. COMMIT → Commit the working state
4. EXPAND → Move to the next smallest increment
```

Never skip to step 4 without completing steps 1–3.

## The Slice Protocol

### Before Starting a Slice

1. Read the slice definition from the task plan
2. Read every file you will modify — understand the existing patterns
3. Identify the acceptance check for this slice
4. Write or locate the test that will verify it

### During a Slice

**The walking skeleton first.**
For the very first slice, implement the end-to-end path in the thinnest possible way:

- Stub responses are fine
- Hardcoded values are fine temporarily
- The goal is: the full path works, even if shallowly

**Make the test pass — nothing more.**
Do not add "while I'm here" improvements. Do not implement the next slice's functionality because it's convenient. Stay in the slice.

**Verify at every meaningful checkpoint.**
After each function or component is written:

- Run the relevant test
- If it fails: fix it now, not later
- Never move forward with a red test except to write the next test in a TDD cycle

### After Completing a Slice

Before moving to the next slice, confirm:

- [ ] The slice's acceptance check passes
- [ ] No previously-passing tests are broken
- [ ] The code is committed (even if on a feature branch)
- [ ] The diff is reviewable — no unrelated changes mixed in

## Scope Discipline

The three most common scope violations during implementation:

**1. The "obvious improvement"**
You notice something adjacent that should be fixed. Write it down as a note to Review. Do not fix it now — it adds unreviewed changes to the current slice.

**2. The "it'll be needed anyway"**
You implement future slice functionality while in the current slice. Don't. The future slice might not exist after Test runs.

**3. The "one more thing"**
Adding a feature that wasn't in the spec because it seems small. Stop. Spec changes go through Core.

## Commit Discipline

Each commit should be:

- A single logical change
- Green (all tests pass)
- Describable in one sentence

Commit message format:

```
[slice-N] verb: what changed

- Why (if not obvious)
- Any caveats
```

Example:

```
[slice-1] feat: add user authentication endpoint

- Returns JWT on valid credentials
- Returns 401 with generic message on invalid (no enumeration)
```

## When to Stop and Escalate

Stop and escalate to Core (do not continue) if:

- The slice is larger than estimated and can't be split further without changing the spec
- You've discovered the task plan is wrong (wrong order, missing dependency)
- A test reveals a spec ambiguity — two valid interpretations exist
- You've been on one slice for more than 3× the estimated time

## Output After Each Slice

```
Slice N complete.
AC checked: [list]
Tests: [added/modified/all passing]
Committed: [sha or "yes"]
Ready for next slice: [yes / no + reason]
```
