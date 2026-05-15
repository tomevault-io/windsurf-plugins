---
trigger: always_on
description: kwin-mcp is an MCP (Model Context Protocol) server for Linux desktop GUI automation on KDE Plasma 6 Wayland.
---

# Claude Code Working Guidelines

## Project Overview

kwin-mcp is an MCP (Model Context Protocol) server for Linux desktop GUI automation on KDE Plasma 6 Wayland.
**Purpose**: Enable AI agents (Claude Code and other MCP clients) to launch, interact with, and observe any Wayland application — either in a fully isolated virtual KWin session for reproducible GUI testing, or by connecting to a live KDE Plasma desktop (real session or container) for collaborative desktop automation workflows.

## Toolchain

- **Package Manager**: `uv` (Astral). Always use uv instead of pip/poetry.
- **Lint + Format**: `ruff` (Astral). Config in `pyproject.toml` under `[tool.ruff]`.
- **Type Check**: `ty` (Astral). Config in `pyproject.toml` under `[tool.ty]`.
- **Build**: `uv build` (uv_build backend)

### Common Commands

```bash
uv sync                   # Install/sync dependencies
uv add <pkg>              # Add dependency
uv add --dev <pkg>        # Add dev dependency
uv run ruff check .       # Lint
uv run ruff format .      # Format
uv run ty check           # Type check
uv run python -m kwin_mcp # Run server
```

## Code Style

- Python 3.12+
- ruff rules: E, F, W, I, N, UP, B, A, SIM, TCH, RUF
- Line length: 100
- Quotes: double quote
- Type hints required
- All documents (.md files), code comments, and docstrings must be written in English

## Architecture

See ROADMAP.md. Key modules:
- `core.py`: AutomationEngine — MCP-independent automation logic (session, input, screenshot, a11y)
- `server.py`: Thin MCP wrappers delegating to AutomationEngine
- `cli.py`: Interactive REPL + pipe mode via `cmd.Cmd` (`kwin-mcp-cli` entry point)
- `session.py`: Virtual sessions (dbus-run-session + kwin_wayland --virtual) and live session attachment (session_connect to real desktop or container)
- `screenshot.py`: KWin ScreenShot2 D-Bus (screenshots)
- `accessibility.py`: AT-SPI2 (widget tree)
- `input.py`: KWin EIS D-Bus + libei (input injection)

## Pre-work Checklist

1. Read `ROADMAP.md` to understand current progress
2. Start from the first incomplete item in the next milestone
3. After code changes, run `uv run ruff check .` + `uv run ruff format .` + `uv run ty check`
4. Update ROADMAP.md checklist when a milestone item is completed
5. **Verify with CLI, not MCP tools**: After modifying any kwin-mcp code, verify via the CLI (`uv run python -m kwin_mcp.cli`), NOT the MCP server tools. The MCP server process is already running with the old code loaded in memory — calling MCP tools after editing source files will NOT reflect the changes. Use the CLI to launch a session and test the modified functionality. Use `keep_screenshots=true` in `session_start` to preserve screenshot files after `session_stop` (they are deleted by default). Files in `/tmp/kwin-mcp-screenshots-*` must be cleaned up manually when this option is used.
6. **Invoke docs-seo agent after relevant file changes**: After modifying any of the trigger files below, invoke the `@docs-seo` agent to evaluate whether documentation is stale. A no-op conclusion is acceptable if nothing needs updating.

### docs-seo Auto-Trigger Files

| Changed File | Trigger Label | docs-seo Action |
|---|---|---|
| `src/kwin_mcp/server.py` | tool-registration, code-general | Check tool count, README tool tables, CONTRIBUTING structure, **integrations/*/skills/*/SKILL.md tool list freshness** |
| `src/kwin_mcp/session.py` | session-api | Check README arch diagram, CONTRIBUTING session docs |
| `src/kwin_mcp/core.py` | engine-api, code-general | Check README arch description, CONTRIBUTING structure |
| `src/kwin_mcp/*.py` (any) | code-general | Check concrete numbers, CONTRIBUTING file listing, **integrations/*/skills/*/SKILL.md "30 capabilities" reference** |
| `pyproject.toml` | package-metadata | Sync keywords with `.claude/positioning.yml`; check CLAUDE.md keyword tiers; **run `python3 scripts/sync_plugin_version.py` to propagate version to integrations manifests** |
| `CHANGELOG.md` | changelog-update | Sync docs-seo.md positioning; add new search intents |
| `README.md` | readme-update | Sync docs-seo.md positioning; update CLAUDE.md keyword tiers |
| `ROADMAP.md` | roadmap-update | Add new long-tail keywords; update CONTRIBUTING.md if milestone adds contributor workflow |
| `.claude/positioning.yml` | manifest-update | Full sync of docs-seo.md + CLAUDE.md SEO sections; **propagate `plugin_manifest_required_keywords` to integrations manifests** |
| `.claude-plugin/marketplace.json` | plugin-manifest | Verify version matches pyproject; verify keywords ⊇ `plugin_manifest_required_keywords` |
| `integrations/claude-code/.claude-plugin/plugin.json` | plugin-manifest | Verify version + keywords (same rule as above) |
| `integrations/opencode/plugin/package.json` | plugin-manifest | Verify version + keywords (same rule as above) |
| `integrations/claude-code/skills/*/SKILL.md` | plugin-skill | This is the source-of-truth SKILL.md. After edit, run `python3 scripts/sync_plugin_version.py` to mirror it into `integrations/opencode/plugin/skill/<name>/SKILL.md` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [isac322/kwin-mcp](https://github.com/isac322/kwin-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
