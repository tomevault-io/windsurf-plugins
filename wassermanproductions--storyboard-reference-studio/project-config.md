---
trigger: always_on
description: This file is the single source of truth for AI coding agents working on this repo. `CLAUDE.md` points here.
---

# AGENTS.md — running & modifying Storyboard Reference Studio with an AI agent

This file is the single source of truth for AI coding agents working on this repo. `CLAUDE.md` points here.

## What this app is

Electron + TypeScript + React 18 desktop tool. Filmmakers turn ANY reference imagery — movie clips, phone footage, pulled stills, mood images — into a **storyboard of stills + image-generator-ready prompts**, so an image generator can recreate the framing and shot elements of each frame. Third app in Sam Wasserman's AI-filmmaking suite (with Blockout and Motion Previs Studio). Full product brief: `DESIGN.md`.

## Commands

```bash
npm install            # once; Node 22+. ffmpeg is BUNDLED via ffmpeg-static; ffprobe uses PATH/absolute probes.
npm run dev            # run the app with hot reload
npm run build          # production build into out/
npm start              # run the production build
npm run typecheck      # strict TS, two projects (renderer+shared+preload, main+e2e)
npm run lint           # ESLint (zero warnings)
npm run smoke          # build + Playwright end-to-end: real ffmpeg extraction + export
npm run package        # macOS DMG into release/ (Phase B; do not run yet)
```

**Definition of done for any change:** `npm run typecheck && npm run lint && npm run build` green, and `npm run smoke` green if you touched main/, shared/, frames/export/describe, the store, or frameOps.

## Repo map

```
src/shared/     PURE data + logic, imported by BOTH main and renderer.
                types.ts (Project/MediaItem/Frame/Crop/ShotMeta/Annotation model),
                schema.ts (factories + parseProject — never throws; migrates
                v1→v2), profiles.ts (generator profiles + dropdown option lists
                incl. MOVEMENTS/TRANSITIONS), annotations.ts (renderAnnotationsSvg
                — pure SVG string for arrows/text, used on-screen AND for export).
src/main/       Electron main process. index.ts (window, project I/O, media +
                audio import + probe, IPC), ffmpeg.ts (resolveFfmpeg/resolveFfprobe
                + probeMedia), frames.ts (extract + extractRange: interval/scene/
                count), describe.ts (Claude vision → structured prompt),
                export.ts (board package + contact sheet + animatic MP4 + shot-list
                CSV), pdf.ts (PDF storyboard via hidden BrowserWindow printToPDF),
                annotate.ts (composite annotations onto stills via hidden
                BrowserWindow canvas), control.ts (agent HTTP control server).
src/preload/    Typed IPC bridge (window.sbr). Keep in sync with main.
src/renderer/   React UI. store.ts (zustand; ALL doc edits via store.mutate;
                also ephemeral view state: viewMode, annotTool, guidesOn,
                presentOpen), App.tsx (welcome/titlebar/keyboard/autosave),
                panels/ (MediaBin, Viewer, FrameStage, Inspector, CropEditor,
                Board, Present, Help, Toasts), lib/ (paths, useMediaUrl,
                frameOps, inspectorHelpers), control.ts (renderer side of the
                agent control surface).
mcp/            storyboard-mcp.mjs — zero-dep stdio MCP bridge to the control server.
tests/e2e/      Playwright smoke test (real app, real ffmpeg, real export).
```

## Hard rules

1. **ffmpeg packaging (load-bearing):** `ffmpeg-static` is EXTERNAL to the main bundle (`rollupOptions.external` in electron.vite.config.ts) + `asarUnpack` + `files` include in electron-builder.yml. Bundling it rewrites the `__dirname` it uses to find the binary and breaks exports. `resolveFfmpeg` order: `SBR_FFMPEG` env → bundled ffmpeg-static (unpacked from asar) → absolute Homebrew/usr paths → bare `ffmpeg`. Finder-launched apps don't inherit the shell PATH, hence the absolute probes.
2. **Claude calls run in the MAIN process only** (`src/main/describe.ts`): model `claude-opus-4-8`, `thinking: { type: 'adaptive' }`, vision, `output_config` json_schema. Auth chain: `ANTHROPIC_API_KEY` → `ant` CLI profile (SDK) → `~/.config/storyboard-reference/anthropic-api-key`. The whole app works offline EXCEPT the Generate button, which returns a structured friendly error when no credentials resolve.
3. **All document edits go through `store.mutate(label, fn)`** (or a store action that calls it) — never assign into `store.doc` directly (breaks dirty tracking + autosave). Destructive actions (removing a prompted frame) confirm.
4. **Selectors must return stable references.** `useStore((s) => s.orderedFrames())` returns a NEW array every call and loops React (error #185). Select the raw `doc.frames` and sort in a `useMemo` (see Board.tsx).
5. **Projects are folders:** `<name>.sbref/` = `project.json` (pretty-printed) + `media/` (COPIES of imports incl. an optional audio scratch track) + `.autosave/` (60s backup) + `.frames/` (extracted full-res stills cache) + `exports/`. `parseProject` must never break on an existing file; sanitize/migrate instead.
6. **Schema is versioned (`PROJECT_VERSION = 2`).** `parseProject` migrates v1→v2 silently — new per-frame fields (`durationS`, `shot`, `annotations`) and `settings.audioFile` get defaults, and the doc's `version` is stamped to current. Never throw on old files.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wassermanproductions/storyboard-reference-studio](https://github.com/wassermanproductions/storyboard-reference-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-15 -->
