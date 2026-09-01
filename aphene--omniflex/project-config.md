---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

`getflex` (Python 3.12+, MIT) — a knowledge engine that compiles data sources into per-cell SQLite databases (`~/.flex/cells/{name}.db`) with embeddings, knowledge-graph enrichments, and self-describing views. Each cell exposes a single read-only MCP tool (`flex_search`); the agent discovers schema at query time and writes SQL.

There are no tests in this repo (the `[tool.pytest.ini_options]` block in `pyproject.toml` references markers like `unit`, `pipeline`, `vec_ops`, etc., but no `tests/` directory exists). Don't claim a change passes tests — there's nothing to run.

## Commands

```bash
# Install for development (editable)
pip install -e .

# Entry points (registered in pyproject.toml [project.scripts])
flex init                       # base install
flex init --module claude-code  # install a module
flex search "query" --cell claude_code
flex sync [--cell NAME] [--full]
flex status [--json] [--all] [--problems]
flex health [--json]
flex module install <path> [--editable] [--force]
flex module list / remove <name>
flex relay [--stop] [--status]

# Long-running processes
python -m flex.daemon                    # local capture + background loops
python -m flex.daemon --no-refresh --no-background   # worker only
python -m flex.refresh                   # one-shot refresh tick (systemd timer target)
python -m flex.serve                     # MCP stdio
python -m flex.serve --http --port 7134  # MCP HTTP
python -m flex.serve --cell A --cell B   # multi-cell
```

The Rust sidecar (`flex-embed/`, `cargo build --release`) is a standalone embedding binary; it reads NULL-embedding rows from a cell DB and writes them back. It is NOT a Python extension — it ships as a separate compiled binary and the Python ONNX path in `flex/onnx/` is the in-process embedder.

## Architecture

The `flex/__init__.py` docstring is authoritative: domains are `compile/` (source → chunks), `manage/` (offline graph intelligence → `_enrich_*` columns), `retrieve/` (query execution: vec_ops, presets, direct SQL), and `core.py` (cell loading + SQL plumbing). Read it before refactoring across these boundaries.

### Cell database invariants

Every cell DB has the same shape — code throughout the repo assumes this:

- `_raw_chunks`, `_raw_sources` — base tables with an `embedding` BLOB column
- `_edges_source` — chunk→source bridge; **invariant** (enforced in `core.validate_cell`): every chunk has exactly one source edge
- `_edges_*` / `_types_*` / `_enrich_*` — auto-discovered by `views.regenerate_views` and joined into chunk/source views
- `_views` — curated views table; entries here override auto-generated views of the same name
- `_meta` — key/value table; `vec:*` keys configure VectorCache modulation
- `_ops` — append-only mutation log; mutators self-log via `core.log_op`

When adding columns or tables, prefer the `_enrich_*` / `_types_*` / `_edges_*` naming — `regenerate_views` LEFT JOINs them into views automatically. Anything else needs a curated entry in `_views`.

`flex/views.py` is re-exported as `flex.core.regenerate_views` for back-compat — both import paths are live.

### Module system (`flex/modules/`)

Each module is a folder with an `install.py` exposing `MODULE_SUMMARY`, optional `register_args(parser)`, and a `run(args, console)` entrypoint. `flex/modules/specs.py` discovers them in two locations:

1. Packaged: `flex/modules/*/install.py` (built-in: `claude_code`, `markdown`, `soma`)
2. External: `~/.flex/modules/*/install.py` and any path in `FLEX_MODULE_PATH` (colon-separated)

A module may also expose:
- `MODULE` dict — declares `cell_type`, `views_from`, `presets_from`, `enrichment_stubs_from` for asset reuse
- `plugin.py` with `register_query_tokens()` / `register_query_materializers()` — extends the SQL/vec_ops surface
- `stock/views/*.sql` and `stock/presets/*.sql` — shipped assets installed into the cell

Cell-type aliases live in `_LEGACY_CELL_TYPE_ALIASES` in `specs.py` (`claude-code` → `claude_code`, `obsidian` → `markdown`). Use `normalize_cell_type()` rather than string-comparing cell types directly.

The `claude_code` module is the canonical example. Its `__init__.py` exports `ENRICHMENT_STUBS` (DDL for the enrichment tables that views LEFT JOIN against) — other coding-agent modules reuse this substrate via `enrichment_stubs_from: claude_code` in their MODULE spec. Stubs let views resolve before the first enrichment pass runs.

### Hook + plugin extensibility

`flex/registry.py` is the cell registry (`~/.flex/registry.db`) AND a generic in-process hook bus:
- `register_hook(name, fn)` / `get_hook(name)` — modules wire optional behavior (e.g., `daemon_tick`, `register_cli_commands`, `register_extra_commands`)
- `load_plugins()` reads `~/.flex/plugins.txt` and imports each line — names are validated against `^flex(\.[a-zA-Z0-9_]+)+$` (security: prevents arbitrary code execution via a poisoned plugins file)

The CLI in `flex/cli.py` calls `load_plugins()` and `flex.sdk` import-for-side-effects before `parser.parse_args()`, so any plugin-registered subcommands appear in `--help`.

### Engine facade


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Aphene/omniflex](https://github.com/Aphene/omniflex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-01 -->
