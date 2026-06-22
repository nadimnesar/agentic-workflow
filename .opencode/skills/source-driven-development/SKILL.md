---
name: source-driven-development
description: Verify against official documentation or source code before implementing anything that depends on an external library, API, or framework. Never implement from memory or assumption — look it up.
license: MIT
compatibility: opencode
metadata:
  phase: build
  agent: build
---

# Source-Driven Development

## What I Do

I prevent the single most common cause of bugs in AI-assisted code: using APIs, options, and behaviors from memory that are either wrong or version-specific. I require verification against the actual source of truth before implementation.

## The Core Rule

> If you didn't write it, look it up before you use it.

This applies to:
- External library APIs and their parameters
- Framework lifecycle hooks and their contracts
- HTTP API endpoints and their request/response shapes
- Database query patterns and their edge cases
- Language built-ins with non-obvious behavior (encoding, precision, timezone)
- Configuration formats and their valid options

## The Verification Protocol

### Step 1: Identify what needs verification

Before writing any code that uses an external dependency, list the specific APIs you plan to use:
```
Need to verify:
- [ ] library.doThing(arg1, arg2) — what are the exact parameter types?
- [ ] What does it return on error — exception or null?
- [ ] Does it handle empty inputs differently in v2 vs v3?
```

### Step 2: Find the authoritative source

Priority order (highest to lowest):
1. **Official documentation** for the exact installed version
2. **Source code** of the library (read from node_modules/vendor)
3. **Type definitions** (.d.ts, stubs) — weaker but fast
4. **Tests in the library's own repo** — often the most honest documentation
5. **Recent Stack Overflow / GitHub Issues** — only for edge case confirmation, never for API shape

Never use:
- Your training data memory for API specifics
- An old blog post
- A code example without checking the version it was written for

### Step 3: Record what you found

Before writing implementation code, write a brief note:
```
Verified: library.doThing(input: string, options?: { timeout: number }): Promise<Result>
Source: https://[docs-url] — version 3.4.2
Behavior on empty string: throws ValidationError (not null)
Note: `timeout` default is 5000ms, not 0
```

### Step 4: Write code against what you verified

If the verified API differs from what you expected:
- Update your implementation plan
- Flag the difference to Core if it affects the task estimate or design

### Step 5: Pin the version assumption

If the behavior is version-specific, note it in the code:
```typescript
// Verified against express@4.18.2 — req.body requires express.json() middleware
// In express@5, this behavior changes (see: link)
```

## What Counts as "External"

Assume something is external (requires verification) if:
- It's in `package.json`, `requirements.txt`, `go.mod`, `Cargo.toml`, etc.
- It's a standard library function with locale, timezone, or precision behavior
- It's a web API (browser APIs can change between major versions)
- It's an internal utility you didn't personally write recently

## Deep Library Source Exploration

For deep library source exploration, read the source directly from `node_modules/` or `vendor/`:
```
Read the source of [library] in node_modules/[library]/ to understand how it handles [specific behavior] in version [X]
```

## Red Flags That Trigger Verification

Immediately stop and verify if you catch yourself:
- "I think the API is..."
- "It probably returns..."
- "I've used this before and it was..."
- "The parameter order is X, Y, Z (I'm pretty sure)"
- Writing code that calls a function with more than 2 parameters from memory

## Output Format

When this skill is active, include a "Sources Verified" section in your slice report:
```
Sources verified:
- [library@version]: [what was verified] — [doc URL or "source"]
- [browser API]: [what was verified] — MDN link
```
