---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A VoiceMeeter-style virtual audio mixer for Linux: Tauri 2 shell + Svelte 5 frontend +
a Rust PipeWire engine. Input strips (hardware capture or virtual sinks apps play into)
run gate → comp → EQ, feed a routing matrix, and land on four buses (A1/A2 → hardware
sinks, B1/B2 → virtual microphones). DSP is the LSP LV2 plugins hosted inside
`libpipewire-module-filter-chain`.

## Commands

Use the Makefile; it is the canonical entry point. `make` with no target lists everything.

```sh
make install      # frontend deps (pnpm install)
make dev          # browser-only UI against the MOCK backend (design iteration)
make app          # full app against the live PipeWire graph
make test         # Rust + frontend unit tests (no audio system needed)
make test-audio   # integration tests against a private throwaway PipeWire daemon
make check        # svelte-check + cargo check --workspace --all-targets
make build        # release binary -> target/release/linuxmeeter
make graph        # headless REPL: full topology — `route a1 0|1`, `links`, `meters 1`, `q`
make spike        # headless REPL: one strip, raw param pokes — `set comp:cr 8`, `vol 0.5`, `q`
```

Also available: `test-rust` / `test-ui` (the halves of `test`), `test-all`, `check-ui` /
`check-rust`, `clippy`, `fmt`, `fmt-check`, `run` (already-built release binary), `clean`,
and `clean-cache` (wipes the WebKitGTK cache — see gotchas below).

`LOG` sets `RUST_LOG` for `app`, `graph`, `spike`, and `run`; it defaults to `info`, so
`make app LOG=debug` or `make graph LOG=lm_engine=debug` for tracing detail. `PNPM` and
`CARGO` are overridable the same way.

## Testing

Three layers, in ascending order of what they need:

**`make test` — pure logic, no audio system.** Rust unit tests live in `#[cfg(test)]`
modules beside the code (`params`, `links`, `filterchain`, `meter`, `persist`,
`lm-protocol`); frontend tests are `src/**/*.test.ts` under vitest. Between them they
pin the things that are silently wrong rather than loudly broken: Props pod shapes,
`match_ports` channel pairing, `state.restore-props` on every node, profile migration,
meter dBFS arithmetic, the camelCase IPC contract with `types.ts`, the fader taper, and
the EQ response curve.

**`make test-audio` — real PipeWire, real LSP plugins, real samples.**
`scripts/with-test-daemon.sh` starts a *private* daemon: its own socket in a temp runtime
dir, one null sink, no session manager, `XDG_CONFIG_HOME` redirected so profiles never
touch `~/.config/linuxmeeter`. `crates/lm-engine/tests/audio.rs` then builds the
production signal path out of the engine's own parts — tone → strip (gate/comp/EQ) →
routing matrix → bus (limiter) → sink, with meter taps — and asserts on measured dBFS.
A 0.5-amplitude sine must arrive at −6.02 dBFS peak / −9.03 RMS; `-12 dB` of gain must
move the meter by 12 dB; mute must reach the noise floor; a gate threshold above the
signal must close it. This is what catches "the knob does nothing" bugs.

These tests are `#[ignore]`d and additionally refuse to run unless `LM_TEST_DAEMON=1` is
set by the harness script — a bare `cargo test -- --ignored` fails fast rather than
creating real devices in the developer's own audio session. Because they share one
daemon and one process environment, they run with `--test-threads=1`.

Without a session manager nothing configures ports, so the harness sets
`adapter.auto-port-config` for both the daemon's null sink and this process's streams.
Without that, every node appears with zero ports and no link can ever form.

**Manual.** Perceptual quality, real hardware enumeration, and WirePlumber policy
interactions are still eyes-and-ears work: `make graph` / `make spike` plus `pw-dump`,
`pw-link`, `pw-top`, `pw-record --target lm.bus.b1 out.wav`.

## Architecture

Three layers, each with a hard boundary:

- **`crates/lm-protocol`** — every type crossing a boundary (`AppState`, `StripState`,
  `EngineCommand`, `EngineEvent`, `MeterFrame`). No dependencies beyond serde.
- **`crates/lm-engine`** — all PipeWire code. Deliberately Tauri-free so it runs headless
  under `examples/`.
- **`src-tauri`** — thin shell: `#[tauri::command]` wrappers that forward to the engine,
  an event-pump thread, tray icon, autostart, close-to-tray.
- **`src/`** — Svelte 5 (runes) frontend, vanilla CSS.

### The engine thread

`lm_engine::engine::spawn()` starts one thread owning a PipeWire main loop. It is the
sole owner of all audio state. Communication is strictly:

- in: `pipewire::channel::Sender<EngineCommand>` (attached to the loop)
- out: `std::sync::mpsc::Receiver<EngineEvent>` → pumped into webview events
  (`state_changed`, `meters`) by a thread in `src-tauri/src/main.rs`

Never reach into engine internals from the Tauri side; add an `EngineCommand` variant.

### Graph construction

The graph is built once, after the registry's initial enumeration completes
(`core.add_listener_local().done(...)` → `build_graph`). Each strip and bus is one
in-process `libpipewire-module-filter-chain` module loaded via the FFI shim in

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dslapelis/linuxmeeter](https://github.com/dslapelis/linuxmeeter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-04 -->
