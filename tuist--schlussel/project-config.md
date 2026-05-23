---
trigger: always_on
description: Schlussel is a cross-platform OAuth 2.0 library with PKCE and Device Code Flow support, now centered on a Rust workspace. It is designed for command-line applications and agent runtimes, with formula-driven provider definitions, token persistence, and automatic refresh.
---

# Claude Instructions for Schlussel

## Project Overview

Schlussel is a cross-platform OAuth 2.0 library with PKCE and Device Code Flow support, now centered on a Rust workspace. It is designed for command-line applications and agent runtimes, with formula-driven provider definitions, token persistence, and automatic refresh.

## Documentation Notes

- Keep `README.md` aligned with the current public API and CLI behavior.
- Keep public doc comments in `crates/schlussel/src/*.rs` accurate when changing interfaces.
- Keep `include/schlussel.h` aligned with `crates/schlussel-ffi` when the C-facing API changes.

## Core Architecture

### Key Modules

1. **PKCE Module** (`crates/schlussel/src/pkce.rs`)
   - Generates cryptographically secure code verifiers and challenges
   - Uses SHA256 for challenge generation
   - Base64 URL-safe encoding without padding

2. **Session Management** (`crates/schlussel/src/session.rs`)
   - Trait-based storage (`SessionStorage`)
   - Three built-in backends: `SecureStorage`, `FileStorage`, `MemoryStorage`
   - File-backed tokens for the CLI and keyring-backed secure storage for library users
   - Stable storage-key format: `{formula}:{method}:{identity}`

3. **OAuth Flow** (`crates/schlussel/src/oauth.rs`)
   - Device Code Flow (RFC 8628) for CLI applications
   - Authorization Code Flow with PKCE
   - Automatic browser opening and callback handling
   - Token refresh with an HTTP client
   - Provider presets and formula-driven configuration

4. **Token Refresher** (`crates/schlussel/src/oauth.rs`)
   - In-process locking (threads)
   - Cross-process locking (file-based)
   - Automatic token refresh (`get_valid_token`)
   - Proactive refresh with thresholds

5. **Callback Server** (`crates/schlussel/src/callback.rs`)
   - Local HTTP server for OAuth redirects
   - Random port assignment
   - HTML success/error pages

6. **Cross-Process Locking** (`crates/schlussel/src/lock.rs`)
   - File-based locks
   - RAII lock guards
   - Check-then-refresh coordination

7. **Formulas and Scripts**
   - Bundled provider formulas live in `src/formulas/*.json`
   - Script resolution lives in `crates/schlussel/src/script.rs`
   - The CLI surface lives in `crates/schlussel-cli/src/main.rs`

## Development Guidelines

### Code Style

- Follow Rust standard conventions (`cargo fmt`)
- Keep crate roots small and push implementation into focused modules
- Document public APIs when the usage is not obvious

### Commits

Follow the [Conventional Commits](https://www.conventionalcommits.org/) specification:
- `feat:` for new features
- `fix:` for bug fixes
- `docs:` for documentation changes
- `style:` for formatting changes
- `refactor:` for code refactoring
- `test:` for adding or updating tests
- `chore:` for maintenance tasks

### Testing

- Unit tests live next to the Rust modules they cover
- End-to-end CLI coverage lives under `e2e/` and runs through ShellSpec against a local OAuth server
- Run: `mise exec -- cargo test --workspace`
- Run: `shellspec`
- All tests must pass before committing

### Building

- Development: `mise exec -- cargo build --workspace`
- Run tests: `mise exec -- cargo test --workspace`
- Format: `mise exec -- cargo fmt`

### CI Requirements

All PRs must pass:
- Cargo tests on Ubuntu, macOS, Windows
- ShellSpec e2e coverage on Ubuntu
- `cargo fmt --check`

## Important Design Decisions

### 1. Security First

- **SecureStorage is the library default recommendation** for production hosts that want OS credential managers
- The CLI uses `FileStorage` so tokens can be enumerated, filtered, and deleted by key
- Always use PKCE for OAuth flows
- Cross-process locking prevents refresh races

### 2. Device Code Flow Priority

- Primary flow for CLI applications
- Simpler UX than a callback server in headless environments
- Works in remote and terminal-only setups
- Falls back to callback flow when Device Code is unavailable

### 3. Automatic Token Refresh

- `get_valid_token()` eliminates manual expiration checking
- Proactive refresh uses configurable thresholds
- Refreshes are safe across concurrent processes when locking is enabled

### 4. Provider Presets

- One-line configuration for common providers
- Reduces endpoint and redirect configuration mistakes
- Self-hosted instance support where applicable

### 5. Storage Abstraction

Three built-in backends:
- **SecureStorage**: Production (OS keychain/credential manager)
- **FileStorage**: CLI and development storage
- **MemoryStorage**: Testing

### 6. Cross-Process Coordination

- File-based locks operate at the refresh level instead of the storage-root level
- Refresh paths use a check-then-refresh pattern to avoid duplicate token exchanges
- File-backed token saves use atomic replacement

## Common Tasks

### Adding a New Provider Preset

1. Add the configuration in `crates/schlussel/src/oauth.rs` or the bundled formula set.
2. Add tests to verify endpoints and behavior.
3. Update `README.md` if the change is part of the public surface.

### Adding a New Storage Backend


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tuist/schlussel](https://github.com/tuist/schlussel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-23 -->
