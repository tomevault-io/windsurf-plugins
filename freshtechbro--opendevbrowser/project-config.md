---
trigger: always_on
description: **Generated:** 2026-04-14 | **Commit:** d7d579f | **Branch:** codex/provider-guidance-live
---

# OpenDevBrowser - Agent Guidelines

**Generated:** 2026-04-14 | **Commit:** d7d579f | **Branch:** codex/provider-guidance-live

## Overview

Agent-agnostic browser automation runtime for CLI workflows, OpenCode tool calls, and Chrome extension relay sessions. Script-first UX: snapshot → refs → actions.

### Install Quick Reference

- Runtime: Node.js `>=18`.
- Recommended installer: `npx opendevbrowser`
- Optional persistent CLI: `npm install -g opendevbrowser`

## Architecture

```text
┌─────────────────────────────────────────────────────────────────┐
│                      Distribution Layer                         │
├──────────────────┬──────────────────┬──────────────────┬──────────────────────────┤
│ OpenCode Tools   │       CLI        │    Hub Daemon    │    Chrome Extension       │
│  (src/index.ts)  │ (src/cli/index)  │ (opendevbrowser  │   (extension/src/)        │
│                  │                  │      serve)     │                           │
└────────┬─────────┴────────┬─────────┴─────────┬────────┴──────────────┬────────────┘
         │                  │                  │                       │
         ▼                  ▼                  ▼                       ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Core Runtime (src/core/)                    │
│  bootstrap.ts → wires managers, sibling desktop runtime,       │
│                   automation coordinator, injects ToolDeps     │
└────────┬────────────────────────────────────────────────────────┘
         │
    ┌────┴────┬─────────────┬──────────────┬──────────┬────────────┬────────────┬────────────┐
    ▼         ▼             ▼              ▼          ▼            ▼            ▼
┌────────┐ ┌────────┐ ┌──────────┐ ┌────────┐ ┌────────────┐ ┌────────────┐ ┌────────────┐
│Browser │ │Script  │ │Snapshot  │ │ Canvas │ │ Annotation │ │  Relay     │ │  Skills    │
│Manager │ │Runner  │ │Pipeline  │ │Manager │ │  Manager   │ │  Server    │ │  Loader    │
└───┬────┘ └────────┘ └──────────┘ └────┬───┘ └─────┬──────┘ └─────┬──────┘ └────────────┘
    │                                   │            │
    ▼                                   ▼            ▼
┌────────┐                         ┌──────────┐ ┌────────────┐
│Target  │                         │AgentInbox│ │ Extension  │
│Manager │                         │          │ │ (WS relay) │
└────────┘                         └──────────┘ └────────────┘
```

### Data Flow

```
Tool Call → Zod Validation → Manager/Runner → CDP/Playwright → Response
                                   ↓
                            Snapshot (AX-tree → refs)
                                   ↓
                            Action (ref → backendNodeId → DOM)
```

### System Workflow (Happy Path)

1. `launch` (extension or managed) → sessionId
2. `snapshot` → refs
3. Action tools (`click`, `type`, `press`, `hover`, `check`, etc.) → repeat snapshot
4. `disconnect` on completion

### Session Modes

| Mode | Entry | Use Case |
|------|-------|----------|
| `extension` | `opendevbrowser_launch` (default) | Attach to logged-in tabs via relay |
| `managed` | `--no-extension` | Fresh Playwright-controlled Chrome |
| `cdpConnect` | `opendevbrowser_connect` | Attach to existing `--remote-debugging-port` |

Extension relay requires **Chrome 125+** and uses flat CDP sessions with DebuggerSession `sessionId` routing. Annotation relay uses a dedicated `/annotation` websocket channel, and design-canvas relay uses a dedicated `/canvas` websocket channel. When hub mode is enabled, the hub daemon is the sole relay owner and enforces FIFO leases (no local relay fallback).

### Connection Flags & Status Semantics

- `--no-extension`: Force managed mode (ignores relay). `--headless` also implies managed mode.
- `--extension-only`: Fail unless extension is connected/handshaken.
- `--wait-for-extension`: Polls for extension handshake up to `--wait-timeout-ms` (min 3s).
- `extensionConnected`: Extension WebSocket is connected to relay.
- `extensionHandshakeComplete`: Extension handshake finished (preferred readiness signal).
- `annotationConnected`: Annotation relay client attached.
- `opsConnected`: At least one active `/ops` client.
- `canvasConnected`: At least one active `/canvas` client.
- `cdpConnected`: At least one active `/cdp` client; false is normal until a tool/CLI connects.
- `pairingRequired`: Relay requires pairing token; extension auto-pair should handle this.

## Structure

```
.
├── src/              # Runtime implementation
│   ├── annotate/     # Annotation transports + output shaping
│   ├── automation/    # Automation helpers and coordinator
│   ├── browser/      # Browser sessions, target orchestration, canvas preview/code-sync
│   ├── cache/        # Chrome executable resolution
│   ├── canvas/       # Design-canvas document store, repo IO, code-sync, export helpers
│   ├── challenges/   # Bounded challenge orchestration plane, evidence, recovery lanes
│   ├── cli/          # CLI commands, daemon, installers
│   ├── core/         # Bootstrap, runtime wiring, ToolDeps
│   ├── desktop/      # Read-only desktop observation runtime; `desktop.*` config; see `desktop/AGENTS.md`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [freshtechbro/opendevbrowser](https://github.com/freshtechbro/opendevbrowser) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
