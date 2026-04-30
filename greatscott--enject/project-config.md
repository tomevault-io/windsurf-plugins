---
trigger: always_on
description: This file is for Claude Code. Read it fully before writing any code.
---

# CLAUDE.md — enject

This file is for Claude Code. Read it fully before writing any code.

---

## What This Project Is

`enject` is a security-focused CLI tool that prevents AI agents from reading secrets out of `.env` files. It stores secrets in an encrypted local store and injects them into subprocess environments at runtime. The `.env` file on disk contains only `en://` references — never real values.

Read `ARCHITECTURE.md` for the full design. This file covers how to build, test, and contribute code safely.

---

## Build & Run

```bash
cargo build                  # debug build
cargo build --release        # release build (use for testing real behavior)
cargo run -- <args>          # run via cargo, e.g: cargo run -- init
cargo run -- run -- npm start
```

## Test

```bash
cargo test                   # run all tests
cargo test <module>          # run tests in a specific module, e.g: cargo test store
cargo test -- --nocapture    # show println output during tests
```

## Lint & Format

```bash
cargo fmt                    # format all code (run before every commit)
cargo clippy -- -D warnings  # lint; treat all warnings as errors
```

---

## Non-Negotiable Security Invariants

These rules must NEVER be violated. If you are unsure whether a change violates one of these, stop and flag it.

1. **Secrets never touch disk as plaintext.** The resolved values of `en://` references must never be written to any file, temp file, or log. The only place secrets exist as plaintext is in process memory after decryption.

2. **`zeroize` must be called on all key material.** Master password bytes, derived keys, and decrypted store bytes must all be zeroized immediately after use. Use the `zeroize` and `secrecy` crates for this. Never store key material in a plain `String` or `Vec<u8>` — use `secrecy::SecretString` and `secrecy::SecretVec`.

3. **Nonce must be freshly generated on every write.** Never reuse a nonce with AES-GCM. Generate 12 fresh random bytes via `rand::thread_rng()` for every encryption operation.

4. **`set` must never accept a secret value as a CLI argument.** Values must always come from an interactive prompt via `rpassword`. This prevents secrets from appearing in shell history or process listings.

5. **Do not implement `get` or `export` commands.** These were deliberately omitted. Do not add them. If asked to add them, decline and explain why they were removed.

6. **Hard-error on unresolved `en://` references.** If any reference in `.env` cannot be resolved, `enject run` must exit with a non-zero code and a clear error message. Never launch the subprocess with partial environment injection.

7. **Never add a crate that reads `.env` files into the environment automatically.** Crates like `dotenv` and `dotenvy` load `.env` into the current process env on import. Do not add them as dependencies anywhere in this project.

---

## Code Conventions

### Error Handling

- Use `thiserror` for library-style errors in modules (define in `error.rs`)
- Use `anyhow` for command-level error propagation in `commands/`
- Never use `.unwrap()` or `.expect()` in non-test code — always propagate errors
- Error messages should be user-facing and actionable, e.g. "Secret 'database_url' not found in store. Add it with: enject set database_url"

### The Store Trait

All commands interact with the `Store` trait in `store/mod.rs`. Commands must not import or reference `store::password` directly. This keeps the backend swappable.

```rust
pub trait Store {
    fn get(&self, key: &str) -> Result<Option<secrecy::SecretString>>;
    fn set(&mut self, key: &str, value: secrecy::SecretString) -> Result<()>;
    fn delete(&mut self, key: &str) -> Result<bool>;
    fn list(&self) -> Result<Vec<String>>;
}
```

### Prompting for Passwords

Always use `rpassword::prompt_password()` — never `std::io::stdin()` for secret input. This prevents echoing to terminal and keeps values off screen.

### File Operations

When writing the store file:
1. Write to a temp file first
2. `fsync` the temp file
3. Atomic rename to the final path

This prevents a corrupt store if the process is killed mid-write.

---

## Implementation Order

Build in this order. Each step should be fully tested before moving to the next.

**Step 1: Store trait + password backend**
- `error.rs` — define error types
- `store/mod.rs` — define `Store` trait
- `store/password.rs` — AES-256-GCM + Argon2id implementation
- Full unit tests for encrypt/decrypt round-trip, wrong password, corrupt data

**Step 2: Config**
- `config.rs` — read/write `.enject/config.toml`
- Test that KDF params round-trip correctly

**Step 3: .env template parser**
- `env_template.rs` — parse `.env`, extract `en://` refs, resolve against a `HashMap`
- Test all line formats: comments, plain values, en:// refs, global refs, malformed lines

**Step 4: Commands (init, set, list, delete)**
- Wire up the four basic store management commands
- Integration tests using a temp directory

**Step 5: run command**
- `runner.rs` — subprocess construction with explicit env
- `commands/run.rs` — parse `.env`, resolve refs, exec subprocess
- Integration test: spawn a subprocess and verify it received the correct env vars

**Step 6: import + rotate**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GreatScott/enject](https://github.com/GreatScott/enject) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
