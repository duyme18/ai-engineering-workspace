# Development Guide — AI Engineering Workspace

> Status: **Baseline v1 (M0)** — setup commands are added in M1.
> Last updated: 2026-10-14

---

## 1. Repository Structure (monorepo — [ADR-0002](adr/0002-monorepo.md))

```text
ai-engineering-workspace/
├── backend/                          # Spring Boot 4, Java 25, Maven (single Maven module)
│   ├── pom.xml
│   └── src/
│       ├── main/java/dev/duyhd/aiworkspace/
│       │   ├── Application.java
│       │   ├── shared/               # error model, pagination, config (no business logic)
│       │   ├── identity/             # M2
│       │   ├── project/              # M3
│       │   ├── task/                 # M3
│       │   ├── note/                 # M3
│       │   ├── knowledge/            # M4–M5
│       │   └── ai/                   # M5–M8
│       ├── main/resources/
│       │   ├── application.yml
│       │   └── db/migration/         # Flyway: V1__init.sql, V2__..., never edited once applied
│       └── test/java/dev/duyhd/aiworkspace/         # mirrors main; + ModularityTest
│
├── frontend/                         # Next.js (App Router), TypeScript, Tailwind, shadcn/ui
│   ├── package.json
│   └── src/
│       ├── app/                      # routes (layouts, pages)
│       ├── components/ui/            # shadcn/ui generated components
│       ├── features/                 # projects/, tasks/, notes/, documents/, assistant/
│       └── lib/                      # api client, auth helpers, utils
│
├── infrastructure/
│   ├── docker/
│   │   ├── compose.yaml              # local dev: postgres (+ keycloak M2, kafka M9, ...)
│   │   └── keycloak/realm-export.json   # M2: reproducible realm, clients, roles, test users
│   └── k8s/ → helm/                  # M14
│
├── docs/
│   ├── ROADMAP.md                    # milestones + acceptance criteria
│   ├── PROJECT-STATE.md              # ← single source of truth for "where are we"
│   ├── LEARNING-LOG.md               # concepts learned, questions, gotchas
│   ├── product/PRD.md
│   ├── architecture.md
│   ├── database.md
│   ├── api-design.md
│   ├── development-guide.md
│   └── adr/                          # architecture decision records
│
├── .github/workflows/                # CI (minimal from M1 (D-06); full in M12)
├── .editorconfig
├── .gitignore
└── README.md
```

Each module grows **only when its milestone starts**. No empty placeholder packages for future modules.

---

## 2. Conventions

### Backend

- Package per module; inside: root = public API, `web` = controllers/DTOs, `internal` = entities/repositories.
- Controllers are thin: validate → call service → map response. No repository calls in controllers.
- `@Transactional` only on service methods. No HTTP/LLM/storage calls inside a DB transaction.
- DTOs are Java `record`s. Entities never leave their module and never reach JSON.
- Use constructor injection only.
- New dependency → must answer "what problem does it solve?" in the PR/commit message.

### Frontend

- Server state via TanStack Query; no ad-hoc `useEffect` fetching.
- Forms: React Hook Form + Zod (decided in M3 if needed).
- All API calls go through `src/lib/api`.
- Tokens are never stored in `localStorage`.

### Git

- `main` is always green. Work on short-lived branches: `m1/backend-skeleton`, `m3/task-filtering`.
- Conventional commits: `feat(task): add status transition endpoint`, `docs(adr): add ADR-0008`.
- One milestone task ≈ one PR (small, reviewable).

### Documentation update rule

| When | Update |
| --- | --- |
| Task finished | `PROJECT-STATE.md` (current task, done list) |
| New concept learned / gotcha | `LEARNING-LOG.md` |
| Significant decision (tech, boundary, data model) | New ADR in `docs/adr/` |
| DB change | `database.md` + Flyway migration |
| API change | `api-design.md` (inventory) + OpenAPI from code |
| Milestone gate passed | `ROADMAP.md` status + `PROJECT-STATE.md` |

---

## 3. Local Development

*To be written in M1* — prerequisites (JDK 25, Node LTS, Docker), `docker compose up`,
running backend and frontend, environment variables, ports.
