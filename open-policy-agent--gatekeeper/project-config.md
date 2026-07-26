---
trigger: always_on
description: Gatekeeper is a Kubernetes admission controller that provides policy-based governance for Kubernetes clusters using Open Policy Agent (OPA). It extends Kubernetes with **validation** and **mutation** capabilities through custom resources and webhooks. **Performance and security are the highest priorities** - admission controllers must minimize latency while maintaining strict security boundaries to protect cluster operations.
---

# Agent Instructions for OPA Gatekeeper

## Project Overview
Gatekeeper is a Kubernetes admission controller that provides policy-based governance for Kubernetes clusters using Open Policy Agent (OPA). It extends Kubernetes with **validation** and **mutation** capabilities through custom resources and webhooks. **Performance and security are the highest priorities** - admission controllers must minimize latency while maintaining strict security boundaries to protect cluster operations.

## Architecture & Repository Overview

**Core Components:**
- **Controller Manager**: Main controller managing constraints, templates, and policies
- **Admission Webhooks**: Validating and mutating admission controllers
- **Audit System**: Periodic compliance checking for existing resources
- **Mutation System**: Resource transformation capabilities
- **External Data**: Integration with external data sources
- **Gator CLI**: Policy testing and verification tool

**Repository Details:**
- **Size & Type:** Large-scale Go project (~165k lines) focused on Kubernetes admission control and policy governance with extensive test coverage
- **Primary Language:** Go 1.24+ with Go modules
- **Key Frameworks:** controller-runtime, OPA/Rego, Common Expression Language (CEL)
- **Container Technology:** Docker with multi-stage builds using buildx
- **Testing Stack:** Go test, BATS (Bash Automated Testing System), envtest for Kubernetes integration
- **CI/CD:** GitHub Actions with comprehensive matrix testing across Kubernetes versions

**Core Dependencies:**
- **Kubernetes API server**: Tested against latest three versions
- **controller-runtime**: Kubernetes controller framework
- **OPA Frameworks/Constraint**: Policy evaluation engine and constraint framework
- **cert-controller**: Automatic TLS certificate management and rotation

## Key Development Workflows

### Project Structure & Architecture

**Core Directory Layout:**
```
├── apis/                              # Kubernetes API definitions (CRDs)
│   ├── config/                        # Configuration CRDs (Config, Provider)
│   ├── connection/                    # Connection CRDs for exporting violations
│   ├── expansion/                     # Expansion template CRDs
│   ├── gvkmanifest/                   # GVK manifest CRDs
│   ├── mutations/                     # Mutation CRDs (Assign, AssignMetadata, ModifySet)
│   ├── status/                        # Status tracking CRDs
│   └── syncset/                       # Data synchronization CRDs
├── cmd/                               # Command line tools
│   ├── build/helmify/                 # Helm chart generation tool
│   └── gator/                         # Gator CLI tool for policy testing
├── main.go                            # main entry point
├── pkg/                               # Core business logic
│   ├── audit/                         # Audit functionality and violation tracking
│   ├── cachemanager/                  # Cache management for constraint evaluation
│   ├── controller/                    # Kubernetes controllers
│   │   ├── config/                    # Config controller
│   │   ├── configstatus/              # Config status controller
│   │   ├── connectionstatus/          # Connection status controller
│   │   ├── constraint/                # Constraint controller
│   │   ├── constraintstatus/          # Constraint status controller
│   │   ├── constrainttemplate/        # ConstraintTemplate controller
│   │   ├── constrainttemplatestatus/  # ConstraintTemplate status controller
│   │   ├── expansion/                 # Expansion controller
│   │   ├── expansionstatus/           # Expansion status controller
│   │   ├── export/                    # Export controller
│   │   ├── externaldata/              # External data controller
│   │   ├── mutators/                  # Mutators controller
│   │   ├── mutatorstatus/             # Mutator status controller
│   │   ├── sync/                      # Sync controller
│   │   └── syncset/                   # Syncset controller
│   ├── drivers/                       # Policy engine drivers (CEL)
│   ├── expansion/                     # Template expansion engine
│   ├── export/                        # Violation export functionality
│   ├── externaldata/                  # External data provider integration
│   ├── gator/                         # CLI implementation and testing utilities
│   ├── instrumentation/               # Metrics and observability
│   ├── logging/                       # Structured logging utilities
│   ├── metrics/                       # Prometheus metrics
│   ├── mutation/                      # Mutation engine and mutators
│   ├── operations/                    # Administrative operations
│   ├── readiness/                     # Health and readiness checks
│   ├── syncutil/                      # Data synchronization utilities

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [open-policy-agent/gatekeeper](https://github.com/open-policy-agent/gatekeeper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
