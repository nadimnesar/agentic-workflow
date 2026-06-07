# Skill Router

## Purpose
Automatically selects the best skill(s) for a given user request. Supports multi-skill chaining, fallback logic, and precision-based dispatch.

## Classification Logic

The router classifies intent along two axes:
- **Phase**: The development lifecycle phase (Define, Plan, Build, Verify, Review, Ship)
- **Domain**: The subject area within that phase

### Phase Grouping

```
Define Phase      — What are we building?
Plan Phase        — How do we break this down?
Build Phase       — Write the code
Verify Phase      — Does it work correctly?
Review Phase      — Is it good enough to ship?
Ship Phase        — Deploy and document
```

### Intent-to-Skill Mapping

```
User Intent              → Phase    → Primary Skill           → Optional Secondary
─────────────────────────────────────────────────────────────────────────────────────
"Define what we need"    → Define   → interview-me            → idea-refine, spec-driven-development
"Design architecture"    → Define   → system-design           → (none)
"Shape a vague idea"     → Define   → idea-refine             → interview-me
"Break down the work"    → Plan     → planning-and-task-breakdown → (none)
"Build a new feature"    → Build    → incremental-implementation → test-driven-development
"Add an API endpoint"    → Build    → api-and-interface-design → incremental-implementation
"Build a UI component"   → Build    → frontend-ui-engineering → incremental-implementation
"Generate boilerplate"   → Build    → code-generation         → (none)
"Write tests"            → Verify   → test-driven-development → testing
"Something is broken"    → Verify   → debugging-and-error-recovery → test-driven-development
"Security audit needed"  → Review   → security-review         → security-and-hardening
"Make it more secure"    → Review   → security-and-hardening  → security-review
"Improve code quality"   → Review   → code-review-and-quality → code-simplification
"Simplify this code"     → Review   → code-simplification     → code-review-and-quality
"Refactor for clarity"   → Review   → refactoring             → testing
"Optimize performance"   → Review   → performance-optimization → (none)
"Set up CI/CD"           → Ship     → ci-cd-and-automation    → (none)
"Deprecate an old API"   → Ship     → deprecation-and-migration → documentation-and-adrs
"Document decisions"     → Ship     → documentation-and-adrs  → (none)
"Prepare for launch"     → Ship     → shipping-and-launch     → (none)
"Research best practices"→ (any)    → researcher agent        → context-engineering
```

## Routing Algorithm

1. **Extract intent** — Parse the user message for action verbs and domain nouns.
2. **Score skills** — Each skill produces a relevance score (0.0–1.0) based on keyword overlap and semantic similarity.
3. **Select primary** — The skill with the highest score above `threshold: 0.4`.
4. **Chain secondaries** — If the primary score > 0.7 AND other skills score > 0.3, chain them as secondary passes.
5. **Fallback** — If no skill scores above 0.4:
   - If the request is exploratory → route to `researcher` agent.
   - If the request is general → route to `planner` for decomposition.
   - If neither → respond with clarifying questions.

## Multi-Skill Chaining

When multiple skills match, execution follows this pattern:

```
Skill A (primary) → Output
    ↓
Skill B (secondary, optional) → Refines or extends output
    ↓
Skill C (tertiary, optional) → Validates or reviews output
```

Chain rules:
- Output of Skill A becomes input context for Skill B.
- Each skill in the chain operates on the accumulated artifact.
- The chain terminates after the final skill or when `max_chain_length: 3` is reached.
- If any skill in the chain produces an error, the chain halts and falls back to the last successful state.

## Fallback Handler

When no skill matches sufficiently, the router responds with:

```
I couldn't confidently map your request to a known skill.
Available phases: Define, Plan, Build, Verify, Review, Ship
(28 skills across all phases)

Could you clarify one of:
• What stage of development are you in?
• What are you trying to accomplish?
• The researcher agent can help explore if you're unsure.
```

Reference checklists are available in `references/` for quick lookups:
- `references/testing-patterns.md`
- `references/security-checklist.md`
- `references/performance-checklist.md`

## Reference Checklists

Skills can pull in supplementary checklists from `references/` for deeper domain coverage without repeating content in every skill:

- `references/tech-stack.md` — Canonical technology preferences (Java 21, Spring Boot 3, Angular 17, PostgreSQL, AWS, Kafka)
- `references/testing-patterns.md` — JUnit 5, Mockito, AssertJ, Spring Boot test slices, Testcontainers
- `references/security-checklist.md` — Spring Security, OWASP Top 10, auth, input validation, AWS infra
- `references/performance-checklist.md` — Spring Boot tuning, Angular performance, PostgreSQL query optimization

## Router Configuration

```yaml
routing:
  threshold_primary: 0.4
  threshold_secondary: 0.3
  max_chain_length: 3
  fallback_to_researcher: true
  require_confirmation: false  # set to true to ask user before executing
  priority: precision          # precision | recall | balanced
```
