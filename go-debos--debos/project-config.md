---
trigger: always_on
description: **debos** is a tool for creating Debian-based OS images. It reads YAML recipe files and executes actions sequentially to build customized OS images. It uses fakemachine (a virtualization backend) to ensure reproducibility across different host environments.
---

# Debos Repository - Copilot Coding Instructions

## Project Overview

**debos** is a tool for creating Debian-based OS images. It reads YAML recipe files and executes actions sequentially to build customized OS images. It uses fakemachine (a virtualization backend) to ensure reproducibility across different host environments.

**Repository Stats:**
- Language: Go (requires Go 1.23+, confirmed working with Go 1.24.7)
- Size: ~14MB (excluding build artifacts)
- Type: Command-line tool
- Architecture: 30 Go source files organized in actions-based architecture

## System Dependencies

**CRITICAL: Must install system dependencies before building:**

```bash
sudo apt-get update && sudo apt-get install -y \
    libglib2.0-dev \
    libostree-dev \
    pkg-config
```

Building without these dependencies will fail with pkg-config errors about missing glib-2.0 and gobject-2.0.

## Build Instructions

### Clean Build Sequence (ALWAYS follow this order)

1. **Download dependencies:**
   ```bash
   go mod download
   ```

2. **Tidy modules (if go.mod changed):**
   ```bash
   go mod tidy
   ```

3. **Pre-build ostree package (REQUIRED before main build):**
   ```bash
   go build github.com/sjoerdsimons/ostree-go/pkg/otbuiltin
   ```
   This step is necessary due to CGO dependencies in the ostree-go package. Skip this and the main build may fail intermittently.

4. **Build debos binary:**
   ```bash
   go build ./cmd/debos
   ```
   Or with version info:
   ```bash
   DEBOS_VER=$(git describe --always --tags HEAD)
   go build -ldflags="-X main.Version=${DEBOS_VER}" ./cmd/debos
   ```

5. **Verify build:**
   ```bash
   ./debos --version
   ```

**Build time:** ~30-60 seconds on modern hardware with cached dependencies.

### Running Tests

**Unit tests (fast, ~1-2 seconds):**
```bash
go test -v ./...
```

All tests should pass. CI requires that no tests are skipped (`! grep -q SKIP test.out`).

**Integration tests:** Integration/recipe tests run in Docker containers with fakemachine. Before submitting changes, run relevant integration test recipes, especially those containing actions that were modified. When adding new features, ensure they are exercised as part of an integration test. More information on how to run Docker based tests can be found later in this file.

**Testing focus:**
- When adjusting actions, focus on integration tests to verify the action behavior end-to-end
- Unit tests should only be added for specific subroutines containing complex computations

### Linting

**ALWAYS run linting before committing:**

1. **Install golangci-lint v2.3.1:**
   ```bash
   curl -sSfL https://raw.githubusercontent.com/golangci/golangci-lint/master/install.sh | \
       sh -s -- -b $(go env GOPATH)/bin v2.3.1
   ```

2. **Run linter:**
   ```bash
   golangci-lint run
   ```
   Or if installed in GOPATH:
   ```bash
   $(go env GOPATH)/bin/golangci-lint run
   ```

Configuration is in `.golangci.yml`. Enabled linters: govet, errorlint, misspell, revive, staticcheck, whitespace, gofmt.

**Expected result:** `0 issues.`

## Project Structure

### Root Directory Files
```
.gitignore           - Ignore patterns for Go, Linux, Vim, VS Code
.golangci.yml        - Linter configuration
README.md            - User documentation with installation and usage
TODO                 - Future feature ideas (informational only)
go.mod, go.sum       - Go module dependencies
action.go            - Core Action interface and Context definitions
commands.go          - Command execution and chroot handling
archiver.go          - Archive handling (tar, gz, etc.)
filesystem.go        - Filesystem operations
net.go               - Network operations (download support)
os.go                - OS-level utilities
debug.go             - Debug shell support
```

### Key Directories

**`cmd/debos/`** - Main entry point
- `debos.go` - CLI argument parsing, fakemachine setup, recipe execution

**`actions/`** - Action implementations (the core functionality)
- Each action type has its own file: `apt_action.go`, `debootstrap_action.go`, `download_action.go`, etc.
- `actions_doc.go` - Documentation for all action types
- Available actions: apt, debootstrap, mmdebstrap, download, filesystem-deploy, image-partition, ostree-commit, ostree-deploy, overlay, pack, pacman, pacstrap, raw, recipe, run, unpack

**`tests/`** - Integration test recipes
- Each subdirectory contains a `test.yaml` recipe file
- Tests: recipes, templating, partitioning, msdos, debian, arch, apertis, raw, exit_test
- These run in CI using Docker + fakemachine

**`doc/`** - Documentation and examples
- `doc/examples/` - Example recipe files (e.g., ospack-debian)
- `doc/man/` - Man page generation

**`docker/`** - Docker container build files
- `Dockerfile` - Multi-stage build for debos container
- `unit-tests.test.yml`, `exitcode-test.yml` - Docker Compose test configs

**`.github/workflows/`** - CI/CD pipeline
- `ci.yaml` - Comprehensive CI with lint, test, build, recipe-tests, example-recipes

### Architecture Overview

debos uses an **action-based architecture**:
1. Parse YAML recipe file

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [go-debos/debos](https://github.com/go-debos/debos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
