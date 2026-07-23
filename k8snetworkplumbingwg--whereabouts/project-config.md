---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Whereabouts is an IP Address Management (IPAM) CNI plugin for Kubernetes that assigns IP addresses cluster-wide. Unlike host-local which only works per-node, Whereabouts provides cluster-wide IP assignment by tracking allocations in either Kubernetes Custom Resources (CRDs) or etcd.

The plugin assigns IPs from a specified range (CIDR notation), always allocating the lowest available address. It supports both IPv4 and IPv6, and is commonly used with Multus CNI for multi-network configurations.

## Common Development Commands

### Building
```bash
# Build the CNI plugin binary
./hack/build-go.sh

# Build Docker image
make docker-build
# or with custom registry/tag:
IMAGE_REGISTRY=myregistry IMAGE_TAG=v1.0 make docker-build
```

### Testing
```bash
# Run unit tests (builds, installs tools, runs tests with coverage)
make test

# Run unit tests without static check (faster for iteration)
make test-skip-static

# Run unit tests for a single package
go test -v ./pkg/allocate/

# Run unit tests without cache (force re-run)
go test -count=1 -v ./pkg/storage/

# Run e2e tests locally
# 1. Install godotenv: go install github.com/joho/godotenv/cmd/godotenv@latest
# 2. Create kind cluster: make kind
# 3. Create e2e/.env with: KUBECONFIG=$HOME/.kube/config
# 4. Run tests from e2e directory:
cd e2e && godotenv -f .env go test -v . -timeout=1h
```

### Code Generation
```bash
# Verify generated code is up to date
./hack/verify-codegen.sh

# Update generated code (clientsets, informers, listers)
./hack/update-codegen.sh

# Regenerate API code and clean up artifacts
make generate-api
```

### Dependencies
```bash
# Update Go dependencies
make update-deps
# This runs: go mod tidy && go mod verify
```

### Local Development Cluster
```bash
# Create a kind cluster with whereabouts installed (default: 2 worker nodes)
make kind

# Create with specific number of worker nodes
make kind COMPUTE_NODES=3
```

## Architecture

### Core Components

**CNI Plugin Entry Point** (`cmd/whereabouts.go`)
- Implements the CNI specification interface (ADD, DEL, CHECK, VERSION commands)
- `cmdAddFunc`: Allocates an IP address when a pod interface is created
- `cmdDelFunc`: Releases an IP address when a pod interface is deleted
- Loads IPAM configuration from stdin and creates a Kubernetes IPAM client

**IP Allocation Logic** (`pkg/allocate/`)
- `AssignIP`: Main allocation function that assigns IPs from a range
- Iterates through the IP range to find the lowest available IP
- Checks for existing allocations by podRef and ifName (idempotent behavior)
- Respects exclude ranges and avoids IPs ending in `.0`

**Storage Backend** (`pkg/storage/`)
- Abstract interface for IP pool management (currently Kubernetes CRDs primary, etcd legacy)
- `Store` interface: GetIPPool, GetOverlappingRangeStore, Status, Close
- `IPPool` interface: Allocations, Update
- Kubernetes implementation in `pkg/storage/kubernetes/` handles:
  - IPPool CRDs for tracking allocations per IP range
  - OverlappingRangeIPReservation CRDs for cross-range IP uniqueness
  - Retry logic with exponential backoff for concurrent updates

**Configuration** (`pkg/config/`)
- Parses CNI configuration from JSON
- Supports both inline config and flat-file configuration (`/etc/cni/net.d/whereabouts.d/whereabouts.conf`)
- Handles range specifications: single CIDR, range with start/end, ipRanges for multi-IP/dual-stack

**IP Reconciler** (`pkg/reconciler/`)
- Runs as a Kubernetes CronJob to clean up stranded IP allocations
- Compares allocated IPs against running pods
- Deallocates IPs for pods that no longer exist (handles node failures, force deletes)
- Reconciler cron expression configurable via ConfigMap or environment variable

**Controllers**
- **IP Reconciliation Controller** (`cmd/controlloop/`): Watches pods and reconciles IP allocations
- **Node Slice Controller** (`cmd/nodeslicecontroller/`): Experimental "Fast IPAM" feature that pre-allocates IP slices per node to reduce contention

### Custom Resource Definitions

**IPPool** (`pkg/api/whereabouts.cni.cncf.io/v1alpha1/ippool_types.go`)
- Stores IP allocations for a specific range
- Key format: `<namespace>-<network-name>-<normalized-range>`
- Contains array of IPReservation entries with IP, PodRef, ContainerID, IfName

**OverlappingRangeIPReservation** (`pkg/api/whereabouts.cni.cncf.io/v1alpha1/overlappingrangeipreservation_types.go`)
- Ensures IP uniqueness across overlapping ranges when `enable_overlapping_ranges: true`
- Prevents same IP from being allocated in different ranges that overlap

**NodeSlicePool** (`pkg/api/whereabouts.cni.cncf.io/v1alpha1/nodeslicepool_types.go`)
- Experimental: Tracks node-specific IP slice allocations for Fast IPAM
- Enabled by setting `node_slice_size` in IPAM config

### Storage Backend Flow

1. **IP Allocation (ADD command)**:
   - Load IPAM config from CNI stdin
   - Create Kubernetes IPAM client with pod context
   - Get or create IPPool CR for the range
   - Check if podRef+ifName already allocated (idempotent)
   - Find lowest available IP using `IterateForAssignment`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [k8snetworkplumbingwg/whereabouts](https://github.com/k8snetworkplumbingwg/whereabouts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
