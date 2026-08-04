---
trigger: always_on
description: This document provides guidance for AI agents working with the Operator Registry project.
---

# AGENTS.md

This document provides guidance for AI agents working with the Operator Registry project.

## Project Overview

The Operator Registry is a Kubernetes/OpenShift component that provides operator catalog data to the Operator Lifecycle Manager (OLM). It manages operator bundles, indexes, and catalogs that enable operators to be discovered and installed in Kubernetes clusters.

## Key Components

### Binaries
- **`opm`**: Main CLI tool for generating and updating registry databases and index images
- **`initializer`**: **(Deprecated)** Converts operator manifests to SQLite database format
- **`registry-server`**: **(Deprecated)** Exposes gRPC interface to SQLite databases
- **`configmap-server`**: Parses ConfigMaps into SQLite databases and exposes gRPC interface

### Libraries
- **`pkg/client`**: High-level client interface for gRPC API
- **`pkg/api`**: Low-level client libraries for gRPC interface
- **`pkg/registry`**: Core registry types (Packages, Channels, Bundles)
- **`pkg/sqlite`**: **(Deprecated)** SQLite database interfaces for manifests
- **`pkg/lib`**: External interfaces and standards for operator bundles
- **`pkg/containertools`**: Container tooling integration

### Alpha Features
- **`alpha/declcfg`**: Declarative configuration format
- **`alpha/template`**: Template system for generating catalogs
- **`alpha/action`**: Action framework for registry operations

## Development Guidelines

### Code Structure
- **Go 1.24.4**: Minimum Go version required
- **Cobra CLI**: Command-line interface framework
- **gRPC**: Primary API communication protocol
- **SQLite**: Database backend for registry data
- **OCI Images**: Container image format for bundles

### Testing
- Unit tests: `go test ./...`
- Integration tests: Located in `test/e2e/`
- Linting: `make lint` (uses golangci-lint)
- Coverage: `make coverage`

### Build System
- **Makefile**: Primary build configuration
- **GoReleaser**: Release automation
- **Docker**: Container image builds

## Common Tasks for AI Agents

### 1. Adding New Template Types
When adding new template types to `alpha/template/`:

```go
// 1. Implement the Template interface
type MyTemplate struct {
    renderBundle template.BundleRenderer
}

func (t *MyTemplate) RenderBundle(ctx context.Context, image string) (*declcfg.DeclarativeConfig, error) {
    return t.renderBundle(ctx, image)
}

func (t *MyTemplate) Render(ctx context.Context, reader io.Reader) (*declcfg.DeclarativeConfig, error) {
    // Implementation
}

func (t *MyTemplate) Schema() string {
    return "olm.template.mytype"
}

// 2. Implement the TemplateFactory interface
type Factory struct{}

func (f *Factory) CreateTemplate(renderBundle template.BundleRenderer) template.Template {
    return &MyTemplate{renderBundle: renderBundle}
}

func (f *Factory) Schema() string {
    return "olm.template.mytype"
}

// 3. Register in cmd/opm/alpha/template/render.go
registry.Register(&mytype.Factory{})
```

### 2. Working with Bundle Images
```bash
# Build a bundle image
podman build -t quay.io/my-namespace/my-bundle:latest -f bundle.Dockerfile .

# IMPORTANT: Bundle images must be published to a registry before they can be consumed
podman push quay.io/my-namespace/my-bundle:latest

# Add bundle to index (deprecated - use file-based catalogs instead)
opm index add --bundles quay.io/my-namespace/my-bundle:latest --from-index quay.io/my-namespace/my-index:latest --tag quay.io/my-namespace/my-index:latest

# Generate index image (deprecated - use file-based catalogs instead)
opm index add --bundles quay.io/my-namespace/my-bundle:latest --tag quay.io/my-namespace/my-index:latest
```

**⚠️ Critical Requirements:**
- **Bundle images must be published** to an image registry before they can be referenced in catalogs
- **FBC content requires published images** - `opm render` and templates can only reference bundle images that exist in registries
- **Catalog images must be built and published** before they can be consumed by OLM

### 3. Working with Declarative Config
```go
// Load declarative config
cfg, err := declcfg.LoadFS(os.DirFS("path/to/catalog"))

// Convert to model
model, err := declcfg.ConvertToModel(cfg)

// Write declarative config
err = declcfg.Write(cfg, "output.yaml")
```

### 4. Database Operations **(Deprecated)**
```go
// Create SQLite database
db, err := sqlite.Open("registry.db")

// Add bundle to database
err = db.AddBundle(bundle)

// Query packages
packages, err := db.ListPackages()
```

### 5. Serving Catalog Content with `opm serve`
The `opm serve` command exposes operator catalog data via a gRPC interface for consumption by OLM.

```bash
# Serve declarative configs with default settings
opm serve ./catalog-directory

# Serve with custom port and caching
opm serve ./catalog-directory --port 8080 --cache-dir /tmp/cache

# Serve with cache integrity enforcement
opm serve ./catalog-directory --cache-dir /tmp/cache --cache-enforce-integrity

# Cache-only mode (build cache without serving)
opm serve ./catalog-directory --cache-dir /tmp/cache --cache-only
```

**⚠️ FBC Requirements:**
- **Bundle images must be published** to image registries before they can be referenced in FBC

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [operator-framework/operator-registry](https://github.com/operator-framework/operator-registry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
