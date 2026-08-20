---
trigger: always_on
description: > This file defines the rules, principles, and idioms that ALL code in this
---

# AGENTS.md — Rust Development Constitution

> This file defines the rules, principles, and idioms that ALL code in this
> repository must follow. It applies to human contributors and AI coding agents
> equally. When in doubt, consult The Rust Programming Language Book
> (https://doc.rust-lang.org/book/) — the canonical reference for every
> rule below.

---

## Code Navigation Rules

ALWAYS use octocode MCP tools before reading files directly:
- Use `semantic_search` to find relevant code by meaning
- Use `view_signatures` to understand file structure
- Use `graphrag` to explore dependencies between files
- Use `structural_search` for AST-level pattern search (replaces grep/rg)

NEVER:
- Run grep, rg, find to locate code — use semantic_search instead
- Read entire files to understand structure — use view_signatures instead
- Guess file locations — use graphrag overview first

WORKFLOW for any task:
1. graphrag overview → understand project structure
2. semantic_search → find relevant files
3. view_signatures → inspect structure of found files
4. Read only specific sections if needed

## 0. Meta-Rules for AI Agents

- **Read this file fully before writing or refactoring any code.**
- **Do NOT change public APIs without explicit user instruction.**
- **Do NOT introduce new dependencies without approval.**
- **Do NOT use `unsafe` unless explicitly requested and justified in comments.**
- **Do NOT fix formatting issues — `rustfmt` handles that automatically.**
- **Every change must compile cleanly:** `cargo check && cargo clippy --all-targets --all-features -- -D warnings`
- **Preserve behavior.** Refactoring must not change observable output or semantics.
- **Always use `#[serde(deny_unknown_fields)]` on all tool param structs** so the JSON Schema shows `additionalProperties: false` — unknown fields are caught by serde, not silently ignored.
- **Map ALL errors to native rmcp error types** via `From<MailMcpError> for McpError` in `error.rs`:
  - `Validation(msg)` → `McpError::invalid_params(msg)` — tool input validation failures
  - `MessageNotFound` / `AttachmentNotFound` → `McpError::invalid_params(...)` — resource not found
  - Everything else → `McpError::internal_error(...)` — server/database failures
- **Tool functions return `Err(MailMcpError)`, not `Ok(Response::error(...))`** — the generic `call_typed_tool` handler auto-converts via `?`. Never wrap errors in a successful `Ok` response body.

### Quality Gate (mandatory before every commit)

> **Zero tolerance for warnings, formatting drift, or test failures.**
> CI is the minimum, not the goal — catch everything locally first.

```bash
# 1. Format — MUST pass before anything else
cargo fmt --all -- --check

# 2. Lint — zero warnings tolerated (all-targets covers test code, matching CI)
cargo clippy --all-targets --all-features -- -D warnings

# 3. Test — all tests must pass, no exceptions
cargo test

# 4. Docs — documentation must build clean
cargo doc --no-deps 2>&1 | grep -q "^error" && exit 1

# 5. Full check — final sanity
cargo check
```

**Rules:**
- **Never** push code that produces a single clippy warning, compiler warning, or `dbg!()` / `todo!()` / `unimplemented!()`.
- **Always** run `cargo fmt --all` before committing. If CI fails on formatting, the commit is rejected.
- **All 4 gates** (fmt, clippy, test, doc) must pass before any push. No exceptions.
- If a test is flaky due to global state, **fix it** with a serialisation lock — never silence or ignore it.
- `#[allow(...)]` attributes are forbidden unless explicitly approved in a code review.

---

## 1. Project Setup

### Edition & Toolchain

```toml
# Cargo.toml
[package]
edition = "2024"  # Always use the latest stable edition (Rust 2024 as of 1.85+)
resolver = "3"
```

#### Required tooling

```bash
# Before every commit
cargo fmt            # Format all code
cargo clippy --all-targets --all-features -- -D warnings  # Treat all warnings as errors (incl. test code)
cargo test           # All tests must pass
cargo doc --no-deps  # Docs must build without warnings
```

#### Recommended dependencies (approved for use without asking)

| Crate | Purpose |
|---|---|
| rmcp | MCP Server framework (server, macros, transport-io) |
| tokio | Async runtime |
| serde + serde_json | Serialization/deserialization |
| thiserror | Derive Error for library error types |
| anyhow | Error handling at application boundaries only |
| schemars | JSON Schema generation from Rust types (paired with serde) |
| tracing + tracing-subscriber | Structured logging |
| rusqlite | SQLite access for Apple Mail envelope index |
| chrono | Date/time handling (with serde feature) |
| clap | CLI argument parsing |
| mail-parser | Email message parsing (emlx) |
| dirs | Platform-specific home directory resolution |
| walkdir | Filesystem recursion for mailbox discovery |
| lru | LRU cache for message/attachment lookups |
| scraper | HTML email content extraction |
| zip | .docx support (deflate feature only) |
| quick-xml | .xlsx support (serialize feature) |
| csv | CSV attachment parsing |
| lopdf | PDF attachment parsing |
| once_cell | Lazy-initialized globals |
| tempfile | Temporary directories in tests (dev-dependency only) |

## 2. Ownership & Borrowing

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [like-a-freedom/rusty_apple_mail_mcp](https://github.com/like-a-freedom/rusty_apple_mail_mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
