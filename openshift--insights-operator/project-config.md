---
trigger: always_on
description: Provides configuration and status for on-demand Insights data gathering (TechPreview):
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Common Development Commands

### Building and Testing
- `make build` - Compiles the insights-operator binary to `./bin/insights-operator`
- `make build-debug` - Compiles with debug symbols for debugging
- `make test` or `make unit` - Runs unit tests with race detection and coverage profiling
- `VERBOSE=-v make unit` - Run unit tests with verbose output
- `VERBOSE=-count=1 make test` - Run tests without caching

### Linting and Code Quality
- `make lint` - Run golangci-lint with project configuration (requires golangci-lint >= 1.39)
- `make precommit` - Execute pre-commit hook (checks stashed changes)
- `make githooks` - Configure repository to use git hooks (recommended setup)

### Running the Operator
- `make run` - Execute insights-operator with default config (`config/local.yaml`)
- `CONFIG=config/custom.yaml make run` - Run with custom configuration
- `bin/insights-operator start --config=config/local.yaml --kubeconfig=$KUBECONFIG` - Direct execution

### Container Operations
- `make build-container` - Build container image using podman/docker
- `make build-debug-container` - Build debug container image

### Documentation and Tools
- `make docs` - Generate gathered-data documentation from code comments
- `make changelog` - Update changelog (requires GITHUB_TOKEN environment variable)
- `make vendor` - Update Go module dependencies (`go mod tidy && go mod vendor && go mod verify`)

## Project Architecture

### Core Components
The Insights Operator is a Kubernetes operator that periodically gathers anonymized cluster data and uploads it to Red Hat Insights for analysis.

**Main Entry Point**: `cmd/insights-operator/main.go` - Sets up cobra CLI with subcommands for start, receive, gather, and gather-and-upload operations.

**Operator Controller**: `pkg/controller/operator.go` - Main operator logic that coordinates periodic data gathering, uploading, and status reporting.

### Key Packages Structure
- `pkg/gather/` - Data gathering logic and gatherer implementations
- `pkg/gatherers/` - Three main gatherer types:
  - `clusterconfig/` - Regular cluster configuration gathering (default 2h interval)
  - `workloads/` - Workload fingerprint data (12h interval, not configurable)
  - `conditional/` - Conditional gathering based on external rules from console.redhat.com
- `pkg/insights/` - Insights client for uploading/downloading data and reports
- `pkg/config/` - Configuration management and observation
- `pkg/controller/` - Operator controllers and status management
  - `gather_commands.go` - Contains `GatherJob` type for non-periodic gathering (gather, gather-and-upload commands)
  - `operator.go` - Main `Operator` type for periodic gathering in production
- `pkg/recorder/` - Archive recording and disk management
- `pkg/anonymization/` - Data obfuscation and anonymization

### Configuration
The operator reads configuration from multiple sources (in order of precedence):
1. `insights-config` ConfigMap in `openshift-insights` namespace
2. `support` Secret in `openshift-config` namespace
3. Default configuration from `config/pod.yaml`

Authentication uses tokens from the `pull-secret` secret in `openshift-config` namespace.

### Scheduling Pattern
The operator uses `wait.Until` for periodic tasks:
- **Gatherer**: Collects cluster data
- **Uploader**: Uploads archives to console.redhat.com
- **Downloader**: Downloads Insights analysis reports
- **Config Observer**: Monitors configuration changes (5min interval)
- **Disk Pruner**: Removes old archives (runs every second interval)
- **SCA Controller**: Simple Content Access certificate management
- **Cluster Transfer Controller**: Handles cluster transfer operations
- **Runtime Extractor Controller**: Manages the insights-runtime-extractor DaemonSet lifecycle (can be disabled via `dataReporting/disableRuntimeExtractor` config)

### Service Accounts
- `operator`: Main service account for the insights-operator deployment
- `gather`: Privileged service account for cluster-wide data gathering (impersonated by operator)

### Custom Resource Definitions

The Insights Operator uses two main CRDs defined in the `openshift/api` repository (both v1alpha2):

#### DataGather (insights.openshift.io/v1alpha2)
Provides configuration and status for on-demand Insights data gathering (TechPreview):
- **Purpose**: Trigger and configure individual data gathering operations
- **Key spec fields**:
  - `dataPolicy` - Optional obfuscation settings (ObfuscateNetworking, WorkloadNames)
  - `gatherers` - Optional gatherer configuration (All or Custom mode with enable/disable per gatherer)
  - `storage` - Optional persistent storage configuration (PersistentVolume or Ephemeral)
- **Key status fields**:
  - `conditions` - Status of gathering phases (DataUploaded, DataRecorded, DataProcessed, RemoteConfiguration*)
  - `gatherers` - Individual gatherer statuses with execution time
  - `insightsRequestID` - Tracking ID for console.redhat.com processing
  - `insightsReport` - Downloaded analysis with health checks and recommendations
  - `startTime`/`finishTime` - Gathering execution timestamps
- **Feature gate**: `InsightsOnDemandDataGather`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openshift/insights-operator](https://github.com/openshift/insights-operator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
