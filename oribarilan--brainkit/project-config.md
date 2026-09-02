---
trigger: always_on
description: Brainkit is a second-brain plugin for AI coding agents. It runs against three harnesses — OpenCode, GitHub Copilot CLI, and Claude Code — and provides a structured markdown vault organized with the PARA method, with skills that teach the agent domain knowledge and (on OpenCode) a TUI that keeps you connected to your vault.
---

# AGENTS.md

## Project Overview

Brainkit is a second-brain plugin for AI coding agents. It runs against three harnesses — OpenCode, GitHub Copilot CLI, and Claude Code — and provides a structured markdown vault organized with the PARA method, with skills that teach the agent domain knowledge and (on OpenCode) a TUI that keeps you connected to your vault.

Design specs live in `specs/`. Feature definitions live in `docs/features.md`. Read them before making architectural decisions.

### Dev-Side vs Client-Side

This repo contains two kinds of agentic files — it's important to know which is which:

- **Dev-side**: Files that guide AI agents working on brainkit's own codebase. These shape how _you_ (the agent helping the developer) behave when contributing to this repo.
  - `AGENTS.md` (this file)
  - `.opencode/` (dev OpenCode config)

- **Client-side**: Files that brainkit delivers to its users at runtime. These shape how the agent behaves inside a user's OpenCode session when brainkit is installed as a plugin.
  - `skills/` (domain knowledge injected into the user's agent)
  - `core/system-prompt.ts` (builds the system prompt for the user's session)
  - `opencode/server.ts` (hooks that inject client-side behavior)

When editing client-side files, you're changing the experience for brainkit's end users, not your own behavior. When editing dev-side files, you're changing how agents work on this repo.

### Structure

```
core/               # TypeScript — shared vault logic
  vault.ts          # Vault discovery, config, file operations, brag stats
  system-prompt.ts  # Dynamic system prompt builder (client-side)
  types.ts          # Shared types (BrainkitConfig, etc.)
opencode/           # TypeScript/TSX — OpenCode plugin
  server.ts         # Server plugin: system prompt injection, hooks (client-side)
  tui.tsx           # TUI plugin: home logo, sidebar, tips, theme
  side.tsx          # Sidebar component (vault stats)
  tips.tsx          # Rotating tips component
  brainkit.json     # Custom color theme
cli/                # TypeScript — CLI entry point for npx @2brain/brainkit
  index.ts          # Entry point, routes to harness launcher
  launch.ts         # Harness detection, config setup, spawn opencode
  claude.ts         # Claude Code launcher (staging dir, theme, settings, isolation)
claude/             # Claude Code plugin (read-only template; copied into `~/.config/brainkit/claude/plugin/` at launch)
  .claude-plugin/   # Plugin manifest
  hooks/            # Hook config consumed by Claude
  scripts/          # statusline, auto-commit, precompact (run by Claude)
  skills/           # Claude-native skill stubs (e.g. doctor)
skills/             # Markdown — client-side domain knowledge for the user's agent
  brainkit/         # Root skill (conventions, setup flow, overview)
  para/             # PARA method
  bragfile/         # Bragfile feature
  contacts/         # Contacts feature
  meeting-notes/    # Meeting notes feature
  maintenance/      # Vault health and maintenance
  onboarding/       # First-run Q&A guidance
specs/              # Design documents (vision, architecture, decisions)
docs/               # Canonical feature definitions
```

### Development Commands

All actions use [just](https://just.systems/). Run `just` to list all available recipes.

```bash
just            # list all recipes
just oc         # launch OpenCode against the dev brainkit (full pipeline)
just cp         # launch Copilot CLI against the dev brainkit (alias: just ghcp)
just cc         # launch Claude Code against the dev brainkit
just test       # run tests
just test-watch # run tests in watch mode
just lint       # eslint + typecheck
just format     # format with prettier
just check      # lint + format check + test (run before committing)
just build-cli  # compile CLI to dist/ for npm publishing
```

### Running

Brainkit ships as an npm package consumed by three harnesses (OpenCode, Copilot CLI, Claude Code). For local development, use the harness recipes:

```bash
just oc         # OpenCode + dev brainkit
just cp         # Copilot CLI + dev brainkit
just cc         # Claude Code + dev brainkit
```

Each recipe rebuilds the npm tarball (full `prepack`: tsc + shim generator), installs it into `.dev/install/`, and launches the harness using the dev binary. Your real `~/.config/brainkit/` is untouched — dev state lives under `.dev/user-config/`. See `CONTRIBUTING.md` § Testing your changes against a real harness for details.

### CLI (for npm)

```bash
just build-cli    # compile TypeScript to dist/
node dist/cli/index.js --help  # test locally
```

### Testing

```bash
just test       # run once
just test-watch # watch mode
```

Tests live in `__tests__/` directories alongside source. Each test file maps to a module.

---

## Core Principles

### Plan Before You Code

- Read relevant specs in `specs/` and `docs/features.md` before touching architecture
- Break complex tasks into smaller steps
- If requirements are unclear, ask first


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oribarilan/brainkit](https://github.com/oribarilan/brainkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-02 -->
