---
trigger: always_on
description: This document provides guidance for AI assistants working with the OpenCost codebase.
---

# AGENTS.md - OpenCost AI Agent Guide

This document provides guidance for AI assistants working with the OpenCost codebase.

## AI Assistant Behaviour

- Never include AI assistant session links or URLs (e.g. claude.ai) in commit messages or pull request bodies.

## Project Overview

OpenCost is an open source Kubernetes cost monitoring tool maintained by the Cloud Native Computing Foundation (CNCF). It provides real-time cost allocation, asset tracking, and cloud cost monitoring for Kubernetes clusters across multiple cloud providers.

**Key Features:**
- Real-time cost allocation by namespace, pod, controller, service, etc.
- Multi-cloud cost monitoring (AWS, Azure, GCP, Alibaba, Oracle, OTC, DigitalOcean, Scaleway)
- Dynamic on-demand pricing via cloud provider APIs
- CSV-based custom pricing for on-prem clusters
- MCP (Model Context Protocol) server for AI agent integration
- Prometheus metrics export

## Repository Structure

```
opencost/
├── cmd/costmodel/          # Main entry point (main.go)
├── core/                   # Core module (shared libraries)
│   └── pkg/
│       ├── clusters/       # Cluster management
│       ├── env/            # Environment variable utilities
│       ├── filter/         # Query filter implementations
│       ├── log/            # Structured logging
│       ├── model/          # Core data models
│       ├── opencost/       # OpenCost domain types (Allocation, Asset, CloudCost)
│       ├── storage/        # Storage abstractions
│       └── util/           # Utility packages
├── modules/
│   ├── collector-source/   # Custom metrics collector (alternative to Prometheus)
│   └── prometheus-source/  # Prometheus data source implementation
├── pkg/
│   ├── cloud/              # Cloud provider implementations
│   │   ├── aws/
│   │   ├── azure/
│   │   ├── gcp/
│   │   ├── alibaba/
│   │   ├── oracle/
│   │   ├── digitalocean/
│   │   ├── scaleway/
│   │   └── otc/            # Open Telekom Cloud
│   ├── cloudcost/          # Cloud cost processing pipeline
│   ├── clustercache/       # Kubernetes cluster caching
│   ├── cmd/costmodel/      # Cost model command implementation
│   ├── config/             # Configuration management
│   ├── costmodel/          # Core cost model logic and API handlers
│   ├── customcost/         # Custom cost plugin support
│   ├── env/                # Environment variable definitions
│   ├── mcp/                # MCP server implementation
│   └── metrics/            # Prometheus metrics
├── configs/                # Default pricing configurations
├── kubernetes/             # Kubernetes manifests (deprecated - use Helm)
├── protos/                 # Protocol buffer definitions
├── spec/                   # OpenCost specification
└── ui/                     # UI components (main UI in opencost/opencost-ui repo)
```

## Development Setup

### Prerequisites

- Go (see go.mod for the required version)
- Docker with `buildx` support
- [just](https://github.com/casey/just) - command runner
- [Tilt](https://tilt.dev/) - for local Kubernetes development
- Kubernetes cluster (local or remote)
- Prometheus instance

### Quick Start Commands

```bash
# Run all unit tests
just test

# Run tests for specific module
just test-core
just test-opencost
just test-prometheus-source
just test-collector-source

# Build local binary
just build-local

# Run locally (requires Prometheus and optionally Kubernetes access)
PROMETHEUS_SERVER_ENDPOINT="http://127.0.0.1:9080" go run ./cmd/costmodel/main.go

# Start development environment with Tilt
tilt up
```

### Running Locally Without Kubernetes

Set `PROMETHEUS_SERVER_ENDPOINT` to your Prometheus URL:

```bash
# Port-forward to Prometheus in your cluster
kubectl port-forward svc/prometheus-server 9080:80

# Run OpenCost
PROMETHEUS_SERVER_ENDPOINT="http://127.0.0.1:9080" go run ./cmd/costmodel/main.go
```

### Running Integration Tests

```bash
INTEGRATION=true just test-integration
```

## Build Commands

```bash
# Build local binary
just build-local

# Build multi-arch binaries
just build-binary <version>

# Build and push Docker image
just build <image-tag> <release-version>

# Validate protobuf definitions
just validate-protobuf
```

## Key Environment Variables

### Core Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `PROMETHEUS_SERVER_ENDPOINT` | (required) | Prometheus server URL |
| `API_PORT` | `9003` | OpenCost API port |
| `CLUSTER_ID` | auto-detected | Cluster identifier |
| `CONFIG_PATH` | `/var/configs` | Configuration directory |

### MCP Server

| Variable | Default | Description |
|----------|---------|-------------|
| `MCP_SERVER_ENABLED` | `false` | Enable MCP server |
| `MCP_HTTP_PORT` | `8081` | MCP server HTTP port |

### Admin Auth

| Variable | Default | Description |
|----------|---------|-------------|
| `ADMIN_TOKEN` | (unset) | Bearer token for admin endpoints. If unset, those endpoints return HTTP 503. |

Admin-protected endpoints:

- `POST /serviceKey`
- `GET /helmValues`
- `GET /cloudCost/rebuild`, `GET /cloudCost/repair`
- `GET /cloud/config/export`, `GET /cloud/config/enable`, `GET /cloud/config/disable`, `GET /cloud/config/delete`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [opencost/opencost](https://github.com/opencost/opencost) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
