---
trigger: always_on
description: These instructions apply to every automated coding agent working in this repository.
---

# AGENTS.md — ComfyUI-Majoor-OmniCam

These instructions apply to every automated coding agent working in this repository.

## 1. Read official sources before coding

For every code request, consult the relevant official ComfyUI source **before** writing or modifying Python, JavaScript/TypeScript/Vue, routes, packaging, docs, or model integrations.

### Required official sources

| Source | URL | Use it for |
|---|---|---|
| ComfyUI Core (Python) | https://github.com/Comfy-Org/ComfyUI | Python APIs, V3 nodes, `PromptServer`, routes, execution, `VIDEO`, `LOAD3D_CAMERA` |
| ComfyUI Server Docs | https://docs.comfy.org/development/comfyui-server/comms_overview | client/server communication, custom HTTP routes, websocket model |
| ComfyUI Frontend | https://github.com/Comfy-Org/ComfyUI_frontend | `app.registerExtension`, node lifecycle, DOM/Vue APIs, Load3D camera/recording patterns |
| ComfyUI Desktop | https://github.com/Comfy-Org/desktop | packaged-app constraints, install paths, Chromium/Electron behavior |
| ComfyUI Manager | https://github.com/Comfy-Org/ComfyUI-Manager | Registry/Manager install/update/security expectations |
| Embedded Docs | https://github.com/Comfy-Org/embedded-docs | user-facing help conventions |
| Registry docs | https://docs.comfy.org/registry/publishing | `pyproject.toml`, publishing, versioning |

If an API has changed since the last commit, adapt the implementation to the current official API rather than preserving obsolete local assumptions.

## 2. Core product invariant

**OmniCam core is model-agnostic.**

The canonical product flow is:

```text
Viewport / Timeline
        ↓
OMNICAM_EDITOR_STATE
        ↓
OMNICAM_MOTION_SCENE
        ↓
Monitor profile compiler
        ↓
model-native artifact
```

`OMNICAM_MOTION_SCENE` is the product interchange contract between Extractor,
Director and Monitor. It owns:

- authoring timeline;
- canvas;
- cameras;
- objects;
- motion layers;
- cuts;
- model-independent metadata.

`MAJOOR_OMNICAM_TRACK` remains the versioned camera primitive used inside
MotionScene and by solve/compiler internals. It is NOT the top-level OmniCam
product interchange contract.

No MiniMax-, Wan-, LTX-, Blender-, Unreal- or API-specific model semantics
belong in MotionScene authoring or the viewport engine. Model-specific behavior
belongs behind Monitor profiles.

## 3. Canonical document contracts

### MotionScene

The public product document is versioned through `motion_scene.version`.

An incompatible schema change requires:

1. incrementing the MotionScene version;
2. registering a migration in `omnicam/core/migrations.py`;
3. keeping previously saved workflows loadable;
4. adding a migration regression test;
5. adding a saved-workflow fixture.

Simply increasing the MotionScene version and rejecting every older workflow is
not a valid migration strategy.

Minimal shape:

```json
{
  "version": 1,

  "timeline": {
    "duration_seconds": 5.0,
    "authoring_fps": 24.0
  },

  "canvas": {
    "width": 1280,
    "height": 720
  },

  "cameras": [],
  "active_camera_id": "camera_1",
  "playblast_camera_id": "camera_1",

  "objects": [],
  "motion_layers": [],
  "cuts": [],

  "metadata": {}
}
```

### Camera Track

`MAJOOR_OMNICAM_TRACK` remains independently versioned as the canonical camera
primitive embedded in MotionScene.

The canonical track is versioned.

Required fields:

```json
{
  "schema_version": 1,
  "fps": 24,
  "duration_frames": 120,
  "width": 1280,
  "height": 720,
  "render_mode": "omni_ref",
  "keyframes": [
    {
      "frame": 0,
      "camera": {
        "position": [6.0, 4.0, 6.0],
        "target": [0.0, 1.5, 0.0],
        "fov": 35.0,
        "roll": 0.0,
        "camera_type": "perspective",
        "zoom": 1.0,
        "near": 0.01,
        "far": 10000.0
      },
      "interpolation": "ease"
    }
  ],
  "objects": [],
  "metadata": {}
}
```

Do not silently break this format. Increment `schema_version` and provide migration code when required.

## 4. Backend rules

- Prefer the modern V3 API (`ComfyExtension`, `IO.Schema`, `IO.NodeOutput`).
- Keep pure math/track code independent of ComfyUI imports so it can be unit-tested.
- Validate all paths from frontend requests.
- Never accept arbitrary filesystem paths from upload routes.
- Restrict uploaded file extensions and size.
- Save user-authored card/playblast files below ComfyUI's managed input/output directories.
- Use `InputImpl.VideoFromFile` for `VIDEO` outputs when following current core behavior.
- Do not add heavyweight runtime dependencies if browser/core APIs already cover the task.

## 5. Frontend rules

- Use `app.registerExtension` or the current documented extension mechanism.
- Do not patch ComfyUI core files.
- Avoid deprecated `/extensions/core/...` imports.
- Preserve node serialization: camera state must survive workflow save/reload.
- Keep keyboard shortcuts scoped to the active OmniCam viewport.
- Do not steal shortcuts while the user is typing into inputs.
- The viewport must degrade gracefully if `MediaRecorder` is unavailable.
- Production WebGL work should use a bundled/pinned dependency, not a CDN, because ComfyUI CSP is restrictive.
- Dispose object URLs, observers, timers, GPU resources, and listeners.

## 6. Viewport UX


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MajoorWaldi/ComfyUI-Majoor-OmniCam](https://github.com/MajoorWaldi/ComfyUI-Majoor-OmniCam) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
