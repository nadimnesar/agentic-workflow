---
name: refactoring
description: Restructures existing code to improve readability, maintainability, performance, or adherence to design patterns without changing external behavior.
disable-model-invocation: false
---

# Skill: refactoring

## Purpose
Restructure existing code to improve readability, maintainability, performance, or adherence to design patterns without changing external behavior.

## When to Use
- Code is difficult to understand or modify
- Duplicated logic exists in multiple places
- A function or module has too many responsibilities
- Migrating from one pattern to another (e.g., callbacks to async/await)
- Improving testability of tightly coupled code
- Reducing technical debt before adding new features

## Input Format
```yaml
refactoring_target:
  files:
    - path: "src/main/java/com/example/legacy/LegacyService.java"
  goals:
    - "reduce cyclomatic complexity"
    - "extract reusable utilities"
    - "improve null safety (Optional)"
    - "remove dead code"
  constraints:
    - "public API surface must not change"
    - "all existing tests must pass"
    - "no new dependencies"
  patterns:
    - "extract-method"
    - "strategy-pattern"
    - "composite-pattern"
```

## Output Format
```yaml
changes:
  - file: "src/main/java/com/example/legacy/LegacyService.java"
    refactoring: "extract-method"
    description: "Extracted validation logic into validateInput()"
    details:
      - "Moved 30 lines from processOrder() into new private method"
      - "Reduced processOrder() cyclomatic complexity from 15 to 7"
      - "Replaced null checks with Optional API"
  - file: "src/main/java/com/example/common/DateUtils.java"
    refactoring: "consolidate-duplicates"
    description: "Merged 3 similar date formatting functions into 1"
metrics_before:
  complexity: 85
  duplication_percent: 12
  lines_of_code: 1200
  checkstyle_violations: 23
metrics_after:
  complexity: 62
  duplication_percent: 4
  lines_of_code: 1050
  checkstyle_violations: 2
verification:
  - "mvn clean test passes"
  - "checkstyle:main passes"
  - "unchanged public API"
```

## Execution Steps
1. **Understand intent** — Read the code and determine what behavior must be preserved.
2. **Measure baseline** — Record current metrics (complexity, duplication, LOC, checkstyle violations).
3. **Identify extraction targets** — Find long functions, duplicated blocks, unclear naming, null-prone patterns.
4. **Apply refactorings** — One atomic change at a time (extract method, rename, introduce Optional, etc.).
5. **Verify after each step** — Run `mvn test` after every atomic refactoring.
6. **Update callers** — Ensure all call sites use the new structure.
7. **Final verification** — `mvn clean verify` + diff review to confirm no behavioral change.

## Examples

### Example 1: Extract method from god function
**Input:** `OrderProcessor.processOrder()` is 200+ lines with null checks, DB calls, email sending, and logging interleaved.
**Output:** Split into `validateOrder()`, `calculateTotals()`, `applyDiscounts()`, `persistOrder()` (JPA repository), `sendNotifications()` (Kafka event).

### Example 2: Replace conditionals with strategy (Java)
**Input:** A 12-branch `if-else` in `ShippingCostCalculator.calculate(carrier, weight, zone)`.
**Output:** `ShippingStrategy` interface with `FedExShippingStrategy`, `UPSShippingStrategy`, `USPSShippingStrategy` implementations injected via Spring `@Component`.
