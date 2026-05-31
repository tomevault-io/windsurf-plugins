---
trigger: always_on
description: LightRAG is a Retrieval-Augmented Generation (RAG) framework that uses graph-based knowledge representation for enhanced information retrieval. The system extracts entities and relationships from documents, builds a knowledge graph, and uses multiple retrieval modes (`local`, `global`, `hybrid`, `mix`, `naive`) for queries.
---

# Repository Guidelines

## Project Overview

LightRAG is a Retrieval-Augmented Generation (RAG) framework that uses graph-based knowledge representation for enhanced information retrieval. The system extracts entities and relationships from documents, builds a knowledge graph, and uses multiple retrieval modes (`local`, `global`, `hybrid`, `mix`, `naive`) for queries.

## Project Structure

Top-level directories:

- **lightrag/**: Core Python package — see *Module Layout* below.
- **lightrag_webui/**: React 19 + TypeScript client (Bun + Vite + Tailwind). UI components in `src/`.
- **scripts/**: `test.sh` (preferred test runner), `setup/` interactive environment wizard (use `make env-*` rather than calling `setup.sh` directly — see *Configuration > Setup Wizard Outputs*), and release tooling.
- **tests/**: Pytest coverage, organized into subdirectories that mirror `lightrag/` (see *Testing* below for layout). Working datasets stay in `inputs/`, `rag_storage/`, and `temp/`; deployment collateral lives in `docs/`, `k8s-deploy/`, and compose files.

### Module Layout (`lightrag/`)

- **lightrag.py**: Main orchestrator class (`LightRAG`) — assembled from mixins (see *LightRAG class composition*). Hosts `ainsert_custom_kg`, `_insert_done`, `_process_extract_entities`, `_refresh_addon_params_cache`, and `addon_params` accessors. Critical: always call `await rag.initialize_storages()` after instantiation.
- **pipeline.py**: `_PipelineMixin` — owns the document ingestion pipeline (`apipeline_enqueue_documents`, `apipeline_process_enqueue_documents`, `apipeline_process_error_documents`), the `parse_native` / `parse_mineru` / `parse_docling` parser dispatchers, multimodal analysis, validation, and the worker scaffolding.
- **utils_pipeline.py**: Pure helpers shared by the pipeline mixin and other entry points: doc-status field access, document identity (source key, content hash), parsed-artifact path resolution, parser payload normalization, multimodal entity augmentation, and `make_lightrag_doc_content`.
- **llm_roles.py**: `RoleSpec` / `RoleLLMConfig` / `_RoleLLMState` / `ROLES` registry plus `_RoleLLMMixin` — role normalization, builder registration, wrapper rebuild, runtime config update, queue cleanup, sanitized config export, queue status reporting. Route role-specific behavior here rather than into provider modules.
- **storage_migrations.py**: `_StorageMigrationMixin` — `check_and_migrate_data`, `_migrate_entity_relation_data`, `_migrate_chunk_tracking_storage`.
- **addon_params.py**: `ObservableAddonParams` plus `default_addon_params` / `normalize_addon_params` helpers.
- **operate.py**: Core extraction and query operations including entity/relation extraction, chunking, and multi-mode retrieval logic.
- **base.py**: Abstract base classes for storage backends (`BaseKVStorage`, `BaseVectorStorage`, `BaseGraphStorage`, `BaseDocStatusStorage`).
- **kg/**: Storage implementations (JSON, NetworkX, Neo4j, PostgreSQL, MongoDB, Redis, Milvus, Qdrant, Faiss, Memgraph, OpenSearch, NanoVectorDB). The backend registry (`STORAGE_IMPLEMENTATIONS` / `STORAGES`) lives in `kg/__init__.py`; `kg/factory.py::get_storage_class()` resolves backend classes from configuration.
- **llm/**: LLM and embedding provider bindings (OpenAI, Ollama, Azure, Gemini, Bedrock, Anthropic, etc.). All async with caching support.
- **parser/**: Unified parsing layer. `parser/routing.py` resolves engine and filename hints for `legacy`, `native`, `mineru`, and `docling` flows; `parser/debug.py` provides an offline LightRAG stub for the `parser/cli.py` debug entry point (`python -m lightrag.parser.cli`). Native format parsers live as sibling sub-packages under `parser/` (currently `parser/docx/`); external HTTP-based adapters live under `parser/external/` (`mineru`, `docling`) with shared helpers in `parser/external/_common.py`, `_manifest.py`, `_zip.py`.
- **chunker/**: Chunking strategies (token-size, recursive character, semantic vector, paragraph semantic).
- **api/**: FastAPI service (`lightrag_server.py`) with REST endpoints and Ollama-compatible API; routers under `routers/`, static Swagger assets, packaged WebUI output, and Gunicorn launcher.

## Core Architecture

### LightRAG class composition

`LightRAG` is assembled from focused mixins (split out of the previously monolithic `lightrag.py`):

```
LightRAG → _RoleLLMMixin → _StorageMigrationMixin → _PipelineMixin → object
```

The `@final` decorator on `LightRAG` is preserved — the mixin layering is an internal implementation detail, not an external subclassing surface. The public API (`ainsert`, `aquery`, `ainsert_custom_kg`, `initialize_storages`, etc.) is unchanged. `ainsert_custom_kg` and its internal construction logic, `_insert_done`, `_process_extract_entities`, `_refresh_addon_params_cache`, and the `addon_params` property accessors stay on `LightRAG` itself because they cut across multiple flows or depend on prompt-profile state.

### Storage Layer

LightRAG uses 4 storage types with pluggable backends:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HKUDS/LightRAG](https://github.com/HKUDS/LightRAG) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
