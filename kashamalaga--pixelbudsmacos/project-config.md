---
trigger: always_on
description: Guidance for AI coding assistants (Claude Code, Cursor, Codex, …) working
---

# AGENTS.md

Guidance for AI coding assistants (Claude Code, Cursor, Codex, …) working
in this repo. Read this before touching anything substantial.

## House style

- **Code, identifiers, comments, commit messages, and these docs are in
  English.** The original developer chats with assistants in Spanish, but
  what lands on disk is English. Don't mix.
- **Comments explain *why*, not *what*.** The code already says what it
  does. Comments earn their keep when they explain a constraint, a
  surprising decision, or something that bit us before.
- **No defensive boilerplate.** Don't add input validation or error
  branches for cases that physically can't happen given how a function is
  called. The codebase trusts itself.
- **No emoji in files unless explicitly requested.** Same for excessive
  formatting in commit messages.

## Build / run / test

```sh
swift build                     # builds everything
swift run BudsSpike             # Phase 0 hex-dump CLI
swift run BudsRead              # end-to-end CLI tester (reads + ANC)
swift test                      # runs the pure-Swift unit tests
./Scripts/build-app.sh          # produces build/PixelBudsBar.app
open ./build/PixelBudsBar.app   # launch the menu-bar app
```

After UI changes, you typically want `./Scripts/build-app.sh` and then
re-launch the `.app` (the existing instance won't pick up a rebuild).

## Repo layout

See [`ARCHITECTURE.md`](ARCHITECTURE.md) for the diagram. TL;DR:

```
Sources/
  Maestro/                 ← pure Swift protocol stack (HDLC, RPC, proto)
  MaestroIOBluetooth/      ← macOS IOBluetooth bridge
  PixelBudsBar/            ← the SwiftUI menu-bar app
  BudsSpike/, BudsRead/    ← diagnostic CLIs
Tests/MaestroTests/        ← unit tests for the protocol stack
Reference/                 ← upstream proto + notes from qzed/pbpctrl
Scripts/build-app.sh       ← .app bundler + ad-hoc signer
```

## Where things should go

- A new **settings toggle** (boolean, single field, no derived state) →
  add a getter/setter to `MaestroService` if missing, then extend
  `BudsSnapshot`, read it in `runOneSession`, add a setter via
  `writeSetting(label:write:update:)`, and surface it in the controls
  section. See the Mono Audio implementation as a clean template.
- A new **slider** or other continuous control → use the `@State
  inProgress` + `onEditingChanged` pattern (see `EqualizerSection` and
  `BalanceSection`) so you fire one RPC per gesture, not one per slider
  tick.
- A new **RPC** that doesn't fit the `readSetting` / `writeSetting`
  pattern → add it to `MaestroService` directly, mirroring an existing
  one. The RPC channel ID is `MaestroService.channel`; method names map
  to the Pigweed hash via `HashId.compute`.
- A protocol-stack change (HDLC, RPC, varint) → make it in `Maestro/`
  *with a test*. The unit tests there are quick (no Bluetooth) and the
  layer is finicky.

## Common gotchas

These cost us hours each:

- **Async close of IOBluetooth RFCOMM.** `IOBluetoothRFCOMMChannel.close()`
  is non-blocking. We wait for the `rfcommChannelClosed` delegate via a
  continuation with a 2s safety timeout. If you bypass this, the next
  `openRFCOMMChannelAsync` hangs.
- **Firmware status 9 ("FAILED_PRECONDITION").** Means "put the buds in
  your ears first" for most settings. Surface it via
  `lastWriteError`, **never** via `connectionState`. The settings
  subscription will push the canonical value back, reverting the
  optimistic UI for free.
- **Settings reads that return status 2 ("not readable").** Some fields
  (notably `speech_detection`) can be written but not read on certain
  firmwares. Treat the read as optional (`try? await …`) and let the
  UI render an indeterminate state until the device subscription
  delivers the canonical value.
- **GFPS channel can drop alone.** Maestro keeps working but the
  RFCOMM-level close fires `rfcommChannelClosed` on the GFPS adapter.
  We observe `GFPSChannel.whenClosed()` and hide the Ring control.
- **Never refresh runtime by re-subscribing.** The firmware allows only
  **one** `SubscribeRuntimeInfo` per channel; opening a second one (e.g.
  via `currentRuntimeInfo()`, which is subscribe-take-one) terminates the
  primary runtime stream and the session driver treats it as a drop. A
  retired 30s "refresh" poll did this and forced a reconnect twice a
  minute. The in-session liveness check is a unary `GetSoftwareInfo`
  heartbeat for this reason. `currentRuntimeInfo()` is fine for the *one*
  initial read (before the main subscription is open), not for polling.
- **`writeSync` can block for over a minute.** On a multipoint half-dead
  link the channel reports "open" but `writeSync` hangs until IOBluetooth's
  internal timeout. `RFCOMMTransportAdapter.write()` races a 5s timeout that
  closes the channel to abort it. Don't remove that timeout, and keep
  writes cheap.
- **Cancellation-aware continuations.** Any `CheckedContinuation` resumed
  by an IOBluetooth delegate or dispatch-queue work (open-complete,
  `writeSync`) must use `withTaskCancellationHandler` + a lock-guarded
  single resume (`openLock`, `WriteContinuationBox`). Otherwise a cancelled
  open/write leaves the child suspended and the enclosing
  `withThrowingTaskGroup` never returns — deadlocking teardown /

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KashaMalaga/PixelBudsMacOS](https://github.com/KashaMalaga/PixelBudsMacOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
