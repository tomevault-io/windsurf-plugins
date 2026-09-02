---
trigger: always_on
description: This repository is a curated **Awesome-Embodied-3DV** list for the research space where 3D/4D perception, reconstruction, generation, simulation-ready assets, and embodied world models meet.
---

# AGENTS.md

## Mission

This repository is a curated **Awesome-Embodied-3DV** list for the research space where 3D/4D perception, reconstruction, generation, simulation-ready assets, and embodied world models meet.

The goal is not to be a generic 3D generation, NeRF, 3DGS, graphics, SLAM, or robot-learning bibliography. The goal is to organize the most relevant resources for:
- 2D/physical sensing signals that feed 3D systems: depth, normals, structured light, panoramic/fisheye perception, and dense mapping
- 3D/4D representations: 3DGS, 4DGS, NeRF/SDF, mesh, voxel, point-cloud, hybrid, and dynamic representations
- offline, streaming, online, semantic, instance-level, object-centric, scene-level, and dynamic 3D reconstruction
- object-level, part-level, articulated, scene-level, editable, and simulation-ready 3D generation
- embodied world models, dynamic scene graphs, 3D grounding, robotics integration, and sim-to-real asset pipelines
- datasets, benchmarks, metrics, simulators, and toolchains that help researchers choose what to use

## Curated-list and archive policy

All manually curated content lives in `README.md`.

`arXiv_daily/` is the sole exception: it is an automatically generated, high-recall candidate archive. It must never be presented as equivalent to the verified curated root README.

Do **not** create other `contents/` pages for paper organization.
Do **not** split the awesome list into multiple manually maintained markdown content files unless the user explicitly requests it.

Allowed root-level files:
- `README.md` - the canonical awesome list
- `AGENTS.md` - governance and maintenance rules
- `LICENSE`
- `.gitignore`
- optional static assets under `assets/` or `imgs/` when they improve presentation
- `arXiv_daily/` - generated candidate archive, configuration, scripts, tests, and data

## Canonical section layout

`README.md` should use this top-level structure:
1. `Data Perception`
2. `3D/4D Representation`
3. `3D Reconstruction`
4. `3D Generation`
5. `Embodiment & World Models`
6. `Datasets, Benchmarks & Infrastructure`

Use `About`, `Must Read`, `News`, and `Contents` near the top of the README. Keep subsection anchors stable.

## Section definitions

### Data Perception

Include the sensor-facing and feature-extraction layer:
- monocular, video, and multi-view depth estimation
- normal and geometry prior estimation
- active imaging, structured light, neural decoding, event/active stereo, and physical sensing
- panoramic, fisheye, egocentric, and wide-FOV perception when relevant to 3D reconstruction or mapping
- dense mapping systems that primarily provide features/signals for downstream 3D systems

Do **not** turn this into a generic 2D perception list.

### 3D/4D Representation

Include the underlying representation structures:
- structure-aware 3D Gaussian Splatting and geometry-oriented Gaussian variants
- voxel, mesh, point-cloud, triplane, tensor, and hybrid representations
- NeRF, SDF, neural implicit surfaces, and related neural rendering foundations
- 4DGS, deformation fields, canonical spaces, deformation graphs, and spatiotemporal representations

Usually exclude purely compression-oriented or renderer-engineering-only 3DGS papers unless they materially affect embodied reconstruction, dynamics, editability, or simulation use.

### 3D Reconstruction

Include systems that recover existing objects or scenes:
- feed-forward multi-view and pose-free reconstruction
- object-centric and scene-level reconstruction
- large-scale scene reconstruction and SLAM-like systems
- streaming and online reconstruction, including persistent state models
- dense semantic / instance mapping such as SAM3D, MV-SAM3D, and open-vocabulary 3D mapping
- dynamic, non-rigid, and 4D reconstruction such as PAGE-4D and dynamic VGGT variants

Do **not** include every classical SfM/MVS paper unless it is still a key baseline, dataset anchor, or conceptual reference.

### 3D Generation

Include methods that create new 3D assets or scenes:
- image/text-to-3D object generation with mesh, texture, materials, or simulation-ready outputs
- PBR material and texture generation when relevant to asset quality
- part-level decomposition, part-aware generation, assembly, and editing
- articulated object generation, kinematic structure prediction, URDF export, and interactive assets
- scene-level generation, procedural worlds, layouts, and targeted 3D scene editing

Do **not** broaden into all 3D editing, all diffusion-based 3D papers, or all stylized asset generation. Include selectively when there is a clear embodied-3DV use case.

### Embodiment & World Models

Include work that connects 3D assets and reconstructed scenes to interaction:
- dynamic scene graphs and object/part tracking
- topologically aware dynamic reconstruction and persistent 4D scene state
- reconstruction-based or 3D-consistent world models such as NeoVerse, Gen3C, Tessart/Tessera-style work
- action-conditioned dynamics synthesis
- robotics integration, sim-to-real through generated assets, and VLM/LLM agents grounded in 3D scenes


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chang-xinhai/Awesome-Embodied-3DV](https://github.com/chang-xinhai/Awesome-Embodied-3DV) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-02 -->
