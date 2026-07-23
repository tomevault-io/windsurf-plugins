---
trigger: always_on
description: This repository is the source package and installer for Microsoft Fabric agent profiles. It is not the day-to-day Fabric project workspace. Install the CLI with `uv tool install fabric-vibecoding-settings` (or `pip install fabric-vibecoding-settings`). Then run `fabric-vibecoding-agents install --profile claude --target /path/to/repo` and open Claude Code from that target repo root. Use `fabric-vibe <group> <cmd>` inside the target for daily helpers.
---

# Fabric Agent Pack — Claude Contributor Guidance

This repository is the source package and installer for Microsoft Fabric agent profiles. It is not the day-to-day Fabric project workspace. Install the CLI with `uv tool install fabric-vibecoding-settings` (or `pip install fabric-vibecoding-settings`). Then run `fabric-vibecoding-agents install --profile claude --target /path/to/repo` and open Claude Code from that target repo root. Use `fabric-vibe <group> <cmd>` inside the target for daily helpers.

## Architecture at a glance

The repo has three top-level packages:

- **`cli/`** — everything installed on the user's laptop: the wheel installer, profile entrypoints + 4 subagents (Claude + Codex), setup scripts, and the target-side tools shipped to `tool/` in the target repo. Tools in `cli/tools/` are invoked via Bash, not MCP.
- **`server/`** — FastMCP HTTP server (Docker). Serves graph knowledge tools plus fabric helpers that run without ms-fabric-cli. Start it with `docker compose up` from the repo root.

Source-package invariants (layout + profile guidance) are enforced by pytest modules `tests/test_install_package.py` and `tests/test_agent_guidance.py`, backed by importable logic in `tests/_validation/`.

See [`docs/architecture.md`](docs/architecture.md) for diagrams.

## MCP server capabilities (`server/`)

The `fabric-server` MCP container exposes:

| Tool | Description |
|---|---|
| `graph_get_entry`, `graph_get_node`, `graph_get_linked`, `graph_search`, `graph_list_kinds` | Knowledge graph read tools |
| `graph_create_node`, `graph_update_node`, `graph_delete_node`, `graph_add_edge`, `graph_remove_edge` | Knowledge graph write tools (atomic rebuild) |
| `pipeline_lineage_check` | Upload notebooks as `{rel_path: content}`, validates staging-path consistency |
| `data_mock_generate` | Generate deterministic synthetic CSV test data |
| `semantic_model_list`, `semantic_model_show` | Inspect Fabric semantic model measures and relationships via `sempy.fabric` |

The server has **no filesystem access** to the user's project. `pipeline_lineage_check` accepts uploaded file contents; `data_mock_generate` requires a `target_dir` mounted into the container.

## CLI / Bash tool capabilities (`cli/tools/` → installed as `tool/`)

Tools shipped to the target repo's `tool/` dir and invoked via Bash. Fabric helper commands that talk to Fabric require `ms-fabric-cli` (`uv tool install ms-fabric-cli`) and SPN credentials in `.env`:

| Path | Description |
|---|---|
| `tool/notebook/build.py` | Build `.Notebook` bundles from `workspace/<topic>/<name>.py` |
| `tool/notebook/deploy.py` | Deploy, run, monitor, fetch notebooks |
| `tool/pipeline/manage.py` | Create, run, list, test Data Factory pipelines |
| `tool/lakehouse/list-tables.py` | Inspect lakehouse tables and schemas |
| `tool/workspace/{init,switch,transfer}.py` | Manage `workspaces.json`, switch workspace, transfer items |
| `tool/lint/` (`python -m tool.lint`) | Deterministic lints: SEC-01 hardcoded secrets, DE-09 Faker seed. Pure Python, no fab. |
| `tool/precommit/pre-commit-check.{sh,ps1}` | Aggregate pre-commit check: runs lints locally. Pipeline lineage check is via the `pipeline_lineage_check` MCP tool. |
| `tool/setup/setup.{ps1,sh}` | One-time target bootstrap: install fab, prompt for SPN creds, populate workspaces.json |

## Source package layout

| Path | Purpose |
|---|---|
| `cli/src/fabric_skills_settings/` | Pip-installable wheel package (`fabric-vibecoding-settings`). Typer CLIs in `cli.py` (`fabric-vibecoding-agents` installer) and `runtime_cli.py` (`fabric-vibe` target-side proxy). Subcommands in `commands/`, shared logic in `core/`. Profiles in `_profiles/`, setup in `_setup/`, tools in `_tools/` (bundled at build time). |
| `cli/profiles/claude/` | Claude-native install assets: `CLAUDE.md`, `.claude/agents/`, `settings.local.json`. |
| `cli/profiles/codex/` | Codex-native install assets: `AGENTS.md`, `.codex/agents/`, `config.toml`. |
| `cli/profiles/shared/` | Shared scaffold (`data/sandbox/`, `workspace/`, `.env.example`, `.gitignore.fragment`). `.mcp.json` is NOT shipped — the target bootstrap writes it with a concrete MCP URL. |
| `cli/setup/` | `setup.{ps1,sh}` — target bootstrap scripts (shipped to `tool/setup/`). |
| `cli/tools/` | Target-side tools (shipped to `tool/`): `notebook/`, `pipeline/`, `lakehouse/`, `workspace/`, `lint/`, `precommit/`. |
| `server/app.py` | FastMCP app — registers all server-side tools, wires auth + CORS middleware. |
| `server/auth/` | Auth layer: `repository.py` (SQLite API-key store via `FABRIC_MCP_API_KEYS_DB`, plus inline `FABRIC_MCP_API_KEYS` env var), `tokens.py` (JWT + JtiStore), `middleware.py` (`FabricAuthMiddleware` + `install_auth_middleware`). |
| `server/tools/` | MCP tool wrappers: `graph/`, `validate/`, `data/`, `semantic_model/`. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [scardoso-lu/fabric-skills-settings](https://github.com/scardoso-lu/fabric-skills-settings) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
