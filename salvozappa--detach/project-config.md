---
trigger: always_on
description: This is a mobile-first "asynchronous coding" app with 4 panels: Agent (terminal with CLI AI assistant running), Explore (file browser (read only)), Terminal (standard bash shell), Git (version control UI to commit / pull / push).
---

# Detach - Context Reference

## Project Vision
This is a mobile-first "asynchronous coding" app with 4 panels: Agent (terminal with CLI AI assistant running), Explore (file browser (read only)), Terminal (standard bash shell), Git (version control UI to commit / pull / push).

**Core workflow:**
1. Queue up coding tasks from your phone, in the Agent view
2. AI agent executes in cloud sandbox
3. Get push notification when complete
4. Review changes in mobile-optimized UI
5. Approve/reject/request changes

See `docs/concept.md` for full vision and business model.

## Current Implementation
Web-based terminal + Git UI prototype. Four-panel interface:

### Bottom Navigation Panels
1. **Agent** - Terminal with Claude Code (or another CLI assistant) running in the sandbox
2. **Explore** - File browser for viewing project files with syntax highlighting
3. **Terminal** - Standard bash shell for running applications, tests, and commands
4. **Git** - Git status viewer for reviewing and committing changes

Users can connect via browser (PWA) to interact with a remote sandbox environment.

## Architecture
- **Frontend (webview/)**: HTML/CSS/JS served via nginx
- **Backend (bridge/)**: Go WebSocket server that bridges browser ↔ SSH sandbox
- **Sandbox**: Ubuntu container with SSH, dev tools, git

## Authentication

Detach uses token-based authentication for secure device pairing. Tokens are auto-generated on first startup and validated on WebSocket connections.

See [docs/authentication.md](docs/authentication.md) for complete authentication specification including token generation, storage, WebSocket handshake, and security considerations.

## UI Features by Panel

### Agent Panel
- xterm.js terminal connected to sandbox
- Mobile keyboard toolbar (Esc, arrows, Enter)
- Session persistence (reconnects to same shell)
- Visual viewport handling for mobile keyboard

### Explore Panel
- File explorer with folder navigation
- Syntax-highlighted code viewer (powered by highlight.js)
- Touch-friendly file browsing

### Git Panel
- Accordion sections: Unstaged Changes / Staged Changes
- Inline diff viewer with syntax highlighting
- Actions: Stage, Unstage, Discard (double-tap confirm)
- Commit button (enabled when changes staged)
- Untracked files: Clean syntax highlighting, no diff indicators
- Tracked files: Traditional diff view with +/- and red/green backgrounds

### Terminal Panel
- Standard bash shell for direct command execution
- Runs independently from Agent terminal
- Full xterm.js terminal with keyboard toolbar support
- Session persistence (reconnects to same shell)
- Ideal for running apps, tests, build commands, etc.


## Key Files

### Frontend

The frontend is written in TypeScript and built via esbuild. Source lives in `webview/src/`.

**Module Structure:**
```
webview/src/
├── app.ts              # Entry point, view orchestration, event wiring
├── connection.ts       # WebSocket lifecycle, reconnection, health checks
├── types.ts            # Type definitions, constants, configuration
├── utils.ts            # Logging infrastructure, utility functions
├── files.ts            # File operations (request/cache/notify)
├── git.ts              # Git operations (request/cache/notify)
└── ui/                 # UI layer (presentation components)
    ├── terminal.ts     # xterm.js wrapper, keyboard toolbar
    ├── code-view.ts    # File browser, syntax-highlighted viewer
    ├── git-view.ts     # Git status UI, diff viewer, staging actions
    ├── toast.ts        # Toast notifications
    └── status.ts       # Connection status display
```

**Architecture Pattern:**
- **UI layer (`ui/`)**: Presentation components that own DOM state and rendering
- **Business logic layer (root)**: Connection, file ops, git ops - no direct DOM manipulation
- **Orchestration (`app.ts`)**: Wires modules together, handles cross-cutting events

**Other Frontend Files:**
- `webview/index.html` - Main HTML structure
- `webview/styles.css` - All styling including git diff display
- `webview/dist/bundle.js` - Built output (generated)

### Backend
- `bridge/server.go` - Main WebSocket server
- `bridge/bridge.go` - Connection handling
- `bridge/session.go` - SSH session management
- `bridge/terminal.go` - PTY/terminal handling
- `bridge/git.go` - Git operations (status, stage, unstage, discard)
- `bridge/types.go` - Go struct definitions for WebSocket messages
- `bridge/Dockerfile` - Multi-stage build (copies all *.go files)

## Build

### Frontend (webview)
TypeScript source in `webview/src/` is compiled to `webview/dist/bundle.js` via esbuild:
```bash
cd webview && npm run build
```

### Backend (bridge)
Both the webview and bridge containers require rebuilding after code changes.

```bash
# Rebuild both containers
docker-compose build --no-cache bridge webview
docker-compose up -d

# Or individually
docker-compose build --no-cache bridge
docker-compose up -d bridge

docker-compose build --no-cache webview
docker-compose up -d webview
```

## WebSocket Protocol

### Terminal Messages
```javascript
// Terminal input/output with routing
{
  type: 'terminal_data',

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [salvozappa/detach](https://github.com/salvozappa/detach) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
