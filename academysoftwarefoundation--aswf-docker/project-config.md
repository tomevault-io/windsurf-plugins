---
trigger: always_on
description: provides standardized build environments for VFX software across multiple
---

# CLAUDE.md - AI Assistant Guide for aswf-docker

## Project Overview

**aswf-docker** builds and manages Docker CI images and Conan packages for the
[Academy Software Foundation (ASWF)](https://www.aswf.io/) VFX Platform. It
provides standardized build environments for VFX software across multiple
platform years (2019-2026).

## Repository Structure

```
aswf-docker/
├── ci-*/                  # Docker image directories (18 images)
│   ├── Dockerfile         # AUTO-GENERATED - do not edit manually
│   ├── README.md          # AUTO-GENERATED - do not edit manually
│   └── image.yaml         # Source specification for the image
├── python/
│   └── aswfdocker/        # Main Python package
│       ├── cli/           # Click CLI entry point (aswfdocker.py)
│       ├── data/          # Jinja2 templates, versions.yaml
│       ├── tests/         # Unit tests (pytest)
│       ├── builder.py     # Docker buildx bake orchestration
│       ├── dockergen.py   # Jinja2 template rendering (Dockerfiles, READMEs, Conan profiles)
│       ├── index.py       # Version/package index from YAML
│       ├── releaser.py    # GitHub release automation
│       └── ...
├── packages/
│   ├── conan/recipes/     # 116+ Conan package recipes (vendored)
│   ├── conan/settings/    # Conan profile settings (profiles/ is auto-generated)
│   └── common/            # Shared build utilities
├── scripts/               # Shell utilities, VFX scripts, tests
├── .github/workflows/     # CI: python.yml, docker-builds.yml, release.yml, python-sonar.yml
├── versions.yaml          # Symlink to python/aswfdocker/data/versions.yaml
└── pyproject.toml         # Python project config (uv, pytest, black)
```

### Key Concepts

- **versions.yaml**: Central version registry defining package versions per VFX
  Platform year. Versions use inheritance (e.g., `2-clang10` inherits from `2`).
- **image.yaml**: Per-image specs listing packages, base image, and metadata.
- **Dockerfiles and Conan profiles are generated**: From Jinja2 templates via
  `aswfdocker dockergen`. Never edit `ci-*/Dockerfile`, `ci-*/README.md`, or
  `packages/conan/settings/profiles/*` directly.
- **Docker buildx bake**: Builds are orchestrated via JSON bake files generated
  by `python/aswfdocker/builder.py`.

## Development Setup

```bash
# Install uv (if not already installed)
curl -LsSf https://astral.sh/uv/install.sh | sh

# Install all dependencies (including dev)
uv sync --all-extras

# Run the CLI
uv run aswfdocker --help
```

## Common Commands

### Testing & Linting

```bash
# Run unit tests
uv run pytest python/aswfdocker

# Type checking
uv run mypy python/aswfdocker

# Linting (score must be >= 8.0)
uv run pylint python/aswfdocker

# Formatting (Black, targets Python 3.10)
uv run black python/ packages/conan/recipes/

# Run all pre-commit hooks
uv run pre-commit run --all-files

# Verify generated Dockerfiles and Conan profiles are up to date
uv run aswfdocker dockergen --check
```

### Building Images

```bash
# Build a specific image group and version
uv run aswfdocker build --ci-image-type IMAGE --group base --version 2026-clang20 --push NO

# Build Conan packages
uv run aswfdocker build --ci-image-type PACKAGE --group common --version 2026-clang20 --push NO

# Dry run (show commands without executing)
uv run aswfdocker build --ci-image-type IMAGE --group base --version 2026-clang20 --dry-run
```

### Regenerating Dockerfiles and Conan profiles

```bash
# Regenerate all Dockerfiles, READMEs, and Conan profiles from templates
uv run aswfdocker dockergen

# Check that all generated files are up to date (used in CI)
uv run aswfdocker dockergen --check
```

## Code Conventions

### Python

- **Python >= 3.10** required
- **Black** formatting (target: py310). Vendored Conan recipes in
  `packages/conan/recipes/` are excluded from Black.
- **MyPy** strict type checking (Python 3.11 target)
- **Pylint** with fail-under score of 8.0
- **Naming**: PascalCase classes, snake_case functions/variables, UPPER_CASE constants
- **Copyright header** on all files:
  `# Copyright (c) Contributors to the aswf-docker Project. All rights reserved.`
  `# SPDX-License-Identifier: Apache-2.0`

### Docker Images

- Image names: `ci-{name}` (e.g., `ci-base`, `ci-openexr`)
- Tags: `{major_version}` or `{major_version}-clang{N}` (e.g., `2026-clang20`)
- Docker orgs: `aswf` (production), `aswftesting` (testing), `aswflocaltesting` (local)
- OCI metadata labels are set automatically from templates

### Version Management

- All package versions are defined in `versions.yaml`
- Versions use parent inheritance: variant versions (e.g., `6-clang18`) inherit
  from base versions (e.g., `6`) and override specific fields
- The `major_version` field maps to VFX Platform years

## CI/CD Workflows

| Workflow | Trigger | Purpose |
|---|---|---|
| `python.yml` | Push/PR | Pre-commit hooks: black, pytest, mypy, pylint, dockergen --check |
| `docker-builds.yml` | Push/PR | Build Docker images and Conan packages (matrix by VFX year) |
| `python-sonar.yml` | Push to main | Code quality analysis with SonarCloud |
| `release.yml` | GitHub Release | Full build + push to Docker Hub |

## Git Conventions

- **Signed-off commits required**: Use `git commit -s`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AcademySoftwareFoundation/aswf-docker](https://github.com/AcademySoftwareFoundation/aswf-docker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
