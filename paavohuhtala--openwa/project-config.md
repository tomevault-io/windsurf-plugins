---
trigger: always_on
description: OpenWA (short for "Open Worms Armageddon") is an incremental Rust reimplementation of Worms Armageddon 3.8.1 (Steam). The replacement strategy is the same as was used in OpenRCT2: a custom launcher (`openwa-launcher`) injects a DLL (`openwa-dll`) into a suspended game process, which replaces game functions with Rust implementations (from `openwa-game`).
---

# OpenWA

## Project

OpenWA (short for "Open Worms Armageddon") is an incremental Rust reimplementation of Worms Armageddon 3.8.1 (Steam). The replacement strategy is the same as was used in OpenRCT2: a custom launcher (`openwa-launcher`) injects a DLL (`openwa-dll`) into a suspended game process, which replaces game functions with Rust implementations (from `openwa-game`).

The original WA.exe is a 32-bit x86 Windows PE binary built with MSVC 2005 + MFC. All Rust code targets `i686-pc-windows-msvc`.

## Crate Architecture

- **`openwa-core`** — Cross-platform, idiomatic Rust fundamentals. No WA.exe memory references, no Ghidra addresses, no Windows APIs. Currently hosts: `dir` (.dir sprite archive parser), `fixed` (16.16 `Fixed` + 48.16 `Fixed64` newtypes), `img` (.img tagged + headerless decoder), `log` (file-logging helper), `pal` (RIFF .pal palette parser), `rng` (WA's LCG PRNG), `scheme` (.wsc parser), `sprite_lzss` (LZSS decompressor), `trig` (sin/cos extracted from WA.exe, plus interpolation helpers), `weapon` (Weapon/FireType/FireMethod/SpecialFireSubtype enums). New portable modules migrate here from `openwa-game` as they're confirmed platform-independent. See `crates/openwa-core/CLAUDE.md` for the charter.
- **`openwa-game`** — WA.exe-specific code (`i686-pc-windows-msvc` only). Types, addresses, parsers, ASLR rebasing, typed WA function wrappers, **and game logic**. The source of truth for all reverse-engineered type layouts, known addresses, and Rust reimplementations of WA functions. Contains `registry` (structured address database + field registries), `rebase` (ASLR delta), `wa_call` (calling convention helpers), `wa/` (typed handle wrappers), and game logic modules (`game/weapon_fire.rs`, `game/weapon_release.rs`, `audio/sound_ops.rs`, `engine/team_ops.rs`).
- **`openwa-derive`** — Proc macro crate. Provides `#[derive(FieldRegistry)]` for struct field maps and `#[vtable(...)]` for typed vtable definitions with introspection, calling wrappers, and replacement support.
- **`openwa-dll`** — Injected DLL (`openwa.dll`): thin hook installation shims (trampolines, `usercall_trampoline!`, `install()`) that wire core's game logic into WA.exe via MinHook. Logs to `OpenWA.log`. Runs registry-driven startup checks automatically at load.
- **`openwa-test-runner`** — Headless replay test runner (`openwa-test` binary). Discovers replay tests, runs them concurrently via WA.exe's `/getlog` mode, compares output logs. See "Replay Testing" section.
- **`openwa-launcher`** — Launches WA.exe with the DLL injected via CREATE_SUSPENDED + remote thread.
- **`openwa-debugui`** — In-process egui debug window (entity census, struct inspector, cheats). Enabled via `OPENWA_DEBUG_UI=1` + `debug-ui` cargo feature.
- **`openwa-debug-cli`** — CLI tool for live memory inspection (`openwa-debug` binary). Connects to the debug server in the DLL.
- **`openwa-debug-proto`** — Shared protocol types (Request/Response enums, MessagePack framing) between CLI and server.
- **`openwa-asset-viewer`** — Standalone egui application (`openwa-asset-viewer` binary) for browsing WA's on-disk asset files (`.img` / `.pal` / `.dir`). Consumes `openwa-core` parsers only; does not depend on `openwa-game`. See `crates/openwa-asset-viewer/CLAUDE.md`.

## Build & Test

Build: `cargo build --release` (default target is `i686-pc-windows-msvc` via `.cargo/config.toml`).

Unit tests: `cargo test`. These are standard Rust tests covering parsers and type logic.

**Replay tests are the primary verification method** — see below.

## Replay Testing

WA.exe can deterministically replay recorded games (`.WAgame` files). Each replay test runs the game with the injected DLL and checks that the output matches a baseline log (`*_expected.log`) captured from unmodified WA.exe.

Two ways to run replay tests:

### Headless (`.\run-tests.ps1`)

Pure CPU simulation, no rendering. Fast, runs in parallel (default 4 concurrent). Validates game logic — a log mismatch means the Rust code caused a desync. Spurious flakes from race conditions can occur; retry with `-j 1` to confirm.

### Headful (`.\replay-test.ps1` or `openwa-test headful`)

Runs with graphics and sound. The game window must be focused once to start. Needed to validate visual/audio hooks — code can pass headless but crash headful (or vice versa) since rendering and sound paths are only exercised headfully. Checks for crashes, panics, and `[GAMEPLAY PASS/FAIL]` markers. Timeout configurable with `--timeout SECS` (default 150s).

See `crates/openwa-test-runner/CLAUDE.md` for test isolation, crash detection, adding new tests, and env vars. Use `/desync-debug` skill after a test failure to diagnose.

### Use replay testing to validate assumptions and test theories

Implement a hypothesis, run tests, iterate. You can add temporary log statements and see their results by running replay tests.

### **IMPORTANT**: Replay tests can only test two things:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [paavohuhtala/OpenWA](https://github.com/paavohuhtala/OpenWA) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
