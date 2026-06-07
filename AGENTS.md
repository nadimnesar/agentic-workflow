# AGENTS.md

This file provides guidance to AI coding agents when working with code in this repository. The primary workflow is **OpenCode running as an ACP-integrated External Agent inside Zed**.

## Repository Overview

A collection of skills for OpenCode and Zed for senior software engineers. Skills are packaged instructions and scripts that extend OpenCode's capabilities.

---

## Zed + OpenCode Setup

### Installation

**Step 1 — Install OpenCode in Zed**

Open Zed's Agent Panel (`ctrl-?` or click ✨ in the status bar). Open Agent Settings and install OpenCode from the ACP Registry. OpenCode manages its own auth and model selection independently of Zed Agent.

**Step 2 — Install skills**

```bash
# Project-local (recommended — committed to Git)
cp -r skills/<skill-name> .opencode/skills/

# Global (available across all projects)
cp -r skills/<skill-name> ~/.config/opencode/skills/
```

**Step 3 — Wire up custom instructions**

AGENTS.md is auto-picked up by OpenCode. To pull in additional instruction files, add `opencode.json` at the project root:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "instructions": [
    "AGENTS.md",
    "docs/development-standards.md"
  ]
}
```

Commit both `AGENTS.md` and `opencode.json` to Git so the whole team shares them.

### Zed-specific Behavior

- OpenCode threads open from the **Agent Panel** or **Threads Sidebar** inside Zed
- Edits land in Zed's unified diff view — accept or reject before committing
- Multiple OpenCode threads can run in parallel, each against its own Git worktree (see [Parallel Agents](#parallel-agents))
- Zed's ACP architecture means **no agent traffic transits Zed's servers** — your code stays local
- AGENTS.md is read by OpenCode as project rules and by Zed Agent as an Instructions file; both honor it

---

## Skill-Driven Execution Model

OpenCode uses a **skill-driven execution model** powered by the `skill` tool and this repository's `skills/` directory. Skills are loaded on-demand — only the name and description are in context at startup; the full `SKILL.md` loads only when the agent decides the skill is relevant.

### Core Rules

- If a task matches a skill, you **MUST** invoke it via the `skill` tool
- Skills live at `skills/<skill-name>/SKILL.md`
- Never implement directly if a skill applies
- Always follow skill instructions exactly — do not partially apply them

### Intent → Skill Mapping

| User intent | Skill |
|---|---|
| Feature / new functionality | `spec-driven-development` → `incremental-implementation`, `test-driven-development` |
| Planning / breakdown | `planning-and-task-breakdown` |
| Bug / failure / unexpected behavior | `debugging-and-error-recovery` |
| Code review | `code-review-and-quality` |
| Refactoring / simplification | `code-simplification` |
| API or interface design | `api-and-interface-design` |
| UI work | `frontend-ui-engineering` |

### Lifecycle Mapping

OpenCode does not support slash commands like `/spec` or `/plan`. The agent internally follows this lifecycle:

| Phase | Skill |
|---|---|
| DEFINE | `spec-driven-development` |
| PLAN | `planning-and-task-breakdown` |
| BUILD | `incremental-implementation` + `test-driven-development` |
| VERIFY | `debugging-and-error-recovery` |
| REVIEW | `code-review-and-quality` |
| SHIP | `shipping-and-launch` |

### Execution Model

For every request:

1. Determine if any skill applies (even 1% chance)
2. Invoke the appropriate skill using the `skill` tool
3. Follow the skill workflow strictly
4. Only proceed to implementation after required steps (spec, plan, etc.) are complete

### Anti-Rationalization

The following thoughts are **incorrect** and must be ignored:

- "This is too small for a skill"
- "I can just quickly implement this"
- "I'll gather context first"

Correct behavior: always check for and use skills first.

---

## Parallel Agents

Zed supports running multiple OpenCode threads simultaneously, each in a dedicated Git worktree branched from HEAD. Use this for:

- Writing tests in one thread while refactoring in another
- Debugging a CI failure in parallel with a feature build
- Running `code-reviewer`, `security-auditor`, and `test-engineer` concurrently for the `/ship` lifecycle (fan-out → merge)

Open a new parallel thread from the Agent Panel. Each thread gets its own terminal and file context.

---

## Orchestration: Personas, Skills, and Commands

Three composable layers — don't confuse their roles:

- **Skills** (`skills/<name>/SKILL.md`) — workflows with steps and exit criteria. The *how*. Mandatory when intent matches.
- **Personas** (`agents/<role>.md`) — roles with a perspective and output format. The *who*.
- **Commands** (`.opencode/commands/*.md`) — user-facing entry points. The *when*. The orchestration layer.

**Composition rule: the user (or a command) is the orchestrator. Personas do not invoke other personas.** A persona may invoke skills.

The only endorsed multi-persona pattern is **parallel fan-out with a merge step** — used by `/ship` to run `code-reviewer`, `security-auditor`, and `test-engineer` concurrently, then synthesize their reports.

See [agents/README.md](agents/README.md) for the decision matrix and [references/orchestration-patterns.md](references/orchestration-patterns.md) for the full pattern catalog.

---

## Creating a New Skill

### Directory Structure

```
skills/
  {skill-name}/
    SKILL.md              # Required: skill definition
    scripts/              # Include if the skill ships runnable helpers
      {script-name}.sh
  {skill-name}.zip        # Packaged for distribution
```

### Skill Discovery Paths

OpenCode checks all of these on startup:

```
.opencode/skills/<name>/SKILL.md           # project-local (recommended)
~/.config/opencode/skills/<name>/SKILL.md  # global
.agents/skills/<name>/SKILL.md             # agent-compat project-local
~/.agents/skills/<name>/SKILL.md           # agent-compat global
```

### Naming Conventions

- **Skill directory**: `kebab-case` (e.g. `web-quality`)
- **SKILL.md**: always uppercase, always this exact filename
- **Scripts**: `kebab-case.sh` (e.g., `deploy.sh`, `fetch-logs.sh`)
- **Zip file**: must match directory name exactly: `{skill-name}.zip`

### SKILL.md Frontmatter

```yaml
---
name: {skill-name}        # 1–64 chars, lowercase alphanumeric + hyphens
description: >
  One sentence describing what the skill does.
  Use when: <trigger phrases>.
---
```

Only `name` and `description` are required. `license`, `compatibility`, and `metadata` (string map) are optional.

### SKILL.md Format

```markdown
---
name: {skill-name}
description: {One sentence + "Use when" trigger conditions.}
---

# {Skill Title}

{Brief overview of what the skill does and why it matters.}

## How It Works

{Numbered list explaining the workflow.}

## Usage (Optional)

Include only if the skill ships runnable helpers under scripts/.

\`\`\`bash
bash ~/.config/opencode/skills/{skill-name}/scripts/{script}.sh [args]
\`\`\`

**Arguments:**
- `arg1` - Description (defaults to X)

## Output

{Show example output.}

## Present Results to User

{Template for how the agent should format results.}

## Troubleshooting

{Common issues and solutions.}
```

### Best Practices for Context Efficiency

- **Keep SKILL.md under 500 lines** — move reference material to separate files
- **Write specific descriptions** — helps the agent know exactly when to activate
- **Use progressive disclosure** — reference supporting files read only when needed
- **Prefer scripts over inline code** — execution doesn't consume context (only output does)
- **File references work one level deep** — link directly from SKILL.md to supporting files

### Script Requirements

```bash
#!/bin/bash
set -e
echo "Running..." >&2           # status → stderr
echo '{"status": "ok"}'        # machine-readable output → stdout
trap 'rm -f /tmp/my-tmpfile' EXIT
```

Reference installed scripts as `~/.config/opencode/skills/{skill-name}/scripts/{script}.sh`.

### Creating the Zip Package

```bash
cd skills
zip -r {skill-name}.zip {skill-name}/
```

### Installation

**Project-local:**
```bash
cp -r skills/{skill-name} .opencode/skills/
```

**Global:**
```bash
cp -r skills/{skill-name} ~/.config/opencode/skills/
```

---

## Rules Precedence (OpenCode)

When OpenCode starts, it resolves instructions in this order:

1. Local `AGENTS.md` (walks up from CWD to git root)
2. Global `~/.config/opencode/AGENTS.md`
3. Any paths listed in `opencode.json` `instructions` field (merged with the above)

The first matching file wins per category. Commit your project `AGENTS.md` to Git so the whole team shares it.

---

## Zed Agent Panel Notes

In Zed v1.4+:

- Always-on project context (this file) is surfaced as **Instructions** in the Agent Panel
- Reusable, on-demand workflows are **Skills** (`skills/*/SKILL.md`) — these replaced the old Rules Library
- If you see "Rules" in older Zed UI, map it to **Instructions** for always-on context or **Skills** for on-demand workflows
