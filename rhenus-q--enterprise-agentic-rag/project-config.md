---
trigger: always_on
description: This file is the durable source of repository guidance for Codex and other coding agents.
---

# Repository Guidelines

This file is the durable source of repository guidance for Codex and other coding agents.
It applies to the entire repository unless a nested `AGENTS.md` or
`AGENTS.override.md` provides more specific instructions.

## Project overview

This repository implements an enterprise internal-document Q&A assistant with
LangGraph. The self-correcting Agentic RAG workflow routes between local retrieval and
web search, grades document relevance, checks answer grounding and usefulness, and
caps retry loops.

The application stack is LangGraph, LangChain, Chroma, Tavily, and `uv`. OpenAI
(`gpt-5-mini` and `OpenAIEmbeddings`) is the default model provider; the optional
process-level local-provider mode uses `ChatOllama` and `OllamaEmbeddings` through an
Ollama-compatible endpoint. Read `README.md` for setup and usage and `structure.md` for
the detailed architecture map.

External dependency failures must not crash the graph. Retrieval, web search,
generation, graders, and query rewriting degrade or stop safely and record an honest
`stop_reason`. Console banners log exception types, not exception messages.
Deployment-mode and provider configuration is validated before the graph by
`main.py::run_startup_preflight()` so privacy-sensitive misconfiguration fails clearly.

## Important paths

- `main.py`: interactive CLI entry point and startup preflight for deployment modes,
  providers, and local endpoint/model/index compatibility.
- `graph/engine.py`: canonical programmatic API, state seeding, streaming merge,
  observability, and metadata-only traces.
- `graph/graph.py`: graph assembly, routing, and `MAX_RETRIES`.
- `graph/state.py`: `GraphState` schema.
- `graph/config.py`: privacy and deployment modes, provider selection, fallback policy,
  request timeout, and per-run budget settings.
- `graph/nodes/`: graph node implementations and terminal notice nodes.
- `graph/chains/`: lazy LCEL chain factories.
- `graph/chains/_llm.py`: shared provider-aware chat-model factory used by all six
  chains.
- `graph/formatting.py`: pure answer/source formatting.
- `ingestion.py`: corpus loading, splitting, embedding, and provider-scoped Chroma
  rebuilds. It resets the active collection, uses deterministic chunk ids, writes an
  embedding-fingerprint sidecar, and exposes a lazy process-cached retriever. A failed
  mid-ingestion rebuild can leave the active index empty until ingestion succeeds again.
- `server/`: thin FastAPI adapter over the engine, including API schemas, status and
  document views, metadata-only run history, error mapping, and cooperative cancellation.
- `frontend/`: Vite, React, and TypeScript UI. Its API types mirror `server/schemas.py`;
  colocated Vitest tests cover critical states.
- `data/acmecorp_internal_docs/`: synthetic corpus; it contains no real company data.
- `tests/node/`, `tests/graph/`, `tests/evals/`, `tests/server/`, and
  `tests/test_env_isolation.py`: keys-free tests collected by default CI. Pure generation
  helper and empty-context tests live in `tests/node/test_generation_context_delimiters.py`
  and `tests/node/test_generation_short_circuit.py`.
- `tests/chains/`: real-model integration tests requiring `OPENAI_API_KEY`; default CI
  excludes the directory. The current suite covers five chain modules and has no live
  query-rewriter integration test.
- `evals/`: behavioral evaluation harness; full runs call real services.
- `docs/adr/`: architecture decision records.
- `docs/roadmap/`: specifications, plans, implementation reports, and review reports.
- `.agents/skills/`: repository-specific Codex workflows.
- `.codex/config.toml`: trusted-project Codex configuration and MCP servers.

## Architecture and implementation rules

- Preserve behavior by default. Do not change graph routing, the `GraphState` schema,
  prompts, application model names, `temperature=0`, chain input variables, or node
  return structures unless the user explicitly requests it.
- Avoid broad architecture changes and wholesale rewrites. Prefer small, mechanical,
  reviewable diffs.
- Keep `GraphState` fields as plain last-value channels. Do not add
  `typing.Annotated` reducers or accumulating channels without first redesigning the
  `dict.update()` streaming merge in `graph/engine.py`.
- Construct `ChatOpenAI`, `OpenAIEmbeddings`, `ChatOllama`, `OllamaEmbeddings`,
  `TavilyClient` (`tavily-python`), `Chroma`, retrievers, and other API-backed clients
  only inside lazy factories. Cache them with `@lru_cache(maxsize=1)` only where
  process-lifetime reuse and cache invalidation are intentional. Import optional
  provider-specific classes inside their factory.
- Route all six chains through `graph/chains/_llm.py::get_chat_model()`. Provider
  selection is process-level, never per-chain or per-run; do not construct a provider
  client in an individual chain module.
- Keep imports side-effect-free. Importing application modules must not require API
  keys, access the network, or construct external clients.
- Preserve lazy backward-compatible chain names through module-level `__getattr__`;
  do not reintroduce eager module-level chain objects.
- Keep code comments and docstrings in English.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rhenus-Q/Enterprise-Agentic-RAG](https://github.com/rhenus-Q/Enterprise-Agentic-RAG) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
