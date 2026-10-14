# Learning Log — AI Engineering Workspace

> Concepts learned, questions, and gotchas — newest first.
> Per technology, separate: **use** (to build it), **debug** (to fix it), **design** (to choose it), **skip for now**.

---

## Entry template

```markdown
## YYYY-MM-DD — Mx — <topic>

**Context:** what I was doing
**Learned:** key points in my own words
**Gotcha:** what surprised me / what broke
**Open questions:** still unclear
**Can I explain it?** yes / partially / no
```

---

## 2026-10-14 — M0 — Architecture foundations

**Context:** Defining product scope and architecture before writing code.

### Concepts to understand before closing M0

| Concept | Why it matters here | Depth needed now |
| --- | --- | --- |
| Modular monolith vs microservices | Core architectural style (ADR-0001) | Design |
| Module boundary & dependency rules | Prevents a big ball of mud; prepares for later extraction | Design |
| ADR (Architecture Decision Record) | How decisions are recorded and reversed | Use |
| Authentication vs authorization; identity in IdP vs permissions in app | ADR-0003; project roles in app DB | Design |
| RAG authorization filtering (`project_id`) is a security control | Prevents cross-project data leakage | Design |
| RFC 9457 Problem Details | Uniform error contract FE ↔ BE | Use |
| Optimistic vs pessimistic locking | `version` column (lost updates) vs row lock (task counter) | Use + debug |
| Transactional outbox (M9 preview) | Why we won't publish to Kafka directly from a transaction | Skip for now — know the name |
| Human-in-the-loop for agent write actions | AI must not exceed user permissions or act without consent | Design |

### M0 learning check — round 1 (2026-10-15)

| # | Question | Result | Gap / key takeaway |
| --- | --- | --- | --- |
| 1 | Why modular monolith first; when to extract a service? | Partial | Machine resources are a practical reason, not the architectural one. Real reasons: one developer, unproven boundaries, distributed costs (network failures, no cross-service ACID, observability, deployments). Extract only for independent scaling, failure isolation, release cadence/team ownership. First realistic candidate here: the document-processing worker (M9). |
| 2 | Why project roles in app DB, not Keycloak? | Partial | Membership is **business data** that changes often. In Keycloak it would bloat tokens (N projects × role), and role changes would stay stale until token refresh. Keycloak = who you are + global roles; app = what you can do in project X. |
| 3 | How does `task` check membership; why not query `project_member`? | Not yet | Call `project`'s public API (`ProjectAccess.require…`). Direct table access duplicates the authorization rules (ADMIN, archived, 404-vs-403) in every module and couples them to another module's schema. |
| 4 | What if the RAG query forgets `project_id`? | Pass (refined) | Top-K similarity runs over **all** projects → chunks from projects the user can't access reach the LLM → silent data leak (Broken Access Control). Filter inside the query, not after top-K. |
| 5 | Concurrent `PAY-12` creation; row lock vs optimistic? | Partial | Both read `next=12` → duplicate number / unique-constraint failure. `FOR UPDATE` makes the 2nd transaction wait, then read 13. Optimistic would fail + need retry on a hot row where conflict is *expected*; optimistic fits rare conflicts that the user should resolve (409 on edit). Alternative: atomic `UPDATE … SET n = n + 1 RETURNING n`. |

**Open questions:** Ways to run microservices on limited local hardware (answered in chat: run one service + stubs, compose profiles, kind/k3d, remote dev cluster).

### Round 2 (2026-10-15)

- **Q2' (revocation timing):** Pass. Key distinction: a still-valid token means **authenticated**, not **authorized**. With roles in the app DB, X gets 404 at 10:02; with roles in the token, X can read until 10:05.
- **Q3' (where does a new rule go?):** Question was unclear and has been re-explained with a concrete example. With `ProjectAccess`, an "archived project is read-only" rule is changed in 1 place. With per-module queries it has to be changed in every write use case (task, note, document, AI tool…), and any place that gets missed is a security hole.

**Q3 restated by owner:** "Authorization logic lives in one place; change it once and every feature using it follows." → Pass. (Also: the owning module can change its schema freely.)

**Can I explain it?** Yes — M0 learning check passed (2026-10-15).
