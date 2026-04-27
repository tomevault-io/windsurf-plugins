---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**embr.el** is an Emacs browser that uses headless Chromium (via CloakBrowser) as its rendering engine. Emacs acts as the display server, while a Python daemon handles browser automation.

## Architecture

Client-server over JSON lines on stdin/stdout:

```
Emacs (embr.el) ←→ JSON over stdin/stdout ←→ Python daemon (embr.py)
  UI / keybindings                            Playwright/CloakBrowser browser control
  Frame display                               CDP screencast or screenshot capture
  Canvas or JPEG rendering                    JPEG frames → file or Unix socket
```

**embr.el** (~2200 lines): Emacs Lisp major mode. Process management, async JSON protocol, two render backends (default + canvas), two frame sources (screencast + screenshot), link hints, tabs, bookmarks, zoom, mute, reader mode, page info, incognito mode.

**embr.py** (~1350 lines): asyncio daemon using CloakBrowser (Playwright API). Browser commands, CDP screencast, screenshot capture loop, canvas frame socket, domain-level ad blocking, extension loading (uBlock Origin, Dark Reader), incognito temp profiles.

**setup.sh** (~110 lines): Creates Python venv at `~/.local/share/embr/.venv/`, installs `cloakbrowser[geoip]`, downloads browser. Separate flags for blocklist, uBlock Origin, and Dark Reader (downloaded from GitHub releases API). Builds in temp venv and swaps atomically.

## Frame Pipeline

Two frame sources (`embr-frame-source`):

- **`'screencast`** (default, recommended): CDP `Page.screencastFrame` pushes frames from Chromium. Python decodes base64 JPEG data and writes to disk or sends over Unix socket.
- **`'screenshot`**: Python polls `page.screenshot()` in a loop at target FPS. Older, slower.

Two render backends (`embr-render-backend`):

- **`'default`**: Python writes JPEG to a temp file (`/tmp/embr-frame.jpg`), renames atomically. Emacs reads the file via `insert-file-contents-literally` and displays with `create-image`. Render timer batches frames.
- **`'canvas`**: Python sends JPEG bytes over a Unix socket with a binary header (seq, width, height, length). Emacs native C module (`embr-canvas-blit-jpeg`) decodes and blits directly to a canvas pixel buffer. Requires canvas-patched Emacs.

## Buffer-Local State and Multi-Session

All per-session state is buffer-local in `embr-mode`. This enables normal + incognito sessions running simultaneously in separate buffers with separate daemon processes.

Key patterns:
- `embr--process`, `embr--callback`, `embr--frame-path`, `embr--active-backend`, all timers, all canvas state are buffer-local (set via `setq-local` in `embr-mode`)
- Process filter (`embr--process-filter`) routes output to the correct buffer via `(process-get proc 'embr-buffer)` + `with-current-buffer`
- Canvas socket filter (`embr--canvas-socket-filter`) uses the same routing pattern
- Hover and render timers capture the owning buffer in a closure at creation time and use `with-current-buffer` in the tick function
- `embr--default-display-frame` captures `embr--frame-path` in a let binding before entering `with-temp-buffer` (buffer-local vars are not visible inside `with-temp-buffer`)
- Each canvas image gets a unique `canvas-id` derived from the buffer name

Incognito mode (`embr-browse-incognito`) uses the same `embr-mode` with `embr--incognito-flag` set. The daemon gets `EMBR_INCOGNITO=1` env var, uses `tempfile.mkdtemp()` for its profile, separate frame/perf-log paths, and `shutil.rmtree()` on quit.

## Key Design Patterns

- **JSON line protocol**: Each message is a single JSON line. Commands from Emacs have a `cmd` field. Responses include `url`, `title`, and optionally `error`. Use `json-serialize` / `json-parse-string` (Emacs 30.1+ native C JSON, not the old `json.el`).
- **Frame batching**: Process filter stashes latest frame in `embr--pending-frame`. Render timer picks it up, skipping intermediate frames if Emacs can't keep up.
- **Async with callbacks**: `embr--send` dispatches commands with optional callback. `embr--send-sync` blocks via `accept-process-output` for synchronous results.
- **Dual click modes**: `atomic` defers mousedown until drag is detected (better iframe compat); `immediate` sends mousedown instantly.
- **Ad blocking**: Domain-level route interception from blocklist.txt (~82K domains).
- **Extension loading**: Python collects extension dirs into a single `--load-extension=dir1,dir2` Chrome arg. Extensions auto-detected at startup if present.
- **Transient dispatch**: `C-c` opens a transient menu (`embr-dispatch`). Top-level bindings shown via `C-c ?` (`embr-dispatch-keys`). Both are `transient-define-prefix` forms.
- **kill-buffer-hook**: `embr--kill-buffer-cleanup` sends quit to the daemon when the buffer is killed by any means, preventing orphan processes.
- **Shared init params**: `embr--build-init-params` builds the init command alist from defcustom values. Used by both `embr-browse` and `embr-browse-incognito`.

## Development Notes

- No formal test suite. Testing is manual via interactive Emacs commands.
- No linter/formatter config. Emacs Lisp follows GNU conventions; Python is PEP-ish.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [emacs-os/embr.el](https://github.com/emacs-os/embr.el) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
