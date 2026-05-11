---
trigger: always_on
description: Docdex URL: http://127.0.0.1:28491
---

---- START OF DOCDEX INFO V0.2.73 ----
Docdex URL: http://127.0.0.1:28491
Use this base URL for Docdex HTTP endpoints.
Health check endpoint: `GET /healthz` (not `/v1/health`).

## Priority Directives (Mandatory)

- Always use both memory lobes early: docdex_get_profile + docdex_memory_recall; save new facts via docdex_memory_save and new preferences via docdex_save_preference.
- Profile memory is always-on and global; never gate on DOCDEX_ENABLE_MEMORY or repo state. docdex_save_preference must persist (fallback if embeddings/LLM unavailable). If profile calls fail, treat it as a bug.
- Maintain agent-specific profile memory: use your agent_id with docdex_get_profile/docdex_save_preference; store generic self-knowledge (role, tooling, preferences).
- Use impact analysis for every code change: prefer MCP tools `docdex_impact_graph` / `docdex_dag_export` (IPC/HTTP). If shell networking is blocked, do not use curl; use MCP/IPC instead. If unavailable, state it and proceed cautiously.
- Apply DAG reasoning for planning: prefer dependency graph facts (impact results and /v1/dag/export) to choose the right change order and scope.
- Use Docdex tools intentionally: docdex_search/symbols/ast for repo truth; docdex_stats/files/repo_inspect/index for index health.
- When session history matters, use conversation-memory tools intentionally: archive/search/read with `docdex_conversation_*`, build compact context with `docdex_wakeup`, record durable notes with `docdex_diary_*`, and inspect temporal graph state with `docdex_kg_*` instead of replaying full transcripts.
- For folder structure, use docdex_tree instead of raw `rg --files`/`find` to avoid noisy folders.
- When you do not know something, run docdex_web_research (force_web=true). Web research is encouraged by default for non-repo facts and external APIs.
- When a Docdex feature makes a task easier/safer, you MUST use it instead of ad-hoc inspection. Examples: `docdex_search` for context, `docdex_open`/`/v1/snippet` for file slices, `docdex_symbols`/`docdex_ast` for structure, `docdex_impact_graph`/`docdex_impact_diagnostics` for dependency safety, and `docdex_dag_export` to review session traces.
- For dependency/library docs, run `docdexd libs discover`/`docdexd libs fetch` and search with `include_libs` rather than web searching.
- For test execution, use `docdexd run-tests` (or `docdexd test run-node`) instead of ad-hoc commands when feasible.
- For daemon stability triage, prioritize FD hardening checks: startup nofile warning threshold (`DOCDEX_MIN_NOFILE_SOFT`), profile lock retry knobs (`DOCDEX_PROFILE_LOCK_MAX_ATTEMPTS`, `DOCDEX_PROFILE_LOCK_RETRY_BASE_MS`), installer pressure defaults (`DOCDEX_REPO_IDLE_SECONDS`, `DOCDEX_REPO_HIBERNATE_SECONDS`, `DOCDEX_REPO_CLEANUP_INTERVAL_SECONDS`), and `docs/ops/fd_exhaustion_playbook.md`.
- For staged-change validation, use `docdexd hook pre-commit`.
- For MCP client registration, use `docdexd mcp add` (or `--all`) instead of editing configs by hand.

# Docdex Agent Usage Instructions

> Context for AI Agents: Docdex is your local-first Dual-Lobe Memory and Code Intelligence daemon. Unlike simple vector stores, it provides structural understanding of code (AST/Graph), persistent behavioral profiles (Agent Memory), and gated web enrichment, all strictly scoped to the local machine.

## Identity & Architecture

Docdex (Documentation Indexer) serves as your persistent "brain" on the user's machine. It operates on a Waterfall Retrieval model:

1. Local First (Tier 1): Instant search of repo code, symbols, and ingested library documentation.
2. Web Enrichment (Tier 2): Gated fallback to the configured web provider (DuckDuckGo by default, `mswarm` when selected) plus Headless Chrome when local confidence is low or explicitly requested.
3. Cognition (Tier 3): Local LLM inference (Ollama) with context assembly.

Key Constraints:

- Repo Isolation: Data never bleeds between repositories. You must identify the active repo for every operation.
- Hierarchy of Truth: Technical Truth (Code/Repo Memory) > Behavioral Truth (Profile Memory).
- Privacy: Code is never uploaded to a cloud vector store.

## The Dual-Lobe Memory System

Docdex v2.1 introduces a strict separation between "facts" and "preferences." Use the correct lobe for the task.

### 1. Repo Memory (The Hippocampus)

- Scope: Project-bound. Specific to the current repository.
- Content: Technical facts, architectural decisions, logic locations.
- Example: "The calculateTax function is located in utils/money.ts."
- Tools: docdex_memory_save, docdex_memory_recall

### 2. Profile Memory (The Neocortex)

- Scope: Global / agent-bound. Persists across all projects.
- Content: Your persona, user preferences, coding style, tooling constraints.
- Example: "Always use Zod for validation," or "User prefers strict TypeScript types."
- Tools: docdex_save_preference, docdex_get_profile
- Agent-specific: Each agent should use its own agent_id and store generic self-knowledge (role, tooling, preferences).
- Usage: Use this to "learn" from corrections. If a user corrects your style, save it here so you do not repeat the mistake in a different repo.

## Tool Capabilities (MCP & HTTP)

### A. Semantic Search & Web (Waterfall)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bekirdag/docdex](https://github.com/bekirdag/docdex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
