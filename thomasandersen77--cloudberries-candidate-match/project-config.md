---
trigger: always_on
description: Cursor loads the same content from `.cursor/rules/cloudberries-backend.mdc` (keep in sync).
---


# Cloudberries Candidate Match Backend Rules (Modular Monolith)

Cursor loads the same content from `.cursor/rules/cloudberries-backend.mdc` (keep in sync).

---

## 0. Rule Contract (Always Enforced)

These rules must be followed for every change in this project:

1.  **Modular Monolith Architecture**
    *   `candidate-match`: Core service for CV analysis, project request matching, and consultant management. Acts as the primary executable. Port: 8080.
    *   `ai-rag-service`: Isolated module for RAG (Retrieval-Augmented Generation) and embeddings (Spring AI). Integrated into the main service but maintained as a modular dependency.
    *   `teknologi-barometer-service`: Isolated module for analyzing technology trends. Port: 8082.
    *   **Module Ownership**: Each module has its own domain and `pom.xml`.
    *   **DDD Anti-Corruption Layers (ACL)**: Communication between modules MUST use ports/adapters to prevent domain model leakage.

2.  **Layering (Strict)**
    *   `Controller` → `Service` (Domain/Application)
    *   `Service` → `Port` (Interface) or `Repository`
    *   `Port` → `Infrastructure Adapter` (External modules, APIs, AI Providers)

3.  **Ultra-thin Controllers**
    *   Controllers are responsible for: routing, extracting auth context, syntactic validation, and mapping DTOs ↔ Domain objects.
    *   Controllers must NOT contain: business rules, direct database queries, complex domain logic, or AI orchestration.

4.  **Domain-Driven Design (DDD) & Clean Architecture**
    *   **Core Isolation**: Domain logic should be isolated in `no.cloudberries.candidatematch.domain`. It must not depend on infrastructure details (JPA, Web, AI SDKs).
    *   **Entities**: Represent objects with identity. Use `class` (not `data class`) for JPA entities to avoid lazy-loading issues in `equals`/`hashCode`.
    *   **Value Objects**: Represent descriptive aspects of the domain with no identity. Use `data class` and ensure immutability.
    *   **Aggregate Roots**: Use repositories only for aggregate roots (e.g., `ProjectRequest`, `Consultant`).
    *   **Ports & Adapters**: Define interfaces in the domain/service layer (Ports) and implement them in the infrastructure layer (Adapters).

5.  **Test-Driven Development (TDD) & Quality**
    *   **Bug Fixes**: ALWAYS write a reproduction test before fixing a bug.
    *   **Unit Tests**: Required for all business logic in services and domain models. Use **MockK** for mocking.
    *   **Integration Tests**: Use `@SpringBootTest` with Testcontainers (Postgres/pgvector) or H2 for database verification.
    *   **MockMvc**: Use for testing API contracts, security (RBAC), and DTO mapping.

6.  **AI & LLM Integration (Specialized Rules)**
    *   **Provider Abstraction**: Never couple domain logic directly to a specific LLM. Use `EmbeddingProvider` or `AIClient` ports.
    *   **Model Strategy**:
        *   **Gemini 3 Pro Preview**: For PDF/requirement analysis.
        *   **Gemini 2.5 Pro**: For batch candidate ranking (Files API).
        *   **Claude 4.5 Sonnet**: Preferred high-end reasoning model.
        *   **Ollama (Local)**: `llama3.2:3b` for local reasoning, `bge-m3` for local embeddings.
    *   **Prompt Management**: Keep prompt templates in dedicated classes/files.
    *   **RAG Flow**: Integrated Retrieval-Augmented Generation logic (Ingestion, Retrieval, API).
    *   **Reliability**: Implement fallbacks (e.g., fallback to Gemini Flash if Pro 503s) and retries.

7.  **Data Handling & PostgreSQL**
    *   **pgvector**: All vector embeddings must be stored in PostgreSQL (port 5433) using the `vector` type.
    *   **Money**: All monetary values must be represented as `Long` in minor units (e.g., øre). Floating-point types are **forbidden**.
    *   **Transactions**: All state-changing service methods MUST be annotated with `@Transactional`.

8.  **Security & Identity**
    *   **Authentication**: Use `@CurrentUser` in controller signatures. No certificate auth for local development (username/password only).
    *   **Authorization**: Enforce RBAC in the service layer.
    *   **DTO Boundary**: Never return domain entities or infrastructure entities directly in API responses.

9.  **Naming & Language**
    *   **English**: All code identifiers, database tables/columns, and technical documentation.
    *   **Norwegian**: Exception messages for end-users and functional/business log messages.

10. **SOLID Principles**
    *   **SRP**: Each class should have one reason to change.
    *   **DIP**: Depend on abstractions, not concretions.

---

## 1. Local Development Setup

### 1.1 Toolchain (Mandatory via SDKMAN)
- **Java**: 21.0.7-tem
- **Maven**: Latest
- **Docker**: For PostgreSQL with pgvector

### 1.2 Database (Docker Compose)
- **Image**: `pgvector/pgvector:pg15`
- **Port**: 5433 (Local) mapping to 5432 (Container)
- **Auth**: Username/Password only (`candidatematch`/`candidatematch123`)
- **Startup**: `docker-compose -f candidate-match/docker-compose-local.yaml up -d`

---

## 2. System Architecture Details

### 2.1 `candidate-match` Structure (The Magnificent Monolith)
- **`controllers`**: REST endpoints, thin, mapping only.
- **`service`**: Orchestrates use-cases, contains domain logic.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/thomasandersen77) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
