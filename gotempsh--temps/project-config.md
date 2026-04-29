---
trigger: always_on
description: Guidance for Claude Code when working with the Temps codebase.
---

# CLAUDE.md

Guidance for Claude Code when working with the Temps codebase.
**Core philosophy: code that works safely, and when it fails, explains why comprehensively.**

## Critical Rules

### NEVER
- Access database directly from HTTP handlers -- ALWAYS use services
- Return untyped JSON (`serde_json::Value`) -- ALWAYS use typed structs
- Use `.context()` from anyhow -- ALWAYS use `.map_err()` with typed errors
- Use `.unwrap()` or `.expect()` in production code -- ALWAYS use `?` or explicit error handling
- Use `anyhow::Result` in service layer -- ALWAYS use typed error enums with `thiserror`
- Expose sensitive data (API keys, tokens) in responses -- ALWAYS mask them
- Create N+1 queries -- ALWAYS use JOINs for related data
- Leave the project in non-compilable state
- Use `#[tokio::main]` when integrating with pingora
- Use plain text logging -- ALWAYS use structured JSONL logging
- Create markdown documentation files unless explicitly requested
- Mark Docker tests with `#[ignore]` -- they MUST skip gracefully at runtime instead
- Create error types with generic messages -- ALWAYS include IDs, names, and operation context

### ALWAYS
- Run `cargo check --lib` after every modification
- New functionality must compile without warnings
- Write tests for all new functionality AND verify they run successfully
- Use structured logging with explicit log levels
- Use Conventional Commits: `type(scope): description`
- Use services for all business logic
- Implement pagination (default: 20, max: 100) and sorting (default: `created_at` DESC)
- Use typed error handling with proper propagation
- Follow the three-layer architecture pattern
- Keep tests in the same file as the code they test
- Return dates in ISO 8601 format with `Z` suffix
- Use `permission_guard!` macro for authorization in handlers
- Add audit logging for all write operations (CREATE, UPDATE, DELETE)
- Include contextual information (IDs, resource names, paths) in every error message

---

## Error Handling as First-Class Architecture

Error handling is the most critical aspect of this codebase. Every error must be typed, contextual, and traceable from origin to HTTP response.

### Error Propagation Chain

```
sea_orm::DbErr / std::io::Error / external error
    |
    v  (From<T> impl or map_err)
Domain Error Enum (per crate: BackupError, DeploymentError, etc.)
    |
    v  (From<DomainError> for Problem, defined in handler module)
Problem (RFC 7807 ProblemDetails)
    |
    v  (IntoResponse)
HTTP JSON response: application/problem+json
```

Every crate owns its error types. Errors flow upward through typed conversions, never through string coercion or anyhow wrapping.

### Defining Error Types

Every domain crate defines its own error enum. Error messages MUST include contextual identifiers.

```rust
use thiserror::Error;

#[derive(Error, Debug)]
pub enum BackupError {
    // GOOD: includes service_id, operation context, and original error
    #[error("Failed to encrypt parameter '{param_name}' for service {service_id}: {reason}")]
    EncryptionFailed { service_id: i32, param_name: String, reason: String },

    #[error("Backup {backup_id} not found in project {project_id}")]
    NotFound { backup_id: i32, project_id: i32 },

    #[error("Cannot delete service {service_id}: still linked to {project_count} project(s)")]
    HasLinkedProjects { service_id: i32, project_count: usize },

    #[error("Validation error: {message}")]
    Validation { message: String },

    #[error("Database error: {0}")]
    Database(#[from] sea_orm::DbErr),

    #[error("IO error: {0}")]
    Io(#[from] std::io::Error),

    #[error("S3 upload failed for backup {backup_id}: {reason}")]
    S3 { backup_id: i32, reason: String },
}
```

**Error type design rules:**
- Use structured fields (`{ service_id: i32, reason: String }`) over bare strings when the error has identifiers
- Use `#[from]` for automatic conversion of common error types (`sea_orm::DbErr`, `std::io::Error`)
- Use `#[error("...")]` messages that a developer can grep for and immediately understand what happened
- Every `NotFound` variant must include the ID that was searched for
- Every operation failure must include what was being operated on

**BAD error variants** (do not create these):
```rust
// These tell you NOTHING when they appear in logs
#[error("Database error")]         // Which database? What operation?
NotFound,                           // What wasn't found?
#[error("Operation failed")]       // Which operation?
#[error("Internal error: {0}")]    // Lazy catch-all
```

### Converting Database Errors

Every domain error implements `From<sea_orm::DbErr>` with semantic mapping:

```rust
impl From<sea_orm::DbErr> for BackupError {
    fn from(error: sea_orm::DbErr) -> Self {
        match error {
            sea_orm::DbErr::RecordNotFound(msg) => BackupError::NotFound {
                backup_id: 0, // When ID is lost, use the DbErr message
                project_id: 0,
            },
            sea_orm::DbErr::RecordNotInserted => BackupError::Validation {
                message: format!("Duplicate record: {}", error),
            },
            _ => BackupError::Database(error),
        }
    }
}
```

### Error Context: map_err over .context()


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gotempsh/temps](https://github.com/gotempsh/temps) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
