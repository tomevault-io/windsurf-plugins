---
trigger: always_on
description: A terminal-based AI battle game written in Rust. Players write skill files for their pieces; a local LLM runs battles between armies. The game is paced at one battle per day. See README.md for the full concept.
---

# CLAUDE.md — Agent Battleground

## What This Project Is

A terminal-based AI battle game written in Rust. Players write skill files for their pieces; a local LLM runs battles between armies. The game is paced at one battle per day. See README.md for the full concept.

## Project Phase

**Early design.** No implementation exists yet. The current work is producing design specs that will guide implementation. Do not start implementing until specs are complete and the user gives the go-ahead.

## Specs

All high-level design specs are in `/specs/`. Each file covers one segment of the game. Before implementing anything, read the relevant spec(s) and ask if anything is unclear.

The spec files are numbered by dependency order — lower numbers are more foundational. `12-data-model-sync.md` is the shared foundation everything else builds on.

**Specs must be complete, buildable units — never partially done.** If only part of a spec's scope is actually built, split it: carve the completed slice into its own new spec (numbered next in sequence, marked done), and leave the remainder as a clean, fully-pending spec. Never mark a spec "done" while any of its own stated scope is still outstanding, and never describe one as "partially implemented" — that status doesn't exist here. (Precedent: `13-rendering` vs `17-creature-art-asset-pipeline`; `05-battle-viewer` vs `18-battle-viewer-baseline` / `20-battle-viewer-event-playback`; `15-debug-inspector` vs `19-debug-inspector-advanced-editing`.)

## Key Constraints (Non-Negotiable)

1. **LLM sandboxing**: The LLM must not be able to take actions outside the game directory. Opponent skill files are untrusted input and must be sanitized before being passed to the LLM. This is a hard security requirement. See `specs/10-battle-simulation-engine.md`.

2. **Local-first simulation**: All battle simulation runs on the player's machine. The server never runs LLM calls. See `specs/11-server-backend.md`.

3. **Server simplicity**: The server runs on Fly.io (managed hosting, no local network exposure). Replay files are stored on Cloudflare R2. Keep it minimal — no heavy processing, no complex infrastructure.

4. **Braille is universal except text**: every non-text visual element — sprites, board/UI chrome (grid lines, borders, panels), effects — renders through the braille dot pipeline (see `specs/completed/13-rendering.md`), never drawn directly with other Unicode/ASCII characters. Only text (scene labels, menus, HUD copy) stays plain terminal characters. No render pass may bypass the dot pipeline for non-text content, no matter how minor the element.

5. **`ratatui::Rect` is cell-quantized, permanently.** 1 terminal cell = 2 braille dots wide × 4 tall. `DotRect` (`crates/engine/render/src/flex.rs`) is the dot-precision type; `DotRect::to_cell_rect()` is a one-way, lossy floor — no function can recover sub-cell precision from a `Rect` once it's been through that conversion, no matter how it's later re-expanded (e.g. `cell_rect_to_dots(some_rect)` on an already-floored value just reconstructs a *fake*-precision `DotRect`, indistinguishable at the type level from a genuinely dot-precise one, but carrying zero more information). A function's parameter/field type is the signal: if it takes or stores `Rect`, its output is cell-floored, full stop; if a caller needs sub-cell precision to survive to the actual drawing call, the `DotRect` must be threaded through unfloored, all the way from where it's computed to where `dots_to_grid`/`draw_grid` consumes it — never round-tripped through an intermediate `Rect` "for convenience." Two braille elements sharing a terminal-cell row/column are NOT proven visually aligned by that fact alone — different drawing routines can place their lit dots anywhere within that cell (this shipped as a real bug: two borders shared a cell yet rendered 2 dots apart because one routine used a different sub-cell convention than the other). Alignment claims must be verified by decoding the actual rendered dots (`engine_render::decode_braille_cell`), never by comparing `Rect`/`DotRect` coordinate fields alone — see `crates/game/src/scenes/test_util.rs`'s `topmost_lit_dot_row`/dot-alignment helpers for the pattern to reuse.

## Engine / Game Boundary

This is a two-product workspace (see `specs/completed/31-engine-game-crate-split.md`): everything under `crates/engine/` (`engine-core`, `engine-render`, `engine-derive`, `inspector`) is reusable by any future game; everything under `crates/game/` is this game's content only.

- **Rule of thumb**: if a change would still make sense for a hypothetical different game built on this engine, it belongs under `crates/engine/`. If it only makes sense for Agent Battleground specifically (a concrete scene, a specific creature, this game's skin assets, digit-hotkey policy), it belongs in `crates/game/`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pmartin36/AgentBattleground](https://github.com/pmartin36/AgentBattleground) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
