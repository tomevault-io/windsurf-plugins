---
trigger: always_on
description: Compact, high-signal notes for OpenCode sessions. Architecture, threat model, and design specs all live **in this file** (`CLAUDE.md` is a deprecated stub that redirects here).
---

# AGENTS.md — MudrikNow (hoverbuddy)

Compact, high-signal notes for OpenCode sessions. Architecture, threat model, and design specs all live **in this file** (`CLAUDE.md` is a deprecated stub that redirects here).

## Where the rules live

- **Behavioral guidelines** (think-before-coding, simplicity, surgical changes, goal-driven execution, the manual-approval gate, update-summary format, context-gathering, explaining concepts, TODO discipline) live in the **global** `~/.config/opencode/AGENTS.md` and are auto-loaded every session — **not duplicated below**. That file is the source of truth. (`.opencode/instructions.md` in this repo is a stray duplicate; do not rely on it.)
- **Approval gate (reinforced):** never commit, push, tag, publish, release, or delete without explicit per-request owner approval. This repo ships a Windows installer via tag-push CI (`release.yml`), so the gate is high-stakes here.
- **`context-index.md`** — map of file → purpose → when-to-read. Read it before any non-trivial task; append when a file proves useful.
- **`open-items.md`** — running TODO list. Write there when told to "add to todos"; don't implement unless asked.

## Build & run

- `npm run build` — webpack bundles into `dist/`. **Required before launch** because `package.json > main` is `dist/main.js`.
- `npm start` — build + launch (`webpack && electron .`).
- `npm run dev` — webpack watch mode. Re-run `electron .` manually to pick up main/preload changes; renderer changes hot-reload on window reload.
- `npm test` — vitest, `src/**/*.test.ts` only. No linter or formatter is configured.
- `npm run pack:dir` — unsigned unpackaged build (`release/win-unpacked/`). Faster than `dist` for manual QA.
- `npx tsc --noEmit -p .` — standalone typecheck. CI runs this **before** `npm run build`.
- `npx vitest run <path>` — run a single test file (e.g. `src/main/action-executor.test.ts`).

### Build prerequisites (Windows only)

- **Node.js** — any recent LTS (20.x–24.x tested). `package-lock.json` was generated with a specific Node version; if `npm install` fails on native modules, remove `package-lock.json` and retry.
- **Visual Studio "Desktop development with C++" workload** — required because `robotjs` and `koffi` are native C++ modules. `node-gyp` needs the VC++ toolset + Windows SDK. Without it, `npm install` will fail at the `robotjs` compile step with `Could not find any Visual Studio installation to use`.
- **Do NOT use `npx electron .`** directly — it pulls the latest remote Electron version instead of the local `devDependency`. Always use `npm start` or `npm run build && electron .`.

### Runtime dependency (not bundled)

- **`opencode-ai` must be installed globally** — MudrikNow spawns the `opencode` CLI binary at runtime. Install via `npm i -g opencode-ai`. The app searches `%APPDATA%/npm/node_modules/opencode-ai/bin/` for both `opencode.exe` (native, ≥1.15.x) and `opencode` (JS shim, ≤1.14.x).

### Assets — regenerating icons from the pack

When a new owl hero/icon pack lands in `D:\SandBoX\Mudrik-Plan\assets\new-hero-icon-pack` (4 transparent PNGs, 1254²: `owl-straight.png`, `owl-thinking.png`, `owl-point-left.png`, `owl-point-right.png`), regenerate the app assets with:

```
./scripts/generate-assets.ps1
# or:  ./scripts/generate-assets.ps1 -Pack D:\path\to\pack
```

Source → output mapping (output names are **fixed** — code references them, do not rename):

| Source PNG | Generated outputs |
|---|---|
| `owl-straight.png` | `mascot.png` (512 — splash + hero-preview), `icon.png` (256 — BrowserWindow icon / tray fallback), `tray.png` (32) + `tray@2x.png` (64), `icon.ico` (16/24/32/48/64/128/256 — electron-builder `win.icon`) |
| `owl-point-left.png` / `owl-point-right.png` | same names at 256 — Auto-Guide pointers (guide overlay, webpack CopyPlugin) |
| `owl-thinking.png` | `hero-mascot.png` (512) — README hero + website mascot. **Not bundled** into the app; lives in `assets/` so GitHub/Docs can serve it. |

Two gotchas baked into the script:
- **`.ico` build** — `[System.IO.BinaryWriter].Write($byteArray)` resolves to the wrong PowerShell overload and silently emits a ~125-byte (header-only) file; the script uses `Write($bytes, 0, $bytes.Length)`. Always sanity-check the `.ico` is ~80–120 KB, not ~125 bytes.
- **Script encoding** — keep `generate-assets.ps1` ASCII-only (or UTF-8 **with BOM**). A UTF-8-no-BOM `.ps1` containing any non-ASCII char (em-dash, box-drawing) makes Windows PowerShell 5.1 misread it and fail to parse.

After regenerating, run `npm run build` so `dist/` picks up the new PNGs. The Windows taskbar/installer icon is cached, so the old icon can linger until the icon cache refreshes or the `.exe` is rebuilt.

## Architecture (the non-obvious parts)

- **Windows-only, end-to-end**. UIA, PowerShell script embedding, robotjs, GDI+ capture, and `findOpenCodeBin` path resolution are all Windows-specific. Do not add `process.platform` branches unless you are also porting the PowerShell layer.
- **Electron tray app** (`src/main/index.ts`). The panel is a frameless, transparent `BrowserWindow`. `window-all-closed` is suppressed so the tray icon survives.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [abdallahmagdy15/mudriknow](https://github.com/abdallahmagdy15/mudriknow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
