---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# Claude-SPARC Automated Development System For Claude Code

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

The SPARC Automated Development System (`claude-sparc.sh`) is a comprehensive, agentic workflow for automated software development using the SPARC methodology (Specification, Pseudocode, Architecture, Refinement, Completion). This system leverages Claude Code's built-in tools for parallel task orchestration, comprehensive research, and Test-Driven Development.

## Features

- **Comprehensive Research Phase**: Automated web research using parallel batch operations
- **Full SPARC Methodology**: Complete implementation of all 5 SPARC phases
- **TDD London School**: Test-driven development with mocks and behavior testing
- **Parallel Orchestration**: Concurrent development tracks and batch operations
- **Quality Assurance**: Automated linting, testing, and security validation
- **Detailed Commit History**: Structured commit messages for each development phase
- **Multi-Agent Coordination**: Sophisticated system for collaborative development
- **Memory Bank**: Persistent knowledge sharing across agent sessions

## Primary Development Methodology: SPARC

This project follows the **SPARC Automated Development System** (`claude-sparc.sh`) which implements a comprehensive, structured approach to software development using the SPARC methodology: **Specification, Pseudocode, Architecture, Refinement, Completion**.

## Usage

### Basic Usage
```bash
./claude-sparc.sh
```

### With Arguments
```bash
./claude-sparc.sh [OPTIONS] [PROJECT_NAME] [README_PATH]
```

### Help
```bash
./claude-sparc.sh --help
```

## Command Line Options

### Core Options
- `-h, --help` - Show help message and exit
- `-v, --verbose` - Enable verbose output for detailed logging
- `-d, --dry-run` - Show what would be executed without running
- `-c, --config FILE` - Specify MCP configuration file (default: .roo/mcp.json)

### Research Options
- `--skip-research` - Skip the web research phase entirely
- `--research-depth LEVEL` - Set research depth: basic, standard, comprehensive (default: standard)

### Development Options
- `--mode MODE` - Development mode: full, backend-only, frontend-only, api-only (default: full)
- `--skip-tests` - Skip test development (not recommended)
- `--coverage TARGET` - Test coverage target percentage (default: 100)
- `--no-parallel` - Disable parallel execution

### Commit Options
- `--commit-freq FREQ` - Commit frequency: phase, feature, manual (default: phase)
- `--no-commits` - Disable automatic commits

### Output Options
- `--output FORMAT` - Output format: text, json, markdown (default: text)
- `--quiet` - Suppress non-essential output

## Examples

### Basic Development
```bash
# Full-stack development with default settings
./claude-sparc.sh my-app docs/requirements.md

# Backend API development with verbose output
./claude-sparc.sh --mode api-only --verbose user-service api-spec.md

# Frontend-only development with custom coverage
./claude-sparc.sh --mode frontend-only --coverage 90 web-app ui-spec.md
```

### Research Configuration
```bash
# Skip research for rapid prototyping
./claude-sparc.sh --skip-research --coverage 80 prototype-app readme.md

# Comprehensive research for complex projects
./claude-sparc.sh --research-depth comprehensive enterprise-app requirements.md

# Basic research for simple projects
./claude-sparc.sh --research-depth basic simple-tool spec.md
```

### Development Modes
```bash
# API-only development
./claude-sparc.sh --mode api-only --commit-freq feature api-service spec.md

# Backend services only
./claude-sparc.sh --mode backend-only --no-parallel backend-service requirements.md

# Frontend application only
./claude-sparc.sh --mode frontend-only --output json frontend-app ui-spec.md
```

### Testing and Quality
```bash
# Skip tests for rapid prototyping (not recommended)
./claude-sparc.sh --skip-tests --commit-freq manual prototype readme.md

# Custom coverage target
./claude-sparc.sh --coverage 95 --verbose production-app requirements.md

# Manual commit control
./claude-sparc.sh --no-commits --dry-run complex-app spec.md
```

### Advanced Usage
```bash
# Dry run to preview execution
./claude-sparc.sh --dry-run --verbose my-project requirements.md

# Custom MCP configuration
./claude-sparc.sh --config custom-mcp.json --mode full my-app spec.md

# Quiet mode with JSON output
./claude-sparc.sh --quiet --output json --mode api-only service spec.md
```

## SPARC Phases Explained

### Phase 0: Research & Discovery
- **Parallel Web Research**: Uses `BatchTool` and `WebFetchTool` for comprehensive domain research
- **Technology Stack Analysis**: Researches best practices and framework comparisons
- **Implementation Patterns**: Gathers code examples and architectural patterns
- **Competitive Analysis**: Studies existing solutions and industry trends

### Phase 1: Specification
- **Requirements Analysis**: Extracts functional and non-functional requirements
- **User Stories**: Defines acceptance criteria and system boundaries
- **Technical Constraints**: Identifies technology stack and deployment requirements

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [davidakpele/wifi-densepose](https://github.com/davidakpele/wifi-densepose) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
