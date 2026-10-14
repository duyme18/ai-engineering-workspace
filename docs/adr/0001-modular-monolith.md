# ADR-0001: Modular monolith, single Maven module, package-per-module

- Status: Accepted (2026-10-15)
- Date: 2026-10-14
- Milestone: M0

## Context

The product has clear business areas (identity, project, task, note, knowledge, ai) but one
developer, one deployment target and modest scale (see PRD §2). The learning goal includes
designing module boundaries. Microservices would add network failure modes, distributed data,
and deployment complexity with no current benefit.

## Decision

- One Spring Boot application, organized as **modules = top-level packages** under the base package.
- Each module exposes a public API (classes in its root package); `web` and `internal` sub-packages are private by convention.
- Dependency rules from `architecture.md` §3.2 (acyclic; no cross-module repository/entity access; references by ID).
- Rules are **verified by an automated test** (Spring Modulith `ApplicationModules.verify()`, test scope only — no Modulith runtime features; D-05).
- **Single Maven module** (not one Maven module per business module).

## Alternatives considered

- **Microservices** — premature; costs exceed benefits at this scale.
- **Layered monolith (controller/service/repository packages at top level)** — no business boundaries; tends to become a big ball of mud.
- **Multi-module Maven (one artifact per module)** — compile-time enforcement, but much more build ceremony for a single developer. Can be adopted later if package-level checks prove insufficient.
- **Hexagonal / clean architecture with ports everywhere** — many interfaces with one implementation; over-engineering for a CRUD-heavy domain.

## Consequences

- \+ Simple build, deploy, debug; local transactions across modules when truly needed.
- \+ Boundaries are explicit and testable; extracting a service later is feasible (e.g. a document worker in M9).
- − Boundaries are enforced by tests, not the compiler — the boundary test must stay green and run in CI.
- − Shared database: modules must respect table ownership (`database.md` §2).
