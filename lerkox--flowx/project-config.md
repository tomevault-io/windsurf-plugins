---
trigger: always_on
description: Please answer in Chinese.
---

# CLAUDE.md

Please answer in Chinese.

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Go-based CI/CD pipeline execution library that supports multiple execution backends (Docker, Kubernetes, SSH, Local). The library uses a DAG (Directed Acyclic Graph) structure to manage pipeline dependencies and supports concurrent execution of independent tasks.

## Development Commands

### Building
```bash
go build ./...          # Build all packages in the project
go mod tidy             # Clean up dependencies
```

### Testing
```bash
go test ./test/         # Run all tests
go test ./test/ -v      # Run tests with verbose output
```

**Note**: All tests should pass. Run `go build ./...` before testing to ensure no compilation errors.

**Test Configuration File Maintenance**:
- Test configuration files are located in `test/fixtures/runtime/` directory
- The mapping between configuration files and test cases is documented in `test/fixtures/runtime/README.md`
- When adding new test cases, update `test/fixtures/runtime/README.md` to document the new configuration files

### Code Quality
```bash
go fmt ./...            # Format all Go code
go vet ./...            # Run static analysis
```

## Architecture Overview

### Core Components

1. **Pipeline Interface** (`pipeline.go`): Main pipeline lifecycle management
2. **DAG Graph Implementation** (`pipeline_impl.go`): Manages task dependencies and traversal
3. **Node System** (`node.go`, `node_impl.go`): Individual task management with state tracking
4. **Executor Pattern** (`executor.go`): Pluggable backend execution system
5. **Runtime** (`runtime.go`, `runtime_impl.go`): Pipeline execution runtime with process safety
6. **Edge System** (`edge.go`, `edge_impl.go`): DAG edges with conditional expression support
7. **Template Engine** (`templete.go`, `templete_impl.go`): Template rendering for dynamic configuration
8. **Metadata Store** (`metadata.go`, `metadata_impl.go`): Process-safe metadata management
9. **Configuration** (`config.go`): Pipeline configuration structures and parsing

### Key Architecture Patterns

- **DAG-based Pipeline**: Tasks are nodes in a directed acyclic graph with dependencies
- **Executor Pattern**: Different execution backends (Function, Docker, K8s, SSH, Local)
- **Event-driven**: Pipeline and node lifecycle events for monitoring
- **Concurrent Execution**: Independent tasks run in parallel using goroutines
- **Conditional Edges**: Edges support conditional expressions for dynamic execution paths
- **Template Engine**: Support for template rendering in configuration
- **Metadata Management**: Process-safe metadata storage and retrieval during execution

### Directory Structure

- `/` - Core pipeline interfaces and implementations
- `/executor/` - Execution backend implementations
  - `/kubenetes/` - Fully implemented Kubernetes executor
  - `/docker/` - Placeholder for Docker executor
  - `/ssh/` - Placeholder for SSH executor
  - `/local/` - Placeholder for Local executor
- `/test/` - Test suite
- `/doc/` - Documentation files
  - `config.md` - Configuration detailed documentation
  - `templete.md` - Template engine documentation

### Configuration

Pipeline configuration uses YAML format with the following structure:
```yaml
Version: "1.0"           # Configuration version
Name: my-pipeline        # Pipeline name

Metadate:                # Metadata configuration
  type: in-config        # Metadata store type (in-config, redis, http)
  data:                  # Initial metadata key-value pairs
    key1: value1

AI:                      # AI-related configuration
  intent: "描述Pipeline意图"
  constraints:           # Key constraints
    - "约束1"
  template: "template-id"

Param:                   # Pipeline parameters
  key: value

Executors:               # Global executor definitions
  local:
    type: local
    config: {}
  docker:
    type: docker
    config: {}

Logging:                 # Log pushing configuration
  endpoint: http://log-center/api/v1/logs
  headers: {}
  timeout: 5s

Graph: |                 # DAG definition (Mermaid stateDiagram-v2 format)
  stateDiagram-v2
    [*] --> Node1
    Node1 --> Node2

Status:                  # Node runtime status
  Node1: Finished

Nodes:                   # Node definitions with execution details
  NodeName:
    executor: local      # Reference to global executor
    image: optional-image
    steps:               # Multi-step execution
      - name: step1
        run: command
```

See `config.example.yaml` for a complete example.

## Current Implementation Status

**Implemented**:
- ✅ Core pipeline DAG structure and traversal
- ✅ Basic pipeline execution with concurrent processing
- ✅ Kubernetes executor (fully functional)
- ✅ Node state management and event system
- ✅ Cycle detection and graph validation
- ✅ Conditional edges with expression evaluation
- ✅ Template engine for dynamic configuration
- ✅ Metadata store interface and implementations
- ✅ Pipeline Runtime with process safety
- ✅ Multi-step node execution
- ✅ Graph text visualization
- ✅ Log pushing interface

**Planned but not implemented**:
- 🔄 Function executor
- 🔄 Docker executor
- 🔄 SSH executor
- 🔄 Local executor

## Development Notes


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LerkoX) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-13 -->
