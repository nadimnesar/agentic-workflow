# Agentic Workflow Template

A reusable, production-ready template for **personalized agentic workflows**.
Works with **Zed Editor** (Agent Panel), **OpenCode**, and multiple LLM providers
(DeepSeek, Minimax, Mimo, Nemotron, and more).

## What This Is

This template provides a structured, extensible agentic system that any project
can adopt. Instead of ad-hoc AI interactions, you get:

- **Skill-first execution** — Well-defined capabilities (code generation, bug
  fixing, system design, testing, security review) with clear input/output contracts.
- **Multi-agent orchestration** — Planner → Builder → Tester → Reviewer pipeline
  that runs autonomously with minimal user interruption.
- **Parallel execution** — Independent tasks run concurrently, merging results
  automatically.
- **Configurable effort** — Choose between low (quick), medium (balanced), and
  high (deep) reasoning depth per task.
- **Internet-aware research** — Researcher agent fetches external knowledge when
  needed.
- **Multi-provider support** — Route different roles to different LLM providers
  based on capability and cost.

## Quick Start

### 1. Clone into your project

```bash
cd your-project
# Copy template files into your project
# or add as a git submodule
```

### 2. Configure providers

Edit `.opencode/config/models.yaml` to add your API keys and preferred models.
By default, DeepSeek is the primary provider with Minimax, Mimo, and Nemotron as
fallback options.

### 3. Set effort level

Edit `.opencode/config/execution.yaml` and set:
```yaml
effort: medium  # low | medium | high
parallel:
  enabled: true
```

### 4. Open with Zed or OpenCode

Open the project in Zed Editor (enable Agent Panel) or use OpenCode CLI:

```bash
# OpenCode will read AGENTS.md and .opencode/ to configure the session
opencode
```

### 5. Make a request

Try something like:
- "Add a paginated search endpoint to the user API"
- "Design the architecture for a real-time notification system"
- "Fix the bug on the billing page where discounts don't apply"
- "Review the auth module for security issues"

## Repository Structure

```
.opencode/
├── skills/                    # Reusable capability definitions
│   ├── code-generation/       # Generate production code from specs
│   │   └── SKILL.md
│   ├── bug-fixing/            # Systematic root-cause debugging
│   │   └── SKILL.md
│   ├── system-design/         # High-level architecture design
│   │   └── SKILL.md
│   ├── api-design/            # API contract design (REST, GraphQL, gRPC)
│   │   └── SKILL.md
│   ├── refactoring/           # Structural improvements without behavior change
│   │   └── SKILL.md
│   ├── testing/               # Comprehensive test suite generation
│   │   └── SKILL.md
│   └── security-review/       # Vulnerability analysis and compliance
│       └── SKILL.md
├── agents/                    # Agent definitions
│   ├── planner.md             # Task decomposition and orchestration
│   ├── builder.md             # Implementation and coding
│   ├── tester.md              # Validation and test generation
│   ├── reviewer.md            # Code review and quality assurance
│   └── researcher.md          # External knowledge retrieval
├── router/
│   └── skill-router.md        # Intent-based skill selection and chaining
└── config/
    ├── execution.yaml         # Runtime behavior (effort, parallel, tools)
    └── models.yaml            # LLM provider and model mapping

AGENTS.md                      # Core workflow lifecycle and rules
README.md                      # This file
```

## How It Works

### The Pipeline

```
User Request
    │
    ▼
Skill Router ──► Planner ──► Builder ──► Tester ──► Reviewer ──► Done
    │              │            │           │            │
    │              │            │           │            │
    ▼              ▼            ▼           ▼            ▼
Researcher    Dependency    Skill-based  Validation   Quality
(fallback)    Graph        Execution    & Edge Cases  Review
```

Each phase:
1. **Route** — Classifies intent and selects the right skill(s).
2. **Plan** — Decomposes into dependency-ordered tasks, identifies parallel work.
3. **Build** — Executes tasks using skills, produces production-ready code.
4. **Test** — Validates correctness, generates edge-case coverage.
5. **Review** — Evaluates quality across correctness, maintainability, performance, security.

If any phase finds issues, the system iterates automatically (up to `max_iterations`).

## Adding New Skills

1. Create a new directory under `.opencode/skills/<skill-name>/`.
2. Create `SKILL.md` following this format:

```markdown
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
...

## Examples
Realistic scenarios with inputs and outputs.
```

3. (Optional) Add the skill to `.opencode/router/skill-router.md` if it needs
   automatic routing.

## Extending Agents

Each agent in `.opencode/agents/` can be customized:

- **Responsibilities** — Add or refine what the agent is responsible for.
- **Input/Output Contract** — Define the data contract for handoffs.
- **Behavior Rules** — Add domain-specific rules (e.g., "always use async for I/O").

To add a new agent:
1. Create `.opencode/agents/<agent-name>.md`.
2. Follow the contract pattern from existing agents.
3. Update the pipeline in `AGENTS.md` if this agent should be part of the core flow.

## Configuration Reference

### Effort Levels

| Level  | Iterations | Test Coverage | Research Depth | Parallelism |
|--------|-----------|--------------|----------------|-------------|
| low    | 1         | Happy path   | None           | Serial      |
| medium | 2         | Full + edge  | If helpful     | Conditional |
| high   | 4         | Exhaustive   | Deep           | Full        |

### Provider Roles

| Role       | Default Provider | Model Type   |
|-----------|-----------------|--------------|
| Planner   | DeepSeek        | reasoning    |
| Builder   | DeepSeek        | chat         |
| Tester    | DeepSeek        | reasoning    |
| Reviewer  | Minimax         | reasoning    |
| Researcher| Nemotron        | chat         |
| Router    | DeepSeek        | reasoning    |

## Requirements

- **Zed Editor** (v0.160+) with Agent Panel enabled, or
- **OpenCode** (any recent version)
- **API keys** for at least one LLM provider (configurable in `models.yaml`)

## License

MIT — Use freely in any project, personal or commercial.
