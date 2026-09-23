---
trigger: always_on
description: > Agent guidance for working effectively in the r2web repository.
---

# AGENTS.md — r2web

> Agent guidance for working effectively in the r2web repository.
> This is a React + TypeScript + Vite application that runs radare2 in the browser via WASI/WASM (Wasmer SDK) with an xterm.js terminal frontend.

---

## Project Overview

**r2web** is a browser-based interface for [radare2](https://rada.re). It lets users upload a binary and interact with a full r2 terminal entirely client-side. The heavy lifting is done by a radare2 WASM build loaded at runtime from GitHub releases.

Key architectural facts:

- Client-side only: no backend required for core functionality, except a small optional proxy for fetching non-default r2 WASM versions.
- The WASM binary is **not** bundled in the repo; it is downloaded on first use and optionally cached in the browser.
- Uses `SharedArrayBuffer`/WASI, so the dev server must send cross-origin isolation headers (`COOP`, `COEP`).

---

## Essential Commands

Use `bun` as the primary package manager (npm/yarn also work as drop-in replacements). All scripts are defined in `package.json`.

| Command | Purpose |
|---------|---------|
| `bun install` | Install dependencies. Also runs `postinstall`, which copies `coi-serviceworker.min.js` into `public/`. |
| `bun dev` | Start the Vite dev server. Serves the app with COOP/COEP headers and proxies `/wasm/*` to `http://localhost:3000`. |
| `bun cc` | Run **both** the Vite dev server and the local WASM proxy server (`api/wasm.cjs`) concurrently. This is the recommended local dev command. |
| `bun run build` | Type-check (`tsc -b`) and build for production. Output goes to `dist/`. |
| `bun run lint` | Run ESLint over the whole project. |
| `bun run preview` | Preview the production build locally. |
| `node api/wasm.cjs` | Start the local proxy server manually on port 3000. Useful if you don't have `bun`. |

### Environment variables for build/deploy

- `VITE_BASE_URL` — base path for production builds. Default is `/`. Example: `VITE_BASE_URL=/online bun run build`.
- `VITE_VERCEL_PROJECT_PRODUCTION_URL` / `VITE_VERCEL_URL` — used at runtime in production to route WASM downloads through the Vercel API (`api/vercel.js`).
- `VITE_WASM_SERVER` — optional override for the WASM proxy base URL in development.

---

## Code Organization

```
/
├── api/                 # Serverless/server helpers for WASM download
│   ├── vercel.js        # Vercel serverless function: downloads r2 release ZIP, streams radare2.wasm
│   └── wasm.cjs         # Local Express proxy for the same purpose
├── public/              # Static assets (favicon, logo, COI service worker)
├── src/
│   ├── main.tsx         # App entry: global styles, viewport lock, router
│   ├── r2tab.tsx        # Core terminal component: xterm + Wasmer instance per tab
│   ├── pages/
│   │   ├── Home.tsx     # Landing page: version picker, file upload, navigate to /r2
│   │   └── r2.tsx       # Main workspace: sidebar, tabs, views, WASM loading
│   ├── store/
│   │   └── FileStore.tsx# Simple singleton holding the uploaded file between pages
│   ├── utils/
│   │   ├── cfgParser.ts # Parses r2 `agfj` JSON into a typed CFG
│   │   └── elkLayout.ts # Runs ELK.js layout on the parsed CFG
│   └── views/
│       ├── CFGView.tsx      # Interactive SVG control-flow graph
│       ├── CodeEditorView.tsx# File editor for /mydir with r2.js completions
│       ├── ConfirmDialog.tsx
│       ├── HexView.tsx      # Paginated hexdump viewer
│       ├── InputView.tsx    # Generic modal input (search, seek)
│       └── StringsView.tsx  # Paginated strings table
```

---

## Architecture & Control Flow

### 1. Boot flow

1. `Home.tsx` lets the user pick a radare2 version and upload a file.
2. The file is stored in `FileStore` (a singleton) and the app navigates to `/r2?version=X&cache=Y`.
3. `r2.tsx` initializes the Wasmer SDK, fetches the requested `radare2.wasm` from GitHub releases (or cache), and calls `Wasmer.fromWasm(buffer)` to create a `pkg`.
4. `R2Tab` creates an `Instance` by running `pkg.entrypoint!.run(...)` with the uploaded file mounted under `./` and a `Directory` mounted as `mydir`.

### 2. Terminal I/O

- `R2Tab` owns an `xterm.js` Terminal.
- User keystrokes are captured via `term.onData`, translated, and written to the instance's `stdin` writer.
- `instance.stdout`/`stderr` are piped to the terminal via `WritableStream`s.
- A local command history and simple line editing are implemented manually inside `onData`.

### 3. Auxiliary views

The sidebar buttons in `r2.tsx` send r2 commands to the active tab's stdin and read results from the mounted `mydir` directory:

- **Strings**: `izj > mydir/.strings`
- **Hexdump**: `px > mydir/.hexdump`
- **Graph**: `aa; agfj > mydir/.graph`

After a short timeout, the file is read from `Directory`, parsed, and passed to the corresponding view. This is the project's standard pattern for extracting structured data from r2.

### 4. File uploads inside a session

Files dropped/selected in `r2.tsx` are written into the active tab's `Directory` via `R2Tab.uploadFiles`. Files ending in `.r2` or `.r2.js` trigger a confirmation dialog; if confirmed, they are uploaded and executed with `. /mydir/<filename>`.

---

## Key Conventions & Patterns

### Styling


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [radareorg/r2web](https://github.com/radareorg/r2web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
