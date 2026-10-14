# Product Requirements — AI Engineering Workspace

> Status: **Baseline v1 (M0)** — decisions accepted, owner-reviewed 2026-10-15
> Last updated: 2026-10-14

---

## 1. Product Vision

**For** software engineers and small engineering teams
**who** have project knowledge scattered across documents, notes, tickets and infrastructure tools,
**AI Engineering Workspace is** a self-hostable workspace
**that** keeps project knowledge and work items in one place and provides an AI assistant that
answers questions **grounded in the team's own data, with citations**, and performs **scoped
actions through tools** — always within the permissions of the user who asked.

**Unlike** a generic chatbot, answers are traceable to sources, and the assistant can see and act
on live project state (tasks, notes, documents, and later GitHub/Jira/Kubernetes).

### What it is NOT

- Not an IDE or coding agent (not Cursor / Claude Code).
- Not a Jira replacement (task management is intentionally minimal).
- Not an observability platform (it *reads* monitoring data via tools later; it does not store it).

### Dual purpose (explicit)

This is a learning + portfolio project. Every feature must be **demoable** and **explainable**.
Scope decisions favor depth of engineering (security, data design, AI correctness) over breadth
of features.

---

## 2. Target Users

| Persona | Description | Main needs |
| --- | --- | --- |
| **P1 — Engineer** (primary) | Works on 1–3 projects. Reads specs, writes notes, picks up tasks. | Find answers in docs fast, know what's blocked, capture notes. |
| **P2 — Tech Lead / Project Owner** | Owns a project, manages members and priorities. | Project status at a glance, ask "what's blocked and why", delegate. |
| **P3 — Platform Admin** | Operates the deployment. | Manage users/roles (in Keycloak), monitor health and AI cost. |

### Scale assumptions (drive design, re-check at M15)

| Dimension | Assumption |
| --- | --- |
| Users | ≤ 100 per deployment |
| Projects | ≤ 50 |
| Documents | ≤ 10,000 (≤ 20 MB each) |
| Chunks / vectors | ≤ ~2 million |
| Tenancy | Single tenant (one organization per deployment) |

Consequences: one PostgreSQL instance is enough; pgvector is enough (no dedicated vector DB);
no sharding; no multi-tenancy layer.

---

## 3. Core Use Cases

| ID | Use case | Actor | Milestone | Priority |
| --- | --- | --- | --- | --- |
| UC-01 | Sign in / sign out via SSO | All | M2 | MVP |
| UC-02 | Create / edit / archive a project; manage members & project roles | P2 | M3 | MVP |
| UC-03 | Manage tasks: create, assign, change status, mark BLOCKED with reason, filter/sort | P1, P2 | M3 | MVP |
| UC-04 | Write and edit Markdown notes in a project | P1, P2 | M3 | MVP |
| UC-05 | Upload documents (PDF, MD, DOCX, TXT) to a project; see processing status | P1, P2 | M4 | MVP |
| UC-06 | Ask a question about project knowledge; get an answer **with source citations** | P1, P2 | M5 | MVP |
| UC-07 | Chat with the assistant: persistent conversations, streaming responses | P1, P2 | M6 | MVP |
| UC-08 | Ask about work state ("which tasks are blocked?") — assistant reads data via tools | P1, P2 | M7 | MVP |
| UC-09 | Ask the assistant to perform a write action ("create task X, HIGH") — **user confirms** before execution | P1, P2 | M7 | MVP |
| UC-10 | Assistant uses external engineering context via MCP (GitHub, Jira, Kubernetes) | P1, P2 | M8 | Post-MVP |
| UC-11 | Incident investigation: correlate pods, logs, metrics, deploys, commits → root-cause hypothesis with evidence | P1 | M8+ | Stretch |
| UC-12 | Admin views system health, AI usage and token cost | P3 | M10 / M15 | Post-MVP |

---

## 4. MVP Scope

**MVP = M1 → M7.** At the end of M7 the product can be demoed end-to-end:
login → project → tasks/notes → upload docs → ask with citations → chat that uses tools.

M8–M16 are the **integration + production-engineering track** (MCP, Kafka, observability,
testing hardening, CI/CD, containers, Kubernetes, hardening, final docs).

### Functional requirements (MVP)

**Identity & access**
- FR-01 Users authenticate via Keycloak (OIDC). No local passwords in the app.
- FR-02 On first successful login the app provisions a local user record (JIT provisioning).
- FR-03 Global roles: `ADMIN`, `USER` (Keycloak realm roles).
- FR-04 Project roles: `OWNER`, `MEMBER`, `VIEWER` (stored in app DB, not Keycloak).
- FR-05 A user only sees projects they are a member of (ADMIN sees all).

**Workspace**
- FR-10 Project: name, unique key (e.g. `PAY`), description, status (`ACTIVE`, `ARCHIVED`).
- FR-11 Project members can be added/removed/changed by OWNER. A project always has ≥ 1 OWNER.
- FR-12 Task: title, description, status, priority, assignee, reporter, due date, blocked reason.
- FR-13 Task status: `TODO`, `IN_PROGRESS`, `BLOCKED`, `DONE`, `CANCELLED`. `BLOCKED` requires a reason.
- FR-14 Task list supports pagination, sorting, filtering (status, priority, assignee, text).
- FR-15 Note: title, Markdown content, author. Paginated list, edit, delete.
- FR-16 VIEWER is read-only; MEMBER can create/edit tasks & notes; OWNER can also manage project & members.
- FR-17 Concurrent edits are detected (optimistic locking) → 409 Conflict, not silent overwrite.

