---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Relink Logs (formerly GBFR Logs; built upon the now-unmaintained [false-spring/gbfr-logs](https://github.com/false-spring/gbfr-logs), no longer affiliated with it) is a DPS parser/overlay for Granblue Fantasy: Relink. It works by DLL-injecting a hook into the running game process, reading game memory + hooking damage functions, and broadcasting events over a named pipe to a Tauri desktop app that parses and displays them. Windows-only; requires running as admin against a live game process.

## Development commands

Requires **nightly Rust** ([rustup.rs](https://rustup.rs/)) + Node.js.

```sh
npm install
npm run tauri dev      # full app: builds hook.dll (release), runs Tauri backend + Vite frontend
```

- `npm run dev` — builds the hook DLL with the `console` feature, then runs Vite (frontend only).
- `npm run build` — `tsc` typecheck + Vite production build of the frontend.
- `npm run test` — Vitest (frontend unit tests). Single file: `npx vitest run src/utils.test.ts`. Watch: `npx vitest`.
- `npm run lint` — ESLint over `./src`.
- `npm run format` / `npm run format-check` — Prettier.
- Rust: `cargo build` / `cargo test` / `cargo clippy` at the workspace root, or `-p gbfr-logs` / `-p hook` / `-p protocol` for a single crate.

The Rust side is well covered — ~750 `#[test]`s, most of them in `src-tauri` (the parser, the group aggregation, the chart builders) and `src-hook`. Prefer a unit test against fake events over a manual run: `cargo test -p gbfr-logs parser::v1::groups` and friends are fast. What the tests CANNOT cover is anything that reads live game memory — the RE'd signatures and offsets — so hook changes still need a run against the game.

## Architecture: five subprojects, two languages

Data flows **game → hook → pipe → parser → frontend**:

1. **`src-hook/`** (crate `hook`, builds `hook.dll`) — Injected into `granblue_fantasy_relink.exe`. Sets up function hooks (`src/hooks/`: damage, death, player load, quest/area, SBA) that read game memory via raw pointers and vtable offsets. Broadcasts `protocol::Message` events over the named pipe `\\.\pipe\gbfr-logs`. Entry is a `#[ctor]` that spawns the server. **The memory offsets and actor-type hashes here (e.g. `get_source_parent` in `hooks/mod.rs`) are reverse-engineered and break on game patches.** The hook also serves the Toolbox RPC channel (see game-reader below).

2. **`protocol/`** (crate `protocol`) — Shared message types (`Message` enum, `DamageEvent`, `PlayerLoadEvent`, etc.). Wire format between hook and parser is **bincode**, so the hook and parser must be compiled together. Read the crate-level doc comment in `src/lib.rs` before changing any message type — adding fields/variants is safe, but the parser's own on-disk format is separate and must stay backward-compatible. The toolbox module carries the request/response channel for the Toolbox tools (`\\.\pipe\gbfr-logs-toolbox` on Windows, TCP 127.0.0.1:39372 under Wine; one request per connection; `TOOLBOX_PROTOCOL_VERSION` guards hook/app wire skew and is a content hash of this crate computed by `protocol/build.rs` — do NOT hand-bump it, it moves itself whenever this crate's sources change. That hash reads every byte here, comments and tests included, so a non-wire edit still flags every deployed hook out of date and rotates `hook.dll` — keep non-wire churn out of `protocol/`).

3. **`game-reader/`** (crate `game-reader`) — Platform-independent snapshot
   walkers plus the RE'd signatures/offsets behind the Toolbox tools
   (synthesis, overmastery), generic over a `MemRead` trait and unit-tested
   against fake memory. Production path: the hook reads in-process (guarded)
   and serves results over the toolbox RPC channel — on both OSes. The diag
   examples (`om_probe`, `synth_probe`, `synth_diag`, `toolbox_probe`) read
   the same structures via `ReadProcessMemory` (`src-tauri/src/game_mem.rs`,
   Windows-only, admin) as an independent cross-check. A game patch that
   moves these structures is fixed in this crate.

4. **`src-tauri/`** (crate `gbfr-logs`, the main binary) — The Tauri backend.
   - `src/main.rs` — Tauri setup, `#[tauri::command]` handlers (the frontend's API surface: `fetch_logs`, `fetch_encounter_state`, `delete_logs`, etc.), system-tray menu, the two app windows, and `check_and_perform_hook` (polls for the game process, injects the DLL, then reads the pipe and feeds events into the parser). If `hook-dbg.dll` exists next to the binary, it's injected instead of `hook.dll`.
   - `src/parser/` — Versioned parsing. `deserialize_version` in `mod.rs` dispatches by stored version byte; `v0` is legacy and upgrades into `v1`. The `v1::Parser` holds an `Encounter` (raw event log, the source of truth) and a `DerivedEncounterState` (computed party/DPS/stun, what the frontend consumes). Logs are re-parsed from the raw event log, so DerivedEncounterState can change between app versions without losing data. Live encounter state is pushed to the frontend via Tauri events (`encounter-update`, `encounter-saved`, `on-area-enter`, `encounter-party-update`, etc.).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [villith/relink-logs](https://github.com/villith/relink-logs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
