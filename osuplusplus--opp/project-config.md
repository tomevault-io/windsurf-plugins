---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project Overview

OPP is a cross-platform osu! desktop toolkit built with Tauri 2, React 19, and Rust. It provides features including OAuth login, beatmap downloads, collection management, similarity matching, PP calculation, skin workshop, replay rendering, and live streaming integration. The project supports Windows x64 and Linux.

**Important**: This is an independent community project. Do not include Client Secrets, Tokens, credential export files, or app data in commits.

## Development Commands

### Frontend (React + Vite)
```bash
pnpm install              # Install dependencies
pnpm tauri dev            # Start development environment (includes Rust backend)
pnpm dev                  # Vite dev server only (without Tauri)
pnpm lint                 # ESLint with max-warnings 0
pnpm test                 # Run Vitest tests
pnpm test:watch           # Run tests in watch mode
pnpm build                # TypeScript compilation + Vite build
pnpm preview              # Preview production build
```

### Backend (Rust)
```bash
cargo fmt --manifest-path src-tauri/Cargo.toml -- --check
cargo clippy --manifest-path src-tauri/Cargo.toml --all-targets -- -D warnings
cargo test --manifest-path src-tauri/Cargo.toml --all-targets
```

### Release Build
```bash
pnpm tauri build          # Build production artifacts
# Windows: src-tauri/target/release/bundle/nsis/
# Linux: src-tauri/target/release/opp
```

## Architecture

### Data Flow
```text
React pages & components
  → features/*/api.ts (TanStack Query state boundary)
  → shared/lib/tauri.ts (typed command adapter layer)
  → Tauri command
  → Rust domain modules
  → osu! API / local files / system credentials / external apps
```

Frontend never directly accesses local files or credentials. Rust commands handle input validation, permission boundaries, and error conversion. Remote responses and local index data are passed through serialization contracts in `shared/types/osu.ts`.

### Frontend Structure
- `src/app/` — Application shell, routing, sidebar, title bar, global client/mode context
- `src/features/` — Business domain pages, query adapters, local components, pure logic
  - Each feature should not import internal state from another feature; use public APIs or small shared events
- `src/shared/components/` — Reusable components without business state
- `src/shared/lib/` — Formatting, style merging, Tauri command adapters
- `src/shared/types/` — Shared data contracts between frontend and backend

Large pages should place pure filtering/transformation logic in model files in the same directory, and split independent visual areas into components. Side effects should only be used for external synchronization, subscriptions, and resource cleanup, not for mirroring derivable state.

### Backend Structure

Rust backend organized by `domain / features / infrastructure`:

```text
src-tauri/src/
├── domain/                 # Shared, semantically stable OPP models
├── features/               # User-facing business capabilities
│   ├── account/           # OAuth, credentials, account caching
│   ├── collections/       # Collection sync, sharing, missing beatmap補齐
│   ├── danser/            # Local replay render queue
│   ├── local_analysis/    # Path detection, scanning, cache, resource analysis
│   ├── online_beatmaps/   # Online beatmap queries and downloads
│   ├── similarity/        # Similarity dataset, query, recommendations
│   ├── skin_workshop/     # Skin component merging and atomic writes
│   ├── live_render/       # Replay live preview (wgpu-based, Windows native window + other platforms canvas)
│   └── ...
├── infrastructure/        # osu! API, platform detection, storage, portable updates
├── tools/                 # Single-purpose tools without complex state
├── state.rs               # Dependency assembly and shared runtime state
├── commands.rs            # Tauri command registry
├── error.rs               # Unified error contract
├── lib.rs                 # App startup & lifecycle
└── main.rs                # Binary entry point
```

**Complex feature module structure**:
```text
<module>/
├── models.rs      # OPP domain data semantics
├── ports.rs       # Capability definitions (trait interfaces)
├── adapters/      # External system adapters (files, network, platform APIs)
├── service.rs     # Business flow orchestration
├── commands.rs    # Tauri command exposure
└── mod.rs         # Module declaration & export control
```

**Dependency direction**:
```text
commands → features → domain + ports ← adapters / infrastructure
```

- `domain/` does not depend on features or infrastructure
- Features can depend on domain and use infrastructure through explicit interfaces
- Cross-feature calls must use explicit `crate::features::<name>` paths
- Infrastructure handles external structure conversion, not business flows

### Module Organization Principles
1. **Models** unify data semantics (convert external structures like Stable, lazer, Realm, HTTP in adapters)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [osuplusplus/OPP](https://github.com/osuplusplus/OPP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
