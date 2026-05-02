---
trigger: always_on
description: > **This file is the single source of truth for project-level conventions, architecture, and workflows.**
---

# AGENTS.md — beautiful-mermaid-cli

> **This file is the single source of truth for project-level conventions, architecture, and workflows.**
> All future updates to project-level rules MUST land here. `CLAUDE.md` is intentionally kept as a one-line
> pointer to this file and should not be expanded. User-global instructions (`~/.claude/CLAUDE.md`) cover
> personal preferences (language, signing, tool-call hygiene) and remain disjoint from this file.

## Project Overview

A command-line wrapper around [`beautiful-mermaid`](https://github.com/lukilabs/beautiful-mermaid) that turns Mermaid diagrams into SVG / PNG / ASCII output.

- **Package name**: `beautiful-mermaid-cli`
- **Bin**: `bm` (primary), `beautiful-mermaid` (fallback)
- **Repo**: `github.com/okooo5km/beautiful-mermaid-cli`
- **License**: MIT
- **Author**: okooo5km(十里) <yetiannow@gmail.com>

## Tech Stack

- Language: TypeScript (ESM)
- Runtime: Node.js ≥ 20 / Bun ≥ 1.0 (dual support)
- Build: `tsc` → `dist/`
- Render core: `beautiful-mermaid`
- PNG: `@resvg/resvg-wasm` (optional dep, WASM) — loaded via `await import()` in `src/core/render-png.ts`; the wasm binary is located at runtime via `createRequire(import.meta.url).resolve('@resvg/resvg-wasm/index_bg.wasm')` and read with `fs.readFile`, so layout works under npm, pnpm, and Bun.
- CLI: `commander`
- Test: `vitest`

## Project Structure

```
src/
├── cli.ts                # Entry, shebang + commander wiring
├── commands/             # Subcommands (render / ascii / themes)
├── core/
│   ├── options.ts        # Theme + flag → RenderOptions builder
│   ├── render-svg.ts     # SVG pass-through to beautiful-mermaid
│   ├── render-ascii.ts   # ASCII / Unicode renderer
│   ├── render-png.ts     # SVG → PNG via resvg-wasm (uses svg-flatten + fonts)
│   ├── svg-flatten.ts    # CSS var() / color-mix() → concrete hex (PNG-only)
│   └── fonts.ts          # System font probing, returns Uint8Array buffers
├── io/                   # input.ts (file/stdin/-c), output.ts
└── utils/                # format inference, error formatting
tests/
└── fixtures/             # Sample .mmd files per diagram type
doc/                      # Design docs (architecture, theming, png)
skills/
└── beautiful-mermaid/    # Claude Agent Skill (SKILL.md + reference.md)
                          # Auto-discovered by `npx skills add okooo5km/beautiful-mermaid-cli`.
                          # Not bundled in the npm tarball — agents pull it from GitHub.
```

## PNG rendering pipeline (resvg-wasm constraints)

PNG output flows through `@resvg/resvg-wasm`, which has two non-obvious
limitations that bit us in v0.1.0 and shaped the current pipeline:

1. **CSS Color L4/L5 unsupported.** beautiful-mermaid's SVG output uses
   `var(--xxx)` and `color-mix(in srgb, ...)` for every color. resvg
   silently falls back to black on any unresolved color expression →
   the result is huge black rectangles with no text. Browsers (and
   macOS Preview) handle this fine, but resvg cannot. Fix: `src/core/svg-flatten.ts`
   walks the SVG once before render, resolves all `var()` and `color-mix()`
   to concrete hex literals, strips `@import url(...)` (resvg cannot fetch
   network resources), and rewrites `font-family` to a name we have actually
   loaded. Only the PNG path is flattened; SVG output is the original
   beautiful-mermaid string.
2. **System font loading is broken in wasm.** The `loadSystemFonts`,
   `fontDirs`, and `fontFiles` options on `Resvg` silently fail in the
   wasm runtime because wasm has no filesystem enumeration. The native
   `@resvg/resvg-js` package supports them, but we deliberately stick
   with wasm to keep "no native build" guarantees. The only working
   path for fonts is `font.fontBuffers: Uint8Array[]` — pre-read font
   files in JS and hand the bytes over. `src/core/fonts.ts` probes a
   short, per-OS list of well-known system font paths (macOS Helvetica,
   Linux DejaVu / Liberation, Windows Arial / Segoe) and caches the
   loaded buffers per process. If nothing exists, text won't render but
   the rest of the diagram will — graceful degradation.

**Consequence**: do NOT switch to `@resvg/resvg-js` to "simplify" font
loading. The wasm path is intentional. If a future version of resvg-wasm
adds CSS L5 support upstream, `svg-flatten.ts` becomes vestigial and can
be deleted in one shot.

## Agent Interface Contract

Since v0.2.0, every subcommand accepts `--json` for machine-readable output.
Treat this as a **stable contract**:

- **stdout** carries JSON success payloads; **stderr** carries JSON error payloads.
  Never mix.
- Each payload is one line of JSON terminated by `\n`. The first key is always
  `"schema_version": 1`.
- Exit codes are part of the contract (see `src/utils/errors.ts`):
  `0` success, `1` unclassified, `2` usage / unknown theme / guard violation,
  `3` parse error, `4` I/O error.
- `--json` mode emits **no ANSI escapes** (errors skip `picocolors`).
- `--json` is opt-in; no flags change default human-facing output.

What is guaranteed not to break inside `schema_version: 1`:

- Existing field names and types in success payloads (`themes`, `format`,
  `output`, `bytes`, `dimensions`, `svg`, `text`, `lines`, `theme`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [okooo5km/beautiful-mermaid-cli](https://github.com/okooo5km/beautiful-mermaid-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-28 -->
