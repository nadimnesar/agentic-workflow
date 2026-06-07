---
name: system-design
description: Designs high-level system architectures, component diagrams, data flows, and trade-off analyses. Use when building new systems, adding major subsystems, or evaluating architectural alternatives.
---

# Skill: system-design

## Purpose
Design high-level system architectures, component diagrams, data flows, and trade-off analyses. Produces structured design documents suitable for review and implementation.

## When to Use
- Building a new system from scratch
- Adding a major subsystem to an existing codebase
- Evaluating architectural alternatives
- Designing for scale, reliability, or compliance
- Creating an ADR (Architecture Decision Record)
- Planning migration from one architecture to another

## Input Format
```yaml
requirements:
  functional:
    - "users can create and share documents"
    - "documents support real-time collaboration"
  non_functional:
    - "99.9% uptime"
    - "p99 latency < 200ms for read operations"
    - "support 10K concurrent users"
  constraints:
    - "must use existing PostgreSQL instance"
    - "team has 5 engineers"
    - "must be deployable within 3 months"
  context:
    - "existing monolith at src/backend/"
    - "current auth via session cookies"
```

## Output Format
```yaml
architecture:
  overview: "1-2 paragraph summary of the design"
  diagrams:
    - type: "c4_context" | "c4_container" | "sequence" | "data_flow"
      description: "..."
      ascii_art: "..."
  components:
    - name: "api-gateway"
      responsibility: "rate limiting, auth, routing"
      technology: "nginx + lua"
    - name: "document-service"
      responsibility: "CRUD + real-time sync"
      technology: "go + websocket"
  data_model:
    entities:
      - name: "Document"
        fields: [id, title, content, owner_id, created_at, updated_at]
  trade_offs:
    - decision: "Use WebSockets instead of SSE"
      rationale: "Bidirectional communication needed for collaborative editing"
      cost: "More complex horizontal scaling"
  adr_links:
    - "docs/adrs/2024-01-websockets-for-collab.md"
```

## Execution Steps
1. **Requirements analysis** — Clarify functional and non-functional requirements.
2. **Scope boundaries** — Define what is in and out of scope.
3. **High-level design** — Identify major components and their responsibilities.
4. **Data model** — Design entities, relationships, and data flow.
5. **Interface contracts** — Define APIs, event schemas, and integration points.
6. **Trade-off analysis** — For each design decision, document alternatives considered and rationale.
7. **Review preparation** — Format as a review-ready document with open questions.

## Examples

### Example 1: URL shortener
**Input:** "Design a URL shortener like bit.ly. 100M URLs, 10M redirects/day."
**Output:** Design with write-sharded PostgreSQL, Redis cache layer, base62 encoding, and CQRS pattern.

### Example 2: Real-time dashboard
**Input:** "Design a real-time analytics dashboard with sub-second refresh."
**Output:** Event-sourced pipeline with Kafka → Flink → Materialized view in Redis → WebSocket push to dashboard.
