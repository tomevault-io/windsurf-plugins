---
trigger: always_on
description: Cincinnati is an update protocol for automatic updates, primarily used by OpenShift. It represents transitions between releases as a directed acyclic graph (DAG) and allows clients to perform automatic updates between releases.
---

# AGENTS.md

## Project Overview

Cincinnati is an update protocol for automatic updates, primarily used by OpenShift. It represents transitions between releases as a directed acyclic graph (DAG) and allows clients to perform automatic updates between releases.

## Build System

- **Language**: Rust (Rust 2018 edition)
- **Command Runner**: Use `just` for common development tasks

### Common Development Commands

```bash
# Run all tests with formatting checks
just test

# Start services locally
just run-graph-builder    # Start graph-builder service (port 8080)
just run-policy-engine    # Start policy-engine service (port 8081)
just run-metadata-helper # Start metadata-helper service

# Run CI test suite and e2e tests
just run-ci-tests
just run-e2e-test-only   # Run only end-to-end tests

cargo build --release    # Release build
```

## Architecture Overview

Cincinnati follows a microservices architecture with two main services:

### Core Services

1. **Graph Builder** (`graph-builder/`) - Scrapes releases from container registries and builds update graphs
   - Serves `/graph` endpoint on port 8080
   - Integrates with Docker registries, GitHub, and OpenShift metadata
   - Plugin-based architecture for extensible release processing

2. **Policy Engine** (`policy-engine/`) - Applies policies to update graphs and serves filtered graphs
   - Serves `/graph` endpoint on port 8081 with OpenAPI specification
   - Filters graphs based on client parameters (channel, architecture)
   - Enforces rollout policies and business logic

### Key Libraries

- **cincinnati/** - Core library with graph types and plugin infrastructure
- **commons/** - Shared utilities (web framework, metrics, logging, HTTP client)
- **quay/** - Container registry client for Docker v2 API
- **metadata-helper/** - Metadata processing service for handling signatures and additional data

### Plugin System

- Plugin interface defined in `cincinnati/src/plugins/interface.proto`
- External web-based plugins for custom logic
- Internal built-in plugins for common operations (filtering, metadata parsing, graph manipulation)

### Policy Engine Plugins

The policy engine uses a plugin-based architecture to process and filter update graphs. All plugins are internal Rust-based implementations that run in sequence to transform the graph. Here's a detailed breakdown of each plugin:

#### Core Filtering Plugins

**1. arch-filter** (`cincinnati/src/plugins/internal/arch_filter.rs`)
- **Purpose**: Filters graph releases by architecture (e.g., `amd64`, `arm64`)
- **How it works**: Reads `arch` parameter from client requests and removes releases that don't match the requested architecture
- **Metadata processing**: Uses `io.openshift.upgrades.graph.release.arch` metadata key to identify compatible architectures
- **Additional features**: Strips architecture suffixes from version strings (e.g., `4.1.0+amd64` → `4.1.0`)
- **Configuration**: Default key prefix `io.openshift.upgrades.graph`, key suffix `release.arch`
- **Use case**: Ensures clients only see releases compatible with their architecture

**2. channel-filter** (`cincinnati/src/plugins/internal/channel_filter.rs`)
- **Purpose**: Filters graph releases by release channel (e.g., `stable-4.2`, `fast-4.3`)
- **How it works**: Reads `channel` parameter from client requests and removes releases not assigned to that channel
- **Metadata processing**: Uses `io.openshift.upgrades.graph.release.channels` metadata key with comma-separated channel values
- **Validation**: Channel names must match regex `^[0-9a-z\-\.]+$`
- **Use case**: Allows different risk tolerance levels (stable vs fast channels) and version streams

#### Graph Manipulation Plugins

**3. edge-add-remove** (`cincinnati/src/plugins/internal/edge_add_remove.rs`)
- **Purpose**: Dynamically adds and removes edges based on release metadata
- **How it works**: Processes metadata labels with syntax `<prefix>.(previous|next).(add|remove)=(version1,version2,...)`
- **Operations**:
  - `*.previous.add`: Add edges from current release to specified previous releases
  - `*.next.add`: Add edges from current release to specified next releases
  - `*.previous.remove`: Remove edges from current release to specified previous releases
  - `*.next.remove`: Remove edges from current release to specified next releases
- **Advanced features**:
  - `*.previous.remove_regex`: Remove edges using regex patterns
  - Conditional edges support for complex update logic with risks and matching rules
- **Processing order**: Add operations first, then remove operations (removes take precedence)
- **Use case**: Implements custom update paths, blocks problematic upgrades, creates conditional updates

**4. node-remove** (`cincinnati/src/plugins/internal/node_remove.rs`)
- **Purpose**: Removes entire releases from the graph based on metadata flags
- **How it works**: Removes releases where `io.openshift.upgrades.graph.release.remove` metadata equals `"true"`
- **Use case**: Allows marking specific releases as unavailable or deprecated

#### Data Fetching Plugins


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openshift/cincinnati](https://github.com/openshift/cincinnati) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
