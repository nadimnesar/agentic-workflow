---
name: testing
description: Designs and generates comprehensive test suites including unit tests, integration tests, property-based tests, and edge-case coverage.
---

# Skill: testing

## Purpose
Design and generate comprehensive test suites including unit tests, integration tests, property-based tests, and edge-case coverage. Validates correctness, robustness, and regression safety.

## When to Use
- Writing tests for new code
- Backfilling tests for existing untested code
- Identifying and filling coverage gaps
- Testing complex state machines or business logic
- Adding property-based or fuzz tests for invariant validation
- Generating test fixtures and factories

## Input Format
```yaml
test_target:
  files:
    - path: "src/services/billing.py"
      functions:
        - "calculate_invoice"
        - "apply_credit"
  test_levels: ["unit", "integration"]
  framework: "pytest" | "jest" | "go-test" | ...
  existing_tests:
    path: "tests/test_billing.py"
    coverage_percent: 34
  requirements:
    - "edge cases for empty invoices"
    - "concurrent credit application"
    - "boundary values for discount percentages"
```

## Output Format
```yaml
test_suite:
  framework: "pytest"
  files:
    - path: "tests/test_billing.py"
      coverage_delta: "+45%"
      tests:
        - name: "test_calculate_invoice_empty"
          type: "unit"
          coverage: "edge-case"
        - name: "test_calculate_invoice_with_items"
          type: "unit"
          coverage: "happy-path"
        - name: "test_apply_credit_concurrent"
          type: "integration"
          coverage: "race-condition"
  property_tests:
    - "discount is never negative"
    - "total always equals sum of line items"
  fixtures:
    - name: "sample_invoice"
      scope: "module"
```

## Execution Steps
1. **Coverage analysis** — Identify untested branches, error paths, and edge cases.
2. **Test structure design** — Organize tests by unit, integration, and property categories.
3. **Generate happy-path tests** — Test primary functionality with valid inputs.
4. **Generate edge-case tests** — Boundary values, empty states, null inputs, large inputs.
5. **Generate error-path tests** — Invalid inputs, auth failures, resource exhaustion.
6. **Property-based tests** — Invariants that must hold for all inputs.
7. **Fixtures and factories** — Create reusable test data builders.
8. **Verify** — Run full suite, measure new coverage, confirm zero regressions.

## Examples

### Example 1: Invoice calculation
**Input:** `calculate_invoice(items, discount_rate, tax_rate)`
**Output:** Tests for empty items, 100% discount, zero tax, negative quantities (error), concurrent modifications, floating-point precision.

### Example 2: User registration
**Input:** `register_user(email, password, profile)`
**Output:** Tests for valid registration, duplicate email, weak password, SQL injection attempt, special characters in name, email normalization.
