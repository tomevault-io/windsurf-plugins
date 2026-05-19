---
trigger: always_on
description: Coda VM provisioning system and terminal integration with Pathfinder.
---


# Coda integration

> For full reference, see [`docs/developer/CODA.md`](../../docs/developer/CODA.md).

## Overview

**Coda** is a separate backend service (`grafana-coda-app`) that provisions ephemeral 30-minute VMs on AWS. Pathfinder's Go backend is the sole consumer of Coda's REST API — the React frontend never calls Coda directly.

```
Frontend (React / xterm.js)
    │  Grafana Live WebSocket (bidirectional)
    ↓
Backend Plugin (Go)
    ├─ REST API ──→ Coda Server  (VM CRUD, sample apps, auth)
    └─ WebSocket ─→ Relay ─→ SSH ─→ EC2 VM
```

Coda itself has four components: **Server** (Node.js/Express/PostgreSQL), **Job Manager** (K8s webhook service), **Runner** (Terraform/Jsonnet provisioning), and **Relay** (Go WebSocket-to-TCP SSH proxy).

## End-to-end connection flow

1. User clicks "Connect" or a guide's terminal-connect button.
2. `useTerminalLive.connect(vmOpts?)` subscribes to a Grafana Live channel.
3. Backend `RunStream` calls `resolveVMForUser` (cache → ListVMs → CreateVM).
4. Backend polls `GetVM` every 3 s until `active` (up to 60 attempts).
5. Backend opens WebSocket to Relay at `wss://{relayURL}/relay/{vmID}`.
6. Relay proxies WebSocket bytes to the VM's SSH port (TCP).
7. Backend performs SSH handshake over the `WSConn` adapter, opens a PTY session.
8. SSH stdout/stderr → `sender.SendFrame` → frontend `terminal.write()`.
9. Frontend keystrokes → `PublishStream` → `session.Write()` → SSH stdin.

## Grafana Live stream path

```
terminal/{vmId}/{nonce}                                → default (vm-aws)
terminal/{vmId}/{nonce}/{template}                     → custom template
terminal/{vmId}/{nonce}/{template}/{app}               → custom template + app (sample-app)
terminal/{vmId}/{nonce}/vm-aws-alloy-scenario/{id}     → alloy scenario (id may contain slashes)
```

`vmId` is `"new"` on first connect; backend resolves the real VM. For `vm-aws-alloy-scenario`, all remaining path segments are joined as the scenario ID.

## VM resolution (`resolveVMForUser`)

Priority: in-memory cache → `FindActiveVMForUser` (ListVMs) → quota cleanup if needed → `CreateVM`.

Reuse is scoped by **template + app/scenario**:

- Same template, same app/scenario → reuse existing VM.
- Same template, different app/scenario → **destroy** old VM, create new.
- Different template → skip old, create new.

Quota: max 3 non-terminal VMs per user (`maxUserVMs`). When quota is full, `cleanupUserVMsForQuota` force-destroys all stale usable VMs and polls until count drops before retrying.

## VM state machine

```
pending → provisioning → active → destroying → destroyed
    │           │           │
    └───────────┴───────────┴──→ error
```

Additional state: `pooled` (pre-provisioned in hot pool, `vm-aws` only).

Usable states: `active`, `pending`, `provisioning`.
Terminal states: `destroyed`, `destroying`, `error`.

## Key files

### Backend (Go)

| File | Purpose |
|------|---------|
| `pkg/plugin/coda.go` | `CodaClient` — REST calls to Coda (CreateVM, GetVM, DeleteVM, ListVMs, ListSampleApps, ListAlloyScenarios), JWT auth refresh |
| `pkg/plugin/stream.go` | `RunStream`, `resolveVMForUser`, `waitForVMActive`, `vmRequestOpts`, heartbeat, VM expiry poll |
| `pkg/plugin/terminal.go` | `ConnectSSHViaRelay`, `NewTerminalSessionWithClient`, PTY management, SSH retry logic |
| `pkg/plugin/wsconn.go` | `WSConn` — `net.Conn` adapter over WebSocket for SSH transport |
| `pkg/plugin/resources.go` | HTTP handlers: `/coda/register`, `/vms`, `/vms/{id}`, `/sample-apps`, `/alloy-scenarios`, `/health` |
| `pkg/plugin/app.go` | Plugin lifecycle, `CodaClient` creation from settings, `streamSessions` map |
| `pkg/plugin/settings.go` | Plugin settings: `CodaRegistered`, `CodaAPIURL`, `CodaRelayURL`, secure `RefreshToken`/`EnrollmentKey` |

### Frontend (TypeScript)

| File | Purpose |
|------|---------|
| `src/integrations/coda/TerminalContext.tsx` | Shared React context: `connect(vmOpts?)`, `openTerminal(vmOpts?)`, `sendCommand`, module-level status |
| `src/integrations/coda/useTerminalLive.hook.ts` | Grafana Live subscription, `connect(vmOpts?)` with `TerminalVMOptions` (`template`/`app`/`scenario`), animated provision progress bar, handshake timeout |
| `src/integrations/coda/TerminalPanel.tsx` | xterm.js panel: fit/resize, scrollback persistence, WebGL renderer, auto-reconnect |
| `src/integrations/coda/terminal-storage.ts` | localStorage/sessionStorage keys for panel state, scrollback, and last VM opts (template/app/scenario for auto-reconnect) |
| `src/components/interactive-tutorial/terminal-connect-step.tsx` | "Try in terminal" button in guides, accepts `vmTemplate`/`vmApp`/`vmScenario` props |
| `src/components/block-editor/forms/TerminalConnectBlockForm.tsx` | Guide authoring form for terminal-connect blocks; uses `useCodaOptions` hook to fetch sample apps or alloy scenarios |
| `src/types/json-guide.types.ts` | `JsonTerminalConnectBlock` type with `vmTemplate`/`vmApp`/`vmScenario` fields |
| `src/types/json-guide.schema.ts` | Zod schema for terminal-connect blocks |

## Configuration

### Plugin jsonData (public)

| Key | Default | Description |
|-----|---------|-------------|
| `enableCodaTerminal` | `false` | Feature gate for terminal UI and block palette |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [grafana/grafana-pathfinder-app](https://github.com/grafana/grafana-pathfinder-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
