# API Design — AI Engineering Workspace

> Status: **Baseline v1 (M0)** — endpoint inventory and conventions. Detailed request/response schemas
> are produced as OpenAPI (springdoc) per milestone.
> Last updated: 2026-10-14

---

## 1. Conventions

| Topic | Convention |
| --- | --- |
| Style | REST over JSON, resource-oriented; state transitions with business rules get **command endpoints** (`PATCH /tasks/{id}/status`, `POST /projects/{id}/archive`) |
| Base path | `/api/v1` (version in path — simple, visible, cache-friendly) |
| JSON | `camelCase` fields; `null` fields omitted; ids are UUID strings |
| Time | ISO-8601 UTC (`2026-10-14T08:30:00Z`); dates as `YYYY-MM-DD` |
| Auth (M2+) | `Authorization: Bearer <access_token>` (Keycloak JWT). All `/api/**` require auth, except `/api/v1/system/info` |
| Errors | RFC 9457 Problem Details — see §2 |
| Pagination | `?page=0&size=20&sort=createdAt,desc` (max `size` = 100) — see §3 |
| Filtering | Plain query params (`?status=BLOCKED&priority=HIGH&assigneeId=…&q=redis`) |
| Concurrency | Update requests include `version`; mismatch → `409 Conflict` |
| Create | `201 Created` + `Location` header + body |
| Async accept | `202 Accepted` when processing continues after the response (documents) |
| Delete | `204 No Content` |
| Streaming | SSE (`text/event-stream`) for assistant responses |
| Docs | OpenAPI 3 at `/v3/api-docs`, Swagger UI in non-prod profiles |

### Authorization model (applies to every project-scoped endpoint)

| Action | VIEWER | MEMBER | OWNER | ADMIN (global) |
| --- | --- | --- | --- | --- |
| Read project, tasks, notes, documents; ask/chat | ✅ | ✅ | ✅ | ✅ |
| Create/edit tasks, notes; upload documents | ❌ | ✅ | ✅ | ✅ |
| Edit/archive project; manage members | ❌ | ❌ | ✅ | ✅ |

Not a member → **404** (not 403) for project-scoped resources, so project existence is not leaked.

---

## 2. Error Format (RFC 9457)

```json
{
  "type": "https://aiworkspace.dev/problems/validation-error",
  "title": "Validation failed",
  "status": 400,
  "detail": "Request has 2 invalid fields",
  "instance": "/api/v1/projects/7f.../tasks",
  "code": "VALIDATION_ERROR",
  "traceId": "4bf92f3577b34da6a3ce929d0e0e4736",
  "errors": [
    { "field": "title", "message": "must not be blank" },
    { "field": "blockedReason", "message": "required when status is BLOCKED" }
  ]
}
```

| HTTP | `code` examples |
| --- | --- |
| 400 | `VALIDATION_ERROR`, `MALFORMED_REQUEST` |
| 401 | `UNAUTHENTICATED` |
| 403 | `FORBIDDEN` (member but insufficient role) |
| 404 | `NOT_FOUND` |
| 409 | `CONFLICT_VERSION`, `DUPLICATE_DOCUMENT`, `PROJECT_KEY_TAKEN` |
| 413 / 415 | `FILE_TOO_LARGE`, `UNSUPPORTED_MEDIA_TYPE` |
| 422 | `INVALID_STATE_TRANSITION`, `LAST_OWNER` |
| 429 | `RATE_LIMITED` (M15) |
| 502 / 504 | `LLM_PROVIDER_ERROR`, `LLM_TIMEOUT` |
| 500 | `INTERNAL_ERROR` (never exposes stack traces) |

---

## 3. Pagination Response

```json
{
  "content": [ { "...": "..." } ],
  "page": { "number": 0, "size": 20, "totalElements": 57, "totalPages": 3 }
}
```

Sortable fields are **allow-listed per endpoint** (unknown sort field → 400), never passed raw to the DB.

---

## 4. Endpoint Inventory

### M1 — Foundation

| Method | Path | Purpose |
| --- | --- | --- |
| GET | `/actuator/health` | Liveness/readiness incl. DB check |
| GET | `/api/v1/system/info` | App name, version, build time — used to prove FE → BE → DB wiring |

### M2 — Identity

| Method | Path | Purpose |
| --- | --- | --- |
| GET | `/api/v1/me` | Current user profile + global roles (triggers JIT provisioning) |

### M3 — Projects

| Method | Path | Purpose | Min role |
| --- | --- | --- | --- |
| GET | `/api/v1/projects?status=&q=&page=&size=&sort=` | My projects (ADMIN: all) | — |
| POST | `/api/v1/projects` | Create; caller becomes OWNER | USER |
| GET | `/api/v1/projects/{projectId}` | Details | VIEWER |
| PUT | `/api/v1/projects/{projectId}` | Update name/description (`version`) | OWNER |
| POST | `/api/v1/projects/{projectId}/archive` | Archive | OWNER |
| POST | `/api/v1/projects/{projectId}/unarchive` | Restore | OWNER |
| GET | `/api/v1/projects/{projectId}/members` | List members | VIEWER |
| POST | `/api/v1/projects/{projectId}/members` | Add `{userId, role}` | OWNER |
| PUT | `/api/v1/projects/{projectId}/members/{userId}` | Change role | OWNER |
| DELETE | `/api/v1/projects/{projectId}/members/{userId}` | Remove (not last OWNER) | OWNER |
| GET | `/api/v1/users?q=` | Search users to add as members | USER |

