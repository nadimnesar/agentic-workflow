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
    - "preferred stack: Java 21, Spring Boot, Kafka, AWS"
  context:
    - "existing monolith at src/main/java/com/example/"
    - "current auth via JWT tokens"
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
      technology: "Spring Cloud Gateway + Nginx"
    - name: "document-service"
      responsibility: "CRUD + real-time sync"
      technology: "Spring Boot 3 + WebSocket + Kafka"
    - name: "notification-service"
      responsibility: "email/push notifications"
      technology: "Spring Boot + RabbitMQ"
  data_model:
    entities:
      - name: "Document"
        storage: "PostgreSQL (JPA entity)"
        search: "Elasticsearch index"
        cache: "Redis (read model)"
        fields: [id, title, content, owner_id, created_at, updated_at]
  infrastructure:
    compute: "EKS (Kubernetes on AWS)"
    storage: "RDS Aurora + ElastiCache Redis + OpenSearch"
    messaging: "Amazon MSK (Kafka) + SQS"
    object_store: "S3 with CloudFront CDN"
  trade_offs:
    - decision: "Use Kafka over direct WebSocket"
      rationale: "Event-driven decoupling, replay capability, multiple consumers"
      cost: "Higher operational complexity, added latency (~10ms)"
  adr_links:
    - "docs/adrs/2024-01-kafka-for-domain-events.md"
```

## Execution Steps
1. **Requirements analysis** — Clarify functional and non-functional requirements. Refer to `references/tech-stack.md` for default technology preferences.
2. **Scope boundaries** — Define what is in and out of scope.
3. **High-level design** — Identify major components and their responsibilities.
4. **Data model** — Design entities, relationships, and data flow (JPA entities, indexes, search mappings).
5. **Interface contracts** — Define REST APIs (SpringDoc OpenAPI), event schemas (Avro/JSON for Kafka), and integration points.
6. **Infrastructure decisions** — Compute (EKS/EC2), storage (RDS/DynamoDB/S3), messaging (Kafka/RabbitMQ/SQS).
7. **Trade-off analysis** — For each design decision, document alternatives considered and rationale.
8. **Review preparation** — Format as a review-ready document with open questions.

## Examples

### Example 1: URL shortener
**Input:** "Design a URL shortener. Stack: Java 21, Spring Boot, PostgreSQL, Redis, AWS. 100M URLs, 10M redirects/day."
**Output:** Design with Spring Boot REST API, write-sharded PostgreSQL (RDS Aurora), Redis cache layer with Caffeine L2 cache, base62 encoding, Liquibase migrations, deployed on EKS with HPA.

### Example 2: Real-time dashboard
**Input:** "Design a real-time analytics dashboard. Stack: Spring Boot, Kafka, Elasticsearch, Angular. Sub-second refresh."
**Output:** Event-sourced pipeline with Spring Kafka producer → Kafka → Kafka Streams aggregation → Elasticsearch → Spring Boot read API → Angular with SSE push.
