---
name: parallel-dispatch
description: Enable concurrent subagent execution for independent tasks. Allows Core to fan out multiple Build slices or parallel investigations simultaneously, then aggregate results before advancing the pipeline. Enforces dependency order and prevents race conditions between agents.
license: MIT
compatibility: opencode
metadata:
  phase: orchestration
  agent: core
  load-order: "3"
---

# Parallel Dispatch

## What I Do

I allow Core to run multiple subagent tasks concurrently when those tasks have no dependency on each other. This reduces
total wall-clock time for multi-slice features without sacrificing the sequential guarantees that correctness depends
on.

## When Parallelism Is Safe

Parallelism is safe when tasks have **no shared mutable state** and **no output dependency**.

### Safe to parallelize:

- Multiple `@explore` / `@scout` investigations with no overlapping files
- `@define` interviews for independent sub-features (if the top-level feature was decomposed)
- `@build` slices that touch **completely separate** files/modules
- `@test` running different test suites for independent slices that are already complete
- `@review` reviewing independently built modules

### Never parallelize:

- Any two tasks that write to the same file
- `@build` slice N+1 before `@build` slice N is complete and tested
- `@test` for a slice while `@build` is still working on it
- `@review` while `@test` has failing tests
- Any pair of tasks where task B depends on task A's output

---

## The Dependency Graph

Before dispatching any tasks in parallel, Core must build an explicit dependency graph from the task plan.

### Step 1: Extract dependency map

From the Plan output, read each slice's `Dependencies` field:

```
Slice 1: deps=[]           → no dependencies, can start immediately
Slice 2: deps=[Slice 1]    → must wait for Slice 1
Slice 3: deps=[Slice 1]    → must wait for Slice 1 (can run WITH Slice 2)
Slice 4: deps=[Slice 2, 3] → must wait for both 2 and 3
Slice 5: deps=[]           → no dependencies, can start immediately with Slice 1
```

### Step 2: Identify parallelizable groups

Topological sort → each "level" in the graph is a parallelizable group:

```
Level 0 (parallel): Slice 1, Slice 5
Level 1 (parallel): Slice 2, Slice 3  (after level 0 complete)
Level 2 (serial):   Slice 4           (after level 1 complete)
```

### Step 3: Dispatch each level concurrently

Use the Task tool to dispatch all slices in a level simultaneously. OpenCode's task tool creates independent child
sessions that run concurrently.

---

## Parallel Dispatch Format

When dispatching multiple tasks in parallel, Core uses this format:

```
PARALLEL_DISPATCH: level=[N] tasks=[count]

TASK_1:
  agent: @build
  slice: [slice name]
  cache_key: plans/[slug].md → slice [N]
  context: [relevant cache + spec for this slice only]
  files_written: [expected files — for conflict detection]
  
TASK_2:
  agent: @build
  slice: [other slice name]
  cache_key: plans/[slug].md → slice [M]
  context: [relevant cache + spec for this slice only]
  files_written: [expected files — must not overlap with TASK_1]

SYNCHRONIZATION_POINT:
  wait_for: [TASK_1, TASK_2]
  aggregate_before: level=[N+1]
  on_failure: halt_parallel, escalate_to_core
```

---

## Conflict Detection

Before dispatching parallel tasks, Core must verify no file conflicts:

```
conflict = any file appears in more than one task's files_written list
```

If a conflict is detected:

- Split the conflicting tasks into separate sequential levels
- Do not attempt to parallelize tasks that share write targets

Example:

```
Task A writes: src/auth/token.ts
Task B writes: src/auth/token.ts  ← CONFLICT
→ Make B depend on A (serialize them)

Task A writes: src/auth/token.ts
Task B writes: src/user/profile.ts  ← no conflict
→ Safe to parallelize
```

---

## Aggregation Protocol

When all tasks in a parallel level complete, Core:

1. **Collects all outputs** from child sessions
2. **Checks all gates** — every task must have passed its gate before the next level starts
3. **Merges cache writes** — each task may have written to a different cache key; Core updates index.md
4. **Runs cross-task integration check** — if tasks touched different modules, confirm their interfaces are compatible
   before proceeding
5. **Dispatches next level** or reports completion

### Aggregation report format (Core produces this internally):

```
LEVEL [N] AGGREGATION:
  Tasks completed: [N/N]
  Gates passed: [list]
  Gates failed: [list — halt if any]
  Files written: [aggregate list]
  Conflicts detected post-hoc: [none / list]
  Interface compatibility: [checked / issues found]
  Proceeding to: Level [N+1] / done
```

---

## Parallel Test Execution

After a parallel Build level completes, Test can also run in parallel per-slice:

```
PARALLEL_TEST_DISPATCH:
  for each completed slice in level N:
    dispatch @test with:
      - the slice's acceptance criteria
      - the specific files changed by that slice
      - its own test files
      
  synchronize: all slice tests must pass before Review
```

Test suites for independent slices can be run concurrently because:

- Each slice has its own test files
- Independent test suites don't share mutable state (assuming proper test isolation)
- A failure in one slice's tests doesn't affect another slice's test run

---

## Parallel Investigation Pattern

For `@explore` and `@scout` tasks (read-only), parallelism is almost always safe:

```
PARALLEL_INVESTIGATION:
  questions:
    - "How does the current auth module handle token refresh?"
    - "What test patterns are used in the payments module?"
    - "Are there existing rate-limiting utilities?"

  dispatch concurrently:
    @explore → question 1 → cache: decisions/auth-token-refresh-pattern.md
    @explore → question 2 → cache: decisions/test-patterns-observed.md
    @scout   → question 3 → cache: decisions/rate-limiting-utilities.md

  aggregate: inject all three findings into @plan context
```

This pattern is especially useful at the start of Plan, where understanding multiple areas of the codebase
simultaneously is both safe and faster.

---

## When to Use This Skill

Load this skill when:

- The task plan has 3+ slices and at least 2 have no dependency between them
- Core needs to investigate multiple independent areas before Planning
- A feature has clearly independent frontend and backend slices

Do not load this skill when:

- The task is a single slice
- All slices have sequential dependencies
- The task is ambiguous and needs full Define first (parallelism introduces confusion when requirements aren't settled)

---

## Failure Handling in Parallel Execution

If one task in a parallel level fails its gate:

1. **Do not cancel sibling tasks** that are still running — let them complete
2. **Do not advance to the next level** — a level is atomic
3. After all tasks complete: report all failures to Core
4. Core routes failing tasks back to their respective agents
5. Re-run only the failed tasks, not the whole level

```
PARALLEL_FAILURE_PROTOCOL:
  failed_tasks: [list]
  passing_tasks: [list — their work is preserved]
  action: re-dispatch failed_tasks only
  next_level: blocked until all tasks in this level pass
```
