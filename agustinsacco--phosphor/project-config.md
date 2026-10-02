---
trigger: always_on
description: Phosphor is an Electron desktop app that wraps the **pi coding agent**
---

# CLAUDE.md

Phosphor is an Electron desktop app that wraps the **pi coding agent**
(`@earendil-works/pi-coding-agent`) — one `pi --mode rpc` subprocess per live
session, spoken to over JSONL on stdio. Phosphor never imports pi's code; the
protocol is hand-mirrored in `shared/rpc.ts`.

**Two maps before you start.** [README.md](README.md#repo-layout) has the repo
tree — the single copy, since three copies drifted.
[docs/README.md](docs/README.md) is the documentation index. Everything in
`docs/` is present tense: how Phosphor behaves **now**, one living contract per
surface. The single exception is
[docs/known-issues.md](docs/known-issues.md), which is defects that reproduce
today. There is no history folder and no specs folder — git is the history, and
a doc is updated in the same diff as the behaviour it describes.

## Commands

```bash
npm run dev        # Electron + Vite with HMR (needs pi on PATH; see below)
npm run typecheck  # tsc for main (tsconfig.node) + renderer (tsconfig.web)
npm run lint       # eslint
npm run format     # prettier --write
npm test           # vitest unit tests
npm run test:e2e   # builds, then Playwright-Electron against the pi stub
npm run validate   # all of the above, quiet: PASS/FAIL summary + a log file
```

CI runs typecheck, lint, `prettier --check .`, unit tests, build, and the e2e
matrix (ubuntu + macOS). Run typecheck + lint + test before considering a
change done; run e2e when touching IPC, session lifecycle, or visible UI flow.

`npm run validate` (`scripts/validate.sh`) is the one to reach for when you
just want a verdict: it prints one line per step and sends everything else to
`$VALIDATE_LOG` (default `/tmp/phosphor-validate-$$.log`). `SKIP_E2E=1` stops
before the slow part.

**E2E windows never appear on your screen**, so a background agent running the
suite can't steal focus mid-keystroke. `scripts/e2e.sh` prefers `xvfb-run`
(real windows on a virtual display, full speed — install with
`sudo apt install xvfb`) and otherwise leaves the windows unmapped
(`hideWindowsForE2E` in `electron/window-chrome.ts`), which is ~2-3x slower
because Chromium deprioritizes rendering for a window that was never shown.
`PHOSPHOR_E2E_SHOW=1 npm run test:e2e` puts them back on your real display when
you want to watch.

## Architecture in six facts

1. **The main process owns all side effects.** The renderer runs sandboxed
   (contextIsolation, no Node) and is pure UI over typed IPC. If a feature
   needs disk/network/subprocess, it goes in `electron/`, not `src/`.
2. **IPC is a typed contract.** A new channel = an entry in
   `shared/ipc.ts` `IpcInvokeMap` + a handler in
   `electron/ipc/<prefix>-handlers.ts` (the module matching the channel prefix
   — 18 of them, listed in [README.md](README.md#repo-layout)) + a case in
   `src/dev/mockPhosphor.ts` if the browser harness should exercise it.
   `electron/ipc.ts` is only the composition root; the session registry lives
   in `electron/registry.ts` so handlers never import their composition root.
3. **RPC to pi goes through `src/lib/rpc.ts`** (`piCall` / `piCallOk`), which
   unwraps the `{success, data?, error?}` envelope and surfaces failures on
   the session's chat. Calling `window.phosphor.piCommand` directly means you own
   the error branch — half the original call sites forgot, so don't.
4. **`shared/rpc.ts` is a mirror of pi's protocol** with compile-time drift
   guards (`_NoMissingResponseKeys` / `_NoExtraResponseKeys`). Adding an RPC
   command means updating both the command union and `RpcResponseDataMap`, or
   it won't compile — that's intentional.
5. **Every session is independent.** There is no cross-session manager: no
   fleet hub, no orchestrator thread, no automatic reclamation of an idle
   session's subprocess. Interactive sessions are created from the renderer;
   explicit local routines also start sessions through main's shared runtime
   ([routines.md](docs/routines.md)). Their bounded scheduler owns only those
   executions, not an agent fleet. `electron/registry.ts` remains the only
   live-process registry. The
   orchestration layer that used to do this was removed on 2026-09-03 for
   maintenance cost — it touched session spawn, IPC, the sidebar, the home
   screen and settings at once. Do not grow it back. The known cost of the
   removal is that nothing reclaims an idle session's ~172 MB pi tree
   ([known-issues.md](docs/known-issues.md) S11).
6. **Stores (`src/stores/`, zustand) are projections of main-process state.**
   `files.ts` and `terminal.ts` are keyed `byWorkspace[path]`; their
   `workspaceFiles()` / `workspaceTerminals()` selectors return a shared
   frozen empty value — never mutate it, never inline a fresh `{}` in a
   selector. "Which workspace am I in?" is `useActiveWorkspace()` (derived,
   prefers the active session's own cwd) — not a global current-workspace.

## Sharp edges (read before touching)

- **`electron/pi/session-writer.ts` appends to pi's own session files**
  (bookmarks, branch jumps, forks). It is only safe while no pi process owns
  the file — call sites enforce this by convention. It depends on pi's on-disk
  format staying stable. Tests: `electron/pi/session-writer.test.ts`.
- **JSONL framing is strict LF via `JsonlDecoder`, never `readline`** —

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agustinsacco/phosphor](https://github.com/agustinsacco/phosphor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
