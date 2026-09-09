---
trigger: always_on
description: **OpenNavMap**: a multi-session topometric mapping + image-goal navigation system.
---

# CLAUDE.md

## Overview

**OpenNavMap**: a multi-session topometric mapping + image-goal navigation system.
**LiteVLoc** (`third_party/litevloc_code`) is a **required** submodule that must be
initialized; it provides visual localization, the graph structures (`image_graph.py`,
`point_graph.py`, etc.), and the shared `utils/` helper functions.

When running any OpenNavMap script, `PYTHONPATH` must include both paths:
```bash
export PYTHONPATH=$(pwd)/python:$(pwd)/third_party/litevloc_code/python
```

## Common Commands

```bash
# Set up the environment (Python 3.8 + CUDA 11.8)
conda create --name opennavmap python=3.8
conda activate opennavmap
conda install pytorch=2.0.1 torchvision=0.15.2 pytorch-cuda=11.8 numpy=1.24.3 -c pytorch -c nvidia
pip install -r requirements.txt

# Verify the torch installation
python test_torch_install.py

# Build the ROS package (optional)
catkin build opennavmap -DPYTHON_EXECUTABLE=$(which python)

# Sanity-check the OpenNavMap core import
PYTHONPATH=$(pwd)/python:$(pwd)/third_party/litevloc_code/python python python/map_merge_pipeline.py --help

# LiteVLoc offline localization pipeline
PYTHONPATH=$(pwd)/third_party/litevloc_code/python python third_party/litevloc_code/python/loc_pipeline.py \
    --map_path <map_dir> \
    --query_data_path <query_dir> \
    --image_size 512 288 --device=cuda \
    --vpr_method cosplace --vpr_backbone=ResNet18 --vpr_descriptors_dimension=256 \
    --img_matcher master \
    --pose_solver pnp --config_pose_solver third_party/litevloc_code/python/config/dataset/matterport3d.yaml

# ROS online localization (simulation / real robot)
roslaunch litevloc run_vloc_online_simuenv.launch
roslaunch litevloc run_vloc_online_anymal.launch

# Map merging
bash scripts/run_map_merging.sh
```

## Directory Structure

```
python/
├── map_merge_pipeline.py   # main entry for multi-session map construction & merging
├── map_manager.py          # multi-graph coordination/management
├── utils_map_merging.py    # map-merging utilities (OpenNavMap-specific)
├── gen_covis_trav_edges.py # covis/trav edge generation script (OpenNavMap-specific)
├── benchmark_mms/          # multi-session mapping benchmark
├── benchmark_vpr/          # VPR evaluation
├── benchmark_kf_selection/ # keyframe-selection evaluation
└── benchmark_map_merge/    # map-merging evaluation

third_party/litevloc_code/python/
├── loc_pipeline.py         # LiteVLoc offline localization entry
├── ros_loc_pipeline.py     # LiteVLoc online localization ROS wrapper
├── global_planner.py       # global planning over the trav graph
├── pose_fusion.py          # odometry + visual-localization fusion
├── image_graph.py          # ImageGraph structure (shared with OpenNavMap)
├── point_graph.py          # PointGraph structure (shared with OpenNavMap)
├── utils/                  # shared helpers (used by both OpenNavMap and LiteVLoc)
└── config/dataset/         # YACS dataset configs (single source of truth)
```

## Map Data Format

```
map_root/
├── seq/                        # image frames
├── timestamps.txt              # img_name timestamp
├── intrinsics.txt              # per-frame: frame_path fx fy cx cy width height
├── poses.txt                   # per-frame: frame_path qw qx qy qz tx ty tz
├── poses_abs_gt.txt            # optional, absolute pose GT
├── gps_data.txt                # optional
├── iqa_data.txt                # optional, image quality assessment
├── edges_covis.txt             # [node_a, node_b, weight]
├── edges_odom.txt
├── edges_trav.txt
└── database_descriptors.txt    # VPR descriptors
```

**poses.txt (mapfree format):** `frame_path qw qx qy qz tx ty tz`
- world-to-camera: `R(q), t` transform a world point into the camera frame, i.e. `Rp + t`.
- `seq0/frame_00000.jpg` is always the identity pose; query poses are given relative to the reference frame.

## Released Datasets ↔ Experiments

The evaluation datasets are released on Google Drive
([data_release folder](https://drive.google.com/drive/folders/1Tpl3Leu0uo1b4iolLFpdfI5LO8CYCRe-);
human faces anonymized). Each dataset maps to one paper experiment:

| Dataset | Paper experiment | Task | Key metric |
|---------|------------------|------|------------|
| `vpr_eval` | Exp 1 — Topological Localization | place retrieval / loop closure over a reference map | Precision@1, Recall@1 @ `[7.5 m, 75°]` |
| `map_free_eval` | Exp 1 — Metric Localization | 6-DoF query pose w.r.t. reference images (Map-Free format) | Precision@`[100 cm, 10°]`, AUC |
| `map_multisession_eval` | Exp 2 & 3 — Map Merging | merge multi-session submaps into a globally consistent map | ATE (trans `[m]` / rot `[deg]`, RMSE) |

- Only raw benchmark inputs are released; `*_results_*`, `*_sfm_*`, `scene_stat`, and `.rrd` files are excluded.
- See [docs/instruction_benchmark_evaluation.md](docs/instruction_benchmark_evaluation.md) for the download list, test-time coverage, and run commands.

## benchmark_map_merge

- **Directory naming convention:**
  - Data directory: `s00000_aria_data_000`
  - SfM result: `s00000_sfm_netvlad_splg_{dist}` (`dist = f"{int(sfm_sample_dist*100):03d}"`, e.g. `0.25` → `_025`)
  - Merge result: `s00000_results_{order_tag}_{method}_{dist}` (no `_sba{n}` suffix)
  - No suffix is appended when `dist=0`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RPL-CS-UCL/OpenNavMap](https://github.com/RPL-CS-UCL/OpenNavMap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
