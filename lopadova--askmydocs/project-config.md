---
trigger: always_on
description: validates (`{ documents.*.project_key + content }`, not
---

# Copilot instructions — AskMyDocs

Mirror of `CLAUDE.md` (root) with the same rules. Whichever assistant edits
this repo, the rules are identical. Skills with detailed examples live under
`.claude/skills/`.

---

## 1. Project at a glance

AskMyDocs is an **enterprise RAG + canonical knowledge compilation** system
on **Laravel 13 + PostgreSQL + pgvector**. Markdown in, grounded answers
with citations out — over a **typed knowledge base** with a lightweight
graph, anti-repetition memory, and a human-gated promotion pipeline.
Optional chat history, feedback/few-shot, hybrid (semantic + FTS) search,
MCP server (10 tools), and a GitHub-Action-based cross-repo ingestion
pipeline. A full React SPA admin shell rides alongside at `/app/*`:
dashboard, users + roles + RBAC, canonical KB explorer with inline
editor and graph viewer, five-tab log viewer, whitelisted Artisan
maintenance runner, and a daily AI insights panel. Every admin page is
Spatie-role-gated and every mutation is audit-trailed
(`kb_canonical_audit` for canonical changes, `admin_command_audit` for
commands).

- PHP `^8.3`, Laravel `^13.0`, Sanctum `^4.2`.
- `symfony/yaml ^7.4|^8.0` for canonical YAML frontmatter parsing.
  Section-aware markdown chunking is custom (line-based fence-aware
  FSM in `MarkdownChunker`) — no external markdown parser library.
- `laravel/mcp ^0.7` as a suggest (required only when exposing the
  `enterprise-kb` MCP server).
- PostgreSQL ≥ 15 + `pgvector`. FTS GIN index migration ships pgsql-only.
- All providers run on the `laravel/ai` SDK (since v8.16/W2, ADR 0015 —
  reverses the earlier "No AI SDK" rule, so FinOps meters every provider
  natively). Anthropic + Gemini fully SDK; OpenAI + OpenRouter HYBRID —
  no-tools chat + embeddings via the SDK, the MCP with-tools turn on raw
  `Illuminate\Support\Facades\Http` `/chat/completions` (the SDK can't host
  AskMyDocs's external-MCP tool loop). Regolo via the
  `padosoft/laravel-ai-regolo` SDK adapter. `laravel/ai` pinned `^0.6.8`.
- Tests: PHPUnit 12 + Orchestra Testbench 11 (SQLite) + Vitest for JS.

---

## 2. Core flows

**Chat** — `KbChatController` → `KbSearchService::searchWithContext()`
(pgvector + optional FTS + `Reranker` fusion `0.55·vec + 0.25·kw + 0.05·head`
shipped defaults, via `kb.reranking.*` + canonical boost + status penalty) → `GraphExpander` (1-hop walk of
`kb_edges` from canonical seeds, config-gated) → `RejectedApproachInjector`
(cosine-correlates query vs `rejected-approach` canonical docs) →
`SearchResult{ primary, expanded, rejected, meta }` → prompt from
`resources/views/prompts/kb_rag.blade.php` (typed blocks: `⚠ REJECTED
APPROACHES` + `📎 RELATED CONTEXT` + primary `## Context`) →
`AiManager::chat()` → `ChatLogManager::log()` (try/catch, never
propagates). Graph expansion + rejected injection no-op when no canonical
docs exist (zero regression for non-canonical consumers).

**Ingest** — two entrypoints converge on one execution path:

- `php artisan kb:ingest-folder` walks the KB disk, dispatches one job per
  file.
- `POST /api/kb/ingest` (Sanctum, ≤ 100 docs/call) writes to the KB disk,
  dispatches one job per doc.
- Both → `IngestDocumentJob` (`$tries = 3`, backoff `[10,30,60]`) →
  `DocumentIngestor::ingestMarkdown()` (SHA-256 upsert on
  `(project_key, source_path, version_hash)` — idempotent by construction).

**Canonical branch** — when the markdown has a valid YAML frontmatter,
`DocumentIngestor` populates the 8 canonical columns (`doc_id`, `slug`,
`canonical_type`, `canonical_status`, `is_canonical`, `retrieval_priority`,
`source_of_truth`, `frontmatter_json` with `_derived` slugs). Prior
canonical identifiers are vacated before the new version is inserted to
avoid violating the per-project composite uniques. After commit,
`CanonicalIndexerJob` populates `kb_nodes` + `kb_edges` from the
frontmatter `_derived` slug lists and every chunk's `metadata.wikilinks`.
Invalid frontmatter degrades gracefully to non-canonical (R4).

**Promotion pipeline** (ADR 0003, human-gated):
- `POST /api/kb/promotion/suggest` → LLM extracts candidates. Writes nothing.
- `POST /api/kb/promotion/candidates` → validates a draft. Writes nothing.
- `POST /api/kb/promotion/promote` → writes markdown + dispatches ingest.
  Returns 202.

Operator CLI equivalent: `kb:promote {path} --project=…`. Claude skills
stop at `suggest` / `candidates`. Only humans (git commit → GH action →
ingest) and operators (`kb:promote`) commit canonical storage.

**Delete** — `kb:delete` / `DELETE /api/kb/documents` /
`kb:ingest-folder --prune-orphans` / scheduled `kb:prune-deleted` all fan in
to `DocumentDeleter`. Default is soft delete (`KB_SOFT_DELETE_ENABLED=true`,
retention `KB_SOFT_DELETE_RETENTION_DAYS=30`). Hard delete **cascades the
graph**: `kb_nodes` owned by the doc are removed (`source_doc_id` match,
fallback `node_uid = slug`); the composite FK on `kb_edges` cascades both
directions. Every hard delete writes a `kb_canonical_audit` row.

**Scheduler** (`bootstrap/app.php`):

| Time  | Command                    |
| ----- | -------------------------- |
| 03:10 | `kb:prune-embedding-cache` |
| 03:20 | `chat-log:prune`           |
| 03:30 | `kb:prune-deleted`         |
| 03:40 | `kb:rebuild-graph`         |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lopadova/AskMyDocs](https://github.com/lopadova/AskMyDocs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->
