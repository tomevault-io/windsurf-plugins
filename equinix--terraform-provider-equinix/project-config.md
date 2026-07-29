---
trigger: always_on
description: This is the official Terraform provider for Equinix Platform, enabling lifecycle management of Equinix resources through Terraform. The codebase is approximately 64,000 lines of Go code across 789 files, implementing a Terraform plugin that supports both Equinix Fabric and Network Edge services.
---

# Terraform Provider for Equinix Platform - Development Guide

## Repository Overview

This is the official Terraform provider for Equinix Platform, enabling lifecycle management of Equinix resources through Terraform. The codebase is approximately 64,000 lines of Go code across 789 files, implementing a Terraform plugin that supports both Equinix Fabric and Network Edge services.

**Key Technologies:**
- Language: Go 1.23.0+
- Framework: Terraform Plugin Framework v1.15.0 and Plugin SDK v2.37.0 (using mux for compatibility)
- Architecture: Mixed SDKv2 and Framework provider using terraform-plugin-mux
- Build Tool: GNU Make + Go toolchain
- Testing: Terraform Plugin Testing framework with acceptance tests

## Project Structure

### Core Directories
- `main.go` - Provider entry point, muxes SDKv2 and Framework providers
- `equinix/` - Legacy SDKv2 provider code (avoid adding new files here per validation workflow)
- `internal/` - Main implementation code
  - `internal/provider/` - Framework-based provider setup
  - `internal/resources/` - Resource implementations (fabric subdirectories)
  - `internal/config/` - Provider configuration and client setup
  - `internal/fabric/`, `internal/network/` - Service-specific helpers
  - `internal/acceptance/` - Acceptance test helpers
  - `internal/sweep/` - Resource cleanup for tests
- `docs/` - Generated documentation (do not edit manually)
- `templates/` - Documentation templates for generation
- `examples/` - Example Terraform configurations for docs
- `scripts/` - Build and validation scripts

### Configuration Files
- `GNUmakefile` - Build automation (primary interface)
- `.golangci.yaml` - Linter configuration (golangci-lint v2.3.0)
- `go.mod` - Go dependencies
- `.github/workflows/` - CI/CD workflows

## Build, Test, and Validation

### Prerequisites
- Go 1.23.0+ (required version from go.mod)
- Terraform 1.0.11+ (for testing)
- GNU Make
- Git

### Essential Commands (Run in Order)

#### 1. Initial Setup
```bash
# Download dependencies (always run first after clone or when go.mod changes)
go mod download
```

#### 2. Build
```bash
# Clean and build the provider binary (takes ~30 seconds)
make clean build

# The binary is created as: ./terraform-provider-equinix
```

#### 3. Unit Tests
```bash
# Run unit tests (takes ~2-3 minutes, skips acceptance tests)
make test

# Run specific test:
# go test -v ./path/to/package -run TestName
```

#### 4. Code Formatting
```bash
# Check formatting (always run before committing)
make fmtcheck

# Auto-format code if needed
make fmt
```

#### 5. Linting
```bash
# Run linter (takes ~3-5 minutes)
# Only checks files changed from origin/main
make lint

# The linter uses golangci-lint v2.3.0 specified in GNUmakefile
```

#### 6. Documentation Generation
```bash
# Generate docs from templates (requires network access)
make docs

# Verify docs are up to date
make docs-check
```

**NOTE:** `make docs-check` runs `go generate ./...` (via GNUmakefile), which invokes `tfplugindocs` and may require network access for Go module downloads and for `tfplugindocs` to contact HashiCorp services (e.g., checkpoint). It may fail in offline or restricted environments for these reasons; it does not download a Terraform CLI binary.

### Complete Pre-Commit Checklist
```bash
# Run these commands in order before committing changes:
make fmt          # Auto-format code
make fmtcheck     # Verify formatting
make build        # Ensure code compiles
make test         # Run unit tests
make lint         # Check code quality
make docs         # Update documentation (if resources/datasources changed)
```

### Acceptance Tests
Acceptance tests create real infrastructure and require credentials:
```bash
export EQUINIX_API_ENDPOINT=https://api.equinix.com
export EQUINIX_API_CLIENTID=<your-client-id>
export EQUINIX_API_CLIENTSECRET=<your-client-secret>
make testacc  # Takes 180+ minutes, runs all acceptance tests

# Run specific acceptance test:
TF_ACC=1 go test -v -timeout=20m ./... -run=TestAccFabricCreateCloudRouter2PortConnection_PFCR
```

**WARNING:** Acceptance tests are expensive and create real resources. Only run locally if necessary.

## GitHub Actions CI/CD

### Pull Request Validation Workflows

All PRs trigger these checks automatically:

1. **validate_pr.yml** - PR Title & File Location Checks
   - Validates PR title follows Conventional Commits (feat:, fix:, docs:, etc.)
   - Blocks new files in `equinix/` package (use `internal/` instead)
   
2. **test.yml** - Unit Tests & Docs Validation
   - Runs `go build` and `go test` with coverage
   - Runs `make docs-check` to ensure docs are generated
   
3. **golangci-lint.yml** - Code Quality
   - Runs golangci-lint via `golangci/golangci-lint-action` with the `--whole-files` flag and `only-new-issues: true`
   - Reports only new issues introduced by the PR (CI does not pass `--new-from-rev=origin/main`; that flag is used only by local `make lint`)
   
4. **fabric_acctest.yml** - Acceptance Tests
   - Run only when specific paths change (fabric/**)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [equinix/terraform-provider-equinix](https://github.com/equinix/terraform-provider-equinix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
