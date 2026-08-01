---
trigger: always_on
description: LightRAG is a mature, production-ready Retrieval-Augmented Generation (RAG) system with comprehensive knowledge graph capabilities. The system has evolved from experimental to production-ready status with extensive functionality across all major components.
---

# LightRAG Project Intelligence (.clinerules)

## Project Overview
LightRAG is a mature, production-ready Retrieval-Augmented Generation (RAG) system with comprehensive knowledge graph capabilities. The system has evolved from experimental to production-ready status with extensive functionality across all major components.

## Current System State (August 15, 2025)
- **Status**: Production Ready - Stable and Mature
- **Configuration**: Gemini 2.5 Flash + BAAI/bge-m3 embeddings via custom endpoints
- **Storage**: Default in-memory with file persistence (JsonKVStorage, NetworkXStorage, NanoVectorDBStorage)
- **Language**: Chinese for summaries
- **Workspace**: `space1` for data isolation
- **Authentication**: JWT-based with admin/user accounts

## Critical Implementation Patterns

### 1. Embedding Format Compatibility (CRITICAL)
**Pattern**: Always handle both base64 and raw array embedding formats
**Location**: `lightrag/llm/openai.py` - `openai_embed` function
**Issue**: Custom OpenAI-compatible endpoints return embeddings as raw arrays, not base64 strings
**Solution**:
```python
np.array(dp.embedding, dtype=np.float32) if isinstance(dp.embedding, list)
else np.frombuffer(base64.b64decode(dp.embedding), dtype=np.float32)
```
**Impact**: Document processing fails completely without this dual format support

### 2. Async Pattern Consistency (CRITICAL)
**Pattern**: Always await coroutines before calling methods on the result
**Common Error**: `coroutine.method()` instead of `(await coroutine).method()`
**Locations**: MongoDB implementations, Neo4j operations
**Example**: `await self._data.list_indexes()` then `await cursor.to_list()`

### 3. Storage Layer Data Compatibility (CRITICAL)
**Pattern**: Always filter deprecated/incompatible fields during deserialization
**Common Fields to Remove**: `content`, `_id` (MongoDB), database-specific fields
**Implementation**: `data.pop('field_name', None)` before creating dataclass objects
**Locations**: All storage implementations (JSON, Redis, MongoDB, PostgreSQL)

### 4. Lock Key Generation (CRITICAL)
**Pattern**: Always sort relationship pairs for consistent lock keys
**Implementation**: `sorted_key_parts = sorted([src, tgt])` then `f"{sorted_key_parts[0]}-{sorted_key_parts[1]}"`
**Impact**: Prevents deadlocks in concurrent relationship processing

### 5. Event Loop Management (CRITICAL)
**Pattern**: Handle event loop mismatches during shutdown gracefully
**Implementation**: Timeout + specific RuntimeError handling for "attached to a different loop"
**Location**: Neo4j storage finalization
**Impact**: Prevents application shutdown failures

### 6. Async Generator Lock Management (CRITICAL)
**Pattern**: Never hold locks across async generator yields - create snapshots instead
**Issue**: Holding locks while yielding causes deadlock when consumers need the same lock
**Location**: `lightrag/tools/migrate_llm_cache.py` - `stream_default_caches_json`
**Solution**: Create snapshot of data while holding lock, release lock, then iterate over snapshot
```python
# WRONG - Deadlock prone:
async with storage._storage_lock:
    for key, value in storage._data.items():
        batch[key] = value
        if len(batch) >= batch_size:
            yield batch  # Lock still held!

# CORRECT - Snapshot approach:
async with storage._storage_lock:
    matching_items = [(k, v) for k, v in storage._data.items() if condition]
# Lock released here
for key, value in matching_items:
    batch[key] = value
    if len(batch) >= batch_size:
        yield batch  # No lock held
```
**Impact**: Prevents deadlocks in Json→Json migrations and similar scenarios where source/target share locks
**Applicable To**: Any async generator that needs to access shared resources while yielding

## Architecture Patterns

### 1. Dependency Injection
**Pattern**: Pass configuration through object constructors, not direct imports
**Example**: OllamaAPI receives configuration through LightRAG object
**Benefit**: Better testability and modularity

### 2. Memory Bank Documentation
**Pattern**: Maintain comprehensive memory bank for development continuity
**Structure**: Core files (projectbrief.md, activeContext.md, progress.md, etc.)
**Purpose**: Essential for context preservation across development sessions

### 3. Configuration Management
**Pattern**: Centralize defaults in constants.py, use environment variables for runtime config
**Implementation**: Default values in constants, override via .env file
**Benefit**: Consistent configuration across components

## Development Workflow Patterns

### 1. Frontend Development (CRITICAL)
**Package Manager**: **ALWAYS USE BUN** - Never use npm or yarn unless Bun is unavailable
**Commands**:
- `bun install` - Install dependencies
- `bun run dev` - Start development server
- `bun run build` - Build for production
- `bun run lint` - Run linting
- `bun test` - Run tests
- `bun run preview` - Preview production build

**Pattern**: All frontend operations must use Bun commands
**Fallback**: Only use npm/yarn if Bun installation fails

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HKUDS/LightRAG](https://github.com/HKUDS/LightRAG) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
