---
trigger: always_on
description: - **Purpose**: AI-powered SDLC pipeline built on Claude Code — from user description to deployed, tested, secured application
---

# Shipwright SDLC Framework

## WHAT
- **Purpose**: AI-powered SDLC pipeline built on Claude Code — from user description to deployed, tested, secured application
- **Architecture**: Monorepo of Claude Code plugins (skills + hooks + scripts)
- **Stack**: Python 3.11+ scripts, Claude Code plugin system, uv package manager

## Structure
```
plugins/                    # Claude Code plugins (one per SDLC phase)
  shipwright-run/           # Orchestrator (entry point)
  shipwright-project/       # Requirements decomposition (IREB)
  shipwright-design/        # UI mockups from IREB specs (HTML)
  shipwright-plan/          # Deep planning + external LLM review
  shipwright-build/         # TDD implementation
  shipwright-test/          # Testing (unit + smoke + Playwright E2E)
  shipwright-security/      # Scanner chain + remediation loop
  shipwright-deploy/        # Deployment (extensible flavors)
  shipwright-changelog/     # Git sync + changelog + PR
  shipwright-compliance/    # IREB traceability, RTM, SBOM, dashboard
  shipwright-iterate/       # Daily iteration (complexity-adaptive)
  shipwright-preview/       # Local browser preview
  shipwright-adopt/         # Brownfield onboarding (analyze an existing repo)
  shipwright-grade/         # Read-only Control Grade (A–F) for any repo (lead magnet)
# Command Center WebUI lives at github.com/svenroth-ai/shipwright-webui since v0.4.0
shared/                     # Shared across all plugins
  contracts/                # Cross-plugin public API (B8): compliance.py, iterate.py
  profiles/                 # Stack profile definitions (JSON) + deploy profiles
  templates/                # CLAUDE.md, .shipwright/agent_docs, CI templates
  prompts/                  # Shared subagent prompts (code_reviewer, iterate_reviewer)
  schemas/                  # JSON schemas (run_config v2)
  config/                   # Shared config (external_review.json)
  scripts/                  # Shared Python utilities
  tests/                    # Tests for shared scripts and hooks
  constitution.md           # ALWAYS / ASK FIRST / NEVER rules for all agents
scripts/                    # Top-level scripts (install.sh, verify-setup.sh)
docs/                       # User-facing docs (guide.md, hooks-and-pipeline.md)
integration-tests/          # Cross-plugin integration tests
CHANGELOG-unreleased.d/     # Pending changelog drop files (aggregated at release)
```

## HOW

### Development
```bash
uv sync                              # Install dependencies
uv run pytest tests/ -v               # Run tests for a plugin (from plugin dir)
uv run pytest integration-tests/ -v   # Run integration tests (from root)
uvx ruff@0.15.15 check .              # Bug-focused lint — GATING in CI (ci.yml)
```

**Lint is a hard CI gate.** `.github/workflows/ci.yml` runs `uvx ruff@0.15.15
check .` with no `|| true` / `continue-on-error`, so a lint failure blocks merge.
The ruleset is deliberately curated (Pyflakes + a few bug-class pycodestyle
rules, cosmetic rules omitted) and lives in the root `pyproject.toml`
`[tool.ruff.lint]` — run it locally before pushing. ruff is pinned (not a project
dependency) so a new release can't silently change the gate.

### Plugin Structure (each plugin follows this pattern)
```
plugins/shipwright-{name}/
  .claude-plugin/plugin.json          # Plugin metadata
  hooks/hooks.json                    # Claude Code hooks
  agents/                             # Subagent definitions (markdown)
  skills/{name}/SKILL.md              # Main skill definition (folder = slash command suffix)
  scripts/                            # Python scripts (checks, hooks, lib, tools)
  tests/                              # Plugin-specific tests
  pyproject.toml                      # Plugin dependencies
```

### Key Environment Variables
```
SHIPWRIGHT_SESSION_ID        # Unified session ID across all plugins
SHIPWRIGHT_PLUGIN_ROOT       # Absolute path to active plugin directory
```

### Conventions
- All scripts invoked via `uv run`
- Hooks use `${CLAUDE_PLUGIN_ROOT}` for path resolution
- Config files: `shipwright_*_config.json` (written to target project)
- Env var prefix: `SHIPWRIGHT_`
- Config file prefix: `shipwright_`

### Hooks & Pipeline Reference
- **Reference doc:** `docs/hooks-and-pipeline.md`
- **ALWAYS read this file first** when working on any plugin. It contains the
  complete context loading matrix (who reads what), artifact write matrix (who
  writes what), hooks registry, config data flow, and between-phase actions.
- **Rule:** When modifying any hook (hooks.json), adding/removing a pipeline phase,
  changing phase validators, altering between-phase actions, or changing what a
  plugin reads at startup (context loading), you MUST update
  `docs/hooks-and-pipeline.md` to reflect the change.
- This document is the single source of truth for understanding what fires when,
  who reads/writes which artifacts, and the impact of pipeline changes.

### When editing plugin-side files

Changes under `plugins/*`, `shared/scripts/`, or any `SKILL.md` file do
NOT auto-sync to the plugin cache at `~/.claude/plugins/cache/shipwright/`
that Claude Code uses at runtime. After `git push`, run:

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [svenroth-ai/shipwright](https://github.com/svenroth-ai/shipwright) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-12 -->
