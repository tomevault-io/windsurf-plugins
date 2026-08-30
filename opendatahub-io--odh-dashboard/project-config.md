---
trigger: always_on
description: This document provides guidance for AI agents and developers working on mod-arch-starter projects.
---

# AGENTS.md - Modular Architecture Starter

This document provides guidance for AI agents and developers working on mod-arch-starter projects.
This is a **starter template** for building modular architecture applications with a React frontend
and Go backend-for-frontend (BFF).

## Mandatory Development Flow

**CRITICAL: Never skip or reorder these stages. A PR that implements UI before the API contract will
be rejected.**

### 1. Contract First

Describe every capability in `bff/openapi/src/` (see [`data-registry.yaml`](bff/openapi/src/data-registry.yaml)
for this module's own contract). All request/response objects must be documented before coding.

### 2. BFF Stub Second

Add handlers and mock-returning services in `bff/internal/api` and `bff/internal/mocks` that satisfy
the new contract. Wire them in `bff/cmd/main.go` and expose feature flags/env vars as needed.

### 3. Frontend Third

Build UI routes inside `frontend/src/app` only after the stub endpoints respond with realistic
shapes. Consume generated types instead of duplicating schemas.

### 4. Production BFF Last

Replace mocks with Kubernetes-aware logic (repositories, clients, RBAC) before shipping. Verify the
BFF against a real cluster or the manifests in `manifests/`.

---

## Project Structure

```text
mod-arch-starter/
├── bff/                         # Go Backend-for-Frontend
│   ├── cmd/
│   │   └── main.go              # Application entrypoint and wiring
│   ├── openapi/
│   │   └── src/
│   │       ├── data-registry.yaml      # BFF's own OpenAPI spec (contract-first)
│   │       └── data-registry-api.yaml  # Vendored upstream Data Registry API contract
│   ├── internal/
│   │   ├── api/                 # HTTP handlers
│   │   ├── config/              # Configuration management
│   │   ├── constants/           # Shared constants
│   │   ├── helpers/             # Utility functions (k8s helpers)
│   │   ├── integrations/        # External service integrations
│   │   ├── mocks/               # Mock data and stub implementations
│   │   ├── models/              # DTOs and data models
│   │   └── repositories/        # Data access layer
│   ├── static/                  # Static assets served by BFF
│   ├── Makefile                 # Build and run commands
│   ├── go.mod                   # Go module definition
│   └── README.md                # BFF documentation
├── frontend/                    # React Frontend
│   ├── src/
│   │   ├── app/
│   │   │   ├── App.tsx          # Root application component
│   │   │   ├── AppRoutes.tsx    # Route definitions
│   │   │   ├── api/             # API client wrappers
│   │   │   ├── context/         # React context providers
│   │   │   ├── hooks/           # Custom React hooks
│   │   │   ├── pages/           # Page components
│   │   │   ├── standalone/      # Standalone mode components
│   │   │   └── utilities/       # Frontend utilities
│   │   ├── __mocks__/           # Jest mocks
│   │   ├── __tests__/           # Test files
│   │   └── images/              # Image assets
│   ├── config/
│   │   ├── webpack.common.js    # Shared webpack config
│   │   ├── webpack.dev.js       # Development webpack config
│   │   ├── webpack.prod.js      # Production webpack config
│   │   └── moduleFederation.js  # Module Federation config
│   ├── docs/                    # Frontend documentation
│   ├── package.json             # NPM dependencies and scripts
│   └── README.md                # Frontend documentation
├── manifests/                   # Kubernetes manifests
│   ├── base/                    # Base Kustomize resources
│   └── overlays/                # Environment-specific overlays
├── docs/                        # Project documentation
│   ├── install.md               # Installation guide
│   ├── local-deployment-guide.md
│   ├── local-deployment-guide-ui.md
│   └── kubeflow-development-guide.md
├── scripts/                     # Utility scripts
├── Dockerfile                   # Container image build
├── Makefile                     # Root-level make commands
└── README.md                    # Project overview
```

---

## Development Requirements

### Frontend

- **Node.js**: >= 22.0.0
- **npm**: >= 10.8.2

### BFF

- **Go**: >= 1.24.3

---

## Quick Start Commands

### Development Environment

```bash
# Install frontend dependencies
make dev-install-dependencies

# Start both BFF and frontend in development mode (mocked)
make dev-start

# Or start them separately:
make dev-bff         # BFF on port 4000 with mocks
make dev-frontend    # Frontend dev server
```

### Deployment Modes

```bash
# Standalone mode (default for local development)
make dev-start

# Kubeflow mode (connects to real cluster)
make dev-start-kubeflow

# Federated mode (micro-frontend with PatternFly theme)
make dev-start-federated
```

### Building

```bash
# Build everything
make build

# Build frontend only
make frontend-build

# Build BFF only
make bff-build

# Docker builds
make docker-build              # Kubeflow mode
make docker-build-standalone   # Standalone mode
make docker-build-federated    # Federated mode
```

### Testing

```bash
# Frontend tests (lint + type-check + unit + cypress)
cd frontend && npm run test

# BFF tests
cd bff && make lint && make test


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [opendatahub-io/odh-dashboard](https://github.com/opendatahub-io/odh-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-30 -->
