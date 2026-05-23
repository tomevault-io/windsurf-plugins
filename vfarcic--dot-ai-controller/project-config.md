---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Info

- **GitHub Organization**: `vfarcic`
- **Repository**: `vfarcic/dot-ai-controller`

## Project Overview

DevOps AI Toolkit Controller - A Kubernetes controller built with Kubebuilder v4.7.1. The controller provides five main capabilities:

1. **Solution CRD** (`dot-ai.devopstoolkit.live/v1alpha1`): Resource tracking and lifecycle management
   - Groups related Kubernetes resources as logical solutions
   - Manages ownerReferences for automatic cleanup
   - Aggregates health status across tracked resources
   - Standalone feature - no external dependencies

2. **RemediationPolicy CRD** (`dot-ai.devopstoolkit.live/v1alpha1`): Event-driven remediation
   - Watches Kubernetes events with configurable filtering
   - Integrates with DevOps AI Toolkit MCP for AI-powered remediation
   - Supports automatic and manual modes
   - Includes rate limiting and Slack notifications
   - Requires external MCP endpoint

3. **ResourceSyncConfig CRD** (`dot-ai.devopstoolkit.live/v1alpha1`): Resource visibility and semantic search
   - Discovers all resource types in cluster via Discovery API
   - Watches for resource changes (create, update, delete)
   - Syncs resource metadata to MCP for semantic search
   - Debounces changes to reduce API calls
   - Periodic full resync catches missed events
   - Requires external MCP endpoint

4. **CapabilityScanConfig CRD** (`dot-ai.devopstoolkit.live/v1alpha1`): Autonomous capability discovery
   - Detects CRD changes (create, update, delete) automatically
   - Triggers capability scans when new CRDs are installed
   - Syncs cluster state with MCP on controller restart
   - Supports resource filtering with include/exclude patterns
   - Requires external MCP endpoint

5. **GitKnowledgeSource CRD** (`dot-ai.devopstoolkit.live/v1alpha1`): Documentation ingestion from Git
   - Clones Git repositories and syncs documents to MCP knowledge base
   - Supports glob patterns for file selection (e.g., `docs/**/*.md`)
   - Change detection syncs only modified files since last commit
   - Scheduled sync with cron or interval expressions
   - Automatic cleanup removes documents when CR is deleted
   - Requires external MCP endpoint

## Development Commands

### Build and Test
```bash
# Build the controller binary
make build

# Run locally (requires kubeconfig)
make run

# Run all tests (unit + integration using envtest)
make test

# Run unit tests only
go test $(go list ./... | grep -v /e2e)

# Run e2e tests (creates Kind cluster named 'controller-init-test-e2e')
# Uses isolated kubeconfig 'e2e-kubeconfig' in current directory
make test-e2e

# Cleanup e2e cluster when done
make cleanup-test-e2e
```

### Code Generation (Required after API changes)
```bash
# Generate manifests (CRDs, RBAC, etc.)
make manifests

# Generate deepcopy methods
make generate

# Format and validate code
make fmt vet

# Lint code
make lint
make lint-fix
```

### Cluster Operations
```bash
# Install CRDs into cluster
make install

# Deploy controller to cluster
make deploy IMG=<registry>/controller:tag

# Apply sample resources
kubectl apply -k config/samples/

# Remove controller and CRDs
make undeploy
make uninstall
```

### Container Operations
```bash
# Build Docker image
make docker-build IMG=<registry>/controller:tag

# Push Docker image
make docker-push IMG=<registry>/controller:tag

# Multi-arch build and push
make docker-buildx IMG=<registry>/controller:tag
```

### Development Environment Setup
```bash
# Full development environment using Nushell scripts
# Sets up Kind cluster with all dependencies
./dot.nu setup \
  --dot-ai-tag "0.144.0" \
  --kubernetes-provider "kind" \
  --crossplane-enabled true \
  --kyverno-enabled true

# Teardown development environment
./dot.nu destroy
```

## Architecture

### Kubebuilder Structure
- **API Definitions**: `api/v1alpha1/`
  - `solution_types.go` - Solution CRD schema for resource tracking
  - `remediationpolicy_types.go` - RemediationPolicy CRD schema for event remediation
  - `resourcesyncconfig_types.go` - ResourceSyncConfig CRD schema for resource visibility
  - `capabilityscanconfig_types.go` - CapabilityScanConfig CRD schema for capability discovery
  - `gitknowledgesource_types.go` - GitKnowledgeSource CRD schema for documentation ingestion
- **Controllers**: `internal/controller/`
  - `solution_controller.go` - Manages Solution CRs and ownerReferences
  - `remediationpolicy_controller.go` - Watches events and triggers remediation
  - `resourcesync_controller.go` - Syncs resource metadata to MCP for semantic search
  - `resourcesync_mcp.go` - MCP client for resource sync
  - `resourcesync_debounce.go` - Debounce buffer for batching changes
  - `capabilityscan_controller.go` - Watches CRDs and triggers capability scans
  - `gitknowledgesource_controller.go` - Syncs Git repository documents to MCP
  - `gitknowledgesource_git.go` - Git client for cloning and change detection
  - `gitknowledgesource_mcp.go` - MCP client for document ingestion
- **Main Entry**: `cmd/main.go` - Sets up manager and registers all five controllers

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vfarcic/dot-ai-controller](https://github.com/vfarcic/dot-ai-controller) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
