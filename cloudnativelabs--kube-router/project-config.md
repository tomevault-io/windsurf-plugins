---
trigger: always_on
description: This document provides comprehensive guidelines for AI coding agents working on the kube-router project. It consolidates
---

# AI Agent Guidelines for kube-router

This document provides comprehensive guidelines for AI coding agents working on the kube-router project. It consolidates
project-specific coding standards, best practices, and workflows to ensure consistent, high-quality contributions.

## Table of Contents

- [Project Overview](#project-overview)
- [Project Structure](#project-structure)
- [Build System and Makefile](#build-system-and-makefile)
- [Go Coding Standards](#go-coding-standards)
- [Error Handling](#error-handling)
- [Logging Practices](#logging-practices)
- [Testing](#testing)
- [Refactoring Guidelines](#refactoring-guidelines)
- [Code Review Process](#code-review-process)
- [Development Workflow](#development-workflow)
- [Planning Guidelines](#planning-guidelines)
- [Markdown Formatting](#markdown-formatting)
- [Kubernetes-Specific Considerations](#kubernetes-specific-considerations)
- [Additional Best Practices](#additional-best-practices)
- [References](#references)
- [Support](#support)

## Project Overview

kube-router is a turnkey solution for Kubernetes networking that provides:

- **IPVS/LVS-based Service Proxy**: High-performance load balancing using Linux kernel features
- **Pod Networking**: Direct routing with BGP protocol via GoBGP
- **Network Policy Controller**: Firewall rules using ipsets and iptables
- **Advanced BGP Capabilities**: Integration with external networking devices

### Key Technologies

- **Language**: Go
- **Kubernetes API**: v0.34.2 or latest
- **Key Libraries**: GoBGP, Netlink, iptables, ipset, IPVS
- **Logging**: klog/v2
- **Testing**: Testify, Ginkgo, Gomega

## Project Structure

```text
.
├── cmd/                    # Main applications
│   └── kube-router/        # Main application entry point
├── pkg/                    # Public library code
│   ├── healthcheck/        # Logic for monitoring control loops
│   └── controllers/        # Main control loop logic
│        ├── lballoc/       # LoadBalancer controller IPAM logic
│        ├── netpol/        # NetworkPolicy enforcement logic
│        ├── proxy/         # Services controller logic
│        └── routing/       # Pod & Service routing logic
├── internal/               # Private implementation code
│   ├── nlretry/            # Netlink retry logic
│   └── testutils/          # Internal test utilities
├── testdata/               # Test fixtures and data
├── daemonset/              # Kubernetes deployment manifests
├── cni/                    # Kubernetes CNI configuration files
└── docs/                   # Documentation in markdown format
```

### File Naming Conventions

1. Use snake_case for file names:

```golang
// Good
api_handler.go
user_service.go

// Bad
apiHandler.go
UserService.go
```

1. Test files should use _test suffix:

```golang
// Good
api_handler_test.go

// Bad
api_handler_tests.go
```

1. Package names should be short and clear:

```golang
// Good
package api
package validator

// Bad
package apihandlers
package utility_functions
```

## Build System and Makefile

kube-router uses a comprehensive Makefile for building, testing, linting, and releasing. **Always use Makefile targets
instead of running commands directly** when available.

### Common Makefile Targets

#### Building

- `make kube-router` - Build the kube-router binary
- `make container` - Build the container image
- `make gobgp` - Build the gobgp binary
- `make all` - Default target: lint, test, build binaries and images

#### Testing and Formatting

- `make test` - Run all tests with verbose output
- `make test-pretty` - Run tests with formatted output using gotestsum
- `make gofmt` - Check which files need formatting
- `make gofmt-fix` - Automatically fix formatting issues

#### Linting

- `make lint` - Run golangci-lint and markdownlint
- `make markdownlint` - Run markdown linting on README and docs

#### Code Generation

- `make gomoqs` - Generate all mock files using moq
- Individual mocks: `make pkg/controllers/proxy/linux_networking_moq.go`

#### Dependency Management

- `make update-deps` - Update all dependency versions, resolve digests, and pin SHAs
- `make update-deps-dry` - Preview what dependency updates would be made (dry-run with diff output)

The `update-deps` target runs the `build/dependency-updater` Go tool, which:

- Discovers all managed files dynamically (no static file list)
- Updates Docker image tags to their latest versions within constraints defined in
  `build/dependency-updater/versions.lock.yaml`
- Resolves Docker image tags to `image:tag@sha256:digest` format for reproducibility
- Updates tool version variables (e.g. `GOBGP_VERSION`) to their latest GitHub releases
- Updates GitHub Action `uses:` lines from bare tags (`@v6`) or existing SHA pins to the
  latest SHA-pinned form (`@sha256...  # vX.Y.Z`)
- Updates the `toolchain` directive in `go.mod`

Set `GITHUB_TOKEN` in your environment to avoid GitHub API rate limits (60/hr unauthenticated
vs 5000/hr authenticated):

```bash
GITHUB_TOKEN=ghp_... make update-deps
```

#### Release Preparation

- `make prep-release` - Full release preparation: update deps, run all checks, build container

This is the single command to run before tagging a new release. It calls `update-deps` first,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cloudnativelabs/kube-router](https://github.com/cloudnativelabs/kube-router) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
