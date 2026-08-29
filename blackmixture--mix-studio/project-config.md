---
trigger: always_on
description: Handbook for AI agents (and humans) continuing development. Read this before touching anything. The app is in daily production use by Nathan (Black Mixture) — his real gallery lives in `data/`.
---

# AGENTS.md — working on KreaStudio

Handbook for AI agents (and humans) continuing development. Read this before touching anything. The app is in daily production use by Nathan (Black Mixture) — his real gallery lives in `data/`.

## What this is

A zero-dependency Node.js (≥22) server (`server.js`) that builds **ComfyUI API-format graphs** server-side and relays progress to a vanilla-JS mobile frontend (`public/`). No npm packages, no build step, no framework. ComfyUI runs on the same Windows machine (URL in `data/settings.json`, `comfyUrl`).

```
server.js                 ~3.7k lines: routes, graph builders, job tracking, SSE
lib/                      focused modules (regional-workflows, video-workflows,
                          upscale-workflows, profiles, private-gallery, queue-health, …)
public/index.html|app.js|style.css   the whole frontend
test/*.test.js            node:test suites — `node --test` must stay green
data/                     LIVE USER DATA — never in git, never test destructively
docs/superpowers/plans/   old planning docs (gitignored)
```

## Architecture in five sentences

1. The frontend POSTs to routes like `/api/generate`, `/api/animate`, `/api/upscale`; the server builds a ComfyUI graph (per engine), POSTs it to `/prompt`, and stores a job in the `jobs` Map keyed by `prompt_id`.
2. A WebSocket to ComfyUI (with polling fallback) tracks progress and fires `completeJob`, which downloads outputs from ComfyUI's `/history` + `/view`, writes them into `data/images|videos`, creates/updates gallery items in `db.json`, and broadcasts over SSE (`/api/events`).
3. Everything user-visible is scoped to a **profile** (signed cookie `ks_profile`; see `lib/profiles.js`); the first profile in `db.profiles` is the owner/admin.
4. Uploaded inputs (refs, masks, audio, driving videos, faces) go to ComfyUI's input dir via `/api/upload` → `uploadToComfy`; `/api/input?name=` proxies them back for reuse previews.
5. Graph builders use two safety layers: `nodeFromOrdered(class, orderedWidgets, links, overrides)` maps workflow-JSON widget arrays onto the real `/object_info` input order, and `filterInputs(graph)` drops keys the installed node version doesn't know.

## Engines and their builders (server.js)

| Engine | Builder | Notes |
|---|---|---|
| Krea2 t2i | `buildT2I` | euler/beta, 12 steps, cfg 1 |
| Krea2 depth guide | `lib/krea2-workflows.buildKrea2DepthControl` | DA3 Large V2-style depth → Krea2 Control LoRA; opt-in from Create image guide |
| Regional t2i | `lib/regional-workflows.buildRegionalT2IGraph` | Ideogram4PromptBuilderKJ (bboxes from slot 2!) + Krea2RegionalMultiLoRAV3 |
| Klein edit | `buildEdit` | ReferenceLatent chains, Flux2Scheduler, 4 steps |
| Qwen edit | `buildEditQwen` | 2511 + Lightning LoRA, source-encoded latent |
| Krea2 inpaint | `lib/regional-workflows.buildKrea2InpaintGraph` | **soft inpaint**: `VAEEncode` + `SetLatentNoiseMask` (see gotchas) |
| LTX 2.3 | `buildAnimate` | two-stage: base sigmas + ×2 latent upsample refine |
| LTX Face ID | `buildAnimateFaceId` | single-stage 24 fps, BFS `LTXIdentityOverlapConditioning`, FaceID LoRA @1.0 over distilled-1.1 @0.6; `opts.audioName` → `audioLatentNodes` freezes a voice recording into the audio latent (LoadAudio → LTXVAudioVAEEncode → SetLatentNoiseMask 0.0) for identity-locked lipsync |
| 10Eros | `buildAnimateEros` | Echo DMD sampler, sigma presets in `EROS_SIGMA_PRESETS` |
| Wan 2.2 | `buildAnimateWan` | dual KSamplerAdvanced handoff, 16 fps 4n+1 frames |
| SCAIL 2 | `buildAnimateScail` (+ chunk/infinity variants in `lib/video-workflows`) | SAM3 tracks driving video + ref, WanSCAILToVideo |
| Upscales | `buildUpscale` / `lib/upscale-workflows` | SeedVR2 / Ultimate SD |
| Composite | `/api/composite` route | ImageStitch side-by-side, per-frame |

Enhancement passes: LTX enhances **in-graph** (`TextGenerateLTX2Prompt` / `TextGenerate` with image input); Krea2/Wan/SCAIL enhance via a **separate server-side job** (`enhancePrompt`, `wanEnhance` — two-pass with `<final_prompt>` sentinel + `cleanEnhancedText`, because Qwen3-VL leaks its reasoning otherwise).

## Hard-won gotchas (do not relearn these the expensive way)

- **V3 DynamicCombo inputs** (TextGenerate `sampling_mode`, Ideogram `style`) must be serialized **flat**: `sampling_mode: 'on'` plus dot-keys like `'sampling_mode.temperature': 0.7`. Objects break validation. See `textGenInputs()`.
- **Literal arrays in API graphs are treated as node links** (`[id, slot]`) — you cannot pass arrays/objects as widget values (`editor_state` on CreateBoundingBoxes is unusable via API; that's why regional bboxes come from Ideogram4PromptBuilderKJ **output slot 2**).
- **Flow/DiT models (Krea2, Flux, Qwen) cannot use `VAEEncodeForInpaint`** — they reproduce the grey erase. Use `VAEEncode` + `SetLatentNoiseMask` and denoise ≥0.75.
- **libx264 requires even dimensions.** Any graph ending in SaveVideo must guarantee even W/H (`ImageResizeKJv2` with `divisible_by: 2`, or compute even dims). MP4 container metadata lies about rotated phone videos — resize decoded frames in-graph, never trust `tkhd`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BlackMixture/Mix-Studio](https://github.com/BlackMixture/Mix-Studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
