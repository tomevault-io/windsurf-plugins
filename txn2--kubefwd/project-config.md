---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

kubefwd is a popular command-line utility for bulk Kubernetes port forwarding, featured in "essential k8s developer tools" lists. It forwards multiple services from one or more namespaces, adding corresponding entries to `/etc/hosts` for local development.

**Key Differentiator from `kubectl port-forward`**: Each service gets its own unique loopback IP address (127.x.x.x), allowing multiple services to use the same port simultaneously (e.g., multiple databases on port 3306, or multiple web services on port 80). This mirrors how services work in the cluster, enabling truly cluster-like local development.

The tool automatically monitors service and pod lifecycle events, starting/stopping port forwards as services are created, deleted, or pods are rescheduled.

## Integration and Usage Patterns

kubefwd is commonly integrated into development workflows:

- **With Tilt**: Blog posts document using kubefwd with Tilt for automated local development setups
- **Development Environment Setup**: Teams use kubefwd to mirror production-like service topologies locally
- **Microservices Development**: Allows developers to run one service locally while accessing dependencies in the cluster via service names
- **Database Access**: Popular for forwarding multiple databases (MySQL, PostgreSQL, MongoDB) on their native ports without port conflicts

The typical workflow: `sudo -E kubefwd svc -n <namespace> --tui` runs in a dedicated terminal while developers work in their IDE, accessing cluster services as if running in-cluster.

## TUI (Terminal User Interface)

kubefwd includes an interactive TUI built with [Bubble Tea](https://github.com/charmbracelet/bubbletea), enabled with `--tui`:

```bash
sudo -E kubefwd svc -n default --tui
```

### TUI Features
- **Real-time service monitoring**: See all forwarded services with connection status
- **Traffic metrics**: Bytes in/out with sparkline graphs
- **Pod log streaming**: View container logs directly in the TUI
- **HTTP activity detection**: Monitor HTTP requests/responses flowing through forwards
- **Auto-reconnect**: Enabled by default in TUI mode, with exponential backoff (1s → 5min max)
- **Keyboard-driven**: Full keyboard navigation (j/k, Enter, Tab, /, ?, q)

### TUI Architecture

The TUI follows the Model-View-Update (MVU) pattern:

- **Event Bus** (`pkg/fwdtui/events`): Decoupled pub/sub communication between components
- **State Store** (`pkg/fwdtui/state`): Centralized, thread-safe state management for all forwards
- **Metrics Registry** (`pkg/fwdtui/metrics`): Traffic counters with atomic operations
- **Components** (`pkg/fwdtui/components`): UI models for services list, logs, detail view

### TUI Keyboard Shortcuts
- `j/k` or arrows: Navigate
- `Enter`: Open detail view
- `Tab`: Switch panels/tabs
- `/`: Filter services
- `r`: Reconnect errored services
- `b`: Toggle bandwidth columns
- `?`: Help overlay
- `q`: Quit

## Git Policy

**All commits require explicit human approval.** Do not run `git commit` unless the user explicitly asks you to commit. Always show the diff or summarize changes and wait for approval first.

## Build and Development Commands

### Building
```bash
# Build the project (uses goreleaser)
go build -o kubefwd ./cmd/kubefwd/kubefwd.go

# Build with version information
go build -ldflags "-X main.Version=dev" -o kubefwd ./cmd/kubefwd/kubefwd.go
```

### Testing
```bash
# Run all tests
go test ./...

# Run tests for a specific package
go test ./pkg/fwdport

# Run tests with verbose output
go test -v ./pkg/fwdport
```

### Running Locally
```bash
# Requires root/sudo for network interface management and /etc/hosts modification
sudo ./kubefwd svc -n <namespace>

# Use -E flag to preserve environment (especially KUBECONFIG)
sudo -E ./kubefwd svc -n <namespace>

# With interactive TUI (recommended for development)
sudo -E ./kubefwd svc -n <namespace> --tui
```

### Dependencies
```bash
# Download dependencies
go mod download

# Update dependencies
go mod tidy
```

### Debugging and Development

```bash
# Enable verbose logging
sudo -E ./kubefwd svc -n <namespace> -v

# Test with a single service using field selector
sudo -E ./kubefwd svc -n <namespace> -f metadata.name=<service-name>

# Test with label selector
sudo -E ./kubefwd svc -n <namespace> -l app=myapp

# Use IP reservation config for reproducible testing
sudo -E ./kubefwd svc -n <namespace> -z example.fwdconf.yml
```

The verbose flag (`-v`) enables debug-level logging (logrus.DebugLevel), which shows:
- Service registry operations
- Pod sync events
- IP allocation details
- Port forwarding lifecycle events

### Demo Environment

A comprehensive demo manifest is available for testing:

```bash
# Deploy 60 services across 2 namespaces
kubectl apply -f test/manifests/demo-microservices.yaml

# Forward all demo services with TUI
sudo -E ./kubefwd svc -n kft1,kft2 --tui

# Cleanup
kubectl delete -f test/manifests/demo-microservices.yaml
```

## Architecture

### Core Components Flow

1. **Entry Point** (`cmd/kubefwd/kubefwd.go`): CLI entry, delegates to services command

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [txn2/kubefwd](https://github.com/txn2/kubefwd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
