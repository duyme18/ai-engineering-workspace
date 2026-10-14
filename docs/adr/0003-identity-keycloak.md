# ADR-0003: Keycloak for identity; project-level authorization in the application

- Status: Accepted (2026-10-15)
- Date: 2026-10-14
- Milestone: M0 (implemented in M2)

## Context

The app needs SSO-style login, standard tokens, and two kinds of permissions:
global (is this user an admin?) and **per project** (is this user OWNER/MEMBER/VIEWER of project X?).
The owner wants to learn OAuth2/OIDC as used in production.

## Decision

- **Keycloak** is the identity provider (OIDC). The app stores no passwords.
- Backend = **OAuth2 Resource Server** validating JWTs (signature via JWKS, issuer, audience, expiry).
- **Global roles** (`ADMIN`, `USER`) live in Keycloak as realm roles and arrive in the token.
- **Project roles** live in the application DB (`project_member`) and are checked by the `project` module.
- A local `app_user` row is created on first login (JIT provisioning), keyed by the token `sub`.
- The frontend integration pattern (server-side session/BFF vs browser PKCE) is decided in a separate ADR in M2.
  Fixed constraint: tokens are never stored in `localStorage`.

## Alternatives considered

- **Project roles in Keycloak** (groups/roles per project) — token bloat, admin overhead on every membership change, IdP coupled to business data. Rejected.
- **Spring Authorization Server** — we would operate our own IdP code; less realistic than integrating a standard IdP.
- **Auth0 / Cognito** — managed, but external dependency and cost; less to learn about running an IdP.
- **Session + local accounts** — doesn't teach OAuth2/OIDC.

## Consequences

- \+ Industry-standard flow; clear separation: *who are you* (Keycloak) vs *what can you do in this project* (app).
- \+ AI tools reuse the exact same authorization checks (PRD FR-52).
- − Keycloak adds a container and realm configuration that must be reproducible (realm export in git).
- − User data is duplicated (`app_user`) and must be refreshed on login.
