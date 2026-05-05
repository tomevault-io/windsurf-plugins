---
trigger: always_on
description: **360FrameTools** is a unified desktop application that combines frame extraction from Insta360 cameras (.INSV/.mp4) with advanced perspective splitting and AI masking for photogrammetry workflows.
---

# 360FrameTools - AI Coding Agent Instructions

## Project Overview

**360FrameTools** is a unified desktop application that combines frame extraction from Insta360 cameras (.INSV/.mp4) with advanced perspective splitting and AI masking for photogrammetry workflows. 

**Three-stage pipeline**: EXTRACT FRAMES → SPLIT FRAMES → MASKING → DONE

This unified tool merges two separate applications:
1. **Frame Extraction Module**: Extracts frames from dual-fisheye .INSV files using official SDK (bypasses Insta360 Studio)
2. **360toFrame**: Converts equirectangular images to perspective views with compass-based positioning

**Key distinction**: This is a complete photogrammetry preprocessing pipeline—from raw Insta360 video to masked perspective frames—in one streamlined batch workflow.

## Project Structure

**Development location**: `C:\Users\User\Documents\APLICATIVOS\360ToolKit\`
**Source reference folders** (DO NOT MODIFY, read-only):
- `Original_Projects\Extraction_Reference\` - Frame extraction source
- `Original_Projects\360toFrame\` - Perspective splitting source

**Note**: Reuse/copy code from source folders. Ignore existing library implementations (`lib/` folders).

---

## Unified Application Architecture

### Stage 1: Frame Extraction (from Insta360toFrames)

**Input**: `.INSV` (dual-fisheye) or `.mp4` files from Insta360 cameras
**Output**: Equirectangular images (stitched panoramas)

**SDK Location**: `C:\Users\User\Documents\Windows_CameraSDK-2.0.2-build1+MediaSDK-3.0.5-build1`

**Four extraction methods** (PRIORITY ORDER):
1. **SDK Stitching** (PRIMARY - HIGHEST PRIORITY): Official Insta360 MediaSDK 3.0.5 for GPU-accelerated stitching
   - **Best quality**: AI-based stitching with seamless blending (chromatic calibration)
   - **Bypasses Insta360 Studio entirely**
   - **Direct .INSV → stitched equirectangular frames** at user-configurable FPS
   - **Stitch types**: AI Stitching (best), Optical Flow, Dynamic, Template
   - **Quality enhancements**: 
     * Chromatic calibration (EnableStitchFusion) - CRITICAL for seamless blending
     * FlowState stabilization (optional)
     * Color Plus enhancement (AI model)
     * Denoising and defringing
   - **Output formats**: JPG, PNG (SetImageSequenceInfo)
   - **GPU acceleration**: CUDA/Vulkan required (auto-detected)
   - **Model files**: ai_stitch_model_v1.ins (X3/X4), ai_stitch_model_v2.ins (X5)
   - **Frame extraction**: SetExportFrameSequence() for specific frame indices
2. **FFmpeg Method** (FALLBACK): Proven dual-stream filter chain (SDK-quality results without SDK)
3. **Dual-Fisheye Export**: Extract raw fisheye lens images (no stitching)
4. **OpenCV Method**: Frame-by-frame extraction (basic, no stitching, lowest quality)

**Key features**:
- **File analysis**: Display metadata from input file (duration, resolution, format, camera model)
- **Time range selection**: Start/end time in seconds (extract specific segment)
- **Interval-based extraction**: Frames per second (0.1 - 30 FPS)
- **Resolution options**: Original, 8K (7680×3840), 6K (6080×3040), 4K (3840×1920), 2K (1920×960)
- **Output format**: JPG (default) or PNG (SetImageSequenceInfo with IMAGE_TYPE enum)
- **Metadata preservation**: Camera info only (NO GPS/GYRO—discard those)
- **SDK quality settings**:
  * **AI Stitching** (BEST): GPU-accelerated AI-based seam blending, highest quality
  * **Optical Flow** (GOOD): High accuracy, moderate speed
  * **Dynamic Stitching** (BALANCED): Suitable for motion scenes
  * **Template** (FAST): Fast preview, lower quality for near-field scenes

### Stage 2: Perspective Splitting (from 360toFrame)

**Input**: Equirectangular images (from Stage 1 or external)
**Output**: Multiple perspective views (rectilinear photos)

**Compass-based camera positioning**:
- **Default**: 8 cameras, 110° FOV, horizontal ring
- **Look-up/Look-down**: Additional compass rings (e.g., +30°, -30° pitch)
- **Customizable**: User modifies split count, FOV, yaw/pitch/roll per ring

**Transform engines** (reuse from source):
- **`e2p_transform.py`**: Equirectangular → Pinhole perspective (spherical mapping + cache)
- **`e2c_transform.py`**: Equirectangular → Cubemap (6-face standard)
  - **NEW**: Support 8-tile cubemap variants
  - Configurable overlap and FOV

**Key features**:
- **Resolution options**: Custom width/height for output images
- **Output format**: PNG, JPEG, or TIFF
- **Real-time preview**: Dual preview system (see UI spec for details)
  - **Perspective mode**: Main camera preview + circular compass with clickable slices
  - **Cubemap mode**: Grid preview showing tile layout with overlap visualization

**InteractiveCircularCompass widget**:
- 4 camera states: Export (Blue), Preview (Yellow), Disabled (Red), Mask (Green)
- Click icons to cycle states
- Clickable slices (pizza-piece style) - each color represents a camera
- Optional look-up/look-down buttons create additional rings

### Stage 3: AI Masking (Enhanced)

**Current capabilities** (from 360toFrame):
- YOLOv8 instance segmentation (person detection)
- 5 model sizes: nano → xlarge
- Binary masks: 0 (Black/mask) = remove, 255 (White/keep) = valid


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Everton-Braz/360toolkit](https://github.com/Everton-Braz/360toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
