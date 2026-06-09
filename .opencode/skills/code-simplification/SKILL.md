---
name: code-simplification
description: Preserve behavior while reducing unnecessary complexity. Remove dead code, flatten indirection that doesn't earn its weight, improve names, and eliminate duplication — without changing what the code does.
license: MIT
compatibility: opencode
metadata:
  phase: review
  agent: review
---

# Code Simplification

## What I Do

I find code that is harder to understand, navigate, or modify than it needs to be — and make it simpler without changing its behavior. I am not a style enforcer. I am a complexity reducer.

## The Simplification Standard

The bar for changing something is: **"This is actively harder to understand or maintain as-is."** Not: "I would have written this differently." Not: "This is not how I learned to do it."

Every change must:

1. Preserve identical external behavior
2. Not remove or weaken any test
3. Make the code demonstrably clearer or less risky

If you're uncertain whether a change is safe: do not make it.

---

## Category 1: Dead Code

Dead code is the easiest to remove and the safest to delete. Find and remove:

- **Unused variables**: declared but never read
- **Unreachable code**: after a `return`, `throw`, or in a condition that's always false
- **Commented-out code blocks**: if it's commented out, it's either wrong (don't restore it) or should be in git history (delete it)
- **Unused exports**: functions/classes exported but imported nowhere
- **Stale feature flags**: flags that are always true/false in all environments

Before removing: confirm the item is genuinely unused (search the repo, not just the file).

---

## Category 2: Over-Abstraction

Abstractions that don't earn their weight add navigation cost (you have to jump to them to understand the code) without adding reusability or clarity.

Signs of over-abstraction:

- A function used exactly once, whose name is not clearer than its body
- A class with one method
- A utility helper that wraps a single standard library call with no additional logic
- A type alias that aliases exactly one concrete type with no added documentation

Test: "If I inline this abstraction, does the call site become harder to read?" If not, inline it.

---

## Category 3: Indirection That Doesn't Help

Related to over-abstraction, but specifically about code that makes you jump through layers to understand what happens:

- Factory functions that always return the same concrete type
- Abstract base classes with a single implementation
- Config objects passed to functions that use one field
- Callbacks used where a direct call would do

Test: "Can I follow what happens here without jumping to another file?" If not, simplify the path.

---

## Category 4: Duplication

The same logic in more than one place is a future bug and a maintenance burden.

Finding duplication:

- Same conditional logic repeated in multiple functions
- Same data transformation done slightly differently in two places
- Same validation logic in the API layer and the service layer
- Copy-pasted test setup across multiple test files (should be a shared fixture)

When consolidating:

- Ensure both duplicates actually do the same thing (subtle differences are common)
- Extract to the right layer (don't put business logic in utility helpers)
- Update all call sites
- Run the full test suite

---

## Category 5: Name Honesty

Names should tell you what a thing does, not what it is or when it was created.

**Functions:**

- Names should describe what the function _does_ (verb phrase): `getUserById`, `calculateTotal`, `validateEmail`
- Not: `userHelper`, `doStuff`, `handle`, `process` (too vague)
- Not: `newUserFunction`, `tempFix` (stale context)

**Variables:**

- Names should describe what the value _is_: `userPermissions`, `filteredItems`, `retryCount`
- Not: `data`, `result`, `temp`, `x` (except in tight mathematical loops)

**Booleans:**

- Names should read as questions: `isLoading`, `hasError`, `canEdit`
- Not: `loading`, `error`, `edit` (ambiguous — is it the state, the action, or the handler?)

**Magic values:**

- Numbers that appear inline without explanation → extract to named constants
- Strings that are repeated as identifiers → extract to constants or enums

---

## Category 6: Complexity of Control Flow

Deeply nested conditionals are harder to reason about than flat code.

Techniques:

- **Early return / guard clause**: instead of `if (condition) { big block }`, invert: `if (!condition) return; [big block now at top level]`
- **Extract condition**: complex boolean → well-named function: `if (isEligibleForDiscount(user))` instead of `if (user.age > 18 && user.loyaltyPoints > 100 && !user.suspended)`
- **Avoid else when return is possible**: if every branch returns, the else is unnecessary indentation

Limit: Don't flatten structure just to reduce nesting levels — sometimes nesting reflects real structure. The question is whether the nesting obscures what the code does.

---

## Simplification Process

For each file changed in this feature:

1. Read the file fresh (as if you've never seen it)
2. Note every moment of "wait, what is this doing?" — those are targets
3. Apply applicable categories above
4. Run the full test suite after each change
5. If any test fails: revert that change. Do not adjust the test to match.

---

## Output: Simplification Report

```markdown
## Simplification Report

### Dead code removed

- [file.ts:42] Unused variable `tempData`
- [utils.ts] Commented-out `legacyFormat()` block (dead since 2023)

### Consolidations

- [validators.ts + api.ts] Email validation logic was duplicated → consolidated into `validators.ts`

### Name improvements

- [auth.ts] `handle()` → `authenticateRequest()`
- [cart.ts] magic number `3600` → constant `SESSION_TTL_SECONDS`

### Control flow simplifications

- [checkout.ts:88] 4-level nesting → 2-level with guard clauses

### Not changed (and why)

- [service.ts] Abstraction looks heavy but is used in 6 places — consolidation is correct
```
