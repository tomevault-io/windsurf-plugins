---
trigger: always_on
description: Written for a model reading the repository cold. Everything here is checked against the
---

# AGENTS.md — how this project works, in full

Written for a model reading the repository cold. Everything here is checked against the
code; where something is unverified it says so.

## What it is

A ComfyUI node pack. One node, `MiniMaxDirector`, turns a timeline of shots into the
single structured prompt the MiniMax H3 video model reads, and hands the sampler
conditioning plus a starting latent. Two helper nodes expose the compile step and the
frame-length rule on their own.

The product is a **string**. Everything else is arithmetic around it.

## End-to-end data flow

```
timeline JSON  (one widget on the node)
     │
     ├── attachments.collect ──► reference ordinals + files to load
     ├── references.assign ────► ordinals for sockets wired by hand
     ▼
compile.compile_timeline ─────► prompt string + frame count
     │
     ▼
core.call("MiniMaxH3ReferenceToVideo" | "MiniMaxH3ImageToVideo")
     │                                   (ComfyUI core, not ours)
     ▼
(positive CONDITIONING, LATENT)  ──► BasicGuider ──► SamplerCustomAdvanced
                                          │
                    ┌─────────────────────┴──────────────────────┐
              VAEDecode(video vae)                    VAEDecodeAudio(audio vae)
                    └──────────────► CreateVideo ◄───────────────┘
                                          ▼
                                      SaveVideo
```

The same `LATENT` goes to both decoders: H3's latent carries video **and** audio
together. There is no separate audio branch to keep in sync.

## The model's hard constraints

These are the model's, not design choices. Verified in
`comfy_extras/nodes_minimax_h3.py` and independently in `deepbeepmeep/Wan2GP`.

| Constraint | Value | Consequence |
|---|---|---|
| Frame rate | fixed **24 fps** (`FPS = 24`) | no rate to expose; Wan2GP raises on anything else |
| Clip length | `length % 17 == 5` | 5, 22, 39, 56, 73, 90, 107, 124 … only 8s, 25s, 42s are whole seconds |
| Output duration | **4–15s** — 96–360 frames; on the lattice, 107 f (4.46s) to 345 f (14.38s) | model card, *System Overview* table. Nothing enforces it: the node accepts up to 3600, and 362 f is already 15.08s |
| Reference clip length | each video and audio clip **2–15s**, total 15s per kind | model card (H3-Base-Ref2VA row). `CLIP_SECONDS` in `media.js` refuses at attach time; `lint._check_clip_lengths` warns |
| Reference caps | 9 images, 3 videos, 3 audios; 15s of video and of audio in total; 12 files across all three | model card (H3-Base-Ref2VA row) and `platform.minimax.io/docs/api-reference/video-generation-v2-create`. A video's soundtrack is not a file of its own. `lint._check_reference_counts` warns; frame anchors are counted apart |
| Picture shape | 256–5760 px each side, 0.4–2.5 wide-to-tall | same API page, for every `image_url`. Recorded by `media.upload` at attach time and checked by `lint._check_image_shape` |
| `used as` per kind | image: reference, storyboard, the three anchors · video: reference, continue from, edit · audio: reference | `ROLES_FOR` in `model.js`; the picker narrows, the store does not. A value already saved outside its kind's list is kept and shown, never rewritten |
| Modes | reference and frame anchor are mutually exclusive | the API refuses a request carrying both; `director.execute` raises the same error for `first frame` and `last frame` |
| Guidance | CFG-free | official graphs use `BasicGuider`, never a negative prompt |

The 17 comes from the video VAE's time axis: the latent is a row of slots, 17 frames pack
into 5 of them after a 5-frame head worth 2 -- core writes it as
`((frames - 5) // 17) * 5 + 2` (`comfy_extras/nodes_minimax_h3.py:39`). A length off the
lattice needs a fraction of a slot. `lattice.snap_up` implements it and **never rounds
down**; `model.js:stretchFor` applies it in the editor, so the document is already on the
lattice before it reaches the compiler.

## The two core nodes, and why the choice matters

| | `MiniMaxH3ImageToVideo` | `MiniMaxH3ReferenceToVideo` |
|---|---|---|
| keyframes | `first_frame`, `last_frame` | **none** |
| references | none | `ref_images`, `ref_videos`, `ref_video_audios`, `ref_audios` |
| audio vae | not taken | required |
| checkpoint | `minimax_h3_fl2va_*` | `minimax_h3_ref2va_*` |

`MiniMaxDirector.execute` picks the reference node when any reference is present,
otherwise the keyframe node. **The checkpoints are not interchangeable** — loading
`ref2va` and taking the keyframe path is a silent mismatch the graph cannot detect.

Both keyframes come off the timeline: a block whose `used as` is one of `ANCHOR_ROLES`
(`first frame`, `last frame`) has its image loaded into that argument instead of into the
reference list — the first block claiming each role takes it. The director declares no
sockets for them, or for references. Because the reference node has no `first_frame`, a
timeline holding an anchor *and* a reference is impossible to honour; the node reports an
error rather than dropping the keyframe quietly.

`keyframe` is deliberately outside `ANCHOR_ROLES`. There is no third input to load it
into, so a block used as one stays in the reference list; only its `ROLE_TASKS` entry

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [imbutus/ComfyUI-MiniMaxDirector](https://github.com/imbutus/ComfyUI-MiniMaxDirector) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
