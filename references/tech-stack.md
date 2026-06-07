# Tech Stack Reference

Canonical technology preferences for this project. All agents and skills default
to these technologies unless the task explicitly specifies otherwise.

## Languages

| Domain | Language | Version Constraints |
|---|---|---|
| Backend | Java | 17+ (prefer 21 LTS) |
| Backend (future) | Golang | 1.22+ |
| Frontend | TypeScript | 5.x with Angular |
| Database | SQL | PostgreSQL dialect |
| Build | SQL, Shell | POSIX-compatible |

## Backend Framework

| Concern | Choice | Notes |
|---|---|---|
| REST API | Spring Boot 3.x | WebFlux for reactive, WebMvc for traditional |
| DI / IoC | Spring Context | Annotation-based configuration |
| Data Access | Spring Data JPA (Hibernate) + JDBC template | JPA for aggregates, JDBC for bulk ops |
| Migrations | Liquibase | XML/YAML changelogs, not SQL files |
| Testing | JUnit 5 + Mockito + AssertJ | JUnit for structure, Mockito for mocking, AssertJ for fluent assertions |
| API Docs | SpringDoc OpenAPI 3 | Swagger UI in dev profile |
| Validation | Jakarta Validation (Hibernate Validator) | Bean validation annotations |
| Mapping | MapStruct / manual | MapStruct for DTO<->Entity, avoid heavy reflection mappers |

## Frontend

| Concern | Choice | Notes |
|---|---|---|
| Framework | Angular 17+ | Standalone components, signals where appropriate |
| State Mgmt | NgRx or Angular services | NgRx for complex state, services for simple |
| Styling | Angular Material + SCSS | Or Tailwind if team prefers |
| Testing | Jasmine + Karma / Jest | Prefer Jest for new projects |

## Data Stores

| Store | Purpose | Key Considerations |
|---|---|---|
| PostgreSQL | Primary relational store | JSONB for flexible schemas, proper indexes |
| DynamoDB | High-scale key-value / session | Single-table design preferred |
| Elasticsearch | Full-text search, analytics | Index mappings matter, plan ahead |
| Redis | Caching, rate limiting, pub/sub | Lettuce connection pooling |

## Messaging

| System | Use Case | Notes |
|---|---|---|
| Kafka | Event-driven / streaming | Record keys for partitioning, idempotent producers |
| RabbitMQ | Task queues, RPC | Dead letter queues for failures |

## Cloud / DevOps

| Service | Purpose | Notes |
|---|---|---|
| EC2 | Compute | Use ASG with launch templates |
| RDS | Managed PostgreSQL | Prefer Aurora for high availability |
| SQS | Async job queue | Visibility timeout == 3x processing time |
| S3 | Object storage | Lifecycle policies for cost management |
| SSM | Parameter store, secrets | Tier: Standard (free) or Advanced |
| CloudWatch | Logging, metrics, alarms | Structured logging (JSON) |
| Kubernetes | Container orchestration | EKS, prefer deployments over StatefulSets |
| Docker | Containerization | Multi-stage builds, distroless images |
| Nginx | Reverse proxy, load balancing | Also serves as k8s ingress |
| MinIO | S3-compatible local storage | For dev/test environments |

## Project Conventions

```
Group ID:         com.<company>.<project>
Package:          com.<company>.<project>.<domain>
Module structure:  <project>-common, <project>-api, <project>-service, <project>-worker
Config:           application-{profile}.yml  (YAML preferred over properties)
```

## CI / Build

| Tool | Use |
|---|---|
| Maven | Primary build (prefer Maven wrapper) |
| Gradle | Alternative build (used by some teams) |

## Methodology

- Scrum with 2-week sprints
- Microservices architecture (bounded contexts)
- Event-driven architecture (Kafka for domain events)
- Test-Driven Development (red-green-refactor)

## Default Testing Stack

| Layer | Tool | Notes |
|---|---|---|
| Unit tests | JUnit 5 + Mockito + AssertJ | Mock external dependencies |
| Integration tests | @SpringBootTest + Testcontainers | Use test slices (@WebMvcTest, @DataJpaTest) |
| Contract tests | Spring Cloud Contract | For inter-service contracts |
| E2E tests | Playwright / Cypress | Angular E2E |

## Quality Gates

| Gate | Threshold |
|---|---|
| Test coverage | >= 80% (line), >= 70% (branch) |
| SonarQube quality gate | Pass |
| Checkstyle | No violations at error level |
| Build | Clean, no warnings as errors |
