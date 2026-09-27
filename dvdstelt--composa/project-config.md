---
trigger: always_on
description: Composa is a from-scratch implementation of the macOS image editor [Compositor](https://github.com/robbietilton/Compositor). The upstream app is Swift on AppKit, SwiftUI, CoreImage, Metal and Vision, none of which are portable, so nothing is shared at the source level: this repository reimplements the same feature set in C# on .NET 10, Avalonia 12 and SkiaSharp 3. It is developed on Linux and released for Linux and Windows; macOS is planned, so keep new code free of platform assumptions. CI runs
---

# Agent notes for Composa

## What this is

Composa is a from-scratch implementation of the macOS image editor [Compositor](https://github.com/robbietilton/Compositor). The upstream app is Swift on AppKit, SwiftUI, CoreImage, Metal and Vision, none of which are portable, so nothing is shared at the source level: this repository reimplements the same feature set in C# on .NET 10, Avalonia 12 and SkiaSharp 3. It is developed on Linux and released for Linux and Windows; macOS is planned, so keep new code free of platform assumptions. CI runs the whole suite on Windows as well.

## Layout

- `src/Composa.Core`: everything that is not UI. Document model, compositor, selections, brush engine, filters, adjustments, text layout (`Text/TextLayout`, `Text/TextEditor`), layer effects (`Rendering/LayerEffectsRenderer`), file IO (including the Photoshop reader in `IO/Psd`) and `EditorSession` (every editing command, split into partial files by area). It must never reference Avalonia.
- `src/Composa.App`: the Avalonia desktop app. UI is built in C# (no XAML). `CanvasView` owns viewport, overlays and pointer tools (text editing in `CanvasView.Text`, rulers and guides in `CanvasView.Guides`); `LayersPanel` the layer stack and effect rows; `MainWindow.*` the menus, shortcuts (a `Shortcut` table for menu commands and tool keys, rebindable through `ShortcutsDialog`), tabs, options bar and file handling.
- `tests/Composa.Core.Tests`: xUnit tests that drive `EditorSession` directly.
- `tests/Composa.App.Tests`: Avalonia headless tests with real Skia rendering. They save screenshots to `artifacts/screenshots/` (git-ignored), which is the way to check UI changes visually without a display.

## Rules that keep the editor correct

- Bitmaps are immutable once they are part of a committed document. An edit copies the bitmap, changes the copy and swaps it into the layer. Undo snapshots (`Document.Clone()`) share bitmaps with the live document, so mutating or disposing a committed bitmap corrupts history and can crash in native code. Only dispose bitmaps you created during the same uncommitted edit.
- Color bitmaps are `Rgba8888` premultiplied; masks and selections are `Alpha8`. Create them through `Pixels.NewColor` and `Pixels.NewMask`.
- After changing a bitmap's pixels in place (only allowed during an uncommitted edit) call `Pixels.Invalidate(bitmap)` so cached `SKImage` wrappers are dropped.
- Every user-visible change goes through `EditorSession.Begin`/`Commit`/`Cancel` (or `Apply`) so it is undoable, then raises `Invalidate`/`InvalidateAll` and `LayersChanged` as needed.
- `DocumentRenderer.Render` splits large areas into bands rendered in parallel. Anything added to the render path must be a per-pixel operation that does not depend on neighbouring bands, or it must report a `SamplingMargin` (as the blur adjustments do): the renderer then draws each band with a halo that wide around it and leaves the halo out again, `EditorSession.AffectedArea` grows invalidations by it, and the halo stops at the canvas, whose edges a spatial adjustment must continue (clamp) rather than fade.
- Never invalidate a visual or raise layout-affecting events from inside `CanvasView.Render`.
- Avalonia's X11 backend sends text input only for a key press nobody marked handled. Key handlers must leave printable keys unhandled while text is being typed; the headless tests feed text directly, so check `e.Handled` in a test when touching key routing.
- Layer lists are stored bottom to top; the Layers panel shows them reversed.
- Layer effects are drawn from an image the renderer caches per (pixels, mask, effects); the image already has the mask applied, so `RenderUnit` must not apply the mask again when effects are present. `Layer.VisibleBounds` includes the effects' margin and is what invalidation uses.
- Text layers are re-rendered from their `TextStyle` through `TextLayout` on every change. The layout's character positions are the caret geometry; keep drawing and layout in that one class so what is typed is what is rendered.
- `EditorSession.TextEdit` is an open edit (a `Begin` without `Commit`); anything that starts another edit or changes the tool or layer selection must go through `FinishText`, which `FinishInteraction` and `SelectLayer` already do.
- View options (rulers, grid, guides, snapping) live on `EditorSession.View` and are carried from tab to tab like the tool settings; guides themselves are document data and undo with it. The window copies them, Auto Select, the transform controls and the pixel grid into `Settings` on every change (`RememberToolSettings`), and the first document of a launch starts from `Settings.View`, so they survive a relaunch. Tests run with `Settings.Persist` off and read the window's `Settings` back.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dvdstelt/Composa](https://github.com/dvdstelt/Composa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
