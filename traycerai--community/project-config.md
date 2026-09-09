---
trigger: always_on
description: Electron shell around `@traycer-clients/gui-app`. Read with repo-root
---

# AGENTS.md — clients/desktop

Electron shell around `@traycer-clients/gui-app`. Read with repo-root
`AGENTS.md`.

## Role

1. Load the `gui-app` renderer.
2. Delegate host lifecycle to the **Traycer CLI** — Desktop never spawns the
   host. Discover WS URL from `~/.traycer/host[/dev]/pid.json`; tail
   `~/.traycer/host[/dev]/host.log`.
3. Expose `IRunnerHost` via `contextBridge` / `ipcMain.handle`.

Transport-agnostic: do **not** proxy host RPC. `gui-app` talks to the host's
localhost HTTP/WS after `LocalHostSnapshot`.

One scoped, sunsetted exception: the v1.2.0 `host.*` maintenance projections
in `host-management-ipc.ts` ("The maintenance-RPC projections") answer four
protocol response shapes from main for a LOCAL host too old to serve them
(negotiated away at handshake). Nothing is proxied — there is no host wire
surface to forward to; main is the _origin_, because only main can shell the
bundled CLI and read the on-disk install records the answers come from. The
block comment there carries the full rationale; delete the lane when the
fleet floor reaches 1.2.0.

A second, permanent exception: the **`browser.sessions` stream is main's**
(browser-security-hardening H10). Main opens it, answers every cookie-bearing
frame on it, and forwards only the opaque UX projection to the renderer over
`browserViewSessions*`. This is not transport proxying for its own sake - it is
the whole point. That stream carries the master cookie jar: capture answers,
seeded storage state, the store-key handshake, the forget ledger and its ack.
A renderer that can read those frames is a fully trusted principal over every
login on the machine, which is what root cause C of
`specs/browser-security-review.md` found it to be; after H10 no cookie value
exists in a renderer process at all. `gui-app` still talks to the host directly
for everything else, including every other browser RPC.

## Commands

```bash
# from clients/desktop/
bun run dev           # shell + renderer only
bun run compile       # type-check (no emit)
bun run build         # main + renderer (no package)
bun run package       # electron-builder
bun run package:dir   # unpacked smoke build
bun run test

# end-to-end (repo root, macOS/Linux) — production cloud + released host
make dev-desktop
make dev-desktop VERSION=1.2.3
```

Details: [`docs/DEVELOPMENT.md`](../../docs/DEVELOPMENT.md).

**Commits:** don't manually run `compile` / `build` / `lint` / `format` before
committing — repo-root `pre-commit` already runs the affected checks (see root
`AGENTS.md`). Tests are CI, not the hook. Re-run checks only when diagnosing
failures.

## Layout

| Path                    | Role                                                                                                                   |
| ----------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| `src/electron-main/`    | Main process (`main-process.ts`); feature folders under `app/`, `auth/`, `host/`, `windows/`, `menu/`, `tray/`, `ipc/` |
| `src/electron-preload/` | `window.runnerHost` bridges (`preload-bridge.ts` + `*-bridge.ts`)                                                      |
| `src/renderer-shell/`   | Thin React host; UI is `gui-app` via vite aliases                                                                      |
| `src/ipc-contracts/`    | Plain-data types shared across main/preload/renderer                                                                   |
| `scripts/`              | `dev/`, `prepack/`, `assets/`                                                                                          |
| `resources/cli/`        | Staged CLI SEA (`<platform>-<arch>/traycer`) → `process.resourcesPath/cli/`                                            |
| `resources/tray/`       | Tray icons (regenerate via `scripts/assets/generate-tray-icons.cjs`)                                                   |
| `resources/host/`       | Placeholder only — **never** ship a host binary here                                                                   |

## Invariants

- Bundle CLI only, not the host. Host lives under `~/.traycer/host/` via CLI.
- Main entry: `dist/main/index.js`.
- Preload stays CommonJS and imports only from `src/ipc-contracts/` — with one
  governed exception, `electron-preload/selection-authority-bridge.ts`, which
  also imports the selection-authority **contract parsers** and the
  **buffered/rotating client** from `clients/shared/host-selection/`. Two
  reasons, and a new exception needs both: (1) the attach/rotate choreography is
  identical for the Electron and browser/dev bindings, so a preload-local copy
  is a second implementation of a protocol that must not drift; (2) parsing at
  this hop is what makes same-major skew safety structural — renderer domain
  code never sees an unparsed envelope. The preload is esbuild-bundled to one
  CommonJS file, so a shared import is inlined and the CommonJS half of the rule
  is unaffected. Everything else still crosses as plain wire types through
  `src/ipc-contracts/`; do not read this as licence to move feature logic into
  preload.
- **Keyboard input while a browser guest has focus is ONE declarative

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [traycerai/community](https://github.com/traycerai/community) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
