---
trigger: always_on
description: Guidance for AI agents working in this repository.
---

# AGENTS.md

Guidance for AI agents working in this repository.

## Project overview

Video Arena is a generative-media SaaS (video + image generation and face swap).
It is a **bun**-managed **Turborepo** monorepo.

- `apps/frontend` — React + Vite + TypeScript SPA. Tailwind v4 + shadcn-style UI
  (components in `src/components/ui`). Auth via `better-auth/react`
  (`src/lib/auth-client.ts`). API calls in `src/lib/api.ts`. Routes: `/` (video),
  `/image`, `/face-swap`, `/user/templates`, `/user/avatar`,
  `/admin/template/create`. Each page reuses the create/library tab layout; shared
  bits live in `components/FileField.tsx` and `components/StatusBadge.tsx`.
  `src/lib/useMe.ts` loads `/api/me` (admin flag) to gate the admin nav link.
  The admin template creator uses a hand-built Premiere-style timeline in
  `components/timeline/` (`Timeline.tsx`, `BlockInspector.tsx`, `AudioClipInspector.tsx`,
  `TemplateSetupForm.tsx`). Templates start with an **empty timeline** (no fixed
  duration — it grows to fit the furthest clip). `Timeline.tsx` has video lanes
  (V1, V2, …) and audio lanes (A1, A2, …); both video blocks and audio clips can be
  dragged/cropped. A program monitor + play/pause/stop transport scrubs a playhead
  (rAF-driven) and previews each block's frames — or, if a block has been "baked"
  (or is an uploaded video), plays that clip — while every audio clip plays via a
  hidden `<audio>` element synced to the playhead (an in-browser mix preview).
- `apps/backend` — TypeScript + Express API. Run directly with **bun** (no build step).
  - `src/auth.ts` — better-auth (email/password + Google), Prisma adapter.
  - `src/lib/openrouter.ts` — OpenRouter client: video (submit + poll) **and**
    image (`generateImage`, `listImageModels`) generation, plus video model list.
  - `src/lib/facefusion.ts` — calls the self-hosted FaceFusion swap service over HTTP.
    The frame face-swap step in `renderBlockClip` is provider-pluggable via
    `SWAP_PROVIDER`: `facefusion` (classic pixel swap) or `flux` (diffusion identity
    edit through an OpenRouter image model `OPENROUTER_SWAP_MODEL`, default
    `black-forest-labs/flux.2-klein-4b`, via `swapFaceWithImageModel`). The `flux`
    path also honors a per-block `swapContext` prompt. Verify a model with
    `scripts/test-flux-swap.ts`.
  - `src/lib/storage.ts` — MinIO (S3-compatible) object-store client.
  - `src/lib/uploads.ts` — shared multer instance + image helpers (`extFromMime`,
    `toDataUrl`); reused by every route that accepts uploads.
  - `src/lib/ffmpeg.ts` — shells out to **ffmpeg** to stitch template clips
    together (scale/pad to a common size) and mix any number of positioned audio
    parts over them (`AudioPart[]`: each trimmed + `adelay`ed to its start, summed
    with `amix`), to extract thumbnails, and to read an uploaded file's duration
    (`probeMediaDuration`, via **ffprobe**). ffmpeg is installed in the backend
    Docker image.
  - `src/lib/templateRender.ts` + `src/lib/runRender.ts` — synchronous template
    render pipeline. `renderBlockClip()` generates one block's clip (the block's
    chosen avatar slot is passed as the OpenRouter reference image and, when
    `faceSwapStart`/`faceSwapEnd` are set, is face-swapped onto the block's base
    start/end frame); `renderTemplate()` runs it for every block, stitches the clips,
    mixes the template's audio clips over them, and makes a thumbnail. The timeline
    has no fixed length — it runs to the furthest clip (video *or* audio), padding
    the video with black if audio extends past it. `renderBlockClip()` is also reused
    by the per-block "bake" route. Blocks reference avatars by slot — never per-block uploads.
    On **export** the cover thumbnail is **AI-generated from the block prompts**
    (`generateAiThumbnail` → OpenRouter image model `OPENROUTER_THUMBNAIL_MODEL`),
    falling back to an ffmpeg frame grab on error; user renders still use a frame grab.
  - `src/lib/templateSerialize.ts` — attaches public URLs to template/block/render rows.
  - `src/middleware/requireAdmin.ts` — gates admin routes; lazily promotes emails
    in `ADMIN_EMAILS` to the `admin` role. `resolveIsAdmin` is reused by `/api/me`.
  - `src/routes/{videos,images,faceswaps}.ts` — CRUD + generation per media type.
  - `src/routes/avatars.ts` — user avatars (1-2 photos; first photo = face source).
  - `src/routes/templates.ts` — user-facing templates (`/api/templates`) + their
    renders (`/api/template-renders`).
  - `src/routes/adminTemplates.ts` — admin CRUD for templates, blocks **and audio
    clips** (`/audio`, `/audio/:clipId`), plus `/blocks/:id/bake` (generate one
    block's preview clip) and `/export` (render + publish). `/api/admin/templates`,
    admin-only. Template create/patch no longer take an upfront audio file or
    duration — audio is added per-clip in the editor.
  - `src/routes/me.ts` — `/api/me` → `{ id, email, isAdmin, credits }`.
  - `src/routes/models.ts` — `/api/models[/video]` (video) and `/api/models/image`.
  - `src/routes/credits.ts` — credits & billing (`/api/credits`): balance + ledger
    (`GET /`), packs + per-action prices + checkout config (`GET /packs`), create a
    Razorpay order (`POST /checkout`), verify a completed payment + grant credits

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [codes30/pixovid](https://github.com/codes30/pixovid) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-02 -->
