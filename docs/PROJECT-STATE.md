# Project State — AI Engineering Workspace

> **Read this first in every new conversation.** Together with `ROADMAP.md` and `LEARNING-LOG.md`
> it is enough to resume the project.
> Last updated: 2026-10-15

---

## Snapshot

```text
Current Milestone: M1 — Project Foundation
Status:            NOT STARTED (M0 gate PASS 2026-10-15)
Current Task:      Commit docs/, then M1 kickoff: technology explanation + implementation plan (no code yet)
Completed:         M0 — Product & Architecture
Blocked:           —
Next:              M1 plan → owner approval → implementation in small steps
```

## Repository State (evidence)

| Area | State |
| --- | --- |
| `backend/` | Empty folder, no code |
| `frontend/` | Empty folder, no code |
| `infrastructure/docker`, `infrastructure/k8s` | Empty folders |
| `docs/` | M0 baseline v1: PRD, architecture, database, API design, development guide, ROADMAP, ADR 0001–0007 |
| Git | 1 commit (`README.md`); `docs/` not yet committed |
| CI | None (minimal CI planned for M1) |
| Tests | None |

## Completed Milestones

| Milestone | Gate | Date |
| --- | --- | --- |
| M0 — Product & Architecture | PASS | 2026-10-15 |

## M0 Deliverables

| Deliverable | File | Status |
| --- | --- | --- |
| Product vision, users, use cases, MVP, out of scope, flows, NFR | [product/PRD.md](product/PRD.md) | Baseline v1 — owner reviewed |
| High-level architecture, modules, flows, risks | [architecture.md](architecture.md) | Baseline v1 — owner reviewed |
| Technology stack | [ROADMAP.md § Technology Selection](ROADMAP.md) | Baseline v1 |
| Repository structure & conventions | [development-guide.md](development-guide.md) | Baseline v1 |
| Initial database design | [database.md](database.md) | Baseline v1 |
| Initial API design | [api-design.md](api-design.md) | Baseline v1 |
| Milestones + acceptance criteria + M0 DoD | [ROADMAP.md](ROADMAP.md) | Baseline v1 |
| ADRs | [adr/](adr/README.md) | 7 × Accepted |

## Decision Log

| ID | Decision | Outcome | Date |
| --- | --- | --- | --- |
| D-01 | Language of docs | **English** (portfolio-ready); conversation in Vietnamese | 2026-10-15 |
| D-02 | Java base package / Maven groupId | **`dev.duyhd.aiworkspace`** (groupId `dev.duyhd`, artifactId `ai-workspace-backend`) — changeable until M1 code exists | 2026-10-15 |
| D-03 | Merge `auth` + `user` | **Yes** → single `identity` module | 2026-10-15 |
| D-04 | Knowledge scope | **Per project**; personal/global knowledge out of scope | 2026-10-15 |
| D-05 | Module boundary enforcement | **Spring Modulith verification test only** (test scope, no runtime features) | 2026-10-15 |
| D-06 | Tests + minimal CI from M1 | **Yes** — M11 becomes testing *hardening* | 2026-10-15 |
| D-07 | AI write actions require user confirmation | **Yes** (PRD FR-51) | 2026-10-15 |
| D-08 | Human-readable task numbers (`PAY-12`) | **Yes** — per-project counter under row lock | 2026-10-15 |
| D-09 | Maven structure | **Single Maven module** + boundary test | 2026-10-15 |

### Deferred decisions (decided at the milestone, not now)

| Decision | When |
| --- | --- |
| Exact versions (Spring Boot 4.x, Next.js, PostgreSQL major, Node LTS) | M1 |
| Postgres image with pgvector from day one? | M1 |
| Frontend auth pattern (server-side session/BFF vs browser PKCE) | M2 (ADR) |
| File storage: local FS vs MinIO/S3 | M4 (ADR) |
| LLM + embedding provider/model, vector dimension, chunking params | M5 |
| LangChain4j vs Spring AI final check (Boot 4 compatibility) | M5 start |
| Conversation memory strategy | M6 |

## Last Milestone Gate — M0 (2026-10-15)

```text
Functional:    PASS     (all M0 deliverables exist)
Architecture:  PASS     (ADR 0001–0007 accepted, no open structural decisions)
Code Quality:  N/A      (no code in M0)
Security:      PASS     (authz model, RAG project filter, AI least-privilege, human-in-the-loop defined)
Performance:   PASS     (scale assumptions + initial targets defined)
Testing:       PASS     (test strategy per milestone defined; tests from M1)
Documentation: PASS     (PRD + architecture reviewed by owner)
Learning:      PASS     (learning check passed after 2 rounds; see LEARNING-LOG)
Overall:       PASS
```

## Known Issues / Risks

See [architecture.md §7](architecture.md). Top items: R1 LangChain4j × Spring Boot 4 compatibility,
R3 prompt injection, R6 local storage vs Kubernetes.

## Change Log

| Date | Change |
| --- | --- |
| 2026-10-15 | Owner reviewed PRD + architecture; learning check passed. **M0 gate PASS.** Current milestone → M1. |
| 2026-10-15 | D-01…D-09 accepted (all as recommended; base package `dev.duyhd.aiworkspace`). ADR 0001–0007 Accepted. Roadmap adjustment (tests + minimal CI from M1) accepted. M0 gate run: NOT READY (owner review + learning check). |
| 2026-10-14 | M0 started. Drafted PRD, architecture, database, API, dev guide, roadmap, ADR 0001–0007. |
