---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Purpose

A USD scene toolset for **asset splitting** and **physics simulation preprocessing**: normalize raw scenes into a reusable model/material library with scene reference structures, then batch-assign collision bodies, rigid bodies, and joints for Omniverse Isaac Sim interaction and navigation simulation.

## Environment

- **Core dependencies**: `usd-core` (pxr), `numpy`, `pandas`
- **Physics preprocessing** (interaction/navigation): requires Omniverse Isaac Sim (`isaacsim`, `omni.*`)
- **Doc management**: `pip install pyyaml`

Isaac Sim scripts must be run via the Isaac Sim Python wrapper:
```bash
# Auto-detects Isaac Sim install; or set ISAAC_SIM_ROOT=/path/to/isaac_sim-<version>
./scripts/isaac_python.sh <script.py> [args...]
```

## Common Commands

**Step 1 – Scene splitting** (standard Python, no Isaac Sim):
```bash
python clean_data.py
# Reads home_scenes/<scene_id>/{start_result_fix.usd|start_result_new.usd}
# Writes target/ (Materials, models, scenes)
```

**Step 2 – Interaction physics** (requires Isaac Sim):
```bash
# Edit scene_path/dest_scene_path in the script first
./scripts/isaac_python.sh set_physics/preprocess_for_interaction.py
# Output: start_result_dynamic.usd
```

**Step 2 alt – Navigation physics** (requires Isaac Sim):
```bash
# Edit paths first
./scripts/isaac_python.sh set_physics/preprocess_for_navigation.py
# Output: start_result_navigation.usd
```

**Step 3 – Export/package**:
```bash
./scripts/isaac_python.sh set_physics/get_all_references.py
./scripts/isaac_python.sh set_physics/export_scene.py
```

**One-shot SimReady CLI** (Option B – no manual splitting needed):
```bash
./scripts/isaac_python.sh -m set_physics.simready --input-usd /path/to/scene.usd
```

**External /root-structured scenes** (Option C – for SimBench/GRScene datasets):
```bash
./scripts/isaac_python.sh scripts/prep_interaction_root_scene.py \
  --input /path/to/scene.usd \
  --output /path/to/scene_interaction_dynamic.usd
```

**Normalized export** (specs_normalizer):
```bash
# GRScenes
python -m specs_normalizer --src-target ./target --dst-root ./export_specs \
  --asset-name GRScenes_assets --scene-name GRScenes100 --scene-category home --with-annotations
# MesaTask
python -m specs_normalizer --src-target ./target --dst-root ./export_mesa_specs \
  --asset-name MesaTask_assets --scene-name MesaTask --scene-category office_table
```

**Fix MDL/texture absolute paths in normalized assets**:
```bash
python scripts/fix_normalized_mdl_paths.py \
  --dataset-root /path/to/GRScenes-test1-normalized \
  [--materials-dir /path/to/GRScenes-test1-normalized/Material/mdl] \
  [--category bottle] [--dry-run] [--workers 8]
```

**Fix portable texture paths**:
```bash
./scripts/isaac_python.sh scripts/collect_textures_to_local_textures.py \
  --input /abs/path/to/scene.usd \
  --output /abs/path/to/scene_textures_fixed.usd \
  --report /abs/path/to/texture_rewrite_report.json
```

**Shape-invariant dedup** (complementary to geom_only):
```bash
python scripts/report_asset_mesh_dedup.py \
  --dataset-root ./GRScenes-test1-normalized \
  --dataset-name GRScenes_assets \
  --category bottle \
  --mode shape_invariant \
  --hausdorff-threshold 0.05 \
  --merge-tolerance 0.005 \
  --out-dir check_reports/shape_invariant/
```

**Documentation maintenance**:
```bash
python scripts/doc_manager.py --find-refs simready.py   # find docs referencing a code file
python scripts/doc_manager.py --validate                 # validate doc metadata
python scripts/doc_manager.py --gen-index               # regenerate docs/INDEX.md
```

## DLC Remote Job Submission

Scripts in `scripts/dlc/` submit physics preprocessing jobs to Alibaba Cloud PAI-DLC GPU workers using the Isaac Sim 4.5.0 container image; inside the container the `dlc-operator` Claude agent can be used for job monitoring and submission automation.

**Single job**:
```bash
bash scripts/dlc/launch_job.sh <name> <chunk_id> <chunk_total> [data_sources] [command_args]
```

**Batch submission** (one job per chunk):
```bash
python scripts/dlc/submit_batch.py --name <name> --total <N> --mode <mode> [--command_args "..."]
```

**Execution modes** (first arg to `run_task.sh`):
- `interaction` – interaction physics preprocessing (Isaac Sim)
- `navigation` – navigation physics preprocessing (Isaac Sim)
- `simready` – one-shot SimReady CLI (Isaac Sim)
- `prep_root_scene` – external `/root`-structured scene prep (Isaac Sim)
- `normalize` – normalized export via `specs_normalizer` (Isaac Sim Python)
- `normalize_assets` – asset transform normalization (recenter + Y-up→Z-up) via `normalize_asset_transforms.py` (Isaac Sim Python)
- `dedup` – asset dedup report by category chunk via `scripts/dlc/dedup_by_category.py` (Isaac Sim Python)
- `clean` – scene splitting via `clean_data.py` (Isaac Sim Python)
- `custom` – pass any script directly to Isaac Sim Python
- `batch` – chunk-based dispatch (default; passes extra args to Isaac Sim Python)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jandan138) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
