# Agentic Workflow Template

A reusable, production-ready template for **personalized agentic workflows**.
Works with **Zed Editor** (Agent Panel) and **OpenCode**.

## What This Is

This template provides a structured, extensible agentic system that any project
can adopt:

- **Skill-first execution** — 28 well-defined capabilities with clear input/output contracts.
- **Multi-agent orchestration** — Planner → Builder → Tester → Reviewer pipeline
  that runs autonomously with minimal user interruption.
- **Parallel execution** — Independent tasks run concurrently, merging results
  automatically.
- **Internet-aware research** — Researcher agent fetches external knowledge when
  needed.

Model and effort selection are handled via the Zed Agent Panel UI.

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
│   ├── code-generation/             # Scaffold modules and boilerplate
│   ├── code-review-and-quality/     # Multi-axis code review
│   ├── code-simplification/         # Reduce complexity, preserve behavior
│   ├── context-engineering/         # Optimize agent context setup
│   ├── debugging-and-error-recovery/# Root-cause debugging
│   ├── api-and-interface-design/    # API and module boundary contracts
│   ├── system-design/               # Architecture and trade-off analysis
│   ├── testing/                     # Comprehensive test suite generation
│   ├── security-review/             # Vulnerability analysis and compliance
│   ├── refactoring/                 # Behavior-preserving restructuring
│   ├── test-driven-development/     # Red-green-refactor cycle
│   ├── security-and-hardening/      # OWASP prevention and hardening
│   ├── frontend-ui-engineering/     # Production-quality UI
│   ├── browser-testing-with-devtools/ # Chrome DevTools MCP
│   ├── ci-cd-and-automation/        # Pipeline automation
│   ├── deprecation-and-migration/   # Sunset old systems safely
│   ├── documentation-and-adrs/      # ADRs and documentation
│   ├── doubt-driven-development/    # In-flight adversarial review
│   ├── git-workflow-and-versioning/ # Commit discipline and branching
│   ├── idea-refine/                 # Divergent/convergent ideation
│   ├── incremental-implementation/  # Thin vertical slices
│   ├── interview-me/                # One-question-at-a-time intent extraction
│   ├── performance-optimization/    # Measure-first optimization
│   ├── planning-and-task-breakdown/ # Task decomposition
│   ├── shipping-and-launch/         # Pre-launch checklist and rollout
│   ├── source-driven-development/   # Official-doc-verified code
│   ├── spec-driven-development/     # Spec before code
│   └── using-agent-skills/          # Meta-skill: discover and route
├── agents/
│   ├── planner.md                   # Task decomposition and orchestration
│   ├── builder.md                   # Implementation and coding
│   ├── tester.md                    # Validation and test generation
│   ├── reviewer.md                  # Code review and quality assurance
│   └── researcher.md                # External knowledge retrieval
└── router/
    └── skill-router.md              # Intent-based skill selection and chaining

AGENTS.md                            # Core workflow lifecycle and rules
README.md                            # This file
```

## How It Works

```
Skill Router ──► Planner ──► Builder ──► Tester ──► Reviewer ──► Done
    │              │            │           │            │
    ▼              ▼            ▼           ▼            ▼
Researcher    Dependency    Skill-based  Validation   Quality
(fallback)    Graph        Execution    & Edge Cases  Review
```

Model and effort level are selected in the Zed Agent Panel UI.
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

3. Add routing in `.opencode/skills/using-agent-skills/SKILL.md` (the meta-skill decision tree).

## Extending Agents

Each agent in `.opencode/agents/` can be customized. To add a new agent:
1. Create `.opencode/agents/<agent-name>.md` with input/output contracts and behavior rules.
2. Update the pipeline in `AGENTS.md` if it should be part of the core flow.

## Requirements

- **Zed Editor** (v0.160+) with Agent Panel enabled, or
- **OpenCode**

## License

MIT — Use freely in any project, personal or commercial.
