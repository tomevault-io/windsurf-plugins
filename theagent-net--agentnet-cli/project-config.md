---
trigger: always_on
description: CLI tool that detects AI coding agents on your system and connects them to the [Agent-net](https://agentnet.market) marketplace via MCP.
---

# agentnet-cli

CLI tool that detects AI coding agents on your system and connects them to the [Agent-net](https://agentnet.market) marketplace via MCP.

## Tech Stack

- **Language:** Python 3.10+ (ruff linting, 100-char line length)
- **Package manager:** uv
- **CLI framework:** Typer + Rich
- **HTTP client:** httpx
- **Testing:** pytest (511 tests), pytest-cov
- **CI:** GitHub Actions (lint + test matrix on 3.10/3.11/3.12/3.13)
- **Publish:** PyPI via trusted publisher (tag `v*`)

## Repository Structure

See [ARCHITECTURE.md](ARCHITECTURE.md). Layers:

- `cli/` — Typer entry (`cli/main.py`), core commands, marketplace JSON commands
- `connectors/` — per-agent wiring + `templates/` for file injection
- `marketplace/` — platform client, external catalogs, skill discovery
- `tools/` — MCP stdio server, Hermes plugin, and `skillfire/` (the every-prompt skill-fire pipeline)
- `infra/` — config, paths, manifest
- `src/agentnet_cli/integrations/` — Claude and OpenClaw native plugin trees (bundled in wheel)

## Key Commands

```bash
uv sync --group dev              # Install deps
uv run pytest -v                 # Run tests
uv run pytest --cov -q           # With coverage
uv run ruff check .              # Lint
uv run agentnet --help           # Run locally
```

## Key Patterns

- **Agent Connector:** Abstract `AgentConnector` base with `detect()`, `connect()`, `disconnect()`. Add new agents by subclassing and registering in `registry.py`.
- **Manifest rollback:** `manifest.py` tracks every file injected during `connect` so `disconnect` can cleanly remove them.
- **Config persistence:** `~/.agentnet/config.json` stores platform credentials (0600 permissions). Agent custom paths stored separately.
- **MCP server:** `agentnet mcp-serve` (hidden command) starts stdio JSON-RPC server. Agents launch this as a subprocess.
- **Marketplace commands:** All output JSON to stdout. Errors output `{"error": "..."}` with exit code 1.
- **Claude Code Plugin:** `agentnet connect claude` delegates to `claude plugin marketplace add` + `claude plugin install` instead of writing files directly. The plugin at `claude-plugin/` is installed via Claude Code's native marketplace system.
- **Hermes Plugin:** `agentnet connect hermes` copies the plugin to `~/.hermes/plugins/agentnet/` and skills to `~/.hermes/skills/agentnet/`, using Hermes's native plugin system.
- **OpenClaw Plugin:** `agentnet connect openclaw` delegates to `openclaw plugins install` + `openclaw plugins uninstall` instead of writing files directly. The plugin at `openclaw-plugin/` is a native OpenClaw plugin with `openclaw.plugin.json` manifest, publishable to ClawHub.
- **Plugin hint:** The CLI emits a `<claude-code-hint>` tag on stderr when `CLAUDECODE=1` is set, prompting Claude Code users to install the plugin.
- **Auto-update on the hook flow:** `maybe_auto_update()` (rate-limited 24h via manifest `last_update_check_at`, gated by `AGENTNET_AUTO_UPDATE`, PyPI version check, background installer upgrade + stale-integration refresh) runs on every non-hook `agentnet` command via the Typer `@app.callback()`. The **hook** commands (`{skill,cursor,hermes}-hook`) are excluded there (`_HOOK_COMMANDS`) so `--pre`/`--peek`/`--post` never block a tool call; the update runs **once per turn from the detached `--fetch` worker** (`skillfire.run_fetch`), off the agent's critical path.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TheAgent-net/agentnet-cli](https://github.com/TheAgent-net/agentnet-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
