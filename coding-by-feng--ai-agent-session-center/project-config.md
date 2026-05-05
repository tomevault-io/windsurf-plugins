---
trigger: always_on
description: Localhost dashboard (port 3333) that monitors AI coding agent sessions (Claude Code, Gemini CLI, Codex) via hooks. Sessions are visualized as 3D robot characters in an interactive cyberdrome. Supports SSH terminals, team/subagent tracking, prompt queuing, workspace snapshots, and session resume.
---

# AI Agent Session Center

Localhost dashboard (port 3333) that monitors AI coding agent sessions (Claude Code, Gemini CLI, Codex) via hooks. Sessions are visualized as 3D robot characters in an interactive cyberdrome. Supports SSH terminals, team/subagent tracking, prompt queuing, workspace snapshots, and session resume.

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Backend | Node.js 18+ (ESM), Express 5, ws 8, tsx |
| Frontend | React 19, Three.js / @react-three/fiber, Zustand 5, Vite 7 |
| Desktop | Electron 34, electron-builder 25 |
| Terminal | node-pty (IPC in Electron, WebSocket in browser) |
| Hooks | Bash script → JSONL file-based MQ (HTTP POST fallback) |
| Persistence | SQLite / better-sqlite3 (server) + IndexedDB / Dexie (browser) |

## Commands

```bash
npm run dev              # Vite + tsx watch (HMR)
npm run build            # Production build
npm start                # Start production server
npm test                 # Vitest
npm run test:e2e         # Playwright E2E
npm run test:coverage    # Coverage report
npm run typecheck        # tsc --noEmit
npm run lint             # ESLint src/
npm run format           # Prettier
npm run electron:dev     # Build + launch Electron app
npm run electron:build   # Build distributable (DMG/NSIS)
npm run install-hooks    # Install hooks into CLI settings
npm run uninstall-hooks  # Remove all dashboard hooks
npm run setup            # Interactive setup wizard
npm run reset            # Remove hooks, clean config, backup
```

## Feature Documentation

**All feature logic is documented in `docs/feature/`.** Each doc covers function, purpose, source files, implementation, cross-feature dependencies, and change risks.

**MANDATORY WORKFLOW — Before implementing any new feature or modifying an existing one:**
1. Read this CLAUDE.md to identify which feature domain is involved
2. Read the corresponding feature doc(s) in `docs/feature/` to understand the current implementation, source files, dependencies, and change risks
3. Check the Impact Matrix below to identify connected features that may be affected
4. Read those connected feature docs too, so you don't introduce regressions
5. After completing the work, update every feature doc that was affected by the change

| Domain | Docs | What they cover |
|--------|------|----------------|
| [`server/`](docs/feature/server/) | 12 | [Hook System](docs/feature/server/hook-system.md), [Session Management](docs/feature/server/session-management.md), [Session Matching](docs/feature/server/session-matching.md), [Approval Detection](docs/feature/server/approval-detection.md), [WebSocket](docs/feature/server/websocket-manager.md), [API](docs/feature/server/api-endpoints.md), [Database](docs/feature/server/database.md), [Terminal/SSH](docs/feature/server/terminal-ssh.md), [Teams](docs/feature/server/team-subagent.md), [Process Monitor](docs/feature/server/process-monitor.md), [Auth](docs/feature/server/authentication.md), [File Index Cache](docs/feature/server/file-index-cache.md) |
| [`frontend/`](docs/feature/frontend/) | 15 | [State](docs/feature/frontend/state-management.md), [Persistence](docs/feature/frontend/client-persistence.md), [WS Client](docs/feature/frontend/websocket-client.md), [Detail Panel](docs/feature/frontend/session-detail-panel.md), [File Browser](docs/feature/frontend/file-browser.md), [Terminal UI](docs/feature/frontend/terminal-ui.md), [Settings](docs/feature/frontend/settings-system.md), [Shortcuts](docs/feature/frontend/keyboard-shortcuts.md), [Queue](docs/feature/frontend/prompt-queue.md), [Views](docs/feature/frontend/views-routing.md), [Agenda](docs/feature/frontend/agenda.md), [Workspace Snapshot](docs/feature/frontend/workspace-snapshot.md), [Setup Wizard](docs/feature/frontend/setup-wizard.md), [Auth UI](docs/feature/frontend/auth-ui.md), [Project Browser](docs/feature/frontend/project-browser.md) |
| [`3d/`](docs/feature/3d/) | 3 | [Cyberdrome Scene](docs/feature/3d/cyberdrome-scene.md), [Robot System](docs/feature/3d/robot-system.md), [Particles/Effects](docs/feature/3d/particles-effects.md) |
| [`multimedia/`](docs/feature/multimedia/) | 2 | [Sound & Alarm System](docs/feature/multimedia/sound-alarm-system.md), [TTS Voice Output](docs/feature/multimedia/tts-voice-output.md) |
| [`electron/`](docs/feature/electron/) | 3 | [App Lifecycle](docs/feature/electron/app-lifecycle.md), [PTY Host](docs/feature/electron/pty-host.md), [IPC Transport](docs/feature/electron/ipc-transport.md) |

See [`docs/feature/README.md`](docs/feature/README.md) for the full index, dependency graph, and impact matrix.

## Architecture

### Data Flow

```
AI CLI (Claude/Gemini/Codex)
  → hooks/dashboard-hook.sh (jq enrichment, ~2-5ms)
  → /tmp/claude-session-center/queue.jsonl (atomic append ~0.1ms)
  → server/mqReader.ts (fs.watch + debounce)
  → server/hookProcessor.ts (validate + route)
  → server/sessionStore.ts (state machine + coordinator)
  → server/wsManager.ts (broadcast to browsers)
  → Browser (Zustand stores → React render)
Total: 3-17ms end-to-end

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [coding-by-feng/ai-agent-session-center](https://github.com/coding-by-feng/ai-agent-session-center) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
