---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Spring Boot 3 REST API that provides semantic search over industry codes (e.g., ICD-10 medical codes) using vector embeddings. It uses Ollama's `nomic-embed-text` model to generate 768-dimensional embeddings and PostgreSQL with the pgvector extension for vector similarity search.

**Core Technologies:**
- Java 21
- Spring Boot 3.5.6
- PostgreSQL 16 with pgvector extension
- Hibernate 6.6.4 with hibernate-vector
- Ollama (local LLM for embeddings)
- Maven
- Spring Retry (fault tolerance with exponential backoff)
- Apache Commons CSV (robust CSV parsing)
- Spring Boot Actuator (health monitoring)

## Development Commands

### Running the Application

**Local development (requires PostgreSQL + Ollama running locally):**
```bash
./mvnw spring-boot:run -Dspring-boot.run.profiles=local
```

**With Docker Compose (recommended):**
```bash
# Start services (PostgreSQL + Ollama)
docker-compose up -d

# Pull Ollama model (first time only)
./scripts/init-ollama.sh
# Or manually: docker exec semantic-search-ollama ollama pull nomic-embed-text

# Run Spring Boot app
./mvnw spring-boot:run
```

### Testing & Building

```bash
# Run tests
./mvnw test

# Build JAR
./mvnw clean package

# Run JAR
java -jar target/semantic-search-0.0.1-SNAPSHOT.jar
```

### Database Access

```bash
# Connect to PostgreSQL (Docker)
docker exec -it semantic-search-postgres psql -U postgres -d semantic_search

# Check if pgvector is installed
# Inside psql:
\dx vector

# View table schema
\d industry_codes

# Drop table to reset (WARNING: deletes all data)
DROP TABLE industry_codes;
```

## Architecture

### Request Flow

1. **Upload Flow** (POST /api/v1/codes or /api/v1/codes/upload-csv):
   - Controller receives code + description
   - OllamaService generates 768-dim embedding via HTTP to Ollama API (with retry on failure)
   - Embedding dimension validated (must be 768)
   - Entity saved to PostgreSQL with vector column and audit timestamps
   - CSV uploads use batched transactions (100 rows per batch, max 1000 rows total)
   - Each batch saved in separate transaction (REQUIRES_NEW) for partial success

2. **Search Flow** (GET /api/v1/codes/search):
   - Controller receives natural language query
   - OllamaService generates embedding for query (with retry on failure)
   - Repository uses pgvector's `<=>` operator (cosine distance)
   - Returns top N results sorted by similarity (1.0 = identical, 0.0 = unrelated)

### Key Components

**Entity Layer** (`entity/IndustryCode.java`):
- JPA entity with vector column using `@JdbcTypeCode(SqlTypes.VECTOR)`
- `@Array(length = 768)` specifies vector dimensions (must match Ollama model)
- Fluent setters return `this` for method chaining
- Audit fields with `@CreatedDate` and `@LastModifiedDate` (requires @EnableJpaAuditing)
- Unique constraint on `code` field (enforced at database level)
- Custom `equals()`, `hashCode()`, and `toString()` implementations

**Repository Layer** (`repository/CodeRepository.java`):
- Native SQL query using pgvector's `<=>` operator for cosine distance
- `findSimilarCodes()` returns interface projection `CodeSimilarity`
- Similarity calculated as `1 - (embedding <=> queryEmbedding)`

**Service Layer**:
- `OllamaService.java`: Wraps Ollama API calls, generates embeddings
  - `@Retryable` with exponential backoff (3 attempts: 1s, 2s, 4s delays)
  - Validates embedding dimensions after generation (must be 768)
  - Throws `OllamaApiException` on failures for proper error handling
- `CodeService.java`: Handles CSV parsing, CRUD operations, and batch processing
  - Uses Apache Commons CSV for robust parsing (handles quotes, commas, newlines)
  - Batched transactions with `@Transactional(propagation = REQUIRES_NEW)` for partial success
  - CSV row limit enforced (max 1000 rows, graceful termination if exceeded)
  - IOException handling for corrupted/invalid CSV files
  - Code uniqueness validation in updateCode() method

**Controller Layer** (`controller/CodeController.java`):
- REST endpoints at `/api/v1/codes` (API versioning)
- Coordinates between services and repository
- Manual mapping from `CodeSimilarity` projection to `SearchResult` DTO
- Bean validation with `@Valid`, `@NotBlank`, `@Min`, `@Max` annotations
- Full CRUD operations: create, read, update, delete, search, list, upload CSV

### Vector Search Implementation

**Critical details:**
- pgvector cosine distance operator: `<=>` (0 = identical, 2 = opposite)
- Similarity score: `1 - distance` (converts to 0-1 scale, higher = more similar)
- Query embedding must be cast: `cast(:queryEmbedding as vector)`
- Embedding dimension is hardcoded to 768 (nomic-embed-text model)

**Repository query:**
```sql
SELECT id, code, description, embedding,
       (1 - (embedding <=> cast(:queryEmbedding as vector))) as similarity
FROM industry_codes
WHERE embedding IS NOT NULL
ORDER BY embedding <=> cast(:queryEmbedding as vector)
LIMIT :limit
```

## Configuration

### Application Profiles

**Default** (`application.properties`):
- Uses container hostnames: `postgres:5432`, `ollama:11434`
- For Docker Compose deployments


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JeremyDahms/semantic-search](https://github.com/JeremyDahms/semantic-search) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
