---
trigger: always_on
description: This file tells Claude Code (and other AI agents) how to work with this repository.
---

# CLAUDE.md — Agent Instructions

This file tells Claude Code (and other AI agents) how to work with this repository.

---

## What This Repo Is

A Claude Code guide, packaged as a plugin. There are seven skills:

- **`/guide:onboard`** — Interactive getting-started walkthrough for new users
- **`/guide:ask`** — Q&A skill backed by comprehensive reference documentation in `skills/ask/references/`
- **`/guide:introspect`** — Introspective Development — audit lifecycle coverage, find gaps, and improve your development environment
- **`/guide:game-mode`** — Gamified usage tracker that rewards feature breadth and depth with a level system
- **`/guide:level-up`** — Feature roadmap and personalized next-step coaching
- **`/guide:migrate-to-claude`** — Smart migration from other AI coding assistants (Cursor, Windsurf, Copilot, Codex, etc.) to Claude Code
- **`/guide:visualize-setup`** — Generate an interactive HTML dashboard of all installed Claude Code skills and MCP servers

This repo serves two audiences: humans browsing the docs on GitHub, and Claude Code users who install it as a plugin to get guided help.

This is primarily a **content** repo — no application code, no build system. Bash hook scripts are tested with bats-core (see `tests/`).

---

## Plugin vs. Project Tooling

This repo serves two purposes — distinguish them before starting any work:

### Plugin features (ships to users)

Everything under `skills/`, `hooks/`, `.claude-plugin/`, and reference docs. These are what users get when they install the plugin:

- Skills (SKILL.md files)
- Hooks (hooks.json + tracking scripts)
- Reference documentation (`skills/ask/references/`)
- Game mode, level-up, migration, visualize-setup
- Plugin manifests (plugin.json, marketplace.json)

### Project tooling (maintains the repo)

Infrastructure that helps develop and maintain the plugin but does NOT ship to users:

- Agents (`agents/`) — doc-verifier, pr-review, version-bump
- CI workflows (`.github/workflows/`) — docs-freshness, pages, tests
- Test suite (`tests/`) — bats tests for hook scripts
- CLAUDE.md, README.md — repo documentation

### Before starting work

**Always ask: "Is this a plugin feature or project tooling?"** This determines:

| | Plugin feature | Project tooling |
|---|---|---|
| **Where** | `skills/`, `hooks/`, `.claude-plugin/` | `agents/`, `.github/`, `tests/` |
| **Version bump?** | Always | Usually not |
| **How to test** | User-facing verification | `bats tests/*.bats` or CI |

If it's unclear, ask before proceeding.

---

## Repository Structure

```text
claude-code-guide/
├── .claude-plugin/
│   ├── plugin.json ........................ Plugin manifest (name: "guide", version, metadata)
│   └── marketplace.json .................. Marketplace manifest
├── .github/
│   └── workflows/
│       ├── docs-freshness.yml ............. Weekly automated docs accuracy checker
│       ├── pages.yml ...................... Jekyll build + raw markdown deploy
│       └── tests.yml ...................... Bash test suite (bats-core)
├── _includes/
│   ├── footer_custom.html ................. Disclaimer footer
│   └── head_custom.html ................... Raw markdown <link> header
├── _sass/
│   └── color_schemes/
│       └── anthropic.scss ................. Anthropic cream color scheme
├── hooks/
│   ├── hooks.json ......................... Hook event configuration (PostToolUse, UserPromptSubmit, Stop)
│   └── scripts/
│       ├── track-usage.sh ................. PostToolUse handler — tracks feature usage
│       ├── track-prompt.sh ................ UserPromptSubmit handler — tracks slash-command usage
│       ├── track-stop.sh .................. Stop handler — token tracking, session counting, and Fibonacci nudges
│       └── migrate-data.sh ................ Lightweight schema migration on version upgrade
├── skills/
│   ├── onboard/
│   │   └── SKILL.md ...................... Interactive getting-started walkthrough
│   ├── ask/
│   │   ├── SKILL.md ...................... Q&A against reference docs
│   │   └── references/ ................... Reference docs organized by difficulty tier
│   │           ├── beginner/ ............. 🌱 Beginner feature docs
│   │           │   ├── built-ins.md
│   │           │   ├── choosing-your-model.md
│   │           │   ├── memory.md
│   │           │   ├── setting-your-environment.md
│   │           │   └── starting-to-work.md
│   │           ├── intermediate/ ......... 🌿 Intermediate feature docs
│   │           │   ├── automating-your-workflows.md
│   │           │   ├── best-practices.md
│   │           │   ├── configuring-your-claude.md
│   │           │   ├── github-actions.md
│   │           │   ├── hooks.md
│   │           │   ├── loop.md
│   │           │   ├── marketplace.md
│   │           │   ├── mcp.md
│   │           │   ├── plugin-examples.md
│   │           │   ├── plugins.md
│   │           │   └── skills.md
│   │           ├── expert/ ............... 🌳 Expert feature docs
│   │           │   ├── agent-sdk.md
│   │           │   ├── hooks-http.md
│   │           │   ├── ongoing-work.md
│   │           │   ├── introspective-development.md
│   │           │   ├── sub-agents.md
│   │           │   └── team-mode.md

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OriNachum/claude-code-guide](https://github.com/OriNachum/claude-code-guide) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
