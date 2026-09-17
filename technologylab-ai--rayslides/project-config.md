---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

rayslides is a minimalistic slideshow presentation tool written in Zig, using raylib for rendering. It's a port of the original [slides](https://github.com/renerocksai/slides) project, designed for editing, presenting, and PDF-exporting slides on Mac (and other platforms via raylib).

## Build Commands

```bash
# Build the project (add -Dneovim=true on Linux/macOS to compile the embedded
# Neovim editor; use the same flag for `zig build test`)
zig build

# Run with a slideshow file
zig build run -- testslides/test_public.sld

# Run tests
zig build test

# Headless release-resilience and baseline-harness gate
zig build release-confidence

# Opt-in Studio visual/performance baselines (needs ReleaseSafe and Python
# Pillow; --workspace 12 is the strict Aerospace contract on macOS, omit it
# elsewhere). See tests/studio_baselines/README.md.
zig build -Doptimize=ReleaseSafe studio-baselines -- --workspace 12
zig build -Doptimize=ReleaseSafe studio-baselines -- --workspace 12 --scenario compact-properties
zig build studio-baseline-test
# Only after visually reviewing every affected scenario:
zig build -Doptimize=ReleaseSafe studio-baselines-update -- --workspace 12
```

**Requirements:** Zig 0.16.x (minimum 0.16.0, specified in build.zig.zon). Video playback additionally needs `ffmpeg`/`ffprobe` installed at runtime.

## Architecture

### Core Components

- **main.zig** - Application entry point, main loop, input handling (keyboard/mouse), and window management. Contains `AppData` global state struct (`G`), `ExportController` for PDF export, `LaserPointer` for presentation annotations.

- **parser.zig** - Parses `.sld` slideshow text files into slide data structures. Handles directives (`@bg`, `@box`, `@push`, `@pop`, `@slide`, `@pushslide`, `@popslide`), variable substitution (`@let`), and font configuration. The `ParserContext` manages parsing state and template contexts.

- **renderer.zig** - `SlideshowRenderer` pre-renders slides into `RenderElement` lists (background/text/image), then renders them at runtime with coordinate transformation from internal 1920x1080 space to window size.

- **slides.zig** - Data structures: `SlideShow` (collection of slides with defaults), `Slide` (items + per-slide settings), `SlideItem` (background/textbox/img with position, size, color, font), `ItemContext` (parsing context for directives).

- **markdownlineparser.zig** - Parses markdown-like inline formatting within text: `**bold**`, `_italic_`, `~~underline~~`, `` `code` ``, `<#rrggbbaa>colored</>`.

- **fonts.zig** - Font loading and management, supports custom fonts via `@font`, `@font_bold`, etc. directives.

- **texturecache.zig** - Caches loaded image textures to avoid redundant loading.

- **studio.zig** - Studio's interaction and overlay logic: selection, live geometry gestures, semantic property controls, creation tools, base/build/morph scene navigation, the Objects/Properties/Motion inspector (Reveal, State, and Transition sections), the step timeline with its preview transport, and the canvas build badges and morph ghosts. It never rewrites `.sld` text itself; it emits `GeometryCommand`/`SemanticCommand` intentions that the integration layer applies.

- **source_editor.zig** - Guarded, byte-preserving `.sld` rewriting for every Studio command: item/geometry patches, layer moves, duplication, morph-state blocks, and the motion primitives (`setItemReveal`/`removeItemReveal`, `setMorphStateTiming`, `setSlideTransition`/`removeSlideTransition`, `setDeckTransitionDefaults`). Writes minimal source and leaves unedited lines untouched.

- **studio_motion.zig** - Studio chrome motion: per-rectangle hover/press/active glow registry (buttons register themselves while drawing, so `self: Studio`-by-value draw code needs no state), the comet that runs around a hovered button, eased open/close reveals for floating instruments (command palette, reusable picker, grid popover, go-to-slide, prompt, file browser, embedded Neovim, toast, inspector tab wipe), a nesting-safe scissor stack (`pushClip`/`popClip`; Studio never calls `rl.beginScissorMode` directly), and pure fold/slide geometry with unit tests. `main.zig` ticks it once per frame before drawing. Diagnostics captures set `enabled = false`, which snaps every reveal and silences glows so pixel baselines stay deterministic. `RAYSLIDES_MOTION_SLOW=N` stretches every timer N× for reviewing animations.

- **file_browser.zig** - Self-drawn modal file chooser used on every platform for Open deck (Cmd/Ctrl-O, Commands, the welcome chooser) and the image/video Browse… buttons. Allocation-free directory model (listing, extension/hidden/type-ahead filtering, navigation, per-purpose remembered folder) that is unit-tested against a temporary tree, plus raylib input/drawing. `--diagnostics-file-browser` opens it for visual QA captures.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [technologylab-ai/rayslides](https://github.com/technologylab-ai/rayslides) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-17 -->
