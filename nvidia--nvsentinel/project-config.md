---
trigger: always_on
description: NVSentinel is a GPU Node Resilience System for Kubernetes that automatically detects, classifies, and remediates hardware and software faults in GPU nodes. It's designed for high-performance computing environments running NVIDIA GPUs.
---

# GitHub Copilot Instructions for NVSentinel

## Project Overview

NVSentinel is a GPU Node Resilience System for Kubernetes that automatically detects, classifies, and remediates hardware and software faults in GPU nodes. It's designed for high-performance computing environments running NVIDIA GPUs.

**Status**: Experimental/Preview Release - APIs and configurations may change.

## Architecture & Technologies

### Core Technologies
- **Language**: Go 1.25+ (primary), Python 3.10+ (monitoring tools)
- **Container Platform**: Kubernetes 1.25+
- **Deployment**: Helm 3.0+, Tilt (development)
- **Storage**: MongoDB (event store with change streams)
- **Communication**: gRPC with Protocol Buffers
- **GPU Monitoring**: NVIDIA DCGM (Data Center GPU Manager)

### Project Structure
```
├── health-monitors/          # Pluggable health detection modules
│   ├── gpu-health-monitor/   # DCGM-based GPU monitoring (Python)
│   ├── csp-health-monitor/   # Cloud provider health checks (Go)
│   └── syslog-health-monitor/ # System log analysis (Go)
├── health-events-analyzer/   # Event classification and routing
├── fault-quarantine/  # Node isolation (cordon)
├── node-drainer/      # Workload eviction
├── fault-remediation/ # Break-fix automation
├── labeler/           # Node labeling (DCGM version, driver status, Kata detection)
├── janitor/                  # State cleanup and maintenance
├── platform-connectors/      # CSP integration (GCP, AWS, Azure)
├── commons/                  # Shared utilities
├── data-models/             # Protocol Buffer definitions
├── store-client/        # MongoDB client library
└── distros/kubernetes/      # Helm charts
```

## Coding Standards

### Go Code Guidelines

#### Module Organization
- Each service is a separate Go module with its own `go.mod`
- Use semantic import versioning
- Keep dependencies minimal and up-to-date
- Use `commons/` for shared utilities across modules

#### Code Style
- Follow standard Go conventions (gofmt, golint)
- Use structured logging via `log/slog`
- Error handling: wrap errors with context using `fmt.Errorf("context: %w", err)`
- Within `retry.RetryOnConflict` blocks, return errors **without wrapping** to preserve retry behavior
- Use meaningful variable names (`synced` over `ok` for cache sync checks)

#### Kubernetes Integration
- Use `client-go` for Kubernetes API interactions
- Prefer informers over direct API calls for watching resources
- Use `envtest` for testing Kubernetes controllers (not fake clients)
- Implement proper shutdown handling with context cancellation

#### Testing Requirements
- Use `testify/assert` and `testify/require` for assertions
- Write table-driven tests when testing multiple scenarios
- Use `envtest` for integration tests with real Kubernetes API
- Test coverage: aim for >80% on critical paths
- Name tests descriptively: `TestFunctionName_Scenario_ExpectedBehavior`

### Python Code Guidelines
- Use Poetry for dependency management
- Follow PEP 8 style guide
- Use Black for formatting
- Type hints required for all functions
- Use dataclasses for structured data

### Protobuf Guidelines
- Define messages in `data-models/protobufs/`
- Use semantic versioning for breaking changes
- Include comprehensive comments for all fields
- Generate code with: `make protos-generate`

## Development Workflows

### Building & Testing
```bash
# Lint and test all modules
make lint-test-all

# Lint specific module
cd labeler && make lint

# Test specific module
cd health-events-analyzer && make test

# Build container images (uses ko)
make images

# View all make targets
make help
```

### Version Management
- All tool versions centralized in `.versions.yaml`
- Use `yq` to read versions in scripts
- Update versions in one place, propagates everywhere
- Check versions: `make show-versions`

### Local Development with Tilt
```bash
cd tilt
tilt up  # Start local development environment
```

## Kata Containers Detection

The labeler implements Kata Containers detection:

### Detection Architecture
- **Input labels** (on nodes): `katacontainers.io/kata-runtime` (default) + optional custom label
- **Output label** (set by labeler): `nvsentinel.dgxc.nvidia.com/kata.enabled: "true"|"false"`
- **Truthy values**: `"true"`, `"enabled"`, `"1"`, `"yes"` (case-insensitive)
- **Lifecycle separation**: Pod events → DCGM/driver labels, Node events → kata labels

### DaemonSet Variants
- Separate DaemonSets for kata vs regular nodes
- Selection via `nodeAffinity` based on kata.enabled label
- Different volume mounts:
  - Regular: `/var/log` (file-based logs)
  - Kata: `/run/log/journal` and `/var/log/journal` (systemd journal)

## Important Patterns

### Error Handling in Retry Loops
```go
// ✅ CORRECT - Return error as-is for retry logic
err := retry.RetryOnConflict(retry.DefaultBackoff, func() error {
    _, err := client.Update(ctx, obj, metav1.UpdateOptions{})
    return err  // Don't wrap!
})

// ❌ WRONG - Wrapping breaks retry detection
err := retry.RetryOnConflict(retry.DefaultBackoff, func() error {
    _, err := client.Update(ctx, obj, metav1.UpdateOptions{})

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NVIDIA/NVSentinel](https://github.com/NVIDIA/NVSentinel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
