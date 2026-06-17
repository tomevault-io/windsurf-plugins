---
trigger: always_on
description: This document provides instructions for agentic coding systems working on the ztlgr Zettelkasten TUI application.
---

# AI Agent Guidelines for ztlgr

This document provides instructions for agentic coding systems working on the ztlgr Zettelkasten TUI application.

## Current Direction: LLM Wiki Integration

ztlgr is evolving from a standalone Zettelkasten TUI into an **LLM-maintained personal
knowledge base** following the "LLM Wiki" pattern. See `docs/ROADMAP-LLM-WIKI.md` for
the full implementation plan.

Key concepts:
- **`.skills/`** directory in each vault provides schema + workflows for LLM agents
- **`raw/`** directory holds immutable source material
- **`index.md`** is an auto-generated catalog of all wiki pages
- **`log.md`** is a chronological activity log
- The LLM does the grunt work (summarizing, cross-referencing, filing); humans curate and direct

Active branch: `feat/llm-wiki-integration`

## 📋 STATUS TRACKING PROTOCOL

**CRITICAL: After each successful commit, you MUST update docs/STATUS.md with:**

1. **Feature/Fix Implemented**: Mark items completed in the relevant Priority section
2. **Test Count**: Update passing tests count from `cargo test --lib` output
3. **Next Sprint Focus**: Add next 3 items to work on
4. **Blockers/Notes**: Document any blockers or architectural decisions

**docs/STATUS.md Structure**:
```markdown
## ✅ PRIORITY [N] ([PHASE]) - [STATUS]

### ✅ Completed:
- ✅ **Feature Name** (description + tests added)
- ✅ **Feature Name 2** 

### Commits:
- `commit-hash` - Feature description (X tests)

## 🟠 NEXT STEPS (Next Sprint)

### [Sprint Name]:
- [ ] Task 1
- [ ] Task 2
```

**Update Format**:
- Use checkmarks (✅) for completed items
- Include test count in parentheses: `(X tests added)`
- Keep NEXT STEPS with 3-5 prioritized items
- Update `cargo test --lib` count after each commit

**When to Update**:
- ✅ After each git commit that adds features/fixes
- ✅ When changing priorities or blocking issues appear
- ❌ NOT after minor formatting/doc-only changes

---

### Development Workflow

```bash
# Format code (run BEFORE commit)
cargo fmt --all

# Run clippy linting
cargo clippy --all-features -- -D warnings

# Run all unit tests
cargo test --lib

# Run single specific test
cargo test test_command_parser --lib -- --nocapture

# Build release binary
cargo build --release

# Development build
cargo run
```

### Makefile Shortcuts

```bash
make fmt                  # Format all code
make lint                 # Run clippy on all features
make test-unit            # Run unit tests (cargo test --lib)
make build                # Build release binary
make check                # Run fmt, lint, and test (pre-commit verification)
make clean                # Clean all build artifacts
```

### Pre-Commit Checklist

All commits MUST pass:

1. `cargo fmt --all` (no diffs allowed)
2. `cargo clippy --all-features -- -D warnings` (zero warnings)
3. `cargo test --lib` (all tests passing)
4. `cargo build` (release build succeeds)

Failure in any step means the commit cannot proceed.

---

## Code Style Guidelines

### General Principles

- **Rust Edition**: 2021 Edition with async support
- **Error Handling**: Use `Result<T>` return types exclusively; propagate errors with `?` operator
- **Naming**: Snake_case for functions/variables, PascalCase for types/traits
- **Documentation**: Public APIs require rustdoc comments with examples

### Imports

```rust
// Order: standard library, external crates, internal modules
use std::fmt;
use std::ops::Range;

use crossterm::event::{KeyCode, KeyEvent};
use ratatui::Frame;

use crate::config::Theme;
use crate::error::Result;
use super::GenericModal;
```

**Rules**:
- Group imports by category (std, external, internal)
- Use absolute paths for external crates and internal modules
- Use relative paths (`super::`, `crate::`) for internal hierarchy
- One import per line for clarity
- Remove unused imports (clippy enforces this)

### Type Definitions

```rust
// Public type aliases for ergonomics
pub type Result<T> = std::result::Result<T, ZtlgrError>;

// Prefer enums over booleans for state
#[derive(Debug, Clone, Copy, PartialEq, Eq)]
pub enum Mode {
    Normal,
    Insert,
    Search,
    Command,
}

// Add traits as needed
#[derive(Debug, Clone)]
pub struct Note {
    id: String,
    content: String,
    // ...
}
```

### Error Handling

```rust
use thiserror::Error;

#[derive(Debug, Error)]
pub enum ZtlgrError {
    #[error("Database error: {0}")]
    Database(#[from] rusqlite::Error),
    
    #[error("Note not found: {0}")]
    NotFound(String),
}

// Usage in functions
pub fn fetch_note(id: &str) -> Result<Note> {
    let note = db.get_note(id)
        .map_err(|_| ZtlgrError::NotFound(id.to_string()))?;
    Ok(note)
}

// Never use `.unwrap()` or `.expect()` in library code
// Always propagate errors or provide meaningful fallbacks
```

### Testing

```rust
#[cfg(test)]
mod tests {
    use super::*;
    use tempfile::TempDir;

    #[test]
    fn test_command_parser_help() {
        let cmd = CommandParser::parse("help");
        assert_eq!(cmd, Command::Help);
    }

    #[test]
    fn test_command_parser_with_args() {
        let cmd = CommandParser::parse("rename New Title");
        assert_eq!(cmd, Command::Rename("New Title".to_string()));
    }
    
    // Use tempfile for filesystem tests

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bakudas/ztlgr](https://github.com/bakudas/ztlgr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
