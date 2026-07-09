---
trigger: always_on
description: > **Single source of truth for agent instructions.** Claude Code reads this via the one-line `@AGENTS.md` import in `CLAUDE.md`; Codex and Kimi read this file directly. Never duplicate content into `CLAUDE.md` — it must stay a single import line.
---

# Pneuma Skills

> **Single source of truth for agent instructions.** Claude Code reads this via the one-line `@AGENTS.md` import in `CLAUDE.md`; Codex and Kimi read this file directly. Never duplicate content into `CLAUDE.md` — it must stay a single import line.
>
> Per-domain constraints and gotchas live in `.claude/rules/` — **before editing files in a domain, read the matching rule file** (Claude Code auto-loads them by path; other agents must read them explicitly). Index in [Development Toolchain](#development-toolchain-claude).

## Project Overview

Pneuma Skills is co-creation infrastructure for humans and code agents. Agents edit files directly (Read/Edit/Write); files remain the canonical collaboration surface. Viewers are live **players** for agent output, rendering work in domain terms (a deck, a board, a project) so humans can watch, intervene in the UI, or hand structured guidance back. Four pillars: a **visual environment** (live players with optional participation), **skills** (domain knowledge + seed templates + session persistence), **continuous learning** (evolution agent for cross-session preference extraction), and **distribution** (mode marketplace, publishing, sharing). Multiple agent backends (Claude Code, Codex, Kimi CLI) selected at startup.

**Formula:** `ModeManifest(skill + viewer + agent_config) × AgentBackend × RuntimeShell`

**Version:** 3.24.0
**Runtime:** Bun >= 1.3.5 (required, not Node.js)
**Builtin Modes:** `webcraft`, `doc`, `slide`, `draw`, `diagram`, `illustrate`, `remotion`, `gridboard`, `kami`, `clipcraft`, `cosmos`, `wordtaste`, `mode-maker`, `evolve`, `project-evolve`, `project-onboard`, `project-tidy`

> Modes can set `hidden: true` to disappear from user-pickable lists (launcher grids, ProjectPanel mode-tile picker). Their sessions are also stamped `internal: true` by `scanProjectSessions` and filtered out of user-facing session lists (project panel, project cards, quick-resume). Internal modes (`evolve`, `project-evolve`, `project-onboard`, `project-tidy`) are hidden — triggered by specific UI affordances or programmatically, never by a "what mode to start?" choice.

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Runtime | Bun >= 1.3.5 |
| Server | Hono 4.7 |
| Frontend | React 19 + Vite 7 + Tailwind CSS 4 + Zustand 5 |
| Terminal | xterm.js 6 + Bun native PTY |
| File Watching | chokidar 5 |
| Drawing | @excalidraw/excalidraw 0.18 |
| Diagramming | draw.io viewer-static.min.js (CDN) + rough.js 4.6 |
| Video | remotion 4.0 + @remotion/player + @remotion/web-renderer + @babel/standalone |
| Desktop | Electron 41 + electron-builder + electron-updater |
| Agent | Claude Code CLI stdio stream-json; Codex CLI `app-server` stdio JSON-RPC; Moonshot Kimi CLI stdio stream-json (`kimi --print … -y`) — all via `node:child_process` |

## CLI Commands

```bash
# Development
bun run dev              # Launcher UI (no mode arg)
bun run dev doc          # Doc Mode (cwd as workspace)
bun run dev doc --workspace ~/notes --port 17996 --backend claude-code --no-open --debug
bun run build            # Vite production build
bun run typecheck        # tsc --noEmit
bun test                 # All tests (bun:test)

# Skill evolution
pneuma evolve <mode>

# Mode management
pneuma mode add <url>        # Install remote mode (single → ~/.pneuma/modes/; library → ~/.pneuma/libraries/)
pneuma mode list             # List published modes on R2
pneuma mode publish          # Publish workspace as mode

# Mode libraries (multi-mode GitHub repos)
pneuma library init <name> [--github user/repo] [--private]
pneuma library link <github:user/repo>           # Alias for `mode add`
pneuma library list
pneuma library sync <id>                         # Pull latest (git fetch + checkout)
pneuma library publish <mode> [--to id] [--as name] [--push]
pneuma library push <id>                         # `git push origin HEAD`
pneuma library activate|deactivate <id> <mode>
pneuma library unlink <id>                       # Remove library + clone

# Project recovery / plugins / snapshot / history
pneuma project add <path>                        # Register existing project into ~/.pneuma/sessions.json
pneuma plugin add|list|remove <source>           # Install to ~/.pneuma/plugins/
pneuma snapshot push|pull                        # R2 workspace snapshot
pneuma history export [--output FILE]            # Session as .tar.gz
pneuma history share [--title NAME]              # Export + upload to R2
pneuma history open <path-or-url>                # Prepare replay package

# Agent command distribution (3.10.0)
pneuma agent-command status [--backend claude-code|codex|all] [--json]
pneuma agent-command install [--backend claude-code|codex|all] [--force] [--json]
pneuma agent-command uninstall [--backend claude-code|codex|all] [--force] [--json]
pneuma agent-command update [--backend claude-code|codex|all] [--json]
pneuma mode list --local [--json]                # builtins + ~/.pneuma/modes + activated library modes
pneuma handoff-from-external --intent <text> --mode <name> [--cwd <path>] \

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pandazki/pneuma-skills](https://github.com/pandazki/pneuma-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
