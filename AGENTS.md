# Agentic Workflow System

This file defines the core agentic workflow lifecycle for this project. All agents
MUST read and follow these rules when operating.

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
The skill router classifies the user's intent and selects the appropriate skill(s).
If no skill matches sufficiently, it falls back to the researcher agent for context-
gathering, then replans.

### Phase 2: Plan
The planner decomposes the request into ordered tasks with a dependency graph.
Each task is assigned a skill and an agent type. Parallelization opportunities are
identified at this stage.

### Phase 3: Build
The builder executes each task using the assigned skill. Tasks within the same
dependency layer run in parallel when `parallel.enabled` is true. Each task
produces an artifact (code, tests, docs).

### Phase 4: Test
The tester validates each artifact against acceptance criteria. Tests are added
or updated. Bugs found during testing are reported back to the builder for
iteration.

### Phase 5: Review
The reviewer evaluates all artifacts across correctness, maintainability,
performance, security, and test quality. If changes are requested, the loop
returns to the builder.

---

## Core Principles

### 1. Autonomy
Agents operate autonomously by default. Do not ask the user for confirmation on
routine decisions (naming, file placement, library choice within established
patterns). Escalate to the user only when:
- The request is ambiguous and cannot be resolved from context.
- A decision has irreversible consequences (data loss, security, public API breakage).
- Two equally valid approaches exist and neither is clearly better.

### 2. Minimal Interruption
Batch questions into a single message. Prefer to make a reasonable default choice
and note it for the user's review rather than blocking on every decision.

### 3. Iterative Refinement
Each phase may trigger a loop to a previous phase:
- Tester finds a bug → Builder fixes it → Tester re-verifies.
- Reviewer requests changes → Builder implements → Reviewer re-reviews.
- After `max_iterations` (from execution config) without resolution, escalate to
  the user with a summary of the impasse.

### 4. Skill-First Execution
Each task references a skill. Before executing, load the skill's SKILL.md and
follow its execution steps precisely. If a task calls for a skill that does not
exist, fall back to the researcher + planner to define a reasonable approach.

---

## Context Management Strategy

### Context Preservation
- The original user request is preserved in full throughout the entire lifecycle.
- Each agent receives only the context slice it needs (the relevant subset of
  files, requirements, and previous outputs).
- Architecture decisions and acceptance criteria are carried forward across all
  phases.

### Context Compression
When approaching the token limit:
1. Trim low-signal conversation history (back-and-forth that did not change the plan).
2. Summarize completed tasks into their input/output contracts (discard internal
   reasoning).
3. Drop file contents that have been read but not modified.
4. Preserve: acceptance criteria, architecture decisions, current task scope.

### Handoff Protocol
When one agent passes work to another:
```
Previous Agent Output (as Input Contract for next agent)
    │
    ▼
Next Agent reads input contract, loads relevant files, executes.
    │
    ▼
Next Agent produces output contract and passes forward.
```
Each handoff includes only the information the next agent needs, not the full
conversation history.

---

## Parallel Execution Rules

### When to Parallelize
- Tasks in the same dependency layer (no inter-task dependencies).
- Tasks that modify different files (avoid merge conflicts).
- Tasks that are conceptually independent (e.g., writing tests for two unrelated
  modules).

### When NOT to Parallelize
- Tasks that modify the same file (risk of conflicting edits).
- Tasks where one produces context that another needs.
- When the effort level is `low` (serial is simpler and avoids coordination overhead).

### Merging Parallel Outputs
- Each parallel task produces its own output contract.
- The planner collects all outputs and merges them in sequence (order defined by
  `merge_strategy` in execution config).
- If two tasks produce conflicting changes to the same file, the merge strategy
  (`diff-merge` or `conflict-resolution`) is applied.

---

## Tool Usage Policy

### Internet Search
- Use only when the task requires information not in the model's training data.
- Always cite sources.
- Cache results to avoid redundant lookups within a session.

### Code Execution
- Run tests, linters, and type-checkers after every change.
- Use the sandbox for untrusted code execution.
- Never execute code that modifies system files or environment outside the project.

### File Operations
- Read before you write — understand existing code before modifying it.
- Prefer small, targeted edits over rewriting entire files.
- Clean up temporary files after use.

---

## Quality Gates

Every artifact must pass these gates before being marked complete:

1. **Syntax check** — No parse errors in generated code.
2. **Lint** — No lint errors (follow project lint rules).
3. **Type check** — No type errors (when type checking is configured).
4. **Tests pass** — All existing tests still pass.
5. **Acceptance criteria met** — Each criterion is validated.
6. **No regressions** — Behavior of unchanged code is preserved.

---

## Error Handling

- If a task fails: document why, roll back partial changes, and report to the
  planner for re-planning.
- If a dependency is missing: attempt to resolve it (install package, create file)
  before reporting failure.
- If the model refuses to generate certain content: document the refusal reason
  and suggest an alternative approach.
- If an external tool or API is unavailable: retry with exponential backoff
  (3 retries max), then fall back to an alternative approach.
