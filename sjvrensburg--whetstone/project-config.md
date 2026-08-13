---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

**Whetstone** — a friction-first writing surface for students under honor codes,
shipped as a single terminal editor (`whetstone-tui`, Rust, edition 2024) with a
Fresh/Micro-style UI (menus, theming, mouse, no modes). See `README.md` for
keybindings, env vars, and config files.

The product claim is **friction, not proof**: the tool adds deliberate friction
to the writing process (paste quarantine, claim-to-own, teach-back, an optional
question-only coach) and can produce an honest, self-reported disclosure of how
a piece was written — it never claims to *verify* authorship or personhood.

This was once a multi-surface project (a web composer and a legacy VS Code
extension). Both have been removed; the TUI is the whole product. The earlier
design history (ADRs, walking-skeleton spec) is not kept in-repo — it was
archived outside the repository.

## Commands

Run from the repo root:

- `cargo run -- path/to/file.qmd` — open (creates the file if missing)
- `cargo run -- lint path/to/file.qmd` — a headless subcommand (also: `coach`,
  `guard`, `ownership`, `disclosure`) printing JSON; `--help` lists them
- `cargo build --release` — produces `./target/release/whetstone-tui`
- `cargo test` — run the test suite
- `cargo run --features screenshots --example screenshots` — regenerate
  `docs/screenshots/*.png` (and `cargo test --features screenshots` covers the renderer)
- `cargo clippy --all-targets -- -D warnings` — lint (CI treats warnings as errors)
- `cargo fmt --check` — formatting check (`cargo fmt` to apply)

CI runs fmt + clippy + test on Linux/macOS/Windows: `.github/workflows/ci.yml`.

## Architecture

Module dependency DAG (each module depends only on modules above it), declared
in `src/lib.rs`:

```text
core -> coach -> instruments -> editor -> grammar -> markdown -> ui
```

- `src/core/` — pure domain logic: n-gram overlap, claim-to-own ownership,
  forbidden-label guard, disclosure rendering, the process-event model, and the
  friction policy. No I/O, no editor/UI imports.
- `src/coach/` — the optional AI coach: a streaming client speaking **two**
  protocols (OpenAI-compatible Chat Completions and Anthropic Messages — see
  `provider.rs`; provider explicit or auto-detected from the base URL), config
  resolution (incl. `env:NAME` references and a separate coach vs. judge model),
  the optional LLM **judge** (`judge.rs`), and per-document chat history.
- `src/cli.rs` — headless subcommands (lint / coach / guard / ownership /
  disclosure / **export**) printing JSON; a bare file path still opens the TUI
  (`main.rs`).
- `src/screenshot.rs` + `examples/screenshots.rs` — in-process PNG rendering of
  the TUI (feature `screenshots`), built on `src/ui/testkit.rs`'s headless
  harness (feature `harness`, also on under `cargo test`).
- `src/instruments/` — the friction instruments (paste cadence, teach-back,
  push-cadence) wired to the friction dial.
- `src/editor/` — the text buffer, change sets, and paste-quarantine regions.
- `src/grammar/` — local grammar checking via `harper-core` (zero external calls).
- `src/markdown/` — markdown/Quarto rendering to terminal cells, LaTeX→Unicode
  math, the heading-outline extractor, and HTML/plain-text export
  (`render_to_html`, `render_to_plain`).
- `src/ui/` — the ratatui app: state, key/paste/mouse handling, layout, menus,
  theming, and all overlays. `app/` is a directory module: `app/mod.rs` holds
  the `App` struct + `impl App` + the test module; `app/render.rs` holds the
  rendering layer (`pub fn draw` + all `draw_*` functions). Also `menu.rs`,
  `theme.rs`, `settings.rs`, and `testkit.rs`.

### Testing the UI headlessly

`src/ui/testkit.rs` (feature `harness`, on under `cargo test`) drives the editor
off-screen via a ratatui `TestBackend`. To add a UI test: build an `App` via the
`test_app` / `rt` helpers in `src/ui/app/mod.rs`'s test module, drive it with
`app.handle_key(KeyEvent::new(...))` or `app.dispatch_for_test(MenuAction::...)`,
and assert on `app.buffer.text()`, `app.journal`, or the rendered string via the
`render()` helper. The same harness regenerates the docs screenshots.

## Project-specific invariants

- **Metadata only**: process events never carry document prose (sizes,
  locations, and the writer's own stated claim only).
- **Claim-to-own direction**: ownership is measured as *how much of the
  ORIGINAL paste survives in the current text* (`src/core/ownership.rs`), never
  the reverse — the reverse is the V1 padding-attack bug, guarded by a test.
- **Forbidden-label guard**: every user-facing artifact must pass
  `assertNoForbiddenLabels`/`screen_*` guards — nothing may imply "verified
  human" / proof-of-personhood. The product claim is *friction, not proof*.
- **Coach is question-only**: every coach/chat reply is screened
  (`src/core/guard.rs`) before it reaches the UI — length cap, rewrite/dictation
  patterns, n-gram overlap with the draft, forbidden labels — so the coach can
  never ghostwrite. **Two paths differ by construction:** *structured* coaching
  (anchored observations, JSON schema with `deny_unknown_fields`) is ghostwrite-

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sjvrensburg/whetstone](https://github.com/sjvrensburg/whetstone) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
