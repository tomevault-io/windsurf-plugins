---
trigger: always_on
description: Preview and inspect local Markdown files through a fast ByteMD + Vite browser bridge controlled by Playwright. Use when the user wants an Overleaf-like workflow for Markdown: load a local .md file, render Mermaid/math/tables/code in Chrome, ask the user for visual feedback, inspect rendered DOM, map rendered blocks back to likely Markdown source lines, or iteratively edit Markdown with Codex/Claude assistance.
---


# Markdown Preview Bridge

Use this skill to provide an Overleaf-like loop for local Markdown:

```text
local .md -> ByteMD/Vite browser preview -> Playwright DOM inspection -> Codex edits local .md -> reload/recheck
```

## Core Design

- Keep the local Markdown file as the source of truth.
- Treat this as a trusted local development bridge, not a shared web service or collaborative editor.
- Use a local Vite dev server with ByteMD `Editor` + `Viewer`.
- Keep the left pane as source editing only and the right pane as rendered review. ByteMD `Editor` must run with `mode="tab"` to avoid rendering a duplicate split preview inside the editor pane.
- Keep page-level scrolling disabled. The editor pane and rendered pane should own their scrollbars; otherwise the user sees duplicated browser/pane scrollbars.
- Enable simple scroll-ratio sync between the source editor and rendered pane by default. It is a navigation aid, not exact source mapping.
- Use official ByteMD plugins for GFM, Mermaid, and math.
- Expose a lightweight DOM bridge at `window.__mdPreviewBridge` so Playwright does not need to snapshot very large DOMs.
- Set the browser title to `<markdown-file-name> - Markdown Preview Bridge` so agents can identify and focus the right tab/window while keeping a stable search suffix.
- Show only the Markdown filename in the header. Put the full absolute path in a hover/focus overlay below the filename; long full paths are too noisy in normal review mode.
- Prefer bridge diagnostics over full `playwright-cli snapshot` for large documents.

## Prerequisites

- `uv` for reproducible execution of the bundled Python launcher and validation scripts.
- Python 3.10 or newer for the bundled launcher scripts. `.python-version` pins the local dev/test baseline.
- Node.js and npm for the Vite preview app. The launcher installs app dependencies into a cache directory when needed.
- `playwright-cli` and a local Chrome/Chromium for browser operation.
- `xdotool` only when headed browser window geometry/focus must be normalized on Linux/X11. Page and DOM inspection should still use Playwright.

## Trust Boundary

- Bind the preview server to `127.0.0.1` unless there is a deliberate reason to expose it.
- Use `--root` to restrict which Markdown files the bridge may read or write. If omitted, root defaults to the input file's parent directory.
- Only Markdown-like extensions are allowed by default: `.md`, `.markdown`, `.mdown`, `.mkd`.
- Local Markdown images are served through `/api/asset` only when their real path stays under `--root` and their extension is in the image asset allowlist.
- Read-only mode is the default. Use `--allow-write` only when browser-side save is needed.
- Do not treat this as a safe renderer for untrusted Markdown. Raw HTML, Mermaid, KaTeX, and plugin behavior should be considered trusted-local-input concerns.

## Quick Start

Start a preview server for a local Markdown file:

```bash
uv sync --frozen
uv run --frozen scripts/preview.py \
  --file /absolute/path/to/document.md \
  --root /absolute/path/to/repo-or-doc-root \
  --port 8777
```

Run commands from the skill root, or resolve `scripts/preview.py` relative to this `SKILL.md`. If `just` is installed, the root `justfile` exposes the common `uv run` wrappers.

Markdown hot reload is enabled by default. Disable it only when file watching is undesirable:

```bash
uv run --frozen scripts/preview.py \
  --file /absolute/path/to/document.md \
  --port 8777 \
  --no-watch
```

Enable browser-side saving only when needed:

```bash
uv run --frozen scripts/preview.py \
  --file /absolute/path/to/document.md \
  --root /absolute/path/to/repo-or-doc-root \
  --port 8777 \
  --allow-write
```

Relative Markdown image links are resolved from the current Markdown file's
directory and served through `/api/asset` if they stay under `--root`. Use
`--asset-ext` to change the allowed local image extensions.

Open the preview with Playwright:

```bash
playwright-cli -s=md-preview open http://127.0.0.1:8777/ --browser chrome
```

For a visible browser:

```bash
playwright-cli -s=md-preview-visible open http://127.0.0.1:8777/ --browser chrome --headed
```

For a visible browser with normalized geometry, prefer the bundled wrapper:

```bash
uv run --frozen scripts/open_visible.py \
  --url http://127.0.0.1:8777/ \
  --session md-preview-visible \
  --width 1600 \
  --height 1000
```

Override geometry per environment with flags or environment variables:

```bash
MD_PREVIEW_WINDOW_X=80 \
MD_PREVIEW_WINDOW_Y=40 \
MD_PREVIEW_WINDOW_WIDTH=1600 \
MD_PREVIEW_WINDOW_HEIGHT=1000 \
uv run --frozen scripts/open_visible.py
```

If using raw `playwright-cli`, normalize after launch:

```bash
window_id=$(xdotool search --onlyvisible --name 'Markdown Preview Bridge' | tail -n 1)
xdotool windowmove "$window_id" <x> <y> windowsize "$window_id" <width> <height>
```

## Switching Files


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yuki-inaho/markdown-preview-bridge](https://github.com/yuki-inaho/markdown-preview-bridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
