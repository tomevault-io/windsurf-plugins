---
trigger: always_on
description: > **For AI coding assistants** (OpenCode, Cursor, Copilot, etc.): This file is the
---

# AGENTS.md

> **For AI coding assistants** (OpenCode, Cursor, Copilot, etc.): This file is the
> primary context source for AI assistants working on this codebase. Use it to
> understand project structure, coding conventions, and build commands.

> **For human contributors**: See [CONTRIBUTING.md](CONTRIBUTING.md) for the
> contribution guide.

## Project Overview

Rust-based AI gateway proxy supporting OpenAI, Anthropic, Gemini, and DeepSeek APIs. Built with Axum for HTTP, Tokio for async runtime, and etcd for configuration storage.

Includes a React-based admin UI (in `ui/`) for managing models, API keys, and a playground for testing chat completions.

## Build, Lint, and Test Commands

### Build
```bash
cargo build           # Debug build
cargo build --release # Release build
```

### Run
```bash
RUST_LOG=info cargo run
```

### UI Development
```bash
cd ui
pnpm install --frozen-lockfile    # Install dependencies
pnpm dev        # Start dev server
pnpm build      # Build for production
pnpm lint       # Run ESLint
pnpm format     # Format with Prettier
pnpm typecheck  # Type check without emit
pnpm preview    # Preview production build
```

### Lint
```bash
cargo clippy --all-targets --all-features --locked -- -D warnings
```
Clippy warnings are treated as errors. Fix all warnings before committing.

### Test
```bash
cargo test                             # Run all tests
cargo test --verbose                   # Run tests with verbose output
cargo test --test api                  # Run specific test file (tests/api.rs)
cargo test test_crud                   # Run specific test by name
cargo test --test admin::models_api    # Run tests in specific module
cargo test -- --nocapture              # Show test output
```

### E2E Test
```bash
pnpm -C tests install --frozen-lockfile  # Install e2e dependencies
pnpm -C tests test                       # Run all e2e tests
```

### Format
```bash
cargo fmt          # Format all code
cargo fmt -- --check  # Check formatting without changes
```

## Code Style Guidelines

### Imports

Imports are auto-organized by `rustfmt` with these rules (see `rustfmt.toml`):
- `reorder_imports = true` — Sort imports alphabetically
- `imports_granularity = "Crate"` — Merge imports from same crate
- `group_imports = "StdExternalCrate"` — Group: std → external crates → local

```rust
// Standard library first
use std::sync::Arc;

// External crates (alphabetical)
use anyhow::Result;
use axum::{Json, extract::State};
use serde::{Deserialize, Serialize};
use tokio::select;

// Local modules last
use crate::config::entities::Model;
```

### Naming Conventions

- **Types/Structs/Enums**: `PascalCase` (e.g., `ProviderConfig`, `ChatCompletionError`)
- **Functions/Methods**: `snake_case` (e.g., `chat_completions`, `create_provider`)
- **Constants/Statics**: `SCREAMING_SNAKE_CASE` (e.g., `MODELS_PATTERN`, `SCHEMA_VALIDATOR`)
- **Modules**: `snake_case` (e.g., `chat_completions`, `rate_limit`)
- **Local variables**: `snake_case`

### Error Handling

Use `thiserror` for library/domain errors, `anyhow` for application errors:

```rust
// Domain errors with thiserror
#[derive(Debug, Error)]
pub enum ProviderError {
    #[error("Not implemented")]
    NotYetImplemented,
    #[error("API error {0}: {1}")]
    ServiceError(http::StatusCode, String),
    #[error("Request error: {0}")]
    RequestError(#[from] reqwest::Error),
}

// Application code uses anyhow::Result
pub async fn create_provider(config: &ProviderConfig) -> Result<Box<dyn Provider>> {
    // ...
}
```

Error types should implement `IntoResponse` for Axum handlers:

```rust
impl IntoResponse for AuthError {
    fn into_response(self) -> Response {
        match self {
            AuthError::MissingApiKey => (
                http::StatusCode::UNAUTHORIZED,
                Json(json!({ "error": { "message": "Missing API key" } })),
            ).into_response(),
        }
    }
}
```

### Async Patterns

- Use `tokio` as the async runtime
- Async functions: `async fn`
- Async tests: `#[tokio::test]`
- Traits with async methods: `#[async_trait]`

```rust
#[async_trait]
pub trait Provider: Send + Sync {
    async fn chat_completion(&self, request: ChatCompletionRequest) 
        -> Result<ChatCompletionResponse, ProviderError>;
    async fn chat_completion_stream(&self, request: ChatCompletionRequest) 
        -> Result<BoxStream<'static, Result<ChatCompletionChunk, ProviderError>>, ProviderError>;
    async fn embedding(&self, request: EmbeddingRequest) 
        -> Result<EmbeddingResponse, ProviderError>;
}
```

### Tracing

Use `fastrace` for distributed tracing:

```rust
#[fastrace::trace]
pub async fn chat_completions(...) -> Result<Response, ChatCompletionError> {
    // Function is automatically traced
}

#[fastrace::trace(short_name = true)]
pub fn create_provider(config: &ProviderConfig) -> Box<dyn Provider> {
    // Short name in trace spans
}
```

### Documentation

Use `///` for doc comments on public items:

```rust
/// Creates a new provider instance based on the configuration.
pub fn create_provider(config: &ProviderConfig) -> Box<dyn Provider> {
    // ...
}
```

## Testing Patterns

### Test Organization

- Integration tests in `tests/` directory

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [api7/aisix-archived](https://github.com/api7/aisix-archived) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
