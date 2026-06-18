---
trigger: always_on
description: Agent skill for docctl multi-format ingestion and provenance-grounded retrieval.
---


# docctl Skill

## When to use
- Use for document-grounded Q&A over local corpora (`.pdf`, `.docx`, `.txt`, `.md`).
- Use when answers must include provenance (`source`, `title`, `chunk_id`).
- Use when the agent can execute shell commands.

## Scope and non-scope
- In scope:
  - `docctl ingest`, `search`, `show`, `stats`, `doctor`, and `session` orchestration.
  - Full lifecycle behavior: bootstrap ingest plus retrieval loops.
  - Metadata-constrained retrieval using `doc_id`, `source`, and `title`.
  - Optional rerank controls and interpretation in retrieval workflows (`--rerank`, `--rerank-candidates`, session `rerank`, `rerank_candidates`).
- Out of scope (agent-owned responsibilities):
  - Query rewriting and query decomposition.
  - Conversation context handling and prior-turn memory policy.
  - Project-specific instruction interpretation and policy reasoning.
  - Hybrid keyword/full-text retrieval design.
  - Reranker model training/tuning and low-level scoring implementation.

## Inputs and assumptions
- Expected inputs:
  - user question,
  - user-preferred response language when observable from the conversation,
  - optional corpus path(s),
  - optional retrieval filters (`doc_id`, `source`, `title`),
  - optional index settings.
- Default CLI assumptions:
  - `--index-path ./.docctl`
  - `--collection default`
  - `--json` enabled for machine consumption.
- Safety assumption:
  - `ingest` is mutating and should only run under explicit lifecycle conditions.
- Language assumption:
  - infer a working retrieval language from the strongest available signal in this order:
    1. explicit user instruction,
    2. language of the latest user turn,
    3. language used by retrieved evidence or cited answers in the active workflow.
  - if the user asks in one language but the evidence and grounded answer are clearly in another, switch query rewriting and follow-up searches to the evidence language while preserving the user's requested answer language unless they ask to change it.

## Operational workflow (ordered)
1. Run readiness checks.
   - Default to `docctl catalog` for readiness and full index inventory.
   - Run `docctl stats` only when quick aggregate counts are specifically needed.
   - Run `docctl doctor` only when diagnostics are needed (for example command failures, config issues, or unexpected index behavior).
2. Apply bootstrap ingest rules (full lifecycle).
   - If index is missing or empty, run `docctl ingest <path>`.
   - Reingest only on explicit user intent or stale corpus signals (file updates/new files).
3. Prepare retrieval query in the agent layer.
   - Rewrite/expand/paraphrase outside `docctl` if needed.
   - Choose the search language from the current working retrieval language, not only from the latest user wording.
   - If the user asks in English but the relevant evidence and grounded answers are in German, reformulate the next retrieval attempts in German.
4. Execute retrieval (session-first).
   - Primary: `docctl session` with `op:"search"` for iterative loops.
   - For two or more read operations in one workflow, open one `docctl session` and send multiple NDJSON requests in that session.
   - Do not run multiple sequential one-shot read commands via repeated tool calls when `session` is available.
   - Secondary fallback: one-shot `docctl search`.
5. Run bounded evidence expansion loop.
   - If no or weak results, broaden query and/or relax filters.
   - If results indicate the corpus language differs from the current query language, retry in the corpus language before exhausting attempts.
   - Increase `top_k` per policy and retry up to max attempts.
6. Inspect top evidence chunks.
   - Call `show` for selected chunk IDs before synthesis when precision matters.
   - Treat high-value returned sentences/snippets as a lead and inspect the full returned chunk before final synthesis to capture qualifiers and surrounding context.
7. Synthesize answer with explicit citations.
   - Include provenance and state uncertainty when evidence is insufficient.

## Tool guidance (docctl command contracts)
- `ingest`:
  - Mutating operation.
  - Use when index is uninitialized/empty or corpus is stale.
  - Avoid repeated reingest unless needed.
- `search`:
  - Use for one-shot retrieval.
  - Do not chain multiple `search`/`show`/`stats`/`catalog` calls via separate tool invocations for the same workflow; switch to `session`.
  - Relevant options: `--doc-id`, `--source`, `--title`, `--top-k`, `--min-score`, `--rerank`, `--rerank-candidates`.
  - Rerank constraints: candidate depth must be in `[1, 100]` and greater than or equal to `top_k`.
- `session`:
  - Use for iterative retrieval workflows.
  - Preferred default for multi-step work: keep one session open and submit all read operations (`search`, `show`, `stats`, `catalog`, `doctor`) as NDJSON lines.
  - Supported operations: `search`, `show`, `stats`, `catalog`, `doctor`.
  - Search request accepts optional fields: `doc_id`, `source`, `title`, `top_k`, `min_score`, `rerank`, `rerank_candidates`.
- `show`:
  - Use to inspect and quote exact chunk evidence by `chunk_id`.
- `stats`:
  - Do not run by default in retrieval loops.
  - Use when quick aggregate counts are needed.
- `catalog`:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GeorgesAlkhouri/docctl](https://github.com/GeorgesAlkhouri/docctl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