### M3 — Tasks

Collection endpoints are nested under the project; item endpoints are flat (`/tasks/{id}`) and the
backend resolves the project from the task for authorization.

| Method | Path | Purpose | Min role |
| --- | --- | --- | --- |
| GET | `/api/v1/projects/{projectId}/tasks?status=&priority=&assigneeId=&q=&page=&size=&sort=` | List/filter | VIEWER |
| POST | `/api/v1/projects/{projectId}/tasks` | Create | MEMBER |
| GET | `/api/v1/tasks/{taskId}` | Details | VIEWER |
| PUT | `/api/v1/tasks/{taskId}` | Update editable fields (`version`) | MEMBER |
| PATCH | `/api/v1/tasks/{taskId}/status` | `{status, blockedReason?, version}` — validates transition | MEMBER |
| DELETE | `/api/v1/tasks/{taskId}` | Delete | MEMBER |
| GET | `/api/v1/me/tasks?status=` | Tasks assigned to me across projects | — |

### M3 — Notes

| Method | Path | Purpose | Min role |
| --- | --- | --- | --- |
| GET | `/api/v1/projects/{projectId}/notes?q=&page=&size=&sort=` | List | VIEWER |
| POST | `/api/v1/projects/{projectId}/notes` | Create | MEMBER |
| GET | `/api/v1/notes/{noteId}` | Read | VIEWER |
| PUT | `/api/v1/notes/{noteId}` | Update (`version`) | MEMBER |
| DELETE | `/api/v1/notes/{noteId}` | Delete | MEMBER |

### M4 — Documents

| Method | Path | Purpose | Min role |
| --- | --- | --- | --- |
| POST | `/api/v1/projects/{projectId}/documents` | Multipart upload → `202` with status | MEMBER |
| GET | `/api/v1/projects/{projectId}/documents?status=&q=&page=&size=&sort=` | List | VIEWER |
| GET | `/api/v1/documents/{documentId}` | Metadata + status | VIEWER |
| GET | `/api/v1/documents/{documentId}/file` | Download original | VIEWER |
| POST | `/api/v1/documents/{documentId}/reprocess` | Retry a `FAILED` document | MEMBER |
| DELETE | `/api/v1/documents/{documentId}` | Delete file + derived data | MEMBER |

### M5 — Knowledge (RAG)

| Method | Path | Purpose | Min role |
| --- | --- | --- | --- |
| GET | `/api/v1/projects/{projectId}/knowledge/search?q=&topK=` | Retrieval only (chunks + scores) — debugging & evaluation aid | VIEWER |
| POST | `/api/v1/projects/{projectId}/knowledge/ask` | `{question}` → `{answer, sources[]}` | VIEWER |

`sources[]` item: `{documentId, filename, chunkId, page?, section?, snippet, score}`.

### M6 — Assistant

| Method | Path | Purpose |
| --- | --- | --- |
| POST | `/api/v1/projects/{projectId}/conversations` | Create conversation |
| GET | `/api/v1/projects/{projectId}/conversations?page=&size=` | My conversations in project |
| GET | `/api/v1/conversations/{conversationId}/messages?page=&size=` | History |
| POST | `/api/v1/conversations/{conversationId}/messages` | Send message; response is an **SSE stream** |
| DELETE | `/api/v1/conversations/{conversationId}` | Delete (owner of conversation only) |

SSE event types (draft):

```
event: message.start   data: {"messageId": "..."}
event: token           data: {"text": "..."}
event: tool.call       data: {"toolExecutionId": "...", "tool": "getBlockedTasks", "status": "EXECUTED"}
event: tool.proposal   data: {"toolExecutionId": "...", "tool": "createTask", "arguments": {...}}   (M7)
event: citation        data: {"documentId": "...", "snippet": "..."}
event: message.end     data: {"inputTokens": 812, "outputTokens": 214}
event: error           data: {"code": "LLM_TIMEOUT", "detail": "..."}
```

### M7 — Tool executions

| Method | Path | Purpose |
| --- | --- | --- |
| POST | `/api/v1/tool-executions/{id}/confirm` | Confirm a proposed write action (optionally with edited arguments) → executes |
| POST | `/api/v1/tool-executions/{id}/reject` | Reject |
| GET | `/api/v1/projects/{projectId}/tool-executions?page=&size=` | Audit log (OWNER) |

### M8 — Integrations

| Method | Path | Purpose |
| --- | --- | --- |
| GET | `/api/v1/integrations` | Configured MCP servers + connection status + available tools (ADMIN) |
