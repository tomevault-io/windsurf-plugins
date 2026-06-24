---
trigger: always_on
description: This file provides project-specific guidance for the Pick penetration testing connector.
---

# CLAUDE.md - Pick Project

This file provides project-specific guidance for the Pick penetration testing connector.

**Inherits from:** `/home/jtomek/Code/CLAUDE.md` (root configuration)

---

## Project Overview

Pick is a multi-platform penetration testing connector built with Dioxus (Rust). Each app instance IS a connector that registers with Strike48 and executes tools locally.

**Key technologies:**
- Rust (stable 1.92+)
- Dioxus (UI framework)
- Tokio (async runtime)
- Strike48 SDK (connector protocol)

---

## Development Environment

### Prerequisites

- Rust 1.92+ (required by egui dependencies)
- Cargo and rustup
- libpcap-dev (for packet capture features)
- libssl-dev (for TLS)
- protobuf-compiler (for Protocol Buffers)

### Launch Commands

```bash
# Headless connector (preferred for testing)
./run-pentest.sh headless dev

# Or with just
just run-headless-env
```

### Configuration

Pick reads its connection settings from a `.env` file at the repository root:

- `STRIKE48_HOST` — WebSocket URL of the Strike48 instance, e.g. `wss://your-tenant.example.com:443`
- `STRIKE48_TENANT` — Tenant identifier registered on that instance
- `MATRIX_API_URL` — HTTPS URL of the Strike48 API (legacy variable name, refers to Strike48's API endpoint)
- `MATRIX_TENANT_ID` — Same tenant identifier as `STRIKE48_TENANT` (legacy variable, kept for backward compatibility)
- `RUST_LOG` — Tracing filter, default `debug` for development

`run-pentest.sh` provides safe defaults pointing at `localhost` so the script
runs even without `.env`. Maintainers and contributors keep their own
operator-specific values in their local `.env` (already gitignored) or in a
gitignored `CLAUDE.local.md` overlay (see "Local Overrides" below).

---

## Merge Conflict Resolution Checklist

When merging PRs that change function signatures, follow this systematic approach:

### 1. Identify Signature Changes

```bash
# Check what changed in the target branch
git diff main...HEAD -- '*.rs' | grep -A 5 -B 5 "^-.*fn.*\|^+.*fn.*"
```

### 2. Search for All Call Sites

**CRITICAL:** Don't assume you found all call sites. Search comprehensively:

```bash
# Find all references to the changed function
rg "function_name" --type rust

# Or use grep
grep -r "function_name" crates/ apps/ --include="*.rs"
```

### 3. Update Each Call Site

- Verify the new signature in the source
- Update ALL call sites to match
- Don't commit until all call sites are fixed

### 4. Local Validation BEFORE Push

**MANDATORY** before pushing any merge conflict resolution:

```bash
# 1. Check compilation
cargo check --all-targets

# 2. Run tests
cargo test --lib --bins

# 3. Run clippy
cargo clippy --all-targets -- -D warnings

# 4. Check for uncommitted changes
git status
git diff
```

### 5. Common Patterns to Watch

| Pattern | What to search for | Why |
|---------|-------------------|-----|
| Function signature change | All call sites of the function | Multiple callers may exist |
| Struct field addition | Struct construction sites | Builder patterns, Default impls |
| Enum variant change | Match statements | Exhaustive matching required |
| Trait method change | All trait implementations | Multiple impls across crates |

### 6. Red Flags

- **"It compiles locally but CI fails"** → Check for uncommitted changes
- **"Fixed one call site"** → Search for others before assuming you're done
- **"Merge conflict in function body"** → Signature may have changed too

---

## Common Gotchas

### Android Clippy Import

Always `use pentest_core::error::{Error, Result};` - missing `Error` import causes CI failure.

### Gitleaks Full History Scan

Gitleaks scans entire PR commit history. Secrets must never have existed in ANY commit. Use interactive rebase if one sneaks in.

### Hot Reload Limitation

`.rs` changes require full rebuild. Hot-reload does NOT pick up logic changes.

### Clippy Strictness

CI runs clippy with `-D warnings` (warnings = errors). Fix all warnings locally before pushing.

---

## Testing Requirements

### Test Coverage

- Minimum 80% coverage required
- Use `#[ignore]` for tests requiring exclusive resource access
- Tests must be concurrent-safe (no shared mutable globals without proper locking)

### Evidence Buffer Tests

Three tests in `crates/tools/src/evidence_producer.rs` are marked `#[ignore]` because they require exclusive buffer access:
- `evidence_buffer_enforces_capacity_limit`
- `evidence_buffer_near_full_detection`
- `push_is_non_blocking`

Run these separately: `cargo test --test evidence_producer -- --ignored`

### Screenshot Tests

Screenshot capture fails gracefully in headless CI environments (Wayland/X11 not available). This is expected behavior.

---

## CI/CD Pipeline

### GitHub Actions Workflows

| Workflow | Purpose | Trigger |
|----------|---------|---------|
| Multi-Arch Docker | Build arm64/amd64 images | PR, push to main |
| Helm Publish | Package Helm chart | PR, push to main |
| PII Check | Scan for sensitive data | PR |
| Rust Tests | Run test suite | PR, push to main |

### Build Time Expectations

- **Cargo check:** ~1-2 minutes (incremental)
- **Full test suite:** ~3-5 minutes
- **Docker multi-arch:** ~15-20 minutes
- **All checks:** ~20-30 minutes total

---

## Git Workflow

### Remote


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Strike48-public/pick](https://github.com/Strike48-public/pick) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
