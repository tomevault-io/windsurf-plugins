---
trigger: always_on
description: Guidance for Claude Code in this repo. Cross-platform desktop music player: **Slint 1.16** UI + pure-Rust backend, direct calls + tokio channels (no WebView, and the only IPC is between two *Melodias* — the single-instance socket below).
---

# CLAUDE.md

Guidance for Claude Code in this repo. Cross-platform desktop music player: **Slint 1.16** UI + pure-Rust backend, direct calls + tokio channels (no WebView, and the only IPC is between two *Melodias* — the single-instance socket below).

## Where the docs live

This file holds what applies repo-wide. Subsystem contracts load on demand — **put new subsystem detail in the rule, not here.**

| file | covers |
|------|--------|
| `.claude/rules/audio-stack.md` | playback state machine, DSP chain (EQ / ReplayGain / limiter), crossfade + decks, visualizer tap |
| `.claude/rules/library-data.md` | scan ingest, the two change channels, track projections, ratings, tag write-through, M3U8, smart playlists |
| `.claude/rules/ui-patterns.md` | shared components (pills, pickers, grids, toasts), cover prewarm, the two teardown paths, shortcuts, **the three tabbed pages**, the Settings page |
| `.claude/rules/desktop-shell.md` | window chrome + winit, tray, media keys, always-on-top, force-exit shutdown |
| `.claude/rules/updater.md` | install methods, manifest signing, the release matrix |
| `.claude/rules/visualizer.md` | the visualizer's UI half — arming, the tick's gates, the strip and its pickers |
| `.claude/rules/radio.md` | the directory facade and its off switch, the two station tables, the stream path's ring and feed thread, the numbers both trees spell |
| `.claude/rules/ci-packaging.md` | the PR gate, the skip matrix, coverage, action pinning, the five package formats and their licence pins |
| `.claude/rules/diagnostics.md` | the logging sink, the crash hook, the bug-report bundle |
| `.claude/rules/testing.md` | testing practice, not this tree's inventory: the ISTQB principles, how to pick cases, the Rust attribute and determinism rules, assertion discipline |
| `.claude/rules/*.md` (rest) | per-crate best practices (tokio, sqlx, slint, symphonia, lofty, rayon, serde, blake3, rust-performance) plus `slint-pitfalls`, `unsafe-rust` and `code-style` |

Rules are **path-scoped** by a `paths:` glob and load when Claude *reads* a matching file, so a grep hit or a clippy failure won't pull one in. **A rule earns its place only when its subject has no single anchor file**: a coupling between trees (Rust ↔ `.slint`, Rust ↔ CI, Rust ↔ a shipped migration nobody may edit), or a comparison across peers that no one of them owns. Everything else is argued *at its anchor*, in the doc comment on the constant, function or migration it constrains, where it cannot drift out of sight of the code it describes. A **prohibition** violable from outside that anchor's directory stays in this file. Hence "no `unwrap`", the zbus footgun and the `--version` contract sitting here.

Two copies of one argument is the failure mode that test exists to prevent, and it is not hypothetical: `tasks/mbid_backfill.rs` and the rule describing it came to disagree about whether a set was persisted, each correct on the day it was written. Prose in a rule that restates a doc comment is the copy to delete.

A subsystem gets a `CLAUDE.md` when it **is** a directory and its contract needs more than the module `//!` docs carry; a `.claude/rules/` entry when the subject spans trees no single file can reach. UI features always span (`.slint` under `crates/melodia-ui/ui/`, Rust under `crates/melodia-views/src/ui/`), so a per-directory file would reach one tree and silently miss the other.

Only this file is re-injected after `/compact`; nested ones come back on the next read in their directory, so a long session can lose one without saying so. Module `//!` docs are the third tier and usually the most current, and tuning constants are justified at their definitions — link, don't restate.

**`docs/adr/` answers the question none of the above can: why this, and not the obvious alternative.** A choice between two libraries has no anchor file, because the code is the consequence of the decision rather than the decision, so it lands nowhere or lands in a working doc the convention then deletes. It carries no `paths:` glob and does not load on its own; start at `docs/adr/README.md`, which holds the index, the shape and when not to write one. **A prohibition here that used to argue itself is argued there now — don't restore the paragraph, and don't put a second copy of one in an ADR either.** A working doc that reaches the point of being deleted leaves an ADR behind first.

## Build & Dev Commands

```bash
cargo run -p melodia                       # debug
cargo build --release -p melodia && target/release/Melodia
cargo fmt --all                            # format; CI gates it with --check
cargo clippy --all-targets --locked --workspace -- -D warnings   # lint + check (never cargo check)
cargo test --locked --workspace            # no doctests — `[lib] doctest = false`
cargo llvm-cov --workspace --html          # coverage → target/llvm-cov/html/
/usr/bin/time -v target/release/Melodia    # peak RSS (release only)
```

## Prerequisites

- **Rust** edition 2024, pinned to 1.97.0 by `rust-toolchain.toml`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KenanSalar/Melodia](https://github.com/KenanSalar/Melodia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
