---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Pluqqy is a terminal-based tool for managing LLM prompt pipelines. It stores everything as plain text files (Markdown and YAML) and provides both a TUI and CLI for managing contexts, prompts, and rules that can be combined into reusable pipelines.

## Development Commands

### Building and Running
```bash
# Build the application
make build

# Build and run the TUI
make run

# Install to $GOBIN or $HOME/go/bin
make install

# Run tests
make test

# Format code
make fmt

# Run go vet for static analysis
make vet

# Update dependencies
make deps

# Build for all platforms (Darwin, Linux, Windows)
make build-all
```

### Testing Commands
```bash
# Run all tests with verbose output
go test -v ./...

# Run tests for a specific package
go test -v ./pkg/files/...
go test -v ./pkg/tui/...
go test -v ./pkg/composer/...

# Run tests with coverage
go test -cover ./...

# Generate coverage report
go test -coverprofile=coverage.out ./...
go tool cover -html=coverage.out
```

## Architecture and Code Structure

### Package Organization

The codebase follows a clean architecture with clear separation of concerns:

- **cmd/** - Entry points and CLI command definitions
  - `cmd/pluqqy/main.go` - Main entry point with root command and TUI launcher
  - `cmd/commands/` - Individual CLI command implementations (set, list, show, etc.)

- **pkg/** - Core business logic and features
  - `pkg/files/` - File operations, YAML/Markdown handling, atomic writes
  - `pkg/models/` - Data structures (Pipeline, Component, Settings, Tags)
  - `pkg/tui/` - Terminal UI components using Bubble Tea framework
  - `pkg/composer/` - Pipeline composition and output generation
  - `pkg/search/unified/` - Search engine with query parsing and filtering
  - `pkg/utils/` - Utility functions (token counting, etc.)

- **internal/** - Internal utilities not exposed as public API
  - `internal/cli/` - CLI helpers, formatters, and validators

### Key Components

#### File Storage System (`pkg/files/`)
- All data stored as plain text files in `.pluqqy/` directory
- Components are Markdown files with optional YAML frontmatter for metadata
- Pipelines are YAML files defining component composition
- Atomic file writes using temp files and rename for data safety
- Path validation to prevent directory traversal attacks

#### TUI System (`pkg/tui/`)
- Built with Bubble Tea (Model-View-Update architecture)
- Main app states: List view, Pipeline builder, Component editor, Settings
- Enhanced editor with undo, file references, token counting
- OS-aware keyboard shortcuts (different bindings for Linux/Windows vs macOS)
- Component usage tracking and visualization (Mermaid diagrams)

#### Pipeline Composition (`pkg/composer/`)
- Reads pipeline definitions and resolves component references
- Handles both active and archived components
- Generates composed output with configurable section ordering
- Supports custom output filenames and paths via settings

#### Search Engine (`pkg/search/unified/`)
- Query parser supporting field-based searches (tag:, type:, content:, status:)
- Multi-filter support with AND/OR operations
- Searches across both pipelines and components
- Handles archived items with status:archived filter

### Data Models

#### Component
- Name, Path, Type (contexts/prompts/rules)
- Content (Markdown)
- Tags (string array)
- Modified timestamp

#### Pipeline
- Name, Description, Path
- Components (array of references)
- Tags (string array)
- Created/Updated timestamps

#### Settings
- Output configuration (filename, export path)
- Section formatting and ordering
- Stored in `.pluqqy/settings.yaml`

### Testing Approach

The codebase includes comprehensive test coverage:
- Unit tests for core logic (files, models, search)
- Component tests for TUI elements
- Test helpers in `pkg/tui/testhelpers/` for TUI testing
- Mock file systems for testing file operations

## Important Patterns and Conventions

### Error Handling
- Wrap errors with context using `fmt.Errorf` with `%w` verb
- Validate inputs early (path validation, file size checks)
- Return meaningful error messages for user-facing operations

### File Operations
- Always use atomic writes (temp file + rename) for data safety
- Validate paths to prevent directory traversal
- Check file sizes before reading (10MB limit)
- Handle both forward and backward compatibility for file formats

### TUI Development
- Follow Bubble Tea patterns (Model, Update, View)
- Keep state management separate from view rendering
- Use composition for complex components
- Implement proper cleanup in component lifecycles

### CLI Commands
- Use Cobra for command structure
- Support multiple output formats (text, JSON, YAML)
- Provide clear help text and examples
- Handle ambiguous names gracefully (e.g., same component name in different types)

## Key Features to Maintain

1. **Plain Text Storage** - Everything must remain as readable text files
2. **Atomic Operations** - File writes must be atomic to prevent corruption
3. **Backward Compatibility** - Support old file formats when reading

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pluqqy/pluqqy-terminal](https://github.com/pluqqy/pluqqy-terminal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
