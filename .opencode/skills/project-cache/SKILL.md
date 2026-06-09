---
name: project-cache
description: Persistent caching of project context across sessions — specs, task plans, architectural decisions, resolved open questions, and intermediate outputs. Eliminates recomputation, reduces token consumption, and provides memory continuity between Core sessions. All agents read from and write to this cache via structured files.
license: MIT
compatibility: opencode
metadata:
  phase: orchestration
  agent: core
  load-order: "2"
---

# Project Cache

## What I Do

I provide a persistent, file-based knowledge store that survives across OpenCode sessions. Because OpenCode agents have no memory between sessions, this skill defines a structured set of files that act as the project's long-term memory. Every agent reads from the cache before working and writes to it after producing an output.

## Cache Location

All cache files live at:
```
.opencode/cache/
├── project.md          ← project-wide context (stack, conventions, key decisions)
├── specs/              ← approved specs, one file per feature
│   └── [feature-slug].md
├── plans/              ← approved task plans, one file per feature
│   └── [feature-slug].md
├── decisions/          ← architectural decisions (ADR format)
│   └── [slug].md
├── sessions/           ← per-session summaries for continuity
│   └── [YYYY-MM-DD-N].md
└── index.md            ← table of contents + cache health summary
```

---

## Cache Read Protocol

### Core: Start of Every Session

```
1. Read .opencode/cache/index.md
2. Read .opencode/cache/project.md
3. For the current task: check specs/ and plans/ for a cache hit
4. Pass relevant cache entries to subagents in the CONTEXT block
```

Core must never re-derive what is already in the cache. If a spec exists and is `status: approved`, use it. Do not re-run @define.

### Subagents: Before Every Task

Each subagent receives relevant cache entries via the Core dispatch CONTEXT block. Subagents do not read the cache directly — Core injects what they need. This preserves context budget.

---

## Cache Write Protocol

### After Each Pipeline Stage

| Stage completes | Core writes to |
|---|---|
| Define → approved spec | `specs/[feature-slug].md` |
| Plan → approved plan | `plans/[feature-slug].md` |
| Doubt-driven decision | `decisions/[slug].md` |
| Session ends | `sessions/[date-N].md` |
| New architectural fact | `project.md` (append) |

### Write Format

**`specs/[feature-slug].md`**
```markdown
---
feature: [human-readable name]
slug: [kebab-case identifier]
status: draft | approved | implemented
created: YYYY-MM-DD
updated: YYYY-MM-DD
---
[full spec content from Define output]
```

**`plans/[feature-slug].md`**
```markdown
---
feature: [name]
slug: [slug]
status: draft | approved | in-progress | complete
spec-ref: specs/[slug].md
created: YYYY-MM-DD
updated: YYYY-MM-DD
slices-total: N
slices-complete: N
---
[full plan content from Plan output]

## Slice Status
| Slice | Status | Tests |
|-------|--------|-------|
| 1. [name] | complete / in-progress / pending | passing |
```

**`decisions/[slug].md`** (ADR format)
```markdown
---
slug: [kebab-case]
date: YYYY-MM-DD
status: accepted | superseded | deprecated
supersedes: [slug or null]
---
# Decision: [title]

## Context
[why this decision was needed]

## Decision
[what was decided]

## Consequences
[what this enables and what it costs]

## Alternatives Rejected
[what was not chosen and why]
```

**`sessions/[date-N].md`**
```markdown
---
date: YYYY-MM-DD
session: N
---
## Work Done
[bullet summary of what happened]

## Decisions Made
[list with links to decisions/]

## Open Items
[what was left unfinished, for next session]

## Cache Updates
[which files were written]
```

**`project.md`** (append-only, Core updates this)
```markdown
# Project Context

## Stack
[language, frameworks, key libraries + versions]

## Conventions
[naming, file structure, test framework, linting rules]

## Architecture
[key modules, their responsibilities, boundaries]

## Known Constraints
[performance budgets, compliance requirements, non-negotiables]

## Resolved Questions
[questions that came up and were answered — with the answer]
```

**`index.md`** (Core maintains this as the directory)
```markdown
# Cache Index
Updated: YYYY-MM-DD

## Active Specs
| Feature | Status | Spec | Plan |
|---------|--------|------|------|
| [name]  | in-progress | specs/[slug].md | plans/[slug].md |

## Recent Decisions
| Decision | Date | Status |
|----------|------|--------|
| [title]  | date | accepted |

## Recent Sessions
| Date | Summary |
|------|---------|
| date | [one line] |

## Cache Health
- Project.md last updated: [date]
- Total specs: N (approved: N, draft: N)
- Total decisions: N
- Sessions recorded: N
```

---

## Cache Hit Logic

Before dispatching @define, Core checks:

```
cache_hit = (
  .opencode/cache/specs/[feature-slug].md exists
  AND status = "approved"
  AND the current request maps to this feature
)
```

If `cache_hit = true`: skip @define, pass the cached spec to @plan.

Before dispatching @plan, Core checks:
```
plan_hit = (
  .opencode/cache/plans/[feature-slug].md exists
  AND status = "approved"
  AND slices-complete < slices-total  (i.e., work is not done)
)
```

If `plan_hit = true`: resume from the first incomplete slice.

---

## Token Budget Management

The cache prevents redundant token spend by:

1. **Spec reuse**: A 500-token spec in cache costs 0 tokens to re-derive.
2. **Decision reuse**: Past decisions are injected as 2–3 line summaries, not re-reasoned.
3. **Session continuity**: The session summary (< 200 tokens) replaces a full conversation replay.
4. **Selective injection**: Core injects only the cache sections relevant to the current subagent's task — not the entire cache.

### Injection budget per subagent dispatch:
| Content | Max tokens to inject |
|---|---|
| Project.md summary | 300 |
| Relevant spec | 800 |
| Relevant plan (current slice only) | 400 |
| Relevant decisions | 200 (2–3 line summaries) |
| Last session summary | 150 |
| **Total injected context** | **~1850 tokens max** |

If the relevant cache content exceeds this budget, summarize — do not inject the full file.

---

## Cache Initialization

If `.opencode/cache/` does not exist, Core creates it on first run:

```bash
mkdir -p .opencode/cache/specs .opencode/cache/plans \
         .opencode/cache/decisions .opencode/cache/sessions
```

Then writes a minimal `project.md` and `index.md` based on a quick repo scan (file structure, package.json/go.mod/Cargo.toml, README).

---

## Cache Invalidation

A cache entry is stale when:
- The spec was written for a different version of the request (user changed requirements)
- A decision was superseded by a newer decision
- A completed plan has all slices done (archive it, don't re-use)

Stale entries: update `status` to `superseded` or `archived`. Do not delete — they are historical record.
