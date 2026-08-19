---
trigger: always_on
description: **Codelens AI** (`codelens-ai` on npm) is a CLI tool that measures ROI from AI coding agents by correlating token usage with git commit output. It parses **Claude Code** session files (`~/.claude/projects/`), **OpenAI Codex CLI** rollout files (`~/.codex/sessions/`), and **GitHub Copilot** sessions from the standalone CLI (`~/.copilot/session-state/`) plus official VS Code Chat/agent mode (`workspaceStorage/*/chatSessions/`), analyzes git history, and serves an interactive dashboard at `http://l
---

# CLAUDE.md — Codelens-AI

## Project Overview

**Codelens AI** (`codelens-ai` on npm) is a CLI tool that measures ROI from AI coding agents by correlating token usage with git commit output. It parses **Claude Code** session files (`~/.claude/projects/`), **OpenAI Codex CLI** rollout files (`~/.codex/sessions/`), and **GitHub Copilot** sessions from the standalone CLI (`~/.copilot/session-state/`) plus official VS Code Chat/agent mode (`workspaceStorage/*/chatSessions/`), analyzes git history, and serves an interactive dashboard at `http://localhost:3457` with per-agent source tabs (All Agents / Claude Code / OpenAI Codex / GitHub Copilot).

**Version:** 0.9.0
**License:** MIT
**npm package:** `codelens-ai` (alias: `claude-roi`)

## Tech Stack

- **Runtime:** Node.js >= 22.12, ES modules (`"type": "module"`)
- **Backend:** Express.js 5.0.0
- **CLI:** Commander.js 13.0.0
- **MCP:** `@modelcontextprotocol/sdk` — `codelens-ai mcp` serves the reports as MCP tools over stdio (`src/mcp.js`); stdout carries only JSON-RPC, so all progress output must go to stderr there
- **Frontend:** Single-file HTML (`src/dashboard.html`) with vanilla JS + Chart.js 4.5.1. The UMD bundle is **committed at `src/vendor/chart.umd.min.js`** and served at `/vendor/chart.umd.min.js` — no CDN, works offline, and does not depend on `chart.js` being resolvable at the user's runtime (npx caches have shipped partial `node_modules`). `chart.js` is a **devDependency**; refresh the vendored copy with `npm run vendor:chart` after upgrading it (also runs on `prepublishOnly`).
- **Testing:** `node --test` unit + server-route tests (`tests/unit/`), a packaging smoke test (`npm run test:package` — packs → clean-installs → boots → asserts the dashboard and vendored chart serve), and Playwright (E2E)
- **Styling:** Inline CSS design tokens, "warm-ink instrument panel" (dark) / "warm-paper ledger" (light) theme; fonts: Bricolage Grotesque (display), Instrument Sans (body), IBM Plex Mono (data). Chart palette is CVD-validated per theme — don't swap chart hues casually.

## Project Structure

```
src/
├── index.js           # CLI entry point & orchestration (Commander)
├── banner.js          # Pixel-block "CODELENS AI" startup splash (interactive TTY dashboard runs only)
├── claude-parser.js   # Parses JSONL session files from ~/.claude/projects/
├── codex-parser.js    # Parses OpenAI Codex rollout files from ~/.codex/sessions/
├── copilot-parser.js  # Parses GitHub Copilot CLI events.jsonl from ~/.copilot/session-state/ (GitHub pricing table + LiteLLM fallback)
├── copilot-vscode-parser.js # Parses VS Code Copilot Chat/agent JSONL (recorded AI Credits + tool/file metadata)
├── git-analyzer.js    # Git log analysis, branch detection, diff stats
├── correlator.js      # Matches sessions to commits via file overlap + time window + Co-authored-by trailers
├── metrics.js         # ROI calculations, grades, insights, heatmap, survival rate, AI code share, value leak
├── report.js          # `codelens-ai report` — terminal / Markdown / HTML ROI scorecard
├── tables.js          # `codelens-ai daily|weekly|monthly` — usage/cost tables + ROI columns
├── blocks.js          # `codelens-ai blocks` — configurable usage windows, burn rate, projection
├── statusline.js      # `codelens-ai statusline` — Claude Code statusline (stdin JSON + quickstats: ROI, burn rate)
├── mcp.js             # `codelens-ai mcp` — MCP server over stdio (roi_summary, usage, blocks, sessions, projects, refresh tools)
├── server.js          # Express REST API routes (?source= selects per-agent views)
├── cache.js           # Smart caching with per-source stale file detection + statusline quickstats
├── pricing.js         # External LiteLLM pricing overlay — auto-prices models the hardcoded tables don't know
└── dashboard.html     # Single-file SPA dashboard (4000+ lines)

tests/
├── unit/              # node --test suites (parsers, correlator, metrics, server)
├── e2e/smoke.spec.js  # Playwright smoke suite (fixture-backed, incl. source tabs)
├── fixtures/          # build-fixtures.js generates Claude + Codex session fixtures
└── local/             # full dashboard suite for local runs

.github/workflows/
├── ci.yml             # CI: syntax check, unit tests, CLI smoke (all agents), Node 22/24 matrix
├── codeql.yml         # CodeQL code scanning (javascript-typescript + actions), weekly + per-PR
└── release.yml        # npm publish on version tag push
```

## Data Flow

```
Claude Sessions (JSONL)  → claude-parser.js  ┐
Codex Rollouts (JSONL)   → codex-parser.js   ┼→ [Cache] → git-analyzer.js
Copilot CLI events       → copilot-parser.js ┐
Copilot VS Code sessions → copilot-vscode-parser.js ┘
→ correlator.js (all sources together) → metrics.js (per-source payloads)
→ server.js (REST API, ?source=) → dashboard.html (source tabs)
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Akshat2634/Codelens-AI](https://github.com/Akshat2634/Codelens-AI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
