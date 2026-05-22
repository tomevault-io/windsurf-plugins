---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with
code in this repository.

## Project Overview

Milo is a "business operating system" for product-led B2B companies built on
Kubernetes. It provides a comprehensive system of record for organizations,
projects, users, audit logs, and other business operations through a
Kubernetes-like API experience.

## Core Architecture

Milo is built on the Kubernetes API server library and provides its own
dedicated API server with custom resource definitions (CRDs). The system follows
standard Kubernetes controller patterns with multi-cluster capabilities.

### Key Components

- **API Server**: Built on k8s.io/apiserver library, provides the main API
  interface
- **Controllers**: Standard Kubernetes controllers that reconcile custom
  resources
- **Multi-cluster Runtime**: Cross-cluster resource management using
  sigs.k8s.io/multicluster-runtime
- **Provider Framework**: Extensible integration system for third-party services

### Resource Hierarchy

1. **Organizations** (cluster-scoped): Top-level business entities with types
   (Personal/Standard/Business)
2. **Projects** (namespaced): Resource organization units within organizations
3. **Users & Groups** (IAM): Identity management with RBAC
4. **OrganizationMemberships**: Link users to organizations
5. **ProjectControlPlanes**: Cross-cluster project provisioning

### API Structure

Custom resources are organized under three main API groups:
- `resourcemanager.miloapis.com/v1alpha1`: Organizations, Projects,
  OrganizationMemberships
- `iam.miloapis.com/v1alpha1`: Users, Groups, Roles, PolicyBindings
- `infrastructure.miloapis.com/v1alpha1`: ProjectControlPlanes

## Development Commands

### Code Generation
```bash
# Generate deepcopy, CRDs, webhooks, and RBAC
task generate

# Generate API documentation
task api-docs
```

### Development Certificates
```bash
# Generate self-signed certs for local webhook development
task generate-dev-certs
```

### Build and Run
```bash
# Build container image (preferred approach)
task dev:build

# For local development, use the complete development setup
task dev:setup    # Sets up test infrastructure + deploys Milo

# Access Milo API server (instead of running locally)
task kubectl -- get organizations
task kubectl -- apply -f config/samples/resourcemanager/v1alpha1/organization.yaml
```

### Testing
```bash
# Run unit tests
task test:unit

# Run end-to-end tests with Chainsaw
task test:end-to-end

# Individual test directories contain chainsaw-test.yaml files
# Use task commands instead of running chainsaw directly
```

### Development Environment
```bash
# PREFERRED: Use complete Task-based development setup
task dev:setup    # Creates test infrastructure + deploys Milo

# Alternative: Start local development dependencies only
docker-compose up -d    # Start OpenFGA, Zitadel, Mailhog

# Use Task commands for deployment instead of direct kubectl
task kubectl -- apply -k config/dev
```

### Test Infrastructure
```bash
# Deploy complete test infrastructure with etcd, API server, and controller manager
task dev:setup

# Deploy test infrastructure with OpenFGA authorization provider
task dev:setup:openfga

# Deploy only (assumes cluster already exists)
task dev:deploy           # Standard deployment
task dev:deploy:openfga   # With OpenFGA provider

# Switch to use Milo API server
export KUBECONFIG=.milo/kubeconfig

# Then test with Task wrapper (automatically uses .milo/kubeconfig):
task kubectl -- get organizations
task kubectl -- get projects  
task kubectl -- get users

# Task kubectl wrapper handles kubeconfig automatically
```

### Test Infrastructure Management
The Milo taskfile includes remote access to the `datum-cloud/test-infra` repository for managing test environments. This uses Go Task's experimental remote taskfiles feature.

```bash
# Direct access to test-infra commands (requires TASK_X_REMOTE_TASKFILES=1)
task test-infra:cluster-up        # Start a new test cluster
task test-infra:cluster-down      # Tear down the cluster
task test-infra:install-tools     # Install required tools
task test-infra:status            # Check infrastructure status

# Convenience wrapper commands (automatically set environment variable)
task test-infra-cluster           # Start cluster (wrapper)
task test-infra-cluster-down      # Stop cluster (wrapper)
task test-infra-install-tools     # Install tools (wrapper)
task test-infra-status            # Status check (wrapper)
task test-infra-clean             # Clean up resources (wrapper)
```

**Note**: Remote taskfiles are an experimental feature. The environment variable `TASK_X_REMOTE_TASKFILES=1` is automatically set in the taskfile configuration.

### Authentication
The test-infra deployment includes pre-configured authentication:
- **Kubeconfig**: `.milo/kubeconfig` (standard location, committed to repo)
- **Admin Token**: `test-admin-token` (system:masters group)
- **User Token**: `test-user-token` (system:authenticated group)
- **API Endpoint**: `http://localhost:8080` (via Envoy Gateway)

## Code Structure

### Controllers
- Located in `internal/controllers/`
- Follow standard controller-runtime patterns
- Support both single-cluster and multi-cluster operations

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [milo-os/milo](https://github.com/milo-os/milo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
