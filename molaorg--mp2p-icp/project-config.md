---
trigger: always_on
description: Quick-start reference for AI agents and new contributors.
---

# mp2p_icp — Agent Context Guide

Quick-start reference for AI agents and new contributors.

## Project identity

**mp2p_icp** (Multi Primitive-to-Primitive ICP) is a C++ library and CLI toolkit for point cloud registration and map building. It is part of the [MOLA](https://github.com/MOLAorg/mola) framework.

- **Version**: 2.11.0
- **License**: BSD-3-Clause
- **Maintainer**: Jose Luis Blanco-Claraco
- **Minimum MRPT**: 2.15.4

Whenever a change is done to the library by an AI agent, keep this document up-to-date so it
always reflects the current state.

---

## Repository layout

Since mid-2026 this repo hosts 3 sibling ROS packages, split so that headless consumers
don't need to pull in GUI/display dependencies (`mrpt_libgui`):

```
mp2p_icp/                    (repo root)
├── mp2p_icp_core/            # ROS package: headless libs + CLI apps, no GUI deps
│   ├── mp2p_icp_common/       # Base utilities, Parameterizable class
│   ├── mp2p_icp_map/          # metric_map_t container, .mm file I/O, georeferencing
│   ├── mp2p_icp/               # ICP algorithms, Matchers, Solvers, QualityEvaluators
│   ├── mp2p_icp_filters/       # 38+ filters, 2 generators, voxel grid utilities
│   ├── apps/                   # 13 headless CLI applications (see below)
│   ├── tests/                  # 50+ gtest unit tests
│   ├── demos/                  # Example configs, demo point clouds
│   ├── 3rdparty/                # robin-map (vendored, header-only, private dep)
│   └── scripts/                 # formatter.sh, etc.
├── mp2p_icp_viz/              # ROS package: GUI apps (mm-viewer, icp-log-viewer)
│   └── apps/
├── mp2p_icp/                  # metapackage (backward compat, build_type cmake, no code): depends on both above
└── docs/                      # Sphinx documentation source
```

Standalone (non-colcon) plain-CMake builds are no longer supported; build via colcon
in a ROS workspace. Each library still exports its own CMake config (e.g.
`find_package(mp2p_icp_map REQUIRED)` + `mola::mp2p_icp_map`, unchanged), so consumers can
depend on individual libraries directly. New downstream ROS packages that only need the
headless libraries/CLI apps should use `<depend>mp2p_icp_core</depend>` (no `mrpt_libgui`);
`<depend>mp2p_icp</depend>` still pulls in everything (core + GUI apps), unchanged, via the
metapackage.

---

## Core data structure: `metric_map_t`

Defined in `mp2p_icp_map/include/mp2p_icp/metricmap.h`.

```cpp
struct metric_map_t {
    // Named point cloud layers (CPointsMap subclasses)
    std::map<layer_name_t, mrpt::maps::CMetricMap::Ptr> layers;

    // Geometric features
    std::vector<mrpt::math::TLine3D>  lines;
    std::vector<plane_patch_t>        planes;

    // Optional metadata
    std::optional<uint64_t>           id;
    std::optional<std::string>        label;
    mrpt::containers::yaml            metadata;

    // Optional georeferencing (added in .mm format v4)
    std::optional<Georeferencing>     georeferencing;
};
```

**Standard layer names** (defined as `PT_LAYER_*` constants):
- `"raw"` — original full point cloud
- `"plane_centroids"` — one point per extracted plane

### `metric_map_t::Georeferencing`

```cpp
struct Georeferencing {
    mrpt::topography::TGeodeticCoords   geo_coord;       // WGS-84 lat/lon/height of origin
    mrpt::poses::CPose3DPDFGaussian     T_enu_to_map;    // pose of {map} as seen from {enu}
};
```

**Convention**: `T_enu_to_map` is the pose of the map frame as seen from ENU. To transform a point from map → ENU, use `T_enu_to_map.mean.composePoint(ptMap)`. The "-" (inverse) is **not** needed for this direction.

**Coordinate conversion chain** (map → geodetic):
```
map coords → T_enu_to_map.mean.composePoint() → ENU
ENU → mrpt::topography::ENUToGeocentric() → geocentric
geocentric → mrpt::topography::geocentricToGeodetic() → WGS-84 lat/lon/alt
```

---

## .mm file format

- Binary, MRPT `CSerializable`-based, gzip-compressed
- Current serialization version: 5
- Load/save: `mm.load_from_file(path)` / `mm.save_to_file(path)`
- Inspect: `mm-info mymap.mm` or `mm-viewer mymap.mm`

---

## CLI applications

CLI argument parsing uses CLI11 (`find_package(CLI11 REQUIRED)`, `CLI11::CLI11`), not TCLAP.

| App | Package | Purpose |
|-----|---------|---------|
| `mm2las` | mp2p_icp_core | Export layers → LAS 1.4 (Point Format 8); supports `--frame map\|enu\|geodetic` |
| `mm2ply` | mp2p_icp_core | Export layers → PLY |
| `mm2txt` | mp2p_icp_core | Export layers → CSV/TXT |
| `mm2grid` | mp2p_icp_core | Generate grid maps from point clouds |
| `mm-filter` | mp2p_icp_core | Apply filter pipelines to .mm files |
| `mm-info` | mp2p_icp_core | Print .mm file summary |
| `mm-georef` | mp2p_icp_core | Inject/extract georeferencing from .mm files |
| `mm-viewer` | mp2p_icp_viz | GUI viewer for .mm files (needs `mrpt_libgui`) |
| `sm2mm` | mp2p_icp_core | Convert SLAM CSimpleMap → .mm |
| `sm-cli` | mp2p_icp_core | Manipulate CSimpleMap files (cut, join, export…) |
| `icp-run` | mp2p_icp_core | Run ICP pipeline from CLI |
| `icp-log-viewer` | mp2p_icp_viz | Debug ICP sessions interactively (needs `mrpt_libgui`) |
| `kitti2mm` | mp2p_icp_core | Convert KITTI .bin → .mm |
| `txt2mm` | mp2p_icp_core | Convert TXT/CSV point clouds → .mm |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MOLAorg/mp2p_icp](https://github.com/MOLAorg/mp2p_icp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
