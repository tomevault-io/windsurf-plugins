---
trigger: always_on
description: Pi extension wrapping the [graphify](https://github.com/safishamsi/graphify) Python CLI for knowledge graph generation and exploration.
---

# Pi Graphify

Pi extension wrapping the [graphify](https://github.com/safishamsi/graphify) Python CLI for knowledge graph generation and exploration.

## Architecture

- `src/config.ts` — Raw/resolved config loader
- `src/lib/runner.ts` — Graphify CLI execution logic (no Pi imports). All functions accept an injected `exec` callback for testability. Includes `getInstalledVersion`, `getLatestVersion`, `installSkillFromCLI`, `updateUpstreamVersion` for upgrade/skill-install.
- `src/lib/runner.test.ts` — Unit tests for runner (40 passing)
- `src/tools/` — LLM-callable tools (thin wrappers around runner)
  - All tools in `graphify-tools.ts`: build, query, path, explain, add, update, watch, cluster, upgrade
  - Integration tests in `graphify.integration.test.ts` (10 passing)
- `src/commands/` — `/graphify` slash command with autocomplete for all subcommands
- Skill is installed by the graphify CLI to `~/.pi/agent/skills/graphify/SKILL.md` via `graphify install --platform pi`. Auto-reinstalled on upgrade via `graphify_upgrade`.

## Tools (11)

| Tool                       | Description                                                                                                                    |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| `graphify_build`           | Full pipeline: detect → extract → cluster → visualize                                                                          |
| `graphify_query`           | BFS/DFS graph traversal                                                                                                        |
| `graphify_path`            | Shortest path between two concepts                                                                                             |
| `graphify_explain`         | Plain-language node explanation                                                                                                |
| `graphify_add`             | Fetch URL and add to corpus                                                                                                    |
| `graphify_update`          | Incremental update (changed files only)                                                                                        |
| `graphify_watch`           | Watch directory for changes                                                                                                    |
| `graphify_cluster`         | Re-run clustering on existing graph                                                                                            |
| `graphify_extract`         | Headless LLM extraction for CI (defaults to deepseek; also supports claude, kimi, openai, gemini, ollama, bedrock, claude-cli) |
| `graphify_export_callflow` | Generate self-contained Mermaid architecture/call-flow HTML                                                                    |
| `graphify_upgrade`         | Check for and install graphifyy CLI updates via uv                                                                             |

## Commands

Subcommands: build, query, path, explain, add, update, watch, cluster, hook, extract, uninstall, upgrade

Build flags: `--mode deep`, `--no-viz`, `--obsidian`, `--svg`, `--graphml`, `--neo4j`, `--callflow`, `--update`, `--cluster-only`
Extract flags: `--backend <claude|kimi|openai|gemini|ollama|bedrock|claude-cli|deepseek>`, `--max-workers N`, `--max-concurrency N`, `--token-budget N`, `--api-timeout N`, `--resolution N`, `--exclude-hubs P`, `--exclude <pattern>`

## Runner Functions (not yet exposed as tools)

These are implemented in `runner.ts` but not yet wired as dedicated tools or command handlers:

- `pushNeo4j` — push graph to Neo4j instance
- `saveResult` — save Q&A feedback loop to graph memory
- `cloneRepo` — clone a GitHub repo for graphing
- `mergeGraphs` — merge multiple graph.json files
- `generateTree` — collapsible tree HTML visualization

## CLI Coverage Gaps

The following graphify CLI commands are **not** exposed as tools through the extension (available via the graphify skill or direct CLI):

- `graphify tree` — D3 collapsible tree HTML
- `graphify clone` — clone GitHub repos
- `graphify merge-graphs` — cross-repo graph merging
- `graphify check-update` — cron-safe update check
- `graphify save-result` — Q&A feedback loop
- `graphify global` — cross-project global graph
- IDE integrations (`claude install`, `cursor install`, etc.) — use `graphify pi install` instead

The `graphify extract` and `graphify export callflow-html` commands are now exposed as dedicated tools (`graphify_extract`, `graphify_export_callflow`) and `/graphify` subcommands.

## Deviation Notes

- No `enabled` toggle: extension is always active; graphify CLI is the gate (auto-installs on first use).
- Uses `pi.exec()` for all shell commands — no `child_process`.
- The runner module (`src/lib/runner.ts`) contains pure domain logic with no Pi imports for testability.

## Dependencies

- `@gaodes/pi-utils-ui` — TUI components (ToolCallHeader, ToolBody, ToolFooter)
- `graphifyy` (Python) — installed at runtime via pip/uv

## Config


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gaodes/pi-graphify](https://github.com/gaodes/pi-graphify) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
