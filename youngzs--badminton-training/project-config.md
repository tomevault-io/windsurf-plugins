---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Badminton video pose detection and analysis tool using YOLOv8 and MediaPipe. The project provides both CLI and GUI interfaces to analyze badminton videos by detecting human pose skeletons, tracking sports equipment (shuttlecock, racket), and generating annotated output videos.

## Core Architecture

### Three Main Entry Points

1. **`pose_detection.py`** - Basic CLI implementation
   - Simple YOLOv8-pose detection
   - 17 COCO keypoints with skeleton rendering
   - Video processing with optional keyframe extraction

2. **`pose_detection_advanced.py`** - Enhanced CLI with multi-model support
   - Supports both YOLO (n/s/m/l/x variants) and MediaPipe
   - Object detection for ball/racket using YOLOv8
   - Performance metrics and FPS tracking
   - Audio merging using ffmpeg/moviepy

3. **`pose_detection_gui.py`** - Tkinter GUI wrapper
   - Inherits from `BadmintonAnalyzer` in `pose_detection_advanced.py`
   - Multi-threaded processing with progress callbacks
   - Configuration persistence via `gui_config.json`

### Class Hierarchy

```
PoseDetectorBase (abstract)
├── YOLOPoseDetector - YOLO-based pose detection
└── MediaPipePoseDetector - MediaPipe-based pose detection

ObjectDetector - YOLOv8 object detection for balls/rackets

BadmintonAnalyzer - Main video processor
└── CustomBadmintonAnalyzer - GUI-enhanced with callbacks
```

### Key Design Patterns

- **Strategy Pattern**: Pose detector selection (YOLO vs MediaPipe)
- **Template Method**: Common skeleton drawing with customizable parameters
- **Observer Pattern**: Progress/status callbacks for GUI integration

## Development Environment

### Setting Up

```bash
# Activate virtual environment
venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

### GPU/CUDA Setup (Critical for Training Version)

**Common Issue:** PyTorch installs CPU version by default, GPU shows as unavailable.

**Quick Fix:**
```bash
# Automated fix
fix_gpu.bat

# Or manual install
install_cuda_pytorch.bat
```

**Verification:**
```bash
python -c "import torch; print('CUDA:', torch.cuda.is_available())"
# Expected: CUDA: True
```

**Key Files:**
- `CUDA_INSTALL_GUIDE.md` - Detailed CUDA installation guide
- `QUICKSTART.md` - 5-minute setup guide
- `fix_gpu.bat` - Automatic GPU troubleshooting
- `install_cuda_pytorch.bat` - CUDA PyTorch installer
- `check_nvidia.bat` - Full environment diagnostics

### Virtual Environment

The project uses a Python virtual environment in `venv/`. All development should be done with this activated to ensure dependency isolation.

**Important:** Ensure PyTorch is installed with CUDA support for GPU acceleration. The training version (`pose_detection_training.py`) requires CUDA for optimal performance (10x speedup).

## Common Commands

### Running the Application

```bash
# Basic CLI usage
python pose_detection.py video.mp4 -o output.mp4

# Advanced CLI with MediaPipe
python pose_detection_advanced.py video.mp4 --pose-model mediapipe

# Advanced CLI with custom ball detection confidence
python pose_detection_advanced.py video.mp4 --ball-confidence 0.1 --object-model-size m

# Model comparison
python pose_detection_advanced.py video.mp4 --compare yolo-n,yolo-m,mediapipe

# Launch GUI
python pose_detection_gui.py

# Training mode with GPU acceleration (NEW)
python pose_detection_training.py video.mp4

# 2x slow motion for technique analysis
python pose_detection_training.py video.mp4 --slowmo 0.5 -o output_slow.mp4

# 4x slow motion with pose analysis
python pose_detection_training.py video.mp4 --slowmo 0.25 --analyze

# Benchmark GPU performance
python benchmark_gpu.py video.mp4 --frames 100
```

### Building Executable

```bash
# Using the build script
python build_exe.py

# Or manually with PyInstaller
pyinstaller --name=BadmintonVideoAnalyzer --onefile --windowed --noconfirm --clean pose_detection_gui.py
```

## Architecture Details

### Video Processing Pipeline

1. **Frame Capture**: OpenCV VideoCapture reads input video
2. **Pose Detection**: YOLO or MediaPipe processes each frame
3. **Object Detection**: YOLOv8 detects balls/rackets (optional)
4. **Skeleton Rendering**: Custom drawing with configurable line width/keypoint size
5. **Frame Writing**: OpenCV VideoWriter saves annotated frames
6. **Audio Merging**: ffmpeg or moviepy merges original audio to output

### Model Management

- **YOLO Models**: Auto-downloaded to `%USERPROFILE%\.ultralytics\weights\` on Windows
- **Model Variants**: n (nano, ~6MB) → s (small) → m (medium) → l (large) → x (extra-large, ~68MB)
- **Trade-offs**: Larger models = higher accuracy but slower processing

### Critical Detection Parameters

- **General confidence**: 0.25 (default for most objects)
- **Ball confidence**: 0.15 (lower threshold since shuttlecocks move fast and are small)
- **Person min confidence**: 0.7 (filters background people in GUI mode)

### Audio Handling Strategy

The code implements a two-stage audio merge:
1. Process video → temp file without audio
2. Merge original audio → final output
3. Attempts moviepy first, falls back to ffmpeg subprocess

### GUI-Specific Implementation


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [youngzs/badminton_training](https://github.com/youngzs/badminton_training) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
