---
name: security-review
description: Analyzes code for security vulnerabilities, misconfigurations, and compliance issues. Produces actionable findings with severity ratings and remediation guidance.
disable-model-invocation: false
---

# Skill: security-review

## Purpose
Analyze code for security vulnerabilities, misconfigurations, and compliance issues. Produces actionable findings with severity ratings and remediation guidance.

## When to Use
- Before merging code that handles user input, authentication, or sensitive data
- When introducing new dependencies or external integrations
- When deploying changes that affect authentication, authorization, or data storage
- When reviewing infrastructure or deployment configuration
- For periodic security audits of the codebase
- When handling PII, financial data, or regulated data

## Input Format
```yaml
review_target:
  files:
    - path: "src/main/java/com/example/auth/AuthController.java"
    - path: "src/main/java/com/example/config/SecurityConfig.java"
    - path: "infra/terraform/variables.tf"
    - path: "src/main/resources/application.yml"
  sensitivity: "pii" | "financial" | "auth" | "infrastructure" | "general"
  threat_model:
    - "unauthenticated access"
    - "privilege escalation"
    - "data exfiltration"
  compliance:
    - "SOC2"
    - "GDPR"
    - "HIPAA"
```

## Output Format
```yaml
findings:
  - id: "SEC-001"
    severity: "critical" | "high" | "medium" | "low"
    category: "injection" | "broken-auth" | "xss" | "ssrf" | "misconfiguration"
    file: "src/main/java/com/example/user/UserRepository.java"
    line: 24
    description: "SQL injection in user lookup query (JPQL concatenation)"
    cwe: "CWE-89"
    remediation: "Use parameterized JPQL query with named parameters (:email)"
    proof_of_concept: "input: ' OR 1=1 --"
  - id: "SEC-002"
    severity: "medium"
    category: "missing-encryption"
    file: "infra/terraform/variables.tf"
    description: "Database password stored in plaintext in terraform variables"
    remediation: "Use AWS Secrets Manager or SSM Parameter Store (SecureString)"
compliance_gaps:
  - standard: "GDPR"
    gap: "No data retention policy in user deletion endpoint"
score:
  overall: "B"
  critical: 0
  high: 1
  medium: 3
  low: 5
```

## Execution Steps
1. **Scope definition** — Define which files, dependencies, and configurations are in scope.
2. **Automated scanning** — Run static analysis (SpotBugs, Semgrep, OWASP Dependency Check, etc.).
3. **Spring Security review** — Examine `SecurityFilterChain`, `@PreAuthorize` annotations, CORS config, CSRF protection.
4. **Manual review** — Examine authentication flows, input validation (`@Valid` / `@Validated`), data storage, and crypto usage.
5. **Dependency audit** — Check for known vulnerabilities (`mvn org.owasp:dependency-check-maven:check`).
6. **Threat modeling** — Identify attack vectors and trust boundaries.
7. **Report generation** — Document findings with severity, remediation steps, and proof of concept.
8. **Remediation guidance** — Provide code-level fixes (Spring Config, annotation corrections, repository fixes).

## Examples

### Example 1: SQL injection (JPA)
**Input:** `@Query("SELECT u FROM User u WHERE u.email = '" + email + "'")`
**Finding:** Critical SQL injection via JPQL string concatenation.
**Remediation:** `@Query("SELECT u FROM User u WHERE u.email = :email")` with `@Param("email")`.

### Example 2: Missing authorization (Spring)
**Input:** `@PostMapping("/api/v1/admin/users")` without `@PreAuthorize`.
**Finding:** High-severity missing authorization — any authenticated user can create admin accounts.
**Remediation:** Add `@PreAuthorize("hasRole('ADMIN')")` and test with `@WithMockUser(roles = "USER")` to verify rejection.

### Example 3: Hardcoded secret
**Input:** `AwsSecretKey = "wJalrXUt..."` in `application.yml`.
**Finding:** High-severity credential exposure in version control.
**Remediation:** `AwsSecretKey=${AWS_SECRET_KEY}` and add to SSM Parameter Store.