**Knowledge base**
- FR-20 Upload PDF, Markdown, DOCX, TXT up to 20 MB into a project.
- FR-21 Document status lifecycle visible in UI: `UPLOADED → PROCESSING → READY` or `FAILED` (with reason).
- FR-22 Failed documents can be reprocessed. Documents can be deleted (including derived data).
- FR-23 Duplicate upload (same content in same project) is detected.

**RAG**
- FR-30 Ask a question scoped to a project; retrieval uses **only** that project's documents.
- FR-31 Answer includes sources: document, location (page/section), snippet.
- FR-32 If the knowledge base has no relevant content, the assistant says so instead of inventing.

**AI assistant**
- FR-40 Conversations are persisted, per user, scoped to a project.
- FR-41 Responses are streamed (SSE).
- FR-42 Token usage and latency are recorded per assistant message.

**Tools / agent**
- FR-50 Read tools: get project, list tasks, list blocked tasks, search documents, list notes.
- FR-51 Write tools (e.g. create task) are **proposed** by the model and executed only after explicit user confirmation.
- FR-52 Tools execute **as the requesting user** — same authorization as the REST API. The AI never has more access than the user.
- FR-53 Every tool execution is audited (who, what, arguments, result, time).

---

## 5. Out of Scope

| Out of scope | Reason |
| --- | --- |
| Multi-tenancy, organizations, billing | Single-tenant assumption; large complexity, low learning ROI here |
| Public self sign-up | Users are created by admin in Keycloak |
| Real-time collaborative editing (notes) | CRDT/OT is a separate domain |
| Sprints, boards, sub-tasks, task comments, task attachments | Not needed to demonstrate the AI value |
| Bidirectional sync with Jira/GitHub | MCP is read-mostly; sync is a product on its own |
| Code editing / committing to repositories | Not an IDE |
| OCR, images, audio, video, spreadsheets | Parsing scope limited to PDF/MD/DOCX/TXT |
| Fine-tuning / self-training models | Uses hosted or local models via OpenAI-compatible API |
| Fully autonomous write actions | Human-in-the-loop is a product principle (FR-51) |
| Email / push notifications | Not needed for demo |
| Mobile app, i18n | Desktop web only, English UI |
| Microservices, multi-region | Modular monolith (see ADR-0001) |

---

## 6. User Flows

### Flow A — First login (M2)

```
Open app → not authenticated → redirect to Keycloak login
→ login success → redirect back with auth code → session established
→ backend receives first request with JWT → provisions app_user (JIT)
→ Dashboard (empty state: "Create your first project")
```

### Flow B — Project & tasks (M3)

```
Dashboard → "New project" (name, key) → user becomes OWNER
→ Members: add user as MEMBER
→ Tasks: create "Investigate Redis timeout" (HIGH) → assign
→ change status to BLOCKED → system requires blocked reason
→ Task list: filter status=BLOCKED, sort by priority
```

### Flow C — Knowledge (M4 → M5)

```
Project → Documents → upload auth-design.pdf
→ list shows status PROCESSING → READY (or FAILED + reason + "Reprocess")
→ Ask: "How is authentication designed?"
→ answer + sources [auth-design.pdf p.4, api-spec.md §Security]
→ click source → view the cited snippet in context
```

### Flow D — Assistant reading via tools (M6 → M7)

```
Project → Assistant → new conversation
→ "Which tasks are blocked?" → model calls getBlockedTasks(projectId)
→ tool runs with the user's permissions → model summarises
→ streamed answer listing tasks + reasons, linked to task pages
```

### Flow E — Assistant write action with confirmation (M7)

```
"Create task 'Investigate Redis timeout', priority HIGH"
→ model proposes createTask(...) → UI shows confirmation card (fields editable)
→ user clicks Confirm → task created → audit record → assistant confirms with link
   (Reject → nothing created, assistant acknowledges)
```

### Flow F — Incident investigation (M8+, stretch)

```
"Why does payment-service return many HTTP 500s?"
→ tools: k8s pods/events → logs → metrics → recent deployments → GitHub commits/PRs → Jira issues
→ answer: timeline + hypothesis + evidence links + confidence; no write actions
```

---

## 7. Non-Functional Requirements

| Area | Requirement |
| --- | --- |
| Security | All APIs authenticated (from M2). Authorization enforced on the backend at project level. RAG retrieval filtered by project membership. Tools run with caller identity. Secrets only via environment/secret store, never committed. Uploaded files validated by type and size. Treat document content as untrusted input to the LLM (prompt-injection awareness). |
| Performance (initial targets, measured in M10) | CRUD p95 < 300 ms (local). RAG retrieval < 500 ms. Time-to-first-token < 3 s excluding provider variance. |
| Reliability | Document processing failures are visible and retryable. LLM/provider failures return a clear error, not a hung request (timeouts everywhere). |
| Observability | Correlation/trace id per request from M1; structured logs; metrics & tracing from M10. |
| Cost | Token usage recorded per message (M6); per-user limits/budgets (M15). |
| Maintainability | Module boundaries enforced; ADRs for significant decisions; docs kept current per milestone. |
| Portability | Runs locally with `docker compose up`; deployable to Kubernetes via Helm (M14). |
