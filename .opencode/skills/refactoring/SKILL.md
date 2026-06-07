---
name: refactoring
description: Restructures existing code to improve readability, maintainability, performance, or adherence to design patterns without changing external behavior.
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
    - path: "src/legacy/module.py"
  goals:
    - "reduce cyclomatic complexity"
    - "extract reusable utilities"
    - "improve type coverage"
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
  - file: "src/legacy/module.py"
    refactoring: "extract-method"
    description: "Extracted validation logic into validate_input()"
    details:
      - "Moved 30 lines from process_order() into new method"
      - "Reduced process_order() cyclomatic complexity from 15 to 7"
  - file: "src/legacy/utils.py"
    refactoring: "consolidate-duplicates"
    description: "Merged 3 similar date formatting functions into 1"
metrics_before:
  complexity: 85
  duplication_percent: 12
  lines_of_code: 1200
metrics_after:
  complexity: 62
  duplication_percent: 4
  lines_of_code: 1050
verification:
  - "full test suite passes"
  - "unchanged public API"
```

## Execution Steps
1. **Understand intent** — Read the code and determine what behavior must be preserved.
2. **Measure baseline** — Record current metrics (complexity, duplication, LOC).
3. **Identify extraction targets** — Find long functions, duplicated blocks, unclear naming.
4. **Apply refactorings** — One atomic change at a time (extract method, rename, move, etc.).
5. **Verify after each step** — Run tests after every atomic refactoring.
6. **Update callers** — Ensure all call sites use the new structure.
7. **Final verification** — Full test suite + diff review to confirm no behavioral change.

## Examples

### Example 1: Extract method from god function
**Input:** `process_order()` is 200 lines with mixed concerns.
**Output:** Split into `validate_order()`, `calculate_totals()`, `apply_discounts()`, `persist_order()`, `send_notifications()`.

### Example 2: Replace conditionals with strategy
**Input:** A 12-branch if-else for shipping cost calculation by carrier.
**Output:** Strategy interface with `FedExShipping`, `UPSShipping`, `USPSShipping` implementations.
