---
trigger: always_on
description: Rust-based LLM proxy implementing the Open Responses API specification for agentic workflows. Provides a unified interface to multiple LLM providers (OpenAI, Anthropic, Google) with load balancing, cost tracking, and observability.
---

# Aura LLM Gateway

## Project Overview

Rust-based LLM proxy implementing the Open Responses API specification for agentic workflows. Provides a unified interface to multiple LLM providers (OpenAI, Anthropic, Google) with load balancing, cost tracking, and observability.

## Tech Stack

- **Language**: Rust (2021 edition)
- **Web Framework**: Axum
- **Database**: PostgreSQL (SQLx), Redis
- **Async Runtime**: Tokio
- **Serialization**: Serde
- **Error Handling**: thiserror, anyhow
- **Logging**: tracing
- **HTTP Client**: reqwest

## Project Structure

```
/crates/
  aura-types/     # Shared type definitions (Open Responses API types)
    src/
      compression.rs   # Compression config types
      consistency.rs   # Response consistency types
      validation.rs    # Response validation types
  aura-core/      # Core business logic (providers, routing, caching)
    src/
      compression/     # Prompt compression (TOON, YAML, AISP, JSON)
      provider/        # LLM providers (OpenAI, Anthropic, Google)
      router/          # Smart routing with health tracking
  aura-proxy/     # Main server binary (Axum routes, middleware)
  aura-db/        # Database models and queries (SQLx)
/apps/
  chat/           # React chat playground (Vite + React 18)
  landing/        # Marketing landing page (Vite + React, MDX docs)
/docs/            # Documentation
  api/            # API documentation (Markdown)
/migrations/      # SQLx database migrations
/sdks/
  python/         # Python SDK (aura-llm on PyPI)
```

## Development Commands

```bash
# Build
cargo build                    # Build all crates
cargo build --release          # Build optimized binary

# Test
cargo test                     # Run all tests
cargo test -p aura-core        # Test specific crate
cargo test -- --nocapture      # Show println output

# Run
cargo run -p aura-proxy        # Run the proxy server
RUST_LOG=debug cargo run -p aura-proxy  # With debug logging

# Lint & Format
cargo clippy                   # Lint all crates
cargo clippy --fix             # Auto-fix lint issues
cargo fmt                      # Format code
cargo fmt -- --check           # Check formatting

# Database (requires sqlx-cli)
sqlx migrate run               # Run migrations
sqlx migrate add <name>        # Create new migration

# Docker
docker-compose up -d           # Start local stack
docker-compose logs -f         # Follow logs

# Frontend Apps
cd apps/chat && npm run dev    # Run chat playground (port 3000)
cd apps/landing && npm run dev # Run landing page (port 3001)
cd apps/chat && npm run build  # Build chat app for production
```

## Key Conventions

### Error Handling
- Use `thiserror` for library error types in `aura-types` and `aura-core`
- Use `anyhow` for application errors in `aura-proxy`
- Always provide context with `.context()` or custom error variants
- Never use `.unwrap()` in production code (use `.expect()` with clear message if truly infallible)

### Logging
- Use `tracing` macros (`info!`, `debug!`, `error!`, `warn!`)
- Never use `println!` or `eprintln!`
- Add structured fields: `info!(provider = %name, latency_ms = %ms, "request completed")`
- Use spans for request correlation

### Async Patterns
- All async functions should be cancellation-safe
- Use `tokio::select!` carefully with proper branch handling
- Prefer `tokio::spawn` for background tasks over blocking
- Always set timeouts on external calls

### Shared State
- Use `Arc<T>` for state shared across handlers
- Use `Arc<RwLock<T>>` only when mutation is required
- Prefer message passing over shared mutable state

### Provider Pattern
```rust
#[async_trait]
pub trait Provider: Send + Sync {
    fn name(&self) -> &str;
    fn models(&self) -> &[&str];
    async fn complete(&self, request: Request) -> Result<Response, ProviderError>;
    async fn complete_stream(&self, request: Request) -> Result<EventStream, ProviderError>;
}
```

### Testing
- Unit tests in same file as implementation (`#[cfg(test)] mod tests`)
- Integration tests in `/tests/` directory
- Use `#[tokio::test]` for async tests
- Mock external APIs with `wiremock`
- Use `insta` for snapshot testing of JSON responses

### Compression Module
The compression system reduces token usage with multiple strategies:
- **JSON Minification** (15-30% savings): Whitespace removal, key shortening
- **TOON** (40-60% savings): Token-Oriented Object Notation for uniform arrays
- **YAML** (10-25% savings): Fewer delimiters for nested objects
- **AISP** (clarity boost): AI Symbolic Protocol for mathematical notation

```rust
use aura_core::compression::{SmartCompressor, Compressor};

let compressor = SmartCompressor::builder()
    .auto_select(true)
    .build();
let result = compressor.compress(input)?;
```

### Validation Module
Response validation reduces hallucinations:
- `logprobs`: Token-level confidence (OpenAI only)
- `best_of_n`: Generate N responses, select best
- `self_consistency`: Pick most consistent answer
- `confidence_threshold`: Reject below threshold

### Feedback API
Adaptive few-shot learning from user feedback:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [UmaiTech/aura-llm-gateway](https://github.com/UmaiTech/aura-llm-gateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
