---
trigger: always_on
description: Rules an agent has to follow when editing this repo. The codebase has a
---

# bridgething agent guide

Rules an agent has to follow when editing this repo. The codebase has a
clean lib/core split that is easy to silently break - these rules exist
to keep that split intact.

## Crate layout and what goes where

The repo splits into two top-level workspace roots:

- `crates/` — Rust workspace members. Two families: the daemon side (`crates/lib`, `crates/core`, `crates/iap2`, `crates/mfi`, `crates/mfi-proxy`, `crates/dsp`, `crates/wakeword`, ...) and the shared companion core (`crates/sdk-runtime`, `crates/io`, `crates/gateway-rs`, `crates/delivery/{core,napi,wasm}`, `crates/companion`, plus `crates/spotify` and `crates/nlu` linked into it). `crates/client-rs` and `crates/host-gateway` are the Rust-side consumers. The cargo workspace also pulls in `tools/codegen/` and `desktop/src-tauri/`.
- `packages/` — Bun/turbo workspace members (`packages/browser`, `packages/client-ts`, `packages/ui`, `packages/updater`, `packages/session-rn`, `packages/webapp-shared`, `packages/webapps/{builtin,catalog}/*`). `builtin` rides the daemon release and is never published to the catalog; `catalog` is what the store distributes. `packages/ui` is preact + tailwind shared by the desktop app and the site. `packages/companion/{swift,kotlin}` and `packages/asr` are the mobile platform shells over the shared core, not bun members.
- `mobile/` — RN app, consumer of the packages.
- `desktop/` — tray-resident Tauri app, laid out the way `create-tauri-app` scaffolds one: the app root is `desktop/` itself (`package.json`, `index.html`, `vite.config.ts`, preact sources under `src/`) with `src-tauri/` beside them. The two deviations from the scaffold are deliberate: `src-tauri/` is a member of the root cargo workspace, and the frontend is a bun workspace member (`@bridgething/desktop-frontend`) so it shares `packages/ui`. `src-tauri/` links `crates/companion` natively and is the only place state lives; the frontend holds none of it.
- `site/` — bridgething.com. Astro + preact islands, deployed to cloudflare.

The lib/core split below is the load-bearing one. Naming convention: Rust crates use kebab-case package names (`bridgething-mfi`); TS packages use scoped names (`@bridgething/lib`).

### crates/lib/ (`libbridgething`) — wire surface only

Anything that crosses a websocket or bluetooth boundary lives here, plus
the codec/framing for those boundaries. That's it.

Allowed in lib:

- Wire DTOs (every type that gets serialized to msgpack on the BT link
  or to JSON on the local websocket).
- The codec / framing in `crates/lib/src/protocol/`.
- Compile-time constants used by the protocol (UUIDs, ports, class IDs).
- `serde`, `ts-rs`, `uuid`, `serde_with`, `derive_more`,
  and the `protocol` feature deps (tokio-util, flate2, rmp-serde).

Forbidden in lib:

- Tokio runtime types (`tokio::sync::mpsc`, `tokio::task`, etc).
- Handlers, managers, daemon state, hardware drivers.
- Errors that aren't pure protocol errors. `EndecError` is fine.
  `BluetoothError` is not.
- Anything that would not be useful to a third party importing
  `libbridgething` purely to speak the wire protocol.

If you reach for `tokio::sync::mpsc::Sender<Foo>` in lib, stop — that
type belongs in core.

### crates/core/ (`bridgething`) — the daemon

Everything else. Handlers, managers, axum server, BlueR plumbing,
chromium CDP driver, persistent state, hardware drivers (ALS, mic),
systemd integration. The binary lives here.

Core depends on lib for wire types and re-exports nothing.

### crates/client-rs/, packages/browser/

Both consume `libbridgething`. They MUST NOT redefine wire types — they
re-export from lib or build on top of lib's types. If a wire type needs
a field added, the field goes in lib and propagates outward, never the
other way.

`packages/browser` is the host-side transport: the delivery core compiled
to wasm, plus the Web Serial and WebSocket plumbing that has to be written
in TypeScript because no other language can reach those APIs. The protocol
logic underneath it is Rust and is not duplicated there. A host reaches the
daemon's network gateway on port 8892 the same way on Linux, macOS, Windows
and in the browser.

## Wrap, don't duplicate

If a runtime variant of an enum doesn't fit a wire enum, do not copy
the wire enum into core and add the variant. Wrap it.

Example of the right pattern (already in `crates/core/src/handler/client/msg.rs`):

```rust
// lib::ClientCommandType is the wire enum.
// core::RecvMsgData wraps it and adds runtime-only variants.
pub enum RecvMsgData {
  Bluetooth(ClientBluetoothCommand),    // re-projected from lib variant
  // ...
  Hole,                                 // runtime only
  Unsupported(PossibleRecvMsg),         // runtime only
  ChangeMode(ClientMode),               // runtime only
  ConnectionClosed(u16, String),        // runtime only
}
```

The fields inside `Bluetooth(ClientBluetoothCommand)` are NOT redefined
in core — they reuse the lib type. Only the _enum shell_ is core-side
because it has runtime-only variants.

The cautionary tale: `WebappInfo` was at one point copy-pasted
identically between lib and core. There is exactly one canonical home
for any wire type: `crates/lib/`. Core imports it.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JoeyEamigh/bridgething](https://github.com/JoeyEamigh/bridgething) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
