---
trigger: always_on
description: This file provides guidance to Agent when working with code in this repository.
---

This file provides guidance to Agent when working with code in this repository.

## Build & Test Commands

```bash
bun run build          # Build CLI to ./dist (outputs cli.js and bridge/bridge-main.js)
npx tsc --noEmit       # Run TypeScript typecheck
npm test               # Run typecheck, then all unit tests (tests/unit/**/*.test.ts)
npm run test:unit      # Alias for above
npm run test:smoke     # Run smoke tests (tests/smoke/**/*.test.ts)
```

`transport.permissionMode` defaults to `approve-all` when omitted, so non-interactive prompt turns do not stop on acpx permission requests unless the user explicitly configures a stricter policy.

**Local daemon CLI (before publish):**
```bash
bun run dev            # Run console in foreground (dev mode)
bun run login          # Show QR code for WeChat login
node ./dist/cli.js start   # Start daemon in background
node ./dist/cli.js status  # Check daemon status
node ./dist/cli.js stop    # Stop daemon
```

**Local dry-run (no WeChat needed):**
```bash
bun run dry-run --chat-key wx:test -- "/session new demo --agent codex --ws backend" "/status"
```

## Architecture Overview

### Core Purpose
xacpx is a WeChat console that lets you remotely control `acpx` sessions. It bridges WeChat messages to agent sessions via `weixin-agent-sdk`.

### Key Modules

- **`src/cli.ts`** - Daemon CLI (`start`/`status`/`stop`/`run`/`login`). Not the main app entry.
- **`src/main.ts`** - `buildApp()` assembles the runtime; `resolveRuntimePaths()` finds config/state files.
- **`src/run-console.ts`** - Orchestrates app build, SDK start, heartbeat, and cleanup.
- **`src/console-agent.ts`** - Bridges WeChat messages to the command router.
- **`src/commands/command-router.ts`** - Routes WeChat commands (`/agent add`, `/session new`, etc.) to handlers.
- **`src/commands/parse-command.ts`** - Parses slash commands into typed structures.

### Transport Layer (src/transport/)

Two transport implementations share the `SessionTransport` interface:

- **`acpx-cli`** - Spawns `acpx` directly as a child process. Uses `node-pty` for PTY allocation.
- **`acpx-bridge`** - Runs `acpx` in a separate bridge subprocess (`src/bridge/bridge-main.ts`). Uses stdin/stdout JSON protocol.

Both transports expose: `ensureSession`, `prompt`, `setMode`, `cancel`, `hasSession`.

### Session Model

There are two session concepts:

1. **Logical session** (managed by `SessionService`) - tracks alias, agent, workspace, and chat context per user.
2. **Transport session** - the actual `acpx` named session on the backend.

`/session new` creates both. `/session attach` only creates the logical session and binds to an existing transport session.

### Config & State

- Config (`~/.xacpx/config.json`) - transport, agents, workspaces. Written via `ConfigStore`.
- State (`~/.xacpx/state.json`) - sessions, chat contexts. Written via `StateStore`.

### Daemon Subsystem

- **src/daemon/daemon-controller.ts** — External daemon control surface (start/status/stop).
- **src/daemon/create-daemon-controller.ts** — Wires platform-specific spawn/terminate behavior into the controller.
- **src/daemon/daemon-runtime.ts** — Writes PID/status metadata and heartbeat from inside the daemon process.
- **src/daemon/daemon-status.ts** — Reads and writes status.json for daemon readiness/state inspection.
- **src/daemon/daemon-files.ts** — Resolves daemon runtime paths (pid, status, logs).

### Bridge Subsystem

- **`src/bridge/bridge-main.ts`** - Entry point for the bridge subprocess (handles acpx stdio).
- **`src/bridge/bridge-server.ts`** - Parses bridge protocol JSON lines and delegates to `BridgeRuntime`.
- **`src/bridge/bridge-runtime.ts`** - Wraps raw acpx commands (sessions new, prompt, cancel).

### Acpx Resolution (priority order)

1. `transport.command` in config (explicit override)
2. Bundled `acpx` in `node_modules`
3. `acpx` in shell `PATH`

### Test Layout

- `tests/unit/` - Mirror of `src/` structure, `*.test.ts` files. Run by default.
- `tests/smoke/` - Real-environment tests (real acpx, real WeChat). Not run by default.
- `tests/helpers/` - Shared test utilities.

## Onboarding Notes (where to look first)

### Mental model

- Treat xacpx as a bridge: **Channel runtime (built-in Weixin or plugin channel such as Feishu/Yuanbao) → Router (slash commands + prompt) → Session mapping → Transport (acpx)**.
- There are two sessions: **logical session** (xacpx-managed) vs **transport session** (acpx-managed). Most bugs are mismatches between the two.

### Start reading from entrypoints

- CLI surface: [`src/cli.ts`](src/cli.ts)
- Wiring/DI and runtime paths: [`src/main.ts`](src/main.ts)
- Main loop (startup/shutdown ordering): [`src/run-console.ts`](src/run-console.ts)
- Router and command boundaries: [`src/commands/command-router.ts`](src/commands/command-router.ts) + [`src/commands/parse-command.ts`](src/commands/parse-command.ts)
- Session state model: [`src/sessions/session-service.ts`](src/sessions/session-service.ts)
- Transport boundary: [`src/transport/types.ts`](src/transport/types.ts)

### When changing behavior, follow the boundaries


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gadzan/xacpx](https://github.com/gadzan/xacpx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
