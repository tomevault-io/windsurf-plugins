---
trigger: always_on
description: Guidance for Claude Code when working in this repository.
---

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## 1. Project Overview

This repository — **Enterprise Office Agent** — is organized as named capability modules
(see [ADR 014](docs/adr/enterprise_rag/014-enterprise-rag-package-and-office-agent-placeholder.md)):

- **`enterprise_rag/`** — ✅ **the completed module**, and the subject of everything below:
  an **enterprise internal-document Q&A engine** built with **LangGraph**, implementing a
  self-correcting Agentic RAG (CRAG-style) workflow. It answers questions from an ingested
  knowledge base and falls back to web search when needed. Public entry point:
  `enterprise_rag.graph.engine.answer_question()`.
- **`office_agent/`** — ✅ **implemented — seven capabilities, complete since v1.6.0 / Phase 7.** A
  deterministic, LLM-free intent router — entry point `office_agent.engine.answer_office_request(user_input, options=None)`
  — over local capabilities. Version map: **v1.0.0 / Phases 1-5** — **Knowledge Q&A** (a thin
  adapter over the `enterprise_rag` engine), **Email Summary**, **Calendar Lookup**,
  **Task / Ticket Assistant**, **Daily Briefing**; **v1.5.0 / Phase 6** — **Meeting Agent /
  Meeting Prep**; **v1.6.0 / Phase 7** — **Workflow / Approval Agent**. All tools except
  Knowledge Q&A run on local mock data with no LLM and no external services — the sole
  exceptions are two **optional, default-off** LLM assists, the email digest
  ([ADR 017](docs/adr/office_agent/017-office-agent-llm-assist-email-digest.md)) and the Daily Briefing
  narrative ([ADR 018](docs/adr/office_agent/018-office-agent-llm-assist-daily-briefing.md)), both gated
  by the single `OFFICE_LLM_ENABLED` switch and inert unless it is set. Office-agent
  work **must not change or regress `enterprise_rag` behavior or its tests** (§3 rules apply:
  side-effect-free imports, lazy `@lru_cache` external clients). See
  [`office_agent/README.md`](office_agent/README.md) (the dedicated Office Agent
  demo / usage doc) and [ADR 015](docs/adr/office_agent/015-office-agent-v1-architecture.md); office-agent
  working rules are in §3.

Root docs (`README.md`, `CLAUDE.md`, `structure.md`, `docs/adr/`) are repository-level;
detailed engine usage lives in `enterprise_rag/README.md`, the dedicated Office Agent demo /
usage doc is `office_agent/README.md`, and engineering / release docs live under
`docs/engineering/` and `docs/releases/` (plus the employee-facing quickstart in
`docs/employee-guide/`).
Most of this file is guidance for working in `enterprise_rag`; office-agent-specific rules are
called out in §3.

**Stack:** LangGraph, LangChain, OpenAI (`gpt-5-mini`, `OpenAIEmbeddings`), Chroma (vector
store), Tavily (web search). Managed with **uv**.

**High-level flow** (see `structure.md` for details):

```
question
→ route_question
    ├── websearch → generate
    └── retrieve → grade_documents
            ├── relevant docs → generate
            └── no relevant docs → websearch → generate
generate
→ grounding check (hallucination_grader)
    ├── not grounded → add grounding feedback → regenerate
    └── grounded → usefulness check (answer_grader)
            ├── useful → END
            └── not useful → rewrite search query → websearch
```

Three quality gates: **document relevance**, **answer grounding** (anti-hallucination), and
**answer usefulness**. A `retries` counter in state caps the regenerate/websearch loop at
`MAX_RETRIES = 5` (defined in `enterprise_rag/graph/graph.py`).

External dependency failures (retriever, Tavily, generation LLM, graders, query rewriter)
never crash the graph: each call site catches the exception, degrades or stops safely, and
records a `stop_reason` (`retrieval_error`, `web_search_error`, `generation_error`,
`tool_error`) so the RAG CLI (`enterprise_rag/cli.py`) appends an honest caveat. Console banners
log only the exception type, never the message.

**Runtime privacy modes** (default off, strict truthy parsing — see
[ADR 019](docs/adr/019-hierarchical-runtime-privacy-modes.md)):

- **`PRIVACY_MODE`** — no data leaves the machine except to OpenAI: forces off Tavily web
  search, LangSmith tracing, and both optional Office LLM assists, while preserving the core
  OpenAI RAG path unchanged.
- **`OFFLINE_MODE`** — higher precedence: implies every `PRIVACY_MODE` restriction and
  additionally disables OpenAI chat/embeddings, ingestion, and every other external-service
  path, failing closed with the additive `offline_mode` stop reason.

Precedence is strict and one-directional:
`OFFLINE_MODE` > `PRIVACY_MODE` > individual environment flags > per-run overrides.
**A mode can only restrict** — while active it overrides `WEB_SEARCH_ENABLED=true`,
`OFFICE_LLM_ENABLED=true`, the tracing variables, and an explicit
`AnswerOptions(web_search_enabled=True)`; no lower-level flag can re-enable an external
service.

## 2. Project Structure

| Path | Purpose |
|------|---------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rhenus-Q/Enterprise-Office-Agent](https://github.com/rhenus-Q/Enterprise-Office-Agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
