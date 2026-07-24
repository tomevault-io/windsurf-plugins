---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Claude Code Environment Switcher (CCE) is a lightweight Go CLI tool that manages multiple Claude Code API endpoint configurations, allowing seamless switching between different environments (production, staging, custom API providers, etc.). The tool acts as a wrapper around Claude Code, injecting appropriate environment variables before launching.

## Commands

### Build and Test
```bash
# Build binary
make build              # or: go build -o cce .

# Run all tests
make test               # or: go test -v ./...

# Run specific test
go test -v -run TestValidateName ./...      # Run single test by name
go test -v -run "TestSecurity" ./...        # Run all security tests
go test -v -run "TestFlagPassthrough" ./...  # Run flag passthrough tests

# Test coverage
make test-coverage      # Generate HTML coverage report
go test -coverprofile=coverage.out ./...    # Generate coverage profile

# Performance benchmarks
make bench              # or: go test -bench=. -benchmem ./...

# Code quality
make quality            # Runs fmt + vet + test
make fmt                # Format all Go files
make vet                # Run static analysis

# Security tests
make test-security      # or: go test -v -run TestSecurity ./...

# Clean build artifacts
make clean              # Remove cce binary and coverage files
```

### Installation
```bash
# Install to system PATH
make install            # Builds and moves to /usr/local/bin/

# Development build
go build -o cce .       # Build binary in current directory
```

## Architecture

The project uses a minimalist 4-file architecture following KISS principles:

### Core Components

**`main.go`** (580+ lines)
- CLI entry point and command routing
- Two-phase argument parser for flag passthrough system
- Model validation with configurable patterns
- Environment validation (name, URL, API key)
- Help text generation with flag passthrough examples

**`config.go`** (367 lines)
- Atomic file operations with temp file + rename pattern
- Automatic backup creation before modifications
- Corruption recovery with `.backup` files
- JSON marshaling with proper indentation
- File permission management (0600 files, 0700 directories)

**`ui.go`** (1000+ lines)
- ANSI-free display core using carriage return and padding
- 4-tier progressive fallback system
- DisplayState tracking for stateful rendering
- TextPositioner for universal cursor control
- LineRenderer with differential updates
- Terminal width detection and responsive formatting

**`launcher.go`** (174 lines)
- Process execution with `exec.Command`
- Clean environment variable injection
- Comprehensive error handling with exit code preservation
- Signal forwarding for graceful shutdown

### Key Design Patterns

**Flag Passthrough System**
- Phase 1: Parse CCE-specific flags (`--env`, `-e`, `add`, `list`, `remove`)
- Phase 2: Collect remaining arguments for Claude Code
- Security validation prevents shell injection
- Supports `--` separator for explicit boundary

**ANSI-Free Display Management**
- Core functionality works without ANSI escape codes
- Uses carriage return (`\r`) and space padding for updates
- Progressive enhancement for capable terminals
- Stateful rendering prevents display accumulation

**4-Tier Terminal Fallback**
1. Full interactive: Arrow keys + ANSI enhancements
2. Basic interactive: Arrow keys without ANSI
3. Numbered selection: Simple numbered menu
4. Headless mode: Auto-select for CI/CD

**Configuration Atomicity**
- Write to temp file first
- Validate JSON structure
- Atomic rename to target
- Automatic backup before changes
- Recovery from corrupted configs

### Recent Enhancements

**Per-Environment API Key Variable** (2024)
- Choose between `ANTHROPIC_API_KEY` (default) and `ANTHROPIC_AUTH_TOKEN` per environment
- Runtime override with `-k` or `--key-var` flag
- Backward compatible with existing configurations

**Flag Passthrough System**
- Two-phase argument parsing separating CCE flags from Claude arguments
- Support for `--` separator for explicit argument separation
- Security validation preventing command injection

**ANSI-Free Display Management**
- DisplayState tracking with differential updates
- TextPositioner using carriage return and padding (no ANSI codes)
- LineRenderer for stateful menu rendering
- Smart truncation preserving essential information

**Additional Environment Variables**
- Configure custom variables per environment (e.g., `ANTHROPIC_SMALL_FAST_MODEL`)
- Interactive configuration during `cce add`
- Automatic injection when launching Claude Code

## Configuration

### File Location and Structure

Environments stored in `~/.claude-code-env/config.json`:
```json
{
  "environments": [
    {
      "name": "production",
      "url": "https://api.anthropic.com",
      "api_key": "sk-ant-api03-xxxxx",
      "api_key_env": "ANTHROPIC_API_KEY",
      "model": "claude-3-5-sonnet-20241022",
      "env_vars": {
        "ANTHROPIC_SMALL_FAST_MODEL": "claude-3-haiku-20240307"
      }
    }
  ]
}
```

### Environment Variables

**Model Validation**
- `CCE_MODEL_PATTERNS`: Comma-separated custom regex patterns

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cexll/claude-code-env](https://github.com/cexll/claude-code-env) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
