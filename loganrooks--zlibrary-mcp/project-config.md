---
trigger: always_on
description: The working contract for any coding agent in this repo — Codex, Claude Code, or
---

# AGENTS.md

The working contract for any coding agent in this repo — Codex, Claude Code, or
otherwise. It is the single canonical copy; `CLAUDE.md` imports this file rather than
restating it, so there is nothing here to drift out of sync with a second version.

## What this is

A Model Context Protocol server that finds books, downloads them, and turns them into
files a RAG pipeline can use. Node.js/TypeScript MCP layer (`src/`) over a Python bridge
(`lib/`) that does the Z-Library API work and document processing.

`VISION.md` states the seven invariants and the non-goals. Read it before proposing
anything architectural — it is short, and it is the document that says what this project
refuses to become.

## Setup and commands

Prerequisites: Node 22+, Python 3.10+, and [uv](https://docs.astral.sh/uv/). The Python
side uses **uv, never pip**.

```bash
npm install          # Node deps
uv sync              # Python deps into .venv/ (or: bash setup-uv.sh)
npm run build        # tsc + validates that every Python bridge file exists
```

| What | Command |
|---|---|
| Node tests | `node --experimental-vm-modules node_modules/jest/bin/jest.js` |
| Python tests | `uv run pytest -m "not slow and not integration and not performance" --benchmark-disable -rs` |
| Full Python suite | `uv run pytest -rs` |
| Lint | `npx eslint src/` and `npx prettier --check src/` |
| Health / drift check | `npm run doctor` |
| Release-record audit | `npm run audit:release` |

**`npm test` does not run pytest.** It runs Jest with coverage. The two suites are
separate and a green `npm test` says nothing about the Python side — which is where most
of the actual logic lives.

**Five real-PDF tests do not run in CI** (issue #85): the repo exceeded its Git LFS
budget, so CI checks out with `lfs: false` and guards in `__tests__/python/conftest.py`
skip those tests with a stated reason. Locally, `git lfs pull` hydrates them. Do **not**
run `git lfs install` — it writes hooks that conflict with the repo's Husky-managed
`core.hooksPath`.

## Invariants that break silently

These are the ones where a plausible-looking change passes every test and breaks
production.

1. **stdout is the protocol channel.** Under the stdio transport stdout carries JSON-RPC
   and nothing else. Use `logger` from `src/lib/logger.ts`, which writes to stderr. A
   single `console.log` in `src/` disconnects strict MCP clients.
   `__tests__/stdio-purity.test.js` fails the build if one appears — do not weaken it.
2. **Files, not payloads.** Tools return paths to artifacts on disk, never raw document
   text through the context window. This is invariant 1 in `VISION.md`, not a style
   preference.
3. **Python scripts live in `lib/`, not `dist/`.** The build does not copy them. Runtime
   resolution walks `dist/lib/` → `dist/` → project root → `lib/`. Use the helpers in
   `src/lib/paths.ts` (`getPythonScriptPath`, `getPythonLibDirectory`) rather than
   hand-rolling a relative path. Rationale: [ADR-004](docs/adr/ADR-004-Python-Bridge-Path-Resolution.md).
4. **Unit tests mock every third-party call**, so the suite stays green after real
   integrations break. Upstream drift is caught by `.github/workflows/upstream-check.yml`
   and `npm run doctor` — not by the tests. Green tests are not evidence the world hasn't
   moved (invariant 6).
5. **The root `pyproject.toml` sets `package = false`.** Without it, the repo's `src/`
   directory flips setuptools to src-layout discovery while the published npm package
   (which ships no `src/`) falls back to flat-layout and refuses to build, breaking
   `uv sync` for every npm-installed user. Do not remove it.

## Architecture

- `src/index.ts` — MCP server entry point and tool definitions
- `src/lib/zlibrary-api.ts` — bridge to Python via PythonShell
- `src/lib/venv-manager.ts` — uv environment lifecycle
- `lib/python_bridge.py` — core Z-Library operations
- `lib/rag_processing.py` — EPUB/TXT/PDF processing
- `lib/sources/` — source adapters
- `zlibrary/` — vendored fork of sertraline/zlibrary with custom download logic

Two behavioural decisions that are easy to reverse by accident:

- **Downloads start from a search result, and there is no direct-from-ID path.**
  `download_book_to_file` takes the `bookDetails` object a search returned.
  `lib/python_bridge.py::download_book` then routes multi-source results through
  `SourceRouter` and Z-Library results through `EAPIClient.download_file`, whose link
  comes from the JSON EAPI. **Neither path scrapes a book detail page** — the
  detail-page scraping described in [ADR-002](docs/adr/ADR-002-Download-Workflow-Redesign.md)
  predates the Phase 7 EAPI migration and no longer exists in the code. The part of
  ADR-002 that still holds is the search-result-first workflow, not the mechanism.
- **`get_book_by_id` is deprecated** as unreliable ([ADR-003](docs/adr/ADR-003-Book-ID-Lookup-Deprecation.md)).
  Find books with `search_books`.

The full tool list lives in `README.md` and is checked against the code by
`scripts/validate-readme-tools.sh` in CI — add a tool, update the README in the same
commit or `docs-check` fails. Retry, timeout, and circuit-breaker tuning is in
[docs/RETRY_CONFIGURATION.md](docs/RETRY_CONFIGURATION.md); the implementations are

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [loganrooks/zlibrary-mcp](https://github.com/loganrooks/zlibrary-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-30 -->
