# Agentic Workflow

An opinionated multi-agent software-delivery workflow for [OpenCode](https://opencode.ai/docs/).

Licensed under the [MIT License](./LICENSE). OpenCode documentation: https://opencode.ai/docs/.

## Overview

Agentic Workflow is a configuration-only package for [OpenCode](https://opencode.ai/docs/) that installs a disciplined,
multi-agent software-delivery pipeline. It contains no application code — everything lives in `.opencode/` as agent
definitions and skill files, which OpenCode auto-discovers on launch. The workflow is built around a single principle:
the agent that receives your request should think, route, and verify — never execute the work itself.

At the center is **Core**, a non-coding orchestrator. Core never writes production code, never edits source files, and
never runs implementation commands. Instead it triages each request, routes non-trivial work through a chain of
specialized subagents — Define, Plan, Build, Test, Review — and checks an explicit gate before advancing from one stage
to the next. Each subagent loads only the skills it needs for the current task, produces a structured report, and hands
control back to Core.

The problem this solves is the most common failure mode of AI coding agents: **self-execution**. Given a familiar task,
an agent recognizes it, jumps straight to a solution, and skips the verification steps that would have caught a wrong
assumption. Agentic Workflow makes that impossible by construction. Core is structurally barred from writing code, every
stage has pass/fail gate criteria, and a failed gate always loops back to the stage that failed — never forward. The
result is a workflow where requirements are surfaced before code exists, work is decomposed into thin verifiable slices,
external APIs are checked against official docs rather than memory, and every bug fix gets a regression test.

## The Pipeline

```
User Request
     |
     v
[Core: triage]  -- trivial? --> @implement (direct)
     |
     v (non-trivial)
+---------+    +---------+    +---------------+    +---------+    +----------+
| @define | -> | @planner| -> | @implement    | -> |  @test  | -> | @review  |
| spec +  |    | slices +|    | [parallel if  |    | TDD +   |    | simplify |
| ACs     |    | deps    |    |  safe]        |    | browser |    | + perf   |
+---------+    +---------+    +---------------+    +---------+    +----------+
     |              |              |                   |              |
     +-- gate ------+-- gate ------+-- gate -----------+-- gate ------+
   failed gate -> feedback loop back to the failing stage (never advances past a failed gate)
```

**Stages:**

- **Triage** (Core) — Core loads `orchestration-protocol` and decides whether the request is trivial (routed directly to
  `@implement`) or non-trivial (enters the full pipeline).
- **Define** (`@define`) — surfaces the real problem through a structured interview, explores alternative approaches, and produces a
  spec with Given/When/Then acceptance criteria.
- **Plan** (`@planner`) — decomposes the approved spec into small, independently verifiable task slices ordered for maximum
  parallelism and minimum risk.
- **Build** (`@implement`) — implements one thin vertical slice at a time, verifying external APIs against official docs and applying
  adversarial review to non-trivial decisions. Slices with no dependencies run concurrently.
- **Test** (`@test`) — writes failing tests first, verifies they pass, checks for regressions, and runs browser-based runtime
  verification for UI slices.
- **Review** (`@review`) — simplifies completed, tested code and optimizes only hot paths that have been measured. Runs only after
  all tests pass.

**Feedback-loop guarantee:** every stage ends in a gate. If a gate fails, Core routes back to the stage that failed with
specific feedback — it never advances past a failed gate. A failed Build gate returns to Build; a failed Test gate
returns to Build (not Review); a failed Review gate routes back through Build and Test.

## Agents

| Agent     | Mode     | Temp | Steps  | Role                                                                                                                                                                                                              | Skills                                                                                                                                                  |
|-----------|----------|------|--------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------|
| core      | primary  | 0.2  | 60     | Primary orchestrator. Coordinates the full lifecycle across Define, Plan, Build, Test, Review. Never writes production code — only thinks, routes, and verifies. Enforces pipeline gates and parallel dispatch.   | orchestration-protocol, parallel-dispatch                                                                                                               |
| define    | subagent | 0.4  | 30     | Surfaces true requirements before any code exists. Interviews the user, refines the idea through divergent/convergent thinking, and produces a spec with Given/When/Then acceptance criteria.                     | interview-me, idea-refine, spec-driven-development                                                                                                      |
| planner   | subagent | 0.1  | 30     | Decomposes an approved spec into small, independently verifiable task slices ordered for maximum parallelism and minimum risk. Never writes code.                                                                 | planning-and-task-breakdown                                                                                                                             |
| implement | subagent | 0.2  | 50     | Implements one thin vertical slice at a time. Verifies against official docs, applies adversarial review of non-trivial decisions, manages context carefully, builds production-quality UI and stable interfaces. | incremental-implementation, source-driven-development, doubt-driven-development, context-engineering, frontend-ui-engineering, api-and-interface-design |
| test      | subagent | 0.1  | 40     | Verifies built slices meet acceptance criteria. Writes failing tests first, then verifies they pass. Uses browser DevTools for runtime UI verification. Applies reproduce, localize, fix, guard debugging.        | test-driven-development, browser-testing-with-devtools, debugging-and-error-recovery                                                                    |
| review    | subagent | 0.1  | 30     | Reviews completed, tested code for simplicity and performance. Preserves behavior while removing unnecessary complexity. Measures before optimizing. Only runs after all tests pass.                              | code-simplification, performance-optimization                                                                                                           |

Core is the only `primary` agent; the other five are `hidden: true` `subagent`s that Core dispatches via the Task tool,
so they do not appear in the `@mention` menu. See the [OpenCode agents docs](https://opencode.ai/docs/agents/) for the
`mode`, `temperature`, `color`, `permission`, `steps`, and `hidden` frontmatter fields.

## Skills

Skills are loaded on demand — never all at once. Each subagent loads only the skills relevant to its current slice,
which keeps context focused. See the [OpenCode skills docs](https://opencode.ai/docs/skills/) for the skill file format.

### Orchestration

| Skill                                                                        | Purpose                                                            |
|------------------------------------------------------------------------------|--------------------------------------------------------------------|
| [orchestration-protocol](./.opencode/skills/orchestration-protocol/SKILL.md) | Mandatory routing and self-audit for Core; prevents self-execution |
| [parallel-dispatch](./.opencode/skills/parallel-dispatch/SKILL.md)           | Concurrent subagent execution for independent slices               |

### Define

| Skill                                                                          | Purpose                                                                 |
|--------------------------------------------------------------------------------|-------------------------------------------------------------------------|
| [interview-me](./.opencode/skills/interview-me/SKILL.md)                       | Structured requirements interview to surface the real problem           |
| [idea-refine](./.opencode/skills/idea-refine/SKILL.md)                         | Divergent then convergent thinking; 3+ approaches before choosing       |
| [spec-driven-development](./.opencode/skills/spec-driven-development/SKILL.md) | Turn refined ideas into a spec with Given/When/Then acceptance criteria |

### Plan

| Skill                                                                                  | Purpose                                                    |
|----------------------------------------------------------------------------------------|------------------------------------------------------------|
| [planning-and-task-breakdown](./.opencode/skills/planning-and-task-breakdown/SKILL.md) | Decompose spec into small, verifiable, ordered task slices |

### Build

| Skill                                                                                | Purpose                                                         |
|--------------------------------------------------------------------------------------|-----------------------------------------------------------------|
| [incremental-implementation](./.opencode/skills/incremental-implementation/SKILL.md) | Thin vertical slices; red-green-commit loop                     |
| [source-driven-development](./.opencode/skills/source-driven-development/SKILL.md)   | Verify external APIs against official docs, never memory        |
| [doubt-driven-development](./.opencode/skills/doubt-driven-development/SKILL.md)     | Adversarial review of every non-trivial decision                |
| [context-engineering](./.opencode/skills/context-engineering/SKILL.md)               | Load only the right context for the current slice               |
| [frontend-ui-engineering](./.opencode/skills/frontend-ui-engineering/SKILL.md)       | Production-quality UI: semantic HTML, accessibility, responsive |
| [api-and-interface-design](./.opencode/skills/api-and-interface-design/SKILL.md)     | Design stable interfaces and contracts before internals         |

### Test

| Skill                                                                                      | Purpose                                                       |
|--------------------------------------------------------------------------------------------|---------------------------------------------------------------|
| [test-driven-development](./.opencode/skills/test-driven-development/SKILL.md)             | Failing test first; tests define behavior, not implementation |
| [browser-testing-with-devtools](./.opencode/skills/browser-testing-with-devtools/SKILL.md) | Runtime UI verification via Chrome DevTools                   |
| [debugging-and-error-recovery](./.opencode/skills/debugging-and-error-recovery/SKILL.md)   | Reproduce, localize, fix, guard debugging protocol            |

### Review

| Skill                                                                            | Purpose                                                 |
|----------------------------------------------------------------------------------|---------------------------------------------------------|
| [code-simplification](./.opencode/skills/code-simplification/SKILL.md)           | Preserve behavior while reducing unnecessary complexity |
| [performance-optimization](./.opencode/skills/performance-optimization/SKILL.md) | Measure first, optimize only what measurably matters    |

## Getting Started

1. Install OpenCode — see https://opencode.ai/docs/.
2. Clone this repository:

   ```bash
   git clone https://github.com/nadimnesar/agentic-workflow.git
   cd agentic-workflow
   ```

3. OpenCode auto-discovers `.opencode/agents/*.md` and `.opencode/skills/*/SKILL.md` — no manual registration of agents
   or skills is needed.
4. An `opencode.json` is included at the repo root for `$schema` validation and safe global permission defaults; see
   the [Configuration](#configuration) section and https://opencode.ai/docs/config/.
5. Launch OpenCode in the repo root and select the `core` agent (press Tab to cycle primary agents). The pipeline
   subagents are hidden from the `@mention` menu and invoked automatically by Core, so `core` is the primary entry point:

   ```bash
   opencode
   ```

## Usage

- For non-trivial features: describe the goal to Core. It routes through the full pipeline automatically and reports a
  Delivery Summary at the end.
- For trivial tasks (typo, rename, config toggle): Core routes directly to Build.
- Each subagent loads only its relevant skills on demand, produces a structured report, and Core checks a gate before
  advancing.
- When 3 or more independent slices exist, Core fans them out concurrently (parallel-dispatch).
- The pipeline subagents are hidden from the `@mention` menu by design — Core invokes them via the Task tool, which enforces the pipeline. Talk to Core to start a task.

**Example.** Send a message like this to Core:

> Add a CSV export button to the reports page that downloads the currently filtered rows.

Core triages this as non-trivial, dispatches `@define` to interview you about format, encoding, and edge cases (empty
result sets, large files), then routes the approved spec to `@planner`, which breaks it into slices (export utility, button
wiring, tests). `@implement` implements each slice, `@test` covers the acceptance criteria, and `@review` simplifies and
checks hot paths before Core returns a Delivery Summary.

## How It Works

### Gate criteria per stage

Each stage has explicit pass/fail criteria that Core checks before advancing:

- **Define gate** — problem statement present, measurable goals present, acceptance criteria present, spec status
  approved.
- **Plan gate** — at least one slice defined; each slice has a name, files, acceptance check, and dependencies; a risk
  register is present.
- **Build gate** — all slice acceptance criteria marked checked, tests passing, no failing tests.
- **Test gate** — all spec acceptance criteria covered by named tests, zero regressions, browser verification if UI
  slices are present.
- **Review gate** — simplification report present, performance report present, verdict approved or blocking items
  listed.

### Feedback loops

A failed gate never advances forward. Core routes back to the failing stage with specific feedback:

- Define to Plan fails: return to `@define` with clarifying questions.
- Plan to Build fails: return to `@planner` with scope concerns.
- Build to Test fails: return to `@implement` with the specific failing criteria.
- Test to Review fails: return to `@test` — Review never runs on broken code.
- Review to Done fails: apply suggestions, then re-test if behavior changes.

### Parallel dispatch

After Plan returns, Core builds a dependency graph from each slice's `Dependencies` field. When 3 or more independent
slices are present, Core loads the `parallel-dispatch` skill and fans those slices out to `@implement` concurrently. Slices
with dependencies remain sequential. The same applies to `@test` per Build level.

### Self-audit block

Core appends a protocol audit to every response so skipped steps are visible and recoverable:

```
---
PROTOCOL AUDIT:
  Skills loaded: [orchestration-protocol, parallel-dispatch if applicable]
  Stage: [current stage]
  Last subagent dispatched: [name + dispatch format used: structured/bare]
  Gate result: [passed / pending / failed — reason]
  Parallel: [yes — level N, tasks M | no]
  Next: [dispatch @X | await subagent | await user | done]
---
```

If any field indicates a skipped step, Core re-evaluates before continuing.

## Design Principles

- Core never writes code — it only thinks, routes, and verifies.
- Every stage has explicit gate criteria; Core never advances past a failed gate.
- Skills load on-demand, not all at once (context engineering).
- Thin vertical slices; each slice is testable end-to-end.
- Source-driven: verify external APIs against official docs, never from memory.
- Doubt-driven: adversarial review of every non-trivial decision.
- Measure-first performance optimization — no intuition-based changes.
- Every bug fix gets a regression test (reproduce, localize, fix, guard).

## Project Structure

```
agentic-workflow/
├── .opencode/
│   ├── agents/
│   │   ├── core.md        # mode: primary  — orchestrator
│   │   ├── define.md      # mode: subagent — requirements -> spec
│   │   ├── planner.md     # mode: subagent — spec -> task slices
│   │   ├── implement.md   # mode: subagent — implements slices
│   │   ├── test.md        # mode: subagent — verifies slices
│   │   └── review.md      # mode: subagent — simplifies + optimizes
│   └── skills/            # 17 SKILL.md files, one per subdirectory
│       ├── orchestration-protocol/SKILL.md
│       ├── parallel-dispatch/SKILL.md
│       ├── interview-me/SKILL.md
│       ├── idea-refine/SKILL.md
│       ├── spec-driven-development/SKILL.md
│       ├── planning-and-task-breakdown/SKILL.md
│       ├── incremental-implementation/SKILL.md
│       ├── source-driven-development/SKILL.md
│       ├── doubt-driven-development/SKILL.md
│       ├── context-engineering/SKILL.md
│       ├── frontend-ui-engineering/SKILL.md
│       ├── api-and-interface-design/SKILL.md
│       ├── test-driven-development/SKILL.md
│       ├── browser-testing-with-devtools/SKILL.md
│       ├── debugging-and-error-recovery/SKILL.md
│       ├── code-simplification/SKILL.md
│       └── performance-optimization/SKILL.md
├── .gitignore             # ignores /.idea/
├── opencode.json          # optional project config (model, permissions, $schema)
└── LICENSE                # MIT
```

## Configuration

This repo ships an `opencode.json` at the root for `$schema` validation and safe global permission defaults:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "permission": {
    "edit": "ask",
    "bash": "ask"
  }
}
```

It asks before any edit or bash command, and the per-agent `permission` blocks in each agent's frontmatter override these
defaults. A `model` field can be added here to set the default model (see the [OpenCode config
docs](https://opencode.ai/docs/config/)). Each agent's frontmatter also supports `model` and `steps` overrides (see the
[agents docs](https://opencode.ai/docs/agents/)); the subagents in this repo use `hidden: true` and `steps` limits so
they are invoked by Core via the Task tool rather than selected manually.

## Contributing

PRs are welcome. When adding new agents or skills, keep them consistent with the existing frontmatter format:

- **Agents** — `description`, `mode`, `temperature`, `permission` (plus optional `color`, `model`, `steps`, `hidden`). See
  the [agents docs](https://opencode.ai/docs/agents/).
- **Skills** — `name`, `description`, `license`, `compatibility`, `metadata`. See
  the [skills docs](https://opencode.ai/docs/skills/).

New subagents should declare which skills they may load in their `permission.skill` block, and new skills should declare
their `phase` and `agent` in `metadata` so they slot into the pipeline cleanly.

## License

MIT — see [LICENSE](./LICENSE).

Copyright (c) 2026 Nesar Ahmed.
