---
trigger: always_on
description: This file provides comprehensive guidance for AI agents working with the ADL CLI project. It covers project architecture, development workflow, testing strategies, and conventions.
---

# AGENTS.md

This file provides comprehensive guidance for AI agents working with the ADL CLI project. It covers project architecture, development workflow, testing strategies, and conventions.

## Project Overview

**ADL CLI** (Agent Definition Language Command Line Interface) is a Go-based tool for generating enterprise-ready A2A (Agent-to-Agent) servers from YAML-based Agent Definition Language (ADL) files. It eliminates boilerplate code and ensures consistent patterns across agent implementations.

### Key Technologies
- **Language**: Go 1.26+
- **Framework**: Cobra CLI framework
- **Templating**: Go templates with Sprig functions
- **Build System**: Taskfile for automation
- **CI/CD**: GitHub Actions with semantic-release
- **Containerization**: Docker with multi-platform support

## Architecture and Structure

### Project Layout
```text
adl-cli/
├── cmd/                    # CLI command implementations
│   ├── root.go            # Main CLI setup
│   ├── generate.go        # Generate command
│   ├── init.go            # Init command
│   ├── validate.go        # Validate command
│   └── *_test.go          # Command tests
├── internal/              # Internal packages
│   ├── generator/         # Code generation engine
│   ├── schema/           # ADL schema definitions and validation
│   ├── templates/        # Template system
│   │   ├── common/       # Cross-language templates
│   │   ├── languages/    # Language-specific templates
│   │   └── sandbox/      # Environment templates
│   └── prompt/           # Interactive prompt system
├── examples/              # Example ADL files
├── .github/workflows/     # CI/CD workflows
├── Taskfile.yml          # Build automation
├── go.mod                # Go dependencies
└── main.go               # Application entry point
```

### Core Components

1. **CLI Interface** (`cmd/`): Cobra-based command structure with `init`, `generate`, and `validate` commands
2. **Template Engine** (`internal/templates/`): Multi-language template system with support for Go, Rust, and TypeScript
3. **Schema System** (`internal/schema/`): YAML schema validation using JSON Schema
4. **Generator** (`internal/generator/`): File generation engine with `.adl-ignore` support
5. **Prompt System** (`internal/prompt/`): Interactive CLI prompts for project initialization

## Development Environment Setup

### Prerequisites
- Go 1.26+ (as specified in `go.mod`)
- [Task](https://taskfile.dev/) for build automation
- Docker (for container operations)
- Git for version control

### Quick Setup
```bash
# Clone the repository
git clone https://github.com/inference-gateway/adl-cli.git
cd adl-cli

# Install dependencies
task mod

# Build the CLI
task build

# Run tests
task test
```

### Development Tools
- **Task Runner**: Use `task <command>` for all development operations
- **Linting**: `golangci-lint` configured in CI workflow
- **Formatting**: `go fmt` and `prettier` for consistent code style
- **Testing**: Go's built-in testing framework with coverage support

## Key Commands

### Build and Development
```bash
# Build the ADL CLI binary
task build                    # Build to bin/adl
task install                  # Install to GOPATH/bin

# Development mode
task dev -- <args>           # Run CLI with arguments
task dev -- init my-agent    # Interactive project setup
```

### Code Quality
```bash
# Format code
task fmt                     # Format Go code
task format                  # Run prettier formatter

# Lint and vet
task lint                    # Run golangci-lint
task vet                     # Run go vet

# Complete CI pipeline
task ci                      # fmt → lint → test → build
```

### Testing
```bash
# Run tests
task test                    # Run all tests
task test:coverage           # Tests with coverage

# Example validation
task examples:test           # Validate all example ADL files
task examples:generate       # Generate projects from examples
```

### Release and Distribution
```bash
# Build release binaries
task release                 # Multi-platform builds via goreleaser

# Docker operations
task docker:build           # Build Docker image
```

## Testing Instructions

### Test Structure
- **Unit Tests**: Test individual functions and methods
- **Integration Tests**: Test command execution and file generation
- **Example Tests**: Validate example ADL files and generated output

### Running Tests
```bash
# Run all tests
go test -v ./...

# Run specific package tests
go test -v ./cmd
go test -v ./internal/generator
go test -v ./internal/schema

# Run with coverage
go test -v -cover ./...
```

### Test Patterns
1. **Table-driven tests**: Use for comprehensive test coverage
2. **Golden files**: Compare generated output with expected results
3. **Mock file system**: Test file operations without touching disk
4. **Command testing**: Test CLI commands with different arguments

### Example Testing
```bash
# Validate example ADL files
adl validate examples/go-agent.yaml
adl validate examples/rust-agent.yaml
adl validate examples/cloudrun-agent.yaml

# Generate and test example projects
rm -rf test-output
mkdir -p test-output

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [inference-gateway/adl-cli](https://github.com/inference-gateway/adl-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
