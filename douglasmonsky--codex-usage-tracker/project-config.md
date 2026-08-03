---
trigger: always_on
description: This repo builds a local Codex plugin and dashboard that track aggregate token usage from Codex session logs.
---

# Codex Usage Tracker Instructions

## Project Purpose

This repo builds a local Codex plugin and dashboard that track aggregate token usage from Codex session logs.

## Tech Stack

- Python 3.10+
- SQLite via the Python standard library
- MCP Python SDK for Codex tool exposure
- Pytest for tests

## Repo Layout

- `src/codex_usage_tracker/` - parser, SQLite store, reports, dashboard, CLI, and MCP server.
- `src/codex_usage_tracker/context.py` - on-demand raw-context reader for one selected usage record.
- `src/codex_usage_tracker/reports.py` - shared application/report services used by CLI and MCP wrappers.
- `src/codex_usage_tracker/api_payloads.py` - shared stable JSON payload builders for CLI and MCP surfaces.
- `src/codex_usage_tracker/schema.py` - single source of truth for persisted usage-event columns.
- `src/codex_usage_tracker/threads.py` - thread attachment inference used by dashboard payload generation.
- `src/codex_usage_tracker/pricing_config.py`, `pricing_openai.py`, `pricing_estimates.py`, and `costing.py` - pricing config, source parsing, estimate policy, and cost calculations behind the `pricing.py` facade.
- `src/codex_usage_tracker/allowance.py` - Codex credit-rate and optional local allowance-window helpers.
- `src/codex_usage_tracker/plugin_installer.py` - package-owned local Codex plugin installer.
- `src/codex_usage_tracker/plugin_data/` - plugin assets, dashboard template/assets, local dashboard guide, screenshots, and skill files bundled into wheels.
- `skills/codex-usage-tracker/` and `src/codex_usage_tracker/plugin_data/skills/codex-usage-tracker/` - operational Codex skill for tracker setup, summaries, dashboard generation, and MCP tools.
- `skills/codex-usage-api/` and `src/codex_usage_tracker/plugin_data/skills/codex-usage-api/` - companion Codex skill for conversational analysis using the stable JSON API/MCP tools.
- `src/codex_usage_tracker/server.py` - localhost dashboard server with live aggregate refresh and lazy context endpoints.
- `~/.codex-usage-tracker/pricing.json` - optional local-only pricing config, never committed.
- `~/.codex-usage-tracker/allowance.json` - optional local-only copied allowance state, never committed.
- `.codex-plugin/plugin.json` - Codex plugin manifest.
- `.mcp.json` - MCP server configuration for Codex.
- `scripts/install_local_plugin.py` - compatibility wrapper around `codex-usage-tracker install-plugin`.
- `scripts/check_release.py` - release-readiness checks for docs, versions, packaging, wheel contents, and tracked secret patterns.
- `.github/workflows/ci.yml` - GitHub Actions test and package build workflow.
- `.github/workflows/pricing-compat.yml` - scheduled/manual non-blocking live pricing parser compatibility check.
- `docs/` - install, dashboard, CLI, pricing/credits, MCP, privacy, architecture, development, JSON-schema docs, and screenshots built from synthetic aggregate fixture data.
- `tests/` - synthetic fixtures and unit tests.

## Setup

```bash
python3 -m venv .venv
. .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install ".[dev]" twine
codex-usage-tracker install-plugin --python .venv/bin/python
```

## Branch And PR Workflow

This project is now a published PyPI package with user-facing docs, JSON/MCP contracts, a release workflow, and privacy guarantees. Treat `main` as always releasable.

### MCP-First Pivot Execution

Work in the MCP-first pivot must follow
`docs/roadmap/mcp-first-pivot.md` and its approved detailed implementation
roadmap. Use one focused `pivot/<task-number>-<slug>` branch per task, implement
only the task's declared interfaces, and update
`docs/roadmap/mcp-first-pivot-execution.md` in the same commit with branch,
commits, focused and full verification, deviations, and follow-up risks.

Do not add a dashboard workspace, top-level MCP concept, top-level CLI command,
runtime dependency, or SQLite table unless the roadmap names it or an approved
design amendment authorizes it. Compatibility removal must also be due in
`docs/deprecations.md`.

- Do not commit directly to `main`.
- Start each coherent task from current `main` with a short-lived branch.
- Use branch prefixes `feature/`, `fix/`, `docs/`, `chore/`, `test/`, `release/`, `hotfix/`, or `pivot/`. Reserve `pivot/` for tasks in the approved MCP-first roadmap.
- Keep each branch focused on one issue, one reviewable task, or one release.
- Do not create a long-lived `develop` branch.
- Do not mix release prep with unrelated feature work.
- Push task branches and open a PR for all changes headed to `main`.
- Prefer squash merge for ordinary task PRs so `main` stays readable.
- Use the PR as the review artifact even when there is only one maintainer.

Recommended branch names:

```text
feature/<issue-number>-short-description
fix/<issue-number>-short-description
docs/<issue-number>-short-description
chore/<issue-number>-short-description
test/<issue-number>-short-description
release/0.4.0
hotfix/0.3.3
```

Before starting a task branch:

```bash
git switch main
git pull --ff-only
git switch -c docs/123-short-description
```

## Agent Boundaries


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [douglasmonsky/codex-usage-tracker](https://github.com/douglasmonsky/codex-usage-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
