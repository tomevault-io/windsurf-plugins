---
trigger: always_on
description: Instructions for AI agents (Claude Code, Cursor, etc.) working in this repository.
---

# AGENTS.md

Instructions for AI agents (Claude Code, Cursor, etc.) working in this repository.

## Project Summary

**podkit** is a TypeScript toolkit for syncing music collections to iPod devices. It provides a CLI and library that handles collection diffing, transcoding (FLAC→AAC), metadata preservation, and artwork transfer.

**Status:** Active development

**Monorepo structure:**
```
packages/
├── demo/            # Animated GIF demo (VHS + mocked CLI build)
├── e2e-tests/       # End-to-end CLI tests (dummy + real iPod)
├── gpod-testing/    # Test utilities for iPod environments (no hardware needed)
├── ipod-db/         # Pure TypeScript iTunesDB/ArtworkDB parser (browser-compatible)
├── ipod-web/        # Virtual iPod UI — React + Jotai web component
├── libgpod-node/    # Native Node.js bindings for libgpod (C library)
├── podkit-core/     # Core sync logic, adapters, transcoding
├── podkit-cli/      # Command-line interface
├── podkit-docker/   # Docker image (Dockerfile, entrypoint, compose files)
├── test-fixtures/   # Test fixture generator (FLAC files with controllable metadata/artwork)
├── virtual-ipod-app/    # Tauri macOS app — frameless iPod-shaped window
└── virtual-ipod-server/ # Lima VM backend — USB gadget + REST/WebSocket API

tools/
├── demo/            # Live demo documentation for the virtual iPod system
├── gpod-tool/       # C CLI for iPod database operations
├── libgpod-macos/   # macOS build scripts for libgpod
└── lima/            # Lima VM configs (Debian, Alpine, virtual-ipod)

devices/             # Device documentation profiles (specs, capabilities, research)
```

## Quick Reference

### Commands

```bash
# Development (uses Bun)
bun install                      # Install dependencies
bun run dev                      # Run in development mode
bun run test                     # Run all tests (unit + integration)
bun run test:unit                # Run unit tests only
bun run test:integration         # Run integration tests only
bun run test:e2e                 # Run E2E tests (dummy iPod)
bun run test --filter podkit-core # Run tests for specific package
mise run lima:test                # Run tests on Debian + Alpine VMs

# Build
bun run build                    # Build all packages for Node.js

# Release
bunx changeset                   # Create a changeset for your changes
bunx changeset version           # Apply pending changesets (CI does this)
bun run compile                  # Build standalone CLI binary locally

# CLI
podkit sync --dry-run                   # Sync all collections (music + video)
podkit sync -t music -c main --dry-run  # Sync specific music collection
podkit sync -d myipod                   # Sync to named device
podkit sync -d /Volumes/iPod            # Sync to device by path
podkit device scan                      # Scan for connected iPods
podkit device info                      # Show device status
podkit device music --format json       # List music on device
```

### System Dependencies

**For end users:** Only FFmpeg is required. libgpod is statically linked into prebuilt binaries.

| Dependency | Debian/Ubuntu | macOS | Alpine | Required for |
|------------|---------------|-------|--------|--------------|
| FFmpeg | `ffmpeg` | `brew install ffmpeg` | `ffmpeg` | Users + developers |
| libgpod | `libgpod-dev` | Build from source (see `tools/libgpod-macos/`) | `libgpod-dev` (community) | Development only |
| GLib | `libglib2.0-dev` | `brew install glib` (installed as libgpod dep) | `glib-dev` | Development only |
| util-linux | Pre-installed | N/A | `lsblk` | Linux device manager |
| Lima | N/A | `brew install lima` | N/A | Cross-platform testing |

See [docs/developers/development.md](docs/developers/development.md) for full setup instructions.

## Documentation

The `docs/` directory is organized for web publication (Starlight-compatible). Read [agents/documentation.md](agents/documentation.md) for the full documentation map, file conventions, and maintenance guidelines.

## Feature Requests & GitHub Discussions

Feature requests are managed through GitHub Discussions (Ideas category), with links in the documentation and backlog tasks. **See [agents/feature-requests.md](agents/feature-requests.md) for the complete guide** covering:

- Creating, updating, and closing discussions via the GitHub API
- The current discussions registry (all feature discussions with numbers and URLs)
- Which doc files reference which discussions and how to update them
- Workflows for moving features between roadmap tiers
- How discussions, docs, and backlog tasks stay in sync

When working on anything related to feature requests, planned features, or the roadmap, read that file first.

## Task Management (Backlog.md)

This project uses Backlog.md for task management via MCP tools. **Never edit backlog files directly** — always use the MCP tools.

### When to Create Tasks

**Create a task** when work requires planning or decisions (investigating bugs, designing features, choosing approaches).

**Skip tasks** for trivial/mechanical changes (typos, version bumps, obvious one-line fixes).

### Workflow

1. **Search first:** Use `task_search` or `task_list` to find existing related work

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jvgomg/podkit](https://github.com/jvgomg/podkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
