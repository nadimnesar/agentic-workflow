---
name: interview-me
description: Surface what the user actually wants before any plan, spec, or code exists. Use this skill to run a structured requirements interview — asking targeted questions to uncover the real problem, constraints, and definition of done.
license: MIT
compatibility: opencode
metadata:
  phase: define
  agent: define
---

# Interview Me

## What I Do

I guide a structured interview to surface the user's true intent before any solution work begins. I prevent the most common failure mode in software: building the right thing wrong, or the wrong thing entirely.

## When to Use Me

Use me at the very start of any non-trivial request where:
- The user has described a solution but not the problem
- The scope is unclear or potentially unbounded
- Multiple interpretations of the request are possible
- You sense a disconnect between what was asked and what is actually needed

Do not use me for trivial, well-scoped tasks ("fix this typo", "rename this variable").

## The Interview Protocol

### Phase 1: The Real Problem (1–3 questions)

Start with the problem, not the solution. Never ask "how do you want it built?"

Questions to choose from (pick the 1–2 most relevant):
- "What specific situation or pain are you trying to fix? Walk me through what happens today."
- "If this feature didn't exist six months from now, what would go wrong?"
- "You mentioned [X]. What made you reach for [X] specifically — is there another way you've tried to solve this?"
- "Who experiences this problem, and how often?"

Wait for the answer before continuing.

### Phase 2: Constraints and Context (1–3 questions)

- "What are the hard constraints? (deadline, existing stack, team size, compliance requirements)"
- "What is the worst thing I could build that technically satisfies your request?"
- "Is there anything that must not change?"
- "What does your current [code/system/process] look like in this area?"

### Phase 3: Definition of Done (1–2 questions)

- "How will you know this is working correctly? What would you test?"
- "What does success look like in one sentence?"
- "If you demo this to a stakeholder, what do they need to see?"

### Phase 4: Out of Scope (1 question)

Always ask this — it prevents scope creep:
- "What is explicitly NOT part of this request?"

## Interview Rules

1. **Ask at most 5 questions per round.** Prioritize ruthlessly.
2. **Never ask a leading question.** "You probably want X, right?" is leading.
3. **Never assume a technical solution.** "So you want a REST API that..." — stop, that's skipping ahead.
4. **Paraphrase back before moving on.** "So if I understand: [restate in your words]. Is that right?"
5. **One topic at a time.** Don't bundle three questions in one message.
6. **Stop when you have enough to write a spec.** Don't interview forever.

## Output

At the end of the interview, produce a structured summary:

```
## Interview Summary

**Real problem:** [1–2 sentences]
**Affected users/systems:** [who/what]
**Constraints:** [list]
**Definition of done:** [what success looks like]
**Out of scope:** [explicit exclusions]
**Unresolved:** [anything still unclear]
```

Hand this to `idea-refine` next.
