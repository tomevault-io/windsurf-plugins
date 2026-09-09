---
trigger: always_on
description: handles as the preview.** `previewCanvas` repaints on every store write, and its
---

# Cartcut

An Electron video editor (`cartcut-app`, formerly "nugget"). Lit web components
and vanilla zustand in the renderer, plain TypeScript in the main process,
FFmpeg for export.

## Build layout — read this first

`electron/` is **source**; `main/` is the **compiled output** of it, and
`package.json` points `"main"` at `main/main.js`. Edit `electron/`, never
`main/`.

`.tsconfig/tsconfig.json` pins `rootDir: ../electron` deliberately. If any file
under `electron/` imports from `apps/app/src`, `rootDir` widens, the whole build
relocates from `main/` to `main/electron/`, and the app stops finding its entry
point. This is why the MCP tools talk to the renderer over IPC instead of
calling the editing functions directly.

`apps/app` has no `package.json` — the root webpack config builds it. The three
folders under `apps/` and `packages/` that *do* have one are standalone Vite
apps with their own lockfiles; this is not an npm workspace.

## Commands

```
npm run dev      # tsc --watch (main) + webpack --watch (renderer), concurrently
npm run start    # electron .   — run in a second terminal
npm test         # vitest run
npx tsc --noEmit -p ./.tsconfig    # typecheck the main process
npx webpack --mode=development     # build the renderer once
npm run build:overlay              # the screen recorder's own Vite app
```

`npm run dev` does **not** build `apps/overlay-record`; it has its own lockfile
and its own `tsc`. Build it after changing anything under it, or the recorder
windows load a stale bundle. The release scripts do run it.

FFmpeg and ffprobe binaries live in `./bin/<platform>-<arch>/` — `darwin-arm64`,
`darwin-x64`, `win32-x64` — and `electron/lib/ffmpeg.ts` picks the directory from
`process.arch`. electron-builder flattens the matching one into `resources/bin`,
so a packaged app sees them directly under `bin/`. `yt-dlp` sits at `bin/` root;
it is already a universal binary. See the README.

The macOS builds must be **native**. An x86_64 FFmpeg runs on Apple Silicon
under Rosetta at roughly half speed and says nothing about it — measured through
the app's own pipeline at 1080p60, H.264 goes 112 → 240 fps and H.265 32 → 102
fps just by being the right architecture. `lipo -archs bin/darwin-arm64/ffmpeg`
is the check.

## How editing works

The most important convention in the codebase. Every edit is a **pure function
`(TimelineDocument) => TimelineDocument`**, applied through
`useTimelineStore.withCheckpoint(fn)`, which records one undo step.

A pure op that declines an edit returns **its input, by identity**.
`withCheckpoint` reads that as "nothing happened" and records no step. This is
load-bearing: it is what makes a split off the end of a clip, or a drag into an
occupied slot, cost the user nothing. Preserve it in any new op.

```
apps/app/src/@types/timeline.ts          element shapes
apps/app/src/features/timeline/tracks.ts TimelineDocument, tracks, z-order
apps/app/src/features/timeline/geometry.ts   trim/duration/speed invariants
apps/app/src/features/timeline/clipOps.ts    split, trim, move, delete, removeRanges
apps/app/src/features/timeline/placement.ts  where a new element lands
apps/app/src/states/timelineStore.ts     the store, undo history
```

Two things about time that are easy to get wrong:

- `trim` is a window into the **source file**, in source ms.
  `duration === trim.endTime - trim.startTime`.
- The clip occupies `[startTime, startTime + duration/speed)` on the
  **timeline**. Use `spanOf`/`spanLength`, and `timelineTimeAt`/`sourceTimeAt`
  to convert between the two. Never open-code the arithmetic.

`priority` is derived from track order, never authored.

## The project file

A `.ngt` is a zip of five JSON entries, written and read entirely in the
renderer by `functions/project.ts`: `project.json` (`{ schemaVersion }`),
`timeline.json` (the element map), `tracks.json`, `renderOptions.json`
(`features/project/renderOptionsFile.ts`) and `assetPaths.json`
(`features/project/assetsFile.ts`).

Load **refuses to open** on a `schemaVersion` mismatch — it is a compatibility
check, not a migrator. So **a new field never moves the version**: absent means
default, answered on the way in. Both file modules state this at the top.

Media is referenced by absolute path, *and* — for anything sitting inside the
`.ngt`'s own folder — by a relative one recorded alongside it in
`assetPaths.json`. That is what makes a project folder portable: hand someone
the folder and the relative paths still name the files. The absolute paths stay
in `timeline.json` as the fallback for a `.ngt` moved on its own, away from its
media, so relinking is a *preference* and never a *replacement*.

One invariant carries the whole feature:

> **The in-memory `TimelineDocument` is always absolute.** A relative path
> exists only inside the archive.

`loadedAssetStore`, `ffmpegArgs`, the MCP tools, the preview and the export all
read `localpath` directly and all needed no changes because of it. Conversion
happens at the two file boundaries and nowhere else. A relative path must not
become a field on the element: it would enter `normalizeDocument`, every undo
snapshot and the agent serializer, and go stale the moment `localpath` changes.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cartesiancs/nugget-app](https://github.com/cartesiancs/nugget-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
