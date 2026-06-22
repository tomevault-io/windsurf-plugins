---
trigger: always_on
description: Fast native macOS git commit/diff tool — an original take on the "commit changes" workflow
---

# Kyde

Fast native macOS git commit/diff tool — an original take on the "commit changes" workflow
familiar from modern IDEs. Goal: **lightning fast**, native, polished and familiar look and
feel. No web, no Electron, no React.

## Hard requirements (the whole point)
- Genuinely fast — native GPU rendering and low input latency are non-negotiable (the
  motivation is JVM/Swing IDEs feeling sluggish for this one workflow).
- Polished, familiar dark look & feel — an original theme tuned to feel at home for IDE
  users (see Theme below). No vendor code or assets copied.
- Side-by-side diff with **word-level inline highlighting** and a **center gutter** whose
  `»` chevrons + checkboxes stage/revert individual hunks (like `git add -p`, IntelliJ-style).
- Folder open + per-file editing with tree-sitter syntax highlighting.

## UI principles (non-negotiable)
- **Every modal is a native OS window (`ModalWindow`), never an in-app overlay.** Rollback,
  Push, Diff, New Branch, **Language Plugins**, **Fonts**, **Clear Data & Restart** — all are
  separate native windows with a real macOS titlebar, opened via `open_modal_window(kind,
  title, w, h, cx)` and dispatched through `ModalKind` → `render_*_body` (each body fills the
  window via `size_full`; the window provides chrome/bg/font). To add a modal: add a
  `ModalKind` variant, a `*_win: Option<WindowHandle<ModalWindow>>` field + `modal_slot` arm,
  a `ModalKind` arm in `ModalWindow::render`, and a `pub(crate) fn render_<x>_body(&mut self,
  cx)`. Do NOT build modal dialogs as `overlay(cx, _)` children of the root. (The fuzzy
  finder and first-run keymap picker are transient *overlays*, not modals — they stay as-is.)
- **Buttons use the shared `btn_primary` / `btn_secondary` helpers** (`render.rs`), never
  hand-rolled. Primary = accent fill + `primary_text`; secondary = transparent + `divider`
  border + `secondary_text`. Caller chains `.on_mouse_down(...)`.

## Stack & why
- **gpui** + **gpui_platform** (Apache-2.0) — Zed's GUI framework. Chosen over Tauri+Monaco
  because the user wants beyond-WebStorm latency, which only a native GPU stack gives.
  Decision was: build FRESH on the gpui crate, STUDY Zed for patterns — do NOT fork Zed
  (Zed editor is GPL-3.0, huge, tightly coupled).
- **git binary, shelled out** — same as Zed's `crates/git`. No libgit2/git2 dependency.
- **similar** (Apache-2.0) — line + word diff. Swap to `imara-diff` (what Zed uses) only if
  large-file diffs lag.

## Layout
```
src/main.rs   entry point + chrome glue: struct Kyde definition, actions!/keymap
              wiring, native menu/dock, ModalWindow, free render helpers
              (overlay/badge/aligned_rows/…), main(). ~1500 lines.
src/app.rs    Kyde controller logic — every non-render method (refresh/select/stage/
              commit/navigation/finder/rollback/…). Sibling of render.rs, so methods
              the view or root calls are `pub(crate)`.
src/render.rs `impl Render for Kyde` + every `render_*` method (the view code, split
              out of main.rs). Child module of the crate root, so it reaches main.rs's
              private Kyde fields/helpers/types directly — only the 4 modal bodies that
              ModalWindow (in main.rs) calls back into are `pub(crate)`.
src/git.rs    Repo: discover/status/base_content/working_content/stage/unstage/
              apply_patch/commit. Pure Rust, shells out to `git`. Stable.
src/diff.rs   FileDiff::compute() → line Hunks + word ranges (two-phase, like Zed/IntelliJ).
              FileDiff::hunk_patch() builds a unified-diff patch for one hunk. Stable.
src/theme.rs  Original hand-authored dark palette (Darcula-family style). Stable.
src/terminal.rs  Embedded PTY terminal (TerminalView entity + TerminalElement), gated
              behind the `terminal` Cargo feature. See "Terminal panel" below.
```

## Theme — runtime config (`src/theme.rs` + `~/.config/kyde/theme.json`)
Colors are a **flat runtime struct** (`theme::Theme`), loaded lazily via `theme::get()`
(`OnceLock`), serialized as hand-editable `"#RRGGBB"` hex. The file **auto-repairs** on load
(`theme::merge`, pure + unit-tested): missing file → write defaults; missing/invalid keys →
filled from defaults; unknown keys → dropped; valid per-key overrides preserved (editing one
color never loses the rest). Rewrites only when something changed. Access anywhere with
`theme::get().<field>` (e.g. `theme::get().primary`). Fonts stay compile-const in
`theme::font` (not themeable): `UI_FAMILY` = **Inter** (all chrome — trees, buttons,
overlays), `FAMILY` = **JetBrains Mono** (code surfaces — diff panes + editor), 13 / 1.2.
Both OFL, bundled in `assets/fonts/`, registered at startup via `main::load_fonts`
(`cx.text_system().add_fonts`). Chrome render fns thread a `ui` family arg; `render_diff`
ignores it and hard-codes `FAMILY`. (SF Mono was rejected — Apple license, not shippable.)

Defaults are an original, hand-authored dark palette in the broad style of modern IDE dark
themes (Darcula-family conventions), tuned for Kyde — not a copied or redistributed theme
file. Key colors and accents:
- `frame_bg` `#0D0E10` — window frame / gaps **behind** the rounded island panels (darkest

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kyle-ssg/kyde](https://github.com/kyle-ssg/kyde) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-22 -->
