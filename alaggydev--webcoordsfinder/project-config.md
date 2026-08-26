---
trigger: always_on
description: handles.
---

# WebCoordsFinder Agent Context

This file is the durable project brief for AI agents working in this repository.
Read it before planning or changing the application.

## Premise

Minecraft selects the visual model variant of certain blocks deterministically
from an absolute world position. Depending on the block and visible face, this
can appear as a rotation or mirror of the texture.

Each observed face constrains the possible world coordinates:

- Top and bottom faces normally expose one of four model states.
- Side faces can fold those four states into two visibly distinct states.
- A useful screenshot normally needs roughly 24 or more independent
  observations to reduce a large search space effectively.

WebCoordsFinder is a local-first browser workbench for collecting and checking
that evidence. It can either export a native CoordsFinder configuration or run
the compatible single-threaded WebAssembly scanner locally in the browser.

## Goal

The application makes the screenshot-analysis and coordinate-search workflow
repeatable without uploading a screenshot or project:

1. Open a screenshot and inspect it with pan and zoom controls.
2. Mark a block-aligned base surface, then build a connected 3D mesh by
   selecting edges and extruding faces.
3. Calibrate the shared perspective and explicitly establish the world-up and
   horizontal axis mapping.
4. Anchor one block, label visible faces, manually confirm their variants or
   review automatic proposals, and configure the search.
5. Run a local background WASM search or download an exact
   `coordsfinder.conf` for the native CPU/CUDA CoordsFinder executable.

Screenshots, project documents, reference comparisons, and browser searches
remain on the current device. Vanilla reference textures and the scanner WASM
are bundled with the application.

## Scope Boundary

The editor deliberately assists a human analyst; it does not infer facts that
require user judgment. It does not currently include:

- Automatic discovery of block boundaries, vanishing points, or geometry from
  an image.
- A fully automatic block classifier.
- Automatic world-up or compass inference.
- GPU/CUDA execution in the browser.

Automatic texture matching is a proposal only. A user must confirm a proposed
variant before it becomes search evidence or is exported.

## User Workflow

The top navigation has three stages:

1. **Geometry** — draw four corners around the first block-aligned surface and
   set its dimensions. The editor saves a planar solve. Select one or more
   exposed edges and use **Extrude** to add connected block faces; sufficient
   non-coplanar observations promote the solve to a shared 3D camera
   projection. Use **Orient** to establish world UP and a horizontal direction,
   and **Anchor** to choose the block represented by `(0, 0, 0)`.
2. **Faces** — select one or more mesh faces; assign a bundled block profile,
   adjust per-block grass tint where supported, inspect a perspective-correct
   crop, and choose a visible variant. The Auto Analyze tab compares eligible
   selected faces with the bundled reference texture off the main thread. Its
   confidence threshold controls which suggestions are proposed; they can be
   reviewed, bulk-confirmed, corrected, or cleared.
3. **Export** — choose the texture algorithm, scan order, optional quarter-turn
   directions, inclusive bounds, error tolerance, and native CPU/CUDA tile
   settings. Review readiness and estimates, run the browser search, or export
   / copy the generated configuration.

Projects are autosaved in IndexedDB. The project library supports multiple
local projects, bundled examples, deletion, and portable `.wcf` project
import/export. Search progress, exact counters, and the first 1,000 browser
search matches are checkpointed with the project and can be resumed when the
search setup is unchanged.

## Geometry and Coordinate Model

The scene is a global integer lattice, not a collection of independent planes.
It stores:

- `MeshFace` entries as a canonical lattice corner plus an abstract local
  normal.
- Weighted `CalibrationObservation` entries mapping lattice corners to image
  points.
- A `PlanarProjection` (base-surface homography) or a fitted `CameraProjection`
  (row-major 3×4 projective matrix), both shared by every face.
- An explicit signed mapping from screenshot-local axes `a`, `b`, and `c` to
  world X/Y/Z labels.
- Persisted world-UP and directed horizontal orientation intents so a later
  projection change can rebuild the same user-confirmed world mapping.

The initial base grid is a resumable planar homography. Extruding from selected
edges adds connected faces and calibration observations. Once the evidence is
well-conditioned, the geometry code fits a camera projection; otherwise it
retains the stable planar solve. Do not reintroduce per-face independent
coordinate systems.

World orientation is intentionally explicit. Establishing UP constrains the
mapping first. The app may preselect one of the four remaining compass
rotations, while choosing a horizontal direction confirms it. Export,
canonical crops, and search require a complete parity-consistent mapping.

### Projected Parity and Right-Handedness

Do not impose a universal cross-product identity on the screenshot-local

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ALaggyDev/WebCoordsFinder](https://github.com/ALaggyDev/WebCoordsFinder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
