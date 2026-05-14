---
trigger: always_on
description: This rule provides comprehensive best practices for developing and maintaining Kubernetes applications and infrastructure, covering coding standards, security, performance, testing, and deployment.
---

# Kubernetes Development and Operations Best Practices

This document outlines a collection of guidelines, style suggestions, and tips for writing code and managing infrastructure within the Kubernetes ecosystem. It emphasizes clarity, maintainability, security, and performance.

## 1. Code Organization and Structure

### 1.1 Directory Structure

- **Root Level:**
  - `cmd/`:  Main application entry points. Each subdirectory represents a separate command-line tool or service.
  - `pkg/`: Reusable libraries and components that can be imported by other projects.
  - `internal/`: Private code that should not be imported by external projects.  Enforces encapsulation.
  - `api/`:  API definitions, including protobuf files and OpenAPI specifications.
  - `config/`: Configuration files, such as YAML manifests, Kustomize configurations, and Helm charts.
  - `scripts/`: Utility scripts for building, testing, and deploying the application.
  - `docs/`: Documentation for the project.
  - `examples/`: Example usage of the library or application.
  - `vendor/`: (If using `go modules` without external dependency management) Contains vendored dependencies.  Generally discouraged in modern Go with `go modules`.
- **Component-Specific Directories:** Inside `pkg/` or `internal/`, organize code by component or module. Each component should have its own directory with clear separation of concerns.

Example:


my-kubernetes-project/
├── cmd/
│   └── controller/
│       └── main.go
├── pkg/
│   └── api/
│       ├── types.go
│   └── controller/
│       ├── controller.go
│       ├── reconciler.go
│   └── util/
│       └── util.go
├── internal/
│   └── admission/
│       └── webhook.go
├── config/
│   ├── deploy/
│   │   └── deployment.yaml
│   └── kustomize/
│       ├── base/
│       │   ├── kustomization.yaml
│       │   └── ...
│       └── overlays/
│           ├── dev/
│           │   ├── kustomization.yaml
│           │   └── ...
│           └── prod/
│               ├── kustomization.yaml
│               └── ...
├── scripts/
│   └── build.sh
├── docs/
│   └── architecture.md
└── go.mod


### 1.2 File Naming Conventions

- **Go Files:** Use lowercase with underscores (e.g., `my_controller.go`).
- **YAML Files:** Use lowercase with dashes (e.g., `deployment.yaml`).
- **Configuration Files:** Be descriptive and consistent (e.g., `config.yaml`, `kustomization.yaml`).
- **Test Files:** Follow the standard Go convention: `*_test.go` (e.g., `my_controller_test.go`).

### 1.3 Module Organization (Go)

- **Packages:**  Organize code into meaningful packages that represent logical units of functionality.
- **Internal Packages:** Use `internal/` directories to create packages that are only visible within the project.
- **Interfaces:** Define interfaces to abstract dependencies and promote testability.

### 1.4 Component Architecture

- **Microservices:** Design applications as a collection of loosely coupled microservices.
- **Separation of Concerns:** Each component should have a single responsibility and well-defined interfaces.
- **API Gateway:** Use an API gateway to handle routing, authentication, and rate limiting for external requests.
- **Service Mesh:** Consider using a service mesh (e.g., Istio, Linkerd) to manage inter-service communication, observability, and security.

### 1.5 Code Splitting Strategies

- **Feature-Based Splitting:** Group code by feature or functionality.
- **Layer-Based Splitting:** Separate code into layers, such as data access, business logic, and presentation.
- **Component-Based Splitting:** Divide code into reusable components that can be shared across multiple projects.

## 2. Common Patterns and Anti-Patterns

### 2.1 Design Patterns

- **Controller Pattern:** Implement controllers to reconcile the desired state of Kubernetes resources with the actual state.
- **Operator Pattern:** Extend the Kubernetes API with custom resources and controllers to automate complex application management tasks.
- **Sidecar Pattern:** Deploy a sidecar container alongside the main application container to provide supporting functionality, such as logging, monitoring, or security.
- **Ambassador Pattern:**  Use an ambassador container to proxy network traffic to the main application container, providing features such as load balancing, routing, and authentication.
- **Adapter Pattern:** Translate requests from one interface to another, allowing different components to work together.
- **Singleton Pattern:** Implement a singleton pattern for managing global resources, such as database connections or configuration settings. Be extremely cautious, as this can hurt testability and introduce implicit dependencies.

### 2.2 Recommended Approaches for Common Tasks

- **Resource Management:** Use Kubernetes resource requests and limits to ensure that applications have sufficient resources and prevent resource contention.
- **Configuration Management:** Use ConfigMaps and Secrets to manage configuration data and sensitive information separately from the application code.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rustic-ai/codeprism](https://github.com/rustic-ai/codeprism) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
