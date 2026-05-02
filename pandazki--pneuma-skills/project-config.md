---
trigger: always_on
description: Pneuma Skills is co-creation infrastructure for humans and code agents. The underlying bet: **coding agents already do the actual work against a directory of files; the system's job is to make human observation and optional participation intuitive**. Agents edit files directly through their native tools (Read/Edit/Write) — files remain the canonical collaboration surface and are not abstracted away. Viewers are live **players** for agent output, rendering the work in domain terms (a deck, a boar
---

# Pneuma Skills

## Project Overview

Pneuma Skills is co-creation infrastructure for humans and code agents. The underlying bet: **coding agents already do the actual work against a directory of files; the system's job is to make human observation and optional participation intuitive**. Agents edit files directly through their native tools (Read/Edit/Write) — files remain the canonical collaboration surface and are not abstracted away. Viewers are live **players** for agent output, rendering the work in domain terms (a deck, a board, a project) so humans can watch what's happening, make direct decisions in the UI when needed, and reach for structured command suggestions when deeper guidance helps. Four pillars for isomorphic collaboration: a **visual environment** (live players for agent work with optional human participation), **skills** (domain knowledge + seed templates + session persistence), **continuous learning** (evolution agent for cross-session preference extraction and skill augmentation), and **distribution** (mode marketplace, publishing, sharing). The runtime supports multiple agent backends (Claude Code, Codex) selected at startup.

**Formula:** `ModeManifest(skill + viewer + agent_config) × AgentBackend × RuntimeShell`

**Version:** 2.30.1
**Runtime:** Bun >= 1.3.5 (required, not Node.js)
**Builtin Modes:** `webcraft`, `doc`, `slide`, `draw`, `diagram`, `illustrate`, `remotion`, `gridboard`, `mode-maker`, `evolve`

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
| Agent | Claude Code CLI via `--sdk-url`; Codex CLI via `app-server` stdio JSON-RPC (`node:child_process`) |

## CLI Commands

```bash
# Development
bun run dev              # Launcher UI (no mode arg)
bun run dev doc          # Doc Mode (cwd as workspace)
bun run dev slide        # Slide Mode
bun run dev doc --workspace ~/notes --port 17996 --backend claude-code --no-open --debug
bun run build            # Vite production build
bun test                 # All tests (bun:test)

# Skill evolution
pneuma evolve <mode>     # Launch evolution agent for a mode's skill

# Mode management
pneuma mode add <url>    # Install remote mode to ~/.pneuma/modes/
pneuma mode list         # List published modes on R2
pneuma mode publish      # Publish current workspace as mode

# Plugin management
pneuma plugin add <source>   # Install plugin from path/github/URL to ~/.pneuma/plugins/
pneuma plugin list           # List builtin + external plugins with enabled status
pneuma plugin remove <name>  # Remove an external plugin

# Snapshot
pneuma snapshot push     # Upload workspace to R2
pneuma snapshot pull     # Download workspace from R2

# History sharing & replay
pneuma history export [--output FILE]  # Export session as shareable .tar.gz
pneuma history share [--title NAME]    # Export + upload to R2, return link
pneuma history open <path-or-url>      # Download/prepare replay package
```

### CLI Flags

| Flag | Description |
|------|-------------|
| `--workspace <path>` | Workspace directory (default: cwd) |
| `--port <n>` | Server port (default: auto) |
| `--backend <type>` | Select backend at startup (`claude-code` or `codex`; session stays fixed to it) |
| `--no-open` | Don't open browser |
| `--no-prompt` | Non-interactive mode (launcher uses this) |
| `--skip-skill` | Skip skill installation (session resume without update) |
| `--debug` | Enable debug mode |
| `--dev` | Force dev mode (Vite) |
| `--replay <path>` | Load a replay package on startup (enters replay mode) |
| `--replay-source <path>` | Source workspace for existing session replay (exports + replays) |
| `--session-name <name>` | Custom session display name (default: `{mode}-{timeTag}`) |
| `--viewing` | Start in viewing mode (`editing: false` — skip skill install + agent spawn) |

## Ports

- **17996** — default Vite dev server / production server
- **17007** — default Hono backend in dev mode
- Dev: browser → Vite, WebSocket → backend directly (`Vite` WS proxy is bypassed)
- Launcher child sessions auto-increment both ports when the defaults are occupied
- Both servers bind `hostname: "0.0.0.0"` to avoid IPv4/IPv6 dual-stack port collision

## Project Structure

```
pneuma-skills/
├── bin/                       # CLI entry — mode resolution, agent launch, session registry
├── core/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pandazki/pneuma-skills](https://github.com/pandazki/pneuma-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
