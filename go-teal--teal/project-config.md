---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Building and Installing
```bash
# Build the teal CLI tool
make

# Install teal CLI globally
make install

# Or directly with go
go build -o bin/teal ./cmd/teal
go install ./cmd/teal
```

### Releasing
Releases are automated via GoReleaser. Pushing a `v*` git tag triggers `.github/workflows/release.yml`, which builds cross-platform CLI binaries (linux/darwin × amd64/arm64), injects the tag into `pkg/configs.TEAL_VERSION` via `-ldflags`, auto-generates a grouped changelog from commits since the previous tag, and publishes a GitHub Release with checksums.

```bash
# Cut a release
git tag -a v1.2.3 -m "v1.2.3"
git push origin v1.2.3
```

- Configuration: `.goreleaser.yaml`
- Workflow: `.github/workflows/release.yml`
- CI/lint: `.github/workflows/ci.yml` (also runs `goreleaser check` on PRs)
- Full procedure and conventions: `RELEASING.md`
- Windows binaries are intentionally excluded — `internal/domain/services/asset_observer.go` uses Unix-only `syscall.Kill`/`Setpgid` for the `teal ui` hot-reload child process. Library use on Windows via `go get` is unaffected.
- `TEAL_VERSION` is a `var` (not `const`); it defaults to `"dev"` and is overridden at release-build time via ldflags.

### Testing Generated Projects
```bash
# Generate test project from scaffold
make test

# Clean generated test files  
make test_clean
```

### Running Teal Commands
```bash
# Initialize a new project
teal init

# Generate Go code from SQL models
teal gen [--project-path=<path>] [--config-file=<path>]

# Clean generated files
teal clean [--project-path=<path>] [--clean-main]

# Show version
teal version
```

## Architecture Overview

Teal is a Go-based ETL tool that generates Go code from SQL models, creating data pipelines with DAG-based execution. The architecture consists of:

### Core Components

1. **CLI Tool (`cmd/teal/`)**: Entry point for all Teal commands (init, gen, clean, version)

2. **Asset Generation Pipeline (`internal/application/`, `internal/domain/`)**: 
   - Parses SQL models from `assets/models/<stage>/*.sql`
   - Extracts profiles from YAML templates within SQL files
   - Generates Go code for each model with proper DAG dependencies
   - Creates test assets from `assets/tests/` directory

3. **DAG Execution Engine (`pkg/dags/`)**: 
   - `ChannelDag`: Implements concurrent execution using Go channels and goroutines
   - Each node represents an asset, edges are Go channels for data flow
   - Supports both with and without test execution modes

4. **Database Drivers (`pkg/drivers/`)**:
   - **DuckDB**: Supports extensions, dataframe operations, and custom configurations
   - **PostgreSQL**: Full SSL support, environment variable configuration
   - Factory pattern for driver instantiation
   - Cross-database references via `gota.DataFrame` when `is_data_framed: true`

5. **Processing Layer (`pkg/processing/`)**:
   - **SQL Assets**: Table, incremental, view, and custom materializations
   - **Raw Assets**: Custom Go functions implementing `ExecutorFunc` interface
   - **Testing**: SQL-based tests that pass when returning zero rows. Test queries are automatically wrapped with `SELECT COUNT(*) ... HAVING count > 0 LIMIT 1` during code generation - users write only the constraint-checking SQL

### Project Structure After Generation

```
project/
├── assets/models/          # SQL model files organized by stages
│   ├── staging/
│   ├── dds/
│   └── mart/
├── assets/tests/           # SQL test files
├── cmd/
│   ├── <project-name>/     # Production binary (Channel DAG only)
│   │   └── <project-name>.go
│   └── <project-name>-ui/  # Debug UI binary (Debug DAG + UI server)
│       └── <project-name>-ui.go
├── internal/assets/        # Generated Go code for each model
│   ├── configs.go         # DAG configuration
│   └── <stage>.<model>.go # Individual model implementations
├── internal/model_tests/   # Generated test implementations
├── config.yaml            # Database connections configuration
└── profile.yaml           # Project and model profiles
```

### File Generation Principles

The `teal gen` command generates two separate main entry points to ensure clean separation of concerns:

1. **Production Binary (`cmd/<project-name>/`)**:
   - Uses Channel DAG for efficient concurrent execution
   - No UI or debugging dependencies
   - Generates unique task names with timestamps (format: `<project_name>_<timestamp>`)
   - Optimized for production deployments
   - Supports custom task names via `--task-name` flag

2. **Debug UI Binary (`cmd/<project-name>-ui/`)**:
   - Uses Debug DAG for visualization and monitoring
   - Includes UI server with REST API endpoints
   - Provides execution tracking and task history
   - Designed for development and debugging
   - Runs on configurable port (default 8080)
   - **Important**: This binary only provides the Debug API server; the UI Dashboard is served by the teal CLI itself

This dual-generation approach ensures:
- Production binaries have no unnecessary dependencies
- Clean separation between production and debugging code
- Developers get powerful debugging tools without affecting production

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [go-teal/teal](https://github.com/go-teal/teal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-12 -->
