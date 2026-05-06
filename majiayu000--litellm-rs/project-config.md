---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Version Management

### Automated Version Bumping (CI/CD)

Version bumping is **fully automated** via GitHub Actions. On every push to `main`:

1. **Analyzes commits** using [Conventional Commits](https://conventionalcommits.org/):
   - `feat:` → Minor bump (0.1.x → 0.2.0)
   - `fix:`, `perf:`, `refactor:` → Patch bump (0.1.3 → 0.1.4)
   - `feat!:`, `BREAKING CHANGE:` → Major bump (0.x.x → 1.0.0)

2. **Auto-updates**:
   - `Cargo.toml` version
   - `CHANGELOG.md` with categorized changes
   - Creates git tag `v0.1.4`

3. **Triggers release pipeline** → builds binaries, Docker images, publishes to crates.io

### Commit Message Format

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

**Types**: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `chore`

**Examples**:
```bash
git commit -m "feat(router): add weighted load balancing"     # → minor
git commit -m "fix(auth): resolve JWT validation issue"       # → patch
git commit -m "feat!: redesign provider interface"            # → major
```

### Version Info in Code

Access build information programmatically:
```rust
use litellm_rs::{VERSION, GIT_HASH, full_version, build_info};

println!("Version: {}", VERSION);           // "0.1.4"
println!("Full: {}", full_version());       // "0.1.4-a1b2c3d"
println!("Info: {}", build_info());         // "0.1.4-a1b2c3d (built 1704067200 with rustc 1.87)"
```

### Manual Release (if needed)

```bash
# 1. Update version
cargo set-version 0.1.4

# 2. Update CHANGELOG.md

# 3. Commit and tag
git add Cargo.toml Cargo.lock CHANGELOG.md
git commit -m "chore(release): bump version to 0.1.4"
git tag -a v0.1.4 -m "Release v0.1.4"
git push && git push --tags
```

### Quick Commands
```bash
make version          # Show current version info
cargo pkgid           # Show package identifier with version
```

## Essential Commands

### Development Commands
- **Start development**: `make dev` or `cargo run` (auto-loads config/gateway.yaml)
- **Build**: `cargo build --all-features` 
- **Test**: `cargo test --all-features`
- **Lint**: `cargo clippy --all-targets --all-features -- -D warnings`
- **Format**: `cargo fmt --all`
- **Quick start**: `make start` (fastest way to start the gateway)

### Testing Commands
- **All tests**: `make test`
- **Unit tests only**: `make test-unit` 
- **Integration tests**: `make test-integration`
- **Test coverage**: `make test-coverage`
- **Single test**: `cargo test <test_name> --all-features`

### Development Services
- **Start dev services**: `make dev-services` (starts PostgreSQL, Redis)
- **Stop dev services**: `make dev-stop`
- **Database migration**: `make db-migrate`
- **Reset database**: `make db-reset`

## Architecture Overview

This is a **high-performance AI Gateway** written in Rust that provides OpenAI-compatible APIs with intelligent routing across 100+ AI providers. It's a Rust implementation of the Python LiteLLM library, designed for production environments requiring maximum throughput and minimal latency.

### Core Components

**Gateway Architecture**: Modular, trait-based design with dependency injection
- `src/core/` - Central orchestrator and business logic
- `src/server/` - Actix-web HTTP server with middleware pipeline
- `src/auth/` - Multi-layered authentication (JWT, API keys, RBAC)
- `src/core/providers/` - Pluggable provider system (OpenAI, Anthropic, Azure, Google, etc.)
- `src/core/router/` - Intelligent routing with multiple strategies
- `src/core/mcp/` - MCP Gateway for external tool integration (90 tests)
- `src/core/a2a/` - A2A Protocol for agent-to-agent communication (48 tests)
- `src/storage/` - Multi-backend storage (PostgreSQL, Redis, S3, Vector DB)
- `src/monitoring/` - Observability (Prometheus, tracing, health checks)

### Key Design Patterns
- **Async-first**: All I/O is non-blocking using Tokio
- **Trait-based abstractions**: Pluggable components via traits
- **Error handling**: Comprehensive error types with context preservation
- **Configuration**: Type-safe config models with Default implementations
- **No backward compatibility**: Break old formats freely - prioritize clean architecture over legacy support

### Provider Integration
- **Unified Provider trait**: Common interface for all AI providers
- **Format conversion**: Automatic translation between OpenAI and provider-specific APIs
- **Health monitoring**: Per-provider health checks and failover
- **Cost calculation**: Built-in token counting and cost estimation

### Request Flow
1. HTTP Request → Authentication → Authorization → Router → Provider → Response
2. Middleware pipeline handles auth, logging, metrics, and transformations
3. Intelligent routing selects optimal provider based on health, latency, cost

## Configuration

- **Main config**: `config/gateway.yaml` (auto-loaded by default)
- **Example config**: `config/gateway.yaml.example`
- **Environment variables**: Override config values with `${ENV_VAR}` syntax
- **Config validation**: `make config-validate`

## Important Files

- `src/main.rs` - Application entry point

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [majiayu000/litellm-rs](https://github.com/majiayu000/litellm-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
