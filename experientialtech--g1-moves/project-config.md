---
trigger: always_on
description: This repository contains 59 motion capture clips for the Unitree G1 humanoid robot (mode 15, 29 DOF). The pipeline takes raw BVH motion capture from MOVIN3D and processes it through retargeting, training data generation, RL policy training, and archival.
---

# G1 Moves — Motion-to-Policy Pipeline

## Project Overview

This repository contains 59 motion capture clips for the Unitree G1 humanoid robot (mode 15, 29 DOF). The pipeline takes raw BVH motion capture from MOVIN3D and processes it through retargeting, training data generation, RL policy training, and archival.

**Robot**: Unitree G1, mode 15, 29 DOF
**Capture systems**: MOVIN TRACIN (markerless, LiDAR + vision), video2robot (monocular video)
**Training framework**: mjlab (MuJoCo-Warp + RSL-RL PPO)
**Workstation**: Dell Pro Max Tower T2, RTX PRO 6000 (96GB), Ubuntu 24.04

## Repository Layout

```
g1-moves/
  dance/                        28 clips
  karate/                       27 clips
  bonus/                         4 clips
  <category>/<clip>/
    capture/                    Original mocap
      <clip>.bvh                BVH motion (51-joint humanoid, 60 FPS)
      <clip>.gif                Preview GIF
      <clip>.mp4                Preview video
      <clip>_{bl,mb,ue,un}.fbx  FBX exports
    retarget/                   G1 retargeting
      <clip>.pkl                Retargeted joints (29 DOF)
      <clip>.csv                Same as PKL in CSV format
      <clip>_retarget.gif       Retarget preview
      <clip>_retarget.mp4       Retarget video
    training/                   RL training data
      <clip>.npz                Training-ready data
      <clip>_training.gif       Training visualization
      <clip>_training.mp4       Training video
    policy/                     Trained RL policy (when available)
      <clip>_policy.pt          PyTorch checkpoint
      <clip>_policy.gif         Policy rollout GIF
      <clip>_policy.mp4         Policy rollout video
      agent.yaml                PPO hyperparameters
      env.yaml                  Full environment config
      training_log.csv          Training metrics
  external/
    video2robot/                video2robot pipeline (monocular video → robot motion)
  manifest.json                 Per-clip metadata index
  quality_report.json           Automated validation
  generate_metadata.py          Regenerate metadata
  retarget_all.py               Batch retarget pipeline
  DATASET_CARD.md               Dataset documentation
```

## Key Paths

| What | Path |
|------|------|
| This repo | `~/Repositories/g1-moves` |
| mjlab-gui | `~/Repositories/mjlab-gui` |
| G1 URDF | `~/Repositories/g1-urdf` |
| MuJoCo XML | `~/Repositories/g1-urdf/g1_mode15_square.xml` |
| Training logs | `~/Repositories/mjlab-gui/logs/rsl_rl/g1_tracking/` |
| video2robot | `~/Repositories/g1-moves/external/video2robot` |
| GMR | `~/Repositories/g1-moves/external/video2robot/third_party/GMR` |
| PromptHMR | `~/Repositories/g1-moves/external/video2robot/third_party/PromptHMR` |

## Pipeline Stages

There are two input paths that converge at the PKL stage:
- **Path A (BVH)**: MOVIN TRACIN → BVH → retarget_all.py → PKL (Stage 1)
- **Path B (Video)**: Any video → PromptHMR → SMPL-X → GMR → PKL (Stage 0)

Both produce the same PKL format and feed into Stage 2+ identically.

### Stage 0: Video to PKL via video2robot

Extracts human pose from monocular video (YouTube, phone, etc.) and retargets to G1 robot joints. Alternative to the BVH pipeline for clips without mocap hardware.

**Input**: Any MP4 video with a visible full-body human
**Output**: `<category>/<clip>/retarget/<clip>.pkl`, `<clip>.csv`

**Environments**: Two separate conda envs required (conflicting deps):
- `phmr` (Python 3.11) — PromptHMR pose extraction (PyTorch 2.9+, xformers, SAM2, detectron2)
- `gmr` (Python 3.10) — GMR motion retargeting (MuJoCo, mink IK solver)

#### Step 1: Set up project directory

```bash
CLIP=V_MyClip
CATEGORY=bonus
V2R=~/Repositories/g1-moves/external/video2robot

# Create project folder with video
mkdir -p $V2R/data/$CLIP
cp /path/to/video.mp4 $V2R/data/$CLIP/original.mp4
```

Or download from YouTube:
```bash
yt-dlp -f "bestvideo[ext=mp4]+bestaudio[ext=m4a]/best[ext=mp4]/best" \
  -o "$V2R/data/$CLIP/original.mp4" "https://youtube.com/..."
```

#### Step 2: Extract pose (PromptHMR)

```bash
cd $V2R
conda run -n phmr python scripts/extract_pose.py \
  --project data/$CLIP --static-camera
```

**What it does**:
1. Converts video to H.264 if needed (AV1, VP9, etc.)
2. Runs person detection + SAM2 tracking
3. Estimates 3D human mesh (SMPL-X) per frame via PromptHMR
4. Exports `smplx.npz` with root_orient, pose_body, betas, trans

**Output**: `data/$CLIP/smplx.npz`, `smplx_tracks.json`, `results.pkl`, `world4d.glb`

**Flags**:
- `--static-camera`: Skip SLAM camera estimation (use for tripod/fixed-camera videos)

**Time**: ~2-5 min on RTX PRO 6000

#### Step 3: Retarget to G1 (GMR)

```bash
cd $V2R
conda run -n gmr python scripts/convert_to_robot.py \
  --project data/$CLIP --robot unitree_g1 --no-twist
```

**What it does**:
1. Loads SMPL-X body model, computes human height from betas
2. Per-frame IK: maps SMPL-X joints → G1 29-DOF joint angles

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [experientialtech/g1-moves](https://github.com/experientialtech/g1-moves) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
