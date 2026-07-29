---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Kubernetes operator for [Restate](https://restate.dev/), written in Rust. The operator manages three main Custom Resource Definitions (CRDs):
- `RestateCluster` - Manages Restate server clusters with StatefulSets
- `RestateDeployment` - Manages Restate SDK service deployments (similar to Kubernetes Deployments but with Restate-specific versioning)
- `RestateCloudEnvironment` - Integrates with Restate Cloud environments

The operator enforces network isolation by default, handles service versioning/draining, and supports both ReplicaSet and Knative Serving deployment modes.

## Development Environment

Required Rust version: 1.80+

## Common Development Commands

### Building and Running

```bash
# Build the operator
just build

# Build with specific features, architecture, or libc
just build features="telemetry"
just build arch="amd64"
just build libc="musl"

# Build Docker image
just docker

# Run the operator locally (requires OPERATOR_NAMESPACE env var)
OPERATOR_NAMESPACE=restate-operator RUST_LOG=info cargo run
```

### Code Generation

The operator uses code generation for CRDs and Pkl schemas:

```bash
# Generate CRD YAML files from Rust code
just generate

# Generate Pkl schema templates
just generate-pkl

# Generate example YAML from Pkl templates
just generate-examples
```

**Important**: When modifying CRD structs in `src/resources/*.rs`, run `just generate` to regenerate the YAML files in `crd/`.

### Testing

```bash
# Run all tests
cargo test

# Run a specific test
cargo test <test_name>
```

### Installing CRDs

```bash
# Install CRDs into the current Kubernetes cluster
just install-crds

# Note: Uses kubectl create (not apply), so remove existing CRDs first if needed
```

### Code Quality

```bash
# Format code
just fmt

# Run clippy linter
just lint

# Run both formatting and linting
just check
```

## Architecture

### Controller Structure

The operator runs three concurrent controllers (see `src/main.rs:94-105`):
1. **RestateCluster Controller** (`src/controllers/restatecluster/`) - Manages StatefulSets, Services, NetworkPolicies for Restate server clusters
2. **RestateCloudEnvironment Controller** (`src/controllers/restatecloudenvironment/`) - Manages tunnel deployments for Restate Cloud integration
3. **RestateDeployment Controller** (`src/controllers/restatedeployment/`) - Manages service deployments with versioning and draining

Each controller implements a reconciliation loop using the Kube-rs runtime.

### CRD Definitions

CRD structs are defined in `src/resources/`:
- `restateclusters.rs` - RestateCluster CRD
- `restatedeployments.rs` - RestateDeployment CRD
- `restatecloudenvironments.rs` - RestateCloudEnvironment CRD
- `knative.rs` - Knative Serving resource definitions

Additional AWS-specific resources:
- `podidentityassociations.rs` - AWS EKS Pod Identity
- `securitygrouppolicies.rs` - AWS Security Groups for Pods
- `secretproviderclasses.rs` - CSI Secret Provider

### Deployment Modes

RestateDeployment supports two modes (see `src/resources/restatedeployments.rs:20-28`):
- **ReplicaSet** (default): Traditional Kubernetes Deployment pattern
- **Knative**: Knative Serving with Configuration-per-tag architecture

**Key Concept - Tag-Based Identity**:
- A Restate deployment is immutable once registered
- The `tag` field determines deployment identity
- Same tag = in-place update (new Knative Revision within same Restate deployment)
- Changed tag = versioned update (new Restate deployment ID)
- No tag = template hash as tag (every template change creates new deployment)

### Network Isolation

By default, RestateCluster enforces network isolation via NetworkPolicies:
- Allows peer-to-peer traffic between Restate pods
- Allows operator access to admin API
- Allows egress to public internet and coredns
- Allows traffic to namespaces labeled with `allow.restate.dev/<cluster-name>`
- Denies all other traffic

Disable with `spec.security.disableNetworkPolicies: true`

### State Management

Controllers share state via `src/controllers/mod.rs:State`:
- Diagnostics (last reconciliation events)
- Prometheus metrics registry
- Operator configuration (namespace, labels, AWS settings)

## Kubernetes Integration

### kubectl Usage

Always use `kubectl apply --server-side` for CRDs to avoid client-side validation issues.

### Local Image Support for Knative Deployments

When developing with Knative deployment mode using locally built images:

**Problem**: Knative Serving's revision-controller resolves image tags to digests before creating pods. This fails for local-only images that don't exist in a remote registry.

**Solution**: Use the `dev.local` image prefix

```bash
# Build your service image
docker build -t ghcr.io/restatedev/restate-operator/greeter:latest \
  examples/services/greeter/

# Tag with dev.local prefix for Knative compatibility
docker tag ghcr.io/restatedev/restate-operator/greeter:latest \
  dev.local/restatedev/restate-operator/greeter:latest
```

**Why it works**:
- `dev.local` is in Knative's default `registries-skipping-tag-resolving` list

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [restatedev/restate-operator](https://github.com/restatedev/restate-operator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
