---
name: doubt-driven-development
description: Adversarial fresh-context review of every non-trivial implementation decision. Before committing to an approach, explicitly ask what is wrong with it. Prevents confident mistakes.
license: MIT
compatibility: opencode
metadata:
  phase: build
  agent: build
---

# Doubt-Driven Development

## What I Do

I force a deliberate pause before committing to any non-trivial decision. I ask the adversarial question — "What is wrong with this?" — before the decision is locked in, not during code review two weeks later.

## When to Activate Me

Activate this skill for **every** decision that is:

- **Architectural**: choosing between patterns, structures, or system boundaries
- **Algorithmic**: picking a data structure, sort strategy, or traversal approach
- **Security-relevant**: authentication, authorization, input validation, secrets, file paths
- **Concurrent**: async patterns, race conditions, shared state, mutex/lock decisions
- **Irreversible**: schema migrations, API contracts, public interfaces
- **Novel**: using a library or pattern you haven't used in this codebase before

Skip for: variable names, file organization preferences, trivial function composition.

## The Doubt Protocol

### Step 1: State the decision clearly

Write it out in one sentence before challenging it:

```
Decision: I will use [X] to solve [problem] because [initial reason].
```

### Step 2: Generate the adversarial case

Ask with a fresh perspective: _"What is the best argument against this decision?"_

Think from the perspective of:

- **The maintainer 6 months from now** — will they understand why?
- **The adversarial user** — can this be abused or broken?
- **The load test** — does this hold under 10× expected volume?
- **The failure scenario** — what happens when the external dependency is down?
- **The refactoring engineer** — is this locked in, or can it change?

Write at least 2 concrete objections:

```
Objection 1: [specific problem this decision creates]
Objection 2: [specific failure mode or maintenance burden]
```

### Step 3: Evaluate the objections honestly

For each objection:

- Is it a real risk in this context, or theoretical?
- Does it disqualify the approach, or is it mitigatable?
- What would the mitigation cost?

### Step 4: Decide or pivot

**If objections are manageable:**

```
Decision confirmed: [X]
Mitigations:
- [Objection 1] → mitigated by [Y]
- [Objection 2] → accepted because [Z]
```

**If an objection is disqualifying:**

```
Decision revised: [X] → [X']
Reason: [The objection that changed the decision]
```

**If you can't resolve an objection:**

```
Escalate to Core: [decision + unresolved objection]
```

## Security-Specific Doubts

For any security-sensitive code, always ask these explicitly:

1. **Input**: Where does this data come from? Is it trusted? What happens if it's malicious?
2. **Output**: Where does this data go? Could it leak? Could it be interpreted as code (injection)?
3. **Auth**: Who is allowed to do this? Is this check consistent with other checks in the system?
4. **Secrets**: Is a secret ever logged, serialized, or passed in a URL parameter?
5. **Path**: If this is a file path, can it be traversed outside the intended directory?
6. **Error**: Do error messages reveal internal structure that an attacker could use?

## Concurrency-Specific Doubts

For async/concurrent code:

1. **Race condition**: Can two operations interleave to produce an invalid state?
2. **Ordering**: Is the ordering of operations guaranteed by the environment, or assumed?
3. **Cleanup**: If this throws, are resources (connections, locks, temp files) cleaned up?
4. **Re-entrancy**: Can this be called again before it finishes? What happens?

## Output Format

Include a "Decisions" section in the slice report:

```
## Decisions Made

### [Decision Name]
Chosen: [approach]
Objections considered:
- [Objection 1] → [resolution]
- [Objection 2] → [resolution]
Security review: [passed / flagged items]
```

## The Meta-Doubt

Apply doubt to this skill itself: the protocol exists to catch mistakes, not to make implementation slow. If you are applying it to a trivial naming choice, stop. The skill's trigger list is the filter. Respect it.
