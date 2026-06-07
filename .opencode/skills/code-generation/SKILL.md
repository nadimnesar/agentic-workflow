---
name: code-generation
description: Generates production-ready source code from specifications, natural language descriptions, or pseudo-code. Use when implementing new features, scaffolding modules, or creating boilerplate.
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
  language: "python" | "typescript" | "go" | "rust" | ...
  framework: "django" | "react" | "fastapi" | ...
  constraints:
    - "performance requirement"
    - "security constraint"
    - "compatibility note"
  interfaces:
    - "existing API or function signature to conform to"
  acceptance_criteria:
    - "criterion 1"
    - "criterion 2"
```

## Output Format
```yaml
generated_files:
  - path: "src/lib/feature.py"
    content: "..."
    type: "implementation" | "test" | "config"
  - path: "tests/test_feature.py"
    content: "..."
    type: "test"
entry_points:
  - "main entry or export"
dependencies_added:
  - "package@version"
```

## Execution Steps
1. **Parse specification** — Extract language, framework, constraints, and acceptance criteria.
2. **Context gathering** — Read existing files in the target area to infer conventions, patterns, and imports.
3. **Architecture outline** — Determine files needed, their responsibilities, and dependencies between them.
4. **Generate implementation** — Write each file following existing code style (naming, formatting, typing).
5. **Generate tests** — Always produce corresponding unit/integration tests.
6. **Validate** — Check for syntax errors, import resolution, and basic type consistency.
7. **Integrate** — Ensure new code connects properly with existing interfaces.

## Examples

### Example 1: New REST endpoint
**Input:** "Add a POST /users endpoint that creates a user from a JSON body. Use FastAPI. Validate email format."
**Output:** Updated routes.py with the new endpoint, a pydantic schema for the request body, and a test file covering valid/invalid email cases.

### Example 2: New React component
**Input:** "Create a DataTable component that accepts columns and rows props, supports sorting and pagination."
**Output:** DataTable.tsx with typed props, a useSort hook, a usePagination hook, and a Storybook story file.
