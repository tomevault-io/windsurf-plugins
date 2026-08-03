---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What is HEAT

The **Heat flux Engineering Analysis Toolkit (HEAT)** is a Python suite for predicting heat flux incident on plasma-facing components (PFCs) in tokamaks. It combines CAD geometry, MHD equilibria, and multiple heat flux models (optical, ion gyro orbit, photon radiation, filaments, runaway electrons, 3D fields) into one framework. Developed by Tom Looby at Commonwealth Fusion Systems; used to design SPARC PFCs.

## Running HEAT

HEAT runs inside Docker. The published image is `plasmapotential/heat:<tag>` (current tag set in `.github/workflows/integration-tests.yml` → `HEAT_IMAGE_TAG`).

**Start the GUI (web app on localhost:8050):**
```bash
cd docker && docker compose up
```

**TUI/batch mode (inside container or from compose):**
```bash
docker run --rm -v "$(pwd):/root/source/HEAT" plasmapotential/heat:v4.2.7 \
  --m t --f /root/source/HEAT/tests/integrationTests/nstxuTestCase/batchFile_optical.dat
```

**Interactive shell in container:**
```bash
docker compose run --entrypoint "" HEAT /bin/bash
```

The `docker/docker-compose.yml` already mounts `~/HEAT` for data and the local repo source into the container, so local code changes are picked up immediately without rebuilding the image.

## Running Tests

All tests run inside the Docker container against the published image (they rely on the full compiled environment: FreeCAD, OpenFOAM, Open3D, Mitsuba).

**Smoke test (sanity check the mount):**
```bash
docker run --rm -v "$(pwd):/root/source/HEAT" --entrypoint "" \
  plasmapotential/heat:v4.2.7 \
  python3 /root/source/HEAT/tests/integrationTests/ciTest.py
```

**Single integration test case (e.g. optical):**
```bash
docker run --rm -v "$(pwd):/root/source/HEAT" \
  plasmapotential/heat:v4.2.7 \
  --m t --f /root/source/HEAT/tests/integrationTests/nstxuTestCase/batchFile_optical.dat
```

Available batch files in `tests/integrationTests/nstxuTestCase/`:
- `batchFile_optical.dat` — optical heat flux
- `batchFile_optical_elmer.dat` — optical + Elmer FEM thermal solve
- `batchFile_gyro.dat` — ion gyro orbit
- `batchFile_rad.dat` — photon radiation (also has golden assertions)
- `batchFile_rzq.dat` — R,Z,q|| profile from CSV
- `batchFile_optical_BYOM.dat` — bring your own mesh (STL input)

**Photon radiation golden checks:**
```bash
python3 tests/integrationTests/verify_nstxu_hf_rad_goldens.py \
  --workspace "$(pwd)" --docker-image plasmapotential/heat:v4.2.7

# or via pytest:
pytest tests/integrationTests/test_nstxu_hf_rad_goldens.py -v
```

CI runs all of the above automatically on push/PR to `main` (`.github/workflows/integration-tests.yml`).

**Updating goldens** when physics intentionally change: re-run the rad verifier, copy the printed "Parsed metrics" into `tests/integrationTests/nstxuTestCase/nstxu_hf_rad_goldens.json`.

## Release

```bash
./scripts/release.sh <IMAGE_TAG> [HEAT_REF] [--build]
# e.g.:
./scripts/release.sh v4.2.8 v4.3 --build
```

This updates `HEAT_IMAGE_TAG` in CI and the docker-compose image tags, optionally builds the image, then prompts you to push and open a PR to `main`.

## Architecture

### Entry point and modes

`source/launchHEAT.py` is the single entry point. It reads the `runMode` environment variable (`docker` or `local`), sets up all paths and `sys.path` entries for external tools (FreeCAD, ParaView, OpenFOAM, EFIT, Open3D), then hands off to either:
- `dashGUI.py` — Plotly Dash web application (GUI mode, `--m g`)
- `terminalUI.py` — batch/terminal mode (`--m t --f batchFile.dat`)

### Engine and physics modules

`engineClass.engineObj` is the central orchestrator. It owns one instance of every physics module and coordinates the time-stepping loop. All modules are instantiated in `engineClass.initializeEveryone()`:

| Engine attribute | Class | Role |
|---|---|---|
| `ENG.MHD` | `MHDClass.MHD` | Reads GEQDSK/gfile equilibria (via EFIT class), field-line mapping |
| `ENG.CAD` | `CADClass.CAD` | Loads STEP or STL geometry via FreeCAD, meshes PFCs |
| `ENG.HF` | `heatfluxClass.heatFlux` | Optical heat flux (Eich profile, multiExp, tophat, qFile) |
| `ENG.GYRO` | `gyroClass.GYRO` | Ion gyro-orbit heat flux tracing |
| `ENG.RAD` | `radClass.RAD` | Photon/radiation heat flux (uses Mitsuba for ray tracing) |
| `ENG.FIL` | `filamentClass.filament` | ELM filament heat and particle fluxes |
| `ENG.RE` | `runawayClass.Runaways` | Runaway electron module |
| `ENG.plasma3D` | `plasma3DClass.plasma3D` | 3D field perturbations via MAFOT/laminar |
| `ENG.OF` | `openFOAMclass.OpenFOAM` | OpenFOAM thermal conduction solver |
| `ENG.FEM` | `elmerClass.FEM` | Elmer FEM thermal solver (via gmsh) |
| `ENG.IO` | `ioClass.IO_HEAT` | Output: VTP meshes, point clouds, CSV |

`rayTracerClass` (extracted from `pfcClass.py`) provides the shared ray–mesh intersection kernels used by RAD, FIL, RE, and PFC shadow detection (wraps Open3D and Mitsuba).

### PFC object — the central data structure


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [plasmapotential/HEAT](https://github.com/plasmapotential/HEAT) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
