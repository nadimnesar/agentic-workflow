# Agentic Workflow System

This project uses a **Core-driven orchestration model**: a single primary agent (Core)
manages all work by coordinating five specialized subagents — **Define**, **Plan**,
**Build**, **Test**, and **Review**.

---

## What This Is

This template provides a structured, extensible agentic system for **OpenCode**:

- **Core orchestration** — One primary agent manages the entire software delivery lifecycle.
- **Specialized subagents** — `@define` (intent & spec), `@plan` (task decomposition),
  `@build` (implementation), `@test` (validation), `@review` (code review).
- **Quality gates** — Every stage has a gate that must be passed before advancing.
  No code moves forward with failing tests, unclear requirements, or unchecked complexity.
- **Feedback loops** — If any stage fails its gate, work flows back to the appropriate
  previous stage, not forward.
- **15 curated skills** — Each subagent loads only the skills relevant to its stage,
  keeping context windows lean and focused.

---

## The Pipeline

```
User Request
     │
     ▼
┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌──────────┐
│ @define │ -> │  @plan  │ -> │ @build  │ -> │  @test  │ -> │ @review  │
└─────────┘    └─────────┘    └─────────┘    └─────────┘    └──────────┘
  Surface         Tasks         Implement       Verify         Simplify &
  true need       & scope       in slices       & debug        Optimize
```

### Stage 0 — Triage

Core assesses the request:

- **Trivial** (typo fix, rename, config tweak) → delegate directly to `@build`.
- **Non-trivial** (new feature, architecture change, complex bug) → run the full pipeline.
- **Ambiguous** → always start with `@define`.

### Stage 1 — Define

`@define` interviews the user, refines the idea, and produces a written spec
with acceptance criteria. Uses: `interview-me`, `idea-refine`, `spec-driven-development`.

**Gate:** No code is planned until the spec is approved.

### Stage 2 — Plan

`@plan` decomposes the spec into small, independently verifiable tasks ordered
for maximum parallelism and minimum risk. Uses: `planning-and-task-breakdown`.

**Gate:** No code is written until every task is a thin vertical slice.

### Stage 3 — Build

`@build` implements one slice at a time using thin vertical slices, verifying
against official docs and applying adversarial review to every non-trivial decision.
Uses: `incremental-implementation`, `source-driven-development`,
`doubt-driven-development`, `context-engineering`, `frontend-ui-engineering`,
`api-and-interface-design`.

**Gate:** Each slice must be functionally complete before the next begins.

### Stage 4 — Test

`@test` writes failing tests first, then verifies they pass. Uses browser DevTools
for runtime UI verification. Applies a structured reproduce→localize→fix→guard
protocol for bugs. Uses: `test-driven-development`, `browser-testing-with-devtools`,
`debugging-and-error-recovery`.

**Gate:** All acceptance criteria must pass. Regressions are blockers.

### Stage 5 — Review

`@review` evaluates simplicity and performance — removes unnecessary complexity
without changing behavior, measures before optimizing. Uses: `code-simplification`,
`performance-optimization`.

**Gate:** No unnecessary complexity, no unguarded performance regressions.

### Feedback Loops

| Failing gate  | Returns to                                     |
|---------------|------------------------------------------------|
| Define → Plan | `@define` with clarifying questions            |
| Plan → Build  | `@plan` with scope concerns                    |
| Build → Test  | `@build` with specific failing criteria        |
| Test → Review | `@test` — do not review broken code            |
| Review → Done | Apply suggestions, re-test if behavior changes |

---

## Setup (Step by Step)

### Prerequisites

- **OpenCode** installed: `curl -fsSL https://opencode.ai/install | bash`
- Configure an AI provider (e.g., OpenRouter, Anthropic, OpenAI), by default OpenCode use Zen.

### Step 2: Verify

1. Run `opencode` in this project directory.
2. The **Core** agent is the default primary agent.
3. Type `@` to see available subagents (define, plan, build, test, review).

---

## Repository Structure

```
.opencode/
├── agents/
│   ├── core.md                  # Primary agent: orchestrator (this is the default)
│   ├── define.md                # Subagent: intent extraction & spec
│   ├── plan.md                  # Subagent: task decomposition
│   ├── build.md                 # Subagent: implementation
│   ├── test.md                  # Subagent: test validation
│   └── review.md                # Subagent: code review
└── skills/                      # 15 skills, loaded on demand by each subagent
    ├── api-and-interface-design/
    ├── browser-testing-with-devtools/
    ├── code-simplification/
    ├── context-engineering/
    ├── debugging-and-error-recovery/
    ├── doubt-driven-development/
    ├── frontend-ui-engineering/
    ├── idea-refine/
    ├── incremental-implementation/
    ├── interview-me/
    ├── performance-optimization/
    ├── planning-and-task-breakdown/
    ├── source-driven-development/
    ├── spec-driven-development/
    └── test-driven-development/
```

---

## How to Use

The **Core** agent is the only agent you interact with directly. It manages everything:

```bash
opencode
# Then simply describe what you want:
# "Add a paginated search endpoint to the UserController"
```

### What happens:

1. **Core** triages your request (trivial, non-trivial, or ambiguous)
2. **Core** → `@define` → interviews, refines ideas, writes spec
3. **Core** reads the spec, confirms acceptance criteria
4. **Core** → `@plan` → decomposes spec into task slices
5. **Core** reviews the task list
6. **Core** → `@build` → implements one slice at a time
7. **Core** → `@test` → validates against acceptance criteria
    - ✅ All clear → `@review` evaluates quality
    - ❌ Bugs found → `@build` fixes, `@test` re-tests
8. **Core** → `@review` → simplifies and optimizes
9. **Core** delivers the final result with a Delivery Summary

### Available Subagents

| Agent      | @mention  | Purpose                                                                      |
|------------|-----------|------------------------------------------------------------------------------|
| **Define** | `@define` | Surfaces true requirements, interviews, writes spec with acceptance criteria |
| **Plan**   | `@plan`   | Decomposes spec into small, independently verifiable task slices             |
| **Build**  | `@build`  | Implements one thin vertical slice at a time                                 |
| **Test**   | `@test`   | Writes failing tests first, verifies behavior, debugs failures               |
| **Review** | `@review` | Removes unnecessary complexity, measures before optimizing                   |

---

## Adding New Subagents

Create `.opencode/agents/<agent-name>.md` with YAML frontmatter:

```markdown
---
description: What this agent does
mode: subagent
permission:
  read: allow
  edit: deny
---

System prompt instructions here...
```

Immediately available via `@<agent-name>` — no restart needed.

---

## License

MIT
