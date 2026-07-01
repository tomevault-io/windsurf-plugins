---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands

**Configure (macOS arm64 example):**
```bash
cmake . -B build_local -DCMAKE_PREFIX_PATH="/path/to/Qt/6.9.1/macos;/path/to/ogre/SDK_arm64" -DCMAKE_OSX_ARCHITECTURES=arm64
```

**Build:**
```bash
cmake --build build_local --target QtMeshEditor -j4
```

**Run (macOS):**
```bash
./build_local/bin/QtMeshEditor.app/Contents/MacOS/QtMeshEditor
```

**Build and run tests:**
```bash
cmake . -B build_local -DBUILD_TESTS=ON -DCMAKE_PREFIX_PATH="..."
cmake --build build_local --target UnitTests -j4
./build_local/bin/UnitTests                    # all tests
./build_local/bin/UnitTests --gtest_filter="Manager*"  # single test suite
```

**Run with MCP server:**
```bash
./build_local/bin/QtMeshEditor.app/Contents/MacOS/QtMeshEditor --with-mcp          # GUI + MCP
./build_local/bin/QtMeshEditor.app/Contents/MacOS/QtMeshEditor --mcp               # headless MCP only
./build_local/bin/QtMeshEditor.app/Contents/MacOS/QtMeshEditor --with-mcp --http-port 8080  # with HTTP API
```

**CLI pipeline (`qtmesh`):**
```bash
# A 'qtmesh' symlink is created automatically during build
qtmesh info model.fbx                          # show mesh info (text)
qtmesh info model.fbx --json                   # show mesh info (JSON)
qtmesh convert model.fbx -o model.gltf2        # convert between formats
qtmesh fix model.fbx -o fixed.fbx              # re-import/export with standard optimizations
qtmesh fix model.fbx --all                     # apply all extra fixes (remove degenerates, merge materials)
qtmesh anim model.fbx --list                   # list animations
qtmesh anim model.fbx --list --json            # list animations (JSON)
qtmesh anim model.fbx --rename "Take 001" "Idle" -o out.fbx  # rename an animation
qtmesh anim base.fbx --merge walk.fbx run.fbx -o merged.fbx
qtmesh anim model.fbx --resample 30 -o optimized.fbx  # resample to 30 keyframes
qtmesh anim model.fbx --decimate-step 5 -o lighter.fbx  # keep every 5th keyframe
qtmesh anim model.fbx --resample 30 --animation "Walk" -o out.fbx  # resample specific animation
qtmesh anim model.fbx --bake-fps 30 -o uniform.fbx     # re-grid every track to uniform 30 FPS
qtmesh anim model.fbx --bake-fps 60 --animation "Run" -o out.fbx  # bake one animation at 60 FPS
qtmesh anim model.fbx --in-between --gap-frames 30 -o filled.fbx  # AI in-betweening: fill the clip with 30 predicted keyframes (RMIB ONNX; smooth spline fallback) (#409)
qtmesh anim model.fbx --in-between --gap-frames 12 --start-time 0.5 --end-time 1.5 --animation "Jump" -o out.fbx  # fill a specific window of one animation
qtmesh anim model.fbx --in-between --gap-frames 12 --no-model -o out.fbx  # force the deterministic spline fallback (skip the ML model)
qtmesh pose model.fbx --animation "Walk" --time 0.5 -o posed.stl  # export single frame
qtmesh pose model.fbx --animation "Dance" --count 4 -o pose_%02d.stl  # export N evenly spaced frames
qtmesh turntable model.fbx -o turntable.png  # PNG sprite sheet (12 frames default)
qtmesh turntable model.fbx -o frame_%02d.png --frames 24 --axis y --camera-height 25
qtmesh isometric model.fbx -o iso.png  # 8-direction static sprite grid (rows=directions)
qtmesh isometric model.fbx --resolution 256 -o iso.png  # square 256px cells
qtmesh isometric model.fbx --animation "Walk" --frames 8 -o iso.png  # 8×8 animated atlas
qtmesh isometric model.fbx -o iso.png --elevation 35  # camera angle in degrees (default 30)
qtmesh isometric model.fbx -o iso.png --padding 1.5  # zoom out (auto-fit × 1.5)
qtmesh isometric model.fbx -o iso.png --camera-distance 5  # fixed orbit distance
qtmesh validate model.fbx                      # validate mesh (exit 1 if errors found)
qtmesh validate model.fbx --json               # validation results as JSON
qtmesh lod model.fbx --info                    # show LOD levels
qtmesh lod model.fbx --info --json             # LOD info as JSON
qtmesh lod model.fbx --count 3                 # generate 3 LODs → model_lod1.fbx, model_lod2.fbx, model_lod3.fbx
qtmesh lod model.fbx --count 2 --reductions 0.25,0.5 -o out.fbx  # custom reductions, named output
qtmesh lod model.fbx --count 3 --algo ogre -o out.fbx     # Ogre's MeshLodGenerator (default; better silhouette preservation in practice)
qtmesh lod model.fbx --count 3 --algo meshopt -o out.fbx  # meshoptimizer backend (preserves UV seams + skin weights, softer silhouette)
qtmesh lod model.fbx --auto                    # auto-generate LODs
qtmesh lod model.fbx --remove -o clean.fbx     # strip LODs and save
qtmesh material model.fbx --preset "Metallic-Roughness" -o out.fbx  # apply a built-in material preset (writes .material sidecar)
qtmesh material --list-presets                 # list built-in preset names (incl. PBR templates)
qtmesh material model.fbx --generate-texture "rusty bronze armor" -o out.fbx  # AI mesh-aware (depth-conditioned) texture → diffuse (needs SD build + base model; run `uv --unwrap` first if no UVs)
qtmesh material model.fbx --generate-texture "..." --model mybase.safetensors --controlnet-strength 0.8 --width 768 --height 768 -o out.fbx  # explicit SD base model + ControlNet strength + size

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fernandotonon/QtMeshEditor](https://github.com/fernandotonon/QtMeshEditor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
