---
trigger: always_on
description: This project automates the lifecycle of KubeVirt [containerdisks](https://kubevirt.io/user-guide/storage/disks_and_volumes/#containerdisk): downloading official qcow2 VM images from upstream OS distributions, packaging them as OCI container images, and publishing them to the `quay.io/containerdisks` registry. These images are pulled by KubeVirt clusters to run virtual machines.
---

# AGENTS.md

## Project Overview

This project automates the lifecycle of KubeVirt [containerdisks](https://kubevirt.io/user-guide/storage/disks_and_volumes/#containerdisk): downloading official qcow2 VM images from upstream OS distributions, packaging them as OCI container images, and publishing them to the `quay.io/containerdisks` registry. These images are pulled by KubeVirt clusters to run virtual machines.

The core CLI tool is called **medius** (`cmd/medius`). It detects new upstream OS releases, compares them against what is already published in Quay.io, and builds + pushes updated containerdisks when a mismatch is found.

## Repository Structure

- `cmd/medius/` - Main CLI entry point built with Cobra. Subcommands: `images push`, `images verify`, `images promote`, `docs`.
- `artifacts/` - Per-distribution implementations. Each subdirectory (e.g. `fedora/`, `ubuntu/`, `centos-stream/`, `debian/`, `opensuse/`) implements the `api.Artifact` and optionally `api.ArtifactsGatherer` interfaces.
- `pkg/api/` - Core interfaces (`Artifact`, `ArtifactsGatherer`) and shared types (`ArtifactDetails`, `Metadata`, `ArtifactResult`).
- `pkg/build/` - Builds OCI container images from qcow2 disk files using `go-containerregistry`. Supports multi-arch image indexes.
- `pkg/repository/` - Interacts with container registries (push images, read metadata/labels).
- `pkg/quay/` - Quay.io-specific API client.
- `pkg/http/` - HTTP client utilities for downloading artifacts with checksum verification.
- `pkg/hashsum/` - Checksum verification utilities.
- `pkg/tests/` - End-to-end test helpers (SSH into VMs, check guest OS info via KubeVirt).
- `pkg/docs/` - Generates VM example docs and cloud-init/ignition user data templates.
- `cmd/medius/common/registry.go` - The artifact registry where all distributions and versions are statically registered. Fedora also uses dynamic gathering via `ArtifactsGatherer`.
- `testutil/` - Mock HTTP getter for unit tests.
- `hack/` - Helper scripts for local KubeVirt cluster (kubevirtci).
- `pipeline.sh` / `pipeline-periodic.sh` - CI pipeline scripts that build, push, verify, and promote containerdisks.

## Language and Build

- **Language**: Go (module: `kubevirt.io/containerdisks`)
- **Build**: `make medius` produces `bin/medius`
- **Test**: `make test` runs linting (`golangci-lint`) and Ginkgo tests
- **Lint**: `make lint` runs golangci-lint v2 with an extensive linter set
- **Format**: `make fmt` runs `gofumpt` and `go mod tidy`
- **Vendor**: Dependencies are vendored (`vendor/`). Run `make vendor` after modifying dependencies

## Architecture

Multi-architecture support: amd64, arm64, s390x (varies per distro).

### Adding a New Distribution

1. Create a new package under `artifacts/` implementing the `api.Artifact` interface.
2. Register it in `cmd/medius/common/registry.go` in the `staticRegistry` slice.
3. For distributions with dynamic version discovery, implement `api.ArtifactsGatherer` (see `artifacts/fedora/` for an example).

### Key Interfaces

- **`api.Artifact`**: `Inspect()` returns download URL and checksum details. `Metadata()` returns image name, version, and architecture. `VM()` creates a KubeVirt VM spec. `Tests()` returns end-to-end test functions.
- **`api.ArtifactsGatherer`**: `Gather()` dynamically discovers available versions from upstream release APIs.

### Execution Environment

This project is designed to run in CI, not on individual developer machines. Two pipeline scripts orchestrate the full workflow on a temporary KubeVirt CI cluster:

- **`pipeline-periodic.sh`** — Runs as a periodic CI job. Spins up a KubeVirt CI cluster, pushes all containerdisks, verifies them, promotes verified images to production, and tears down the cluster. Supports `FORCE_REBUILD=true` to rebuild all images regardless of checksum state and `PROMOTE_DRY_RUN` to control whether promotion writes to the production registry.
- **`pipeline.sh`** — Runs as a presubmit CI job. Executes the same push/verify/promote flow but scoped to a single distribution via `FOCUS` (defaults to `centos-stream:9`). Promotion is always a dry run.

### Pipeline Flow

1. **Push** (`medius images push`): For each registered artifact, inspect upstream for the latest release, compare checksum with what's published in the registry, download and build a containerdisk if outdated, and push to the target registry.
2. **Verify** (`medius images verify`): Boot each containerdisk in a KubeVirt cluster and run tests (SSH, guest OS info).
3. **Promote** (`medius images promote`): Move verified images from staging to production tags.

## Testing

- Unit tests use Ginkgo/Gomega and live alongside source files (`*_test.go`). Use `DescribeTable` as much as possible for table-driven tests.
- Tests mock HTTP calls via `testutil.MockGetter`.
- End-to-end tests run against a local KubeVirt cluster via kubevirtci (`make cluster-up`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kubevirt/containerdisks](https://github.com/kubevirt/containerdisks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
