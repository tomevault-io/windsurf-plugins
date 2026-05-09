---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository. `CLAUDE.md` is a symlink to this file.
---

# AGENTS.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository. `CLAUDE.md` is a symlink to this file.

## Project Overview

> **Note for agents:** This project is branded **OpenClawDreams**. The npm package name, plugin id, tool names, service names, and CLI are all `openclawdreams`. The repo directory is still called `ElectricSheep`.

OpenClawDreams (internal package name: `openclawdreams`) is an OpenClaw extension (TypeScript) that gives an agent an encrypted memory system. It synthesizes the agent's interactions with their human operator, enriching them with context from web searches and (optionally) the Moltbook AI agent community. The core conceit: all memories are encrypted in deep storage — only the dream process can decrypt them. The waking agent sees nothing from ElectricSheep directly; dream insights surface through OpenClaw memory.

The agent processes its daily work into surreal dream narratives at night, then can notify its operator with "I had a dream last night..." to spark conversation about the dream's themes and insights.

Designed to be installed into an existing OpenClaw instance via `openclaw plugins install`. Requires OpenClaw as a runtime dependency — all LLM calls route through the OpenClaw gateway.

## Commands

```bash
# Setup
npm install
npm run build

# OpenClaw integration
openclaw plugins install -l .   # link for development
openclaw plugins list            # verify loaded

# CLI utilities (standalone, no OpenClaw needed)
npx openclawdreams register --name "Name" --description "Bio"  # Moltbook registration (optional)
npx openclawdreams status     # show agent state, memory stats, budget info
npx openclawdreams dreams     # list saved dream journal files

# Tests
npm test                     # node:test + tsx, runs test/**/*.test.ts
```

Tests use Node's built-in test runner (`node:test`) with `tsx` for TypeScript. Each test file creates an isolated temp directory via `OPENCLAWDREAMS_DATA_DIR` so tests don't touch real data.

```bash
# Linting & formatting
npm run lint          # ESLint (typescript-eslint, flat config)
npm run lint:fix      # auto-fix lint issues
npm run format        # Prettier
npm run format:check  # check formatting without writing
```

ESLint uses flat config (`eslint.config.js`) with `typescript-eslint` and `eslint-config-prettier`. Prettier handles formatting (`.prettierrc`). TypeScript strict mode is enabled. Unused variables are errors (prefix with `_` if intentionally unused). CI runs build → lint → format:check → test on every PR.

**AGENT INSTRUCTION:** Before committing any code changes, you MUST run `npm run lint:fix` and `npm run format` to ensure all linting and format issues are automatically resolved. Do not commit code if `npm run lint` or `npm run format:check` continue to produce errors.

## PR Process & Releasing

Use conventional commit prefixes in PR titles. When a PR merges to main, the release workflow automatically bumps the version, updates the changelog, and pushes a git tag.

| Prefix | Version Bump | Example |
|--------|--------------|---------|
| `major:` | 0.2.0 → 1.0.0 | `major: redesign plugin API` |
| `feat:` | 0.2.0 → 0.3.0 | `feat: add slack notifications` |
| `refactor:` | 0.2.0 → 0.3.0 | `refactor: new synthesis pipeline` |
| `fix:` | 0.2.0 → 0.2.1 | `fix: memory leak in dreamer` |
| `bug:` | 0.2.0 → 0.2.1 | `bug: crash on empty input` |
| `docs:` | 0.2.0 → 0.2.1 | `docs: update setup guide` |
| `chore:` | 0.2.0 → 0.2.1 | `chore: bump dependencies` |

Commits containing `BREAKING CHANGE` in the body also trigger a major bump.

**Workflow:**
1. Create PR with conventional commit prefix in title
2. CI runs build → lint → format:check → test
3. Merge to main
4. Release workflow automatically:
   - Determines version bump from commit message
   - Runs `standard-version` to update version and CHANGELOG.md
   - Creates a release branch + PR back to main with tag `vX.Y.Z`

No manual release steps required — just merge and the release happens.

## Architecture

### OpenClaw Extension Entry

`src/index.ts` exports a `register(api)` function called by the OpenClaw plugin loader. It registers:

**5 tools:**
- `openclawdreams_reflect` — run the reflection cycle (analyze conversations, gather context, synthesize)
- `openclawdreams_check` — legacy alias for `openclawdreams_reflect`
- `openclawdreams_dream` — run the dream cycle (decrypt, dream, consolidate). Note: when triggered via this tool (manually), the `api` is not passed to `runDreamCycle`, so OpenClaw memory storage and operator notifications are skipped
- `openclawdreams_journal` — post latest dream to Moltbook (no-op if Moltbook disabled)
- `openclawdreams_status` — return agent state and deep memory stats

**2 hooks:**
- `before_agent_start` — captures `workspaceDir` for identity loading
- `agent_end` — captures `conversationSummary` and runs `git diff --stat HEAD` to record `file_diffs`; both are encrypted into deep memory as an `interaction`

**1 background scheduler service (replaces cron jobs):**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RogueCtrl/OpenClawDreams](https://github.com/RogueCtrl/OpenClawDreams) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
