---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

OpenCrawling is a Java 25 / Spring Boot 4 enterprise data ingestion and vector search platform. It crawls document repositories, extracts text via Apache Tika, chunks it via Spring AI's `TokenTextSplitter`, generates embeddings (Ollama or OpenAI), and stores vectors in PostgreSQL + pgvector. An MCP server exposes the vector store to AI clients with server-side ACL enforcement.

## Build Commands

```bash
# Full build (all Maven modules)
mvn clean install

# Run the ingestion runtime (dev profile, connects to local Docker infra)
mvn spring-boot:run -pl oc-runtime -Dspring-boot.run.profiles=dev

# Run the embedding microservice (separate process)
mvn spring-boot:run -pl oc-embedding-service

# Run unit tests only
mvn test

# Run integration tests (requires Docker infrastructure running)
mvn verify -pl oc-runtime

# Run a single test class
mvn test -pl oc-runtime -Dtest=McpVectorServerTest

# Run a single integration test
mvn verify -pl oc-runtime -Dit.test=OpenCrawlingIT

# Admin UI
cd oc-admin-ui && npm install && npm run dev   # dev server at http://localhost:5173
cd oc-admin-ui && npm run build                # production build
cd oc-admin-ui && npm run lint                 # ESLint
```

**Preview features are required.** `--enable-preview` is pre-configured in `pom.xml` for the compiler, Surefire, and Failsafe. The Spring Boot Maven plugin also passes `--enable-preview` at runtime. If your IDE fails to compile, enable preview features manually.

## Infrastructure (Docker)

```bash
# Start infrastructure only (PostgreSQL+pgvector, Redis, Ollama, Kafka KRaft)
docker compose up -d

# Build + start all application containers
docker compose -f docker-compose-apps.yml up -d --build

# Run the end-to-end decoupled integration test script
./scripts/test-docker-decoupled.sh
```

Ports: PostgreSQL `5432`, Redis `6379` / Insight UI `8001`, Ollama `11434`, Kafka `9092`, Backend `8080`, Embedding Service `8082`, Frontend `3000`.

## Architecture

### Module Structure

| Module | Role |
|---|---|
| `oc-core` | Shared interfaces (`Connector`, `RepositoryConnector`, `OutputConnector`, `TransformationConnector`), Kafka message records (`IngestionMessage`, `DocumentChunkMessage`, `DocumentEmbeddedMessage`), `RepositoryDocument`, `SecurityConfig` / `PermissionRule` |
| `oc-runtime` | Spring Boot main app. REST API (`/api/jobs`, `/api/connectors`, `/api/system`), `JobOrchestrator`, `IngestionConsumer`, `VectorStoreWriterConsumer`, `McpVectorServer`. Depends on all other modules. |
| `oc-filesystem-repository-connector` | `FileSystemRepositoryConnector` — implements `RepositoryConnector`, returns a `Flux<RepositoryDocument>` by walking a local path |
| `oc-alfresco-repository-connector` | `AlfrescoRepositoryConnector` — uses Java `HttpClient` + Structured Concurrency to batch-fetch nodes from Alfresco REST API, streams content as Claim Check files |
| `oc-vector-output-connector` | `VectorOutputConnector` + `VectorStoreConfig` — configures four named `PgVectorStore` beans (`vector_store`, `vector_store_384`, `vector_store_768`, `vector_store_1024`) using `PrecomputedEmbeddingModel` (a no-op model that accepts pre-computed vectors from Kafka) |
| `oc-embedding-service` | Standalone Spring Boot app. `EmbeddingConsumer` reads chunks from Kafka, `EmbeddingModelFactory` dispatches to Ollama or OpenAI dynamically per-job, publishes embedded vectors back to Kafka |
| `oc-admin-ui` | Vite + React 19 + TailwindCSS admin dashboard. Built by Maven's frontend plugin and bundled as a static resource inside `oc-runtime`'s JAR |

### Kafka Pipeline (Claim Check Pattern)

```
RepositoryConnector.scan()
  -> JobOrchestrator publishes IngestionMessage (URI only, no content) -> [opencrawling-ingestion]
    -> IngestionConsumer: reads file from URI, Tika extract, TokenTextSplitter chunk -> [opencrawling-chunks]
      -> EmbeddingConsumer (oc-embedding-service): EmbeddingModelFactory -> [opencrawling-embedded]
        -> VectorStoreWriterConsumer: PrecomputedEmbeddingModel -> PgVectorStore
```

- For non-filesystem connectors (e.g., Alfresco), the connector saves content to a local `data/claims/` directory and sets the URI to that file path before publishing. The `IngestionConsumer` always reads from a local `file://` URI.
- The `EmbeddingModelFactory` caches model clients by `"engine-modelName"` key in a `ConcurrentHashMap`, so per-job model configs are hot after the first request.
- `VectorStoreWriterConsumer` uses `@ConditionalOnProperty` (`opencrawling.consumer.vector-writer.enabled`); `IngestionConsumer` uses `opencrawling.consumer.ingestion.enabled`. These allow deploying consumers as separate containers in the decoupled mode.

### MCP Server

`McpVectorServer` (in `oc-runtime`) exposes three `@McpTool` methods over SSE at `http://localhost:8080`:
- `secureVectorSearch` — similarity search with pgvector filter on `acl` field, then server-side re-check against the richer `SecurityConfig` JSON (OIS permissions model: `identity`, `identityType`, `access`)
- `getDocumentContent` — fetch full document chunks by URI with ACL guard

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [opencrawling/opencrawling](https://github.com/opencrawling/opencrawling) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
