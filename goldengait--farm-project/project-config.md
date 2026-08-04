---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Official implementation of **FARM: Find Anything using Relational Spatial Memory** (arXiv:2606.15476) — real-time 3D scene-graph construction and language-based retrieval from RGBD streams. Objects are detected/segmented (YOLOE, open-vocabulary), tracked as 3D Gaussians with sparse voxel evidence, merged across frames (DINOv3 features + Hellinger distance + union-find), captioned asynchronously (vLLM), embedded multi-modally, and queried with relational language (LLM query parsing → spatial predicate execution).

Online (ROS 2) and offline (datasets) share the **same algorithm code** — `StreamingMapper` runs in both; only the frame ingress differs. The layout separates the ROS-free `src/scene_graph/` library from the thin `ros/` integration layer. The Python package imports as `scene_graph`.

## Build & Run Commands

All runtime work happens inside the Docker container; there is no supported host-side pip/uv/conda path. See `README.md` for full setup; summary:

```bash
git submodule update --init --recursive   # (host, once) third_party/yoloe
./bootstrap_models.sh                      # (host, once) YOLOE + MobileCLIP + SigLIP2 from public URLs
./run.sh build                             # CUDA 12.8 + ROS 2 Humble image
./run.sh shell /path/to/data               # container shell; mounts the dir at /data
```

The entrypoint sources ROS, activates `~/.venv`, and colcon-builds `mapping_msgs` + `mapping` against the bind-mounted tree.

### Offline mapping

```bash
python -m scene_graph.offline.run --source sens --sens-path /data/scans/scene0000_00/scene0000_00.sens --stride 5 --save-path /data/out/scene0000_00.pt --covisibility
python -m scene_graph.offline.run --source frames-json --frames-json-dir /data/scenes/grandtour/2024-11-25_warehouse --save-path /data/out/warehouse.pt --covisibility
```

Sources: `sens`, `rosbag`, `npz`, `frames-json` (FARM-Scenes format — uint16-PNG depth decoded via the scene's `depth_encoding` block). Add `--caption` (needs `./run.sh vllm`), `--viser` (live 3D view :8080, ~6× throughput cost), `--regions`, `--extra-param key:=value` passthrough.

For Replica-style PNG-depth datasets driven by a typed `PipelineConfig` YAML: `python scripts/run_pipeline.py --config configs/replica.yaml`.

### View + query a saved scene state

```bash
python scripts/view_scene_state.py --pt /data/out/warehouse.pt [--cloud /data/scenes/.../cloud.npz]   # viser :8080, incl. Query panel
python scripts/query_scene_graph.py --pt /data/out/warehouse.pt --query "a backpack"                  # CLI retrieval (needs embed server)
```

Python API: `SceneGraphRetriever.from_scene_state(pt, embedder=EmbedInterface())` for embedding retrieval; `parse_query` + `execute_spatial_query` (`scene_graph.retrieval.spatial_reasoning`) for the full relational pipeline the paper evaluates.

### Online ROS 2

```bash
./run.sh ros2 caption_enabled:=true    # vLLM servers + scenegraph_validation_exploration.launch.py
```

One `frame_pub` per camera (topics from `CAMERA_CONFIG` in `src/scene_graph/camera_config.py`) feeds the single `streaming_mapper`, which batches by `camera_names` / `expected_batch`. `mapping_five_cam.launch.py` does this for the 5 Spot cameras. Depth-less LiDAR sensors work the same way: `odin1_depth_pub` projects the `PointCloud2` into the camera and republishes RGBD topics (`ros2 launch mapping mapping_odin1.launch.py`).

Registered executables (`ros/mapping/setup.py`): `streaming_mapper`, `frame_pub`, `odin1_depth_pub`, `visual_search_yoloe_text_prompt`. `nodes/tf_listener.py` is a helper import of `frame_pub`, not an executable.

### run.sh / in_docker.sh

- `./run.sh build | shell [dir] | vllm | ros2 [args...] | stop`
- `./run.sh vllm` starts three servers in tmux: **Qwen3.5-9B** :8000 (captioning + query parsing), **Qwen3-Embedding-0.6B** :8002, **Qwen3-VL-Embedding-2B** :8006. GPUs via `GPU_VL8`/`GPU_EMB`/`GPU_VL_EMB`.
- `./scripts/in_docker.sh <cmd>` runs a command in a long-lived container (default name `scene-graph-batch`, override `SG_CONTAINER`); rewrites `python` to the venv python and forwards the vLLM/SigLIP2 env vars.

### Models

`models/dinov3-vits16` (DINOv3 ViT-S/16 merge backbone) is **committed in-repo** with its license, so a fresh clone maps fully offline. `bootstrap_models.sh` fetches the rest from official public sources — YOLOE (`hf.co/jameslahm/yoloe`), MobileCLIP (Apple CDN), SigLIP2 (`scripts/download_siglip2.py`). No Git LFS, no HF account. The gated `dinov3-vits16plus` (paper backbone, tighter merging) is auto-preferred by `resolve_dino_backbone()` whenever `models/dinov3-vits16plus/` exists — see README. Both variants are `hidden_size=384`, so switching changes only merge-feature quality. **Each model has its own license — `THIRD_PARTY_NOTICES.md`.**

## Architecture

- **`src/scene_graph/`** — core library, zero ROS imports.
  - `offline/` — driver (`run.py`) + frame sources (`frame_sources/{sens,rosbag,npz,frames_json}.py`), `viser_recording.py`.
  - `pipeline/` — `PipelineOrchestrator` + pure-function `steps.py` shared with the ROS node.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GoldenGait/FARM-Project](https://github.com/GoldenGait/FARM-Project) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-31 -->
