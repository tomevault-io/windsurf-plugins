---
trigger: always_on
description: <!-- SPDX-FileCopyrightText: Copyright (c) 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved. -->
---

<!-- SPDX-FileCopyrightText: Copyright (c) 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved. -->
<!-- SPDX-License-Identifier: Apache-2.0 -->

# AGENTS.md - AI Agent Guide for USD Validation NVIDIA

This file gives AI coding agents the minimum context needed to work effectively with this repository. Use it as a
starting map, then go to `.agents/skills/` for task-level validation guidance.

## What This Repo Is

`usd-validation-nvidia` is a Python validation engine for OpenUSD assets. It provides:

- A Python package under `src/usd_validation_nvidia/`
- A command line validator, `nvidia_usd_validate`
- Rule, requirement, capability, feature, and profile registries
- JSON and CSV reporting for automation
- Entry-point based plugin discovery for external rule/profile packages

The package contains the validation engine and built-in validators. It is commonly used with generated profile
definitions; profile packages or custom plugins must be installed in the same Python environment as the engine so their
entry points are discovered.

## Start Here

- Read `README.md` for package installation and basic CLI/API examples.
- Read `.agents/skills/README.md` to understand the skill format and available task guides.
- For agents that do not automatically load `.agents/skills/`, such as Claude in some setups or Codex CLI when skills are not
  auto-discovered, explicitly read `AGENTS.md`, `.agents/skills/README.md`, and then the relevant `.agents/skills/*/SKILL.md` file
  before following a workflow.
- Use `nvidia_usd_validate --help` inside the target Python environment to discover the profiles, features,
  capabilities, requirements, categories, and rules actually registered there.

## Fresh Checkout Prerequisite

When running this repository from source with `--with .`, generate the ignored capabilities package first:

```bash
uv run \
  --no-project \
  --with usd-profiles-nvidia \
  python -m usd_profiles_nvidia.codegen \
    --docs-root specs \
    --destination-dir src \
    --package-name usd_validation_nvidia.capabilities \
    --reverse-domain com.nvidia.usd
```

Published wheels already include `src/usd_validation_nvidia/capabilities`; fresh source checkouts do not. Skipping this
step causes a hatchling `Forced include not found: .../capabilities` error when building or installing the local source
with `--with .`.

## Repo Layout (High Level)

- `src/usd_validation_nvidia/` - Python package source
- `src/omni/asset_validator/` - compatibility namespace for legacy `omni.asset_validator` imports
- `specs/` - source Markdown specs for capabilities, features, and requirements
- `examples/` - runnable examples referenced by skill files
- `src/usd_validation_nvidia/capabilities/` - generated package from `specs/`; do not edit by hand
- `tests/` - unit and CLI tests
- `.agents/skills/` - task-oriented agent skills (`*/SKILL.md`)

## Common Workflows

### Minimal Plugin Example

Install the repo and minimal plugin example into the run environment, then run the custom rule against the example
asset:

```bash
uv run \
  --with . \
  --with examples/python/minimal \
  nvidia_usd_validate --rule ExampleDefaultPrimChecker examples/assets/asset.usda
```

On Windows:

```powershell
uv run `
  --with . `
  --with examples\python\minimal `
  nvidia_usd_validate --rule ExampleDefaultPrimChecker examples\assets\asset.usda
```

### Requirement-Backed Plugin Example

Generate the example requirement package, then run the plugin with one custom requirement registered for CLI filtering
and JSON mapping:

```bash
uv run \
  --no-project \
  --with usd-profiles-nvidia \
  python -m usd_profiles_nvidia.codegen \
    --docs-root examples/python/requirement/specs \
    --destination-dir examples/python/requirement \
    --package-name example_requirements \
    --reverse-domain com.nvidia.usd
```

On Windows:

```powershell
uv run `
  --no-project `
  --with usd-profiles-nvidia `
  python -m usd_profiles_nvidia.codegen `
    --docs-root examples\python\requirement\specs `
    --destination-dir examples\python\requirement `
    --package-name example_requirements `
    --reverse-domain com.nvidia.usd
```

After codegen, run the plugin:

```bash
uv run \
  --with . \
  --with examples/python/requirement \
  nvidia_usd_validate --requirement EXAMPLE.001 examples/assets/asset.usda
```

To see a requirement-mapped failure:

```bash
uv run \
  --with . \
  --with examples/python/requirement \
  nvidia_usd_validate --requirement EXAMPLE.001 examples/assets/asset-missing-default-prim.usda
```

On Windows:

```powershell
uv run `
  --with . `
  --with examples\python\requirement `
  nvidia_usd_validate --requirement EXAMPLE.001 examples\assets\asset.usda
```

To see a requirement-mapped failure on Windows:

```powershell
uv run `
  --with . `
  --with examples\python\requirement `
  nvidia_usd_validate --requirement EXAMPLE.001 examples\assets\asset-missing-default-prim.usda
```

### CLI Example

Discover registered validation scopes in the active environment:

```bash
uv run nvidia_usd_validate --help
```

Run a smoke validation and write JSON:

```bash
mkdir -p reports
uv run nvidia_usd_validate \
  --json-output reports/asset.validation.json \
  examples/assets/asset.usda
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NVIDIA-Omniverse/usd-validation-nvidia](https://github.com/NVIDIA-Omniverse/usd-validation-nvidia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
