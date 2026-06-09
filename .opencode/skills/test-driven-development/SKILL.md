---
name: test-driven-development
description: Write a failing test first, then write the minimum code to make it pass. Tests define behavior — not implementation. A test suite that only passes is not enough; it must fail when the behavior breaks.
license: MIT
compatibility: opencode
metadata:
  phase: test
  agent: test
---

# Test-Driven Development

## What I Do

I enforce the discipline of test-first: every piece of behavior has a test that was written before (or alongside) the implementation. Tests are the specification. If a behavior isn't tested, it isn't guaranteed.

## The TDD Loop

```
1. WRITE a failing test for the next behavior
2. VERIFY it fails (don't skip this — a test that always passes is worse than no test)
3. WRITE the minimum implementation to pass it
4. VERIFY the test now passes
5. VERIFY no other tests broke
6. REFACTOR if needed (without changing tests)
7. REPEAT for the next behavior
```

## What Makes a Good Test

A test is good when:
- **It fails when the behavior breaks.** This is the only real measure. Ask: "If I deleted this function, would this test catch it?"
- **It tests behavior, not implementation.** The test shouldn't care *how* the result is achieved, only *what* was achieved.
- **It is readable as a spec.** A new engineer should understand the behavior from the test name and body alone.
- **It is fast and deterministic.** No sleeps, no random, no "usually passes."

A test is bad when:
- It tests that a mock was called (instead of that the outcome happened)
- It tests private methods directly
- It passes even when the behavior is broken
- It requires reading the implementation to understand what it's testing

## Test Structure

Every test should follow the **Given / When / Then** pattern:

```typescript
describe('getUserPermissions', () => {
  it('returns empty array when user has no permissions assigned', async () => {
    // Given
    const userId = 'user-with-no-perms';
    await db.users.create({ id: userId });
    // no permissions assigned

    // When
    const result = await getUserPermissions(userId, defaultContext);

    // Then
    expect(result).toEqual([]);
  });

  it('throws AuthError when userId does not exist', async () => {
    // Given
    const nonExistentId = 'ghost-user';

    // When / Then
    await expect(
      getUserPermissions(nonExistentId, defaultContext)
    ).rejects.toThrow(AuthError);
  });
});
```

## Coverage Strategy

Write tests in this priority order:

1. **Happy path**: The main case that the feature exists to handle
2. **Boundary conditions**: Empty inputs, maximum sizes, zero values, null/undefined
3. **Error cases**: Invalid inputs, missing resources, network failures, auth failures
4. **Edge cases from the spec**: Any AC marked as "edge case" or "given invalid input"

Do not write tests for:
- Implementation internals that aren't observable from the outside
- Private methods (test them through their public callers)
- Third-party library behavior (that library has its own tests)

## The Coverage Bar

Coverage numbers are a floor, not a goal:
- Aim for 100% coverage of the files changed in this slice
- Below 80% is a red flag — investigate what's missing
- 100% line coverage with no assertions is worthless — check assertion quality

To verify a test is meaningful: comment out the implementation body and run the test. It should fail.

## Test File Organization

```
// File: src/auth/token.test.ts (mirrors src/auth/token.ts)

describe('[module name]', () => {
  describe('[function/method name]', () => {
    describe('[scenario group, e.g., "when user is authenticated"]', () => {
      it('[specific behavior in plain English]', () => { ... });
    });
  });
});
```

Test names should be readable as sentences:
- ✓ "returns null when product is not found"
- ✓ "throws ValidationError when email format is invalid"
- ✗ "test 1"
- ✗ "should work correctly"

## Integration vs Unit Tests

| Type | Tests what | Speed | Use for |
|---|---|---|---|
| Unit | Single function/module in isolation | < 10ms each | Logic, transformations, validations |
| Integration | Multiple modules together, real DB/APIs | < 1s each | Service boundaries, DB queries |
| E2E | Full system, browser | < 30s each | Critical user flows (3–5 total) |

For most slices, unit + integration is sufficient. E2E tests are for user-facing critical paths only.

## Handling Flaky Tests

Flaky tests (non-deterministically passing or failing) are bugs. They erode trust in the suite. Fix them:

- **Timing-based flakiness**: Replace `sleep(100)` with proper async waiting (wait for element, wait for state)
- **State pollution**: Each test must set up and tear down its own state
- **External service flakiness**: Use a real test instance, not production; or mock at the network boundary

Never add `retry(3)` to a flaky test — find the root cause.

## Output: Test Report

After running the suite:
```
Test run results:
  New tests added: N
  Tests modified: N
  Total tests: N
  Passing: N
  Failing: 0 (or list failures)
  Coverage on changed files: N%

Behaviors now guaranteed:
  - [behavior 1]
  - [behavior 2]
  - [edge case covered]
```
