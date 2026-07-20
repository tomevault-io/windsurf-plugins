---
trigger: always_on
description: Instructions for AI agents working in the king-context repository. This file is the canonical, tool-neutral rule set. `CLAUDE.md` imports it and adds Claude-Code-specific guidance on top. Do not duplicate content between the two.
---

# AGENTS.md

Instructions for AI agents working in the king-context repository. This file is the canonical, tool-neutral rule set. `CLAUDE.md` imports it and adds Claude-Code-specific guidance on top. Do not duplicate content between the two.

## What this project is

King Context is a CLI-first, local-first retrieval layer for AI agents. It indexes any corpus (vendor docs, web research, architecture decisions) into a flat-file store under `.king-context/` and serves it through three console scripts: `kctx` (search and read), `king-scrape` (documentation scraping pipeline), and `king-research` (web research pipeline). It ships as the npm installer `@king-context/cli` plus a Python package with four sub-packages: `king_context`, `context_cli`, `llm_providers`, `scraper_providers`.

The MCP server (`king-context-server`) still ships but is the legacy, secondary surface per ADR-0001. Do not frame the project, its docs, or new features around MCP.

## Non-negotiable rules

- All code, comments, identifiers, documentation, commits, and PR descriptions are written in English.
- Never use em-dashes or en-dashes anywhere: code, comments, docs, READMEs, commits, PR bodies, issues. Use periods, commas, colons, or parentheses instead.
- Never mix languages in the same file. Internal architecture drafts in `.docs/` may be Portuguese; everything else is English. `README-pt-br.md` is the one intentionally Portuguese published file.
- ADRs are created and updated only through the `kctx adr` CLI: draft the markdown elsewhere, import with `kctx adr new --from-file`, then run `kctx adr index` and `kctx adr validate`. Never hand-write or hand-edit files inside `.king-context/adr/`.

## The two retrieval stacks (critical orientation)

The repo contains two parallel retrieval stacks that do not share data:

1. **Canonical: the CLI flat-file stack.** `src/context_cli/` reads and writes JSON under `.king-context/docs/`, `.king-context/research/`, and `.king-context/decisions/`. Search is reverse-index metadata scoring. No SQLite, no embeddings.
2. **Legacy: the MCP SQLite stack.** `src/king_context/server.py` (FastMCP) plus `db.py` (SQLite, FTS5, embedding rerank) with `docs.db` and `data/` at the repo root. Kept as an integration layer per ADR-0001.

Consequences:

- Never couple new code (CLI, web UI, pipelines) to `king_context.db` or `docs.db` (ADR-0005).
- Never use `seed_data` or `python -m king_context.seed_data` to index a corpus; that feeds the legacy MCP database. Index with `kctx index` instead.
- The 4-layer cascade (cache, metadata, FTS5, embeddings) described in older docs applies only to the legacy stack. See `docs/architecture/overview.md`.

## Commands

```bash
# Install for development
pip install -e ".[all,dev]"

# Tests
pytest -q
pytest -k <test_name>

# Search and read indexed corpora (canonical CLI)
kctx list
kctx search "query" --source all|docs|research
kctx read <doc> <section-path>
kctx grep <pattern>
kctx topics <doc>
kctx index <file.json>
kctx adr <list|search|read|timeline|new|supersede|link|index|status|validate>
kctx llm-doctor
kctx ui

# Scrape and index documentation
king-scrape <url>                      # discover, filter, fetch, chunk, enrich, export
king-scrape <url> --provider crawl4ai  # pick scraper provider (default: firecrawl)
king-scrape <url> --step <stage> --stop-after <stage>
king-scrape audit <name>               # read-only corpus drift check
king-scrape update <name>              # incremental refresh reusing unchanged content

# Web research into an indexed corpus
king-research "<topic>" --basic|--medium|--high|--extrahigh
```

Full command reference: `docs/CLI_GUIDE.md`.

## Architecture map

- `src/context_cli/` is the `kctx` CLI, the canonical product surface.
- `src/king_context/scraper/` and `src/king_context/research/` are the two pipelines; `src/king_context/web/` is the local UI behind `kctx ui`.
- `src/scraper_providers/` and `src/llm_providers/` are pluggable provider abstractions (entry-point registries).
- `src/king_context/server.py`, `db.py`, `seed_data.py` are the legacy MCP stack.
- `installer/` is the zero-dependency npm package that bootstraps `.king-context/` into host projects.
- All paths resolve from `PROJECT_ROOT = Path.cwd()`; run commands from the project root.

Deep dives: `docs/architecture/`. Storage details: `docs/reference/storage-layout.md`. Corpus JSON format: `docs/reference/corpus-schema.md`. Environment variables: `docs/reference/env-vars.md`.

## Code conventions

- New code uses PEP 604 unions (`X | None`) and `from __future__ import annotations`; legacy core modules keep `typing.Optional` for local consistency.
- `pathlib` over `os.path`; dataclasses for config and value objects; stdlib `json`.
- CLI modules follow the `_build_parser()` plus `_cmd_<name>(args)` pattern with `set_defaults(func=...)`.
- Errors print `error: <message>` to stderr. The scraper CLI uses exit codes 1 (general), 2 (ValueError), 3 (provider unavailable); `kctx` exits 1 on failure.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [deandevz/king-context](https://github.com/deandevz/king-context) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
