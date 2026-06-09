---
name: debugging-and-error-recovery
description: Structured debugging protocol — reproduce, localize, fix, and guard. Never guess. Every fix is paired with a regression test. Every unexplained failure is fully understood before the fix is applied.
license: MIT
compatibility: opencode
metadata:
  phase: test
  agent: test
---

# Debugging and Error Recovery

## What I Do

I enforce a disciplined debugging process that finds the actual cause of failures — not the apparent cause. I prevent "fix it until it stops being red" in favor of "understand it, then fix it, then guarantee it stays fixed."

## The Four Steps

```
REPRODUCE → LOCALIZE → FIX → GUARD
```

Never skip steps. The most common debugging failure is jumping from "error message" to "fix" without localizing the actual root cause.

---

## Step 1: REPRODUCE

**Goal: Confirm you can consistently trigger the failure.**

A failure you can't reproduce reliably cannot be fixed reliably.

1. Run the failing test in isolation: `npx jest path/to/test.test.ts -t "test name"`
2. Confirm it fails consistently (run 3× if needed)
3. Confirm it fails on a clean state (not just in a dirty test environment)

**If the failure is intermittent:**

- Do not proceed — find what makes it intermittent first
- Common causes: timing dependencies, test ordering, shared mutable state, randomness
- Fix the flakiness before fixing the underlying bug

**If you can't reproduce:**

- Check if the test was already failing before your changes (`git stash`, run again)
- Check if it only fails in CI (environment differences, missing env vars, different Node version)
- Document "could not reproduce locally" and escalate to Core

---

## Step 2: LOCALIZE

**Goal: Find the exact line/function/state that causes the failure.**

Work from the symptom inward. Never start from a hypothesis about the cause.

### The Bisection Method

Start at the error message and work backward:

1. Read the full error, stack trace, and test name — understand what failed, not what you think failed
2. Find the earliest point in the call stack you own
3. Add a log or breakpoint at that point — confirm the inputs are what you expect
4. If inputs are wrong: move backward in the call stack
5. If inputs are right but output is wrong: move forward (the bug is in this function)
6. Repeat until you've isolated the exact line

### State of the World

Before looking at code, understand the state:

- What data is the test providing as input?
- What is the actual output vs expected output?
- What state changes happened before this call?

### Use the tools

- `console.log` or debugger statements at narrow checkpoints (remove them after)
- Git blame: when was this line last changed?
- Git log: what changed recently in this file?
- LSP: check if a type mismatch is the source of a runtime error

---

## Step 3: FIX

**Goal: Apply the minimal change that addresses the root cause.**

Once you know the root cause (not the symptom), apply the smallest fix that corrects it.

### What "minimal" means

- If the bug is a missing null check: add the null check at the right layer (caller? callee? both?)
- If the bug is wrong logic: fix the logic — don't work around it with a special case
- If the bug is a misunderstood API: fix the usage — update the call to match the actual API

### What it does NOT mean

- "Minimal" does not mean "hack the test to pass"
- "Minimal" does not mean "suppress the error message"
- "Minimal" does not mean "add a try/catch that swallows the exception"

### Before applying the fix

- Confirm that your understanding of the root cause predicts the failure
  - "If X is the cause, then changing Y should make the test pass AND Z test should still pass"
- Apply the fix
- Run the failing test: confirm it now passes
- Run the full test suite: confirm nothing else broke

### If the fix is complex

If the fix touches more than 2–3 files or changes more than a single logical thing:

- Stop
- Break it into smaller steps
- Each step should keep the suite green

---

## Step 4: GUARD

**Goal: Ensure this bug cannot silently reappear.**

A bug that was fixed without a regression test is a bug that will return.

After every fix, ask: _"What test, if it had existed, would have caught this before it reached me?"_

Write that test. It should:

- Reproduce the exact failing condition (same input, same state)
- Assert the correct behavior (not just "doesn't throw")
- Be named to describe the bug: "does not return null when cache is empty"

If a test already existed and didn't catch this:

- Understand why (was the assertion too weak? was the case not covered?)
- Strengthen the existing test, or add a new one
- A test that doesn't catch the bug it was written for is not a guard

---

## Error Message Reading Guide

**Null / undefined errors:**

- The call stack tells you where the access happened
- The fix is upstream: find where null enters and handle or reject it there

**Type errors at runtime (in JS/TS):**

- Find where the type contract is broken (caller or callee)
- Fix at the source; don't cast to `any`

**Async/Promise rejection unhandled:**

- Find the promise that threw — it's in the stack trace
- Add proper error handling; don't silence with `catch(() => {})`

**Test assertion failures:**

- Read "expected: X, received: Y" carefully
- Is Y a completely wrong value? (wrong logic)
- Is Y "undefined"? (function not returning correctly, or test setup incomplete)
- Is Y an object reference mismatch? (deep equality vs. reference equality)

**Timeout failures:**

- The async operation didn't complete — find why (network, infinite loop, missing resolve/reject)
- Use `await` correctly; check that your async test is properly `await`-ing

---

## Output: Debug Report

```markdown
## Debug Report: [failing test/behavior name]

Reproduced: yes / intermittent (flakiness fixed first)
Root cause: [precise description of what was wrong and where]
Fix applied: [what changed, in what file(s), and why]
Regression test added: [test name] in [file]
All tests passing: yes
```
