---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Development

```bash
pnpm install                         # Install all deps
pnpm build                           # Build both packages (core → react)
pnpm build:core                      # Build only @acp-components/core
pnpm build:react                     # Build only @acp-components/react
pnpm lint                            # Lint all TypeScript in packages/*/src
```

### Web Demo

```bash
pnpm dev:server                      # Terminal 1 — WebSocket ↔ stdio bridge
pnpm dev                             # Terminal 2 — Vite dev server at localhost:5173
pnpm dev:tauri                       # Tauri desktop dev (stdio transport)
pnpm build:tauri                     # Tauri production build
```

### Running a single test

```bash
pnpm --filter @acp-components/core test -- -t "test name pattern"
pnpm --filter @acp-components/react test -- -t "test name pattern"
```

### Bridge server env vars

| Variable | Default | Description |
|----------|---------|-------------|
| `ACP_PORT` | `3100` | WebSocket port |
| `ACP_HOST` | `127.0.0.1` | WebSocket host |
| `ACP_AGENT` | `opencode` | Agent binary |
| `ACP_AGENT_ARGS` | `acp` | Agent arguments |

## Architecture

### Package Layering

```
Application Layer (Vite Demo / Tauri / Custom Apps)
  └─ @acp-components/react (UI Layer)
       Components (15+) + Hooks (useSyncExternalStore) + AcpContext + PlatformContext + Theme + i18n
       depends on ↓
     @acp-components/core (Data Layer)
       createAcpProvider (multi-agent lifecycle) + AcpClient (per-agent, wraps ACP SDK)
       + Transport (Stdio/WebSocket/HTTP/Custom) + acpStore + sessionStore + fileTreeStore (vanilla Zustand)
       + Actions (sessions/prompt/permission/fileTree/extensions, agent-aware)
       built on ↓
     @agentclientprotocol/sdk (ACP protocol types + ClientSideConnection)

Platform Layer (orthogonal to the above): a `Platform` interface (defined in `@acp-components/react`) + `PlatformContext`/`usePlatform()` provide host-native capabilities to the UI. The interface is **sharded** into cohesive slices — `fs?` (readDirectory/readFileContent/writeFileContent?/watchFileTree?), `dialogs?` (openLink/openFilePicker/notify), `storage` (always required), `openExternalEditor?` (host takes over file opening, bypassing built-in FileViewer), `updater?`, `system?` (restart?/exportLogs?) — capability is expressed by slice / method presence; callers guard with `?.` at the use site. Each host (web demo, Tauri template) implements its own `Platform` (`createWebPlatform` / `createTauriPlatform`). Workspace load/save is NOT on `Platform` — it lives in the `useWorkspacesPersistence` hook, backed by `storage('workspaces')`. core does NOT implement `Platform` — it only owns the shared primitive types (`PlatformKind`, `PlatformStorage`, `UpdaterState`, …).
```

**Critical rules**:
- `@acp-components/core` has zero React dependency. It uses vanilla Zustand stores. React layer subscribes via `useSyncExternalStore`. Never add React imports to core.
- `AcpClient`'s client-side callbacks are **only** `sessionUpdate` / `requestPermission` / `extMethod` / `extNotification`. File access is a UI-side capability consumed via `usePlatform()`.
- `Platform` and `AcpContext` are orthogonal: `Platform`/`usePlatform()` for native capabilities, `AcpContext`/`useAcpContext()` for agent connection/session state. Agent transport is configured via `AgentConfig.transport` on `AcpProvider` and is **not** part of `Platform`.

### Multi-Agent & Multi-Workspace State Model

```
acpStore (global):
  agents: Map<agentId, AgentConnection>        — per-agent status, info, capabilities, authMethods
  workspaces: Map<cwd, WorkspaceState>          — per-workspace sessions
  activeSessionId: SessionId | null             — global active session (workspace derived via SessionMeta.cwd)
  pendingAuth: { agentId } | null               — pending auth agent

sessionStore (per-session, keyed by SessionId):
  messages[], isStreaming, pendingToolCalls (Map),
  pendingPermissions[], plan[], usage, configOptions[], availableCommands[]

fileTreeStore (per-workspace):
  per-cwd file-tree state (nodes / expanded / loaded / error); directory reader is injected from Platform.readDirectory
```

- **Agent** = independent ACP connection (transport + status + capabilities). Connected in parallel.
- **Workspace** = directory (cwd). Holds sessions from multiple agents.
- **Session** = belongs to workspace + agent pair (`SessionMeta.agentId` + `SessionMeta.cwd`).
- **Active Workspace** = derived from `activeSessionId` by looking up `SessionMeta.cwd` in workspaces.

### Data Flow (Unidirectional)

```
Agent → NDJSON stream → Transport.readable
  → AcpClient.onSessionUpdate (per agent)
    → createAcpProvider dispatches sessionUpdate type to store actions
      → acpStore / sessionStore (Zustand vanilla)
        → useSyncExternalStore → React Hooks → Components re-render
          → user action → Actions → AcpClient.prompt/cancel → Transport.writable → Agent
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zvzuola/acp-components](https://github.com/zvzuola/acp-components) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
