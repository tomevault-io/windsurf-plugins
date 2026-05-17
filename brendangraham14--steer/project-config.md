---
trigger: always_on
description: This repo includes `flake.nix`. Run commands directly by default. Only use the Nix devshell when a command fails due to missing dependencies that are expected to be provided by the devshell (for example, `nix develop -c <command>`).
---

# Dev Environment
This repo includes `flake.nix`. Run commands directly by default. Only use the Nix devshell when a command fails due to missing dependencies that are expected to be provided by the devshell (for example, `nix develop -c <command>`).

# Commands

Run `just` commands directly first. If one fails due to missing deps, retry it inside `nix develop`.

```bash
# Default
just check
just test
just build

# Fallback when deps are missing
nix develop --command just check
nix develop --command just test
nix develop --command just build

# For multiple commands (fallback)
nix develop --command sh -c "just check && just test"
```

Using the devshell this way keeps the environment consistent without requiring an interactive shell session.

## When to use which command

- **`just pre-commit`** - Runs clippy, format check, and tests. Use before committing to verify code is ready.
- **`just fix`** - Auto-fixes all issues (cargo fix, clippy fix, formatting). Use to resolve lint/format errors.
- **`just check`** - Runs `cargo check --all-features`. Fast type checking without full compilation.
- **`just clippy`** - Runs `cargo clippy --all-features` with warnings as errors. Use for lint checking.
- **`just test`** - Runs `cargo test --all-features`. Use after making functional changes.
- **`just build`** - Runs `cargo build --all-features`. Use when you need a full build.
- **`just fmt`** - Runs `cargo fmt --all`. Use to format code.
- **`just ci`** - Runs `nix flake check`, which is comprehensive but slow. **Agents should almost never need this.** It's primarily for CI pipelines and final verification before merging.


# Version Control
This project uses `jj` (Jujutsu) for version control.

Create new commits frequently as checkpoints during development. Small, incremental commits make it easier to track progress, revert problematic changes, and understand the evolution of the codebase. Don't wait until a feature is complete—commit working states along the way.

# Commits
Follow the conventional commits format.


# Conventions
- NEVER use anyhow errors. Always use well-typed errors with thiserror.
- If you need to convert errors to a fuzzy representation for user-facing messages, use eyre, NEVER anyhow.
- Prefer to preserve the original error rather than swallowing it and re-raising a new error type. Swallowing errors and raising new ones in their stead typically means that we'll lose information about the root cause of the error.
- Algebraic data types are useful. Use them where appropriate to write more ergonomic, well-typed programs.
- You generally should not implement the `Default` trait for structs unless explicitly instructed.
- In production code, DO NOT unwrap errors. Use the try operator `?` and propagate errors appropriately. In tests, `.unwrap()` and `panic!` are allowed for brevity, but prefer `assert!` or `expect()` with descriptive messages where possible.
- NEVER use panic! in production code. Handle errors properly instead of panicking.
- When adding new packages, prefer to use `cargo add`, rather than editing Cargo.toml.
- The workspace Cargo.toml uses glob pattern `crates/*` to include all crates in the workspace.

# Cancellation Propagation
- For user-scoped operations, thread an operation-scoped `CancellationToken` through the entire async call chain, including helper functions and spawned tasks.
- Do not create fresh root tokens (`CancellationToken::new()`) inside operation logic unless the work is intentionally detached from user cancellation.
- If a callee needs local cancellation control, derive a child token from the operation token so parent cancellation still propagates.
- Any async API that can block or cause side effects should accept a cancellation token (or a context object carrying one) rather than implicitly creating its own.
- Check cancellation before committing side effects (for example session config writes, state mutations, event emission) and after long await boundaries.
- Add cancellation tests for new operation flows that assert canceling an in-flight operation prevents post-cancel side effects.

# Type-Safe Identifiers

When using primitive types (integers, UUIDs, strings) as identifiers, **always wrap them in dedicated newtype structs** to ensure type safety.

```rust
// BAD: Raw primitives are easily confused
fn get_user(user_id: i64, org_id: i64) -> User { ... }
get_user(org_id, user_id); // Compiles but wrong!

// GOOD: Newtype wrappers prevent mistakes
#[derive(Debug, Clone, Copy, PartialEq, Eq, Hash)]
pub struct UserId(pub i64);

#[derive(Debug, Clone, Copy, PartialEq, Eq, Hash)]
pub struct OrgId(pub i64);

fn get_user(user_id: UserId, org_id: OrgId) -> User { ... }
get_user(org_id, user_id); // Compile error!
```

**When to use typed identifiers:**
- Database primary/foreign keys
- External API identifiers (Stripe IDs, GitHub IDs, etc.)
- Session/request/correlation IDs
- Any identifier that could be confused with another of the same primitive type

**Recommended derives:**
- `Debug`, `Clone`, `Copy` (for small types), `PartialEq`, `Eq`, `Hash`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BrendanGraham14/steer](https://github.com/BrendanGraham14/steer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
