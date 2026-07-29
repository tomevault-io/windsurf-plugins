---
trigger: always_on
description: The FoundationDB Kubernetes Operator is a sophisticated Kubernetes controller that manages FoundationDB clusters in Kubernetes environments. It automates deployment, scaling, backup, restore, and maintenance operations for FoundationDB distributed database clusters.
---

# FoundationDB Kubernetes Operator - Claude Development Guide

## Project Overview

The FoundationDB Kubernetes Operator is a sophisticated Kubernetes controller that manages FoundationDB clusters in Kubernetes environments. It automates deployment, scaling, backup, restore, and maintenance operations for FoundationDB distributed database clusters.

### Architecture Components

- **API Types** (`api/v1beta2/`): Custom Resource Definitions (CRDs) for FoundationDBCluster, FoundationDBBackup, FoundationDBRestore
- **Controllers** (`controllers/`): Reconciliation logic for cluster lifecycle management
- **Internal Packages** (`internal/`): Core business logic for coordination, replacements, maintenance, etc.
- **PKG Packages** (`pkg/`): Reusable components like admin clients, pod managers, status checks
- **E2E Tests** (`e2e/`): Comprehensive end-to-end testing with chaos engineering

### Key Patterns

- **Controller-Runtime**: Built on `sigs.k8s.io/controller-runtime` framework
- **Reconciliation Loops**: Event-driven state reconciliation
- **Mock-Based Testing**: Extensive mocking for unit tests
- **Chaos Engineering**: Production-like failure testing with chaos-mesh

## Development Environment Setup

### Prerequisites

```bash
# Go 1.24+ required
go version

# Install dependencies
make deps

# Install FoundationDB client package
# For macOS: Download from https://github.com/apple/foundationdb/releases
# For arm64 Mac: Make sure to install the arm64 package
```

### Local Development

```bash
# Clone repository
git clone https://github.com/FoundationDB/fdb-kubernetes-operator
cd fdb-kubernetes-operator

# Set up test certificates
config/test-certs/generate_secrets.bash

# Build and deploy operator (requires local K8s cluster)
make rebuild-operator

# Create test cluster
kubectl apply -k ./config/tests/base
```

## Build System & Tooling

### Primary Make Commands

| Command | Purpose |
|---------|---------|
| `make all` | Complete build pipeline: deps, generate, fmt, vet, test, build |
| `make test` | Run unit tests (Ginkgo with race detection if TEST_RACE_CONDITIONS=1) |
| `make lint` | Run golangci-lint with project rules |
| `make fmt` | Format code using golines + goimports + golangci-lint --fix |
| `make vet` | Run go vet static analysis |
| `make generate` | Generate deepcopy methods and CRDs |
| `make manifests` | Generate CRD YAML files |
| `make container-build` | Build Docker image |
| `make deploy` | Deploy operator to Kubernetes cluster |
| `make rebuild-operator` | Build, push (if remote), deploy, and bounce operator |

### Environment Variables

- `IMG`: Operator image name (default: `fdb-kubernetes-operator:latest`)
- `SIDECAR_IMG`: Sidecar image name
- `REMOTE_BUILD`: Set to 1 for remote builds (enables image push)
- `BUILD_PLATFORM`: Override build platform (e.g., `linux/amd64`)
- `TEST_RACE_CONDITIONS`: Set to 1 to enable race detection in tests
- `SKIP_TEST`: Set to 1 to skip tests in build

## Testing Framework

### Unit Testing (Ginkgo v2 + Gomega)

```go
// Example test structure from controllers/suite_test.go
var _ = Describe("ControllerName", func() {
    BeforeEach(func() {
        // Setup test environment
        k8sClient = mockclient.NewMockClient()
    })

    It("should reconcile successfully", func() {
        // Test implementation
        Expect(result).To(BeNil())
    })
})
```

### E2E Testing

- **Location**: `e2e/` directory with test packages
- **Framework**: Ginkgo + Gomega + chaos-mesh for failure injection
- **Types**: Upgrades, HA failures, stress testing, maintenance mode
- **Run**: `make test` with e2e labels

### Mock Objects

- **Kubernetes Client**: `mockclient.MockClient`
- **FDB Admin Client**: `mock.DatabaseClientProvider`
- **Pod Client**: `mockpodclient.NewMockFdbPodClient`

## Code Standards & Conventions

### Linting & Formatting

**golangci-lint Configuration** (`.golangci.yml`):
- **Enabled Linters**: errcheck, govet, staticcheck, revive, misspell, ineffassign, unused
- **Formatters**: gofmt with golines (120 char limit)
- **Dependency Guard**: Restricted import paths for clean architecture

**Formatting Tools**:
- `golines`: Line length formatting with gofmt base
- `goimports`: Import organization
- `golangci-lint run --fix`: Auto-fix issues

### Package Organization

```
├── api/v1beta2/           # CRD types and API definitions
├── controllers/           # Controller reconciliation logic
├── internal/              # Internal business logic packages
├── pkg/                   # Reusable library packages
├── e2e/                   # End-to-end tests
├── kubectl-fdb/           # kubectl plugin
├── fdbclient/             # FDB client utilities
└── setup/                 # Operator setup and configuration
```

### Naming Conventions

- **Types**: PascalCase (e.g., `FoundationDBCluster`)
- **Functions**: PascalCase for exported, camelCase for internal
- **Constants**: PascalCase or UPPER_SNAKE_CASE for public constants
- **Files**: snake_case.go
- **Test Files**: `*_test.go` with corresponding suite_test.go

### Error Handling

```go

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FoundationDB/fdb-kubernetes-operator](https://github.com/FoundationDB/fdb-kubernetes-operator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
