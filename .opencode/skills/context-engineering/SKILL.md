---
name: context-engineering
description: Load the right context at the right time. Keep the working context window lean, precise, and relevant to the current slice — not a dump of the whole codebase.
license: MIT
compatibility: opencode
metadata:
  phase: build
  agent: build
---

# Context Engineering

## What I Do

I prevent context pollution — the state where an agent is loaded with so much irrelevant information that it makes poor decisions, hallucinates connections, or misses what matters. I make context deliberate, not accidental.

## Core Principle

> Context is not free. Every irrelevant file in context is noise that competes with the signal you need. Load only what matters for the current slice.

## The Context Layers

Think of context in concentric layers, loaded only as needed:

```
Layer 1 (Always): Spec + current slice task
Layer 2 (Always): Files being directly modified
Layer 3 (Usually): Types/interfaces those files depend on
Layer 4 (Often): Tests for the area being changed
Layer 5 (As needed): Related patterns from similar existing code
Layer 6 (Rarely): Full module; only if truly cross-cutting
```

Never load Layer 6 when Layer 2 is enough.

## The Pre-Slice Context Checklist

Before starting a slice, explicitly answer:

**What is in scope for this slice?**

```
Current slice: [name]
Direct files: [list — files I will modify]
Dependency files: [types, interfaces, utilities I must read]
Test files: [tests for the changed area]
Pattern reference: [1–2 examples of similar existing code, if needed]
```

**What is NOT loaded (and why it doesn't need to be)?**

```
Not loaded:
- [file.ts] — unrelated module
- [bigService.ts] — only need the interface, not the implementation
```

## Context Loading Strategies

### Strategy 1: Interface-First

When you need to use a module you didn't write, load only its interface (types, exported signatures), not its full implementation.

```
Load: types/user.ts, not services/user.ts (unless debugging the service itself)
```

### Strategy 2: Test-as-Spec

Before reading an implementation, read its tests. Tests often tell you more about behavior and contracts than the implementation does — in fewer tokens.

### Strategy 3: Diff-Focused

When reviewing or extending existing code, start with `git diff` from the last known-good commit. The diff shows exactly what changed — it's a compressed, high-signal view of the problem space.

### Strategy 4: Pattern Sampling

When you need to follow existing patterns (naming, error handling, logging), load 1–2 examples of the pattern done well, not the entire codebase's worth of examples.

### Strategy 5: Lazy Loading

Don't load files speculatively. Load a file when you actually need it — not because you might need it. The `@explore` subagent can find files on demand without loading them into context.

## When Context Gets Too Large

Signs that context has grown unwieldy:

- You're reading the same file section repeatedly to find something
- You've lost track of which file contains which function
- Earlier context is no longer relevant to the current decision

When this happens:

1. Identify the minimal set of context required for the remaining work
2. Complete the current slice
3. Start a clean session for the next slice, with only what's needed

## Context for Each Slice Type

| Slice Type        | Load This                                                        | Don't Load This                                               |
| ----------------- | ---------------------------------------------------------------- | ------------------------------------------------------------- |
| New API endpoint  | Route file, types, test file, 1 existing route as pattern        | All other routes, services not used                           |
| UI component      | Component file, design system types, existing similar component  | Unrelated pages, global store (unless needed)                 |
| Data model change | Schema file, migration tool pattern, affected types              | All query files (load only the ones the model change affects) |
| Bug fix           | The failing test, the buggy file, surrounding caller context     | Unrelated modules                                             |
| Refactor          | File being refactored, its tests, callers (for interface impact) | Internal implementation of callers                            |

## Notes for the Build Agent

When invoking `@explore` to find files, use the result to decide what to load — not as a reason to load everything it finds.

Keep a running "context manifest" for the current slice:

```
Active context:
- spec.md (slice 2 section)
- src/auth/token.ts
- src/auth/token.test.ts
- src/types/user.ts
```

If you add a file to context, note why. If you no longer need a file, mentally prune it.
