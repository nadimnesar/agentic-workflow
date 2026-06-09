---
name: spec-driven-development
description: Write requirements and acceptance criteria before any code is written. Transforms an interview summary and chosen approach into a precise, testable spec that Plan and Build can execute against without ambiguity.
license: MIT
compatibility: opencode
metadata:
  phase: define
  agent: define
---

# Spec-Driven Development

## What I Do

I transform fuzzy requirements into a precise, written specification that acts as the contract between Define, Plan, Build, and Test. Nothing gets built without a spec. The spec is the source of truth.

## When to Use Me

Use me after `interview-me` and `idea-refine` have completed. I am the final step in the Define phase. Do not call me first — the interview and refinement feed me.

## The Spec Format

Every spec must follow this structure. No section is optional except "Open Questions" (which may be empty).

---

```markdown
# Spec: [Feature Name]

**Status:** Draft | Approved | Implemented
**Author:** [agent/user]
**Date:** [YYYY-MM-DD]

---

## 1. Problem Statement

[1–3 paragraphs. What exists today, what is broken or missing, and why it matters.
Measurable if possible: "currently X% of users fail at step Y."]

---

## 2. Goals

What this spec will achieve:

- [ ] Goal 1 — [measurable outcome]
- [ ] Goal 2 — [measurable outcome]

---

## 3. Non-Goals

What is explicitly out of scope (prevents scope creep and sets expectations):

- Not in scope: [X]
- Not in scope: [Y]
- Future work: [Z — acknowledge it exists, but not now]

---

## 4. Approach

[3–6 sentences. The chosen approach from `idea-refine`. Why this, not alternatives.
What changes and what stays the same. No implementation detail yet — that's Plan's job.]

---

## 5. Acceptance Criteria

Each criterion must be:

- **Verifiable**: a human or automated test can confirm it
- **Specific**: no "should work well" or "should be fast"
- **Behavioral**: describes observable output, not internal implementation

Format: `Given [precondition], when [action], then [observable outcome].`

### Happy Path

- [ ] AC1: Given [context], when [action], then [result].
- [ ] AC2: Given [context], when [action], then [result].

### Edge Cases

- [ ] AC3: Given [boundary condition], when [action], then [graceful result].
- [ ] AC4: Given [invalid input], when [action], then [specific error behavior].

### Non-Functional

- [ ] AC5: [Operation] completes in under [N]ms for [scale] inputs.
- [ ] AC6: [Feature] is accessible: keyboard navigable and screen-reader compatible.

---

## 6. Interface Changes (if any)

List any new or changed public interfaces:

- New route: `POST /api/[resource]` — [description]
- New component: `<ComponentName props={...}>` — [description]
- Changed function signature: `oldFn()` → `newFn(arg: Type): ReturnType`

---

## 7. Data Model Changes (if any)

Any schema, store, or data structure changes:
```

// Before
type Foo = { ... }

// After  
type Foo = { ..., newField: string }

```
Migration required: yes/no

---

## 8. Open Questions

Questions that must be answered before or during Plan/Build:
- [ ] Q1: [Question] — Owner: [agent/person]
- [ ] Q2: [Question] — Owner: [agent/person]
```

---

## Spec Quality Checklist

Before marking the spec as Approved, verify:

- [ ] Every AC is verifiable — a test can be written for it
- [ ] No AC describes internal implementation ("uses Redux", "calls the DB directly")
- [ ] Non-Goals explicitly exclude the most tempting scope additions
- [ ] Interface changes are listed — Plan needs them
- [ ] Open Questions are assigned owners
- [ ] The problem statement could convince a new engineer why this matters

## Rules

- An AC that says "should work correctly" is not an AC. Rewrite it.
- Ambiguous ACs are a gift to the person who wants to skip work. Make them unambiguous.
- If you can't write an AC for a goal, the goal isn't defined yet. Go back to `interview-me`.
- The spec is NOT a design doc. Don't include class diagrams or sequence diagrams unless essential.
- The spec may evolve, but changes require explicit review — not silent edits.
