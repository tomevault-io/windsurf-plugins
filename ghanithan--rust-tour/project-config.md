---
trigger: always_on
description: This file provides development guidance for working with code in this repository.
---

# CLAUDE.md

This file provides development guidance for working with code in this repository.

## Project Overview
This is Rust Tour, an interactive Rust tutorial that provides progressive, exercise-based education following "The Rust Programming Language" book structure. The platform uses a hybrid approach with GitHub Repository + Codespaces as the primary delivery method.

## Architecture Documentation
For comprehensive understanding of the system architecture, code editing flows, WebSocket communication, and technical implementation details, refer to:

📋 **[Technical Architecture Documentation](docs/plan/TECHNICAL_DOCUMENTATION.md)**

This document provides detailed information about:
- Complete code editing, saving, and testing flow
- WebSocket architecture and real-time features  
- Terminal PTY integration
- Exercise management system
- Progress tracking system
- API reference and message protocols

## Development Commands

### Project Setup
```bash
# Initialize the project structure
./scripts/setup.sh

# Set up development container (if using Codespaces)
# The .devcontainer/ configuration handles this automatically

# Start Rust Tour (interactive menu)
./scripts/welcome.sh

# Start development mode (Vite + Rust server)
./scripts/run.sh dev

# Start Rust server only (development mode)
cargo run --package rust-tour --no-default-features

# Start Rust server only (with download capability for testing)
./scripts/run.sh server
```

### Testing and Validation
```bash
# Run all tests for exercises
cargo test

# Run tests for specific exercise
cd exercises/ch01_getting_started/ex01_hello_world && cargo test

# Check exercise solution
./scripts/check-exercise.sh ch01 ex01

# Run all exercise validations
./scripts/run.sh test

# Lint code (Clippy integration)
cargo clippy -- -D warnings

# Format code
cargo fmt

# Test Docker build
docker build -t rust-tour-test .

# Test CLI help system
cargo run --package rust-tour -- --help

# Test with custom options
cargo run --package rust-tour -- --port 8080 --debug-websocket
```

### Publishing to crates.io
```bash
# Build with embedded assets and download features
cargo build --release --package rust-tour --features "embed-assets,download-exercises"

# Test the published binary workflow
./target/release/rust-tour

# Publish (dry run)
cargo publish --package rust-tour --dry-run

# Actual publish
cargo publish --package rust-tour
```

### Docker Deployment
```bash
# Build production Docker image
docker build -t rust-tour .

# Run with port forwarding and progress persistence
docker run -d \
  --name rust-tour \
  -p 3000:3000 \
  -v $(pwd)/progress:/app/progress \
  rust-tour

# Development with exercise volume mount
docker run -d \
  --name rust-tour-dev \
  -p 3000:3000 \
  -v $(pwd)/exercises:/app/exercises:ro \
  -v $(pwd)/progress:/app/progress \
  rust-tour

# View logs and manage container
docker logs -f rust-tour
docker stop rust-tour && docker rm rust-tour
```

### Progress Tracking
```bash
# Update progress (automated via scripts)
./scripts/progress-tracker.py update ch01 ex01

# View current progress
cat progress.json
```

## Architecture

### Core Structure
The project follows a chapter-based exercise structure:
- `exercises/chXX_topic/` - Individual chapter exercises
- `solutions/` - Reference solutions for all exercises
- `scripts/` - Automation scripts for testing and progress tracking
- `web-server/` - Rust web server with integrated terminal support
- `progress.json` - Git-trackable progress file

### Exercise Framework
Each exercise follows this pattern:
- `src/main.rs` or `src/lib.rs` - Student implementation area
- `tests/` - Automated test cases
- `hints.md` - Progressive hint system (3 levels)
- Individual `Cargo.toml` for exercise-specific dependencies

### Exercise Types
1. **Code Completion** - Fill missing parts in skeleton code
2. **Bug Fixing** - Fix compilation/logic errors
3. **From Scratch** - Complete implementation from specs
4. **Code Review** - Analyze and improve existing code
5. **Performance Challenges** - Optimization exercises with benchmarks

## Development Workflow

### Creating New Exercises
1. Create chapter directory: `exercises/chXX_topic_name/`
2. Add exercise subdirectory: `exYY_exercise_name/`
3. Include: `src/`, `tests/`, `hints.md`, `Cargo.toml`
4. Update chapter README with exercise list
5. Add automated tests in `tests/` directory
6. Create progressive hints (beginner → intermediate → advanced)

### Exercise Validation
- All exercises must pass `cargo test`
- Include both unit tests and integration tests
- Use `cargo clippy` for code quality
- Performance exercises should include benchmarks
- Test cases should cover edge cases and common mistakes

### Progress System
- JSON-based tracking in `progress.json`
- Automatic updates via exercise completion scripts
- Tracks: completion status, time spent, difficulty rating, hints used
- Supports skill tree visualization and adaptive difficulty

## Testing Strategy
- Unit tests for individual functions/modules
- Integration tests for complete exercise solutions
- Benchmark tests for performance challenges
- Automated CI via `.github/workflows/test.yml`
- Clippy linting integrated into validation pipeline

## File Naming Conventions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ghanithan/rust-tour](https://github.com/ghanithan/rust-tour) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
