---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Klama is an AI-powered CLI tool for troubleshooting DevOps issues, specifically focusing on Kubernetes debugging. It uses an interactive TUI (Terminal User Interface) built with Bubble Tea to provide a conversational interface where users can ask questions and receive AI-powered guidance with optional command execution.

## Development Guidelines

- **Never push directly to main branch**
- If changes are made to the main branch, always:
  - Create a new branch for changes
  - Open a Pull Request (PR)
  - Ask for review before merging
  - Wait for explicit permission to merge

## Architecture

The project follows a clean architecture pattern with these key components:

- **cmd/**: CLI command structure using Cobra framework
  - `root.go`: Main CLI setup with global flags (`--config`, `--debug`)
  - `k8s.go`: Kubernetes debugging command that initializes the full application stack
- **config/**: Configuration management using Viper
  - Supports multiple config file locations (XDG, legacy home directory)
  - Handles OpenAI-compatible API configuration with pricing
- **internal/agent/**: AI agent abstraction layer
  - Defines system prompts for different agent types (currently only Kubernetes)
  - Handles structured JSON responses with retry logic for malformed responses
- **internal/llm/**: Language model client implementation
  - OpenAI-compatible API client with conversation history management
  - Token usage tracking and cost calculation
- **internal/executer/**: Command execution interface
  - Terminal command execution with validation
  - Currently focused on safe kubectl commands only
- **internal/ui/**: Interactive TUI using Bubble Tea
  - Conversational interface with viewport for chat history
  - Command confirmation workflow for safety
- **internal/logger/**: Debug logging system

## Common Commands

### Development
```bash
# Run the application
go run main.go k8s

# Run with debug logging
go run main.go k8s --debug

# Run tests
go test ./...

# Run tests for specific package
go test ./internal/agent

# Build binary
go build -o klama main.go
```

### Configuration
The application requires a config file with AI model settings:
```yaml
agent:
  name: "gpt-4o-mini"
  base_url: "https://api.openai.com/v1"
  auth_token: ""  # Set via KLAMA_AGENT_TOKEN env var
```

## Key Implementation Details

### Agent System
- Agents are typed (currently only `AgentTypeKubernetes`)
- All responses must be valid JSON with specific structure: `{"answer": string, "run_command": string, "reason_for_command": string}`
- Built-in retry mechanism for malformed JSON responses (3 attempts)
- Conversation history maintained throughout session

### Command Execution Safety
- Only read-only kubectl commands are allowed (get, describe, logs)
- All commands validated before execution
- User confirmation required for every command
- Commands limited to 4-hour log lookups by default

### UI State Management
- Four main states: `StateTyping`, `StateAsking`, `StateExecuting`, `StateWaitingForConfirmation`
- Keyboard shortcuts: Ctrl+C (exit), Ctrl+R (restart), Ctrl+S (toggle command response visibility)
- Mouse scrolling and keyboard navigation supported

### Configuration Management
- Supports XDG Base Directory specification
- Environment variable override for auth token (`KLAMA_AGENT_TOKEN`)
- Automatic default config generation if none exists

## Testing

Tests are located alongside source files with `_test.go` suffix. Key test areas:
- Agent JSON parsing and validation
- Configuration loading and validation
- LLM client functionality
- Terminal command execution
- UI state transitions

## Build and Release

### Release Workflow Overview

The project uses an automated release pipeline with GoReleaser and GitHub Actions:

#### CI/CD Pipeline

**Continuous Integration** (`.github/workflows/build-test.yaml`):
- **Triggers**: Pull requests to `main` branch
- **Jobs**:
  - `fmt_and_vet`: Code formatting and vetting checks (Go 1.22, Ubuntu)
  - `unit_tests`: Test matrix across Ubuntu and Windows with Go 1.22
- **Environment**: `CGO_ENABLED=0` for static builds

**Release Pipeline** (`.github/workflows/release.yaml`):
- **Trigger**: Git tag push (`tags: ['*']`)
- **Process**: Uses GoReleaser Action v6 with `goreleaser release --clean`
- **Requirements**: `PERSONAL_ACCESS_TOKEN` secret for GitHub operations

#### Version Management

- Version stored in `cmd/root.go` with defaults: `version = "dev"`, `arch = "dev"`
- Build-time injection via ldflags: `-X github.com/eliran89c/klama/cmd.version={{.Version}} -X github.com/eliran89c/klama/cmd.arch={{.Arch}}`

#### Supported Platforms

- **Linux**: amd64, arm64
- **macOS (Darwin)**: amd64, arm64
- **Windows**: amd64 (arm64 excluded)
- **Archive formats**: tar.gz (Unix-like), zip (Windows)

#### Release Artifacts

Each release includes:
- Source code archives
- Cross-platform binaries
- Checksums file
- Automated Homebrew formula publication

#### Distribution Channels

1. **GitHub Releases**: Direct binary downloads
2. **Homebrew**: `brew install eliran89c/tap/klama`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eliran89c/klama](https://github.com/eliran89c/klama) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-11 -->
