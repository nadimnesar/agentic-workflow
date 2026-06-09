# Agentic Workflow System

This project uses a **Core-driven orchestration model**: a single primary agent (Core)
manages all work by coordinating specialized subagents (Definer, Planner, Builder, Tester, Reviewer).

Full architecture documentation: `docs/ARCHITECTURE.md`
OpenCode agent reference: `docs/primary-agent-reference.md`

---

## What This Is

This template provides a structured, extensible agentic system for **OpenCode**:

- **Core orchestration** — One primary agent manages the entire workflow.
- **Specialized subagents** — Definer (intent & spec), Planner (decomposition), Builder (implementation), Tester (validation), Reviewer (code review).
- **Skill-first execution** — 28 well-defined capabilities across 6 lifecycle phases.
- **Documented workflow** — Plans and test results are persisted to `docs/agentic/` for traceability.
- **Adaptable** — Stack defaults live in `references/tech-stack.md`.

---

## Architecture Overview

```
User Request
    │
    ▼
┌──────────────────────────────────────────────────────────────┐
│                      CORE                                     │
│  1. Understand the task                                       │
│  2. @definer → writes docs/agentic/definer.md                 │
│  3. @planner → writes docs/agentic/planner.md                 │
│  4. @builder → implements according to plan                   │
│  5. @tester → writes docs/agentic/tester.md                   │
│  6. @reviewer → evaluates code quality                        │
│  7. Iterate if bugs or changes needed                         │
│  8. Deliver final result                                      │
└──────────────────────────────────────────────────────────────┘
```

---

## Setup (Step by Step)

### Prerequisites

- **OpenCode** installed: `curl -fsSL https://opencode.ai/install | bash`
- An AI provider configured (e.g., OpenRouter, Anthropic, OpenAI)

### Step 1: Install the skills globally

```bash
mkdir -p ~/.agents
cp -r skills ~/.agents/skills
```

### Step 2: Configure your provider in `~/.config/opencode/opencode.jsonc`

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "model": "openrouter/your-model",
  "provider": {
    "openrouter": {
      "options": {
        "apiKey": "sk-or-..."
      }
    }
  }
}
```

### Step 3: Verify

1. Run `opencode` in this project directory.
2. The **Core** agent is the default primary agent.
3. Type `@` to see available subagents (definer, planner, builder, tester, reviewer).

---

## Repository Structure

```
AGENTS.md                            # Core workflow lifecycle and rules

.opencode/
├── agents/
│   ├── core.md                      # Primary agent: orchestrator
│   ├── definer.md                   # Subagent: intent extraction & spec
│   ├── planner.md                   # Subagent: task decomposition
│   ├── builder.md                   # Subagent: implementation
│   ├── tester.md                    # Subagent: test validation
│   └── reviewer.md                  # Subagent: code review
└── router/
    └── skill-router.md              # Intent-based skill selection

docs/
├── ARCHITECTURE.md                  # Full system design
├── primary-agent-reference.md       # Orchestrator design notes
└── agentic/
    ├── definer.md                   # Spec output (written by @definer)
    ├── planner.md                   # Plan output (written by @planner)
    └── tester.md                    # Test results (written by @tester)

skills/                              # 28 skills (copy to ~/.agents/skills/)
├── api-and-interface-design/
├── code-generation/
├── testing/
└── ... (28 total)

references/
├── tech-stack.md                    # Default technology preferences
├── testing-patterns.md              # Testing patterns and anti-patterns
├── security-checklist.md            # Security checklist
└── performance-checklist.md         # Performance optimization patterns
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

1. **Core** understands your request
2. **Core** → **@definer** → extracts intent, refines ideas, writes spec to `docs/agentic/definer.md`
3. **Core** reads the spec
4. **Core** → **@planner** → writes plan to `docs/agentic/planner.md`
5. **Core** reads the plan
6. **Core** → **@builder** → implements code and tests
7. **Core** → **@tester** → validates, writes results to `docs/agentic/tester.md`
8. **Core** reads the results
   - ✅ All clear → **@reviewer** evaluates quality
   - ❌ Bugs found → Builder fixes, Tester re-tests
9. **Core** delivers the final result to you

### Available Subagents

| Agent | @mention | Purpose |
|-------|----------|---------|
| **Definer** | `@definer` | Extracts intent, refines ideas, writes spec to `docs/agentic/definer.md` |
| **Planner** | `@planner` | Decomposes tasks, writes to `docs/agentic/planner.md` |
| **Builder** | `@builder` | Implements according to plan |
| **Tester** | `@tester` | Validates, writes to `docs/agentic/tester.md` |
| **Reviewer** | `@reviewer` | Read-only code quality evaluation |

---

## Tech Stack Adaptation

Edit `references/tech-stack.md` to retarget for any technology stack.

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
