---
trigger: always_on
description: This document serves as a comprehensive briefing for AI coding assistants working with the operator-controller
---

# AGENTS.md - AI Coding Assistant Briefing

This document serves as a comprehensive briefing for AI coding assistants working with the operator-controller
repository. It covers the WHAT, WHY, and HOW of contributing to this codebase.

---

## Architecture Overview

operator-controller is the central component of Operator Lifecycle Manager (OLM) v1, extending Kubernetes with APIs to
install and manage cluster extensions. The project follows a microservices architecture with two main binaries:

**operator-controller** 
 - manages `ClusterExtension` and `ClusterObjectSet` CRDs
 - resolves bundles from configured source
 - unpacks bundles and renders manifests from them
 - applies manifests with phase-based rollouts
 - monitors extension lifecycle

**catalogd**
 - manages user-defined `ClusterCatalog` resources to make references catalog metadata available to the cluster.
 - unpacks and serves operator catalog content via HTTP.
 - serves catalog metadata to clients in the cluster that need to use or present information about the contents of the
   catalog. For example, operator-controller queries catalogd for available bundles.

---

## Tech Stack

**Languages:**
- **Go:** The Go version used by the project often lags the latest upstream available version in order to give
  integrators the ability to consume the latest versions of OLMv1 without being required to also consume the latest
  versions of Go.
- **Runtime Platform:** Linux containers (multi-arch: amd64, arm64, ppc64le, s390x)
- **Developer Platform:** Generally macOS and Linux. It is important that all shell commands used in Makefiles and
  other helper scripts work on both Linux and macOS.

**Core Frameworks:**
- **Kubernetes:** client-go, api, apimachinery
- **controller-runtime**
- **operator-framework/api:** For OLMv0 API types that are relevant to OLMv1
- **operator-registry** For file-based catalog (FBC) processing
- **Helm:** helm-operator-plugins (which depends on helm itself)

**Key Dependencies:**
- **cert-manager**
- **boxcutter (package-operator.run)**

**Container Base:**
- Base image: `gcr.io/distroless/static:nonroot`
- User: `65532:65532` (non-root)

**Build Tags:**
- `containers_image_openpgp` - required for image handling

**Tools (managed via .bingo/):**
- controller-gen, golangci-lint, goreleaser, helm, kind, kustomize, mockgen, setup-envtest, operator-sdk

**Test Mocking:**
- **gomock** (`go.uber.org/mock`): Used for generated mock implementations. `//go:generate mockgen` directives
  live in `internal/testutil/mock/generate.go` (external and internal exported interfaces) and at interface
  definitions for internal unexported interfaces (source mode). Run `make generate-mocks` to regenerate.
  All interface mocks should be gomock-generated. Test fakes that return preconfigured values without
  interaction verification (e.g., `FakePuller`, `FakeCache` in `image/fakes.go`) are not mocks and stay
  hand-written.

---

## Build & Test Commands

### Build

```bash
# Build for local platform
make build

# Build for Linux (required for docker)
make build-linux

# Build docker images
make docker-build

# Full release build
make release
```

### Test

```bash
# Unit tests (uses ENVTEST)
make test-unit

# E2E tests
make test-e2e                      # Standard features
make test-experimental-e2e         # Experimental features
make test-extension-developer-e2e  # Extension developer workflow

# E2E tests in parallel (runs standard and experimental on separate KIND clusters)
# Recommended when machine has enough resources (4+ CPU cores, 8+ GB RAM) for faster feedback
make -j2 test-e2e test-experimental-e2e
# Linux prerequisite: sudo sysctl fs.inotify.max_user_instances=512

# Regression tests
make test-regression

# All (non-upgrade, non-experimental) tests
make test
```

### Iterative E2E Development

```bash
# Set up a persistent e2e cluster (does not tear down after tests)
make e2e-setup                     # Standard features
make experimental-e2e-setup        # Experimental features

# Run e2e scenarios against the running cluster
make e2e/install                              # All scenarios in install.feature
make e2e/install/Install                      # Scenarios starting with "Install"
make "e2e/install/Install latest"             # Exact prefix with spaces
make e2e/install E2E_TIMEOUT=30m              # Override timeout
make e2e/install KUBECONFIG=~/.kube/config    # Override kubeconfig

# Run against experimental cluster (override KUBECONFIG)
make e2e/install/Install KUBECONFIG=.kubeconfig/operator-controller-experimental-e2e.kubeconfig

# Tear down the e2e cluster when done
make e2e-teardown                  # Standard cluster
make experimental-e2e-teardown     # Experimental cluster
```

### Linting & Verification

```bash
# Run golangci-lint
make lint

# Run helm lint
make lint-helm

# Verify all generated code is up-to-date
make verify

# Format code
make fmt

# Fix lint issues automatically
make fix-lint
```

### Local Development

```bash
# Create kind cluster and deploy
make run                    # Standard manifest
make run-experimental       # Experimental manifest

# OR step by step:
make kind-cluster          # Create cluster

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [operator-framework/operator-controller](https://github.com/operator-framework/operator-controller) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
