---
trigger: always_on
description: UNDER NO CIRCUMSTANCES SHOULD YOU EVER PUSH TO A REMOTE GIT REPOSITORY
---

# Copilot Instructions for gdalcli

UNDER NO CIRCUMSTANCES SHOULD YOU EVER PUSH TO A REMOTE GIT REPOSITORY

## Package Overview

`gdalcli` is a generative R frontend for GDAL's CLI (>=3.11). It provides auto-generated R wrapper functions for GDAL commands with lazy evaluation and composable pipelines.

## Architecture

### Two-Layer Design

1. **Frontend Layer** (User-facing R API)
   - Auto-generated functions from GDAL JSON API (`build/generate_gdal_api.R`)
   - Composable modifiers: `gdal_with_co()`, `gdal_with_env()`, etc.
   - S3 methods for extensibility
   - Lazy `gdal_job` specification objects

2. **Engine Layer** (Command Execution)
   - `gdal_run()` executes `gdal_job` objects
   - Uses `processx` for subprocess management
   - Environment variable injection for credentials
   - VSI streaming support (`/vsistdin/`, `/vsistdout/`)

### Key Design Patterns

- **Lazy Evaluation**: Commands built as `gdal_job` objects, executed only via `gdal_run()`
- **S3 Composition**: All modifiers are S3 generics that return modified `gdal_job` objects
- **Environment-Based Auth**: Credentials read from environment variables, never passed as arguments
- **Process Isolation**: Each command runs in isolated subprocess with injected environment
- **Pipe Composition**: Use native R pipe (`|>`) to compose jobs into pipelines naturally

### GDAL Version Conflicts and API Evolution

**GDAL 3.12+ Native Commands:**
- GDAL 3.12.0+ introduced native `gdal pipeline` command
- This conflicts with gdalcli's original `gdal_pipeline()` convenience wrapper function
- **Resolution**: Renamed function to `gdal_compose()`, marked deprecated for 0.5.x removal
- **Rationale**: 
  - Piping with `|>` is more idiomatic R for composition
  - Explicit type specification (`gdal_raster_pipeline()` vs `gdal_vector_pipeline()`) is clearer than type auto-detection
  - Function added minimal value over direct function calls
  - Users can still pass lists directly: `gdal_raster_pipeline(jobs = list(j1, j2, j3))`

**Deprecated Functions:**
- `gdal_compose()` - Deprecated as of 0.4.x, removal planned for 0.5.x
  - Issues warning via `.Deprecated()` on use
  - Docs recommend pipe approach instead
  - Will remove unless real-world use cases emerge

## CI/CD Workflows

The project uses GitHub Actions for automated testing, building, and deployment. Workflows are organized by purpose:

### Testing Workflows

**R-CMD-check-docker.yml** (Primary CI for main branch)
- **Purpose**: R CMD check in isolated Docker containers
- **Trigger**: Push to main, pull requests
- **Environment**: Custom Docker images with controlled GDAL versions (3.11.4 + latest 3.12.x)
- **Coverage**: Full R CMD check with dynamic API generation, vignettes, tests, optional features (Arrow, gdalg, explicit args)
- **When to use**: Standard CI for all code changes

**R-CMD-check-release.yml** (Release branch verification)
- **Purpose**: R CMD check for release branches
- **Trigger**: Automatically runs on `release/gdal-*` branches when updated by `build-releases.yml`
- **Environment**: Docker `deps-gdal-X.Y-amd64` image matching the branch's GDAL version
- **Coverage**: R CMD check using pre-committed generated API files; verifies package builds, passes checks, and loads
- **When to use**: Safety gate for releases (not triggered by contributors)

### Build Workflows

**build-docker-images.yml**
- **Purpose**: Build GDAL Docker images (base and runtime) for specific GDAL versions
- **Flexibility**: Accepts any GDAL version via `gdal_version` input in workflow_dispatch
  - When `gdal_version` provided: builds only that specific version
  - When triggered by schedule/push: builds default matrix (3.11.4, 3.12.0)
- **Trigger**: Weekly schedule (Saturdays), main branch pushes, manual dispatch
- **When to run**: New GDAL patch versions, when dependencies need updates, or to build arbitrary versions

**build-releases.yml**
- **Purpose**: Generate gdalcli for specific GDAL version and create releases with tagged commits
- **Branching Strategy**:
  - New minor version: Creates `release/gdal-X.Y` from main
  - Patch version: Continues on existing `release/gdal-X.Y` without reset to main
- **Releases**: Creates tags like `v0.3.0-3.12.0`, `v0.3.0-3.12.1` for each patch
- **Trigger**: Manual dispatch with `gdal_version`
- **When to run**: After building Docker image for new GDAL patch or minor version

### Workflow Selection Guide

| Scenario | Recommended Workflow | Notes |
|----------|---------------------|-------|
| Code changes (main) | R-CMD-check-docker.yml | Runs automatically on PRs to main |
| Release branch updates | R-CMD-check-release.yml | Runs automatically on `release/gdal-*` updates |
| New GDAL version | build-docker-images.yml → build-releases.yml | Build image first, then release |
| GDAL patch update | build-docker-images.yml → build-releases.yml | Same workflow for new patch |
| Docker issues | R-CMD-check-docker.yml | Isolated testing environment |
| Performance testing | R-CMD-check-docker.yml | Consistent environment |

### Manual Workflow Triggers

Some workflows require manual triggering via GitHub Actions:

- **build-docker-images.yml**: 
  - Optional `gdal_version`: Leave empty for default matrix, or specify any version (e.g., 3.12.1)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [brownag/gdalcli](https://github.com/brownag/gdalcli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
