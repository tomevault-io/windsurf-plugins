---
trigger: always_on
description: > **Primary Claude Code workspace.** All sessions run from this repo root.
---

# CLAUDE.md — Code Insights

> **Primary Claude Code workspace.** All sessions run from this repo root.
> See `docs/` for detailed documentation. This file is the quick reference.

---

## Project Overview

**Code Insights** is an open-source CLI tool and embedded dashboard for analyzing AI coding sessions. It parses session history from multiple AI coding tools (Claude Code, Cursor, Codex CLI, Copilot CLI, VS Code Copilot Chat), stores structured data in a local SQLite database, and provides both terminal analytics and a browser-based dashboard with LLM-powered insights.

**Architecture:** Single-repo pnpm workspace monorepo with three packages: CLI, dashboard (Vite + React SPA), and server (Hono API).

**Privacy model:** Fully local-first. No cloud accounts, no sign-ups, no data leaves the machine. SQLite database at `~/.code-insights/data.db`.

---

## Development Philosophy (CRITICAL)

**No MVPs, no prototypes, no half-measures.** This product is LIVE with real users. Every feature ships as a full, complete implementation. We do not build "minimum viable" anything — we build the real thing, iterate based on feedback, and revert or update if it doesn't work out.

This principle applies to planning, designing, AND implementation:
- **Planning:** Don't scope down to "MVP facet set" vs "ideal set." Design the complete solution.
- **Designing:** Don't propose phased rollouts with "ship phase 1, add phase 2 later." Design it right the first time.
- **Implementing:** Don't cut corners with "we can add this later." Build it now or explicitly decide not to build it.

---

## Configuration Hierarchy

| Priority | Source | Scope |
|----------|--------|-------|
| 1 (Highest) | This project CLAUDE.md | Code Insights workflows, ceremony, agents |
| 2 | Session Mode | Educational context, learning mode |
| 3 | Global ~/.claude/CLAUDE.md | General best practices |

**Key overrides from global config:**

| Behavior | Global Default | Code Insights Override |
|----------|---------------|----------------------|
| Planning | Ask first | Sub-agents autonomous in their domain |
| File Creation | Ask first | Agents create files autonomously in their domain |
| Review Process | Single reviewer | Triple-layer (TA Insider + Outsider + Synthesis) |
| PR Merges | Normal | **BLOCKED** — only founder merges |

---

## Documentation Index

| Document | Contents |
|----------|----------|
| [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) | Repository structure, data flow, provider architecture, SQLite schema, type system, API routes, dashboard pages |
| [docs/AGENTS.md](docs/AGENTS.md) | Agent suite, orchestrator role, development ceremony, team workflow, triple-layer code review, document ownership |
| [docs/DEVELOPMENT.md](docs/DEVELOPMENT.md) | Branch discipline, hookify rules, pre-action verification, version bump, configuration, dev notes |
| [docs/PRODUCT.md](docs/PRODUCT.md) | Product description, features, source tools, insight categories, export, reflect/patterns |
| [docs/VISION.md](docs/VISION.md) | Philosophy, core beliefs, phase history, non-goals |
| [docs/ROADMAP.md](docs/ROADMAP.md) | Phase milestones, version table, upcoming work |

---

## Supported Source Tools

| Source Tool | Provider ID | Provider Class | Data Format | Location |
|-------------|-------------|---------------|-------------|----------|
| Claude Code | `claude-code` | `ClaudeCodeProvider` | JSONL | `~/.claude/projects/**/*.jsonl` |
| Cursor | `cursor` | `CursorProvider` | SQLite (state.vscdb) | Platform-specific |
| Codex CLI | `codex-cli` | `CodexProvider` | JSONL (rollout files) | `~/.codex/sessions/YYYY/MM/DD/rollout-*.jsonl` |
| Copilot CLI | `copilot-cli` | `CopilotCliProvider` | JSONL (events) | `~/.copilot/session-state/{id}/events.jsonl` |
| VS Code Copilot Chat | `copilot` | `CopilotProvider` | JSON | Platform-specific Copilot Chat storage |

---

## Commands

```bash
cd cli
pnpm install          # Install dependencies
pnpm dev              # Watch mode (tsc --watch)
pnpm build            # Compile TypeScript to dist/

# After building, link for local testing:
npm link
code-insights                          # Sync + open dashboard (zero-config)
code-insights init                     # Optional: customize settings
code-insights sync                     # Sync sessions to SQLite
code-insights sync --force             # Re-sync all sessions
code-insights sync --dry-run           # Preview without changes
code-insights sync -q                  # Quiet mode (for hook usage)
code-insights sync --source cursor     # Sync only from a specific tool
code-insights status                   # Show sync statistics
code-insights open                     # Open dashboard in browser (no server start)
code-insights dashboard                # Start server + open dashboard (auto-syncs first)
code-insights dashboard --no-sync      # Start server + open dashboard without syncing
code-insights install-hook             # Auto-sync + auto-analysis on session end
code-insights install-hook --sync-only # Install sync hook only (no analysis)
code-insights uninstall-hook           # Remove all Code Insights hooks
code-insights config                   # Show current configuration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [melagiri/code-insights](https://github.com/melagiri/code-insights) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
