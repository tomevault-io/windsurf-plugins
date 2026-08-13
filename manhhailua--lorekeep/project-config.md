---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Lorekeep compiles a team's raw markdown docs into a **temporal knowledge graph** (`facts.jsonl`) and exposes it to coding agents (Claude Code, Cursor, Codex, opencode) over MCP, with per-namespace permission. The MCP surface has 7 composable tools plus passive context resources. Agent writes are confidence-gated journals merged on resolve. Knowledge is processed once at compile time, not re-RAG'd per query.

## Commands

Python 3.11+, managed with **uv**. The CLI is `lorekeep` (entry: `src/lorekeep/cli.py`, Typer).

```bash
uv run pytest                                # full suite (~1,200 tests)
uv run pytest tests/test_perm.py -q          # one file
uv run pytest tests/test_perm.py::test_name  # one test
uv run pytest -k perm                        # by name match
uv run pytest --cov=lorekeep                 # coverage (pytest-cov)
uv build                                     # sdist + wheel (hatchling)
uv run lorekeep <command>                    # run the CLI in dev mode
```

### CLI commands (all run via `uv run lorekeep …`)

| Command | Purpose |
|---|---|
| `init` | Bootstrap data home (config + schema + raw/graph dirs) |
| `compile [--foreground]` | `raw/*.md` → `graph/facts.jsonl` + `manifest.json` + `wiki/` — defaults to **background** in interactive mode (delegates to daemon via `.compile-requested` sentinel), `--foreground` for synchronous |
| `wiki` | Regenerate `wiki/` from `facts.jsonl` (Obsidian-compatible markdown) |
| `serve [--transport stdio\|http]` | Run the MCP server (8 tools + passive resources) |
| `agent watch` | Start the daemon (polls 60s: auto-import sessions, auto-compile raw/, auto-resolve pending/) |
| `agent heal` | Run self-heal standalone (remove dangling edges, dedupe, flag issues) |
| `agent service install/uninstall/status` | Install/uninstall/status the daemon as an OS service (launchd/systemd) |
| `schema upgrade` | Upgrade stock schema to latest version (backs up previous, `--dry-run`/`--force` for custom schemas) |
| `mcp add --agent claude\|cursor\|codex\|opencode --ns NS` | Write agent MCP config |
| `config show` | Print config.yaml |
| `config set <key> <value>` | Set nested config value (dot notation) |
| `import --from claude\|cursor\|codex\|opencode` | Import agent sessions into `raw/` |
| `doctor` | Validate full install: graph loads (no dangling edges), schema valid, MCP tools respond, provider reachable |
| `backup [--init <remote-url>] [--force]` | Sync durable inputs plus graph/wiki snapshot to a private backup Git repo; `--force` auto-resolves snapshot conflicts (remote wins) |
| `version` | Print version |
| `update [--check]` | Upgrade lorekeep to latest from PyPI (detects uv/pipx/pip); `--check` previews without upgrading |

**Offline / no-LLM mode:** tests inject `FakeProvider` via monkeypatch (`patch_make_provider` / `patch_make_import_provider` fixtures in `conftest.py`). **All CLI/compile/import tests use this** — no API key or real model required.

## Architecture: two strictly separated phases

```
COMPILE (offline, curator):  raw/<ns>/*.md → ingest → extract(LLM) → resolve → writer → facts.jsonl → wiki/
SERVE   (runtime, per device): facts.jsonl → GraphStore → ScopedGraph(ns) → MCP → agent
```

`compile` mutates `facts.jsonl` and auto-generates `wiki/`; `resolve` regenerates wiki only on actual merge (gated on `merge_count > 0`); `serve` reads `facts.jsonl` and lazily reloads on mtime change. Core write tools (`propose_change`, `review_note`) append to `pending/` journals; resolve merges accepted entries into the graph. Wiki regen is **best-effort** — never blocks `compile` or `resolve`. Wiki builds into a temp dir then `os.rename` swaps into place (atomic — never partially populated).

### Compile pipeline (`src/lorekeep/compile/`, orchestrated by `pipeline.py`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [manhhailua/lorekeep](https://github.com/manhhailua/lorekeep) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
