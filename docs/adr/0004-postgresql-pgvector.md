# ADR-0004: PostgreSQL + pgvector as the single datastore

- Status: Accepted (2026-10-15)
- Date: 2026-10-14
- Milestone: M0 (pgvector from M5)

## Context

We need relational data (projects, tasks), semi-structured data (metadata, tool arguments),
and vector similarity search for RAG, at ≤ ~2M vectors.

## Decision

Use **PostgreSQL** for everything; enable the **pgvector** extension in M5.
Use `jsonb` for flexible metadata. Chunks and embeddings live in `document_chunk`,
filtered by `project_id` in the same SQL query as the similarity search.

## Alternatives considered

- **Dedicated vector DB (Qdrant, Weaviate, Milvus)** — better at very large scale and advanced filtering, but a second datastore to operate and keep consistent with relational data.
- **Elasticsearch/OpenSearch** — hybrid search is attractive; heavy to run; revisit only if keyword + vector hybrid search becomes a requirement.

## Consequences

- \+ One datastore to run, back up and observe; transactional consistency between documents and chunks.
- \+ Authorization filter (`project_id`) and similarity search in one query.
- − Embedding dimension is fixed per column; changing the model requires migration + re-embedding.
- − At much larger scale, HNSW index build time and memory need tuning — revisit in M15.
- Local/CI images must include pgvector (e.g. the `pgvector/pgvector` image) by M5 — possibly from M1 to avoid an image switch (decide at M1).
