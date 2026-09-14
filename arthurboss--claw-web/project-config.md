---
trigger: always_on
description: Guidance for Claude Code when working in this repository.
---

# CLAUDE.md

Guidance for Claude Code when working in this repository.

> **Repository:** all pushes, branches, PRs, and deploys go to `arthurboss/claw-web`. When using `gh`, pass `--repo arthurboss/claw-web`.

## Overview

Captain Claw WASM is a browser-based fork of Captain Claw (1997), focused on optimizing the WebAssembly build. The original archive (`CLAW.REZ`, ~113MB) is **not** bundled: the user uploads it once and it's stored in IndexedDB. Startup is fast (~48MB download, 2-3s) because level assets lazy-load on demand.

## Prerequisites

- **`CLAW.REZ`** — original game archive (~113MB, uploaded at runtime, not in repo).
- **`ASSETS.ZIP`** — custom assets, auto-generated from `Build_Release/ASSETS/`.
- Emscripten SDK (`emsdk/`, gitignored), CMake 4.3.2+, Node.js 18+ (Vite).
- Browsers: Chrome 105+ / Firefox 121+ / Safari 16.4+ / Edge 105+.

## Build

```bash
source ./emsdk/emsdk_env.sh
./build_wasm.sh          # regenerate ASSETS.ZIP, configure, compile, patch SDL2 shaders, relink
cd build && make -j$(nproc)   # C++-only incremental build
rm -rf build && ./build_wasm.sh   # clean build
```

> **SDL2 shader patch** (`patch_sdl2_shaders.sh`, run by `build_wasm.sh`) is required for WebGL: it disables `GL_OES_EGL_image_external` and swaps `samplerExternalOES` for `sampler2D`. A hand-rolled `cmake`/`make` build must run it after the first `make`, then clear `emsdk/upstream/emscripten/cache/build/sdl2` and rebuild.

## Running Locally

```bash
yarn dev                 # Vite at http://localhost:5173/
```

Vite gives hot reload and rewrites `/` → `/claw-web.html``. Serve over `localhost` (a **secure context**), which Keyboard Lock and Web Audio / AudioWorklet require; a plain-HTTP LAN IP breaks those APIs.

## Deployment (GitHub Pages)

Deploys are manual via `scripts/`. Both environments share the `gh-pages` branch: **production at root**, **staging under `/staging/`**. Neither script can clobber the other (prod excludes `staging/` from `--delete`; staging writes only `/staging/`). Both verify the WASM artifacts are fresh (guards the `ASM_CONSTS` crash from a stale loader+wasm pair) and self-clean their temp worktree.

- **Production:** <https://arthurboss.github.io/claw-web/> — `./scripts/deploy-prod.sh ["msg"]`
- **Staging:** <https://arthurboss.github.io/claw-web/staging/> — `./scripts/deploy-staging.sh ["msg"]`

**Default workflow:** deploy every new branch to **staging first** (safe — never touches prod), test on a real device, then merge and run `deploy-prod.sh` to promote. Skip only for deploy-only/non-visual changes.

> **SW cache version rule:** any PR that modifies a file listed in `SHELL_ASSETS` in `sw.js` (i.e. `claw-web.html`, `game-init.js`, any `.js` bridge, `site.webmanifest`, icons, or `preview.png`) **must** bump `CACHE_VERSION` in `sw.js` (e.g. `v5` -> `v6`). Without a bump, returning users with a cached SW will not receive the updated files. Also ensure any new `Build_Release/` file served to the browser is added to `SHELL_ASSETS`.

**Env isolation** (prod + staging share one origin, so storage is scoped deliberately):

- **SW cache** named per scope (`openclaw::<scope>::<version>`) — a version bump in one env can't delete the other's cache; kill-switch teardown is scoped too.
- **`localStorage`** namespaced by scope: prod keeps bare keys (preserves existing saves), `/staging/` gets a `staging:` prefix. A `Storage.prototype` shim inlined in `<head>` does this — it **must** stay inline (runs before the save glue baked into the WASM runtime), unlike the external `sw-register.js` / `env-marker.js`.
- **`IndexedDB`** (`CLAW.REZ` + assets) is intentionally **shared** — staging needs no re-upload.
- Gold **STAGING** badge marks non-production builds.

## Architecture

### Layout

```text
OpenClaw/Engine/       # Core C++ engine
  Actor/ Audio/ Events/ GameApp/ Physics/ Resource/ Scene/ UserInterface/ Process/ Logger/ Util/
  Graphics/WASM/       # WebGL renderer (Graphics/Generic/ = platform-agnostic abstraction)
libwap/ Box2D/ ThirdParty/     # WAP parser, physics, deps (TinyXML, FastDelegate, sigc++)
Build_Release/         # Output: openclaw.{wasm,js,data,html}, config.xml, *-bridge.js, ASSETS/, ASSETS.ZIP
scripts/ docs/ emsdk/(gitignored)
```

### Lazy-loading

Startup loads only menu/UI assets (~150ms). Level assets and level metadata (XML) load on first entry to each level; a resource cache (default 150 items, in `config.xml`) holds hot assets. Only visited levels consume memory.

### Assets & resources

- First run: no `CLAW.REZ` in IndexedDB → upload UI → store (~113MB). Every load: retrieve → mount to FS.
- `ASSETS.ZIP` is preloaded with the WASM (<1MB) and **overrides** `CLAW.REZ` files with matching paths.
- Resource paths are **case-sensitive, forward-slash**: `/STATES/MENU/*`, `/GAME/IMAGES/MENU/*`, `/GAME/FONTS/*`, `/LEVEL1..14/*`.
- Level metadata XMLs are gzip-compressed inside `ASSETS.ZIP`; `XmlLoader.cpp` decompresses transparently. Edit via `scripts/decompress_metadata.sh` → edit → `scripts/compress_metadata.sh`.

### C++/JS bridge


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [arthurboss/claw-web](https://github.com/arthurboss/claw-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
