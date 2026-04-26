---
trigger: always_on
description: Provides a zone-based, capability-gated connector architecture where self-contained binaries implement the FCP interface to bridge AI agents with external services (messaging, cloud, databases, browsers) through a central Gateway with cryptographic verification and WASI sandboxing.
---

# AGENTS.md — Flywheel Connector Protocol (FCP)

> Guidelines for AI coding agents working in this Rust codebase.

---

## RULE 0 - THE FUNDAMENTAL OVERRIDE PREROGATIVE

If I tell you to do something, even if it goes against what follows below, YOU MUST LISTEN TO ME. I AM IN CHARGE, NOT YOU.

---

## AGENT MAIL (am) PROCESS PROTECTION — DO NOT TOUCH

**NEVER run any of these commands:**
- `am service restart` / `am service stop`
- `am doctor fix` / `am doctor repair` / `am doctor reconstruct`
- `kill` targeting any `am`, `am serve-http`, or `mcp-agent-mail` process

The `am serve-http` process is a **shared singleton** that all agents depend on. Restarting or killing it disrupts every other agent. Multiple agents running `am service restart` create a **restart cascade** that makes the service permanently unavailable.

**If `am` commands fail or the API is unreachable:** retry once after a few seconds, then proceed with your work WITHOUT agent-mail. Do NOT attempt to diagnose, repair, or restart the service.

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
- **Unsafe code:** Denied at workspace level (`unsafe_code = "deny"`); individual crates that legitimately need unsafe (e.g., `fcp-sandbox`) may override

### Key Dependencies

| Crate | Purpose |
|-------|---------|
| `asupersync` | Async runtime (FCP3 native; replaces tokio) |
| `fcp-async-core` | Runtime abstraction layer (wraps asupersync, quarantines tokio compat) |
| `async-trait` | Async trait support for connector interfaces |
| `futures-util` | Async stream combinators |
| `serde` + `serde_json` | JSON serialization for FCP protocol messages |
| `ciborium` | CBOR serialization for binary protocol |
| `ed25519-dalek` + `x25519-dalek` | Cryptographic signatures and key exchange |
| `blake3` | Fast cryptographic hashing |
| `chacha20poly1305` | AEAD symmetric encryption |
| `hpke` | Hybrid Public Key Encryption |
| `coset` | COSE (CBOR Object Signing and Encryption) |
| `reqwest` | HTTP client for request-response connectors |
| `asupersync` (websocket) | WebSocket support for streaming connectors (native, replaces tokio-tungstenite) |
| `wasmtime` + `wasmtime-wasi` | WASI sandbox runtime for connector isolation |
| `raptorq` | Fountain codes for reliable data transfer |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dicklesworthstone/flywheel_connectors](https://github.com/Dicklesworthstone/flywheel_connectors) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
