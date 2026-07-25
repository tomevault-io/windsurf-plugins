---
trigger: always_on
description: **helm-gcs** is a Helm plugin that enables managing private Helm chart repositories using Google Cloud Storage (GCS) as the backend storage. It allows developers to store, push, pull, and manage Helm charts in GCS buckets instead of traditional HTTP/S registries.
---

# CLAUDE.md - Project Guide for helm-gcs

## Project Overview

**helm-gcs** is a Helm plugin that enables managing private Helm chart repositories using Google Cloud Storage (GCS) as the backend storage. It allows developers to store, push, pull, and manage Helm charts in GCS buckets instead of traditional HTTP/S registries.

**Repository**: https://github.com/hayorov/helm-gcs
**Current Version**: 0.7.0
**Language**: Go 1.25+ (toolchain 1.25.6)
**License**: MIT

### Key Features
- Initialize Helm repositories in GCS buckets
- Push/pull Helm charts to/from GCS
- Support for both private and public GCS buckets
- Multiple authentication methods (ADC, service account, OAuth token)
- Concurrent update handling with optimistic locking
- Cross-platform support (Linux, macOS, Windows on amd64/arm64)
- **Helm 4 native support** with separate CLI and Getter plugins

---

## Architecture Overview

### Layered Architecture
```
┌─────────────────────────────────────┐
│   CLI Layer (Cobra Commands)       │  cmd/helm-gcs/cmd/*.go
├─────────────────────────────────────┤
│   Domain Layer (Repo Management)   │  pkg/repo/repo.go
├─────────────────────────────────────┤
│   Infrastructure (GCS Client)      │  pkg/gcs/gcs.go
├─────────────────────────────────────┤
│   External Services (GCS API)      │  cloud.google.com/go/storage
└─────────────────────────────────────┘
```

### Directory Structure
```
helm-gcs/
├── cmd/
│   ├── helm-gcs/              # CLI plugin binary
│   │   ├── main.go           # Entry point for CLI commands
│   │   └── cmd/              # Cobra command definitions
│   │       ├── root.go       # Root command with GCS client init
│   │       ├── init.go       # Initialize repo command
│   │       ├── push.go       # Push chart command
│   │       ├── pull.go       # Pull chart command
│   │       ├── rm.go         # Remove chart command
│   │       └── version.go    # Version command
│   └── helm-gcs-getter/       # Getter plugin binary (Helm 4)
│       └── main.go           # Downloads from GCS to stdout
├── pkg/                       # Core business logic (shared)
│   ├── gcs/                   # GCS client wrapper
│   └── repo/                  # Repository operations
├── plugins/                   # Helm 4 plugin packages
│   ├── gcs/                   # CLI plugin (cli/v1)
│   │   ├── plugin.yaml
│   │   └── scripts/install.sh
│   └── gcs-getter/            # Getter plugin (getter/v1)
│       ├── plugin.yaml
│       └── scripts/install.sh
├── scripts/                   # Legacy scripts (Helm 3 compat)
│   └── install.sh
├── plugin.yaml                # Legacy plugin manifest (Helm 3)
└── .github/workflows/         # CI/CD pipelines
```

---

## Key Files Reference

### Entry Points
- **cmd/helm-gcs/main.go** - Simple main() that delegates to Cobra
- **cmd/helm-gcs/cmd/root.go:15** - CLI initialization and GCS client setup

### Core Business Logic
- **pkg/repo/repo.go** (408 lines) - Main repository management logic
  - `Repo` struct - Manages repository operations
  - `Create()` - Initialize new repository
  - `PushChart()` - Upload chart with index update
  - `RemoveChart()` - Delete chart versions
  - `uploadIndexFile()` - Update Helm index with optimistic locking
  - `indexFile()` - Fetch and cache repository index

### Infrastructure
- **pkg/gcs/gcs.go** (53 lines) - GCS client abstraction
  - `NewClient()` - Creates authenticated GCS client
  - `Object()` - Returns object handle for gs:// paths

### Configuration
- **plugin.yaml** - Helm plugin manifest (name, version, command, protocols)
- **go.mod** - Go module dependencies
- **.goreleaser.yml** - Cross-platform build configuration

---

## Important Patterns & Conventions

### Authentication (Priority Order)
1. **OAuth Token**: If `GOOGLE_OAUTH_ACCESS_TOKEN` env var is set
2. **Service Account**: If `--service-account` flag provided with JSON key file path
3. **ADC (Application Default Credentials)**: Fallback default method

See: cmd/helm-gcs/cmd/root.go:15-60

### Concurrency Control
Uses **optimistic locking** via GCS object generation numbers to prevent concurrent index corruption:
```go
o = o.If(storage.Conditions{GenerationMatch: r.indexFileGeneration})
```
Returns `ErrIndexOutOfDate` (HTTP 412) if index was modified concurrently. Use `--retry` flag for automatic retries.

See: pkg/repo/repo.go:320-340

### Error Handling
- Uses `pkg/errors` for error wrapping with context
- Custom error: `ErrIndexOutOfDate` for concurrent update detection
- Panics on init failure for immediate feedback on auth/config issues

### Logging
- Controlled by `--debug` flag or `HELM_GCS_DEBUG=true` env var
- Uses logrus structured logging (INFO/DEBUG levels)
- Outputs to stderr

---

## Technology Stack

| Component | Package | Version | Purpose |
|-----------|---------|---------|---------|
| CLI Framework | github.com/spf13/cobra | v1.10.2 | Command structure |
| GCS Client | cloud.google.com/go/storage | v1.39.1 | GCS operations |
| Helm Integration | helm.sh/helm/v4 | v4.0.0 | Chart/index handling |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hayorov/helm-gcs](https://github.com/hayorov/helm-gcs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
