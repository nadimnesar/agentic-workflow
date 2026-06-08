# Agentic Workflow Template

An **OpenCode** workflow template for **Zed Editor** (Agent Panel).
28 skills across 6 lifecycle phases. Multi-agent orchestration with parallel execution.

## What This Is

This template provides a structured, extensible agentic system:

- **Skill-first execution** — 28 well-defined capabilities grouped by phase (Define, Plan, Build, Verify, Review, Ship).
- **Multi-agent orchestration** — Planner → Builder → Tester → Reviewer pipeline
  that runs autonomously with minimal user interruption.
- **Reference checklists** — Curated quick-reference guides for testing, security, and performance.
- **Adaptable** — Stack defaults live in `references/tech-stack.md`. Edit one file to retarget any technology.
- **Zed + OpenRouter ready** — Pre-configured `.zed/settings.json` with OpenRouter as the AI provider.

---

## Setup (Step by Step)

### Prerequisites

- **Zed Editor** (v0.160+) with Agent Panel enabled
- **OpenRouter** account (free at https://openrouter.ai)

### Step 1: Install the skills globally

Skills live in `~/.agents/skills/` so both Zed and opencode can find them.
Move the bundled skills there:

```bash
mkdir -p ~/.agents
cp -r skills ~/.agents/skills
```

> The `skills/` folder contains 28 skills. Once copied, **Zed auto-loads them** from `~/.agents/skills/` — no config needed.
> You can delete the project `skills/` folder after copying if you don't need it as a reference.

### Step 2: Configure OpenRouter in Zed

**Option A — Via the UI (recommended):**

1. Open Zed
2. `Cmd+Shift+P` (Mac) / `Ctrl+Shift+P` (Linux) → `agent: open settings`
3. Go to the **OpenRouter** section
4. Paste your API key (from https://openrouter.ai/keys)
5. Select model `zebn/deepseek-v4-flash-free`

**Option B — Via environment variable:**

```bash
export OPENROUTER_API_KEY="sk-or-..."
```

The `~/.config/opencode/opencode.jsonc` is pre-configured with `zebn/deepseek-v4-flash-free` as the default model.

### Step 3: Verify the Agent Panel

1. Open the Agent Panel (`Cmd+Shift+E` / `Ctrl+Shift+E`)
2. You should see the project's `AGENTS.md` loaded as instructions
3. Type `/` in the message editor and check that skills from `~/.agents/skills/` appear

Try a request like:

- "Add a paginated search endpoint to the user API"
- "Design the architecture for a real-time notification system"
- "Review the auth module for security issues"

---

## Repository Structure

```
.opencode/
├── agents/
│   ├── planner.md                   # Task decomposition and orchestration
│   ├── builder.md                   # Implementation and coding
│   ├── tester.md                    # Validation and test generation
│   ├── reviewer.md                  # Code review and quality assurance
│   └── researcher.md                # External knowledge retrieval
├── router/
│   └── skill-router.md              # Intent-based skill selection and chaining
└── ... (no project-level skills — using global ~/.agents/skills/)

references/
├── tech-stack.md                    # Default technology preferences (edit to retarget)
├── testing-patterns.md              # Testing patterns and anti-patterns
├── security-checklist.md            # Security checklist (OWASP, auth, infra)
└── performance-checklist.md         # Performance optimization patterns

AGENTS.md                            # Core workflow lifecycle and rules
README.md                            # This file
```

### Where Skills Live

| Scope                    | Path                        | Loaded by                     |
| ------------------------ | --------------------------- | ----------------------------- |
| Global (all projects)    | `~/.agents/skills/`         | Auto-loaded by Zed & opencode |
| Project-local (optional) | `<project>/.agents/skills/` | Only when trusted             |

Global skills are available in every project. Project-local skills override globals
when they share the same name. There are **no project-level skills** in this template —
all 28 skills are installed globally.

---

## How It Works

```
                          ┌──────────────────────────────────────────┐
                          │  Skill Router                            │
                          │  (intent → phase → skill)                │
                          │  ┌──────┐ ┌────┐ ┌─────┐ ┌──────┐ ┌───┐ │
                          │  │Define│ │Plan│ │Build│ │Verify│ │...│ │
                          │  └──────┘ └────┘ └─────┘ └──────┘ └───┘ │
                          └────────────────┬─────────────────────────┘
                                           │
                    ┌──────────────────────▼──────────────────────┐
                    │              Planner Agent                   │
                    │  (dependency graph → parallel groups)       │
                    └──────────────────────┬──────────────────────┘
                                           │
                    ┌──────────────────────▼──────────────────────┐
                    │              Builder Agent                   │
                    │  (skill-first execution, 28 skills)         │
                    └──────────────────────┬──────────────────────┘
                                           │
                    ┌──────────────────────▼──────────────────────┐
                    │              Tester Agent                   │
                    │  (validation, edge cases, regression)       │
                    └──────────────────────┬──────────────────────┘
                                           │
                    ┌──────────────────────▼──────────────────────┐
                    │             Reviewer Agent                  │
                    │  (6-axis quality scoring)                   │
                    └──────────────────────┬──────────────────────┘
                                           │
                                           ▼
                                       Done
```

Model and effort level are selected in the **Zed Agent Panel UI**.
This template includes no config files that duplicate Zed UI functionality.
All other behavior (parallel execution, context management, tool policies)
is controlled by the agents at runtime.

---

## Tech Stack Adaptation

To adapt this template for a different stack:

1. Edit `references/tech-stack.md` — replace the language, framework, database, cloud, and messaging preferences.
2. Update agent examples in `.opencode/agents/` to match your stack conventions.
3. Customize reference checklists for your ecosystem.

All skills and agents reference `tech-stack.md` for default technology decisions.

---

## Adding New Skills

Skills are **global** (`~/.agents/skills/`) so they're available in every project.
To add a new skill:

1. Create `~/.agents/skills/<skill-name>/SKILL.md`.
2. Follow the template format:

```markdown
---
name: <skill-name>
description: One-line description of what this skill does.
---

# Skill: <skill-name>

## Purpose

What this skill does and when to use it.

## When to Use

Conditions that trigger this skill.

## Input Format

Structured YAML describing the expected input.

## Output Format

Structured YAML describing the guaranteed output.

## Execution Steps

1. Step one
2. Step two

## Examples

Realistic scenarios with inputs and outputs.
```

3. The skill is immediately available in Zed (via `/` and `@skill`) and opencode — no restart needed.

---

## Extending Agents

Each agent in `.opencode/agents/` can be customized. To add a new agent:

1. Create `.opencode/agents/<agent-name>.md` with input/output contracts and behavior rules.
2. Update the pipeline in `AGENTS.md` if it should be part of the core flow.

---

## Requirements

- **Zed Editor** (v0.160+) with Agent Panel enabled
- **OpenRouter** account (or another LLM provider supported by Zed)
- **OpenCode** (optional — for opencode-native workflows)

---

## License

MIT — Use freely in any project, personal or commercial.
