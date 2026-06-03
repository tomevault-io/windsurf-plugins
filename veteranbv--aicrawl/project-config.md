---
trigger: always_on
description: Project guidance for Codex and other coding agents working in `aicrawl`.
---

# AGENTS.md

Project guidance for Codex and other coding agents working in `aicrawl`.

## Purpose

`aicrawl` is a local-first OpenClaw-style archive for personal AI conversation history. The v0.1 product is a high-integrity importer and search/export CLI for official Claude and ChatGPT data exports.

Optimize for durable, private, verifiable preservation of user data. Do not optimize for convenience at the cost of correctness, privacy, maintainability, or provider terms.

## Public-repo hygiene

This repository is intended to be public.

Do not commit private planning artifacts, Codex prompts, raw exports, local source data, archive databases, generated snapshots, logs, caches, credentials, or machine-local configuration. In particular, keep these out of git:

- `docs/IMPLEMENTATION_PLAN.md`
- `docs/CODEX_GOAL_PROMPT.md`
- Claude or ChatGPT export ZIPs
- extracted export files such as `conversations.json` or `chat.html`
- Claude Code or other local transcript JSONL files
- SQLite databases and WAL/SHM files
- local `.aicrawl/`, `.crawlbar/`, `.crabbox/`, export, import, raw-data, backup, and snapshot directories

If a private file is already tracked, `.gitignore` is not enough. Remove it from the index with `git rm --cached <path>` and verify with `git status` before committing.

Synthetic fixtures are allowed only when they are small, clearly fake or redacted, and intentionally named as fixtures, for example `testdata/redacted/*.fixture.json`. Never commit real user conversation text, account identifiers, email addresses, attachment contents, API tokens, session tokens, or provider export files.

## Source of truth

Use the public repo files as the durable source of truth: code, tests, `README.md`, committed public docs, and this `AGENTS.md`.

Private local planning files may exist while bootstrapping the project. They are context, not public project artifacts. Do not require them for builds or tests, and do not commit them.

When docs and code disagree, inspect the current code and update the smallest necessary surface. Do not silently invent missing architecture. If a documented requirement is wrong or stale, explain the evidence and make the smallest correction needed.

## OpenClaw / crawlkit boundary

Use `github.com/openclaw/crawlkit` for reusable local archive mechanics where it naturally fits: config paths, SQLite helpers, snapshots, mirrors, sync state, output helpers, progress/status/control payloads, terminal browsing, and safe local cache reads.

Keep provider-specific behavior in `aicrawl`, not in `crawlkit`: Claude and ChatGPT schemas, export parsing, source detection, privacy filtering, attachments, account/workspace quirks, user-facing import behavior, and app-specific database schema.

If a change to `crawlkit` seems necessary, keep the API additive and small. Only move logic upstream when it is clearly reusable across multiple crawl apps. Otherwise keep the downstream app change narrow.

## Non-negotiable product constraints

- Use official/local sources only for v0.1: Claude export ZIP/JSON, ChatGPT export ZIP/JSON, and later documented local transcript files.
- Do not implement session-token scraping, private product API calls, browser-console snippets, headless browser export automation, or hidden network sync.
- Preserve raw source payloads alongside parsed normalized columns so parser bugs can be repaired later.
- Re-imports must be idempotent. Duplicate conversations/messages from overlapping exports are bugs.
- Treat ChatGPT `mapping` as a graph. Store every node and explicit edges; never collapse branches/regenerations into a lossy linear transcript.
- Parse Claude exports defensively. Tolerate unknown fields and optional missing metadata, but fail clearly on missing required stable IDs unless a documented fallback is implemented.
- Keep provider-specific parsing downstream under `internal/ingest/*`.
- Protect private data by default: private file permissions, redacted logs, safe ZIP handling, read-only SQL, synthetic/redacted fixtures only, and no imported content sent to cloud services by default.

## Engineering rules

Build clean, robust, maintainable, production-ready code. Keep changes small, coherent, and scoped to the task.

Do:

- Prefer the simplest solution that fully meets the need.
- Use the Go standard library and `crawlkit` before adding dependencies.
- Preserve existing conventions before introducing new ones.
- Add abstractions only after the concrete need is proven.
- Name constants for limits and defaults; explain why values exist.
- Write tests for correctness, edge cases, idempotency, malformed input, and security-sensitive behavior.
- Use temp dirs and temp SQLite files in tests. Never touch live user stores.
- Review your diff like a senior engineer before calling the task done.

Do not:

- Add fake implementations, placeholders, stubs, TODO-driven behavior, hidden technical debt, or unexplained hardcoded values.
- Add dependencies, layers, config files, generated code, background services, or architectural patterns unless clearly justified.
- Broaden scope into embeddings, TUI, watch daemons, enterprise compliance APIs, cloud sync, or mirror/publish flows unless the task explicitly asks for that phase.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [veteranbv/aicrawl](https://github.com/veteranbv/aicrawl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
