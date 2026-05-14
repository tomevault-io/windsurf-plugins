---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

LazyCelery is a terminal UI for monitoring and managing Celery workers and tasks, inspired by lazydocker/lazygit. This is a Rust project with a fully functional architecture and UI framework. Currently implementing real Celery protocol integration to replace mock data systems.

## Development Commands

```bash
# Core development commands using mise:
mise run dev                            # Run with auto-reload (auto-starts Redis)
mise run test                           # Run all tests
mise run test-watch                     # Run tests in watch mode
mise run fmt                            # Format code
mise run lint                           # Lint code (clippy)
mise run audit                          # Security audit
mise run pre-commit                     # Run all checks before committing

# Setup and environment:
mise run setup                          # Setup development environment
mise run redis-start                    # Start Redis server via Docker
mise run redis-stop                     # Stop Redis server

# Git hooks setup (run once after cloning):
git config core.hooksPath .githooks     # Enable pre-commit quality checks

# Specific commands for development:
cargo test --test integration           # Run specific test file
cargo test worker::tests               # Run specific module tests
cargo run -- --broker redis://localhost:6379/0  # Run with specific broker
```

## Architecture Overview

### Core Design Principles
1. **Single App State**: All application state lives in `src/app.rs` in the `App` struct
2. **Async Broker Operations**: All broker interactions use async/await with Tokio
3. **Trait-Based Broker Interface**: Common trait for Redis/AMQP implementations
4. **Widget-Based UI**: Each UI component is a separate widget with render() and handle_key()

### Data Flow Architecture
```
Broker (Redis/AMQP) → Async Broker Client → App State → UI Widgets → Terminal
                           ↑                     ↓
                           └── Background refresh task (1 second interval)
```

### Key Architectural Decisions
- **Error Handling**: Custom error types with `thiserror` for broker operations, `anyhow::Result` for main()
- **State Updates**: Background task updates data every second, UI thread only handles rendering
- **Event Loop**: Separate UI event handling from data updates to prevent blocking
- **UI Refresh**: Limited to 10 FPS to reduce CPU usage

### Module Responsibilities
- `broker/`: Async broker clients implementing common `Broker` trait (Redis fully functional, AMQP placeholder)
- `models/`: Complete data structures (Worker, Task, Queue) with serde serialization
- `ui/widgets/`: Fully implemented widgets (workers.rs, tasks.rs, queues.rs) with navigation and search
- `ui/events.rs`: Keyboard event handling including search mode and vim-style navigation
- `app.rs`: Central state management with tab navigation, selection, and search functionality
- `main.rs`: Complete CLI parsing, async tokio runtime, and event loop coordination
- `config.rs`: TOML configuration support with broker connection settings
- `error.rs`: Custom error types using `thiserror` for broker and application errors

## Current Implementation Status

**Fully Implemented (100%):**
- Complete terminal UI with workers/queues/tasks widgets and navigation
- Application architecture with centralized state management
- Configuration system with TOML support
- Error handling with custom error types
- Complete data models with serialization
- **Redis broker client with real Celery protocol integration**
- **Task retry/revoke functionality implemented**
- **Queue purge operations with confirmation dialogs**
- **Comprehensive CI/CD pipeline with automated releases**
- **Performance optimizations and caching**
- **Pre-commit hooks and quality gates**

**Partially Implemented (75%):**
- AMQP/RabbitMQ broker client (placeholder structure exists, needs real implementation)
- Advanced queue management features (basic operations work)

**Not Implemented (0%):**
- Real-time worker monitoring and stats collection
- Advanced task scheduling and workflow management
- Plugin system for custom brokers

## Current Development Focus

The project has successfully completed the MVP phase for Redis integration. Current status:
- **Branch**: `feature/mvp-core-monitoring` (ready for merge)
- **Redis Integration**: ✅ Complete with real Celery protocol parsing
- **Task Management**: ✅ Retry, revoke, and queue purge operations
- **CI/CD**: ✅ Fully automated releases and quality checks
- **Performance**: ✅ Optimized with aggressive caching

**Next priorities for future development:**
1. Complete AMQP/RabbitMQ broker implementation
2. Add real-time worker monitoring and health checks
3. Implement advanced task filtering and search
4. Add support for task routing and scheduling
5. Create plugin architecture for custom brokers

## Testing Strategy

**Comprehensive test suite exists for:**
- Model serialization/deserialization (complete)
- UI widget state management and navigation (complete)
- Configuration loading from TOML files (complete)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Fguedes90/lazycelery](https://github.com/Fguedes90/lazycelery) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
