---
trigger: always_on
description: > Single source of truth for the project across AI runtimes. Claude Code, Codex, Cursor, Copilot, etc. all defer to this file.
---

# AGENTS.md

> Single source of truth for the project across AI runtimes. Claude Code, Codex, Cursor, Copilot, etc. all defer to this file.
> Runtime-specific adaptation belongs in each runtime's own file (Claude reads `CLAUDE.md`); do not mix it in here.

## Project Overview

**MemOS / MemoryOS**: a memory operating system for LLM agents. Python library plus a FastAPI service, providing multiple memory types (textual / tree / preference / skill / KV cache / LoRA parametric) plus scheduling, version management, and vector & graph storage.

- **Repository**: https://github.com/MemTensor/MemOS
- **Documentation**: https://memos-docs.openmem.net/home/overview/
- **PyPI**: https://pypi.org/project/MemoryOS/
- **License**: Apache-2.0
- **Top-level package**: `src/memos/`. Distribution name `MemoryOS`; import name `memos`.
- **CLI**: `memos` (entry `memos.cli:main`)
- **API service**: `memos.api.start_api:app`

## Repository Layout

| Path | Purpose |
|------|---------|
| `src/memos/mem_os/` | `MOS` / `MOSCore` — top-level Memory OS entry |
| `src/memos/mem_cube/` | `GeneralMemCube` — memory container aggregating multiple memory types |
| `src/memos/memories/` | Memory implementations: `textual/`, `activation/`, `parametric/` |
| `src/memos/mem_scheduler/` | Memory scheduler + monitors + ORM + task scheduling |
| `src/memos/mem_user/` | User / multi-tenant management (MySQL / Redis backends) |
| `src/memos/mem_chat/` `mem_reader/` `mem_agent/` `mem_feedback/` `multi_mem_cube/` | Chat sessions, ingest pipeline, agent integration, feedback channel, multi-cube routing |
| `src/memos/llms/` `embedders/` `vec_dbs/` `graph_dbs/` `chunkers/` `parsers/` `reranker/` | Provider implementations (`base.py` + `factory.py` + each backend) |
| `src/memos/api/` | FastAPI service (routers / handlers / middleware / MCP server) |
| `src/memos/configs/` | All pydantic configuration classes (one-to-one with the modules above) |
| `src/memos/context/` | Cross-thread context (trace_id / user / env) |
| `tests/` | pytest cases, subdirectories mirror `src/memos/` |
| `apps/` | Independent sub-projects, each with its own README; not part of the main Harness flow |
| `extensions/` | Official plugin examples |
| `docker/` `docs/` `evaluation/` `scripts/` | Deployment, documentation, evaluation, helper scripts |
| `.claude/agents/`, `.codex/agents/` | Project-recommended AI sub-agent definitions |

## Command Cheatsheet

- Install: `make install` (= `poetry install --extras all --with dev --with test` + pre-commit + push hook)
- Start API: `make serve`
- Export OpenAPI: `make openapi` (writes to `docs/openapi.json`)
- Run full tests: `make test`
- Run a single test: `poetry run pytest tests/<path>/test_xxx.py -q`
- Lint + format: `make format`
- Full pre-commit: `make pre_commit`
- Build: `poetry build` (publishing is automated by `python-release.yml` on GitHub release)

## Core API

### Python top-level entries (`from memos import ...`)

| Symbol | Purpose | Source |
|--------|---------|--------|
| `MOS` | Memory OS top-level entry (inherits `MOSCore`) | `memos.mem_os.main` |
| `GeneralMemCube` | General memory container | `memos.mem_cube.general` |
| `MOSConfig` / `GeneralMemCubeConfig` | Primary configs | `memos.configs.mem_os` / `memos.configs.mem_cube` |
| `GeneralScheduler` / `SchedulerFactory` / `SchedulerConfigFactory` | Scheduler and factories | `memos.mem_scheduler.*` |

Common `MOS` methods: `MOS.simple()` (auto-configure from env), `register_mem_cube(cube)`, `add(...)`, `search(...)`, `chat(...)`, `create_user(...)` / `list_users()`.

### API entry

- ASGI app: `memos.api.start_api:app`
- Routers: `src/memos/api/routers/` (`admin_router`, `product_router`, `server_router`)
- OpenAPI contract: `docs/openapi.json` (must run `make openapi` after touching the API)

## Import Patterns

| Use | Import |
|-----|--------|
| Top-level entries | `from memos import MOS, GeneralMemCube, MOSConfig` |
| Config classes | `from memos.configs.<submodule> import <Config>` |
| Any provider factory | `from memos.<category>.factory import <Category>Factory` |
| Logger | `from memos.log import get_logger`; `logger = get_logger(__name__)` |
| Context (trace) | `from memos.context.context import get_current_trace_id, get_current_user_name` |
| Exceptions | `from memos.exceptions import <semantic Exception>` |

## Provider Matrix

Every provider follows the same three-piece pattern: `base.py` abstract class + `factory.py` registry + `configs/<category>.py` config. The authoritative list of registered backends is the factory's `backend_to_class`; the snapshot below is provided for quick reference:

| Category | Base class | Factory | Registered backends |
|----------|-----------|---------|---------------------|
| LLM | `BaseLLM` | `LLMFactory` | `openai` / `openai_new` / `azure` / `ollama` / `huggingface` / `huggingface_singleton` / `vllm` / `qwen` / `deepseek` |
| Embedder | `BaseEmbedder` | `EmbedderFactory` | `ollama` / `sentence_transformer` / `ark` / `universal_api` |
| Vector DB | `BaseVecDB` | `VecDBFactory` | `qdrant` / `milvus` |
| Graph DB | `BaseGraphDB` | `GraphStoreFactory` | `neo4j` / `neo4j_community` / `nebular` / `polardb` / `postgres` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MemTensor/MemOS](https://github.com/MemTensor/MemOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
