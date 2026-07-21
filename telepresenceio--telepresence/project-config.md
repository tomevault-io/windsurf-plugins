---
trigger: always_on
description: This file provides guidance for contributors and AI assistants working with this repository.
---

# AGENTS.md

This file provides guidance for contributors and AI assistants working with this repository.

## Project Overview

Telepresence is a Kubernetes development tool that enables fast local development by connecting your local workstation to a Kubernetes cluster. It allows developers to run services locally while accessing cluster resources and intercepting traffic from the cluster to their local machine.

## Git Workflow

- Never commit directly to the `release/v2` branch. Always create a feature branch with a name following the pattern `username/topic` (e.g., `thallgren/fix-dns-resolution`).
- All commits must be signed and signed-off (`git commit -s -S`).
- Limit commit message subjects to 72 characters. Do not wrap subjects;
  rewrite them shorter instead. Wrap commit message body lines at 72
  characters by default unless preserving exact external text requires a
  longer line.
- **Always run `make lint` and fix every reported issue before pushing.** This is non-negotiable — CI runs the same linters and a push with lint errors wastes a CI cycle. If `make lint` finds problems, fix them in the appropriate commit (use `git commit --fixup=<sha>` followed by `GIT_SEQUENCE_EDITOR=: git rebase -i --autosquash --gpg-sign <base>` to fold them in) before pushing.
- Push the branch and create a pull request for review.
- Always merge PRs with a merge commit (never squash or rebase).

## Design Plans

Major work (multi-file changes, new features, refactors) starts with a written plan
under `docs/plans/<topic>/`, presented for review before implementation begins.

A plan is scaffolding for review, not a lasting artifact. It is removed in the last
commit on the PR that implements it. By then, everything in the plan must have been
implemented and documented, so the plan no longer has a purpose.

## Build Artifacts

The Open Source version of Telepresence consists of three artifacts:

**Client-side (runs on developer workstation):**
- **`telepresence` binary** - The same binary serves as CLI, user daemon, and root daemon.
- **`telepresence` Docker image** - Used as both user and root daemon when running `telepresence connect --docker`.

**Cluster-side (runs in Kubernetes):**
- **`tel2` Docker image** - Used by the traffic-manager deployment and injected as traffic-agent sidecars.

## Build Commands

```bash
# Set required environment variables
export TELEPRESENCE_VERSION=v2.x.x-alpha.0  # or use auto-generated version
export TELEPRESENCE_REGISTRY=local          # 'local' for Docker Desktop, or 'ghcr.io/telepresenceio'

# Build the telepresence binary
make build

# Build Docker images (for local Kubernetes development)
make client-image    # Client container image
make tel2-image      # Traffic-manager/traffic-agent image

# Build everything for local development
make build client-image tel2-image

# Install to system
make install

# Clean build artifacts
make clean
make clobber  # Also removes tools
```

Environment variables:
- `TELEPRESENCE_REGISTRY` (required) - Docker registry for images. Use `local` for docker-based Kubernetes, or `ghcr.io/telepresenceio` for the release registry.
- `TELEPRESENCE_VERSION` (optional) - Version string to compile into binaries and images. If not set, auto-generated from CHANGELOG.yml and source hash.

Run `make help` for more information.

### Building on Windows

Windows builds use `build-aux\winmake.bat` instead of `make` directly. Pass the same parameters as you would to make. The script runs make inside a Docker container with appropriate parameters for Windows binaries.

## Testing

```bash
# Run unit tests
make check-unit

# Run all integration tests (requires Kubernetes cluster)
make check-integration

# Run a single integration test
go test ./integration_test/... -v -testify.m=Test_InterceptDetailedOutput

# Run an integration test suite
TEST_SUITE='^WorkloadConfiguration$' go test ./integration_test/... -v

# Build tests without running (useful for caching)
make build-tests
```

Integration tests use testify suites. The test harness is in `integration_test/itest/`. Use `-testify.m=<pattern>` to filter tests by name. Verbose output (`-v`) is recommended as tests produce human-readable output with timestamps that correlate with log files.

### Integration Test Environment Variables

| Environment Name           | Description                                   | Default                   |
|----------------------------|-----------------------------------------------|---------------------------|
| `DEV_KUBECONFIG`           | Cluster configuration used by the tests       | Kubernetes default        |
| `DEV_CLIENT_REGISTRY`      | Docker registry for the client image          | ${TELEPRESENCE_REGISTRY}  |
| `DEV_MANAGER_REGISTRY`     | Docker registry for the traffic-manager image | ${TELEPRESENCE_REGISTRY}  |
| `DEV_AGENT_REGISTRY`       | Docker registry for the traffic-agent image   | Traffic-manager registry  |
| `DEV_CLIENT_IMAGE`         | Name of the client image                      | "telepresence"            |
| `DEV_MANAGER_IMAGE`        | Name of the traffic-manager image             | "tel2"                    |
| `DEV_AGENT_IMAGE`          | Name of the traffic-agent image               | Traffic-manager image     |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [telepresenceio/telepresence](https://github.com/telepresenceio/telepresence) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
