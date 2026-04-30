---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

CLI-Anything-Web is a Claude Code plugin that generates production-grade Python CLIs for any web application by capturing live HTTP traffic. Point at a URL → capture API traffic via playwright-cli → analyze endpoints → generate a complete CLI with auth, REPL mode, `--json` output, and tests.

## Plugin Structure

```
cli-anything-web-plugin/          # The plugin itself
├── .claude-plugin/plugin.json    # Plugin manifest
├── HARNESS.md                    # Core methodology SOP (read this first)
├── commands/                     # Slash commands (/cli-anything-web, /record, /refine, etc.)
├── skills/                       # 4-phase skill system (capture → methodology → testing → standards)
├── scripts/                      # parse-trace.py, mitmproxy-capture.py, analyze-traffic.py (v1.3.0), site-fingerprint.js, capture-checkpoint.py, phase-state.py, repl_skin.py
│                                 # Pipeline automation: scaffold-cli.py, validate-checklist.py, generate-test-docs.py, smoke-test.py
└── templates/                    # .tpl files used by scaffold-cli.py (exceptions, client, auth, CLI entry, setup, RPC, etc.)
```

Generated CLIs live in their own directories (e.g., `futbin/agent-harness/`) with namespace packages under `cli_web/`.

## Pipeline Phases

The skill sequence is strictly ordered: `capture → methodology → testing → standards → DONE`

| Phase | Skill | What it does |
|-------|-------|-------------|
| 1 | capture | Site assessment + browser traffic recording (or public API shortcut) |
| 2 | methodology | Analyze traffic, design CLI architecture, implement code |
| 3 | testing | Write unit + E2E + subprocess tests |
| 4 | standards | Implementation review (3 agents), 75-check checklist, publish, smoke test, generate skill |

### Site Profiles

Not all sites need the same pipeline path. Identify the profile early:
- **Auth + CRUD** (full pipeline): Google apps, Monday.com, Notion, Jira
- **Auth + read-only**: Dashboards, analytics viewers
- **No-auth + CRUD**: Sites with optional API key auth (Dev.to)
- **No-auth + read-only**: Hacker News, Wikipedia, npm — skip auth.py, skip write tests

## Tool Hierarchy (Strict)

1. **PRIMARY (traffic capture)**: `npx @playwright/cli@latest` — handles traffic recording during Phase 1 via tracing.
1b. **OPTIONAL (traffic capture)**: `mitmproxy-capture.py` — proxy-based capture with no body truncation, real-time filtering, enhanced metadata. Activated with `--mitmproxy` flag on `/cli-anything-web`. Requires `pip install mitmproxy` (Python 3.12+).
2. **PRIMARY (auth login)**: Python `sync_playwright()` — handles browser-based auth login in generated CLIs
3. **FALLBACK**: `mcp__chrome-devtools__*` — only if playwright unavailable
4. **NEVER**: `mcp__claude-in-chrome__*` — cannot capture request bodies

## Generated CLI Structure

Every generated CLI follows this exact layout under `<app>/agent-harness/`:

```
cli_web/                    # Namespace package (NO __init__.py)
└── <app>/                  # Sub-package (HAS __init__.py)
    ├── <app>_cli.py        # Click entry point + REPL default
    ├── core/               # exceptions.py, client.py, auth.py, session.py, models.py
    │   └── rpc/            # Optional: types.py, encoder.py, decoder.py (non-REST protocols)
    ├── commands/           # One file per resource group
    ├── utils/              # helpers.py, repl_skin.py, output.py, config.py
    └── tests/              # test_core.py (unit), test_e2e.py (E2E + subprocess)
```

Key files alongside: `setup.py`, `<APP>.md` (API map), `README.md`, `TEST.md`

## Running Tests

```bash
cd <app>/agent-harness
pip install -e .
python -m pytest cli_web/<app>/tests/ -v -s

# Single test
python -m pytest cli_web/<app>/tests/test_core.py::test_player_search -v -s
```

Set `CLI_WEB_FORCE_INSTALLED=1` for subprocess tests to find the installed CLI binary.

## Validating the Plugin

```bash
bash cli-anything-web-plugin/verify-plugin.sh
```

## Critical Conventions

- **Naming**: CLI command = `cli-web-<app>`, Python namespace = `cli_web.<app>`, config dir = `~/.config/cli-web-<app>/`
- **Namespace packages**: `cli_web/` has NO `__init__.py`; sub-packages DO have `__init__.py`
- **Typed exceptions**: Every CLI has `core/exceptions.py` with `AppError → AuthError, RateLimitError, NetworkError, ServerError, NotFoundError, RPCError`. No generic `RuntimeError`.
- **Auth**: Credentials in `auth.json` with `chmod 600`, never hardcoded. Env var `CLI_WEB_<APP>_AUTH_JSON` for CI/CD.
- **Auth cookie priority**: For Google apps, `.google.com` cookies MUST take priority over regional duplicates (`.google.co.il`, `.google.de`, etc.). Naive `{c["name"]: c["value"]}` flattening is BROKEN for international users. See `auth-strategies.md` "Cookie domain priority".

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ItamarZand88/CLI-Anything-WEB](https://github.com/ItamarZand88/CLI-Anything-WEB) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
