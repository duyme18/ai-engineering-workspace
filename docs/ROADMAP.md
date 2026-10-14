# Roadmap — AI Engineering Workspace

> Last updated: 2026-10-14
> Rule: a milestone starts only after the previous milestone's gate is **PASS**.
> Rule: technologies from future milestones are not introduced early unless the owner decides so.

## Status Overview

| Milestone | Name | Status |
| --- | --- | --- |
| M0 | Product & Architecture | ✅ PASS (2026-10-15) |
| M1 | Project Foundation | 🟡 Next — planning |
| M2 | Authentication & Authorization | ⚪ Not started |
| M3 | Workspace (Project, Member, Task, Note) | ⚪ Not started |
| M4 | Knowledge Base (upload & parsing) | ⚪ Not started |
| M5 | RAG | ⚪ Not started |
| M6 | AI Assistant | ⚪ Not started |
| M7 | AI Tools / Agent | ⚪ Not started |
| M8 | MCP | ⚪ Not started |
| M9 | Event-Driven Architecture (Kafka) | ⚪ Not started |
| M10 | Observability | ⚪ Not started |
| M11 | Testing hardening | ⚪ Not started |
| M12 | CI/CD | ⚪ Not started |
| M13 | Containerization | ⚪ Not started |
| M14 | Kubernetes | ⚪ Not started |
| M15 | Production Hardening | ⚪ Not started |
| M16 | Final (docs, demo, lessons learned) | ⚪ Not started |

MVP = M1–M7. See [PRD §4](product/PRD.md).

---

## Technology Selection

| Area | Choice | Why | Main alternative | Introduced |
| --- | --- | --- | --- | --- |
| Backend | Java 25 (LTS) + Spring Boot 4 | Owner's core stack; LTS; virtual threads; Boot 4 = current major | Kotlin, Quarkus | M1 |
| Build | Maven | Owner familiarity; predictable | Gradle | M1 |
| Persistence | Spring Data JPA / Hibernate | CRUD-heavy domain; fast to build | jOOQ (better for complex SQL) | M1/M3 |
| Migrations | Flyway | Plain SQL, simple versioning ([ADR-0007](adr/0007-flyway-migrations.md)) | Liquibase | M1 |
| Database | PostgreSQL | Relational core + pgvector + jsonb in one engine ([ADR-0004](adr/0004-postgresql-pgvector.md)) | MySQL | M1 |
| Frontend | Next.js + TypeScript + Tailwind + shadcn/ui | Industry standard; server components; good DX | Vite + React SPA | M1 |
| FE data | TanStack Query + Zod | Server-state cache; runtime validation | SWR | M3 |
| Identity | Keycloak (OIDC/OAuth2) | Real IdP, self-hosted, standard protocols ([ADR-0003](adr/0003-identity-keycloak.md)) | Auth0, Spring Authorization Server | M2 |
| API docs | springdoc-openapi | Generated from code, stays in sync | Hand-written OpenAPI | M3 |
| File storage | Local FS behind `FileStorage` interface → maybe MinIO/S3 | Decide at M4 | MinIO from day one | M4 |
| Vector store | pgvector | Same DB, transactional with metadata, enough for scale | Qdrant, Weaviate | M5 |
| AI framework | LangChain4j | Java-native RAG/tools/agents ([ADR-0005](adr/0005-ai-framework-langchain4j.md)) | Spring AI | M5 |
| LLM access | OpenAI-compatible API | Swap providers (OpenAI, local Ollama, others) via config | Provider-specific SDK | M5 |
| Streaming | SSE | One-way server→client stream; simple over HTTP ([ADR-0006](adr/0006-api-style-rest-sse.md)) | WebSocket | M6 |
| External tools | MCP | Standard protocol for tool servers | Custom REST integrations | M8 |
| Messaging | Apache Kafka | Durable log, consumer groups, replay | RabbitMQ, DB-backed queue | M9 |
| Observability | OpenTelemetry + Prometheus + Grafana | Vendor-neutral standard | Vendor APM | M10 |
| Testing | JUnit 5, Mockito, Spring Boot Test, Testcontainers | Real DB in tests, no H2 mismatch | H2 in-memory | M1+ |
| CI/CD | GitHub Actions, SonarCloud | Free for public repos, standard | GitLab CI, Jenkins | M1 minimal, M12 full (D-06) |
| Runtime | Docker, Docker Compose → Kubernetes + Helm | Local parity → production-like deploy | Docker Swarm, plain manifests | M1 / M14 |
| Cache / rate limit | Redis | **Only when a use case exists** (likely M15) | In-memory (Caffeine), Bucket4j | M15 (tentative) |

