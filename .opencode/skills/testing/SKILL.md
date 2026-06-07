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
    - path: "src/main/java/com/example/billing/BillingService.java"
      methods:
        - "calculateInvoice"
        - "applyCredit"
  test_levels: ["unit", "integration"]
  framework: "junit5" | "jasmine" | "go-test" | ...
  existing_tests:
    path: "src/test/java/com/example/billing/BillingServiceTest.java"
    coverage_percent: 34
  requirements:
    - "edge cases for empty invoices"
    - "concurrent credit application"
    - "boundary values for discount percentages"
```

## Output Format
```yaml
test_suite:
  framework: "junit5"
  build_tool: "maven"
  files:
    - path: "src/test/java/com/example/billing/BillingServiceTest.java"
      coverage_delta: "+45%"
      tests:
        - name: "calculateInvoice_shouldReturnZero_whenNoItems"
          type: "unit"
          coverage: "edge-case"
        - name: "calculateInvoice_shouldApplyDiscount_whenEligible"
          type: "unit"
          coverage: "happy-path"
        - name: "applyCredit_concurrentModification"
          type: "integration"
          coverage: "race-condition"
  property_tests:
    - "discount is never negative"
    - "total always equals sum of line items"
  fixtures:
    - name: "sampleInvoice"
      factory: "InvoiceTestFactory.createDefault()"
```

## Execution Steps
1. **Coverage analysis** — Identify untested branches, error paths, and edge cases. Check JaCoCo report if available.
2. **Test structure design** — Organize by level: unit (JUnit + Mockito), integration (@SpringBootTest / test slices), E2E.
3. **Generate happy-path tests** — Test primary functionality with valid inputs.
4. **Generate edge-case tests** — Boundary values, null/empty inputs, large payloads.
5. **Generate error-path tests** — Validation failures, auth failures (`@WithMockUser` / `@WithAnonymousUser`), resource exhaustion.
6. **Parameterized tests** — Use `@ParameterizedTest` for multiple input variations.
7. **Fixtures and factories** — Use Builder pattern or `@TestMethodOrder` for sequential test data.
8. **Verify** — Run `mvn test`, check JaCoCo coverage, confirm zero regressions.

## Examples

### Example 1: Invoice calculation (Spring Boot)
**Input:** `BillingService.calculateInvoice(List<LineItem> items, BigDecimal discountRate, BigDecimal taxRate)`
**Output:** `@ExtendWith(MockitoExtension.class)` tests for empty items, 100% discount, zero tax, negative quantities (`assertThrows`), concurrent modifications with `@Nested` + `@RepeatedTest`.

### Example 2: User registration (Spring Boot)
**Input:** `AuthService.registerUser(CreateUserRequest request)`
**Output:** Tests with `@ParameterizedTest` + `@NullSource` + `@EmptySource` for email, `@WebMvcTest` for controller validation, `@DataJpaTest` for unique constraint, Testcontainers for real PostgreSQL.
