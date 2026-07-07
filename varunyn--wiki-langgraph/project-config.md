---
trigger: always_on
description: Agent guide for working in `wiki-langgraph`.
---

# AGENTS.md
Agent guide for working in `wiki-langgraph`.

This repository is a Python 3.12+ LangGraph pipeline that ingests raw markdown, compiles an Obsidian-style wiki, optionally runs LLM authoring and semantic-link suggestions, refreshes QMD indexes when enabled, and lints the vault.

## Scope and priority
- This file is for coding agents operating inside this repository.
- Prefer repo-specific guidance here over generic habits.
- There are currently **no** `.cursor/rules/`, `.cursorrules`, or `.github/copilot-instructions.md` files in this repo.
- Do not invent tooling or conventions that are not present in the codebase.

## Environment and tooling
- Python: **3.12+**
- Dependency manager: **uv**
- Test runner: **pytest**
- Build backend: **uv_build**
- Main package: `src/wiki_langgraph/`
- CLI entrypoint: `wiki-langgraph`

## Core commands
Run commands from the repository root.

### Install / sync
```bash
uv sync
```

### Run the pipeline
```bash
uv run wiki-langgraph run
uv run wiki-langgraph run -v
```
Use `-v` to print the graph step log.

### Lint the raw/wiki pair
```bash
uv run wiki-langgraph lint
uv run wiki-langgraph lint --strict
```
`--strict` makes warnings fail.

### Auto-fix unresolved wikilinks in raw notes
```bash
uv run wiki-langgraph lint --fix --dry-run
uv run wiki-langgraph lint --fix
uv run wiki-langgraph lint --fix --fix-mode strip
uv run wiki-langgraph lint --fix --fix-mode rewrite
```

### Version and tests
```bash
uv run wiki-langgraph version
uv run pytest
uv run pytest tests/test_lint.py
uv run pytest tests/test_lint.py::test_lint_unresolved_wikilink
uv run pytest -k orphan
```

## Project layout
- `src/wiki_langgraph/config.py` — environment-backed settings
- `src/wiki_langgraph/graph.py` — LangGraph topology and `run_once`
- `src/wiki_langgraph/nodes.py` — ingest / compile / index / lint nodes
- `src/wiki_langgraph/linking.py` — wikilinks, backlinks, semantic sections, `Index.md`
- `src/wiki_langgraph/lint.py` — vault lint and raw-note fixups
- `src/wiki_langgraph/manifest.py` — incremental hash and semantic cache manifest
- `src/wiki_langgraph/llm_author.py` — raw-to-wiki LLM authoring
- `src/wiki_langgraph/linking_qmd.py` — QMD semantic search and QMD refresh
- `src/wiki_langgraph/deep_agent.py` — Deep Agents factory / filesystem backend
- `tests/` — pytest coverage for graph, linking, lint, manifest, config, and deep agent
- `README.md` — user-facing usage/config docs
- `docs/ARCHITECTURE.md` — graph/module behavior and provenance rules

## Working style
### General principles
- Follow existing patterns in neighboring files before introducing new structure.
- Keep diffs small and local when fixing behavior.
- Treat compile as **incremental wiki authoring**, not just file copying.
- Preserve the distinction between authored wikilinks and semantic suggestions.
- Prioritize correctness and repository consistency over clever abstractions.

### Imports and formatting
- Use standard library imports first, then third-party, then local package imports.
- Prefer explicit imports from `wiki_langgraph.*` modules.
- Avoid unused imports.
- Match the repo’s existing Python style: readable functions, concise docstrings, moderate line lengths.
- Prefer small helpers when they clarify logic, but do not split code mechanically.

### Types and naming
- Use type hints throughout.
- Prefer concrete types already used here: `dict[str, object]`, `list[str]`, `Path`, TypedDict state.
- Avoid `Any` unless genuinely required at an external boundary.
- Do not suppress type checking unless absolutely unavoidable.
- Use descriptive snake_case names tied to repo concepts such as `raw_uris`, `semantic_edges`, `index_md_written`, and `content_overrides`.

### Error handling
- Do not swallow errors silently unless the current pattern explicitly does so for best-effort behavior.
- Preserve existing soft-failure behavior: invalid/missing manifest → empty structure; missing QMD binary or query timeout → graceful degradation; lint issues → `LintReport` or `last_error`.
- When failure should be visible to users, surface it via return values, warnings, or CLI exit code behavior consistent with current code.
- Empty `except` blocks are not acceptable.

## Repository-specific behavior to preserve
### Index and ingest
- `Index.md` must be regenerated on every compile run.
- Do not change behavior toward “write only if missing”.
- `raw_uris` are posix paths relative to the raw root.
- Ingest is recursive and skips `.gitkeep` plus anything under a `.git` directory.

### Linking and provenance
- Backlinks come from **authored** `[[wikilinks]]` only.
- Semantic suggestions are a separate provenance layer.
- Do not merge semantic edges into the explicit backlink graph.
- Managed markdown blocks use comment markers and should be replaced safely on recompile.

### Lint
Current lint behavior includes unresolved wikilinks, orphan notes with no outgoing `[[wikilinks]]`, stale compiled wiki output, `Index.md` drift, and read errors. Generated/index notes are exempt from orphan-note warnings.

### Manifest and incremental behavior
- The manifest stores raw content hashes for incremental LLM compile.
- When semantic links are enabled, it also stores semantic cache entries.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [varunyn/wiki-langGraph](https://github.com/varunyn/wiki-langGraph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
