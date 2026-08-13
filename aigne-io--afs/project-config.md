---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**AFS (Agentic File System)** is a virtual file system abstraction layer that provides a unified, file-system-like interface for AI agents to access various types of storage backends. It enables agents to interact with different data sources through a consistent, path-based API.

## Development Commands

```bash
# Package management (pnpm workspaces)
pnpm install              # Install all dependencies
pnpm build                # Build all packages using Turbo
pnpm dev                  # Run dev mode for all packages
pnpm lint                 # Run Biome linter + type checking
pnpm format               # Auto-fix formatting issues with Biome
pnpm test                 # Run all tests
pnpm test:coverage        # Run tests with coverage
pnpm check-types          # Type check all packages
```

### Per-package commands
```bash
cd packages/core          # Navigate to a package
pnpm build               # Build single package
pnpm test                # Run package tests
pnpm check-types         # Type check package
```

## Tooling

- **Package manager**: pnpm with workspaces
- **Build system**: Turbo (turborepo)
- **Runtime**: bun (for development and tests)
- **Linting/Formatting**: Biome
- **Testing**: bun:test
- **Type checking**: TypeScript 5.9.2

### Type Checking

Always run `pnpm check-types` to verify TypeScript compilation before committing:

```bash
cd packages/cli && pnpm check-types       # Check single package (cd into it)
pnpm --filter @aigne/afs-cli check-types  # Check single package by name
pnpm --filter ./packages/cli check-types  # Check single package by path
pnpm check-types                          # Check all packages from root
```

This catches type errors that may not surface during runtime or tests.

## Package Structure

```
packages/
├── core/                      # Core AFS implementation (@aigne/afs)
├── explorer/                  # AFS explorer utilities
└── compute-abstraction/       # Cross-cloud compute instance abstraction

providers/                     # AFS provider implementations (categorized)
├── core/                      # Core data providers
│   ├── json/                  # JSON/YAML virtual FS (@aigne/afs-json)
│   ├── toml/                  # TOML virtual FS (@aigne/afs-toml)
│   ├── kv/                    # Key-value store (@aigne/afs-kv)
│   ├── memory/                # In-memory FS (@aigne/afs-memory)
│   ├── markdown/              # Markdown structure (@aigne/afs-markdown)
│   ├── registry/              # Provider registry (@aigne/afs-registry)
│   ├── vault/                 # Secret vault (@aigne/afs-vault)
│   ├── workspace/             # Workspace manager (@aigne/afs-workspace)
│   └── proc/                  # Process manager (@aigne/afs-proc)
├── basic/                     # Basic I/O providers
│   ├── fs/                    # Local filesystem (@aigne/afs-fs)
│   ├── http/                  # HTTP endpoints (@aigne/afs-http)
│   ├── sandbox/               # Sandboxed FS (@aigne/afs-sandbox)
│   ├── ash/                   # Agent shell (@aigne/afs-ash)
│   └── ...
├── platform/                  # Cloud platform providers
│   ├── s3/                    # AWS S3 (@aigne/afs-s3)
│   ├── gcs/                   # Google Cloud Storage (@aigne/afs-gcs)
│   ├── ec2/                   # AWS EC2 (@aigne/afs-ec2)
│   ├── gce/                   # Google Compute Engine (@aigne/afs-gce)
│   ├── dns/                   # Cloud DNS (@aigne/afs-dns)
│   ├── git/                   # Git repository (@aigne/afs-git)
│   ├── github/                # GitHub Issues/PRs (@aigne/afs-github)
│   ├── sqlite/                # SQLite database (@aigne/afs-sqlite)
│   └── ...
├── messaging/                 # Messaging providers (Slack, Discord, etc.)
├── iot/                       # IoT providers (Home Assistant, Tesla, etc.)
├── cost/                      # Cost tracking providers
├── ai/                        # AI service providers
└── runtime/                   # Runtime providers (MCP, UI, etc.)

scripts/                       # Build and utility scripts
typescript-config/             # Shared TypeScript configurations
```

## Core Concepts

### AFS (Agentic File System)

The core abstraction layer that provides a unified interface for mounting and accessing different storage backends. All providers implement the AFS interface to ensure consistent behavior.

### Providers

Providers are modules that can be mounted into AFS at specific paths. Each provider implements:

- **List operations**: List files/directories with metadata
- **Read operations**: Read file contents
- **Write operations** (optional): Modify or create files
- **Search operations** (optional): Search within mounted data

Available providers:
- `AFSFS` - Access local filesystem directories
- `AFSGit` - Access Git repository branches and files
- `AFSJSON` - Navigate JSON/YAML files as virtual filesystems
- `AFSSQLite` - SQLite-based storage backend

### Mount Paths

Providers are mounted at paths following the pattern `/modules/{provider-name}/{path}`. The mount system allows:
- Multiple providers at different paths

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AIGNE-io/afs](https://github.com/AIGNE-io/afs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
