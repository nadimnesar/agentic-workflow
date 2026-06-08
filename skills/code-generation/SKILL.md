---
name: code-generation
description: Generates production-ready source code from specifications, natural language descriptions, or pseudo-code. Use when implementing new features, scaffolding modules, or creating boilerplate.
disable-model-invocation: false
---

# Skill: code-generation

## Purpose
Generate production-ready source code from specifications, natural language descriptions, or pseudo-code. Covers new feature implementation, scaffolding, and boilerplate generation.

## When to Use
- Implementing a new feature from a spec or ticket
- Scaffolding a new module, component, or file
- Generating boilerplate (CRUD endpoints, data models, API clients)
- Translating pseudo-code or algorithm descriptions into real code
- Creating integration glue between existing components

## Input Format
```yaml
specification:
  description: "Natural language or structured description of what to build"
  language: "java" | "typescript" | "go" | "sql" | ...
  framework: "spring-boot" | "angular" | "micronaut" | ...
  constraints:
    - "performance requirement"
    - "security constraint"
    - "compatibility note"
  interfaces:
    - "existing API or interface to conform to"
  acceptance_criteria:
    - "criterion 1"
    - "criterion 2"
```

## Output Format
```yaml
generated_files:
  - path: "src/main/java/com/example/feature/FeatureController.java"
    content: "..."
    type: "implementation" | "test" | "migration" | "config"
  - path: "src/test/java/com/example/feature/FeatureControllerTest.java"
    content: "..."
    type: "test"
  - path: "src/main/resources/db/changelog/changelog-003.yml"
    content: "..."
    type: "migration"
entry_points:
  - "REST endpoint exposed at /api/v1/feature"
dependencies_added:
  - "org.springframework.boot:spring-boot-starter-web"
  - "com.example:common:1.0.0"
```

## Execution Steps
1. **Parse specification** — Extract language, framework, constraints, and acceptance criteria. Refer to `references/tech-stack.md` for defaults.
2. **Context gathering** — Read existing files in the target area to infer conventions, patterns, and imports.
3. **Architecture outline** — Determine files needed, their responsibilities, and dependencies between them.
4. **Generate implementation** — Write each file following existing code style (naming, formatting, package structure).
5. **Generate tests** — Always produce corresponding unit/integration tests using JUnit 5 + Mockito.
6. **Validate** — Check for compilation, import resolution, and basic type consistency.
7. **Integrate** — Ensure new code connects properly with existing interfaces.

## Examples

### Example 1: New REST endpoint (Spring Boot)
**Input:** "Add a POST /users endpoint that creates a user from a JSON body. Use Java 21 + Spring Boot 3. Validate email format."
**Output:** `UserController.java` with `@RestController`, `UserRequest` DTO with `@Email` validation, `UserService` with business logic, `UserEntity` JPA entity with Liquibase changelog, and `@WebMvcTest` + `@DataJpaTest` test classes.

### Example 2: New Angular component
**Input:** "Create a DataTable component that accepts columns and rows inputs, supports sorting and pagination. Angular 17 standalone."
**Output:** `DataTableComponent` with `@Input()` columns/rows, `OnPush` change detection, Angular Material table, sort directive, paginator, and Jasmine tests.
