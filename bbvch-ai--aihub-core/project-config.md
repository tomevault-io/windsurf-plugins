---
trigger: always_on
description: **Swiss AI Hub**: Open-source, self-hosted AI platform for enterprises. Organizations run it on their own infrastructure
---

# Swiss AI Hub Developer Guide for AI Agents

## Project Overview

**Swiss AI Hub**: Open-source, self-hosted AI platform for enterprises. Organizations run it on their own infrastructure
— no SaaS dependency, full data sovereignty. The platform handles authentication, multi-tenancy, cost control,
observability, LLM routing, vector storage, and document parsing as commodity infrastructure. Developers focus on
building agents, pipelines, and processes using the SDK; the platform provides the runtime.

**Platform vs SDK**: The platform is the runtime infrastructure (Docker, Traefik, LiteLLM, Milvus, Dagster, NATS,
PostgreSQL — handles SSO, routing, cost tracking, UI hosting). The SDK is where you build: agents, data pipelines, bot
integrations, and processes. SDK code inherits all platform capabilities — no custom REST APIs, WebSocket management,
database schemas, or auth logic needed.

**Swiss AI Agent Protocol**: Event-driven protocol over NATS. Strict Control Event (workflow) vs Display Event
(observability) separation. Hierarchical scoping (Thread → Display → Run). See
`docs/docs/2_platform/2_architecture/3_swiss_ai_agent_protocol/index.en.md`.

## Platform Architecture

The platform follows an event-driven microservice architecture where NATS serves as the central communication backbone.
All inter-component communication flows through the Swiss AI Agent Protocol, which distinguishes between Control Events
(workflow state transitions consumed by agents and orchestrators) and Display Events (observability data consumed by
frontends and tracing systems). This separation allows the chat UI to visualize agent reasoning in real-time without
interfering with workflow execution.

**LLM Gateway**: All model access is mediated by LiteLLM, which provides a unified OpenAI-compatible interface to cloud
providers (Swiss LLM Cloud), local models (vLLM for chat, embedding, and reranking on GPU deployments), and audio models
(Speaches for STT/TTS). Presidio intercepts requests for PII detection and anonymization before they reach external
providers. This gateway pattern decouples application code from specific providers — switching models requires only a
configuration change in LiteLLM, not code modifications.

**Agent Runtime**: Agents run as independent microservices that subscribe to NATS topics and publish events. The
dispatcher replays the event history from JetStream to determine which steps to execute, while durable state in Valkey
(RunContext, ThreadContext) holds arbitrary agent-set data that cannot be reconstructed from events. Agents access
organizational knowledge through Milvus vector search, where document embeddings are indexed for semantic retrieval.
This design allows agents to scale horizontally and be deployed or updated independently without affecting the rest of
the platform.

**Data Pipeline**: Dagster orchestrates the ingestion workflow: sources (SharePoint, OneDrive via Rclone) are monitored
for changes, documents are downloaded to SeaweedFS, parsed by MinerU (OCR + structural extraction), chunked
semantically, embedded via configured models, and stored in Milvus. The asset-based pipeline model provides lineage from
every vector embedding back to its source document.

**Process Orchestration**: The process engine coordinates multi-step workflows involving agents, humans, and external
systems (Power Automate, n8n, UiPath). Process state is persisted in FerretDB (MongoDB-compatible API over PostgreSQL).
When a step requires human judgment, a task appears in the Process UI; when it requires AI, the engine delegates to an
agent via NATS; when it requires an external action, it triggers a webhook.

**Storage Layer**: The platform uses purpose-specific storage: PostgreSQL for relational data (OpenWebUI, Langfuse,
Dagster, LiteLLM), FerretDB for document storage (conversations, app data, events), Milvus for vector embeddings,
SeaweedFS for S3-compatible file storage (uploads, artifacts, pipeline data), Valkey for ephemeral agent state, and
Neo4j for graph-based memory. etcd serves as the metadata backend for both Milvus and SeaweedFS.

**Observability**: OpenTelemetry collects distributed traces across all services, forwarded by the OTEL Collector.
Langfuse adds AI-specific observability on top — full prompt/response capture, per-trace cost tracking, RAG retrieval
tracing, and evaluation datasets. Both integrate via the same trace context, providing end-to-end visibility from user
request to LLM response.

**Network Isolation**: Docker Compose defines five network zones: `proxy` (external ingress via Traefik), `backend`
(application services), `data` (databases, caches, message broker), `storage` (SeaweedFS cluster), and `egress`
(outbound internet with inter-container communication disabled). Services are assigned only the networks they require.

## Dev Stack Services

The `infra/docker-compose.dev.yml` runs ~30 containers. Key services by role:

**User-Facing**: OpenWebUI (chat UI, :8080), Admin UI and Process UI (Nuxt, :3333, run locally outside Docker)

**API & Gateway**: FastAPI REST + WebSocket (:8000, run locally), LiteLLM universal LLM proxy (:4000), Traefik reverse
proxy (production only)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bbvch-ai/aihub-core](https://github.com/bbvch-ai/aihub-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-28 -->
