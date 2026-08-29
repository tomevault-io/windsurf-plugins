---
trigger: always_on
description: This file is the maintenance contract for every coding agent or contributor touching this repository.
---

# AGENTS.md — ComfyUI-Majoor-H3-GuideMaster

This file is the maintenance contract for every coding agent or contributor touching this repository.

## 1. Mission

`ComfyUI-Majoor-H3-GuideMaster` is a thin, maintainable UI/compiler layer over **native MiniMax H3 behavior in ComfyUI**. It must never become a fork of the H3 model, sampler, VAE, or conditioning implementation.

The public product surface is **one node only**:

- `MajoorH3GuideMaster`

The node compiles:

- visual H3 guides -> native `conditioning[...]["minimax_keyframes"]`
- audio H3 guides -> native keyframe `audio_latent`

Do not monkey-patch ComfyUI core.

### Current node surface

Sockets, in schema order. Changing this list is a compatibility-sensitive modification.

| Socket | Type | Role |
| --- | --- | --- |
| `positive` | CONDITIONING | in/out; carries `minimax_keyframes` |
| `latent` | LATENT | in/out; passed through unchanged |
| `vae` / `audio_vae` | VAE | optional; required only by visual / audio guide slots |
| `first_frame` / `last_frame` | IMAGE | optional AddGuide-style IN / OUT anchors |
| `reference_media` | IMAGE, VIDEO (`io.MultiType`) | optional, **preview only**; discarded in `execute` |
| `audio_reference` | AUDIO | optional, **preview only**; waveform under the ruler, discarded in `execute` |
| `guide_image_N` | IMAGE | Autogrow visual guide slots |
| `guide_audio_N` | AUDIO | Autogrow audio guide slots |
| `timeline_json` | STRING | the only persisted UI state |

---

## 2. Mandatory source-reading gate — BEFORE EVERY MODIFICATION

Before changing Python behavior, node schemas, H3 timing, guide logic, frontend socket handling, or serialization, **re-read the relevant current upstream source**. Do not rely on memory, old local copies, or this document alone.

### Always read

1. Current ComfyUI repository / current target commit:
   - https://github.com/Comfy-Org/ComfyUI
2. Current native MiniMax H3 nodes:
   - https://github.com/Comfy-Org/ComfyUI/blob/master/comfy_extras/nodes_minimax_h3.py
3. Native Add Guide PR and all follow-up commits touching the H3 node/model since it merged:
   - https://github.com/Comfy-Org/ComfyUI/pull/15439
4. Current H3 packed model / timing behavior:
   - https://github.com/Comfy-Org/ComfyUI/blob/master/comfy/ldm/minimax/model.py
5. Current H3 `BaseModel` integration, especially `MiniMaxH3.extra_conds`:
   - https://github.com/Comfy-Org/ComfyUI/blob/master/comfy/model_base.py

### Additionally read for V3 node/schema changes

6. Current `comfy_api.latest` examples in ComfyUI. Prefer an actively maintained core node using `io.ComfyNode`, `io.Schema`, `io.Autogrow`, `io.NodeOutput`, and `ComfyExtension`.
6b. Before touching `reference_media`, re-read `io.MultiType` in the installed `comfy_api/latest/_io.py`. Its `Input(id, types=[...])` signature and the comma-joined `io_type` it produces are what make one socket accept both `IMAGE` and `VIDEO`.
7. Official custom-node V3 / migration documentation if the API contract has changed:
   - https://docs.comfy.org/custom-nodes/overview
   - https://docs.comfy.org/custom-nodes/backend/server_overview

### Additionally read for frontend changes

8. ComfyUI frontend extension architecture and current hooks:
    - https://github.com/Comfy-Org/ComfyUI_frontend/blob/main/docs/extensions/core.md
    - https://docs.comfy.org/custom-nodes/js/javascript_overview
    - https://docs.comfy.org/custom-nodes/js/javascript_hooks
9. Current DOM widget implementation before changing `addDOMWidget`, widget visibility, sizing, or serialization:
    - https://github.com/Comfy-Org/ComfyUI_frontend/blob/main/src/scripts/domWidget.ts
10. Before changing drag/drop or file-open handling, re-read the frontend's own document-level drop handler (`addDropHandler` in the installed frontend bundle). It claims dropped media unless the panel calls `preventDefault()` **and** `stopPropagation()` first.
11. Before changing `web/ui/preview_sources.js`, re-read the loader nodes it inspects — native `LoadVideo` (`comfy_extras/nodes_video.py`, widget `file`), `LoadImage`, and any VHS loader in use. This module reads other nodes' widget/preview state, so it is the most upstream-fragile file in the repository.

### Record what changed upstream

For any compatibility-sensitive modification, note in the PR/commit message or development notes:

- upstream ComfyUI commit SHA inspected;
- whether PR #15439 semantics changed;
- any changed values/types for `FRAME_PER_TOKEN`, `FRAME_RESCALE`, or keyframe fields.

If the current upstream behavior conflicts with this repository, **update tests first**, then adapt the smallest compatibility layer. Never copy a large upstream file into this repo.

---

## 3. Architecture invariants

### Backend boundaries

- `h3_guidemaster/core/constants.py` — only H3 constants mirrored for validation/fallback logic.
- `h3_guidemaster/core/types.py` — immutable-ish data contracts/dataclasses.
- `h3_guidemaster/core/state.py` — JSON schema parsing and validation only.
- `h3_guidemaster/core/slots.py` — Autogrow socket mapping only.
- `h3_guidemaster/core/latent.py` — H3 AV latent inspection/timeline geometry only.
- `h3_guidemaster/core/media.py` — visual/audio encode adapters only.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MajoorWaldi/ComfyUI-Majoor-H3-GuideMaster](https://github.com/MajoorWaldi/ComfyUI-Majoor-H3-GuideMaster) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
