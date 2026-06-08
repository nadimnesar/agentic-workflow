# Agentic Workflow System

This project uses an agentic workflow lifecycle (Route → Plan → Build → Test → Review).
Coding agents should follow these rules when operating on this project.

---

## Quick Start

```bash
# This project uses opencode for agentic workflows
# Install: curl -fsSL https://opencode.ai/install | bash
```

---

## File Locations

| Component         | Path                             | Scope                                                     |
| ----------------- | -------------------------------- | --------------------------------------------------------- |
| Skills (global)   | `~/.agents/skills/`              | Always loaded; 28 skills available                        |
| Builder agent     | `.opencode/agents/builder.md`    | Project-level agent definition                            |
| Planner agent     | `.opencode/agents/planner.md`    | Project-level agent definition                            |
| Researcher agent  | `.opencode/agents/researcher.md` | Project-level agent definition                            |
| Reviewer agent    | `.opencode/agents/reviewer.md`   | Project-level agent definition                            |
| Tester agent      | `.opencode/agents/tester.md`     | Project-level agent definition                            |
| Lifecycle & rules | `AGENTS.md` (this file)          | Project instructions (auto-loaded by Zed, opencode, etc.) |

> **Note:** Skills only live at `~/.agents/skills/` (global). The project `.opencode/skills/` has been removed — do not look for skills there.

---

## Skill Router

The skill router classifies intent and selects the appropriate skill from **`~/.agents/skills/`** (28 skills). Skills are organized as:

```
~/.agents/skills/
├── api-and-interface-design/   # SKILL.md + optional scripts/references
├── code-generation/
├── testing/
├── test-driven-development/
├── refactoring/
├── code-review-and-quality/
├── debugging-and-error-recovery/
├── planning-and-task-breakdown/
├── security-and-hardening/
├── frontend-ui-engineering/
├── ... (28 total)
```

Each skill folder contains a `SKILL.md` with metadata and instructions.

### Resolution Order

1. Agent checks skill name against `~/.agents/skills/<name>/SKILL.md`
2. If no match, falls back to researcher agent for context-gathering, then replans
3. If skill still not found, implement with general best practices

---

## Lifecycle Overview

```
User Request
    │
    ▼
┌─────────────┐     ┌──────────────┐     ┌──────────────┐
│  SKILL      │────▶│   PLANNER    │────▶│   BUILDER    │
│  ROUTER     │     │   AGENT      │     │   AGENT      │
└─────────────┘     └──────────────┘     └──────┬───────┘
    │                                            │
    │  (fallback)                                ▼
    ▼                                     ┌──────────────┐
┌─────────────┐     ┌──────────────┐     │   TESTER     │
│ RESEARCHER  │────▶│   PLANNER    │     │   AGENT      │
│ AGENT       │     │   (replan)   │     └──────┬───────┘
└─────────────┘     └──────────────┘            │
                                                 ▼
                                          ┌──────────────┐
                                          │  REVIEWER    │
                                          │  AGENT       │
                                          └──────┬───────┘
                                                 │
                                           ┌─────┴─────┐
                                           │           │
                                       Approved    Changes
                                           │      Requested
                                           ▼           │
                                      ┌────────┐       │
                                      │ DONE   │◀──────┘
                                      └────────┘  (iterate)
```

### Phase 1: Route

Skill router classifies intent and selects skill from `~/.agents/skills/`. Falls back to researcher + planner if no match.

### Phase 2: Plan

Planner decomposes into ordered tasks with dependency graph. Identifies parallelization opportunities.

### Phase 3: Build

Builder executes tasks using the assigned skill. Parallel tasks within same dependency layer run concurrently.

### Phase 4: Test

Tester validates artifacts against acceptance criteria. Reports bugs back to builder.

### Phase 5: Review

Reviewer evaluates across correctness, maintainability, performance, security, test quality.

---

## Core Principles

### Autonomy

- Operate autonomously by default
- Escalate only when: ambiguous request, irreversible consequences, equally-valid approaches
- Batch questions into single messages

### Skill-First Execution

- Load the assigned skill's `SKILL.md` before executing
- Follow its instructions precisely
- If skill doesn't exist, fall back to researcher + planner

### Iterative Refinement

- Tester finds bug → Builder fixes → Tester re-verifies
- Reviewer requests changes → Builder implements → Reviewer re-reviews
- After max iterations without resolution, escalate

---

## Convention Adherence

Before writing code:

1. Read 2-3 existing files in the same directory to infer conventions
2. Match existing: package structure, naming, import ordering, error handling, annotations
3. Default to conventions used by the majority of the codebase

### Production Quality

- Handle errors gracefully (no silent failures)
- Add input validation at all public boundaries
- Log at meaningful levels (debug=details, info=state, error=failures)
- Use appropriate abstraction — don't over-engineer

### Testing Discipline

- Always produce tests alongside implementation
- Minimum: happy path + 2 edge cases per function
- For bugs: regression test that fails before the fix
- Match existing test patterns (framework, fixtures, assertions)

### Effort Adaptation

- **low**: Minimal implementation, basic error handling, happy-path tests only
- **medium**: Full error handling, edge cases, standard test coverage
- **high**: Exhaustive edge cases, property-based tests, performance considerations

---

## Context Management

- Preserve original request throughout the entire lifecycle
- Each agent receives only the context slice it needs
- Handoff protocol: Input Contract → Execute → Output Contract
- Each handoff includes only what the next agent needs

---

## Quality Gates

Every artifact must pass:

1. **Syntax check** — No parse errors
2. **Lint** — No lint errors
3. **Type check** — No type errors
4. **Tests pass** — All existing tests still pass
5. **Acceptance criteria met** — Each criterion validated
6. **No regressions** — Preserve behavior of unchanged code

---

## Zed IDE + OpenRouter Configuration

This project is configured for use with [Zed IDE](https://zed.dev/) and [OpenRouter](https://openrouter.ai/).

### OpenRouter Setup (in Zed)

Configure OpenRouter as your model provider through Zed's UI:

1. Open **Agent Settings**: `Cmd+Shift+P` → `agent: open settings`
2. Go to the **OpenRouter** section
3. Enter your OpenRouter API key (from https://openrouter.ai/keys)
4. Select a default model

Or set `OPENROUTER_API_KEY` in your environment.

### Default Model (Zed)

This project is pre-configured to use `zebn/deepseek-v4-flash-free` via OpenRouter.
Set via `Cmd+Shift+P` → `agent: open settings` → OpenRouter → select model, or in `settings.json`:

```json
{
  "agent": {
    "default_model": {
      "provider": "openrouter",
      "model": "zebn/deepseek-v4-flash-free"
    }
  }
}
```

The `~/.config/opencode/opencode.jsonc` is already configured.

### Zed Skills Integration

Zed auto-loads skills from `~/.agents/skills/` — same location as the skill router. Skills appear in Zed's Agent Panel via `/` commands and `@skill` mentions. No additional configuration needed.

---

## Error Handling

- Task fails → document why, roll back partial changes, report to planner
- Missing dependency → attempt to resolve (install, create) before reporting
- Model refuses content → document reason, suggest alternative
- External tool unavailable → retry with exponential backoff (3 max), then fallback
