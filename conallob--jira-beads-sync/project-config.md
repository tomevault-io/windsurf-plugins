---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Go-based CLI tool to sync Jira task trees with beads issues. It handles the hierarchical structure of Jira tasks (epics, stories, subtasks) and provides bidirectional synchronization with the beads issue tracking system while preserving dependencies and relationships.

The tool supports multiple modes:
1. **Quickstart mode**: Fetch issues directly from Jira API and sync them to beads
2. **Sync mode**: Sync beads state changes back to Jira (bidirectional)
3. **Convert mode**: One-way conversion of previously exported Jira JSON files

## Language & Tooling

**Language**: Go (Golang)

### Commands

**Build**:
```bash
go build -o jira-beads-sync ./cmd/jira-beads-sync
```

**Run**:
```bash
go run ./cmd/jira-beads-sync [args]
```

**Test**:
```bash
go test ./...                    # Run all tests
go test -v ./...                 # Run with verbose output
go test -run TestFunctionName    # Run specific test
```

**Lint & Format**:
```bash
go fmt ./...                     # Format all code
golangci-lint run                # Run linter
```

**Generate Protobuf Code**:
```bash
protoc --go_out=. --go_opt=module=github.com/conallob/jira-beads-sync --proto_path=proto proto/jira.proto proto/beads.proto
```

### Go Project Structure

- `cmd/jira-beads-sync/` - Main application entry point with CLI commands
- `internal/jira/` - Jira integration
  - `adapter.go` - JSON to protobuf adapter for Jira exports
  - `client.go` - Jira REST API v2 client for fetching issues
- `internal/beads/` - JSONL rendering layer on top of protobuf
  - `jsonl.go` - Renders beads protobuf to JSONL format
- `internal/converter/` - Conversion logic between Jira and beads protobuf
- `internal/config/` - Configuration management (credentials, base URL)
- `proto/` - Protocol Buffer definitions (source of truth for data structures)
- `gen/jira/` - Generated Go code from jira.proto
- `gen/beads/` - Generated Go code from beads.proto
- `testdata/` - Sample Jira exports and expected beads output
- `go.mod` - Go module definition

## Beads Integration

This tool creates issues for the beads system (git-backed issue tracker).

**Official Beads Repository**: https://github.com/steveyegge/beads

Key beads concepts:

- **Issues**: Work items stored in `.beads/issues.jsonl` as JSONL (JSON Lines) format
- **Dependencies**: Issues can depend on other issues using `dependsOn` field
- **Status**: `open`, `in_progress`, `blocked`, or `closed`
- **Priority**: `p0` (critical) through `p4` (low)
- **Epics**: Large features that group related issues using `epic` field, stored in `.beads/epics.jsonl`

Relevant beads commands for testing:
- `bd list` - List all issues
- `bd show <id>` - Show issue details
- `bd create` - Create new issue interactively
- `bd dep add <issue> <dependency>` - Add dependency between issues
- `bd epic create <name>` - Create a new epic

## Architecture

This tool uses Protocol Buffers as the internal data structure format with rendering layers for external formats:

### Data Flow

```
JSON (Jira) → Protobuf (Jira) → Protobuf (Beads) → JSONL (Beads)
     ↓              ↓                  ↓               ↓
Adapter    Generated Types    Converter      Renderer
```

1. **Jira Adapter** (`internal/jira/adapter.go`): Parses Jira JSON exports into protobuf messages defined in `proto/jira.proto`
2. **Converter** (`internal/converter/proto_converter.go`): Transforms Jira protobuf to beads protobuf with mappings for status, priority, and dependencies
3. **JSONL Renderer** (`internal/beads/jsonl.go`): Renders beads protobuf to JSONL files compatible with the beads CLI

### Why Protocol Buffers?

- **Single Source of Truth**: Data structures defined once in `.proto` files
- **Type Safety**: Strong typing across all layers
- **Versioning**: Built-in support for schema evolution
- **Multiple Formats**: Easy to add new rendering formats without changing core logic
- **Performance**: Efficient serialization when needed

### Output Format

The tool outputs JSONL (JSON Lines) format, which is the standard format used by beads:
- Reads JSON (Jira exports)
- Processes data as protobuf internally
- Writes JSONL (beads issues) to `.beads/issues.jsonl` and `.beads/epics.jsonl`

This architecture separates concerns: protobuf handles data structure and validation, while format-specific code handles I/O.

## Development Approach

### Git Workflow

**IMPORTANT**: All new development must be done in feature branches, not directly on `main`.

1. **Create a feature branch** before starting any new work:
   ```bash
   git checkout -b feature/your-feature-name
   # or for bug fixes:
   git checkout -b fix/bug-description
   ```

2. **Make your changes** in the feature branch

3. **Commit your changes** following conventional commit format:
   - `feat:` for new features
   - `fix:` for bug fixes
   - `docs:` for documentation changes
   - `test:` for test changes
   - `refactor:` for code refactoring
   - `ci:` for CI/CD changes

4. **Push the feature branch** to remote:
   ```bash
   git push origin feature/your-feature-name
   ```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [conallob/jira-beads-sync](https://github.com/conallob/jira-beads-sync) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
