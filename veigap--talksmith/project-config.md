---
trigger: always_on
description: This file is the project-instruction file for **plugin authors and contributors** working in this repo. It is loaded by Claude Code only when a session is opened at the root of this repo to develop the plugin. End users never see it.
---

# Talksmith — plugin development notes

This file is the project-instruction file for **plugin authors and contributors** working in this repo. It is loaded by Claude Code only when a session is opened at the root of this repo to develop the plugin. End users never see it.

> **Two unrelated `CLAUDE.md` files — don't confuse them.**
> - **This file** (`/Users/.../talksmith/CLAUDE.md`) is the plugin source repo's dev notes. It exists only here.
> - A **user's `CLAUDE.md`** is a per-directory stub that activates Talksmith for one subject working directory. It is created by `/talksmith:init` from [`talksmith-orch.md`](talksmith-orch.md), lives in the user's cwd, and is completely separate from this file.
>
> Installing the plugin (`/plugin install talksmith@talksmith`) is a one-time, machine-wide action — it does **not** create any `CLAUDE.md` anywhere. Initializing Talksmith for a working directory (`/talksmith:init`) is a separate, per-directory action that writes the stub. A user can install the plugin once and then run `/talksmith:init` in many different directories.

For the user-facing project overview, see [`README.md`](README.md). For the full Presenter Agent operating spec (eight subagents, eight steps, schemas, interaction defaults), see [`orchestrator.md`](orchestrator.md) — that file stays in the plugin install and is auto-imported at session start by the thin [`talksmith-orch.md`](talksmith-orch.md) stub that `/talksmith:init` writes into a user's subject working directory.

## What this repo is

The **Talksmith** Claude Code plugin. Installable surface:

| Path | Purpose |
|---|---|
| [`.claude-plugin/plugin.json`](.claude-plugin/plugin.json) | Plugin manifest (name, version, description). |
| [`agents/`](agents/) | Eight Claude Code subagents — `librarian`, `composer`, `editor`, `diagram-illustrator`, `image-illustrator`, `diagram-critic`, `slide-classifier-critic`, `global-librarian`. Each has YAML frontmatter (`name:`, `description:`) so it can be dispatched by name. **`diagram-illustrator`** and **`image-illustrator`** are Step-6 siblings: the former renders authored ASCII → SVG (diagrams); the latter generates atmospheric aside imagery from `<!-- generate-image: … -->` directives (tool-agnostic, degrades when no image capability is present). **`diagram-critic`** is the odd one out: it is dispatched by the `diagram-illustrator`'s per-block subagent rather than by the orchestrator, and its `tools: Read` restriction is load-bearing — it reviews a rendered diagram from the PNG alone, and never receives the SVG path. That blindness is the point (see its own file). **`slide-classifier-critic`** is its twin one artifact earlier: dispatched by the `md-to-deck` skill (not the orchestrator) once per content slide between FILL and RENDER, it re-runs the catalog's discriminator walk on one slide's source and confirms or overturns the template the fill chose. Its `tools: Read` restriction is load-bearing for the same reason — it is blind to every *other* slide's classification, because a critic that can see the deck is `concept-breakdown` twenty times reads the twenty-first as normal. |
| [`commands/`](commands/) | Slash commands. Currently one: [`/talksmith:init`](commands/init.md). |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [veigap/talksmith](https://github.com/veigap/talksmith) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-15 -->
