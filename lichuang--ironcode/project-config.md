---
trigger: always_on
description: All code must pass `cargo fmt`:
---

# AGENTS.md

## Code Quality Requirements

### Formatting

All code must pass `cargo fmt`:

```bash
cargo fmt --all
```

Check formatting without modifying files:
```bash
cargo fmt --all -- --check
```

### Linting

All code must pass clippy with strict warnings:

```bash
cargo clippy --all-features -- -D warnings
```

This includes:
- No unused imports
- No dead code (unless explicitly marked with `#[allow(dead_code)]`)
- No clippy warnings of any kind

## Code Style Guidelines

### 1. Type Import Rules

**DO NOT** use long path references to types directly in code:

```rust
// ❌ Wrong
pub fn to_openai_tool(&self) -> async_openai::types::chat::ChatCompletionTools {
    // ...
}

// ❌ Wrong
pub fn process_response(response: async_openai::types::chat::CreateChatCompletionResponse) {
    // ...
}
```

**MUST** use `use` to import types at the top of the file, then use short names:

```rust
// ✅ Correct
use async_openai::types::chat::ChatCompletionTools;

pub fn to_openai_tool(&self) -> ChatCompletionTools {
    // ...
}

// ✅ Correct
use async_openai::types::chat::CreateChatCompletionResponse;

pub fn process_response(response: CreateChatCompletionResponse) {
    // ...
}
```

### 2. Import Grouping Rules

Group imports in the following order:

1. Standard library (`std::`)
2. Third-party crates
3. Internal modules (`crate::`)

```rust
use std::collections::HashMap;
use std::path::PathBuf;

use anyhow::{Context, Result};
use serde::{Deserialize, Serialize};

use crate::tools::Tool;
use crate::utils::string::display_width;
```

### 3. Type Renaming

If type names conflict or are too long, use `as` to rename them:

```rust
use async_openai::types::chat::ChatCompletionTools as OpenAITool;
use async_openai::types::chat::FunctionObject as OpenAIFunction;
```

### 4. Exceptions

The following situations allow using full paths:

- When full path is needed in type definitions or documentation comments
- When macros require full paths
- When two modules have types with the same name and need to be distinguished

```rust
// Allowed: In doc comments to specify type origin
/// Converts to [`async_openai::types::chat::ChatCompletionTools`] format

// Allowed: Distinguish between same-named types
fn convert(a: crate::tools::Tool, b: async_openai::types::chat::Tool) {
    // ...
}
```

### 5. Path Types

Use `&Path` instead of `&PathBuf` for function parameters:

```rust
// ❌ Wrong
pub fn load_config(path: &PathBuf) -> Config { }

// ✅ Correct
use std::path::Path;
pub fn load_config(path: &Path) -> Config { }
```

### 6. Let Chains

Prefer let-chains over nested `if let`:

```rust
// ❌ Wrong
if let Some(value) = option {
    if condition {
        // ...
    }
}

// ✅ Correct
if let Some(value) = option && condition {
    // ...
}
```

### 7. Error Handling

Use `is_err()` instead of pattern matching:

```rust
// ❌ Wrong
if let Err(_) = result {
    // ...
}

// ✅ Correct
if result.is_err() {
    // ...
}
```

### 8. Dead Code

Mark intentionally unused code with `#[allow(dead_code)]`:

```rust
#[allow(dead_code)]
pub fn unused_but_keep_for_api() { }

#[allow(dead_code)]
pub struct ReservedForFutureUse { }
```

### 9. IO Errors

Use `std::io::Error::other()` for custom IO errors:

```rust
// ❌ Wrong
return Err(std::io::Error::new(
    std::io::ErrorKind::Other,
    format!("Failed: {}", e),
));

// ✅ Correct
return Err(std::io::Error::other(format!("Failed: {}", e)));
```

### 10. Redundant Closures

Use function references instead of closures:

```rust
// ❌ Wrong
items.iter().map(|e| serde_json::to_string(e))

// ✅ Correct
items.iter().map(serde_json::to_string)
```

### 11. File Open Options

Explicitly specify truncate behavior:

```rust
// ❌ Wrong
let file = OpenOptions::new()
    .create(true)
    .write(true)
    .open(&path)?;

// ✅ Correct
let file = OpenOptions::new()
    .create(true)
    .truncate(false)
    .write(true)
    .open(&path)?;
```

### 12. Unwrap Or Default

Use `unwrap_or_default()` instead of match:

```rust
// ❌ Wrong
match fs::read_to_string(&path) {
    Ok(content) => content,
    Err(_) => String::new(),
}

// ✅ Correct
fs::read_to_string(&path).unwrap_or_default()
```

## Tool Implementation Guidelines

### Tool Architecture

Tools are implemented following a handler pattern inspired by codex-rs:

```
src/tools/
├── mod.rs           # Tool definitions, ToolHandler trait, registries
├── loader.rs        # Loading tools from Markdown files
└── handlers/        # Tool implementations
    ├── mod.rs       # Re-exports all handlers
    └── read_file.rs # ReadFile tool implementation
```

### Implementing a New Tool

1. **Define the tool in `prompts/tools/{tool_name}.md`**:

```markdown
---
name: ToolName
description: Description of what the tool does
---

## Parameters

```json
{
  "type": "object",
  "properties": {
    "param1": {
      "type": "string",
      "description": "Description of param1"
    }
  },
  "required": ["param1"]
}
```
```

2. **Create a handler in `src/tools/handlers/{tool_name}.rs`**:

```rust
//! ToolName tool handler.

use async_trait::async_trait;
use serde::Deserialize;

use crate::tools::{
    parse_arguments, ToolError, ToolHandler, ToolInvocation, ToolKind, ToolOutput,
};

/// Handler for the ToolName tool
pub struct ToolNameHandler;

/// Arguments for the ToolName tool

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lichuang/ironcode](https://github.com/lichuang/ironcode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
