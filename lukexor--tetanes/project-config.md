---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

TetaNES is a cross-platform NES emulator. The workspace has three crates:

- **`tetanes-core`** — the emulation library (CPU/PPU/APU/mappers/cart). Published, aims for stronger
  API stability, and must compile on stable and MSRV `1.88` in addition to nightly.
- **`tetanes`** — the UI binary: `winit` event loop + `egui` GUI + `wgpu` renderer. Targets desktop
  and `wasm32-unknown-unknown` (web via `trunk`).
- **`tetanes-utils`** — unpublished dev binaries (`chrdump`, `generate_db`, `list_boards`,
  `screenshot`).

## Commands

The toolchain is pinned to **nightly** (`rust-toolchain.toml`) and edition 2024. `.cargo/config.toml`
sets nightly-only `RUSTFLAGS`; when invoking a non-nightly toolchain you must unset
`CARGO_ENCODED_RUSTFLAGS` (CI does this for the stable/1.88 clippy jobs).

Most workflows go through [`cargo-make`](https://github.com/sagiegurari/cargo-make) (`Makefile.toml`):

```sh
cargo make dev -- path/to/rom.nes     # debug run (opt-level 1, playable)
cargo make run -- path/to/rom.nes     # release run
cargo make dev-web / run-web          # trunk serve, dev/release
cargo make lint                       # clippy for native + wasm, --all-features
cargo make check-fmt
cargo make test -- <args>             # cargo nextest run --all-features --no-fail-fast
cargo make docs                       # rustdoc native + wasm (CI treats warnings as errors)
cargo make bench                      # perf stat around the clock_frame benchmark
cargo make build                      # PGO build (cargo-pgo)
```

Clippy must be clean with `-D warnings` for: native `tetanes`, wasm `tetanes`, and `tetanes-core` on
nightly/stable/1.88.

### Tests

Both crates have tests, each with its own CI job. `tetanes-core`'s are the bulk of them; `tetanes`'
cover the audio-rate control loop and the rewind ring.

```sh
cargo nextest run -p tetanes-core --all-features           # everything
cargo nextest run -p tetanes-core nestest                  # substring filter for one test
cargo nextest run -p tetanes-core common::tests::cpu::     # a whole ROM-test group
cargo make update-snapshots -- <test>                      # rewrite expected frame hashes
```

Most tests are **ROM snapshot tests**. The harness lives in `tetanes-core/src/common.rs` (`mod tests`):
the `test_roms!` macro declares one `#[test]` per named ROM, expectations come from
`test_roms/<dir>/tests.json` (frame number → frame-buffer or audio hash, plus optional `Action`s to
inject), and rendered PNGs land in `tetanes-core/test_results/{pass,fail}/`. Adding a ROM test means
adding the ROM + a `tests.json` entry + a name in the relevant `test_roms!` invocation at the bottom
of `common.rs`. `cargo make update-snapshots` rewrites `tests.json` in place — only use it when a
hash change is intentional and the resulting PNG has been eyeballed. It is `UPDATE_SNAPSHOT=1` plus
`--test-threads=1`; the harness merges its own entry into the file under a lock either way, so the
raw env var is safe too, but serialising keeps the diff readable.

### Commit messages

Conventional Commits (`cliff.toml` / release-plz generate the changelog and releases from them).

A message is a **synopsis of the theme and the reason for it**, not a record of how the work went.
Leave out what the diff already says: implementation walkthroughs, verification narration ("191
tests pass"), plan phase numbers and scratch-doc references, enum-size and boxing bookkeeping,
benchmark tables, and references to sibling commits by hash. Rationale tied to a specific line
belongs in a `//` comment next to that line, where the next reader will actually find it. What
survives is the theme, the reason, and anything that would cost time to re-learn.

A `BREAKING CHANGE:` footer must be **one line**. `cliff.toml` renders
`commit.breaking_description`, and git-conventional truncates that at the first continuation line,
so a wrapped footer silently loses everything after the first line in the changelog.

## Architecture

### Emulation core

`ControlDeck` (`control_deck.rs`) is the public entry point: it owns a `Bus`, loads `Cart`s, and
exposes `clock_frame`, save states, rewind data, and `Action` handling.

```
ControlDeck → Bus → { Cpu, Ppu, Mapper, Memory, Apu, Input, WRAM }
```

`Bus` is the container the components are wired into, and the whole of the emulated state — a save
state, a rewind frame and a run-ahead snapshot are each exactly one `Bus`, which is why it holds
bus state and nothing else: the emulated components, plus what the bus itself needs to run them
(`ram_state`, the attached `debugger`, `disasm`). The session — video, run-ahead buffers,
`sram_dir`, config — stays on `ControlDeck`.

`Cpu` and `Ppu` are the state a 6502 and a 2C02 keep. **What they do is an `impl Bus` block**,
because an access moves the whole machine: reading a byte clocks the PPU, the APU and the board on
the way past, and a CHR fetch goes through the board's page tables. Those blocks live in the file
that owns the state they read — the CPU's in `cpu.rs`, the instruction set's in `cpu/instr.rs`, the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lukexor/tetanes](https://github.com/lukexor/tetanes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
