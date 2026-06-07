# Agentic Workflow Template

An **OpenCode** workflow template for **Zed Editor** (Agent Panel).
Provides structured multi-agent orchestration with 28 skills across 6 lifecycle
phases.

## What This Is

This template provides a structured, extensible agentic system:

- **Skill-first execution** — 28 well-defined capabilities grouped by phase (Define, Plan, Build, Verify, Review, Ship).
- **Multi-agent orchestration** — Planner → Builder → Tester → Reviewer pipeline
  that runs autonomously with minimal user interruption.
- **Parallel execution** — Independent tasks run concurrently, merging results
  automatically.
- **Reference checklists** — Curated quick-reference guides for testing, security, and performance.

Model and effort selection are handled via the Zed Agent Panel UI (not
duplicated in config files).

## Quick Start

```bash
# Copy into your project
```

Open in Zed with the Agent Panel enabled. Make a request like:
- "Add a paginated search endpoint to the user API"
- "Design the architecture for a real-time notification system"
- "Review the auth module for security issues"

## Repository Structure

```
.opencode/
├── skills/                          # 28 reusable capability definitions
│   ├── api-and-interface-design/    # API and module boundary contracts
│   ├── browser-testing-with-devtools/ # Chrome DevTools MCP
│   ├── ci-cd-and-automation/        # Pipeline automation
│   ├── code-generation/             # Scaffold modules and boilerplate
│   ├── code-review-and-quality/     # Multi-axis code review
│   ├── code-simplification/         # Reduce complexity, preserve behavior
│   ├── context-engineering/         # Optimize agent context setup
│   ├── debugging-and-error-recovery/# Root-cause debugging
│   ├── deprecation-and-migration/   # Sunset old systems safely
│   ├── documentation-and-adrs/      # ADRs and documentation
│   ├── doubt-driven-development/    # In-flight adversarial review
│   ├── frontend-ui-engineering/     # Production-quality UI
│   ├── git-workflow-and-versioning/ # Commit discipline and branching
│   ├── idea-refine/                 # Divergent/convergent ideation
│   ├── incremental-implementation/  # Thin vertical slices
│   ├── interview-me/                # One-question-at-a-time intent extraction
│   ├── performance-optimization/    # Measure-first optimization
│   ├── planning-and-task-breakdown/ # Task decomposition
│   ├── refactoring/                 # Behavior-preserving restructuring
│   ├── security-and-hardening/      # OWASP prevention and hardening
│   ├── security-review/             # Vulnerability analysis and compliance
│   ├── shipping-and-launch/         # Pre-launch checklist and rollout
│   ├── source-driven-development/   # Official-doc-verified code
│   ├── spec-driven-development/     # Spec before code
│   ├── system-design/               # Architecture and trade-off analysis
│   ├── test-driven-development/     # Red-green-refactor cycle
│   ├── testing/                     # Comprehensive test suite generation
│   └── using-agent-skills/          # Meta-skill: discover and route
├── agents/
│   ├── planner.md                   # Task decomposition and orchestration
│   ├── builder.md                   # Implementation and coding
│   ├── tester.md                    # Validation and test generation
│   ├── reviewer.md                  # Code review and quality assurance
│   └── researcher.md                # External knowledge retrieval
├── router/
│   └── skill-router.md              # Intent-based skill selection and chaining
└── ... (no other tool-specific config files)

references/
├── testing-patterns.md              # Test structure, assertions, patterns
├── security-checklist.md            # OWASP Top 10, auth, input validation
└── performance-checklist.md         # Core Web Vitals, budgets, tooling

AGENTS.md                            # Core workflow lifecycle and rules
README.md                            # This file
```

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

## Adding New Skills

1. Create `.opencode/skills/<skill-name>/SKILL.md`.
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

3. Add routing in `.opencode/skills/using-agent-skills/SKILL.md` (the meta-skill decision tree) and `.opencode/router/skill-router.md` (the phase-grouped mapping table).

## Extending Agents

Each agent in `.opencode/agents/` can be customized. To add a new agent:
1. Create `.opencode/agents/<agent-name>.md` with input/output contracts and behavior rules.
2. Update the pipeline in `AGENTS.md` if it should be part of the core flow.

## Requirements

- **Zed Editor** (v0.160+) with Agent Panel enabled
- **OpenCode** (for OpenCode-native setups)

## License

MIT — Use freely in any project, personal or commercial.
