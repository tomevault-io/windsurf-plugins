---
trigger: always_on
description: KubeNexus Scheduler is a Kubernetes scheduler extender optimized for GPU-intensive AI/ML workloads at hyperscale. Built on the kube-scheduler framework (K8s 1.35+), it provides 14+ scheduling plugins for gang scheduling, GPU topology awareness, VRAM management, network fabric co-location, NUMA optimization, and multi-tenant resource isolation.
---

# KubeNexus Scheduler - Agent Development Guide

KubeNexus Scheduler is a Kubernetes scheduler extender optimized for GPU-intensive AI/ML workloads at hyperscale. Built on the kube-scheduler framework (K8s 1.35+), it provides 14+ scheduling plugins for gang scheduling, GPU topology awareness, VRAM management, network fabric co-location, NUMA optimization, and multi-tenant resource isolation.

## Build/Lint/Test Commands

### Building
```bash
make build                    # Build scheduler + webhook binaries
make docker-build             # Build Docker images
go build -o bin/kubenexus-scheduler ./cmd/scheduler/
go build -o bin/kubenexus-webhook ./cmd/webhook/
```

### Linting
```bash
make lint                     # Run golangci-lint
make fmt                      # Format Go code
make vet                      # Run go vet
```

### Testing
```bash
make test                     # Run all unit tests
go test ./pkg/... -v          # Run all package tests
go test ./pkg/plugins/coscheduling/ -v  # Test single plugin
go test ./test/integration/ -v          # Integration tests
go test ./test/benchmark/ -bench=.      # Benchmarks

# Run specific test
go test -run TestCoscheduling ./pkg/plugins/coscheduling/

# E2E tests (requires kind cluster)
./hack/test-setup.sh          # Set up kind cluster
./hack/test-workloads.sh      # Deploy test workloads
go test ./test/e2e/ -v        # Run e2e suite
```

### Code Generation
```bash
./hack/update-codegen.sh      # Generate deepcopy, client code for CRDs
```

## Repository Structure

### Core Services (`/cmd/`)
- `scheduler/main.go` - Main entry point, registers all plugins with kube-scheduler framework
- `webhook/main.go` - Admission webhook for pod mutation (injects scheduler name, defaults)

### Scheduler Plugins (`/pkg/plugins/`) - Execution Order

**Phase 1: Classification (PreFilter)**
1. `profileclassifier` - Central hub: classifies tenant tier, workload type, gang status. Stores in CycleState for all other plugins.

**Phase 2: Gang Coordination (PreFilter, QueueSort, Permit, Reserve)**
2. `coscheduling` - Gang scheduling: FIFO ordering, starvation prevention, permit-wait-for-gang
3. `resourcereservation` - Palantir-style capacity booking via ResourceReservation CRD

**Phase 3: Filtering (Filter)**
4. `networkfabric` - NVLink clique filtering: hard-reject nodes in wrong NVLink partition for gang pods with `require-clique: true` or `co-locate: strict`
5. `numatopology` - NUMA alignment filter (single-numa-node policy for large pods)
6. `vramscheduler` - VRAM capacity filter (reject nodes with insufficient GPU memory)

**Phase 4: Scoring (Score)**
7. `workloadaware` - Bin-pack batch, spread services
8. `topologyspread` - Zone-aware HA spreading
9. `backfill` - Opportunistic scheduling for preemptible pods
10. `resourcefragmentation` - GPU island protection, pristine island preservation
11. `tenanthardware` - Match tenant tier to GPU tier (Gold→H100, Bronze→T4)
12. `vramscheduler` - VRAM utilization scoring with tenant-specific thresholds
13. `networkfabric` - Multi-level topology co-location for gang members (NVLink clique > fabric domain > rack > AZ)
14. `numatopology` - NUMA fit quality scoring

**Phase 5: Preemption (PostFilter)**
16. `preemption` - Gang-aware multi-victim preemption with tenant-tier protection

### Supporting Packages (`/pkg/`)
- `apis/scheduling/v1alpha1/` - ResourceReservation CRD types
- `scheduler/` - Shared types (SchedulingConfig, PodGroupStatus) and Prometheus metrics
- `utils/` - Pod group label parsing, PodGroupManager
- `workload/` - Pod workload classification (Service/Batch/Training/Inference)
- `webhook/` - Pod mutation webhook logic

### Configuration (`/config/`)
- `config.yaml` - Scheduler configuration
- `crd-resourcereservation.yaml` - ResourceReservation CRD manifest
- `crd-workload.yaml` - Workload CRD manifest

### Deployment (`/deploy/`)
- `kubenexus-scheduler.yaml` - Single-instance deployment
- `kubenexus-scheduler-ha.yaml` - HA deployment with leader election
- `webhook.yaml` - Admission webhook deployment

## Code Style Guidelines

### Import Organization
Organize imports in three groups separated by blank lines:
```go
import (
    // 1. Standard library
    "context"
    "fmt"
    "time"

    // 2. External dependencies (k8s, prometheus)
    v1 "k8s.io/api/core/v1"
    "k8s.io/apimachinery/pkg/runtime"
    framework "k8s.io/kube-scheduler/framework"

    // 3. Internal packages
    "github.com/kube-nexus/kubenexus-scheduler/pkg/plugins/profileclassifier"
    schedulermetrics "github.com/kube-nexus/kubenexus-scheduler/pkg/scheduler"
)
```

### Naming Conventions
- **Files**: snake_case (`gang_preemption.go`, `pod_mutation.go`)
- **Plugin names**: PascalCase constants (`Name = "Coscheduling"`)
- **Labels/Annotations**: DNS-style (`scheduling.kubenexus.io/vram-request`)
- **Test files**: Same directory as source, `_test.go` suffix
- **Packages**: Lowercase, single word when possible

### Plugin Pattern
Every plugin follows this structure:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kube-nexus/kubenexus-scheduler](https://github.com/kube-nexus/kubenexus-scheduler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
