---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# Developing claude-kit

This repository **is** claude-kit — an evidence-gated SDLC for Claude Code: a scaffolder that
installs a stack-agnostic, autonomous-SDLC **configuration** (no application code, no Docker) into
a Claude Code project. It is
distributed two ways from one source of truth:

1. **As a Claude Code plugin** — components are auto-discovered from the repo root.
2. **As a pip package** — `claude-kit` (CLI: `claude-kit` / `ckit` / `claude-sdlc`) scaffolds the
   same content into any project's `.claude/`, driven by the `catalog/` (stacks · profiles · MCP).

> Note: the files in this repo are the **kit's payload**, not rules for this repo itself. The
> generic engineering ruleset that gets installed into user projects lives in
> `templates/CLAUDE.md`, **not** this file.

## Core architecture

The kit's central data flow is **Selection → catalog.resolve() → ResolvedPlan → scaffold.install_sdlc()**:

1. **`Selection`** (`models.py`) — user choices (stack, profile, MCP servers, scope, autonomy, …)
   collected interactively by `prompts.py` or passed via `--defaults`/`--config`.
2. **`catalog.resolve()`** (`catalog.py`) — reads `catalog/stacks.yaml`, `profiles.yaml`,
   `mcp.yaml`, `org.yaml` and resolves the Selection into a **`ResolvedPlan`**: which agents,
   skills, hooks, overlay rules, overlay agents, gates, and MCP fragments to install. This
   function must stay **branch-free** — no stack- or org-specific code paths.
3. **`scaffold.install_sdlc()`** (`scaffold.py`) — takes the ResolvedPlan and writes files into
   the target project: `CLAUDE.md`, `.claude/{rules,agents,skills,hooks,templates,config}`,
   optional `.mcp.json` and org packs. Records per-file checksums + ownership in
   `.claude/config/init-options.json` for safe upgrades.
4. **`upgrader.upgrade()`** (`upgrader.py`) — compares the live install against the plan from the
   current kit version, refreshes kit/overlay files whose checksums match, preserves user-edited
   files (drops new versions as `.claude-kit` sidecars), and writes a transactional journal
   (`upgrade-in-progress.json`) so an interrupted run can be safely resumed.
5. **`export`** (`export.py`) — projects the same `ResolvedPlan` into Cursor (`.cursor/rules/*.mdc`),
   a root `AGENTS.md`, or GitHub Copilot (`.github/copilot-instructions.md`) for non-Claude-Code editors.

### key source modules

| Module | Role |
|--------|------|
| `cli.py` | Typer app — all CLI commands (`init`, `validate`, `doctor`, `diff`, `export`, `upgrade`, `pipeline`, `list-options`, `status`, `privacy-report`). Experimental/planned commands are hidden unless `CLAUDE_KIT_EXPERIMENTAL=1`. |
| `models.py` | Dataclass contracts: `Selection`, `ResolvedPlan`, `OrgPlan`, `InitOptions`, `FileRecord`, `UpgradeJournal`. The typed seam between prompts → resolver → installer → upgrader. |
| `catalog.py` | The resolver — converts a `Selection` into a `ResolvedPlan` by reading the catalog YAML files. Must stay branch-free. |
| `prompts.py` | Interactive `init` question flow (stack, profile, MCP, org scope, …). |
| `scaffold.py` | Writes the resolved plan to disk. Also provides `payload_dir()` which locates the bundled payload (source checkout or wheel `_payload/`). |
| `render.py` | Jinja2 rendering — `CLAUDE.md` and `README.claude-sdlc.md` templates. |
| `hooks.py` | The `HOOK_REGISTRY` (all hook definitions + script mappings) plus `PLUGIN_HOOK_IDS`, `STARTER_HOOK_IDS`, and `PLUGIN_ONLY_HOOKS`. Contains the drift-test logic used by `gen_hooks.py --check`. |
| `upgrader.py` | Safe, edit-preserving upgrade: checksum comparison, transactional journal, sidecar fallback for user-edited files. |
| `validator.py` | Structural validation of an installed config (`validate` + `--strict` mode). |
| `schemas.py` | JSON Schema definitions for catalog integrity checks (`validate --strict`). |
| `pipeline.py` | Deterministic `/sdlc` state-file ops (validate, status, close-gate, skip-gate, abort) — owns the append-only `gate_history` ledger (order enforcement, evidence sha256, atomic locked writes); inspects/records state, does **not** run the pipeline. |
| `export.py` | Projection into Cursor / AGENTS.md / Copilot formats. |
| `detect.py` | Stack detection from a target repo (heuristic-based; non-blocking). |
| `report.py` | Human-readable reporting for `doctor` and `status` commands. |
| `__init__.py` | The `__version__` string — one of the five places that must be bumped on release. |

## Repository layout

| Path | Purpose |
|------|---------|
| `.claude-plugin/plugin.json` | Plugin manifest (name, version, hooks path) |
| `.claude-plugin/marketplace.json` | Marketplace entry so `/plugin marketplace add` works |
| `agents/` | SDLC pipeline subagents (auto-discovered by the plugin); each carries a `tier:` field |
| `skills/` | Agent skills (auto-discovered by the plugin); `skills/sdlc/` is the `/sdlc` entrypoint |
| `commands/` | Slash commands: `/claude-kit:init`, `:sdlc`, `:status`, `:abort` (init prefers the pip CLI, falls back to `init.sh`; sdlc delegates to the `sdlc` skill; abort tears down an in-progress `/sdlc` run) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ajyadav013/claude-kit](https://github.com/ajyadav013/claude-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
