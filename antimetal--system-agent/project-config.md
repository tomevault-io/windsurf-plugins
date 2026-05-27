---
trigger: always_on
description: This file provides comprehensive guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides comprehensive guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Code Review Guidelines

When performing code reviews on pull requests:

### Feedback Structure
- **IMPORTANT**: Use collapsible sections (`<details>` tags) for non-actionable feedback, explanations, or background information
- Keep actionable items (bugs, required changes) visible by default
- Use this format for non-critical suggestions:

```markdown
<details>
<summary>💡 Suggestion: [Brief description]</summary>

[Detailed explanation or rationale]

</details>
```

### Example Review Format
```markdown
## Review Summary
✅ **Required Changes** (visible by default)
- Fix memory leak in line 42
- Add error handling for null case

<details>
<summary>📚 Code Quality Observations</summary>

- Consider using early returns to reduce nesting
- The function could be split into smaller units
- Variable naming could be more descriptive

</details>

<details>
<summary>🔍 Performance Considerations</summary>

While not critical, you might consider:
- Using a map instead of repeated array lookups
- Caching the compiled regex pattern

</details>
```

### Review Priorities
1. **Always visible**: Security issues, bugs, breaking changes
2. **Collapsible**: Style suggestions, minor optimizations, educational content
3. **Focus on**: Constructive, actionable feedback over nitpicking

## Project Overview

The Antimetal Agent is a Kubernetes controller that connects infrastructure to the Antimetal platform for cloud resource management. It collects K8s resources, monitors system performance, and streams data via gRPC.

### Key Technologies
- **Go 1.24** with controller-runtime framework
- **Kubernetes** custom controller patterns
- **gRPC** for streaming data to intake service
- **BadgerDB** for embedded resource storage
- **Docker** with multi-arch support (linux/amd64, linux/arm64)
- **KIND** for local development and testing

## Architecture Overview

### Core Components

| Component | Path | Purpose |
|-----------|------|---------|
| **Kubernetes Controller** | `internal/kubernetes/agent/` | Watches resources, reconciliation, leader election |
| **Intake Worker** | `internal/intake/` | gRPC streaming, batching, retry logic |
| **Performance Monitoring** | `pkg/performance/` | System metrics from /proc and /sys |
| **Resource Store** | `pkg/resource/store/` | BadgerDB storage, RDF triplets, event subscriptions |
| **Cloud Providers** | `internal/kubernetes/cluster/` | EKS, KIND, extensible provider interface |

### Directory Structure
- `cmd/` - Application entry points
- `internal/` - Private application code (intake, kubernetes controller)
- `pkg/` - Public packages (aws, performance, resource store)
- `config/` - K8s manifests and Kustomize
- `ebpf/` - eBPF programs and build system

## Development Workflow

### Prerequisites
- **Docker** (rootless, containerd snapshotter enabled)
- **kubectl** for K8s operations
- **Go 1.24+** as specified in go.mod

### Common Commands

Run `make help` for the full list. Key commands:

| Category | Command | Purpose |
|----------|---------|---------|
| **Build** | `make build` | Build binary for current platform |
| | `make build-all` | Build for all platforms |
| | `make docker-build-all` | Build multi-arch Docker images |
| **Test** | `make test` | Run tests with coverage |
| | `make lint` | Run golangci-lint |
| | `make fmt` | Format Go code |
| | `make fmt.clang` | Format C/C++/eBPF code |
| **Generate** | `make generate` | Generate K8s manifests |
| | `make gen-license-headers` | **ALWAYS run before committing** |
| **KIND** | `make cluster` | Create local KIND cluster |
| | `make build-and-load-image` | Quick rebuild and deploy |
| | `make destroy-cluster` | Delete KIND cluster |

### Key Development Patterns

#### Code Generation
Always run `make generate` after:
- Modifying kubebuilder annotations (`+kubebuilder:rbac`)
- Changing CRD definitions
- Updating webhook configurations

#### License Headers
- **ALWAYS** run `make gen-license-headers` before committing
- **ALL** .go files must have the PolyForm Shield license header located in @tools/license_check/license_header.txt

#### Testing Philosophy
- Use standard Go testing framework
- Tests located alongside implementation files
- Table-driven tests for comprehensive coverage
- Mock external dependencies (gRPC, AWS, K8s)

#### Git Commits and PRs
- **ALWAYS** run `make lint-fix` before creating a commit
- **ALWAYS** use the `commit-author` agent for creating commit messages, reviewing commits, or generating PR descriptions
- The agent ensures compliance with project commit conventions and formatting standards

#### Linux Systems and eBPF Development
- **ALWAYS** use the `linux-systems-expert` agent when developing or debugging Linux system collectors, eBPF programs, or /proc or /sys parsers
- The agent has deep expertise in kernel interfaces, CO-RE/BTF, performance monitoring, and cross-kernel compatibility

## Performance Collectors


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [antimetal/system-agent](https://github.com/antimetal/system-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
