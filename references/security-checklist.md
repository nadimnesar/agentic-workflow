# Security Checklist Reference

Quick reference for security checks. Use alongside `security-review` and `security-and-hardening` skills.

## Authentication

- [ ] Passwords hashed with bcrypt/scrypt/argon2 (salt rounds >= 12)
- [ ] Session tokens are httpOnly, secure, sameSite
- [ ] Login has rate limiting (max 10 attempts/15min)
- [ ] Password reset tokens expire (15-60 min)
- [ ] MFA available for sensitive actions

## Authorization

- [ ] Every endpoint checks user permissions
- [ ] Users can only access their own resources
- [ ] Admin actions require admin role verification
- [ ] No IDOR vulnerabilities (check object ownership server-side)

## Input Validation

- [ ] All user input validated at system boundaries
- [ ] SQL queries are parameterized (no string concatenation)
- [ ] HTML output is encoded/escaped (use framework defaults)
- [ ] File uploads: type whitelist, size limit, scan for malware
- [ ] Third-party API responses validated before use

## Data Protection

- [ ] No secrets in code or version control
- [ ] Sensitive fields excluded from API responses
- [ ] PII encrypted at rest (if applicable)
- [ ] Secrets managed via environment variables or vault

## Infrastructure

- [ ] Security headers: CSP, HSTS, X-Frame-Options, X-Content-Type-Options
- [ ] CORS restricted to known origins (no wildcard)
- [ ] Dependencies audited: `npm audit` (or equivalent)
- [ ] Error messages don't expose internals or stack traces
- [ ] Rate limiting on all API endpoints

## OWASP Top 10 Quick Check

| # | Category | Check |
|---|---|---|
| 1 | Broken Access Control | Authorization on every endpoint |
| 2 | Cryptographic Failures | HTTPS, hashed passwords, encrypted PII |
| 3 | Injection | Parameterized queries, input validation |
| 4 | Insecure Design | Threat modeling, rate limiting |
| 5 | Security Misconfiguration | Headers, CORS, debug mode off |
| 6 | Vulnerable Components | Dependency audit, update regularly |
| 7 | Auth Failures | MFA, rate limiting, session management |
| 8 | Data Integrity | CSRF tokens, signed webhooks |
| 9 | Logging Failure | Error tracking, audit logs |
| 10 | SSRF | Validate URLs, restrict outbound traffic |