Exact versions are pinned at M1 to the latest stable releases and recorded in `PROJECT-STATE.md`.

---

## Milestones & Acceptance Criteria

### M0 — Product & Architecture

Goal: a foundation clear enough that later milestones can be implemented sequentially without losing context.

Acceptance criteria (Definition of Done for M0):

- [x] PRD: vision, users, use cases, MVP scope, out of scope, user flows, NFRs — reviewed by owner (2026-10-15).
- [x] Architecture doc: context diagram, module list, dependency rules, key flows, risks — reviewed by owner (2026-10-15).
- [x] Technology selection with reason + alternative for each choice.
- [x] Repository structure defined.
- [x] Initial database design (M2–M3 detailed, M4+ high-level).
- [x] Initial API design: conventions, error format, pagination, endpoint inventory.
- [x] ADRs 0001–0007 Accepted (2026-10-15).
- [x] Decisions D-01…D-09 resolved (2026-10-15, see `PROJECT-STATE.md`).
- [x] `ROADMAP.md`, `PROJECT-STATE.md`, `LEARNING-LOG.md` exist and are current.
- [x] Owner can explain: why modular monolith, module dependency rules, why authorization lives in the app DB and not Keycloak, why RAG filtering by project is a security control. (Learning check passed 2026-10-15.)
- [x] No application code written.

### M1 — Project Foundation

Goal: frontend, backend and database run locally and talk to each other.

Acceptance criteria:

- [ ] `docker compose up` starts PostgreSQL (with healthcheck).
- [ ] Backend (Java 25, Spring Boot 4, Maven) starts with profile `local`, connects to DB, Flyway runs `V1`.
- [ ] `GET /actuator/health` returns `UP` including DB.
- [ ] `GET /api/v1/system/info` works; frontend page displays it (FE → BE → DB path proven).
- [ ] Global error handler returns RFC 9457 Problem Details; unknown route → JSON 404.
- [ ] CORS configured with explicit allow-list.
- [ ] Correlation id per request in logs.
- [ ] Next.js + TypeScript + Tailwind + shadcn/ui skeleton with a layout and one page.
- [ ] Module boundary test exists (Spring Modulith `ApplicationModules.verify()`, test scope), even with only `shared`.
- [ ] At least: one Spring Boot integration test with Testcontainers PostgreSQL; one frontend build check.
- [ ] Minimal CI: GitHub Actions running `mvn verify` + frontend `npm run build` on every push/PR.
- [ ] `development-guide.md` §3 explains local setup; fresh clone → running in < 15 min.

### M2 — Authentication & Authorization

Acceptance criteria:

- [ ] Keycloak in Compose with **reproducible realm import** (clients, realm roles `ADMIN`/`USER`, test users).
- [ ] Frontend login/logout via OIDC (Authorization Code + PKCE); auth pattern documented in an ADR.
- [ ] Backend is an OAuth2 Resource Server: validates signature, issuer, audience, expiry.
- [ ] `GET /api/v1/me` returns user; JIT provisioning creates/updates `app_user`.
- [ ] Unauthenticated → 401 Problem Details; insufficient role → 403.
- [ ] Protected frontend routes redirect to login.
- [ ] Tests: valid token, expired/invalid token, missing token, role check.
- [ ] Owner can explain: OIDC vs OAuth2, access vs ID vs refresh token, JWKS, PKCE, where tokens live in the browser.

### M3 — Workspace

Acceptance criteria:

- [ ] Projects, members, tasks, notes per [API design](api-design.md) with project-level RBAC.
- [ ] Pagination, allow-listed sorting, filtering on list endpoints.
- [ ] Bean Validation + business rules (BLOCKED requires reason, last OWNER cannot be removed, valid transitions).
- [ ] Optimistic locking → 409; non-member → 404.
- [ ] OpenAPI available; frontend pages for projects, tasks, notes.
- [ ] Module boundaries respected (boundary test green).
- [ ] Tests: service unit tests for rules; API tests for authz matrix; repository tests for filters.

### M4 — Knowledge Base

