---
trigger: always_on
description: Canonical instructions for AI coding agents working in Row-Bot. Keep this file
---

# AGENTS.md

Canonical instructions for AI coding agents working in Row-Bot. Keep this file
concise, concrete, and aligned with `scripts/run_test_matrix.py`.

## Project Identity

Row-Bot is a local-first desktop AI assistant with provider-aware agent
runtimes, tools, workflows, durable memory/knowledge graph data, MCP, plugins,
skills, channels, voice, Developer Studio, Designer Studio, and platform
installers.

Priorities, in order:

1. Protect local user data, secrets, and local-first defaults.
2. Avoid surprise network calls, provider calls, or real channel messages.
3. Preserve approval gates and graceful recovery for destructive actions.
4. Add deterministic tests for changed behavior.
5. Keep Windows and macOS first-class; keep Linux browser/server mode healthy.

## Ground Rules

- Do not add Row-Bot first-party telemetry, analytics, or hidden phone-home
  behavior. A third-party dependency with telemetry may be used only when its
  behavior is reviewed, documented, disclosed before installation or first
  execution, and accepted by the user. Never describe third-party telemetry
  as Row-Bot telemetry, and never allow dependency telemetry to include
  Row-Bot prompts, files, memories, secrets, screenshots, tool arguments, or
  channel content. The reviewed, opt-in Cua Driver integration is the only
  currently approved exception; future dependencies require a separate review.
- Do not commit secrets, API keys, provider tokens, private local paths, or real
  user data in code, tests, docs, fixtures, snapshots, or logs.
- Do not make default tests depend on live providers, live MCP servers, real
  messaging channels, real network availability, or a specific local Ollama
  model. Mark those tests `live_provider` or `e2e`.
- Do not add substantive tests to `tests/test_suite.py`,
  `tests/integration_tests.py`, or `tests/test_memory_e2e.py`; they are retired
  compatibility shims.
- Do not edit `requirements.txt` by hand. It is generated from `uv.lock`.
- Do not add runtime implementation code to root wrappers such as `app.py` or
  `launcher.py`; application code belongs under `src/row_bot/`.
- Do not recursively delete ignored/generated directories such as `.tmp/`,
  `.testtmp/`, `dist/`, or `installer/build/` unless explicitly asked.
- Keep changes scoped. Avoid unrelated refactors, reformatting, and metadata
  churn.

## Repository Map

- `src/row_bot/providers/`: provider config, catalogs, readiness, selection,
  transports, and model routing.
- `src/row_bot/tools/`: agent tools, registry, media tools, shell/browser, MCP,
  Developer and Designer tools.
- `src/row_bot/tasks.py`: workflows, scheduling, approvals, delivery defaults,
  pipeline state, and run history.
- `src/row_bot/knowledge_graph.py`, `memory*.py`, `dream_cycle.py`,
  `wiki_vault.py`, `documents.py`: memory, recall, extraction, Dream Cycle,
  wiki vault, and document knowledge.
- `src/row_bot/channels/`: channel adapters, registry, auth, approvals, media,
  and tunnel helpers.
- `src/row_bot/mcp_client/`: MCP config, runtime, safety, requirements, and
  marketplace/client integration.
- `src/row_bot/developer/`: Developer Studio sandbox, runtime, import gate, Git
  helpers, inspector, and state.
- `src/row_bot/designer/`: Designer Studio state, export, templates, rendering,
  previews, thumbnails, and AI content.
- `src/row_bot/plugins/`, `skills_hub/`, `skills.py`: plugin and skill systems.
- `src/row_bot/ui/`: NiceGUI UI surfaces and reusable UI helpers.
- `tests/contracts/`: fake adapter and interface contracts.
- `tests/subsystem/`: deterministic subsystem end-to-end tests with fakes.
- `tests/integration/`: deterministic cross-subsystem tests.
- `tests/e2e/`: opt-in live provider or real-service tests.
- `tests/fixtures/` and `tests/helpers/`: fakes, snapshots, source-test map,
  legacy inventory, subprocess helpers, and coverage inventory.
- `scripts/run_test_matrix.py`: local and CI test matrix source of truth.
- `installer/` and `.github/workflows/`: packaging, CI, release, installer
  verification, live e2e, lockfile, update manifest, and notarization flows.

## Before Editing

1. Read the relevant source and nearby tests first.
2. Identify the subsystem owner and test lane before changing behavior.
3. Prefer existing helpers, fixtures, UI primitives, and local patterns.
4. Use structured parsers/APIs for structured data when reasonable.
5. Add or update focused tests for behavior changes.
6. If a change crosses subsystem ownership, update `tests/helpers/source_test_map.py`
   and any affected inventory tests.
7. Treat sandbox/import gates, shell execution, MCP safety, updater/installer
   flows, signing, and release workflows as security sensitive.

## Dependencies

`pyproject.toml` is canonical. `uv.lock` is the locked resolution.
`requirements.txt` is a generated installer export.

For dependency changes:

```powershell
uv lock
python scripts/export_locked_requirements.py
uv sync --locked --all-extras --group test
uv run python scripts/verify_runtime_dependencies.py all
uv run python scripts/run_test_matrix.py pr
```

For dependency verification only:

```powershell
uv lock --check
python scripts/export_locked_requirements.py --check
uv sync --locked --all-extras --group test

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [siddsachar/row-bot](https://github.com/siddsachar/row-bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
