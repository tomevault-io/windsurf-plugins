---
trigger: always_on
description: This document provides machine-readable context for AI agents working on this codebase.
---

# Agent Instructions

This document provides machine-readable context for AI agents working on this codebase.

For detailed project rules and TDD workflow, see `.claude/CLAUDE.md`.

---

## Project Overview

**Name**: Compound Agent
**Purpose**: Learning system that helps Claude Code avoid repeating mistakes across sessions
**Stack**: Go (primary) + Rust (embedding daemon) + Node/pnpm (npm wrapper distribution)
**CLI**: `ca` (alias: `compound-agent`), built with Cobra
**Module**: `github.com/nathandelacretaz/compound-agent`

### What It Does

1. Captures lessons from user corrections, self-corrections, and test failures
2. Stores lessons in JSONL (git-tracked) with SQLite index (cache)
3. Retrieves relevant lessons via local embeddings (ONNX Runtime, Rust daemon)
4. Injects lessons at session-start and plan-time via hooks

### Key Components

| Component | Location | Purpose |
|-----------|----------|---------|
| CLI entrypoint | `go/cmd/ca/` | Cobra root command, hook dispatch |
| Commands | `go/internal/cli/` | All CLI subcommand definitions |
| Storage | `go/internal/storage/` | SQLite + FTS5 (search, cache, sync, knowledge DB) |
| Search | `go/internal/search/` | Hybrid search (keyword + vector ranking) |
| Capture | `go/internal/capture/` | Trigger detection + quality filters |
| Retrieval | `go/internal/retrieval/` | Session-start and plan-time retrieval |
| Compound | `go/internal/compound/` | Compound synthesis (clustering, patterns) |
| Knowledge | `go/internal/knowledge/` | Knowledge indexing and embedding |
| Embed | `go/internal/embed/` | Embedding daemon IPC (client, lifecycle) |
| Hook | `go/internal/hook/` | Hook runner, phase state, failure tracking |
| Memory | `go/internal/memory/` | Memory types and JSONL operations |
| Setup | `go/internal/setup/` | Template installation (embedded templates) |
| Util | `go/internal/util/` | Shared utilities (stdin, shell escape, cosine) |
| Build | `go/internal/build/` | Build version injection |
| npm dist | `go/internal/npmdist/` | npm distribution wrapper |
| Embed daemon | `rust/embed-daemon/` | Rust ONNX Runtime embedding daemon |

### Architecture

```
go/
├── cmd/ca/                     <- CLI entrypoint (Cobra root command)
├── internal/                   <- All packages (unexported)
│   ├── cli/                    <- Cobra command definitions
│   ├── storage/                <- SQLite + FTS5
│   ├── search/                 <- Hybrid search (keyword + vector)
│   ├── capture/                <- Lesson capture
│   ├── retrieval/              <- Session retrieval
│   ├── compound/               <- Compound synthesis
│   ├── knowledge/              <- Knowledge indexing
│   ├── embed/                  <- Embedding daemon IPC
│   ├── hook/                   <- Hook management
│   ├── memory/                 <- Memory types / JSONL
│   ├── setup/                  <- Template installation
│   ├── util/                   <- Shared utilities
│   ├── build/                  <- Version injection
│   └── npmdist/                <- npm wrapper
rust/
└── embed-daemon/               <- Rust embedding daemon (ONNX Runtime)
.claude/
├── CLAUDE.md                   <- Always-loaded project rules
├── compound-agent.json         <- Config
├── agents/                     <- Subagent definitions (TDD pipeline)
├── commands/                   <- Slash commands
├── skills/compound/            <- Skill definitions (cook-it, spec-dev, plan, work, review, compound, etc.)
└── lessons/
    └── index.jsonl             <- Source of truth (git-tracked)
.claude/.cache/
    └── lessons.sqlite          <- Rebuildable index (.gitignore)
```

---

## Build, Test, Run Commands

```bash
# Build CLI binary
cd go && go build ./cmd/ca

# Run full test suite
cd go && go test ./...

# Static analysis
cd go && go vet ./...

# Lint (golangci-lint v2)
cd go && golangci-lint run ./...

# Build via Makefile
make -C go build

# Test via Makefile
make -C go test
```

### Build Requirements

- Go 1.26+

### Dependencies (minimal)

| Dependency | Purpose |
|------------|---------|
| `modernc.org/sqlite` | Pure-Go SQLite driver with FTS5 (no CGO) |
| `github.com/spf13/cobra` | CLI framework |

### CLI Usage

```bash
# Core commands
ca search <query>              # Search lessons (hybrid: keyword + vector)
ca list                        # List all lessons
ca learn                       # Capture a new lesson
ca load-session                # Load high-severity lessons for session context
ca check-plan --plan "..."     # Check a plan against learned lessons

# Knowledge
ca knowledge                   # Knowledge indexing commands

# Maintenance
ca stats                       # Database health
ca compact                     # Reduce lesson database size

# Setup
ca init                        # Setup hooks, templates, config

# Verification
ca verify-gates <epic-id>      # Verify review + compound tasks closed
ca phase-check                 # Cook-it phase state management

# Hooks
ca hooks run <hook-name>       # Run a hook handler

# Advanced
ca capture                     # Structured capture from JSON input
ca detect                      # Detect triggers from JSON input
```

---

## Code Style and Conventions

### File Organization


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Nathandela/compound-agent](https://github.com/Nathandela/compound-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