Acceptance criteria:

- [ ] Upload PDF/MD/DOCX/TXT ≤ 20 MB; type validated by content; duplicate detected.
- [ ] Pipeline Upload → Store → Parse → Save content & metadata; status visible; FAILED with reason; reprocess.
- [ ] Delete removes file + DB rows.
- [ ] No DB transaction held during file I/O / parsing.
- [ ] Storage decision recorded (ADR).
- [ ] Tests: each parser, invalid file, oversized file, failure path.

### M5 — RAG

Acceptance criteria:

- [ ] Chunk → embed → store in pgvector; HNSW index.
- [ ] Retrieval strictly filtered by `project_id` (test proves no cross-project leakage).
- [ ] `ask` returns answer + sources; says "not found" when nothing relevant.
- [ ] `search` endpoint for inspecting retrieval.
- [ ] Small evaluation set (≈ 10–20 questions with expected sources) to judge chunking/top-K changes.
- [ ] Owner can explain: embeddings, cosine similarity, chunk size/overlap trade-offs, HNSW, why RAG over fine-tuning.

### M6 — AI Assistant

Acceptance criteria:

- [ ] Conversations & messages persisted; history loaded; per-user isolation.
- [ ] Streaming over SSE end-to-end, including error events and client cancel.
- [ ] Memory strategy defined (window/summarization) with token budget.
- [ ] Token usage & latency stored per message.
- [ ] Structured output used where the UI needs structure (e.g. citations).
- [ ] Timeouts on LLM calls; provider errors mapped to Problem Details / SSE error.

### M7 — AI Tools / Agent

Acceptance criteria:

- [ ] Read tools: getProject, getTasks, getBlockedTasks, searchDocuments, getProjectNotes.
- [ ] Write tool createTask requires confirmation (PROPOSED → CONFIRMED/REJECTED).
- [ ] Tools execute with the user's permissions (test: VIEWER cannot create via AI).
- [ ] Max iterations / timeouts; every execution audited.
- [ ] UI shows tool activity and confirmation card.

### M8 — MCP

Acceptance criteria:

- [ ] Backend acts as MCP client to ≥ 1 external server (GitHub first), configured by env.
- [ ] Read-only by default; credentials scoped minimally; integration status endpoint.
- [ ] Assistant can answer a question using MCP tool results with evidence links.

### M9 — Event-Driven Architecture

Acceptance criteria:

- [ ] Kafka in Compose; document pipeline async via transactional outbox.
- [ ] Idempotent consumer (`processed_event`), retry with backoff, DLQ, and a way to replay.
- [ ] Owner can explain: partitions, consumer groups, offsets, at-least-once, why outbox.

### M10 — Observability

- [ ] OTel traces across HTTP → DB → LLM → Kafka; metrics in Prometheus; Grafana dashboards
  (HTTP latency/errors, DB, Kafka lag, LLM latency & tokens, RAG latency).
- [ ] One request traceable end to end.

### M11 — Testing hardening

- [ ] Coverage gaps closed for critical paths; Kafka integration tests; flaky tests eliminated; test pyramid documented.

### M12 — CI/CD

- [ ] PR pipeline: build → unit → integration → quality gate → security scan (deps + image) → Docker build.

### M13 — Containerization

- [ ] Multi-stage, small, non-root images for BE and FE; env-based config; production-like Compose.

### M14 — Kubernetes

- [ ] Helm chart: Deployment, Service, ConfigMap, Secret, Ingress, probes, resources, HPA.

### M15 — Production Hardening

- [ ] Rate limiting, caching where justified, AI cost controls, security review (OWASP + LLM top 10), DB query review, resilience (timeouts, retries, circuit breaker where justified).

### M16 — Final

- [ ] README, architecture diagram, API docs, ADRs, deployment & monitoring guides, demo script, lessons learned.

---

## Roadmap Adjustments (accepted 2026-10-15)

1. **Testing from M1, not only M11** (D-06). Every milestone's DoD already requires tests; M11 becomes "hardening" (gaps, Kafka tests, flakiness), not "start testing".
2. **Minimal CI in M1** (D-06): one GitHub Actions job `mvn verify` + `npm run build`. ~30 minutes of work, catches breakage from day one. Full pipeline remains M12.
3. **Basic logging/health from M1**: correlation id + actuator. Full observability remains M10.
