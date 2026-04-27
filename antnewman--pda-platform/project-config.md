---
trigger: always_on
description: The PDA Platform is a Python monorepo providing AI-powered project delivery assurance for UK government. It exposes 116 MCP tools across 17 modules via a unified server, deployable locally or via Render.
---

# Claude Code Instructions — PDA Platform

## Project Overview
The PDA Platform is a Python monorepo providing AI-powered project delivery assurance for UK government. It exposes 116 MCP tools across 17 modules via a unified server, deployable locally or via Render.

## Repo Structure
```
packages/
  pda-platform/          Meta-package (pip install pda-platform)
  pm-data-tools/         Core library: parsers, validators, AssuranceStore (SQLite)
  pm-mcp-servers/        17 MCP modules, 116 tools
  agent-task-planning/   AI reliability: confidence extraction, outlier mining
docs/                    Practitioner guides and technical references
.github/workflows/       publish.yml — PyPI publish on version tag
```

## Key Commands
- **Run unified server locally:** `pda-platform-server`
- **Run remote SSE server:** `pda-platform-remote`
- **Run MCP tests:** `cd packages/pm-mcp-servers && python -m pytest`
- **Run API tests:** `cd packages/pm-api && python -m pytest`
- **Run data-tools tests:** `cd packages/pm-data-tools && python -m pytest`

## Working with Claude
1. For ALL git commits, use author `antnewman <antjsnewman@outlook.com>` — NEVER include a "Co-Authored-By: Claude" line
2. Branch workflow: always work on `dev` branch. Never push directly to `main` — that is for PRs/production only
3. Prefer editing existing files over creating new ones
4. After any change to `server.py` or registries, run the MCP test suite to confirm tool counts

## Package Versions — Keep in Sync

All four packages are versioned together. Current version: **1.2.0**

| Package | PyPI | pyproject.toml location |
|---|---|---|
| `pda-platform` | [pypi.org/project/pda-platform](https://pypi.org/project/pda-platform/) | `packages/pda-platform/pyproject.toml` |
| `pm-mcp-servers` | [pypi.org/project/pm-mcp-servers](https://pypi.org/project/pm-mcp-servers/) | `packages/pm-mcp-servers/pyproject.toml` |
| `pm-data-tools` | [pypi.org/project/pm-data-tools](https://pypi.org/project/pm-data-tools/) | `packages/pm-data-tools/pyproject.toml` |
| `agent-task-planning` | [pypi.org/project/agent-task-planning](https://pypi.org/project/agent-task-planning/) | `packages/agent-task-planning/pyproject.toml` |

### When to bump versions

Bump ALL four packages together whenever:
- A new MCP tool or module is added
- A breaking change is made to any public API or store schema
- A significant new feature lands in pm-data-tools or agent-task-planning

Use semantic versioning:
- **Patch** (1.0.x) — bug fixes, doc updates, no new tools
- **Minor** (1.x.0) — new tools added, backward-compatible
- **Major** (x.0.0) — breaking changes to store schema, tool signatures, or APIs

### How to release to PyPI

1. Bump all four `version = "x.y.z"` fields in their respective `pyproject.toml` files
2. Update inter-package dependency pins to `>=` the **previous published version** (NOT the new version — see warning below)
3. Commit: `chore: bump versions to x.y.z`
4. Merge dev → main via PR
5. Tag on main: `git tag vx.y.z && git push antnewman vx.y.z`
6. The GitHub Actions workflow (`.github/workflows/publish.yml`) builds and publishes all four packages automatically
7. After publish completes, update the inter-package pins to `>=x.y.z` in a follow-up commit if needed

### What NOT to do
- Do not publish manually with `twine upload` — use the workflow
- Do not bump only one package — all four must stay in sync
- Do not tag on `dev` — always tag on `main` after the PR is merged
- **Do not pin inter-package deps to the NEW version before it is published** — Render builds from source but pip resolves deps from PyPI. If `pm-data-tools` requires `agent-task-planning>=1.1.0` but 1.1.0 is not yet on PyPI, Render will fail. Keep pins at the last confirmed published version until the new version is live on PyPI.

## MCP Tool Count — Keep in Sync

The tool count (currently **116**) appears in multiple places. When adding new tools, update ALL of these:

| File | What to update |
|---|---|
| `packages/pm-mcp-servers/src/pm_mcp_servers/pda_platform/server.py` | Docstring total, module count in `logger.info` |
| `packages/pm-mcp-servers/tests/test_pda_platform.py` | `test_total_tool_count`, `test_tool_ordering`, expected tool sets |
| `README.md` | MCP modules table, overview paragraph, key features bullet |
| `docs/architecture-overview.md` | Module table |
| `docs/connection-guide.md` | Tool count references |
| `docs/getting-started.md` | Tool count references |
| `packages/pm-mcp-servers/README.md` | Module table, total tool count |
| `packages/pda-platform/README.md` | Module table, total tool count |
| `packages/pda-platform/pyproject.toml` | `description` field tool count |
| `packages/pm-mcp-servers/pyproject.toml` | `description` field tool count and module count |
| `docs/hackathon/CHALLENGE.md` | Numbers section |
| `docs/hackathon/QUICKSTART.md` | What's Available section |
| `server.json` | `description` field (root level) |
| `CLAUDE.md` (this file) | Package versions table, MCP tool count |

## Store Schema — Keep in Sync

When adding a new module that needs persistence:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/antnewman) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-11 -->
