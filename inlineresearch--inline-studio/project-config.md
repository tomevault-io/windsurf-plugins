---
trigger: always_on
description: Inline Studio is a **narrative-first desktop app for filmmakers** that uses **ComfyUI** as an open
---

# Inline Studio — Engineering Guide

Inline Studio is a **narrative-first desktop app for filmmakers** that uses **ComfyUI** as an open
generative render-farm. Creators compose on a free-form node canvas and work frame-by-frame, while
ComfyUI does the actual image/video/audio/LLM generation behind each frame.

> Read this file before changing code. It defines the architecture and the non-negotiable rules.

## Mental model (everything is organised around this)

```
Project → Sequence → Frame → Take[]
```

- **Project** — a portable `.inlinestudio` folder (see Storage below).
- **Sequence / Scene** — an ordered group of frames.
- **Frame** — the atomic unit. **A Frame is a _slot with a history of takes_, never a single file.**
  Its inputs are library assets _or_ another frame's output (the refine/flow link).
- **Take** — one immutable ComfyUI render of a frame. Generating again adds a new take; nothing is
  overwritten. The frame points at its `heroTakeId` (the chosen take), which flows downstream.
- **Moodboard ↔ Timeline** — a frame is either pinned on the free-form canvas or surfaced in the
  Timeline panel. Same frame, different surface.

If you're tempted to treat a frame as a file, stop — the take history is the core value Comfy lacks.
(Note: the domain was renamed shot → frame; some older migrations still reference `shot_*` tables.)

## Architecture

Electron, two processes, one direction of dependency: **`renderer → IPC → main`**.

- **Main process** (`electron/main/`) — owns all "trusted" work: filesystem, the project
  SQLite DB, the ComfyUI client, and ffmpeg. Node APIs live here only.
- **Preload** (`electron/preload/`) — the _only_ bridge. Exposes a typed, minimal surface on
  `window.inlineStudio` via `contextBridge`. No raw `ipcRenderer`/channels leak to the renderer.
- **Renderer** (`src/renderer/`) — all React UI. Reaches the outside world _only_ through
  `window.inlineStudio`. Never imports `electron`, `fs`, `path`, `better-sqlite3`, `ws`, or
  `fluent-ffmpeg` (ESLint enforces this).
- **Shared** (`src/shared/`) — types + the IPC contract imported by both processes.

### Directory map

```
electron/
  main/
    index.ts            app entry + BrowserWindow (security baseline)
    db/                 SQLite: schema.ts (tables+migrations), index.ts (open/close)
    project/            project lifecycle: store.ts (.inlinestudio folders), recents.ts
    ipc/                handler.ts (Result wrapper), <feature>.ts handlers, index.ts (register)
    comfy/              client.ts — ComfyUI bridge (link/upload/capture); all Comfy knowledge here
    frames/             frame + take + input store
    moodboard/          canvas items + connectors store
    export/             folder.ts — hero-take export (file copy today; ffmpeg later)
  preload/
    index.ts            contextBridge → window.inlineStudio
src/
  shared/
    types.ts            domain types (Project/Sequence/Frame/Take/MoodboardItem/...)
    ipc.ts              IpcChannels + InlineStudioApi (the typed contract)
    result.ts           Result<T> = Ok | Err
  renderer/
    main.tsx, App.tsx
    store/              Zustand stores (feature-scoped: moodboardStore, frameStore, ...)
    views/              feature-foldered screens (ProjectLauncher, Workspace, Moodboard, Library, Generate)
    components/         shared UI
```

### Storage — a project is a portable folder

```
MyFilm.inlinestudio/
  project.db   (SQLite — source of truth; "save" is implicit)
  assets/      (imported library media, by id)
  takes/       (generated outputs from ComfyUI, by take id)
  thumbs/      (cached thumbnails / waveforms)
  workflows/   (durable per-frame ComfyUI workflow copies, by frame id)
```

The recent-projects list lives in Electron `userData` (app-global), not in any project.

### ComfyUI integration (`electron/main/comfy/client.ts`)

`COMFYUI_URL` comes from an in-app setting (falling back to env, see `.env.example`). The Generate
tab **embeds ComfyUI in a `<webview>`** rather than driving it headlessly, so the full node graph is
always available. The bridge flow:

- **Link a frame** → ensure a workflow exists at `/userdata/workflows/<name>.json` (Inline Studio keeps
  the durable copy under the project's `workflows/`); seed a minimal one if none.
- **Inputs** → upload the frame's inputs (assets, or a flow link resolved to the source frame's hero
  take) via `/upload/image`, then wire them into the workflow's `LoadImage` nodes so the displayed
  input is the one ComfyUI loads.
- **Save** → an injected in-page hook (forces `overwrite: true`) catches saves and pulls the workflow
  JSON back into the durable copy.
- **Capture** → finished outputs are read from `/history` + `/view` and pulled into `takes/` as new
  takes; the chosen hero flows to downstream frames.

"Open in ComfyUI" / the embedded webview is the power-user surface; it's deliberately first-class
here, not just an escape hatch.

## Code standards (non-negotiable)

- **TypeScript strict.** No implicit `any`, no `as any` to silence errors. `npm run typecheck`.
- **Typed IPC only.** Channels live in `src/shared/ipc.ts`; the preload implements `InlineStudioApi`;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [inlineresearch/Inline-Studio](https://github.com/inlineresearch/Inline-Studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-20 -->
