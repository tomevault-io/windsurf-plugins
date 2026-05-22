---
trigger: always_on
description: Compact guide for AI agents. Full context in CLAUDE.md.
---

# Pixors — Agent Quick Reference

Compact guide for AI agents. Full context in CLAUDE.md.

---

## What is Pixors?

Open-source image editor. Rust workspace + TypeScript MCP server. Pipeline-based GPU/CPU image processing engine with a desktop GUI (Iced) and a headless API (MCP).

---

## Crates at a glance

| Crate | Language | What it owns | What it does NOT own |
|---|---|---|---|
| `pixors-engine` | Rust | `Stage` enum, `Producer`/`Processor`/`Consumer` traits, GPU scheduler, data types (`Tile`, `Buffer`, `Neighborhood`…), runtime, color science types (`ColorSpace`, `TransferFn`, `Matrix3x3`…) | No operations, no image I/O, no app state |
| `pixors-shader` | Slang/Rust | `.slang` GPU shaders + `#[kernel]` proc-macro generated SPV + Rust kernel types | No runtime, no pipeline logic |
| `pixors-shader-macro` | Rust (proc-macro) | `#[kernel]` attribute macro — reads annotation, calls slangc, generates SPV + `GpuKernel` impls | No runtime |
| `pixors-image` | Rust | Image codecs (PNG, TIFF), `Image` struct, `CacheWriter`, pixel types (`Rgba<T>`, `Rgb<T>`, `Gray<T>`…) | No color science, no operations |
| `pixors-ops` | Rust | `Blur`, `Compose`, `MipDownsample`, `MipFilter`, `CacheReader`, `ColorConvert` | No app state, no GUI |
| `pixors-document` | Rust | `Document`, `SessionState`, `PreviewState`, `DocumentMutation`, `History`, `DocumentView`, `EditorState`, `Tab`, actions, `Dispatcher` | No GUI widgets, no wgpu textures, no file dialogs |
| `pixors-desktop` | Rust | Iced GUI, wgpu GPU atlas (`TiledTexture`), screen render (`ViewportSink`), dialogs | No business logic, no pipeline construction |
| `pixors-mcp` | TypeScript | MCP server — calls `pixors-document` headlessly over stdio | No GUI |

---

## Dependency order (no cycles allowed)

```
pixors-engine
    ↑
pixors-shader ──→ pixors-shader-macro
    ↑
pixors-image
    ↑
pixors-ops
    ↑
pixors-document
    ↑
pixors-desktop    pixors-mcp
```

If your change would reverse an arrow, stop — you have a design problem.

---

## Where does new code go?

**New pixel format?** → See "How to add a new PixelFormat" in CLAUDE.md.

**New GPU operation (blur-like)?** → `pixors-ops/src/processor/`, shader in `pixors-shader/shaders/`.

**New image codec?** → `pixors-image/src/{png,tiff}/`.

**New editor action (open, export, filter…)?** → `pixors-document/src/action/actions/`. Must implement `Action` trait. No Iced or wgpu imports.

**New document mutation (undoable edit)?** → `pixors-document/src/mutation/impls.rs`. Implement `DocumentMutation` + `Action` (use `impl_document_action!` macro).

**New UI panel or widget?** → `pixors-desktop/src/components/` or `panel/`. No `EditorState` mutation here — emit a `Msg::Action(…)` instead. See `UI.md` for component guidelines.

**New MCP tool?** → `pixors-mcp/src/`, calls `Dispatcher::dispatch()` on `EditorState`.

**New pipeline stage for tile I/O tied to in-memory cache?** → `pixors-document/src/` (like `ViewportCacheSource/Sink`). Not in desktop — MCP needs these too.

---

## The document/desktop split (most confusing part)

`pixors-document` is the **model**. It has no window, no Iced, no wgpu textures. It can run headlessly (MCP, CLI, tests).

`pixors-desktop` is the **view+controller**. It renders `EditorState` using Iced and uploads tiles to the GPU atlas.

### Three rules

1. **Desktop never writes to Document.** All writes are Action → DocumentMutation. No exceptions.
2. **Desktop never reads Document directly in hot paths.** Reads `DocumentView` (derived cache) or `SessionState`.
3. **Preview lives in SessionState, commit goes to Document.** Slider drags run a preview overlay pipeline (no mutation); release dispatches `CommitBlur` which writes directly to `LayerNode.transforms` and calls `recomposite_current_view()`.

### Decision test

> "Does this code need to know about Iced widgets, wgpu textures, GPU atlases, or file dialogs?"

- Yes → `pixors-desktop`
- No → `pixors-document` (if it's app/action logic) or a lower crate (if it's pure pipeline logic)

---

## Pipeline rules (non-negotiable)

1. **Processors never move data between CPU↔GPU.** The runtime injects `Upload`/`Download` automatically. Trust `context.device`.
2. **Processors never call wgpu directly.** All GPU work goes through `Scheduler`. No `wgpu::Device`, `wgpu::Queue`, or `wgpu::CommandEncoder` in a `Processor`.
3. **`context.device` is set by the compiler (`assign_devices`), not by the processor.**
4. **`Scheduler::download_buffer` does not exist.** Batch GPU→CPU is done by `DownloadProcessor`. Single reads use `Scheduler::read_from_buffer`.

---

## Action pattern

Every state mutation is an `Action`:

```rust
trait Action {
    fn prepare(&mut self, state: &mut EditorState) -> Result<PreparedAction, String>;
    fn apply(&mut self, state: &mut EditorState, status: ActionStatus);
    fn undo(&mut self, state: &mut EditorState);
    fn record_in_history(&self) -> bool;
}
```

For simple document edits, mutations implement BOTH `DocumentMutation` and `Action` (dual-trait pattern). Use `impl_document_action!(T, tab_field)` macro in `mutation/impls.rs`.

---

## Key files

| File | Purpose |
|---|---|
| `pixors-engine/src/stage/node.rs` | `Stage` enum (`Producer`, `Processor`, `Consumer`), `StageHints` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vkeslarek/pixors](https://github.com/vkeslarek/pixors) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
