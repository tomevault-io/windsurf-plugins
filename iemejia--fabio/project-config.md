---
trigger: always_on
description: Fabio is an agent-first CLI for Microsoft Fabric, written in Rust (edition 2024, MSRV 1.96). It manages 55+ command groups with 370+ subcommands across all Fabric artifact types. All output is structured JSON by default.
---

# Copilot Instructions for fabio

## Overview

Fabio is an agent-first CLI for Microsoft Fabric, written in Rust (edition 2024, MSRV 1.96). It manages 55+ command groups with 370+ subcommands across all Fabric artifact types. All output is structured JSON by default.

## Language & Framework Conventions

### Rust

- **Edition 2024**, `rust-version = "1.96"`
- Clippy: `pedantic` + `nursery` lints enabled, zero warnings required
- `unsafe_code = "forbid"` — no unsafe code allowed anywhere
- Use `thiserror` for error types, `anyhow` for propagation in command handlers
- Use `clap` derive macros for CLI argument parsing
- Use `tokio` for async runtime (full features)
- Use `serde` derive for serialization — all API response structs need `#[derive(Deserialize, Serialize)]`
- Use `serde_json` with `preserve_order` feature (required for JSON key-order sensitive APIs)
- Use `reqwest` with `rustls` (no OpenSSL dependency)
- Prefer `.lines()` over splitting on `\n` (handles CRLF)

### Windows Compatibility (Critical)

All code must work on Windows:
- Use `Path::new().join()` — never hardcode `/` in filesystem paths
- Use `dirs::home_dir()` — never read `HOME`/`USERPROFILE` manually
- Use `.lines()` for text parsing — handles CRLF transparently
- No Unix-specific APIs (`std::os::unix`, signals, etc.)
- `.gitattributes` enforces LF line endings in the repo

## Architecture Patterns

### Command Structure

Every command module follows this pattern:
1. Define a `{Name}Command` enum with clap `Subcommand` derive in `src/commands/{name}.rs`
2. Implement an `execute(cli: &Cli, cmd: &{Name}Command)` async function
3. Register the module in `src/commands/mod.rs` (add `pub mod` + match arm)
4. Add the variant to `src/cli.rs` `Command` enum

### Output Envelope

- Lists: `{"data":[...],"count":N}` with optional `"continuationToken"`
- Objects: `{"data":{...}}`
- Errors (stderr): `{"error":{"code":"...","message":"...","hint":"..."}}`
- Use `render_list()` / `render_list_with_token()` / `render_object()` from `src/output.rs`
- Never print raw text to stdout — always use the output helpers

### Error Handling

- Use `ErrorCode` enum from `src/errors.rs` for machine-readable codes
- Include `hint` field with valid values or corrected command examples
- Map HTTP status codes: 401→`AuthRequired`, 403→`Forbidden`, 404→`NotFound`, 409→`Conflict`, 429→`RateLimited`
- Use `enrich_forbidden()` to add required role hints on 403 errors
- Not-found errors should include `fabio <group> list` suggestions

### HTTP Client

- All API calls go through `FabricClient` in `src/client.rs`
- GET/POST/PUT/PATCH/DELETE helpers handle auth token injection
- LRO polling: `post(..., poll: true)` follows `Location`/`x-ms-operation-id` headers (2s interval, 120s max)
- OneLake DFS: 3-step upload (create → append → flush)
- OneLake Blob: server-side copy via `x-ms-copy-source` header
- Parallel operations: `src/parallel.rs` provides concurrent execution with rate-limit retry
- Two auth scopes: Fabric (`https://analysis.windows.net/powerbi/api/.default`) and Storage (`https://storage.azure.com/.default`)
- SQL auth: `require_sql_auth()` for TDS connections

### Global Flags

All commands must respect these (handled by output helpers):
- `--output json|table|plain` — format selection
- `--query` — dot-notation field projection
- `--quiet` — suppress stdout
- `--dry-run` — preview mutations via `dry_run_guard()`
- `--limit` — client-side truncation
- `--all` — auto-paginate all pages
- `--continuation-token` — resume from token
- `--profile` — use named profile defaults

### Throttling Reduction

Prefer bulk/batch APIs to minimize throttling:
- `item bulk-create`, `item bulk-delete` for multi-item operations
- Use single list API + client-side filter rather than N individual show calls
- Parallel execution framework handles rate-limit retry automatically

## Test Conventions

- Unit tests: inline `#[cfg(test)]` modules using `wiremock` for HTTP mocking
- E2E tests: `tests/e2e_*.rs` files, require live Fabric tenant via `FABIO_TEST_*` env vars
- E2E tests use `#[ignore]` attribute (run with `cargo test -- --ignored`)
- Test harness: `tests/common/mod.rs` provides `TestConfig` with workspace/lakehouse IDs
- Use `assert_cmd` + `predicates` for CLI binary assertions
- Test names follow pattern: `test_{command}_{subcommand}_{scenario}`
- Currently: 202 unit tests + 505 E2E tests = 707 total

## Code Style

- No comments on obvious code — only clarify non-obvious logic
- Prefer exhaustive `match` over `if let` chains
- Use `visible_alias` for common flag short forms (e.g., `--sw` for `--source-workspace`)
- Keep modules focused: one file per command group
- Imports: group by std → external crates → internal crates, separated by blank lines

## Commit Conventions

- Use `Assisted-by: Copilot:claude-opus-4.6` trailer
- Prefix with type: `feat`, `fix`, `chore`, `docs`, `test`, `refactor`
- Example: `feat(lakehouse): add sync command for file delta transfer`

## API Reference

- Fabric REST: `https://api.fabric.microsoft.com/v1`
- OneLake DFS: `https://onelake.dfs.fabric.microsoft.com`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iemejia/fabio](https://github.com/iemejia/fabio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-12 -->
