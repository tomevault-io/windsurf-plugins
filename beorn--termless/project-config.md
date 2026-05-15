---
trigger: always_on
description: Pluggable headless terminal library for cross-terminal TUI testing. Composable region selectors + matchers. Write tests once, run against any backend.
---

# Termless - Headless Terminal Library

Pluggable headless terminal library for cross-terminal TUI testing. Composable region selectors + matchers. Write tests once, run against any backend.

## Documentation Site

VitePress docs at `docs/` — deployed to termless.dev via GitHub Pages.

- **Source**: `docs/` (edit files here)
- **Config**: `docs/.vitepress/config.ts`
- **Build**: `bun run docs:build` (runs `vitepress build docs`)
- **Build output**: `docs/.vitepress/dist/` (gitignored)
- **Logo**: `docs/public/logo.svg`
- **CI**: `.github/workflows/docs.yml` — auto-deploys on push to main

**Do NOT create or edit `docs/site/`** — docs live directly in `docs/`.

## Packages

| Package                | Description                                                                | Runtime       | Status                 |
| ---------------------- | -------------------------------------------------------------------------- | ------------- | ---------------------- |
| `@termless/core`       | Core: types, Terminal, PTY, SVG/PNG screenshots, key mapping, region views | Bun + Node.js | Active                 |
| `@termless/xtermjs`    | xterm.js backend (@xterm/headless)                                         | Bun + Node.js | Active                 |
| `@termless/ghostty`    | Ghostty backend (ghostty-web WASM)                                         | Bun + Node.js | Active                 |
| `@termless/vt100`      | Pure TypeScript VT100 emulator (zero native deps)                          | Bun + Node.js | Active                 |
| `@termless/vterm`      | Full-featured vterm.js backend (100% terminfo.dev coverage)                | Bun + Node.js | Active                 |
| `@termless/alacritty`  | Alacritty backend (alacritty_terminal via napi-rs)                         | Bun + Node.js | Needs Rust build       |
| `@termless/wezterm`    | WezTerm backend (wezterm-term via napi-rs)                                 | Bun + Node.js | Needs Rust build       |
| `@termless/peekaboo`   | OS-level terminal automation (xterm.js + real app)                         | Bun + Node.js | Active (macOS)         |
| `@termless/vt100-rust` | Rust vt100 crate via napi-rs                                               | Bun + Node.js | Needs Rust build       |
| `@termless/libvterm`   | neovim's libvterm via WASM                                                 | Bun + Node.js | Needs Emscripten build |
| `@termless/kitty`      | Kitty VT parser built from GPL source (not distributed)                    | Bun + Node.js | Needs C build          |
| `@termless/test`       | Vitest integration: matchers, fixtures, snapshots                          | Bun + Node.js | Active                 |
| `@termless/cli`        | CLI + MCP server                                                           | Bun + Node.js | Active                 |

### Runtime Notes

- **PTY support** (`spawnPty` / `terminal.spawn()`) uses Bun's native PTY on Bun, and `node-pty` on Node.js. On Node.js, install `node-pty` as a peer dependency: `npm install node-pty`.
- **Peekaboo** uses OS automation (osascript, screencapture) — macOS only on both runtimes.
- **Pure backends** (xtermjs, ghostty, vt100, vterm) have zero runtime-specific dependencies and work on both Bun and Node.js.
- **napi-rs backends** (alacritty, wezterm, vt100-rust) load native `.node` binaries — work on any runtime that supports N-API.
- **WASM backends** (ghostty, libvterm) require async initialization to load the WASM module.

## Architecture

```
@termless/test (Vitest matchers + fixtures)
  └── @termless/core (TerminalBackend interface + PTY + SVG/PNG + region views)
        ├── @termless/xtermjs (@xterm/headless)
        ├── @termless/ghostty (ghostty-web WASM)
        ├── @termless/vt100 (pure TypeScript — VT100-era)
        ├── @termless/vterm (pure TypeScript — full standards)
        ├── @termless/alacritty (Rust napi-rs)
        ├── @termless/wezterm (Rust napi-rs)
        ├── @termless/vt100-rust (Rust napi-rs)
        ├── @termless/libvterm (C via Emscripten WASM)
        ├── @termless/kitty (C built from GPL source)
        └── @termless/peekaboo (xterm.js + OS automation)
```

## Commands

```bash
bun test                              # Run all tests
bun cli backend list                  # List backends and install status
bun cli backend install [names...]    # Install or upgrade backends
bun cli backend update                # Check upstream versions
bun cli doctor                        # Health check installed backends
```

## Backend Registry

`backends.json` pins backend versions (like Playwright's `browsers.json`). The registry in `src/backends.ts` provides:

- `backend("ghostty")` — async resolution (handles WASM/native init)
- `backends()` — list all backend names
- `isReady(name)` — check if installed and built
- `entry(name)` — get manifest entry for a backend
- `manifest()` — get full manifest
- `buildBackend(name)` — build native/WASM backends

Two ways to choose a backend:

```typescript
// Factory function (explicit, sync)
import { createXtermBackend } from "@termless/xtermjs"
const term = createTerminal({ backend: createXtermBackend() })
```

```typescript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [beorn/termless](https://github.com/beorn/termless) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
