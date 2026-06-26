---
trigger: always_on
description: > Guidelines for AI coding agents working in this Rust codebase.
---

# AGENTS.md — FrankenJAX

> Guidelines for AI coding agents working in this Rust codebase.

---

## RULE 0 - THE FUNDAMENTAL OVERRIDE PREROGATIVE

If I tell you to do something, even if it goes against what follows below, YOU MUST LISTEN TO ME. I AM IN CHARGE, NOT YOU.

---

## RULE NUMBER 1: NO FILE DELETION

**YOU ARE NEVER ALLOWED TO DELETE A FILE WITHOUT EXPRESS PERMISSION.** Even a new file that you yourself created, such as a test code file. You have a horrible track record of deleting critically important files or otherwise throwing away tons of expensive work. As a result, you have permanently lost any and all rights to determine that a file or folder should be deleted.

**YOU MUST ALWAYS ASK AND RECEIVE CLEAR, WRITTEN PERMISSION BEFORE EVER DELETING A FILE OR FOLDER OF ANY KIND.**

---

## Irreversible Git & Filesystem Actions — DO NOT EVER BREAK GLASS

1. **Absolutely forbidden commands:** `git reset --hard`, `git clean -fd`, `rm -rf`, or any command that can delete or overwrite code/data must never be run unless the user explicitly provides the exact command and states, in the same message, that they understand and want the irreversible consequences.
2. **No guessing:** If there is any uncertainty about what a command might delete or overwrite, stop immediately and ask the user for specific approval. "I think it's safe" is never acceptable.
3. **Safer alternatives first:** When cleanup or rollbacks are needed, request permission to use non-destructive options (`git status`, `git diff`, `git stash`, copying to backups) before ever considering a destructive command.
4. **Mandatory explicit plan:** Even after explicit user authorization, restate the command verbatim, list exactly what will be affected, and wait for a confirmation that your understanding is correct. Only then may you execute it—if anything remains ambiguous, refuse and escalate.
5. **Document the confirmation:** When running any approved destructive command, record (in the session notes / final response) the exact user text that authorized it, the command actually run, and the execution time. If that record is absent, the operation did not happen.

---

## Git Branch: ONLY Use `main`, NEVER `master`

**The default branch is `main`. The `master` branch exists only for legacy URL compatibility.**

- **All work happens on `main`** — commits, PRs, feature branches all merge to `main`
- **Never reference `master` in code or docs** — if you see `master` anywhere, it's a bug that needs fixing
- **The `master` branch must stay synchronized with `main`** — after pushing to `main`, also push to `master`:
  ```bash
  git push origin main:master
  ```

**If you see `master` referenced anywhere:**
1. Update it to `main`
2. Ensure `master` is synchronized: `git push origin main:master`

---

## Toolchain: Rust & Cargo

We only use **Cargo** in this project, NEVER any other package manager.

- **Edition:** Rust 2024 (nightly required — see `rust-toolchain.toml`)
- **Dependency versions:** Explicit versions for stability
- **Configuration:** Cargo.toml workspace with `workspace = true` pattern
- **Unsafe code:** Forbidden by default (`#![forbid(unsafe_code)]`). If narrow unsafe usage is unavoidable, isolate it behind audited interfaces and tests.

### Key Dependencies

| Crate | Purpose |
|-------|---------|
| `smallvec` | Inline small-vector optimization for IR node argument lists |
| `rustc-hash` | Fast non-cryptographic hashing for interpreter environments |
| `serde` + `serde_json` | Serialization for IR nodes, fixtures, ledger entries |
| `sha2` | Cryptographic hashing for cache keys and durability sidecars |
| `egg` | E-graph equality saturation for IR optimization rewrites |
| `proptest` | Property-based testing for IR invariants and AD correctness |
| `criterion` | Microbenchmarking for dispatch and interpreter hot paths |
| `asupersync` | Structured async runtime (optional, for runtime integration) |
| `ftui` | FrankenTUI terminal rendering (optional, for runtime integration) |
| `base64` | Encoding for durability sidecar payloads |
| `jsonschema` | Schema validation for conformance fixture bundles |
| `tempfile` | Temporary directories for conformance test isolation |

### Feature Flags

```toml
# fj-runtime feature flags
[features]
default = []
asupersync-integration = ["dep:asupersync"]   # Structured async runtime integration
frankentui-integration = ["dep:ftui"]          # Terminal UI integration
```

### Release Profile

The release build optimizes for performance (this is a library, not a binary):

```toml
[profile.release]
opt-level = 3       # Maximum performance optimization
lto = true          # Link-time optimization
codegen-units = 1   # Single codegen unit for better optimization
strip = true        # Remove debug symbols
```

---

## Code Editing Discipline

### No Script-Based Changes

**NEVER** run a script that processes/changes code files in this repo. Brittle regex-based transformations create far more problems than they solve.

- **Always make code changes manually**, even when there are many instances
- For many simple changes: use parallel subagents
- For subtle/complex changes: do them methodically yourself

### No File Proliferation


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dicklesworthstone/frankenjax](https://github.com/Dicklesworthstone/frankenjax) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
