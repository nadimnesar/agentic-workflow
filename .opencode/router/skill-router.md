# Skill Router

## Purpose
Automatically selects the best skill(s) for a given user request. Supports multi-skill chaining, fallback logic, and precision-based dispatch.

## Classification Logic

The router classifies intent along two axes:
- **Domain**: The subject area (code-generation, bug-fixing, design, testing, etc.)
- **Action**: The type of work requested (create, fix, refactor, analyze, research)

### Intent-to-Skill Mapping

```
User Intent → Primary Skill → Optional Secondary Skills
─────────────────────────────────────────────────────────
"Build a new feature"        → code-generation   → testing
"Something is broken"        → bug-fixing        → testing
"Design the architecture"    → system-design     → api-design
"Improve this code"          → refactoring       → testing
"Add an API endpoint"        → api-design        → code-generation
"Make it more secure"        → security-review   → refactoring
"Write tests"                → testing           → code-generation
"How should I structure X"   → system-design     → api-design
"Research best practices"    → researcher agent  → system-design
```

## Routing Algorithm

1. **Extract intent** — Parse the user message for action verbs and domain nouns.
2. **Score skills** — Each skill produces a relevance score (0.0–1.0) based on keyword overlap and semantic similarity.
3. **Select primary** — The skill with the highest score above `threshold: 0.4`.
4. **Chain secondaries** — If the primary score > 0.7 AND other skills score > 0.3, chain them as secondary passes.
5. **Fallback** — If no skill scores above 0.4:
   - If the request is exploratory → route to `researcher` agent.
   - If the request is general → route to `planner` for decomposition.
   - If neither → respond with clarifying questions.

## Multi-Skill Chaining

When multiple skills match, execution follows this pattern:

```
Skill A (primary) → Output
    ↓
Skill B (secondary, optional) → Refines or extends output
    ↓
Skill C (tertiary, optional) → Validates or reviews output
```

Chain rules:
- Output of Skill A becomes input context for Skill B.
- Each skill in the chain operates on the accumulated artifact.
- The chain terminates after the final skill or when `max_chain_length: 3` is reached.
- If any skill in the chain produces an error, the chain halts and falls back to the last successful state.

## Fallback Handler

When no skill matches sufficiently, the router responds with:

```
I couldn't confidently map your request to a known skill.
Available skills: code-generation, bug-fixing, system-design,
                  api-design, refactoring, testing, security-review

Could you clarify one of:
• What you want to build (code-generation)
• What's broken (bug-fixing)
• What you want to design (system-design, api-design)
• What needs improvement (refactoring)
• What needs testing (testing)
• What needs security review (security-review)
```

## Router Configuration

```yaml
routing:
  threshold_primary: 0.4
  threshold_secondary: 0.3
  max_chain_length: 3
  fallback_to_researcher: true
  require_confirmation: false  # set to true to ask user before executing
  priority: precision          # precision | recall | balanced
```
