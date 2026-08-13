---
trigger: always_on
description: This document provides comprehensive information for AI agents and developers working on the Structyl codebase.
---

# AGENTS.md - Repository Internals & Development Guide

This document provides comprehensive information for AI agents and developers working on the Structyl codebase.

> **Note:** This document uses prescriptive language ("Always", "Do not") to indicate strong recommendations for maintaining consistency across the codebase. These are guidelines, not absolute requirements—deviations are acceptable with documented justification.

> **Synchronization:** Exit codes, error handling, and specification details in this file must match the formal specifications in `docs/specs/`. If discrepancies arise, the specifications are authoritative.

## Project Overview

**Structyl** is a multi-language build orchestration CLI written in Go. It provides unified commands (`build`, `test`, `clean`, etc.) that work across different programming language implementations in a monorepo.

**Primary Use Case:** Managing polyglot projects where multiple language implementations must produce semantically identical outputs (e.g., a statistical library implemented in Rust, Python, Go, and C#).

## Build System

**This project uses [mise](https://mise.jdx.dev/) as the primary build system.** All tools and commands should be run via mise.

```bash
# Always use mise to run commands
mise run build        # Build the project
mise run test         # Run tests
mise run check        # Run lint and static analysis
mise run check:fix    # Auto-fix static analysis issues

# List available tasks
mise tasks
```

**Important:** Do not run Go commands directly (e.g., `go build`, `go test`). Always use the corresponding mise tasks to ensure consistent tooling and environment.

## Architecture Summary

```
┌─────────────────────────────────────────────────────────────────┐
│                        cmd/structyl/main.go                      │
│                         (Entry Point)                            │
└─────────────────────────────────────────────────────────────────┘
                                 │
                                 ▼
┌─────────────────────────────────────────────────────────────────┐
│                        internal/cli                              │
│         (Command Parsing, Routing, Global Flags)                 │
└─────────────────────────────────────────────────────────────────┘
                                 │
              ┌──────────────────┼──────────────────┐
              ▼                  ▼                  ▼
┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐
│  internal/config │  │ internal/project │  │  internal/runner │
│  (JSON Loading)  │  │  (Discovery)     │  │  (Orchestration) │
└──────────────────┘  └──────────────────┘  └──────────────────┘
              │                  │                  │
              └──────────────────┼──────────────────┘
                                 ▼
┌─────────────────────────────────────────────────────────────────┐
│                       internal/target                            │
│            (Target Interface, Registry, Execution)               │
└─────────────────────────────────────────────────────────────────┘
                                 │
              ┌──────────────────┼──────────────────┐
              ▼                  ▼                  ▼
┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐
│internal/toolchain│  │   internal/mise  │  │  internal/output │
│ (Presets, Detect)│  │  (Task Runner)   │  │  (Formatting)    │
└──────────────────┘  └──────────────────┘  └──────────────────┘
```

## Package Structure

```
structyl/
├── cmd/structyl/
│   └── main.go                 # CLI entry point - calls cli.Run()
├── internal/
│   ├── cli/                    # Command-line interface
│   │   ├── cli.go              # Run(), parseGlobalFlags(), printUsage()
│   │   ├── commands.go         # cmdMeta(), cmdTarget(), cmdCI(), etc.
│   │   └── init.go             # cmdInit() - project initialization
│   ├── config/                 # Configuration loading
│   │   ├── config.go           # Load(), LoadWithDefaults(), LoadAndValidate()
│   │   ├── schema.go           # Config, TargetConfig, etc. structs
│   │   ├── defaults.go         # ApplyDefaults()
│   │   ├── validate.go         # ValidateProjectName(), validation logic
│   │   └── unknown.go          # Unknown field detection/warnings
│   ├── project/                # Project discovery
│   │   ├── root.go             # FindRoot() - walks up to find .structyl/config.json
│   │   ├── project.go          # LoadProject() - loads config + creates registry
│   │   └── discover.go         # Auto-discovers targets from directories
│   ├── target/                 # Target management
│   │   ├── target.go           # Target interface definition
│   │   ├── impl.go             # targetImpl struct, Execute(), interpolateVars()
│   │   └── registry.go         # Registry, TopologicalOrder(), validateDependencies()
│   ├── runner/                 # Build orchestration
│   │   ├── runner.go           # Runner, Run(), RunAll(), runParallel()
│   │   ├── docker.go           # DockerRunner, IsDockerAvailable()
│   │   ├── compose.go          # Docker Compose file generation
│   │   └── ci.go               # RunCI(), CI pipeline simulation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AndreyAkinshin/structyl](https://github.com/AndreyAkinshin/structyl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
