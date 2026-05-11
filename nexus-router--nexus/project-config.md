---
trigger: always_on
description: When editing code in this repository, you are working on **Nexus**, an AI router that aggregates MCP (Model Context Protocol) servers and LLMs. This system helps reduce tool proliferation by intelligently routing and indexing available tools.
---

# LLM Code Editing Guidelines for Nexus

When editing code in this repository, you are working on **Nexus**, an AI router that aggregates MCP (Model Context Protocol) servers and LLMs. This system helps reduce tool proliferation by intelligently routing and indexing available tools.

## Commit Messages

- Follow the [Conventional Commits 1.0.0](https://www.conventionalcommits.org/en/v1.0.0/) specification for every commit message.
- Use recent commits (`git log`) as a reference for tone and formatting if you need examples.
- Structure messages as `type(issue number): subject`, where
- Common `type` values include `feat`, `fix`, `chore`, `docs`, `refactor`, `test`, and `build`; pick the one that best matches the change.
- The issue number is a linear issue number, in the format of `gb-1234`. The user should give you this in the prompt, but if you don't have it, you can ask the user for it.
- Keep the subject in the imperative mood, ≤72 characters, and avoid punctuation at the end.

## Domain Context

- **MCP (Model Context Protocol)**: A protocol for connecting AI models with external tools and data sources
- **AI Router**: Nexus acts as an intelligent intermediary between LLMs and multiple MCP servers
- **Tool Indexing**: Uses Tantivy (full-text search engine) to create searchable indexes of available tools
- **Dynamic vs Static Tools**: Static tools are shared across users; dynamic tools require user authentication

## Key Technologies

- **Rust**: The primary programming language
- **Axum**: Web framework for HTTP routing and middleware
- **Tantivy**: Full-text search engine for tool indexing
- **Tokio**: Async runtime
- **Serde**: Serialization/deserialization for JSON/TOML
- **RMCP**: Rust MCP client/server implementation
- **Anyhow**: Error handling with context and backtrace
- **Reqwest**: HTTP client for external API calls
- **JWT-Compact**: JWT token handling for authentication
- **Tower/Tower-HTTP**: Middleware and service layers
- **Rustls**: TLS implementation for secure connections
- **Insta**: Snapshot testing framework
- **Clap**: Command-line argument parsing
- **Logforth**: Structured logging with tracing support
- **Docker Compose**: For integration testing with Hydra OAuth2 server
- **Governor**: Rate limiting with token bucket algorithm
- **Mini-moka**: In-memory caching for rate limit buckets
- **Redis**: Redis support for distributed rate limiting
- **Deadpool**: Connection pooling
- **OpenTelemetry**: Observability with metrics collection and OTLP export

## Rust Coding Guidelines

### Error Handling
Always handle errors appropriately - never silently discard them:

```rust
// Good: Propagate errors
let result = some_operation().await?;

// Good: Custom error handling
match some_operation().await {
    Ok(value) => process(value),
    Err(e) => handle_specific_error(e),
}

// Bad: Silent error discarding
let _ = some_operation().await;

// Bad: Panic on errors
let result = some_operation().await.unwrap();
```

### String Formatting
Use modern Rust string interpolation:

```rust
// Good
let message = format!("User {username} has {count} items");

// Bad
let message = format!("User {} has {} items", username, count);

// Good
assert!(
    startup_duration < Duration::from_secs(5),
    "STDIO server startup took too long: {startup_duration:?}",
);

// Bad
assert!(
    startup_duration < Duration::from_secs(5),
    "STDIO server startup took too long: {:?}",
    startup_duration
);

// Good
log::debug!("creating stdio downstream service for {name}");

// Bad
log::debug!("creating stdio downstream service for {}", name);
```

When accessing fields or calling methods, interpolation is not needed:

```rust
// Good: Direct field/method access
let message = format!("Status: {}", server.status());
let info = format!("User: {}", user.name);

// Bad: Unnecessary named interpolation
let message = format!("Status: {status}", status = server.status());
let info = format!("User: {name}", name = user.name);
```

And so on. You will find many places where these rules apply, not only for format! or log macros.

### Control Flow and Readability
Avoid nested if-lets and matches. Use let-else with early return to reduce indentation. Horizontal space is sacred and nested structures are hard to read:

```rust
// Good: Early return with let-else
let Some(user) = get_user() else {
    return Err(anyhow!("User not found"));
};

let Some(profile) = user.profile() else {
    return Ok(Response::default());
};

process_profile(profile);

// Bad: Nested if-let
if let Some(user) = get_user() {
    if let Some(profile) = user.profile() {
        process_profile(profile);
    }
}

// Good: Flat match with early returns
let config = match load_config() {
    Ok(cfg) => cfg,
    Err(e) => return Err(e.into()),
};

let parsed = match config.parse() {
    Some(p) => p,
    None => return Ok(Default::default()),
};

// Bad: Nested matches
match load_config() {
    Ok(cfg) => {
        match cfg.parse() {
            Some(p) => {
                // deeply nested logic
            }
            None => { /* ... */ }
        }
    }

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Nexus-Router/nexus](https://github.com/Nexus-Router/nexus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
