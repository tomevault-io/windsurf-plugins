---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**aigogo** is a lightweight agent manager that uses Docker registries as a transport mechanism. It enables sharing and distributing reusable AI agents across projects. Key distinction: this manages AGENTS, not dependencies.

## Build & Test Commands

```bash
# Build
make build              # Build for current platform (output: bin/aigg)
make build-all          # Build for Linux (AMD64/ARM64), macOS (Intel/ARM), Windows

# Test
go test -v ./...        # Run all tests
go test -v ./pkg/manifest  # Run tests for specific package
go test -coverprofile=coverage.out ./...  # With coverage

# Lint
golangci-lint run --timeout=5m   # Same linter config as CI

# Other
go fmt ./...            # Format code
go vet ./...            # Check for issues
go mod tidy             # Manage dependencies
make install            # Install to /usr/local/bin (requires sudo)
make install-user       # Install to ~/bin (no sudo)
```

## Post-Edit Linting (IMPORTANT)

After modifying any `.go` file, **always** run the linter before considering the task complete:

```bash
# Install if not available
go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest

golangci-lint run --timeout=5m
```

This matches the CI lint job (`.github/workflows/test.yml` — `golangci/golangci-lint-action@v9`). Fix any errors before moving on. Common lint issues:
- `errcheck`: unchecked error return values (e.g. `.Close()`, `os.RemoveAll`) — use `_ = x.Close()` or check explicitly
- `staticcheck ST1005`: error strings should not be capitalized
- `govet`: suspicious constructs (shadowed variables, printf format mismatches)

## Architecture

### Entry Point
- `main.go` - Entry point with version injection via ldflags (`-X main.Version`)

### CLI Commands (`cmd/`)
22 commands built without external CLI framework. Key files:
- `root.go` - Command routing and argument parsing
- `add.go` - Add packages to lock file, or files/dependencies to manifest
- `install.go` - Install packages from aigogo.lock (creates symlinks)
- `uninstall.go` - Remove installed packages, .pth file, register.js, and .aigogo/ directory
- `build.go` - Local build with auto-versioning
- `push.go` - Push to registry (requires `--from` flag for local builds)
- `exec.go` - Execute agent scripts (npx-like workflow with dependency isolation)
- `exec_unix.go` - Unix implementation of process replacement via syscall.Exec
- `exec_windows.go` - Windows stub returning unsupported error
- `clean.go` - Disk usage summary and cleanup of envs/cache/store

### Core Packages (`pkg/`)

**store/** - Content-Addressable Storage (CAS)
- `store.go` - Immutable package storage by SHA256 hash (~/.aigogo/store/)
- Packages stored at `~/.aigogo/store/sha256/<prefix>/<hash>/`
- Files made read-only after storage

**lockfile/** - Lock file management
- `lockfile.go` - Load/Save/Find aigogo.lock files
- Tracks package versions, integrity hashes, and sources
- `NormalizeName()` converts package names for Python (`my-utils` → `my_utils`)

**imports/** - Language-specific import setup
- `setup.go` - Creates `.aigogo/imports/` directory structure
- `pth.go` - Manages `.pth` file in Python site-packages for automatic path configuration
- `register.go` - Generates `.aigogo/register.js` for Node.js module resolution, resolves JS entry points
- Python namespace: `.aigogo/imports/aigogo/<package>/` with `__init__.py` (directory symlink to store)
- JavaScript scope: `.aigogo/imports/@aigogo/<package>/` (real dir with file symlinks + generated `package.json`)
- Auto-updates `.gitignore` to exclude `.aigogo/`

**manifest/** - Manifest (aigogo.json) handling
- `types.go` - Data structures: Manifest, Language, Dependencies, FileSpec
- `loader.go` - Load/Save/Validate manifest JSON
- `finder.go` - Find aigogo.json by walking up directory tree (like git)
- `discovery.go` - Auto-discover files by language patterns
- `ignore.go` - `.aigogoignore` file support (gitignore-compatible pattern matching)

**docker/** - Registry and local cache operations
- `local_builder.go` - Build packages to local cache (~/.aigogo/cache)
- `builder.go` - Create Docker image tar structures
- `extractor.go` - Extract files from cached packages
- `puller.go` / `pusher.go` - Registry pull/push operations
- `utils.go` - Image ref parsing, cache directory utilities, hash functions

**depgen/** - Dependency file generation
- `generator.go` - Generate requirements.txt, package.json, go.mod, Cargo.toml
- `scanner.go` - Scan source files for imports
- `validator.go` - Validate declared vs actual dependencies

**auth/** - Registry authentication
- Stores credentials in `~/.aigogo/auth.json` (mode 0600)
- Docker Hub OAuth2 token exchange support

### Key Design Patterns

1. **Content-Addressable Storage**: Packages stored by SHA256 hash for immutability
2. **Lock File Workflow**: `aigogo.lock` pins exact versions and hashes for reproducibility
3. **Namespace Imports**: Python uses `from aigogo.package_name`, JS uses `@aigogo/package-name`
4. **Local-First Workflow**: Build locally first, then push with explicit `--from` flag

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aupeachmo/aigogo](https://github.com/aupeachmo/aigogo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
