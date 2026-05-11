---
trigger: always_on
description: All documentation for the Armature project must be generated in the `docs/` directory following these standards.
---


# Documentation Standards

All documentation for the Armature project must be generated in the `docs/` directory following these standards.

## Directory Structure

All documentation files go directly in the `docs/` root directory. **Do NOT create subfolders.**

```
docs/
├── README.md                    # Documentation index
├── getting-started.md           # Getting started guide
├── auth-guide.md                # Authentication guide
├── cache-guide.md               # Cache guide
├── cron-guide.md                # Cron guide
├── queue-guide.md               # Queue guide
├── deployment-guide.md          # Deployment guide
└── *.md                         # All other documentation
```

**Important:** Do NOT create subdirectories like `guides/`, `modules/`, or `examples/`. All `.md` files belong in `docs/` root.

## File Naming

- Use **lowercase with hyphens**: `my-feature-guide.md`
- Be **descriptive**: `oauth2-providers-guide.md` not `oauth.md`
- Use **.md extension** for all Markdown files
- Avoid abbreviations unless widely understood

### Good Examples ✅

- `websocket-sse-guide.md`
- `authentication-guide.md`
- `rate-limiting-configuration.md`

### Bad Examples ❌

- `WS_SSE.md` (uppercase, abbreviation)
- `auth.md` (too generic)
- `guide-1.md` (not descriptive)

## Documentation Requirements

### Every Feature Must Have Documentation

When adding a new feature, you MUST create corresponding documentation in `docs/`:

1. **Feature Guide** (`docs/<feature>-guide.md`)
   - Overview of the feature
   - Key concepts
   - Configuration options
   - Step-by-step instructions
   - Code examples
   - Best practices
   - Troubleshooting

2. **API Reference** (inline code docs)
   - Rust doc comments (`///`)
   - Examples in doc comments
   - Clear parameter descriptions

3. **Code Examples** (`examples/` directory - code only)
   - Working code example
   - Comments explaining key parts
   - README if complex

## Documentation Format

### Markdown Structure

```markdown
# Title

Brief one-paragraph introduction.

## Table of Contents

- [Section 1](#section-1)
- [Section 2](#section-2)

## Overview

High-level explanation of what this is and why it exists.

## Features

- ✅ Feature 1
- ✅ Feature 2
- ✅ Feature 3

## Usage

### Basic Example

\`\`\`rust
// Working code example
use armature::prelude::*;

#[tokio::main]
async fn main() {
    // Example code
}
\`\`\`

### Advanced Example

More complex usage...

## Configuration

Detailed configuration options...

## Best Practices

1. Practice one
2. Practice two

## Common Pitfalls

- ❌ Don't do this
- ✅ Do this instead

## API Reference

Link to generated API docs or inline reference.

## Summary

Quick recap of key points.
```

### Required Sections

Every guide must include:

1. **Title** - Clear, descriptive
2. **Overview** - What and why
3. **Features** - Bullet list of capabilities
4. **Usage** - At least one working example
5. **Best Practices** - Dos and don'ts
6. **Summary** - Quick reference

## Code Examples

### Requirements

- **Must be runnable** without errors
- **Include necessary imports**
- **Add comments** for non-obvious code
- **Use realistic scenarios**
- **Show error handling**

### Good Example ✅

```rust
use armature_queue::*;

#[tokio::main]
async fn main() -> Result<(), QueueError> {
    // Connect to Redis
    let queue = Queue::new("redis://localhost:6379", "default").await?;

    // Enqueue a job
    let job_id = queue.enqueue(
        "send_email",
        serde_json::json!({
            "to": "user@example.com",
            "subject": "Welcome!"
        })
    ).await?;

    println!("Job enqueued: {}", job_id);
    Ok(())
}
```

### Bad Example ❌

```rust
// Incomplete, won't compile
let queue = Queue::new("redis://localhost:6379");
queue.enqueue("send_email", data);
```

## Inline Code Documentation

### Rust Doc Comments

```rust
/// Brief one-line description.
///
/// More detailed explanation of what this does,
/// including any important details.
///
/// # Arguments
///
/// * `param1` - Description of param1
/// * `param2` - Description of param2
///
/// # Returns
///
/// What this function returns and when.
///
/// # Errors
///
/// Possible error conditions and what causes them.
///
/// # Examples
///
/// ```
/// use armature_queue::Queue;
///
/// # async fn example() -> Result<(), QueueError> {
/// let queue = Queue::new("redis://localhost:6379", "default").await?;
/// # Ok(())
/// # }
/// ```
///
/// # Panics
///
/// Conditions under which this panics (if any).
pub async fn example_function(param1: String, param2: i32) -> Result<String, Error> {
    // Implementation
}
```

### Module-Level Documentation

```rust
//! Job queue module for background processing.
//!
//! This module provides a Redis-backed job queue system with
//! automatic retries, priorities, and scheduled jobs.
//!
//! # Examples
//!
//! ```no_run
//! use armature_queue::*;
//!
//! # async fn example() -> Result<(), QueueError> {
//! let queue = Queue::new("redis://localhost:6379", "default").await?;
//! queue.enqueue("task", serde_json::json!({})).await?;
//! # Ok(())
//! # }
//! ```
```

## Documentation Types

### 1. Getting Started Guides


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [quinnjr/armature](https://github.com/quinnjr/armature) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->
