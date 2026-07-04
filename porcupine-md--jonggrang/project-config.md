---
trigger: always_on
description: > Guide for any AI agent (Claude Code, Codex, OpenCode, Jonggrang) working in this repo.
---

# CLAUDE.md — Project Guide for AI Agents

> Guide for any AI agent (Claude Code, Codex, OpenCode, Jonggrang) working in this repo.
> `AGENTS.md` is a symlink to this file — both point to the same source.

---

## What Jonggrang Is

Jonggrang is a **CLI orchestrator for AI development workflows**. It runs the pipeline `Plan → Implement → Simplify → Test → Review` with one atomic task per fresh-context agent, enforced by hooks (secrets blocker, context overload guard, quality gate). The goal is not to make agents faster — they are already fast enough — but to force them to stop and clean up before complexity quietly accumulates.

Two primary modes:
- **Work Loop** — iterative, stateless, one agent per task. Good for small-to-medium features.
- **Orchestrate Mode** — deterministic 16-phase, 5 specialist agents, persistent state via `MANIFEST.yaml`. For serious delivery.

Agent backends: `opencode`, `claude`, `codex`, `jonggrang` (Pi SDK in-process). Selected via `.jonggrang/jonggrang.json`.

Entry points: `bin/jonggrang` (CLI), `client/` (Pi TUI), `server.js` (web dashboard). Hooks live in `hooks/`, skills in `skills/`, templates in `templates/`.

→ Full detail: [`docs/JONGGRANG.md`](docs/JONGGRANG.md) · [`docs/PHILOSOPHY.md`](docs/PHILOSOPHY.md) · [`docs/WORKFLOW.md`](docs/WORKFLOW.md)

---

## Read These When You Need Context

Two docs are the canonical source of truth for the project's *intent*. Read them before any non-trivial change — do not infer architecture from code alone:

- **[`docs/PHILOSOPHY.md`](docs/PHILOSOPHY.md)** — Why Jonggrang exists, the pipeline as a quality gate, the five-layer stack, hook layers, autonomy modes, project file structure. Read this when:
  - You are touching the pipeline (`Plan → Implement → Simplify → Test → Review`), any hook in `hooks/`, the compaction gate, or the feedback loop.
  - You are unsure *why* a constraint exists (e.g., why coordinators cannot edit files, why exit is blocked until tests pass).
  - You are tempted to "simplify" something that looks redundant — it is probably load-bearing.

- **[`docs/JONGGRANG.md`](docs/JONGGRANG.md)** — Full blueprint: core principles (Thin Agent / Fat Platform, Context Engineering, Deterministic Enforcement), 16-phase orchestration, five-role assembly line with tool restrictions, skill tier system, team/parallel mode, quality gates, config schema, compound learning channels. Read this when:
  - You are adding/changing a phase, role, skill tier, or agent backend.
  - You are touching `MANIFEST.yaml`, `.jonggrang/.output/`, `.jonggrang/.ephemeral/`, or any persistent state file.
  - You need to understand work-type classification (BUGFIX/SMALL/MEDIUM/LARGE) and phase-skipping logic.
  - You are working on team mode, file ownership locks, or parallel worktree execution.

**Rule of thumb:** if your change would alter any concept defined in those two files, read the relevant section first, then update both the code *and* the doc in the same task. Do not let the code drift from the philosophy.

---

## Iron Rule: Every Change, Check the Docs

**README.md is the storefront. `docs/` is the manual.** Both must stay aligned with the actual state of the repo.

Before declaring a task done, walk through the checklist below based on what you changed.

### Change → docs to update

| If you change… | Check & update… |
|---|---|
| **CLI command** (add/rename/remove a subcommand in `bin/`) | `README.md` ("Commands at a Glance" table + examples), `docs/QUICKSTART.md`, `docs/EXAMPLE.md` |
| **Flag / option** on an existing command | `README.md` ("Quick flags"), `docs/CONFIG.md` if it touches the config schema |
| **Config schema** (`.jonggrang/jonggrang.json`, `~/.jonggrang/settings.json`) | `docs/CONFIG.md` (required), `README.md` (jsonc snippet), `templates/` |
| **Hook** added or modified in `hooks/` | `docs/PHILOSOPHY.md` (hooks section), `docs/WORKFLOW.md` if the hook participates in a specific phase |
| **Skill** in `skills/` (core or library) | `docs/SKILLS.md` (required), `SKILL.md` if the format or tier model changes |
| **Pipeline phase** / workflow order | `docs/WORKFLOW.md` (required), `docs/JONGGRANG.md` (16-phase section), `docs/PHILOSOPHY.md` (pipeline diagram), `docs/ORCHESTRATION.md`, `docs/EXAMPLE.md` |
| **Role / agent definition** (Lead, Developer, Reviewer, Test Lead, Tester) | `docs/JONGGRANG.md` (Five-Role Assembly Line), `docs/PHILOSOPHY.md` (tool restriction boundary), `templates/agents/` |
| **Hook layer / compaction gate / feedback loop** | `docs/PHILOSOPHY.md` (Hooks section), `docs/JONGGRANG.md` (Eight-Layer Defense) |
| **Skill tier model** (Core vs Library, Gateway pattern) | `docs/SKILLS.md` (required), `docs/JONGGRANG.md` (Skill System), `docs/PHILOSOPHY.md` (Two-Tier section) |
| **New agent backend** or integration change | `docs/AGENTTOOLS.md` (required), `README.md` (Requirements section + config example) |
| **Persistent state schema** (`MANIFEST.yaml`, `.jonggrang/.output/`, `.ephemeral/`, locks) | `docs/JONGGRANG.md` (Project File Structure + state sections), `docs/PHILOSOPHY.md` (Persistent State) |
| **Philosophy / architecture** shift | `docs/PHILOSOPHY.md`, `docs/JONGGRANG.md`, `docs/ORCHESTRATION.md` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [porcupine-md/jonggrang](https://github.com/porcupine-md/jonggrang) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
