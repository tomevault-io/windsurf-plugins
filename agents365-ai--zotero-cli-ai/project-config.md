---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

`zotero-cli` (binary: `zot`; PyPI package: `zotero-cli-ai`) — a Zotero CLI for any AI agent. It combines **direct local SQLite reads** with **Zotero Web API writes**, and exposes the same surface via an MCP server. The CLI follows an agent-native contract documented in `docs/agent-interface.md` (stable JSON envelope, typed exit codes, `zot schema` introspection, `--dry-run`, `--idempotency-key`, NDJSON streaming).

## Common commands

Uses `uv` as the package manager (`uv.lock` is authoritative). CI runs on Python 3.10–3.13.

```bash
# Install dev environment (mirrors CI)
uv sync --group dev --extra mcp

# Lint / format / type-check / test — same order as .github/workflows/ci.yml
uv run ruff check src/ tests/
uv run ruff format --check src/ tests/   # use `ruff format` (without --check) to auto-fix
uv run mypy src/zotero_cli_cc/
uv run pytest tests/ -v

# Run a single test / file / node
uv run pytest tests/test_reader.py -v
uv run pytest tests/test_reader.py::test_name -v
uv run pytest -k "search and not pdf" -v

# Run the CLI from source
uv run zot search "foo"
uv run zot schema                         # emit full command tree (agent introspection)

# Build / publish artifacts (hatchling backend)
uv build
```

Note on PyPI publish gating: the `publish.yml` workflow gates release on lint+mypy only, **not** full pytest (see commit `6267da8`). Keep that intentional when editing CI.

## Architecture

### Read/write split (the central design constraint)

- **Reads** go through `core/reader.py` — opens `zotero.sqlite` directly (read-only) from the auto-detected Zotero data directory. No network, no API key, works offline, works while Zotero.app is running.
- **Writes** go through `core/writer.py` — uses `pyzotero` against the Zotero Web API so Zotero's sync engine sees the change. Never write to `zotero.sqlite` directly; doing so corrupts Zotero's sync state.

This split is load-bearing for the project's value proposition. Preserve it when adding commands: a new mutating command belongs on the Web API side, not SQLite.

### CLI shape (`cli.py` + `commands/`)

- `cli.py` is the Click root group. It registers every subcommand from `commands/*.py` and classifies them into safety tiers (read / mutating / destructive) which drive `--help` grouping and the agent schema. When adding a command, register it in `cli.py` AND add it to the appropriate tier set, or help/schema will misreport its risk.
- Each `commands/<name>.py` is a self-contained Click command/group. They orchestrate `core/*` modules and pass results through `formatter.py`.
- `formatter.py` implements the dual-output contract: Rich tables when stdout is a TTY, JSON envelope when piped (auto-detected) or when `--json` / `ZOT_FORMAT=json` is set. Every command's output must flow through the formatter to keep the envelope stable — don't `click.echo` structured data directly.
- `exit_codes.py` enumerates the typed exit codes (1 runtime, 2 auth, 3 validation, 4 not-found, 5 network, 6 conflict). Errors must map to one of these via `emit_error(...)`; the agent contract in `docs/agent-interface.md` promises stability. Avoid the legacy `print_error(...); return` pattern — it silently exits 0 and breaks agent error-handling.
- `schema.py` (command) reflects the entire Click tree into the JSON schema that agents consume. If you add options/arguments, they appear here automatically — but only if you use standard Click constructs.

### Core subsystems (`core/`)

- `reader.py` — SQLite read layer (search, list, read, collections, tags, attachments metadata). Zotero 10 holds `locking_mode=EXCLUSIVE` + WAL on zotero.sqlite while running, so reads are live when Zotero is closed and fall back to a consistent snapshot copy (DB + WAL replay, per-instance) while it runs. The FTS5 full-text index lives in a separate `fulltext.sqlite` (not exclusively locked; plain read-only open).
- `writer.py` — pyzotero-backed writes (add, update, delete, note, tag mutations, attachment upload).
- `pdf_extractor.py` + `pdf_cache.py` — pluggable extraction backends (`pdfium` default — permissive BSD/Apache; `pymupdf` opt-in via the `[pymupdf]` extra for annotations/highlights + better markdown; `mineru` opt-in with auto-fallback to `pdfium`) with on-disk cache keyed per-extractor; feeds `zot pdf`, `summarize`, and the on-the-fly passage extraction in `rank.py`. pymupdf is lazy-imported so the base install ships no AGPL code.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Agents365-ai/zotero-cli-ai](https://github.com/Agents365-ai/zotero-cli-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
