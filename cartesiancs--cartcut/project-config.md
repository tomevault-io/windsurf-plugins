---
trigger: always_on
description: An Electron video editor (`cartcut-app`, formerly "nugget"). Lit web components
---

# Cartcut

An Electron video editor (`cartcut-app`, formerly "nugget"). Lit web components
and vanilla zustand in the renderer, plain TypeScript in the main process,
FFmpeg for export.

This file is a map and a list of invariants. It is loaded into every session, so
keep it short: record what a reader cannot get from the code in a minute, and
nothing else. Design history, changelogs and "how we know it works" belong in
commit messages and in the tests.

## Writing rules

**Never use an em-dash (`—`) or a middle dot (`·`).** Not in code comments, not
in documentation, not in commit messages, not in UI strings, not here. Write
what the dash was standing in for: a comma or brackets for an aside, a colon for
a consequence, a semicolon or two sentences for two joined thoughts, `1 to 10`
for a range, `a * b` for multiplication, a real list for a list. An en-dash
(`–`) is fine in a numeric range and nowhere else; a hyphen is always fine.

Two habits for the same reason: no "It's not X, it's Y" constructions, and no
rhetorical flourish where a fact belongs. A comment earns its length by naming
the specific failure the line below it prevents.

## Build layout: read this first

`electron/` is **source**; `main/` is its **compiled output**, and `package.json`
points `"main"` at `main/main.js`. Edit `electron/`, never `main/`.

**`electron/` may not import from `apps/app/src`.** `.tsconfig/tsconfig.json`
pins `rootDir: ../electron`; one such import widens it, the build relocates from
`main/` to `main/electron/`, and the app stops finding its entry point. This is
why the MCP tools talk to the renderer over IPC instead of calling the editing
functions directly, and why the recorder tray crosses the boundary as data.

`apps/app` has no `package.json`; the root webpack config builds it. The folders
under `apps/` and `packages/` that *do* have one are standalone Vite apps with
their own lockfiles. This is not an npm workspace.

`plugins/cartcut-editing/` is **published to users**, through the marketplace
declared at `.claude-plugin/marketplace.json`. Its skill is the one the app's
users get; it is no longer under `.claude/`, so editing it changes what ships.
The plugin has to stay a subdirectory: a plugin's root `package.json` is
installed by Claude Code, and pointing a plugin at the repository root would
make installing it build the whole Electron app. The two versions, in
`plugin.json` and in the marketplace entry, have to agree; `claude plugin tag`
checks that, `claude plugin validate <path> --strict` checks the rest.

FFmpeg and ffprobe live in `./bin/<platform>-<arch>/` (`darwin-arm64`,
`darwin-x64`, `win32-x64`); `electron/lib/ffmpeg.ts` picks by `process.arch`.
The macOS binaries must be **native**: an x86_64 ffmpeg runs under Rosetta at
roughly half speed and says nothing. `lipo -archs bin/darwin-arm64/ffmpeg`.

## Commands

```
npm run dev      # tsc --watch (main) + webpack --watch (renderer)
npm run start    # electron .   (run in a second terminal)
npm test         # vitest run
npx tsc --noEmit -p ./.tsconfig    # typecheck the main process
npx webpack --mode=development     # build the renderer once
npm run build:overlay              # the screen recorder's own Vite app
npm run build:speech               # the native STT sidecar (Swift, macOS only)
```

`npm run dev` does **not** build `apps/overlay-record`; build it yourself after
touching it or the recorder windows load a stale bundle. It does build
`native/cartcut-stt` (a staleness check plus two `swiftc` calls) and skips
itself loudly on Windows or without the macOS 26 SDK.

## How editing works

The most important convention in the codebase. Every edit is a **pure function
`(TimelineDocument) => TimelineDocument`**, applied through
`useTimelineStore.withCheckpoint(fn)`, which records one undo step.

**A pure op that declines returns its input, by identity.** `withCheckpoint`
reads that as "nothing happened" and records no step, which is what makes a
split off the end of a clip or a drag into an occupied slot cost the user
nothing. Preserve it in any new op, and give every new op a co-located suite
that covers the decline path as well as the happy one.

```
apps/app/src/@types/timeline.ts          element shapes, FILETYPES, animatable properties
apps/app/src/features/timeline/tracks.ts     TimelineDocument, tracks, z-order
apps/app/src/features/timeline/geometry.ts   trim/duration/speed invariants
apps/app/src/features/timeline/clipOps.ts    split, trim, move, delete, removeRanges
apps/app/src/features/timeline/placement.ts  where a new element lands
apps/app/src/states/timelineStore.ts         the store, undo history
```

Two things about time that are easy to get wrong:

- `trim` is a window into the **source file**, in source ms.
  `duration === trim.endTime - trim.startTime`.
- The clip occupies `[startTime, startTime + duration/speed)` on the
  **timeline**. Use `spanOf`/`spanLength` and `timelineTimeAt`/`sourceTimeAt`;
  never open-code the arithmetic.

`priority` is derived from track order, never authored. Layering is track order:
index 0 is the top row and the front of the composite.

## Conventions that repeat across every feature

Learn these five once and most of the codebase stops needing explanation.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cartesiancs/cartcut](https://github.com/cartesiancs/cartcut) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
