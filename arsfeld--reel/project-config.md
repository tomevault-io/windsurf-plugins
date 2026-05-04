---
trigger: always_on
description: Reel is a native, cross-platform media player application written in Rust that brings your Plex and Jellyfin libraries to the desktop with a premium, Netflix-like experience. The project uses:
---

# GitHub Copilot Instructions for Reel

## Project Overview

Reel is a native, cross-platform media player application written in Rust that brings your Plex and Jellyfin libraries to the desktop with a premium, Netflix-like experience. The project uses:

- **Language**: Rust 2021 edition with strict quality standards
- **UI Framework**: Relm4 (reactive component-based) with libadwaita for GNOME integration
- **Architecture**: Offline-first with SQLite cache, multi-backend support (Plex, Jellyfin, Local files)
- **Build System**: Cargo with Nix flakes for reproducible development environment
- **Platform**: Linux (primary), macOS (in development)

## Development Environment Setup

**CRITICAL**: This project uses Nix flakes for dependency management. All development commands must be run within the Nix development shell:

```bash
# Enter the development environment (REQUIRED FIRST)
nix develop

# Then proceed with cargo commands
cargo build
cargo run
```

The Nix environment provides:
- Correct Rust toolchain version
- GStreamer plugins and development libraries
- MPV/libmpv2 development libraries (Linux)
- GTK4 and libadwaita development files
- All system dependencies properly configured

## Build Commands

```bash
# Check compilation without building
cargo check

# Build debug version
cargo build

# Build release version
cargo build --release

# Run the application
cargo run

# Run with specific features (e.g., macOS without MPV)
cargo run --no-default-features --features gstreamer
```

**Important**: Always use `cargo check` or `cargo build` to verify compilation. The last line of output shows the actual number of errors/warnings.

## Code Quality

### Formatting
```bash
# Format all Rust code (REQUIRED before committing)
cargo fmt

# Check formatting without modifying files
cargo fmt -- --check
```

Pre-commit hooks automatically run `cargo fmt`, so code should always be formatted.

### Linting
```bash
# Run Clippy lints (REQUIRED before committing)
cargo clippy

# Run Clippy on all targets
cargo clippy --all-targets

# Run Clippy with specific features
cargo clippy --no-default-features --features gstreamer
```

**Standards**: Fix all Clippy warnings. Do not use `#[allow]` attributes without strong justification.

### Testing
```bash
# Run all tests
cargo test

# Run specific test
cargo test test_name

# Run tests with output
cargo test -- --nocapture

# Run tests for specific package
cargo test -p reel
```

## Architecture Guidelines

### Module Structure

- `src/main.rs` - Entry point and application initialization
- `src/app/` - Application core with Relm4 AsyncComponent
- `src/ui/` - All UI components (pages, dialogs, factories, shared)
  - Use AsyncComponents for data-heavy pages
  - Use Factory pattern for collections (media grids, lists)
  - Use Worker components for background tasks
- `src/services/` - Service layer with brokers, commands, and core services
- `src/backends/` - Media server integrations (all implement `MediaBackend` trait)
- `src/db/` - Database layer with SeaORM entities and repositories
- `src/player/` - Media playback (MPV for Linux, GStreamer for macOS)
- `src/workers/` - Background worker components
- `src/models/` - Data models and type-safe identifiers

### Key Design Patterns

1. **Relm4 Reactive Architecture**
   - Use AsyncComponents for pages with data loading
   - Use `#[tracker::track]` for efficient change tracking
   - Use Factory pattern (FactoryVecDeque) for dynamic collections
   - Use Worker components for background tasks
   - Use MessageBroker for inter-component communication
   - Use Command pattern for structured async operations

2. **Backend Abstraction**
   - All media sources implement the `MediaBackend` trait
   - Never hardcode backend-specific logic outside backend implementations
   - Use the repository pattern for data access

3. **Offline-First Architecture**
   - Always load from SQLite cache first
   - Update UI immediately with cached data
   - Sync from backend in background
   - Handle offline scenarios gracefully

4. **Three-Tier Caching**
   - Memory Cache (LRU, 1000 items)
   - Database Cache (SQLite with SeaORM)
   - Backend API (source-specific)

### Code Style

- **No backward compatibility code**: Tackle the root cause, not symptoms
- **No fallbacks**: They introduce confusion and errors
- **Minimal comments**: Code should be self-documenting; add comments only for complex logic
- **Type safety**: Use newtype patterns for IDs and domain concepts
- **Error handling**: Use `Result` types, provide context with error chains
- **Async/await**: Use Tokio runtime, avoid blocking operations

## Platform-Specific Notes

### Linux
- Default features: Both MPV and GStreamer backends
- MPV is the primary/preferred player backend
- Full hardware acceleration support

### macOS
- Use `--no-default-features --features gstreamer` (MPV has OpenGL issues)
- GStreamer is the only supported backend
- Some features may be limited compared to Linux

## Testing Guidelines

- Write tests for new functionality
- Follow existing test patterns in the repository

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [arsfeld/reel](https://github.com/arsfeld/reel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
