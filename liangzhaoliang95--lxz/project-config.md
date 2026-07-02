---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

LXZ is a Go-based Terminal User Interface (TUI) application for DevOps resource management. It provides graphical interfaces for managing databases (MySQL), Redis, Docker containers, file systems, Kubernetes (K9s), and SSH connections—all within the terminal.

Built with the tview library (forked version), it follows a clean layered architecture with separation between UI components, business logic, drivers, and configuration.

## Build and Development Commands

### Building

```bash
# Build for current platform
make build
# OR
./scripts/build.sh build

# Build for specific platforms
make build-linux      # Linux AMD64
make build-windows    # Windows AMD64
make build-darwin     # macOS AMD64

# Cross-compile all platforms (6 targets: Linux/macOS/Windows on AMD64/ARM64)
make cross-build
# OR
./scripts/build.sh cross
```

The build script (`scripts/build.sh`) automatically injects version information via ldflags:
- Git tag version
- Commit hash
- Build date/time
- Go version
- Platform and architecture

Output binaries go to `dist/` directory for cross-builds.

### Testing

```bash
# Run all tests
make test
# OR
go test ./...

# Run tests with coverage
make test-coverage   # Generates coverage.html

# Run benchmarks
make bench
```

Note: The codebase has minimal test coverage currently (only 2 test files found).

### Development

```bash
# Run in development mode
make dev
# OR
go run main.go

# Run with specific flags
go run main.go --logLevel debug
go run main.go --refresh 5      # 5 second refresh rate
go run main.go --headless       # No header UI
go run main.go --splashless     # Skip splash screen

# Format code
make fmt

# Static analysis
make vet

# Lint (requires golangci-lint)
make lint

# All quality checks (fmt + vet + lint + security)
make quality
```

### Version Management

```bash
# Show version info
make version
# OR
go run main.go version

# Check for updates
make check-update
# OR
go run main.go version --check-update
```

### Releasing

```bash
# Create new release (interactive script)
make release
# OR
./release.sh
```

The release process:
1. Checks git status
2. Prompts for version bump type (major/minor/patch)
3. Creates and pushes git tag (e.g., `v1.0.0`)
4. GitHub Actions automatically builds 6 platform binaries and creates GitHub Release

See RELEASE_GUIDE.md and VERSION_MANAGEMENT.md for details.

### Other Commands

```bash
# Clean build artifacts
make clean

# Download dependencies
make deps

# Update dependencies
make deps-update

# Install to system
make install
```

## Architecture Overview

### Layer Structure

```
cmd/                    Entry point (Cobra CLI)
    ↓
internal/view/          Resource browsers (business logic)
    ↓
internal/ui/            tview component wrappers
    ↓
internal/drivers/       External service adapters
internal/config/        Configuration management
```

### Key Architectural Patterns

**1. Component Lifecycle**
All views implement the `Component` interface (internal/model/types.go):
- `Init(ctx)` - Initialization with dependency injection
- `Start()` - Called when component becomes active
- `Stop()` - Called when component is removed
- `Hints()` - Returns keyboard shortcuts for menu

Lifecycle managed by `PageStack` which pushes/pops components onto a navigation stack.

**2. View Layer (internal/view/)**

The `view.App` struct is the main controller:
- Routes F1-F6 keys to different resource browsers
- Manages the `PageStack` for navigation
- Injects dependencies into components via `inject()`

All browsers inherit from `BaseFlex`:
- `DatabaseBrowser` - MySQL table browsing, query execution
- `RedisBrowser` - Redis key-value operations
- `DockerBrowser` - Container management, logs, shell access
- `FileBrowser` - Directory tree navigation with preview
- `K9SBrowser` - Kubernetes monitoring
- `SshConnect` - SSH connection manager

Each browser can have sub-views that get pushed onto the stack (e.g., DatabaseDbListView → DatabaseTableView → DatabaseQueryView).

**3. UI Layer (internal/ui/)**

`ui.App` wraps `tview.Application` and manages:
- Global components: Logo, Menu (breadcrumbs), SubMenu (keyboard hints), Status, Flash (notifications)
- Keyboard routing via `KeyActions` map
- Thread-safe UI updates via `QueueUpdateDraw()`

Modal dialogs in `ui/dialog/`:
- Connection forms (database, Redis)
- File operations (create, delete, rename)
- Confirmation dialogs
- Loading screens

**4. Driver Layer (internal/drivers/)**

Abstract interfaces for external services:
- `IDatabaseConn` - Database operations (currently MySQL via GORM)
- `RedisClient` - Redis operations (go-redis wrapper)
- Docker driver (moby/moby SDK)

Drivers use:
- Factory pattern for initialization
- Connection pooling via `sync.Map`
- Lazy initialization with `GetConnectOrInit()`

**5. Configuration (internal/config/)**

YAML-based config stored in `~/.config/lxz/`:
- `Config` → `LXZ` → resource-specific configs (DatabaseConfig, RedisConfig)
- JSON schema validation (config/json/schemas/)
- Load/Save/Merge methods
- `Styles` system for theming with listener pattern

### Data Flow Example


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [liangzhaoliang95/lxz](https://github.com/liangzhaoliang95/lxz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->
