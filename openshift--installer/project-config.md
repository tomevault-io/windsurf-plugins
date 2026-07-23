---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

@AGENTS.md

## Quick Reference Documentation

- **Getting Started**: See [README.md](README.md) for quick start guide
- **Contributing**: See [CONTRIBUTING.md](CONTRIBUTING.md) for contribution workflow, linting, testing, and commit message format
- **Build Dependencies**: See [docs/dev/dependencies.md](docs/dev/dependencies.md) for required system packages and Go version

## Build and Development Commands

### Building the Installer

```sh
# Build the openshift-install binary
hack/build.sh

# Skip Terraform build (faster)
SKIP_TERRAFORM=y hack/build.sh

# Development build (with debugging symbols)
MODE=dev hack/build.sh
```

The binary is output to `bin/openshift-install`.

### Testing

```sh
# Run unit tests
hack/go-test.sh

# Run specific tests with additional arguments
hack/go-test.sh -v -run TestSpecificTest

# Run integration tests
hack/go-integration-test.sh

# Run node joiner integration tests
hack/go-integration-test-nodejoiner.sh
```

### Linting and Formatting

See [CONTRIBUTING.md](CONTRIBUTING.md#contribution-flow) for the complete list of linters to run before submitting a PR. Quick reference:

```sh
# Format Go code and organize imports
hack/go-fmt.sh .

# Run Go linter
hack/go-lint.sh $(go list -f '{{ .ImportPath }}' ./...)

# Run Go vet
hack/go-vet.sh ./...

# Check shell scripts
hack/shellcheck.sh

# Format Terraform files
hack/tf-fmt.sh -list -check

# Lint Terraform
hack/tf-lint.sh

# Lint YAML files
hack/yaml-lint.sh
```

### Generating Code

```sh
# Regenerate mocks for unit tests
hack/go-genmock.sh

# Update install config CRD (after bumping github.com/openshift/api)
go generate ./pkg/types/installconfig.go
```

## Architecture

### Asset-Based Architecture

The installer uses a dependency-graph architecture where everything is an "Asset". See [docs/design/assetgeneration.md](docs/design/assetgeneration.md) for complete details.

Key points:
- **Asset**: Interface with `Dependencies()`, `Generate()`, and `Name()` methods
- **WritableAsset**: Assets that can be written to disk and loaded
- Main assets in `pkg/asset/`: install-config, manifests, ignition-configs, cluster

### Cluster API Integration

The installer uses Cluster API (CAPI) controllers running in a local control plane. See [docs/dev/cluster-api.md](docs/dev/cluster-api.md) for complete details.

Key points:
- Local `kube-apiserver` and `etcd` run via envtest
- Platform-specific infrastructure providers in `cluster-api/providers/`
- Build CAPI binaries with `hack/build-cluster-api.sh` (called automatically by `hack/build.sh`)

### Platform Types

Platform-specific logic lives in `pkg/types/<platform>/`:
- Platform type definitions
- Validation logic in `validation/`
- Default values in `defaults/`

Supported platforms: AWS, Azure, GCP, vSphere, OpenStack, IBM Cloud, Power VS, Nutanix, bare metal.

### Bootstrap Process

The installer creates a temporary bootstrap machine that:
1. Hosts resources for control plane machines to boot
2. Forms initial etcd cluster with control plane nodes
3. Starts temporary Kubernetes control plane
4. Schedules production control plane on control plane machines
5. Injects OpenShift components
6. Shuts down once cluster is self-hosting

## Dependency Management

See [docs/dev/dependencies.md](docs/dev/dependencies.md) for complete dependency management instructions including:
- Adding/updating Go dependencies with `go get`, `go mod tidy`, `go mod vendor`
- Updating CAPI provider dependencies (detailed multi-step process)
- Special case: updating after bumping `github.com/openshift/api`

**Important**: Always commit vendored code in a separate commit from functional changes.

## Commit Message Format

See [CONTRIBUTING.md](CONTRIBUTING.md#commit-message-format) for the complete format specification.

Quick reference:
```
<subsystem>: <what changed>

<why this change was made>

Fixes #<issue-number>
```

Common subsystems:
- `baremetal`, `vsphere`, `aws`, `azure`, `gcp`, etc. - for platform-specific changes
- `agent`, `ibi` (image-based installer) - for installation method changes
- `terraform`, `cluster-api` - for infrastructure provider changes
- `docs` - for documentation changes
- `unit tests`, `integration tests` - for test-only changes (makes it clear the change doesn't affect the actual installer)

## JIRA Integration

When work is associated with a JIRA issue:
- **PR titles** must be prefixed with the JIRA issue number: `<JIRA-KEY>: <subsystem>: <description>` (e.g., `CORS-3997: azure: update default instance types from v3 to v5`)
- **Commit messages** should reference the JIRA issue in the body when relevant

## Testing Approach

The installer has different types of tests with varying external requirements:

### Pure Unit Tests

Most tests in `pkg/` are pure unit tests that test Go code logic without external dependencies. These can be run with:

```sh
go test ./pkg/...
```

These tests should pass in any environment with Go installed.

### Integration Tests with External Requirements

Some test files have external dependencies and will fail without specific tools installed:

#### Node Joiner Integration Tests

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openshift/installer](https://github.com/openshift/installer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
