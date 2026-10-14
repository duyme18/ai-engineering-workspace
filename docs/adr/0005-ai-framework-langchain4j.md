# ADR-0005: LangChain4j as AI integration framework

- Status: Accepted (2026-10-15)
- Date: 2026-10-14
- Milestone: M0 (implemented in M5)

## Context

We need chat models via OpenAI-compatible APIs, embeddings, document splitting, retrieval,
tool calling, streaming, and an MCP client — from Java.

## Decision

Use **LangChain4j**. Wrap it behind a small number of our own services in the `ai` / `knowledge`
modules so business code does not depend on LangChain4j types directly.
Access models through the **OpenAI-compatible** API so the provider (hosted, or local such as Ollama)
is a configuration choice.

## Alternatives considered

- **Spring AI** — first-party Spring integration that tracks Spring Boot releases closely; a strong alternative. If LangChain4j's Spring Boot 4 support is not ready at M5, Spring AI is the fallback.
- **Raw HTTP to provider APIs** — maximum control and wire-level learning, but we would rebuild tool-calling loops, stream parsing and splitters.

## Consequences

- \+ Rich Java-native building blocks for RAG, tools, agents, MCP.
- − Library evolves fast; APIs change between versions → pin versions, isolate usage behind our services.
- − Risk R1 (Spring Boot 4 compatibility) must be verified at the start of M5.
