---
trigger: always_on
description: This document provides guidelines for LLM agents to read. It provides conventions for writing code, writing documentation, and collaborating on this project. Please follow these instructions to ensure consistency and maintainability.
---

# LLM agent instructions for Karl2D

This document provides guidelines for LLM agents to read. It provides conventions for writing code, writing documentation, and collaborating on this project. Please follow these instructions to ensure consistency and maintainability.

> Human can read this file too, but it might not be optimized for human consumption. Also, note that no form of vibe coded changes are allowed. You must understand each line submitted in a pull request. You can use an LLM to do code reviews and generate initial implementations, but you _must_ understand the code generated.

## Project Overview
- **Karl2D** is a 2D game development library written in the Odin programming language, licensed under MIT.
- The focus is on being beginner-friendly, using a minimal set of dependencies and minimizing issues when you actually want to ship the game.
- Karl2D usually requires the latest release of Odin.
- The main entry point is `karl2d.odin`, which contains the platform-independent API and core logic.
- Platform and rendering backends are implemented in separate files (e.g., `platform_windows.odin`, `render_backend_gl.odin`).
- See `karl2d.doc.odin` for a full API overview.

## Contribution Guidelines
- **Draft Pull Requests** are always welcome and do not need to follow strict rules.
- When submitting a _ready for review_ Pull Request, you must:
  1. Ensure your code is working and tested.
  2. Submit only complete, non-rudimentary code.
  3. Avoid modifying unrelated code or using auto-formatters (e.g., odinfmt).
  4. If you make unintended changes, revert them in additional commits (squash merges are used).
  5. Regenerate `karl2d.doc.odin` after API changes: `odin run tools/api_doc_builder`.
  6. Follow the code style described below.

## Code Style
- **Tabs, not spaces** for indentation.
- **Max line length:** 100 characters. Use a ruler in your editor. Always split API comment lines that start with `//` at the 100 character ruler. Do not go beyond it!
- **Procedure signatures** that are too long should be split across lines (see `init` in `karl2d.odin`).
- **Spacing:**
  - Place `:` and `=` with consistent spacing as in `karl2d.odin`.
  - Opening braces `{` should be on the same line as the declaration.
- **API Comments:**
  - Use clear, concise comments above procedures and types.
  - Document parameters and return values where appropriate.
- **File organization:**
  - Group related procedures and types together.
  - Use clear section comments as in `karl2d.odin`. The format is three lines: a dash line, a centered text line, and another dash line. The dashes match the width of the text. Example:
    ```
    //-------//
    // INPUT //
    //-------//
    ```

## Architecture Notes
- The core API is in `karl2d.odin`.
- Platform-specific code is in files like `platform_windows.odin`, `platform_linux.odin`, `platform_mac.odin`, `platform_web.odin`.
- Rendering backends are in files like `render_backend_gl.odin`, `render_backend_d3d11.odin`, `render_backend_webgl.odin`.
- The project uses an **interface/chooser pattern** for extensible subsystems:
  - `*_interface.odin` defines a struct of function pointers (the contract).
  - `*_chooser.odin` selects the implementation at compile time based on platform/config.
  - This pattern is used for platforms (`platform_interface.odin`), render backends (`render_backend_interface.odin`, `render_backend_chooser.odin`), and audio backends (`audio_backend_interface.odin`, `audio_backend_chooser.odin`).
- **Render backend selection:** On Windows, the default backend is **D3D11**. On Linux/macOS, it's **GL**. On web, it's **WebGL**. Override with `-define:KARL2D_RENDER_BACKEND=gl` (or `d3d11`, `webgl`, `nil`). The `(GL)` VS Code build tasks use this flag.
- **GL glue files** (e.g., `platform_windows_glue_gl.odin`, `platform_linux_glue_gl_x11.odin`) handle platform-specific OpenGL context setup. These exist alongside the platform files.
- Audio backends follow the same pattern: `audio_backend_waveout.odin` (Windows), `audio_backend_core_audio.odin` (macOS), `audio_backend_alsa.odin` (Linux), `audio_backend_web_audio.odin` (web), `audio_backend_nil.odin` (fallback).
- Audio streaming has platform-split files: `audio_stream_default.odin` (non-web) and `audio_stream_web.odin`.
- File system access is split: `file_system_default.odin` (non-web, uses `core:os`) and `file_system_web.odin` (stub — file reading not yet supported on web).
- `log/log.odin` provides the internal logging utility (`karl2d_logger` package) with `debugf`, `infof`, `warnf`, `errorf`, `fatalf`.
- `default_fonts/` contains `roboto.ttf` (the default embedded font). `default_shaders/` contains HLSL, GLSL, and WebGL GLSL shaders used by render backends. Avoid modifying these unless you are changing rendering behavior.
- `platform_bindings/` contains supplementary platform-specific bindings (subdirs: `linux/`, `mac/`).
- No external windowing libraries (like GLFW) are used; all window/event handling is custom.
- Rendering is batch-based for performance.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [karl-zylinski/karl2d](https://github.com/karl-zylinski/karl2d) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
