# ADR-0006: REST + RFC 9457 Problem Details + SSE streaming

- Status: Accepted (2026-10-15)
- Date: 2026-10-14
- Milestone: M0

## Context

One first-party web client, CRUD-heavy resources, and token streaming for AI responses.

## Decision

- REST/JSON under `/api/v1`; command endpoints for state transitions that carry business rules.
- Errors as **RFC 9457 Problem Details** with a stable `code` and `traceId` (Spring's built-in `ProblemDetail` support).
- AI responses streamed via **Server-Sent Events**; the frontend consumes them with `fetch` + `ReadableStream`
  (the request is a POST with a body, so `EventSource` cannot be used).
- Conventions in `api-design.md`.

## Alternatives considered

- **GraphQL** — flexible querying, but extra schema/tooling complexity without a multi-client need.
- **WebSocket** for streaming — bidirectional, but we only need server → client; SSE is plain HTTP and passes proxies/load balancers more easily.
- **Custom error JSON** — no reason to invent a format when a standard exists.

## Consequences

- \+ Simple, standard, well supported by Spring and browsers.
- − Long-lived SSE connections need proxy/ingress timeouts configured (M13–M14).
- − Versioning in the path means duplicating routes for `/api/v2` if a breaking change happens (acceptable, unlikely).
