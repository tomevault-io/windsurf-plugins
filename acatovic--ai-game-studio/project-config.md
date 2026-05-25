---
trigger: always_on
description: A lightweight web app for creating 2D game sprites from text prompts, turning the generated sprite into an image-to-video motion sequence, extracting transparent-background frames, and composing them into a 1×N spritesheet with a looping animated preview. Named projects can be saved, listed, loaded, and deleted; the in-progress state always lives in `projects/latest/` and is auto-persisted.
---

# AGENTS.md

## Project

A lightweight web app for creating 2D game sprites from text prompts, turning the generated sprite into an image-to-video motion sequence, extracting transparent-background frames, and composing them into a 1×N spritesheet with a looping animated preview. Named projects can be saved, listed, loaded, and deleted; the in-progress state always lives in `projects/latest/` and is auto-persisted.

The app is implemented as a Vite + TypeScript single-page client and a small Express server (run concurrently in dev via `tsx watch`). Keep the UI lightweight and avoid heavy UI libraries.

## Non-negotiable requirements

- The UI must adhere closely to `mockup.png` (the three-column "Sprite Sheet Builder" layout).
- Use xAI / Grok Imagine for image and video generation.
- Expect `XAI_API_KEY` to be defined in a local `.env` file.
- Never expose `XAI_API_KEY` in client-side code.
- All xAI calls go through server-side routes; the browser never talks to xAI directly.
- Generated artifacts live under `projects/` (gitignored). Source frames, videos, spritesheets, gifs, and the working manifest are never committed.
- Do not commit `.env`, `projects/`, `frames/`, `*.mp4`, `*.mov`, `*.webm`.

## Expected environment

`.env.example` documents:

```bash
XAI_API_KEY=
```

Developer copies it locally:

```bash
cp .env.example .env
```

Dependencies that the project requires:

```bash
npm install ai @ai-sdk/xai express
npm install -D vite typescript tsx concurrently dotenv @types/express @types/node
```

The `ai` and `@ai-sdk/xai` packages must be a version that exports `experimental_generateVideo` and `xai.video()`. At the time of writing those are `ai@beta` and `@ai-sdk/xai@4.0.0-canary.66` or newer. The stable v5 / v2 releases do not yet expose video generation.

`ffmpeg` must be available on `PATH` — used for both frame extraction (with chroma-key alpha) and animated GIF preview build.

## UI requirements

Use `mockup.png` as the source of truth for the three-column layout, spacing, and visual hierarchy. The UI supports:

1. A prompt input for the initial reference sprite.
2. A "Generate Reference Sprite" button.
3. A preview area for the generated sprite (with dimensions caption).
4. A motion / sequence description input (e.g. "walking left", "jump", "attack right").
5. A "Generate Frames" button that runs xAI image-to-video, downloads the clip, and extracts transparent PNG frames.
6. A scrollable grid of every extracted frame, each tile click-toggleable to include/exclude it from the spritesheet.
7. A "Generate Spritesheet" button that composes the selected frames client-side into a 1×N PNG.
8. A horizontally scrollable spritesheet preview, an Export PNG button, and a looping animated GIF preview.
9. Header controls: New (start fresh), Load (dropdown of saved projects with delete), Save (prompt-for-name with overwrite confirm), and a label showing the current project name (`untitled` when unsaved).
10. Clear loading, success, and error states inline near each step. Buttons disable while their work is in flight.

Keep the interface focused on sprite creation. Do not add unrelated dashboards, auth, billing, social features, or project-management bloat.

## Implementation architecture

```text
.
├── AGENTS.md
├── .env.example
├── mockup.png
├── package.json
├── tsconfig.json
├── vite.config.ts                # proxies /api and /projects → :8787
├── index.html
├── src/
│   ├── main.ts                   # entry
│   ├── app.ts                    # shell + handlers + render loop
│   ├── lib/
│   │   ├── api.ts                # fetch wrappers + types
│   │   ├── state.ts              # Store, hydrateFromView, cacheBust
│   │   └── spritesheet.ts        # canvas-based composer
│   ├── components/
│   │   └── icons.ts              # inline SVG icons
│   └── styles/
│       └── main.css
├── server/
│   ├── index.ts                  # Express app + route handlers
│   ├── files.ts                  # paths, PNG dim parser, safe name
│   ├── projects.ts               # manifest read/write, save/load
│   ├── xai-image.ts              # sprite generation with chroma directive
│   ├── xai-video.ts              # motion video with chroma directive
│   ├── extract-frames.ts         # ffmpeg wrapper with chromakey
│   └── build-gif.ts              # animated preview GIF build
├── scripts/
│   └── extract-frames.sh         # ffmpeg chromakey + scale → transparent PNGs
└── projects/                     # gitignored
    ├── latest/                   # working state
    │   ├── sprite.json
    │   ├── ref/sprite.png
    │   ├── source.mp4
    │   ├── frames/frame-XXXXX.png
    │   ├── spritesheet.png
    │   └── preview.gif
    └── <name>/                   # snapshots after Save
        └── (same layout)
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [acatovic/ai-game-studio](https://github.com/acatovic/ai-game-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-25 -->
