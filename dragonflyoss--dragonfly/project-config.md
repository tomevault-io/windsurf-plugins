---
trigger: always_on
description: Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.
---

# Dragonfly P2P File Distribution System

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Project Overview

Dragonfly is a **CNCF Graduated** project that delivers efficient, stable, and secure data distribution and
acceleration powered by P2P technology. It provides an optional content-addressable filesystem that accelerates
OCI container launch. The project targets cloud-native architectures and large-scale delivery of files, container
images, OCI artifacts, AI/ML models, caches, logs, and dependencies.

- **Website**: <https://d7y.io>
- **Module path**: `d7y.io/dragonfly/v2`
- **Go version**: 1.25.5
- **Current version**: v2.4.0
- **License**: Apache 2.0

## Architecture

Dragonfly consists of the following components:

### Components in this repository

| Component     | Binary      | Default Ports       | Description                                                    |
| ------------- | ----------- | ------------------- | -------------------------------------------------------------- |
| **Manager**   | `manager`   | gRPC :65003, REST :8080 | Cluster management, peer lifecycle, dynamic configuration, web console |
| **Scheduler** | `scheduler` | gRPC :8002          | Optimizes P2P download scheduling and task routing             |

### Submodule components

| Submodule              | Repository                          | Description                                           |
| ---------------------- | ----------------------------------- | ----------------------------------------------------- |
| `client/`              | dragonflyoss/client                 | Rust-based dfdaemon (dfget, dfcache, dfstore, proxy)  |
| `manager/console/`     | dragonflyoss/console                | Node.js/React web console frontend for manager        |
| `deploy/helm-charts/`  | dragonflyoss/helm-charts            | Kubernetes Helm charts for deploying Dragonfly        |

### Data Flow

```
Client (dfdaemon/dfget) → Scheduler → Manager
                              ↓
                     Seed Peer / Origin Server
```

- **Manager**: stores cluster configuration in MySQL/PostgreSQL and caches in Redis; exposes both gRPC and REST APIs
- **Scheduler**: stateless gRPC service that manages the P2P DAG (task graph); stores transient state in Redis
- **Client (dfdaemon)**: runs on each node; intercepts HTTP/HTTPS requests and coordinates peer-to-peer downloads

## Repository Structure

### Key Directories

```
cmd/
  manager/          # Manager binary entry point (main.go)
  scheduler/        # Scheduler binary entry point (main.go)
  dependency/       # Shared CLI base options (base.Options) and plugin/version commands

manager/            # Manager service implementation
  config/           # Configuration structs and defaults (GRPC :65003, REST :8080)
  handlers/         # Gin HTTP request handlers
  router/           # HTTP router setup
  rpcserver/        # gRPC server implementation
  service/          # Business logic
  models/           # GORM database models
  database/         # Database initialization (MySQL, MariaDB, PostgreSQL)
  middlewares/      # Authentication, CORS, rate-limiting middleware
  job/              # Async job processing (dragonflyoss/machinery)
  metrics/          # Prometheus metrics
  searcher/         # Scheduler cluster selection logic
  auth/             # JWT authentication
  permission/       # Casbin RBAC authorization
  gc/               # Garbage collection

scheduler/          # Scheduler service implementation
  config/           # Configuration structs and defaults (port :8002)
  scheduling/       # Core P2P scheduling algorithms and evaluators
  resource/         # In-memory peer/task/host resource management
  rpcserver/        # gRPC server implementation
  service/          # Business logic
  announcer/        # Announces scheduler to manager
  job/              # Task job handling
  metrics/          # Prometheus metrics

pkg/                # Shared libraries
  auth/             # Authentication utilities
  balancer/         # gRPC load balancing
  cache/            # Cache utilities
  container/        # Container helpers
  dfnet/            # Network type definitions
  dfpath/           # Standard Dragonfly paths (/var/lib/dragonfly, /var/log/dragonfly)
  digest/           # Content digest (SHA-256, MD5, etc.)
  gc/               # Generic garbage collection framework
  graph/            # DAG implementation for task graphs
  idgen/            # ID generation for tasks, peers, hosts
  math/             # Math utilities
  net/              # Network utilities (IP, FQDN, HTTP)
  os/               # OS utilities
  redis/            # Redis client wrapper
  rpc/              # gRPC client wrappers (manager, scheduler, dfdaemon)
  slices/           # Generic slice utilities
  strings/          # String utilities
  structure/        # Data structure utilities
  time/             # Time utilities
  types/            # Shared type definitions

internal/           # Internal packages (not importable externally)
  dferrors/         # Dragonfly error types

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dragonflyoss/dragonfly](https://github.com/dragonflyoss/dragonfly) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
