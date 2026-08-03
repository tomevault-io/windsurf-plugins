---
trigger: always_on
description: Enables secure traffic for Knative Serving + Eventing.
---

# Project AGENTS.md for OpenShift Serverless Operator

This AGENTS.md file provides comprehensive guidance for AI assistants and coding agents (like Claude, Gemini, Cursor, and others) to work with this codebase.

This repository contains the **Red Hat OpenShift Serverless Operator**, which provides a collection of APIs to support deploying and serving serverless applications and functions on OpenShift. It manages the lifecycle of Knative Serving, Knative Eventing, and Knative Kafka components.

## Project Overview

The OpenShift Serverless Operator is an **Operator Lifecycle Manager (OLM)**-based operator that:
- Deploys and manages Knative Serving (serverless application runtime)
- Deploys and manages Knative Eventing (event-driven architecture)
- Deploys and manages Knative Kafka (Kafka integration for eventing)
- Integrates with OpenShift Service Mesh, Strimzi (Kafka operator), and distributed tracing
- Provides OpenShift-specific customizations and downstream patches to upstream Knative

## Project Structure and Repository Layout

```
serverless-operator/
├── hack/                       # Build, install, and test automation scripts
│   ├── install.sh             # Main installation script
│   ├── images.sh              # Image building script
│   ├── dev.sh                 # Development mode setup
│   ├── generate/              # Code generation scripts
│   └── lib/                   # Shared shell libraries
├── knative-operator/          # Upstream Knative operator vendored code
├── openshift-knative-operator/ # OpenShift-specific operator wrapper
├── pkg/                       # Go packages
│   ├── client/                # Kubernetes client helpers
│   ├── common/                # Common utilities
│   └── istio/                 # Service Mesh integration
├── test/                      # Test code and helpers
├── olm-catalog/               # OLM bundle metadata
│   └── serverless-operator/
│       └── project.yaml       # Version and dependency configuration
├── templates/                 # Templates for generated files
├── must-gather/               # Must-gather support for debugging
├── docs/                      # Documentation
├── .tekton/                   # Tekton CI/CD pipelines
├── .github/                   # GitHub Actions workflows
├── Makefile                   # Build targets and automation
└── README.md                  # Project documentation
```

## Upstream Relationships

This project maintains downstream forks and integrations with several upstream projects:
- **Knative Serving** - https://github.com/openshift-knative/serving (fork of knative.dev/serving)
- **Knative Eventing** - https://github.com/openshift-knative/eventing (fork of knative.dev/eventing)
- **Knative Kafka** - Integration with Apache Kafka via Knative Eventing
- **Strimzi** - Kafka operator for Kubernetes/OpenShift
- **Istio/Service Mesh** - For secure traffic and gateway management

## Development Environment Setup

### Prerequisites

Before working with this repository, ensure you have:

- **Container runtime**: `podman` (aliased to `docker`) or `docker` (17.05+)
- **Shell**: `bash` (4.0.0+)
- **Build tools**: `make`
- **Kubernetes tools**: `helm`
- **OpenShift cluster**: CRC (recommended for local development) or any OpenShift cluster
- **Go**: Version specified in `go.mod` (check current requirement)

### Recommended CRC Configuration

For local development with CodeReady Containers (CRC):

```bash
crc start --cpus=6 --memory 16384
```

This configuration has been tested to work with operator E2E tests.

### Environment Variables

- **`DOCKER_REPO_OVERRIDE`**: Set to your container registry (e.g., `quay.io/username`) for building custom images
- **`ON_CLUSTER_BUILDS`**: Set to `true` to build images on-cluster using OpenShift Build
- **`GOPATH`**: Required for upstream integration tests (must contain knative.dev/serving and knative.dev/eventing)

## Building

### Quick Start

```bash
# Format code, tidy dependencies, and build images
make images

# Build and push to your registry
export DOCKER_REPO_OVERRIDE=quay.io/username
make images
```

### Build Targets

- **`make images`**: Build and push all container images
- **`make dev`**: Deploy operator without Knative components (development mode)
- **`make install`**: Deploy operator with Serving and Eventing
- **`make install-all`**: Deploy operator with Serving, Eventing, and Kafka
- **`make install-serving`**: Deploy operator with only Knative Serving
- **`make install-eventing`**: Deploy operator with only Knative Eventing
- **`make install-kafka`**: Deploy operator with Knative Kafka (requires Strimzi)

### On-Cluster Builds

Instead of building locally with podman/docker:

```bash
ON_CLUSTER_BUILDS=true make images
# Images will be at: image-registry.openshift-image-registry.svc:5000/openshift-serverless-builds/<image_name>

# Install using those images
DOCKER_REPO_OVERRIDE=image-registry.openshift-image-registry.svc:5000/openshift-serverless-builds make install
```

## Testing

### Quick Local Testing

Run the most relevant tests (same as CI):

```bash
crc start --cpus=6 --memory 16384
export DOCKER_REPO_OVERRIDE=quay.io/username
make images test-operator
```

### Test Targets

#### Operator Tests
- **`make test-unit`**: Run unit tests

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openshift-knative/serverless-operator](https://github.com/openshift-knative/serverless-operator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
