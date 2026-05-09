---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## IMPORTANT: If User Just Pasted This Repo URL

**Guide them through installation immediately. Don't wait for them to ask.**

Say: "I see you want to install OSS Autopilot! Let me help you set it up."

Then follow the steps below.

### Step 1: Check prerequisites

```bash
node --version  # Need 20+
gh auth status  # Need GitHub CLI authenticated
```

If `gh` is not installed or authenticated:
> "You'll need the GitHub CLI for this plugin. Install it from https://cli.github.com/ and run `gh auth login`."

### Step 2: Install the plugin (marketplace)

```
/plugin marketplace add costajohnt/oss-autopilot
/plugin install oss-autopilot@oss-autopilot
```

### Step 3: Restart and run setup

> "Great! The plugin is installed. Please restart Claude Code to load it, then run `/setup-oss` to configure your preferences."

After restart, `/oss` and `/setup-oss` commands will be available.

The CLI auto-builds on first run (requires Node.js 20+ and npm).

---

## For Developers: Project Overview

oss-autopilot is a **Claude Code plugin with a TypeScript CLI backend** for managing open source contributions. The repo is structured as a **pnpm monorepo**.

### Architecture

The system has three layers:

1. **Plugin Layer** (`commands/`, `agents/`, `skills/`) — Markdown-based Claude Code plugin components. Commands like `/oss` orchestrate the workflow. Agents handle specific tasks (PR response, CI diagnosis, issue scouting). Skills contain contribution best practices.

2. **TypeScript CLI** (`packages/core/src/cli.ts` → `packages/core/dist/cli.bundle.cjs`) — Commander-based CLI that the plugin invokes with `--json` for structured output. Entry point is `packages/core/src/cli.ts`, which registers subcommands from `packages/core/src/commands/`. The CLI is bundled into a single CJS file via esbuild for portability.

3. **Core Logic** (`packages/core/src/core/`) — The domain layer. Key modules:
   - `types.ts` — All type definitions. Key PR type: `FetchedPR` (ephemeral, fetched fresh each run in v2). `TrackedPR` was removed in v2.
   - `state.ts` — `StateManager` singleton. Reads/writes `~/.oss-autopilot/state.json`. Handles v1→v2→v3 migration and auto-backups
   - `pr-monitor.ts` — `PRMonitor` class. Fetches open PRs from GitHub Search API, enriches each with CI status, review decision, merge conflicts, maintainer comments, and computes `FetchedPRStatus`
   - `github.ts` — Shared Octokit instance with `@octokit/plugin-throttling` for rate limit handling
   - `utils.ts` — GitHub URL parsing, date helpers, token detection (tries `$GITHUB_TOKEN` then `gh auth token`)
   - Issue discovery and vetting are delegated to `@oss-scout/core` via `commands/scout-bridge.ts`

### Key Design Decisions

- **v2 "Fresh Fetch" architecture**: PRs are NOT stored in local state. On each `daily` run, all open PRs are fetched from GitHub's Search API. The `TrackedPR` type and legacy PR arrays have been fully removed.
- **`--json` contract**: Every CLI command supports `--json`, outputting `{ success: boolean, data?: T, error?: string, timestamp: string }` (see `packages/core/src/formatters/json.ts`). The plugin layer parses this structured output.
- **State lives in `~/.oss-autopilot/`**, not in the repo. This separates user data from plugin code.
- **GitHub auth**: The CLI checks for a token via `$GITHUB_TOKEN` env var (preferred) or `gh auth token` CLI fallback. Commands that don't need GitHub access are marked `localOnly` in the command registry.
- **pnpm monorepo**: Development uses pnpm workspaces. Plugin auto-build scopes `npm install` to `packages/core/` (end users don't need pnpm).

### File Structure

```
Repo root (also the Claude Code plugin directory):
├── commands/oss.md, setup-oss.md       # Plugin slash commands
├── agents/*.md                          # 7 specialized agents
├── skills/oss-contribution/SKILL.md     # Contribution index (universal rules)
├── skills/pr-etiquette/SKILL.md         # Review responses, PR descriptions, dormant follow-up
├── skills/contribution-ethics/SKILL.md  # AI attribution, AI-tell avoidance, defer-to-human
├── hooks/session-start.sh               # Plugin session start hook
├── workflows/*.md                       # Workflow orchestration files
├── .claude-plugin/plugin.json           # Plugin manifest
├── .claude-plugin/marketplace.json      # Marketplace catalog
├── packages/
│   ├── core/                            # @oss-autopilot/core (npm package)
│   │   ├── src/
│   │   │   ├── cli.ts                   # CLI entry point (commander setup)
│   │   │   ├── commands/                # CLI subcommands (daily, search, track, etc.)
│   │   │   ├── core/                    # Domain logic + tests
│   │   │   └── formatters/json.ts       # JSON output formatter
│   │   ├── dist/cli.bundle.cjs          # Built bundle (gitignored, auto-generated)
│   │   ├── package.json                 # Published to npm, has bin + exports
│   │   └── tsconfig.json
│   └── dashboard/                       # @oss-autopilot/dashboard (interactive SPA)
│       └── package.json
├── pnpm-workspace.yaml                  # Workspace definition

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [costajohnt/oss-autopilot](https://github.com/costajohnt/oss-autopilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
