---
trigger: always_on
description: Guidance for Codex working in this repo. Keep this file short — codebase facts belong in code, README, or commit messages, not here. Only document non-obvious gotchas and conventions an agent can't infer in 30 seconds.
---

# AGENTS.md

Guidance for Codex working in this repo. Keep this file short — codebase facts belong in code, README, or commit messages, not here. Only document non-obvious gotchas and conventions an agent can't infer in 30 seconds.

## What this is

BerryCode is a native IDE built specifically for the Bevy game engine. Same stack as the games it edits: **Bevy 0.18 + bevy_egui 0.39 + egui 0.33 + WGPU**. Two binaries ship from the same `berrycode` crate:

- `berrycode` (`src/main.rs`) — the production binary
- `berrycode-egui` (`src/bin/berrycode-egui.rs`) — a thinner harness used in dev / testing

Both wire up `BerryCodePlugin` from `src/bevy_plugin.rs`. **Add new app-level systems to the plugin, not to either bin.** Otherwise `berrycode-egui` silently misses behavior.

## Build / run

```bash
cargo run --bin berrycode               # debug
cargo build --release --bin berrycode   # release
cargo check -p berrycode                # fast iteration when iterating UI code
```

The dev build is large; `cargo check` is the right loop for editing egui UI. The app uses a PID lockfile at `~/Library/Caches/berrycode.lock` (macOS) — if the binary crashes mid-run, delete it before relaunching.

## Operating rules (read first)

These exist to stop the "止まった?" feedback loop. Follow them by default; only break them if the user explicitly asks.

### Never let the user lose the thread

- **Always announce intent before the first tool call.** When the user asks for something, the first thing in the reply is one line stating what you're about to do — not silence followed by tool calls. The user must never see "thinking…" with no follow-up text.
- **Every assistant turn must end with one of three things, never silence:** (a) a concrete result statement, (b) a single specific question that unblocks you, or (c) "next: <X>" when you're handing off to a tool that fires asynchronously. Never end with "確認待ちです" alone — pair it with the specific thing you need.
- **Don't send "ビルド待ち" / "monitoring" filler.** If you've armed a Monitor, the next message should come from the Monitor event, not from you. Filler messages create the appearance of stalls.

### Batching and momentum

- **Batch related edits.** When editing the same file 3+ times in a row, do it inside a single assistant turn — no narration between them. For >5 edits to one file or pervasive structural changes, prefer a single Write.
- **Until-done by default.** When the user says やって / 進めて / 全部直して / これも / 同じやり方で, complete edit → verify → rebuild → relaunch in one turn without checking in mid-flow.
- **One restart per change cycle.** Never `pkill + cargo run` more than once for the same logical change.

### Tool-call ergonomics

- **Monitor timeout default: 60 seconds.** Never use 120s+ unless the cargo build is genuinely slow (cold start, new dependency). Long timeouts produce dangling "Monitor timed out" notifications that read as stalls.
- **Trust `cargo check`, not rust-analyzer diagnostics.** rust-analyzer's `<new-diagnostics>` blocks are often stale right after an edit. Run `cargo check -p berrycode` to confirm; only react to its output.
- **Stale rust-analyzer warnings don't deserve a reply.** If `<new-diagnostics>` shows the same dead-code warnings you've already seen, ignore them silently. Don't quote them back.

### What not to do

- **No mid-flow recap.** Don't summarise what was just done unless the user asks; the diff and the working app are the report.
- **No "止まってない、続行中" reassurance posts.** If the user asks "止まった?" the answer should be a tool call (the next concrete step), not a paragraph explaining you weren't stopped.

## Gotchas you will hit

1. **`PrimaryEguiContext` is spawned exactly once — in the plugin.** `setup_primary_camera` in `bevy_plugin.rs` spawns the window-targeted `Camera2d` with `RenderLayers::none()`, `order: 100`, and `ClearColorConfig::None`. Do **not** also spawn one from `main.rs` — `setup_egui_fonts_and_style` queries with `With<PrimaryEguiContext>` and panics on multiple matches with `Multiple entities fit the query`.

2. **Bevy's default `ClearColor` is pure black.** The egui camera doesn't clear (`ClearColorConfig::None`), so any sub-pixel seam between `SidePanel`s shows through as a black band. The plugin sets `ClearColor` to `#191A1C` (`SIDEBAR_BG`) to hide this. Don't remove it.

3. **`SidePanel` width persists in egui memory by id.** If you change `default_width` / `width_range` / sizing logic for an existing panel, also bump the id (`"sidebar"` → `"sidebar_v2"`). Otherwise users will keep stale persisted widths that may sit outside the new range. Current ids in use: `sidebar_v2`, `ai_chat_panel_v2`, `activity_bar`.

4. **Don't read `ui.available_width()` back into the value driving `exact_width`.** That creates a feedback loop where the panel shrinks by `inner_margin` every frame until clamped. Use `default_width` + `width_range` for resizable panels.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KyosukeIshizu1008/berryscode](https://github.com/KyosukeIshizu1008/berryscode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
