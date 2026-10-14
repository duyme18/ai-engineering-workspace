# ADR-0002: Monorepo for backend, frontend, infrastructure, docs

- Status: Accepted (2026-10-15)
- Date: 2026-10-14
- Milestone: M0

## Context

Backend, frontend, infrastructure and documentation evolve together milestone by milestone.
API changes usually touch both backend and frontend.

## Decision

One Git repository with top-level `backend/`, `frontend/`, `infrastructure/`, `docs/`.
CI jobs are path-filtered per folder (M12).

## Alternatives considered

- **Polyrepo** — independent versioning, but cross-cutting changes need coordinated PRs; overhead for one developer.

## Consequences

- \+ Atomic changes across BE/FE/docs; one place for project state (useful for AI-assisted development).
- − CI must avoid rebuilding everything on every change (path filters).
