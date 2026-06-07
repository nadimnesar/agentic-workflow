---
name: security-review
description: Analyzes code for security vulnerabilities, misconfigurations, and compliance issues. Produces actionable findings with severity ratings and remediation guidance.
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
    - path: "src/api/auth.py"
    - path: "src/api/middleware.py"
    - path: "infra/terraform/variables.tf"
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
    file: "src/api/auth.py"
    line: 42
    description: "SQL injection in user lookup query"
    cwe: "CWE-89"
    remediation: "Use parameterized query instead of string interpolation"
    proof_of_concept: "input: ' OR 1=1 --"
  - id: "SEC-002"
    severity: "medium"
    category: "missing-encryption"
    file: "infra/terraform/variables.tf"
    description: "Database password stored in plaintext in terraform variables"
    remediation: "Use AWS Secrets Manager or HashiCorp Vault"
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
2. **Automated scanning** — Run static analysis (bandit, semgrep, CodeQL, etc.).
3. **Manual review** — Examine authentication flows, input handling, data storage, and crypto usage.
4. **Dependency audit** — Check for known vulnerabilities in dependencies (npm audit, cargo audit, etc.).
5. **Threat modeling** — Identify attack vectors and trust boundaries.
6. **Report generation** — Document findings with severity, remediation steps, and proof of concept.
7. **Remediation guidance** — Provide code-level fixes for each finding.

## Examples

### Example 1: SQL injection
**Input:** `cursor.execute(f"SELECT * FROM users WHERE email = '{email}'")`
**Finding:** Critical SQL injection.
**Remediation:** `cursor.execute("SELECT * FROM users WHERE email = %s", (email,))`

### Example 2: Hardcoded secret
**Input:** `API_SECRET = "sk-abc123..."` in source code.
**Finding:** High-severity credential exposure.
**Remediation:** Move to environment variable or secrets manager.
