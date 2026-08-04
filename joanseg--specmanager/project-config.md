---
trigger: always_on
description: <!-- specmanager:start -->
---

<!-- specmanager:start -->
## Project lifecycle (managed by SpecManager — do not edit by hand)

Specs live in `.claude/specs/features/`. Read the approved doc for a feature's stage before implementing it.

| Feature | Current stage | Notes |
|---------|---------------|-------|
| Redesign | PRD (approved) | — |
| Dummy feature | PRD | — |
| Post-phase design conformance check | PRD (draft) | — |
| Markdown viewer | PRD (approved) | — |
| Reinstall refactor | PRD (approved) | — |
| Interview command | PRD (approved) | — |
| Antigravity plugin | PRD (approved) | — |
| Share docs on public URL | PRD (approved) | — |
| Cursor plugin | PRD (approved) | — |
| Codex plugin | PRD (approved) | — |
| User adoption acceleration | PRD (draft) | — |
| Token usage optimisation | PRD (approved) | — |
| Viral loop feature | PRD (approved) | — |
| Feature demo recording | PRD | — |
| Spec-stage tier dispatch | PRD (approved) | — |
| GitHub spec sync (issues/PRs) | PRD | — |
| Security review stage | PRD (approved) | — |
| Multi-repo nested docs (CLAUDE.md / DESIGN.md) | PRD (draft) | — |
| Multi-session boards (auto-port) | PRD (approved) | — |
| Website agent readiness | PRD | — |

_8 features shipped — full history on the board._

**Rules:** don't start a feature's tasks until its Plan is approved; treat ⚠️ stale docs as needing reconciliation.

**Commands:**
`/specmanager-prd` · `/specmanager-architecture` · `/specmanager-design` (optional) · `/specmanager-plan` · `/specmanager-build` · `/specmanager-walkthrough` · `/specmanager-board` · `/specmanager-interview` (optional, pre-PRD)

_Last synced: 2026-07-10T12:08:11.497Z_
<!-- specmanager:end -->

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

This repo **is** the SpecManager plugin (implemented, not a spec). SpecManager is a Claude Code **plugin** that turns a project's lifecycle (PRD → Architecture → optional Design → Plan + tasks → Build → Walkthroughs) into a localhost kanban board backed by plain markdown in the *target* project's repo. Single-user, fully local, bound to `127.0.0.1`, no auth. Claude drafts each stage from the previous approved one plus the existing codebase; the human edits and approves in the board; git tracks every artifact.

The repo also dogfoods itself: its own features live under `.claude/specs/features/` and are driven with the same `/specmanager:specmanager-*` commands.

## Layout

- **`.claude-plugin/marketplace.json`** — marketplace manifest, at the repo root.
- **`plugins/specmanager/`** — the plugin itself:
  - `.claude-plugin/plugin.json` — plugin manifest (`board_port` user config, default 4317 — a *preferred* port: the board falls forward to the next free port if it's taken, so concurrent sessions each get their own board).
  - `.mcp.json` — wires the MCP server: `node server/dist/mcp.js`, with `SPECMANAGER_PROJECT_DIR=${CLAUDE_PROJECT_DIR}`, `SPECMANAGER_BOARD_PORT=${user_config.board_port}`, `NODE_PATH=${CLAUDE_PLUGIN_DATA}/node_modules`.
  - `commands/*.md` — the user-facing slash commands (orchestration prompts). `specmanager-interview.md` is the exception to the delegation pattern: a multi-turn conversation can't live in a single-shot subagent, so its full interview protocol runs in the main session.
  - `agents/*.md` — the subagents the drafting/build commands delegate to (prd-writer, architect, designer, planner, builder, walkthrough-writer, plus `reviewer` — a read-only spec-compliance reviewer the build command runs after a phase's tasks build).
  - `hooks/hooks.json` — `SessionStart` installs runtime deps into `${CLAUDE_PLUGIN_DATA}` once and symlinks them back into `server/node_modules`; `FileChanged` on `.claude/specs/**` nudges a re-read; `Stop` runs `hooks/stop-gate.sh` (see Build leverage primitives below).
  - `server/` — `@specmanager/server`, TypeScript, ships compiled `dist/`.
  - `ui/` — `@specmanager/ui`, React 18 + Vite, ships compiled `dist/`.
- **`docs/`** — `docs/DESIGN.md` is the managed design-system spec; the original full spec and phased plan are archived under `docs/temp/original-specs/` (historical snapshots — don't edit).

## Architecture (the big picture)

Two server entry points, **one shared `core/` module** under `server/src/core/` (re-exported from `core/index.ts`) imported by both. Every mutation — agent or human — flows through `core`, so validation, state transitions, and events are identical; do not duplicate that logic in either entry point.

- **`server/src/mcp.ts`** — the MCP stdio server (Claude's interface). Registers all the tools (`specmanager_init`, `list/create_feature`, `*_document`, `set_status`, `check_gate`, `list_stale`, `*_task`, `list_phases`, `get_next_phase`, `get_phase_completion`, `sync_claude_md`, `sync_design_md`, `open_board`, …). **It also boots the board server in-process** (`startBoardServer`), so one `claude` session brings up everything. It runs `startClaudeMdAutoSync` / `startDesignMdAutoSync` listeners that refresh the managed CLAUDE.md block on doc/status events and `docs/DESIGN.md` on `feature.shipped`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joanseg/specmanager](https://github.com/joanseg/specmanager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
