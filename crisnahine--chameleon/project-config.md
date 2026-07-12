---
trigger: always_on
description: This file provides guidance to Claude Code when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with code in this repository.

## What this repo is

`chameleon` — a Claude Code plugin that auto-derives codebase conventions and injects archetype-aware guidance per-edit (conformance), and answers codebase-comprehension queries like search_codebase, describe_codebase, get_callees, and get_blast_radius (comprehension). Supports TypeScript/JavaScript, Ruby, and Python as first-class languages — framework-agnostic by default (it learns each repo's own conventions, so any framework works), with deeper framework-aware guidance where conventions are strong: Rails for Ruby, Django, DRF, Flask, and FastAPI for Python, and Next.js and NestJS for TypeScript/JavaScript.

See [docs/architecture.md](./docs/architecture.md) for the full design.

## Project structure

```
chameleon/
├── .claude-plugin/    marketplace.json (points at ./plugin as the plugin source)
├── plugin/            the installable plugin surface (this is what a marketplace install copies)
│   ├── .claude-plugin/  plugin.json (plugin manifest)
│   ├── .mcp.json        MCP server registration (${CLAUDE_PLUGIN_ROOT}-relative)
│   ├── hooks/           session-start, preflight-and-advise, posttool-recorder,
│   │                    posttool-verify, callout-detector, stop-backstop
│   │                    (+ _resolve-python.sh, run-hook.cmd, hooks.json)
│   ├── skills/          using-chameleon (auto) + 14 user-invocable slash commands
│   ├── agents/          code-scout, pattern-reviewer, web-researcher
│   ├── mcp/             chameleon-mcp Python server (FastMCP, stdio transport)
│   ├── scripts/         ts_dump.mjs, prism_dump.rb, libcst_dump.py, merge driver, setup.sh
│   └── bin/             chameleon-statusline.sh (status line, <100ms budget)
├── scripts/           dev-only tooling: bump-version.sh, prune-plugin-cache.sh, ...
├── tests/             unit/ + journey/ + effectiveness/ harnesses + qa_*.py real-repo batteries
└── docs/              architecture.md (design) + install.md + language-support-matrix.md + parity-progress.md + qa-team.md
```

The user-invocable commands: `init`, `refresh`, `status`, `teach`, `auto-idiom`, `trust`, `disable`, `pause-15m`, `doctor`, `journey`, `pr-review`, `receiving-code-review`, `explain`, `deep-work` (all invoked as `/chameleon-*`).

## Conventions

- **Language**: all code, comments, docs, error messages, and commit messages MUST be in English.
- **Versioning**: `bump-version.sh <new-version>` keeps six manifest files in sync (see `.version-bump.json`).
- **Locks**: `plugin/mcp/package-lock.json` and `plugin/mcp/uv.lock` are committed.
- **Atomic transactions**: profile writes use `.chameleon/.tmp/<txn-id>/COMMITTED` sentinel + flock-serialized rename.
- **Production-ref derivation**: when `.chameleon/config.json` has `production_ref` (auto-locked at init/refresh for origin-backed repos, or set explicitly), bootstrap/refresh analyze a materialized worktree of that ref instead of the checkout; refresh noop/staleness is tip-SHA-keyed. Refresh (manual + auto) first runs a default-ON, non-interactive `git fetch origin <branch>` so the tip it resolves is the latest production, not the user's last fetch (kill: `CHAMELEON_FETCH_PRODUCTION_REF=0` / config `auto_refresh.fetch_production_ref=false`; auto-suppressed under CI; fails open). Local-only repos (most test fixtures) never auto-lock — they keep working-tree derivation. An explicit `"production_ref": null` is a durable opt-out (migration never re-locks over it). See `plugin/mcp/chameleon_mcp/production_ref.py` and docs/architecture.md "Production-ref derivation".

## Working on this codebase

### Lint and format

Python is linted with ruff (line-length 100, config in `plugin/mcp/pyproject.toml`; `E402` and `E501` are intentionally ignored — see the comments there):

```bash
plugin/mcp/.venv/bin/ruff check .          # lint
plugin/mcp/.venv/bin/ruff format .         # format
```

### Run the journey harness

```bash
plugin/mcp/.venv/bin/python -m tests.journey.runner               # full run (~$33, ~65 min)
plugin/mcp/.venv/bin/python -m tests.journey.runner --list        # list acts
plugin/mcp/.venv/bin/python -m tests.journey.runner --dry-run     # preflight only, no Claude spawn
plugin/mcp/.venv/bin/python -m tests.journey.runner --max-budget-usd 20
```

The journey harness drives real `claude -p` subprocesses against committed seed fixtures. Run before each release. All state is isolated to a per-run dir under `tests/journey/results/`; the developer's own `~/.local/share/chameleon/` is never touched.

### Run unit tests for chameleon

```bash
PYTHONPATH=. plugin/mcp/.venv/bin/python -m pytest tests/unit/ -v
```

These verify chameleon's hook functions (posttool_verify, etc.) with mocked dependencies.

### Run unit tests for the harness library

```bash
PYTHONPATH=. plugin/mcp/.venv/bin/python -m pytest tests/journey/harness/tests/ -v
```

These verify the harness library itself (context, checkpoints, expect, fixtures setup). They do NOT test chameleon; that's the journey runner's job.

### QA batteries against a real profiled repo


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [crisnahine/chameleon](https://github.com/crisnahine/chameleon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
