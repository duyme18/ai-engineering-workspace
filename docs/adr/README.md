# Architecture Decision Records

Format: lightweight [MADR](https://adr.github.io/madr/)-style. One decision per file, numbered, never deleted —
a reversed decision gets a new ADR that **supersedes** the old one.

Status values: `Proposed` → `Accepted` | `Rejected` → (later) `Superseded by ADR-XXXX`.

| ADR | Title | Status |
| --- | --- | --- |
| [0001](0001-modular-monolith.md) | Modular monolith, single Maven module, package-per-module | Accepted |
| [0002](0002-monorepo.md) | Monorepo for backend, frontend, infrastructure, docs | Accepted |
| [0003](0003-identity-keycloak.md) | Keycloak for identity; project-level authorization in the application | Accepted |
| [0004](0004-postgresql-pgvector.md) | PostgreSQL + pgvector as the single datastore | Accepted |
| [0005](0005-ai-framework-langchain4j.md) | LangChain4j as AI integration framework | Accepted |
| [0006](0006-api-style-rest-sse.md) | REST + RFC 9457 Problem Details + SSE streaming | Accepted |
| [0007](0007-flyway-migrations.md) | Flyway for database migrations | Accepted |

## Template

```markdown
# ADR-XXXX: <title>

- Status: Proposed
- Date: YYYY-MM-DD
- Milestone: Mx

## Context

What problem / force requires a decision?

## Decision

What we will do.

## Alternatives considered

Option — why not.

## Consequences

Positive, negative, and what we must now take care of.
```
