---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
./run.sh                  # start everything at http://localhost:8000 (creates/syncs .venv itself)
PORT=9000 ./run.sh        # different port
uv run python -c "..."    # run anything against the project venv
uv add <pkg>              # add a dependency (updates pyproject.toml + uv.lock)
```

`run.sh` scopes `--reload` to `server/` on purpose: watching the whole tree would restart the
server on every upload written into `media/`.

## Environment constraints

These are hard limits on this machine, not preferences — check before proposing a change that
assumes otherwise:

- **No Node/npm, and no `pip`** (`ensurepip` is broken). `uv` is the only route to packages.
- **The frontend has no build step.** `web/` is plain ES modules served straight off disk.
  Introducing a bundler would mean introducing a Node toolchain that isn't there.
- **ffmpeg and ffprobe must be on `PATH`** — the whole server side is built on them.
- Fonts are *discovered* on the machine (`server/fonts.py`), not vendored. The list can differ
  per machine; code must tolerate a font key being absent (`fonts.get` falls back).

## Architecture

### The invariant that matters most

The same image has to come out of **three renderers**:

| Renderer | File | Used for |
|---|---|---|
| GLSL fragment shader | `web/js/gl/renderer.js` | the live preview |
| JS on the CPU | `web/js/sample.js` | TXT / ANSI / HTML / clipboard export |
| numpy | `server/ascii_core.py` | MP4 / WebM / GIF export |

The adjustment, noise-field and glyph-selection maths therefore lives in **one module per
language**:
`web/js/adjust.js` (which also emits the GLSL as a template string) and `server/settings.py`.
**Editing the maths in one place without the other two silently desynchronises exports from
the preview.** Both files say so in their docstrings.

Verified parity: identical glyph choice on 500 randomised inputs, and on a full cell grid
across all eight combinations of dither mode × noise field. On a real frame the browser and
server agree on ~93% of cells and are never more than one ramp step apart, the remainder being
the difference between the browser's downscale and ffmpeg's.

**Two rules decide where a new effect goes.** If a fragment shader can evaluate it per cell
from position and time — the adjustments, the noise field, ordered dithering — write it twice
in `adjust.js` (JS + GLSL) and once in `settings.py`. If it is sequential, it cannot go in the
shader at all: **Floyd–Steinberg and Atkinson run on the CPU** in `computeIndices` /
`compute_indices`, and the finished glyph indices are uploaded as an R8 texture that the
shader reads instead of picking a glyph itself (`uDitherMode == 2`). The grid is only a few
thousand cells, so this still runs per frame during playback.

Anything random must use `hash2`, the exact 32-bit integer hash. **Do not reach for
`fract(sin(...))`** — it differs between GPU vendors and between GPU and CPU, which silently
desynchronises the preview from exports. Only its top 24 bits become a float, so the value is
exactly representable in a shader's 32-bit floats.

### The frame pipeline

Every frame is first reduced to **one average colour per character cell**, and that tiny
`cols × rows` grid is the only per-frame data the renderers see. This is why frame cost is
independent of how many characters are on screen.

- Browser: `drawImage` into a `cols × rows` canvas (`web/js/sample.js`), done by **repeated
  halving** so large reductions don't alias.
- Server: `ffmpeg -vf scale=cols:rows:flags=$SCALE_FLAGS` (`server/media.py`).

`SCALE_FLAGS` is `bilinear` because it was *measured* as the closest match to the browser's
halving chain (7 scalers compared). Changing it re-opens the preview/export parity question.

Two details that exist to protect grid parity:

- The client sends **both `cols` and `rows`** in the export payload rather than letting the
  server re-derive rows from font metrics and land on a different number.
- The backend serves the browser the **same TTF file** Pillow rasterises for exports
  (`/api/fonts/{key}.ttf` + `@font-face`), so glyphs are identical in preview and video.

### Animated input is normalised to WebM

On upload, a GIF is transcoded to VP9 WebM (`server/media.py: normalize_to_webm`). Browsers
cannot seek the frames of a GIF but seek video natively, so after this conversion the frontend
has exactly two source kinds — still image or `<video>` (`web/js/source.js`) — and playback and
scrubbing are one code path. `meta.playback` names the file the browser previews;
`meta.source` is the original, kept for export.

Video seeking depends on HTTP Range, which Starlette's `FileResponse` handles natively. Don't
replace it with a plain byte-stream response.

### Where exports run

Split deliberately, in `web/js/export.js`:

- **In the browser** (instant, no round trip): PNG via `canvas.toBlob` — which is why the GL
  context is created with `preserveDrawingBuffer: true` — plus TXT, 24-bit ANSI, standalone
  HTML and clipboard, all built from `gridToChars`/`gridToColors`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AntonioLAvila/ascii_art](https://github.com/AntonioLAvila/ascii_art) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-18 -->
