---
trigger: always_on
description: **Why**: Pixal3D (SIGGRAPH 2026, TencentARC) is a state-of-the-art single-image → textured-GLB pipeline built on the TRELLIS.2 backbone. It's already wrappable: a clean Python API (`Pixal3DImageTo3DPipeline.from_pretrained()` + `.run(image, camera_params=...)`) and a reference `inference.py`. Our `ComfyUI-TRELLIS2` pack is the closest precedent — same dependency stack (torch 2.6 / cu124, flash-attn-3, cumesh, flex_gemm, o_voxel, DinoV3) and same cascade architecture. We will mirror its layout, s
---

# ComfyUI-Pixal3D — Wrapping Plan

## Context

**Why**: Pixal3D (SIGGRAPH 2026, TencentARC) is a state-of-the-art single-image → textured-GLB pipeline built on the TRELLIS.2 backbone. It's already wrappable: a clean Python API (`Pixal3DImageTo3DPipeline.from_pretrained()` + `.run(image, camera_params=...)`) and a reference `inference.py`. Our `ComfyUI-TRELLIS2` pack is the closest precedent — same dependency stack (torch 2.6 / cu124, flash-attn-3, cumesh, flex_gemm, o_voxel, DinoV3) and same cascade architecture. We will mirror its layout, swap in the Pixal3D-specific pipeline, and ship as an MVP that produces a GLB end-to-end.

**Licensing posture (locked)**:
- **Wrapper code**: MIT, with a prominent README disclaimer.
- **Pixal3D model weights**: academic-only per Tencent's license (`repo/LICENSE:15`); README must state non-commercial and EU-excluded clauses verbatim.
- **No nvdiffrast / nvdiffrec_render** anywhere — they are non-commercial-only NVIDIA licenses and the core inference path doesn't need them. We will use the same `_vb_ap` / `_ap` cuda-wheels variants TRELLIS2 uses, which are sourced from `PozzettiAndrea/Trellis.2.drtk` (already nvdiffrast-free). Preview = GLB viewer only. A later phase will port `pixal3d/renderers/{pbr_mesh_renderer,mesh_renderer}.py` from `nvdiffrast` → `drtk` (mirroring what we did for TRELLIS2) to unlock HDRI/clay/basecolor preview modes; **out of scope for this PR**.

**Distribution posture (locked)**:
- **First publish: `PozzettiAndrea/ComfyUI-Pixal3D` as a PRIVATE GitHub repo.** Do not flip to public until we have done a license-review pass and confirmed the academic-only / EU-exclusion disclaimers are correctly surfaced in the README.
- **Install entry point**: `cds get pixal3d`. The config lives at `/home/work/coding-scripts/comfy-dev-cli/config/setup/pixal3d.yml` and follows the same shape as `trellis2.yml` / `sam3dobjects.yml`.
- Since the repo is private, `cds get pixal3d` will require `gh auth` on the dev machine. Once we go public this is transparent.

**Intended outcome**: a `ComfyUI-Pixal3D` pack (private GitHub repo) installable via `cds get pixal3d`, exposing 5 MVP nodes that reproduce `repo/inference.py` end-to-end inside a ComfyUI workflow, isolated in its own pixi env via comfy-env, with no nvdiffrast dependency.

---

## Approach

Process-isolated pack, mirroring `ComfyUI-TRELLIS2`. All Pixal3D Python code (torch 2.6 + cu124) runs in a comfy-env subprocess; ComfyUI's host env is untouched. The `pixal3d/` package is **vendored** (not pip-installed from the repo root) so we control the import surface.

---

## Repo Layout

```
ComfyUI-Pixal3D/
├── comfy-env-root.toml          # [cuda] = ["flash-attn","sageattention"]; [node_reqs] for GeometryPack
├── install.py                   # from comfy_env import install; install()
├── prestartup_script.py         # setup_env(); copy_files(assets); copy_viewer("glb_three", web/)
├── __init__.py                  # register_nodes() + register TRELLIS2-compatible model configs if helpful
├── requirements.txt             # comfy-env, comfy-3d-viewers, comfy-sparse-attn, trimesh[easy]
├── pyproject.toml               # version, MIT, publisher
├── LICENSE                      # MIT (wrapper code only)
├── README.md                    # Must include Pixal3D academic-only + EU-exclusion disclaimers verbatim
├── nodes/
│   ├── comfy-env.toml           # python 3.10; [cuda]=["flex_gemm_ap","cumesh_vb","o_voxel_vb_ap","flash-attn","sageattention","drtk"]
│   ├── __init__.py              # node class mappings
│   ├── pixal3d/                 # vendored copy of repo/pixal3d/ (no compiled extensions)
│   ├── stages.py                # heavy lifting: load, preprocess, camera, generate, extract
│   ├── nodes_loader.py          # LoadPixal3DPipeline
│   ├── nodes_inference.py       # Pixal3DPreprocessImage, Pixal3DEstimateCamera, Pixal3DGenerateGLB
│   └── utils/
│       └── hf_download.py       # _comfy_tqdm() shim
├── web/                         # populated at prestartup by copy_viewer("glb_three")
└── workflows/
    └── pixal3d_basic.json
```

---

## MVP Node Decomposition

Four nodes; reproduces `inference.py` end-to-end. Mirrors SAM3DObjects' Load → Estimate → Generate → Export pattern.

| Node | File | Wraps |
|------|------|-------|
| `Pixal3DLoadPipeline` | `nodes_loader.py` | (was) `init_pipeline()`. (Now planned) returns thin config dict only. |
| `Pixal3DPreprocessImage` | `nodes_inference.py` | `pipeline.preprocess_image()` (rembg + alpha bbox crop + 1024 resize). |
| `Pixal3DEstimateCamera` | `nodes_inference.py` | `get_camera_params_wild_moge()` — refactored to take a tensor (not a temp file). |
| `Pixal3DGenerateGLB` | `nodes_inference.py` | Fused cascade `pipeline.run()` + vertex-color GLB export via trimesh. `is_output_node=True`. |

---

## Verification status


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PozzettiAndrea/ComfyUI-Pixal3D](https://github.com/PozzettiAndrea/ComfyUI-Pixal3D) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
