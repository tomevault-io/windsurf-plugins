---
trigger: always_on
description: handles this: filling left to right would put Back on the right whenever the button count
---

# AGENTS.md

Guidance for AI coding agents working in this repository. Human contributors may
find it useful too.

## Project

PeakNav ([peaknav.com](https://peaknav.com)) renders world mountains in 3D, with
paths, ways, and nearby peak names projected onto the terrain. It is a
[libGDX](https://libgdx.com/) cross-platform app built with Gradle, shipping on
Desktop, Android and iOS (the HTML target exists but is incomplete).

Terrain and OpenStreetMap data come from two HuggingFace datasets, retiled to the
[slippy map](https://wiki.openstreetmap.org/wiki/Slippy_map_tilenames) convention
(see `README.md`).

## Module layout

Gradle modules (`settings.gradle`): `core`, `desktop`, `android`, `ios`, `html`,
`headless`.

- **`core`** — all shared, platform-independent logic. Package root
  `com.peaknav`. Almost every change belongs here. Sub-packages of note:
  - `viewer/` — the libGDX app (`MapApp`), screens, camera, rendering, tiles.
  - `viewer/controller/` — `MapController` (the app-wide context, reached via
    `PeakNavUtils.getC()`), `CurrentLocation` (`getC().L`).
  - `compatibility/` — abstractions each platform implements (`NativeScreenCaller`,
    `LoadFactory`, etc.).
  - `elevation/`, `pbf/`, `database/`, `network/`, `satellite/`, `utils/`.
  - `geo/` — the map's own geometry: `Tile`, `LatLong`, `BoundingBox` and
    `MercatorProjection` (the web Mercator tile grid). Written for PeakNav, with no
    mapsforge left anywhere; `TestGeo` pins the behaviour the rest relies on - tile edges,
    clamping at the map's edge, neighbours wrapping round, and tile equality including the
    tile size (caches key tiles of size 1 and of `MapTile.TILE_SIZE` apart). `pbf/` reads
    the downloaded `.osm.pbf` tiles into its own `Way`, `PointOfInterest` and `Tag`.
  - `skyline/` — matching a photograph's skyline to the terrain: `TerrainHorizon`
    (the horizon all around a point, from an `ElevationSampler`), `SkylineExtractor`
    (the sky/ground line in a picture: `SkyFeatures` + `SkyClassifier`, a
    gradient-boosted forest read from the resource `sky_model.bin` next to it, give
    every pixel a sky probability; `BoundaryFeatures` + `boundary_model.bin` score
    every position as "the skyline passes here"; a minimum-cost path traces the
    boundary; both forests are trained by `tools/skyline_train.py`) and
    `SkylineMatcher` (bearing, pitch and field of view by optimisation, with a
    calibrated "confident" verdict). Pure Java, no libGDX; `viewer/PhotoSkylineAligner`
    is the app-side glue that runs it when a geotagged photo is loaded and offers
    to point the camera, or on demand from the photo bar's match button; debug
    builds (`LoadFactory.isDebugBuild()`) get a third button that saves the photo,
    pose and overlay as a dataset sample under `LoadFactory.getDebugSamplesDir()`.
    `gesture/PhotoPin` is the pinned-point state the input controller rotates and
    zooms around while a photo is shown. Its accuracy is measured, not assumed: see the
    `skylineBenchmark` tool below, and keep the thresholds in `SkylineMatcher`
    tied to what the benchmark reports.
  - `pistes/` — the ski slopes viewer, drawn from its own data layer `PBF_PISTES` (OpenSnowMap's
    extract, zoom-10 tiles in zoom-8 archives on the HuggingFace dataset beside the highways,
    queued with every download and served after the highways, before AREAS). `PisteRasterizer`
    writes each nearby tile's `SKI_SLOPES` texture - flow phase down the nearest downhill run as
    sine and cosine (oriented by the loaded terrain's heights at its ends, else by the way's
    direction), blue/red/black difficulty, coverage - in `TileRendererRunnerPistes`, on the
    roads' executor; the ski slopes block of `assets/fragment_shader.glsl` paints fat runs with a
    band flowing downhill on `u_time`, and piste areas as a translucent fill. `LiftRasterizer`
    writes the `SKI_LIFTS` texture from the same ways - travel phase uphill, the kind of lift, a
    distance field across the line - and the lifts block draws each kind its own way: red cabins
    for cable cars, orange for gondolas, white chairs, yellow handles on a dashed line for drag
    lifts, blue stripes for magic carpets. Runs and lifts are labelled through `RoadLabelPlanner.
    planPistes`/`planLifts` and `RoadNameRenderer`. Switched in the Roads "..." submenu: "Ski
    pistes" (`P.isSkiSlopesVisible()`), and in its "..." piste names, lifts and lift names; independent of
    `PISTES_IN_MAP_DATA`, which still gates the pistes the road textures would carry.
  - `roads/` — roads, tracks, trails and pistes, drawn by the GPU on every platform.
    `RoadClassifier` sorts the OSM ways of `PBF_HIGHWAYS` into classes (road rank, SAC trail
    difficulty, piste difficulty; tunnels, pavements and plazas dropped; relation tags from
    `PbfTileBinaryParser` read in groups so only hiking routes lend a trail their number).
    `RoadTileRasterizer` writes each tile's two **distance** textures - per class, how far every
    texel is from the nearest line, plus a dash phase and the difficulties - and
    `assets/fragment_shader.glsl` turns them into lines of any width, colour, outline and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Upabjojr/PeakNavApp](https://github.com/Upabjojr/PeakNavApp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-21 -->
