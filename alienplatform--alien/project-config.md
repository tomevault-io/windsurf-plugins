---
trigger: always_on
description: **Always follow the coding guidelines in `AGENTS.md`** — it covers error handling, testing, imports, and all Rust conventions for this project.
---

# Guidelines

**Always follow the coding guidelines in `AGENTS.md`** — it covers error handling, testing, imports, and all Rust conventions for this project.

- **First principles thinking** — ask "why" until you truly understand the problem
- **Root cause analysis** — don't patch symptoms, find and fix the real bug
- **No hacks** — implement things properly, even if it takes longer
- **Quality over speed** — write code your future self will thank you for
- **Keep it simple** — the best code is the code you don't write
- **Leave code better than you found it** — improve something small in every change
- **Explicit over implicit** — if it's not obvious, make it obvious
- **Fail fast, fail loud** — surface errors immediately, don't swallow them

# Rust Guidelines

## Dependencies

Use `workspace = true` for all dependencies defined in the workspace `Cargo.toml`. Never duplicate versions.

## Code Style

### Import Organization

Place all `use` statements at the top of the file or module. Never scatter imports throughout the code.

```rust
// Good: All imports at the top
use std::collections::HashMap;
use tokio::sync::RwLock;
use tracing::{info, warn};

async fn do_work() {
    // ...
}

// Bad: Import in the middle of the code
async fn do_work() {
    use std::collections::HashMap; // Don't do this
    // ...
}
```

### Consolidate Imports

Avoid repeating the same module path multiple times. Group related imports together.

```rust
// Good
use alien_core::{Platform, ResourceType, StackConfig};

// Bad
use alien_core::Platform;
use alien_core::ResourceType;
use alien_core::StackConfig;
```

### Serde Serialization

Always use `camelCase` for JSON serialization and document fields:

```rust
#[derive(Debug, Clone, Serialize, Deserialize)]
#[serde(rename_all = "camelCase")]
pub struct DeploymentConfig {
    /// Unique identifier for the deployment
    pub deployment_id: String,
    /// Target environment (production, staging, etc.)
    pub target_environment: String,
}
```

### Use Builders for Multiple Parameters

Use the `bon` crate's builder pattern for functions or structs with more than a few parameters instead of long parameter lists.

---

## Preferred Crates

| Purpose | Crate | Notes |
|---------|-------|-------|
| Time & Date | `chrono` | Never use `time` crate |
| Logging | `tracing` | Use `info!`, `warn!`, `error!`, `debug!` macros |
| Serialization | `serde` | Always with `rename_all = "camelCase"` |
| Async Runtime | `tokio` | Our standard async runtime |
| Error Handling | `alien-error` | Our custom error library (see below) |

---

## Git

### Commits

Use [Conventional Commits](https://www.conventionalcommits.org/): `<type>: <short summary>`

**Types**: `feat`, `fix`, `refactor`, `chore`, `docs`, `test`, `ci`, `perf`

- Imperative mood, lowercase, no period, max ~72 chars
- Body (optional): explain *why*, not *what*; wrap at 72 chars
- Breaking changes: add `!` after type (e.g. `feat!: remove v1 API`)

### Branches

- Branch from `main`; use `<type>/<short-description>` (e.g. `feat/azure-containers`, `fix/duplicate-cleanup`)
- Keep branches short-lived; rebase on `main` before merging

### Pushing and PRs

- Never force-push to `main`
- Push feature branches and open a PR for review
- Squash-merge PRs into `main`

---

## Alien Errors

### Why Not `thiserror` or `anyhow`?

We use our custom `alien-error` library instead of the popular alternatives because:

1. **Structured metadata**: Every error carries machine-readable `code`, `retryable`, `internal`, and `http_status_code` fields—essential for API responses and automated retry logic.

2. **Error chaining with context**: Like `anyhow`, we support chaining errors, but each layer preserves structured metadata, not just string messages.

3. **API-ready**: Errors serialize to JSON with consistent schema, and the `internal` flag controls whether sensitive details are exposed to external clients.

4. **Inheritance**: The `retryable` and `internal` flags can be inherited from source errors using `"inherit"`, so wrapper errors don't need to guess.

### The Two Aspects of Error Handling

There are two distinct aspects to working with errors:

1. **Designing errors** - Defining error enums for your crate/module
2. **Using errors** - Propagating and creating errors in your code

---

## Designing Error Enums

Each crate should have an `error.rs` file that defines its error types.

### Design Principles

1. **Name variants after the problem, not dependencies**
   Variants should reflect domain-level issues, not internal implementation details or external libraries.

2. **Embed actionable, self-contained data**
   Include sufficient context directly in variants so callers don't need to downcast or guess. Almost always include a human-readable `message` field.

3. **Provide useful context for recovery**
   Include fields like paths, URLs, IDs, or parameters to assist in debugging and handling errors effectively.

4. **Use a generic catch-all variant when needed**
   If there are too many possible errors, provide a generic variant (e.g., `Other` or `InfrastructureError`) for unexpected or uncommon cases.

5. **Always document each variant concisely**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alienplatform/alien](https://github.com/alienplatform/alien) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
