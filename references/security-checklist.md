# Security Checklist Reference

Quick reference for security checks in Java/Spring Boot applications.
Use alongside `security-review` and `security-and-hardening` skills.

## Authentication & Session Management

- [ ] Passwords hashed with BCrypt (Spring Security `BCryptPasswordEncoder`, strength >= 12)
- [ ] JWT tokens signed with RS256, short expiry (15 min access, 7 day refresh with rotation)
- [ ] CSRF protection enabled for cookie-based auth (disabled for JWT-only APIs)
- [ ] Session fixation protection (`SessionFixationProtectionStrategy`)
- [ ] Login rate limited (max 10 attempts/15 min, use Redis)
- [ ] Logout invalidates tokens server-side (token blacklist in Redis)

## Authorization

- [ ] Method-level security: `@PreAuthorize`, `@PostAuthorize` on every sensitive endpoint
- [ ] Role hierarchy checked: `hasRole('ADMIN')` not just `isAuthenticated()`
- [ ] Row-level security: users can only access their own resources (repository-level checks)
- [ ] No IDOR: verify ownership before returning data — `findByIdAndOwnerId(id, userId)`
- [ ] API keys restricted by scope and origin

## Input Validation

- [ ] Jakarta Validation: `@NotNull`, `@Email`, `@Size`, `@Pattern` on all DTOs
- [ ] SQL injection: use JPA/Hibernate or parameterized JDBC (`PreparedStatement`), never string concatenation
- [ ] NoSQL injection: sanitize query parameters for Elasticsearch/DynamoDB
- [ ] File uploads: restrict extensions, MIME types, file size (`spring.servlet.multipart.max-file-size`)
- [ ] XML/JSON deserialization: disable external entity processing (XXE prevention)

## Spring Security Configuration

```java
@Bean
SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http
        .headers(headers -> headers
            .xssProtection(XssProtectionHeaderWriter::enable)
            .contentSecurityPolicy(csp -> csp.policyDirectives("script-src 'self'")))
        .sessionManagement(sm -> sm.sessionCreationPolicy(STATELESS))
        .exceptionHandling(eh -> eh.authenticationEntryPoint(new Http403EntryPoint()))
        .cors(Customizer.withDefaults())
        .csrf(csrf -> csrf.disable())        // only if stateless JWT
        .authorizeHttpRequests(auth -> auth
            .requestMatchers("/api/v1/auth/**").permitAll()
            .requestMatchers("/actuator/health").permitAll()
            .requestMatchers("/api/v1/admin/**").hasRole("ADMIN")
            .anyRequest().authenticated()
        );
    return http.build();
}
```

## Data Protection

- [ ] No secrets in code or version control (use SSM Parameter Store or Vault)
- [ ] PII encrypted at rest in RDS (PostgreSQL `pgcrypto` or column-level AES)
- [ ] `@JsonIgnore` on sensitive fields (password, tokens) in entities
- [ ] Logging: never log passwords, tokens, or full PII (use Logback masking)
- [ ] TLS enforced: `server.ssl.enabled=true` in production, HTTPS redirect

## Infrastructure & DevOps

- [ ] Security headers: CSP, HSTS, X-Frame-Options, X-Content-Type-Options
- [ ] CORS restricted to known origins (`allowedOrigins` in config, no wildcard)
- [ ] Dependencies: `mvn dependency-check:check` or OWASP plugin in CI
- [ ] Docker images use distroless or slim base (not `latest`)
- [ ] CloudWatch/ELK alerts on: 5xx spikes, failed auth bursts, known attack patterns
- [ ] S3 buckets: block public access, enable encryption, versioning

## OWASP Top 10 Quick Check

| # | Category | Java/Spring Check |
|---|---|---|
| 1 | Broken Access Control | `@PreAuthorize` on every endpoint, test for IDOR |
| 2 | Cryptographic Failures | BCrypt, HTTPS, no custom crypto |
| 3 | Injection | JPA parameterized queries, @Valid DTOs |
| 4 | Insecure Design | Rate limiting, input validation at boundaries |
| 5 | Security Misconfiguration | Spring Security headers, debug=false |
| 6 | Vulnerable Components | `mvn dependency-check`, Dependabot |
| 7 | Auth Failures | JWT rotation, MFA for admin |
| 8 | Data Integrity | CSRF for cookies, signed webhooks |
| 9 | Logging Failure | Logback masking, structured JSON logs |
| 10 | SSRF | URL validation, restrict outbound from EC2/EKS |
