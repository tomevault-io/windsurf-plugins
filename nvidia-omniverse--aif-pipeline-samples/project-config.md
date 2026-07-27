---
trigger: always_on
description: Sample scripts and presets for creating SimReady USD assets in NVIDIA Omniverse. Covers CAD ingestion, Scene Optimizer optimization, validation, and metadata workflows for AI Factory digital twin applications.
---

# AIF Pipeline Samples

Sample scripts and presets for creating SimReady USD assets in NVIDIA Omniverse. Covers CAD ingestion, Scene Optimizer optimization, validation, and metadata workflows for AI Factory digital twin applications.

## Repository Layout

| Directory | Purpose |
|-----------|---------|
| `cli/` | Unified `aif-pipeline` CLI tool (convert, optimize, validate, metadata, run, config) |
| `scripts/` | Pipeline scripts for batch processing; `scripts/examples/` has simpler demos |
| `so/` | Scene Optimizer presets — `generic/` template plus vendor examples (`vertiv/`, `spt/`, `trane/`) |
| `so/generic/lib/` | Reusable Python processor scripts for presets |
| `oav/` | Custom USD validation rules (standalone, no Kit/GPU required) |
| `metadata/` | Metadata templates and tools for AIF equipment properties |
| `connections/` | Connection points workflow (thermal, electrical, airflow interfaces) |
| `asset_processor/` | Browser-based visual preset builder for Scene Optimizer |
| `docs/` | Sphinx/MyST documentation source |
| `assets/` | Sample USD assets |

## Core Workflow

The standard pipeline is: **Convert** → **Optimize** → **Validate** → **Metadata**

```bash
# Full pipeline in one command
aif-pipeline run input_cad/ output/ --spec scripts/data/creo_spec.json --preset so/generic/generic_preset.json

# Or step by step
aif-pipeline convert input_cad/ output_usd/ --spec scripts/data/creo_spec.json
aif-pipeline optimize output_usd/ optimized/ --preset so/generic/generic_preset.json
aif-pipeline validate optimized/ validation/
aif-pipeline metadata apply metadata.json --output metadata.usda --prim Root
```

All commands require `uv run` prefix or an activated venv (`.venv/Scripts/Activate.ps1` on Windows, `source .venv/bin/activate` on Linux).

## Agent Knowledge Files

When working with USD assets or the pipeline, read these rule files for domain-specific guidance:

| File | When to Read |
|------|-------------|
| `.cursor/rules/aif-pipeline-cli.mdc` | **Always** — CLI command reference with natural language mappings |
| `.cursor/rules/usd-universal.mdc` | Validation fails, user reports visual artifacts, or you need to map a failed rule to a Scene Optimizer fix |
| `.cursor/rules/usd-aif-profile.mdc` | User mentions metadata, connection points, equipment types (CDU/CRAH/UPS/GB300), or `aif:` properties |
| `.cursor/rules/scene-optimizer-presets.mdc` | User wants to create, edit, customize, or understand a Scene Optimizer preset JSON |
| `.cursor/rules/usd-issues-catalog.mdc` | User reports a specific error, rendering problem, or you need to diagnose a known failure pattern |

## Conventions

- **License headers:** All source files use SPDX MIT headers (`SPDX-FileCopyrightText: Copyright (c) 2024-2026 NVIDIA CORPORATION & AFFILIATES`)
- **Python:** 3.10-3.12, managed with `uv`; snake_case naming
- **Package manager:** `uv` (not pip) — `uv sync` to install, `uv run` to execute
- **Documentation:** Markdown with MyST syntax (Sphinx-compatible); use regular hyphen dashes (` - `), not em/en dashes
- **Presets:** JSON arrays of operation objects in `so/` directory
- **Metadata namespaces:** `aif:core:` (common properties), `aif:spec:` (equipment-specific)

## Agent Behavior

### General Principles

- **Check environment first:** Run `aif-pipeline config show` before any Kit-dependent command (convert, optimize, Kit-based validate). If Kit is not configured and the task requires it, guide setup with `aif-pipeline config add <name> --from <kit-root>` before proceeding.
- **Prefer non-destructive operations:** When optimizing, default to a separate output directory unless the user explicitly asks for in-place.
- **`uv run` prefix:** All commands need `uv run` prefix unless the user has activated a venv.

### Conversion Requests

When a user asks to convert CAD files to USD:

1. **Determine CAD format** to select the correct spec file:
   - Creo/PTC (`.prt`, `.asm`): `scripts/data/creo_spec.json`
   - JT (`.jt`): `scripts/data/jt_spec.json`
   - DGN (`.dgn`): `scripts/data/dgn_spec.json`
2. **Check Kit config** - conversion requires Kit.
3. If no Kit config exists, guide setup first.
4. Confirm input directory, output directory, and spec before running.

### Optimization Requests

When a user asks to optimize USD assets:

1. **Ask about the preset:**
   - Generic: `so/generic/generic_preset.json` (good default for most CAD assets)
   - Vendor-specific: check `so/vertiv/`, `so/spt/`, `so/trane/` for existing presets matching their equipment
2. **Ask destructive vs. non-destructive:** separate output directory (default) or in-place.
3. **Check Kit config** - optimization requires Kit.
4. If they want to customize the preset, read `.cursor/rules/scene-optimizer-presets.mdc` for the operation catalog and ordering rules.

### Validation Requests

When a user asks to validate assets:

1. **Check for an active Kit config** by running `aif-pipeline config show`. If Kit is already configured, offer both validation paths directly.
2. **Ask which path** they want to use:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NVIDIA-Omniverse/aif-pipeline-samples](https://github.com/NVIDIA-Omniverse/aif-pipeline-samples) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
