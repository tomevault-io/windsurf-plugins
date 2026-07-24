---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

KECS (Kubernetes-based ECS Compatible Service) is a standalone service that provides Amazon ECS compatible APIs running on Kubernetes. It enables a fully local ECS-compatible environment that operates independently of AWS environments.

## Common Development Commands

### Building and Running
```bash
make build          # Build both CLI and server binaries
make build-cli      # Build CLI binary only (bin/kecs - no DuckDB/CGO)
make build-server   # Build server binary (bin/kecs-server - with DuckDB/CGO)
make run            # Build and run CLI
make run-server     # Build and run server
make all            # Clean, format, vet, test, and build
```

#### Binary Architecture
KECS is split into two binaries to resolve cross-compilation issues:
- **kecs** (CLI): Lightweight client without DuckDB, cross-platform compatible
- **kecs-server**: Full server with DuckDB support, runs in containers

### Container-based Execution
```bash
kecs start          # Start KECS in a Docker container
kecs stop           # Stop and remove KECS container
kecs status         # Show container status
kecs logs -f        # Follow container logs

# Multiple instances
kecs start --instance dev --api-port 8080
kecs start --instance staging --api-port 8090 --auto-port
kecs list           # List all instances
```

### Testing and Code Quality
```bash
make test           # Run tests with race detection
make test-coverage  # Run tests with coverage report
make vet            # Run go vet
make fmt            # Format code with gofmt
```

#### Testing Framework
KECS uses **Ginkgo** as the primary testing framework for Go tests:
- Ginkgo provides BDD-style testing with descriptive test specifications
- Tests should be written using Ginkgo's `Describe`, `Context`, `It` patterns
- Use Gomega matchers for assertions
- Place test files as `*_test.go` alongside the code they test

Example test structure:
```go
var _ = Describe("ClusterHandler", func() {
    Context("when creating a cluster", func() {
        It("should return existing cluster when name already exists", func() {
            // Test implementation
            Expect(response).To(Equal(expectedResponse))
        })
    })
})
```

### Docker Operations
```bash
make docker-build   # Build Docker image
make docker-push    # Build and push Docker image
```

### Development Workflow
```bash
make deps           # Download and verify dependencies
make clean          # Clean build artifacts and coverage files

# Hot reload development workflow
./bin/kecs start    # Start KECS instance
make dev            # Build and hot reload controlplane
make dev-logs       # Build, hot reload, and tail logs

# For specific instance
KECS_INSTANCE=myinstance make dev
```

### Kubeconfig Management
```bash
# List all available KECS clusters
kecs kubeconfig list

# Get kubeconfig for a specific instance
kecs kubeconfig get <instance-name>

# Write kubeconfig to a file
kecs kubeconfig get <instance-name> -o ~/.kube/kecs-config

# Use the kubeconfig with kubectl
export KUBECONFIG=$(kecs kubeconfig get <instance-name>)
kubectl get pods -n kecs-system

# Or specify inline
kubectl --kubeconfig <(kecs kubeconfig get <instance-name>) get pods -n kecs-system
```

### Hot Reload Development
KECS supports hot reloading of the controlplane during development:
1. **Start KECS**: Run `./bin/kecs start` to create a k3d cluster with KECS
2. **Make changes**: Edit your Go code in the controlplane
3. **Hot reload**: Run `make dev` to build and deploy changes
4. **View logs**: Use `make dev-logs` to reload and tail logs in real-time

The Docker hot reload workflow:
- Builds a new Docker image with your changes
- Pushes it to the local k3d registry
- Updates the running deployment without cluster restart
- Maintains all existing ECS resources and state


## Architecture Overview

KECS implements a clean architecture with the following key components:

### Dual Server Design
- **API Server (port 8080)**: Handles ECS-compatible API requests at `/v1/<action>` endpoints
- **Admin Server (port 8081)**: Provides operational endpoints like `/health`

### Directory Structure
- `cmd/controlplane/`: Entry point for the control plane binary
- `internal/controlplane/cmd/`: CLI command implementations using Cobra
- `internal/controlplane/api/`: ECS API endpoint implementations
- `internal/controlplane/admin/`: Admin server with health checks
- `internal/converters/`: Task definition to Kubernetes resource converters
- `internal/kubernetes/`: Kubernetes client and resource managers
- `internal/storage/`: Storage interfaces and DuckDB implementation
- `docs/adr/records/`: Architectural Decision Records
- `docs-site/`: VitePress-based documentation site

### API Implementation Pattern
Each ECS resource type has its own file in `internal/controlplane/api/` with:
- Request/Response struct definitions matching AWS ECS API
- Handler function registered in `api/server.go`
- Current implementations return mock responses with TODO comments

### Key Architectural Decisions
- Uses standard `net/http` for HTTP servers
- Graceful shutdown with 10-second timeout

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nandemo-ya/kecs](https://github.com/nandemo-ya/kecs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
