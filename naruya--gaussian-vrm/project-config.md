---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**gs-edit** is a web-based 3D application that combines Gaussian Splatting (3DGS) point clouds with VRM (Virtual Reality Model) characters to create animated rigged avatars. The system processes PLY files containing Gaussian splats and binds them to VRM character skeletons, enabling real-time animation of photorealistic 3D scans.

## Core Technology Stack

- **Three.js** (v0.170.0): 3D rendering engine
- **Gaussian Splats 3D**: Custom library for rendering Gaussian splatting point clouds (`lib/gaussian-splats-3d.module.js`)
- **@pixiv/three-vrm** (v2.1.0): VRM character model support
- **TensorFlow.js + Pose Detection**: Used for automated pose estimation during preprocessing
- **JSZip**: For GVRM file format (zipped archive containing VRM + PLY + metadata)

## Architecture

### Directory Structure

```
gvrm-format/          # GVRM file format handlers and core algorithms
  ├── gvrm.js         # Main GVRM class, load/save/update logic
  ├── vrm.js          # VRM character management
  ├── gs.js           # Gaussian splatting viewer wrapper
  ├── ply.js          # PLY file parser
  └── utils.js        # Utilities (pose operations, visualization, PMC)

apps/                 # Application-specific modules
  ├── preprocess/     # Preprocessing pipeline
  │   ├── preprocess.js    # Main preprocessing orchestration
  │   ├── preprocess_gl.js # GPU-accelerated splat assignment
  │   ├── pose.js          # Pose detection wrapper
  │   ├── check.js         # Validation and error checking
  │   └── utils_gl.js      # WebGL utilities
  ├── avatarworld/    # Avatar World demo application
  │   ├── main.js          # Entry point for avatar world
  │   ├── walker.js        # Autonomous avatar walking behavior
  │   ├── scene.js         # Scene creation (sky, floor, houses)
  │   └── index.html       # Avatar world HTML page
  ├── fps.js          # FPS counter
  └── vr.js           # VR mode support

lib/                  # Third-party libraries (TensorFlow.js, pose detection, Three.js modules)
  └── gaussian-vrm.min.js  # Minified GVRM library for distribution
assets/               # Sample GVRM files, FBX animations, default configurations
examples/             # Example usage (simple-viewer.html)
main.js               # Main application entry point
index.html            # Main HTML page
server.js             # HTTPS development server
error.js              # Error logging and reporting utilities
```

### Key Concepts

**GVRM Format**: A proprietary format (`.gvrm` files) that packages:
- `model.vrm`: VRM character model
- `model.ply`: Gaussian splat point cloud (PLY format)
- `data.json`: Binding metadata including:
  - `splatVertexIndices`: Maps each splat to nearest VRM mesh vertex
  - `splatBoneIndices`: Maps each splat to VRM skeleton bone
  - `splatRelativePoses`: Relative positions of splats to their bound vertices
  - `boneOperations`: Pose adjustments applied to VRM skeleton
  - `modelScale`: Scale factor applied to VRM model

**PMC (Points/Mesh/Capsules)**: Visualization helpers showing:
- Points: VRM mesh vertices in world space
- Mesh: VRM mesh wireframe
- Capsules: Bone visualizations for debugging splat assignments

**Dynamic Scene Sorting**: Splats are split into multiple Three.js scenes, grouped by bone or vertex indices, to optimize rendering and enable per-bone transformations.

## Development Commands

### Running the Application

**Prerequisites**:
- Node.js and npm installed
- Express.js: `npm install -g express` (or install locally)

**SSL Certificate Setup** (required for HTTPS):
```bash
# Generate self-signed certificates
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes -subj "/CN=localhost"
mkdir -p .server
mv *.pem .server/
```

**Start Development Server**:
```bash
# Start HTTPS development server (required for WebRTC and some browser features)
node server.js
# Access at https://localhost:8080
```

**Applications**:
- Main editor: `https://localhost:8080/`
- Simple viewer: `https://localhost:8080/examples/simple-viewer.html`
- Avatar World demo: `https://localhost:8080/apps/avatarworld/`

### Loading Files

**URL Parameters** (main.js):
- `?gs=<path>` - Load PLY file for preprocessing
- `?gvrm=<path>` - Load existing GVRM file
- `?stage=<0|1|2|3>` - Skip preprocessing stages (for debugging)
- `?vr` - Enable VR mode
- `?gpu` - Use GPU-accelerated preprocessing
- `?nobg` - Remove background splats
- `?nocheck` - Skip validation checks
- `?saveply` - Save processed PLY alongside GVRM
- `?customtype=<float>` - Custom shader material type
- `?size=<width>,<height>` - Canvas size override

**Note**: VRM model is fixed to `./assets/sotai.vrm`. VRM scale is automatically calculated from the Gaussian splat height.

**File Selection**: If no `gs` or `gvrm` parameter provided, UI prompts for file selection (supports drag-and-drop for `.ply` or `.gvrm` files).

### Keyboard Controls (main.js:272-328)

- **Space**: Pause/play animation
- **V**: Update PMC (Points/Mesh/Capsules) visualization
- **C**: Cycle splat color modes (empty/assign/original)
- **X**: Toggle VRM mesh visibility

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [naruya/gaussian-vrm](https://github.com/naruya/gaussian-vrm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
