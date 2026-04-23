---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

reskill is a Git-based package manager for AI agent skills, similar to npm/Go modules. It provides declarative configuration (`skills.json`), version locking (`skills.lock`), and seamless synchronization for managing skills across projects and teams.

**Supported Agents:** Cursor, Claude Code, Codex, OpenCode, Windsurf, GitHub Copilot, and more.

## Tech Stack

- **Language:** TypeScript (ES Modules)
- **Runtime:** Node.js >= 18.0.0
- **Build Tool:** Rslib (Rspack-based library bundler)
- **Testing:** Vitest with @vitest/coverage-v8
- **CLI Framework:** Commander.js
- **Package Manager:** pnpm

## Development Commands

```bash
pnpm install          # Install dependencies
pnpm dev              # Development mode (watch)
pnpm build            # Build with Rslib
pnpm test             # Run tests in watch mode
pnpm test:run         # Run tests once
pnpm test:coverage    # Run tests with coverage
pnpm test:integration # Build and run integration tests
pnpm lint             # Lint with Biome
pnpm lint:fix         # Lint and auto-fix
pnpm typecheck        # TypeScript type checking
```

## Architecture

### System Overview

```
┌────────────────────────────────────────────────────────────────┐
│                         User Interface                         │
│                                                                │
│   CLI Commands: init, install, list, info, update, outdated    │
│                 uninstall, link, unlink                        │
└──────────────────────────────┬─────────────────────────────────┘
                               │
                               ▼
┌────────────────────────────────────────────────────────────────┐
│                        SkillManager                            │
│                    (Core Orchestrator)                         │
│                                                                │
│   - Coordinates all skill operations                           │
│   - Manages installation workflow                              │
│   - Handles multi-agent distribution                           │
└───────────────────────────────┬────────────────────────────────┘
                                │
    ┌──────────────────┼────────┼─────────┼──────────────────┐
    │                  │                  │                  │
    ▼                  ▼                  ▼                  ▼
┌──────────────────┐   ┌──────────────────┐   ┌──────────────────┐
│   GitResolver    │   │  CacheManager    │   │    Installer     │
│                  │   │                  │   │                  │
│ - Parse refs     │   │ - Cache ops      │   │ - Symlink        │
│ - Resolve        │   │ - degit          │   │ - Copy           │
│   versions       │   │ - Storage        │   │ - Multi-         │
│ - Build URLs     │   │                  │   │   agent          │
└──────────────────┘   └──────────────────┘   └──────────────────┘
    │                  │                  │                  │
    └──────────────────┼──────────────────┼──────────────────┘
                       │                  │
    ┌──────────────────┼──────────────────┼──────────────────┐
    │                  │                  │                  │
    ▼                  ▼                  ▼                  ▼
┌──────────────────┐   ┌──────────────────┐   ┌──────────────────┐
│  ConfigLoader    │   │   LockManager    │   │  AgentRegistry   │
│                  │   │                  │   │                  │
│ - skills.        │   │ - skills.        │   │ - Agent          │
│   json           │   │   lock           │   │   types          │
│ - Read/          │   │ - Version        │   │ - Paths          │
│   Write          │   │   tracking       │   │ - Detection      │
└──────────────────┘   └──────────────────┘   └──────────────────┘
```

### Directory Structure

```
src/
├── cli/              # CLI commands (Commander.js)
│   ├── commands/     # Individual command handlers
│   └── index.ts      # CLI entry point
├── core/             # Core business logic
│   ├── skill-manager.ts    # Main orchestrator
│   ├── git-resolver.ts     # Git URL parsing and version resolution
│   ├── cache-manager.ts    # Global cache (~/.reskill-cache)
│   ├── config-loader.ts    # skills.json handling
│   ├── lock-manager.ts     # skills.lock handling
│   ├── installer.ts        # Multi-agent installation (symlink/copy)
│   ├── agent-registry.ts   # Agent type definitions and paths
│   └── skill-parser.ts     # SKILL.md parsing
├── types/            # TypeScript type definitions
└── utils/            # Shared utilities (fs, git, logger)
```

### Core Module Responsibilities

- **SkillManager**: Main orchestrator integrating all components
- **GitResolver**: Parse skill refs (`github:user/repo@v1.0.0`), resolve versions, build repo URLs
- **CacheManager**: Global cache operations using degit
- **ConfigLoader**: Read/write `skills.json`
- **LockManager**: Read/write `skills.lock` for version locking
- **Installer**: Handle multi-agent installation to paths like `.cursor/skills/`, `.claude/skills/`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kanyun-inc/reskill](https://github.com/kanyun-inc/reskill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
