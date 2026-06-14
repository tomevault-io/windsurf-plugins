---
trigger: always_on
description: > This document is intended for AI agents (Antigravity, Codex, etc.) to quickly orient themselves when working on this repository.
---

# memory-server — AI Agent Developer Guide

> This document is intended for AI agents (Antigravity, Codex, etc.) to quickly orient themselves when working on this repository.  
> Authoritative design reference: `../云记忆框架/设计文档/` (v1.3)

---

## Repository Overview

`memory-server` is the **sole core business backend** for the YuYi (羽忆) Unified Cloud Memory Infrastructure. It owns all business logic, API routing, memory judgment, recall orchestration, and security control.

- **Language / Framework**: Java 21 + Spring Boot 3.3.5
- **Database**: PostgreSQL 16 + pgvector (vector search) + zhparser (Chinese full-text search)
- **ORM**: MyBatis-Plus 3.5.7
- **Migrations**: Flyway
- **Auth model**: Admin HttpOnly Session Cookie + API Token (dual credential)
- **API docs**: SpringDoc OpenAPI 3.1, available at `/swagger-ui.html` at runtime
- **Build artifact**: `memory-server.jar` (fixed via `archiveFileName` in `build.gradle.kts`)

---

## Package Structure

Root package: `top.zyaire.memory`

| Sub-package | Responsibility |
|-------------|----------------|
| `auth` | Admin login / logout / `me` / bootstrap-status |
| `security` | API Token management (CRUD, revoke), auth filter chain, Spring Security config |
| `gateway` | Global exception handling (`advice/`), CORS config, OpenAPI config, rate-limit filter |
| `identity` | Current user resolution, Scope validation and auto-fill |
| `ingest` | Write Pipeline entry: write stable memories / history records |
| `judge` | Memory Judge: three-stage decision chain (hard guard → LLM → rule fallback) |
| `stable` | Stable Memory Service: CRUD, versioning, state machine, lexical/vector search |
| `history` | History Record Service: append writes, TTL policy, lexical/vector search |
| `recall` | Recall Orchestrator: hybrid retrieval → scoring → dedup → conflict detection → token budget |
| `search` | HybridSearchService: RRF fusion (lexical + vector), shared by recall and search endpoints |
| `embedding` | Embedding Provider management: create / activate / test; runtime embedding generation |
| `llm` | LLM Provider management; HTTP client wrapper for LLM calls in Judge |
| `audit` | Audit event recording and query |
| `dashboard` | Aggregated view APIs for Web Console (no business logic duplication) |
| `control` | Mode switching (manual / assistive / temporary), auto-write toggle |
| `common` | Unified response wrapper, exception hierarchy, enums, validators, config classes |

---

## Key Entry Points

| File | Description |
|------|-------------|
| `MemoryServerApplication.java` | Spring Boot main class, application entry point |
| `security/config/SecurityConfig.java` | Filter chain declaration, CORS config, public path whitelist |
| `gateway/advice/GlobalExceptionHandler.java` | Unified JSON error response for all unhandled exceptions |
| `common/response/ApiResponse.java` | **Unified response wrapper for all endpoints** — format: `{code, message, data, httpStatus, timestamp}` |
| `common/exception/ErrorCode.java` | Full business error code registry — format: `MEM-{category}{number}` |
| `common/config/MemoryProperties.java` | `memory.*` configuration property binding |

---

## Three Core Pipelines and Key Classes

### A. Write Pipeline

```
IngestController → IngestService → MemoryJudge → StableMemoryService / HistoryRecordService
```

| Class | Location | Description |
|-------|----------|-------------|
| `IngestController` | `ingest/controller/` | `POST /v1/memories`, `POST /v1/history-records` |
| `IngestService` | `ingest/service/` | Scope auto-fill, temporary mode check, Judge invocation, storage routing |
| `MemoryJudge` | `judge/service/` | Decision facade: hard guard first, then LLM, then rule fallback |
| `RuleEngine` | `judge/rule/` | L1 deterministic rule set (sensitive content, length, keyword type inference, importance scoring, semantic_key generation) |
| `LlmJudgeService` | `judge/service/` | L2 LLM call (structured JSON output validation, auto-fallback on low confidence) |

**Decision chain execution order** (strictly enforced in code, cannot be bypassed):

1. `RuleEngine.evaluateHardGuards()` — sensitive content / length violations → immediate `REJECT`, **LLM cannot override**
2. `LlmJudgeService.judge()` — optional; confidence must be ≥ `memory.judge.llm-confidence-threshold` (default `0.65`)
3. `RuleEngine.evaluateFallback()` — keyword-based type inference + importance + semantic_key + tags defaults

**Expected rejection scenarios** (400/403 on write):

- `MEM-MEM004` — Judge rejected (noise, too short, too long)
- `MEM-MEM007` — Sensitive content detected (password/key assignment pattern, Bearer token literal, PEM private key block)
- `MEM-MEM006` — Temporary mode is active

### B. Recall Pipeline

```
RecallController → RecallOrchestrator → HybridSearchService → ImportanceRecencyScorer → RecallContextBlock
```

| Class | Location | Description |
|-------|----------|-------------|
| `RecallController` | `recall/controller/` | `POST /v1/recall`, `GET /v1/memories/recent` |
| `RecallOrchestrator` | `recall/service/` | 8-step recall flow: candidates → score → sort → dedup → conflicts → budget → freshness → assemble |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PlumeMemory/memory-server](https://github.com/PlumeMemory/memory-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-14 -->
