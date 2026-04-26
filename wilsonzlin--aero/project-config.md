---
trigger: always_on
description: > **Target:** Windows 7 SP1 (32-bit and 64-bit) running performantly in modern web browsers
---

# Aero: Windows 7 Browser Emulator - Coordination Document

> **Project:** Aero
> **Target:** Windows 7 SP1 (32-bit and 64-bit) running performantly in modern web browsers
> **Scope:** Complete x86/x86-64 system emulation with GPU acceleration

---

## 🛡️ Defensive Mindset (Read This First)

**Assume all code and processes can misbehave.** Every subprocess you spawn might:

- Hang forever
- Consume infinite memory
- Spin the CPU endlessly
- Leave zombie children
- Ignore SIGTERM
- Corrupt shared state

**You are not debugging your code. You are operating in a hostile environment where your own code, dependencies, and the system itself may fail in arbitrary ways.**

This mindset applies to:

- Cargo/rustc (can OOM, hang on codegen, deadlock on locks)
- npm/node (can leak memory, hang on network, spawn zombie processes)
- Playwright/Chrome (can hang, crash, leak GPU memory)
- Any I/O operation (can block indefinitely, return garbage, timeout inconsistently)

**Default behaviors:**

1. **Always use timeouts** — no command runs without a deadline
2. **Always use `-k` with timeout** — SIGKILL after grace period (misbehaving code ignores SIGTERM)
3. **Always use memory limits** — no build runs without a ceiling
4. **Always check exit codes** — success is never assumed
5. **Always capture stderr** — silent failures are unacceptable
6. **Kill aggressively** — SIGTERM, wait briefly, SIGKILL

---

## 📀 Windows 7 Test ISO

A Windows 7 Professional x64 ISO is available for integration testing:

```
/state/win7.iso
```

Use this for:

- Boot testing once the emulator reaches that stage
- Driver installation testing
- Integration test fixtures

**Do not redistribute or commit this file.** See `[LEGAL.md](./LEGAL.md)` for licensing.

---

## ⚠️ Memory Limits for Concurrent Execution

**If running many agents concurrently**, enforce **memory limits**. CPU and disk I/O contention are handled fine by the Linux scheduler, but memory exhaustion will OOM-kill the host.

**The one rule:** Use `bash ./scripts/safe-run.sh <command>` for all non-trivial operations. It enforces both timeout (10 min default) and memory limit (12G default).

```bash
bash ./scripts/agent-env-setup.sh               # One-time: validate environment
source ./scripts/agent-env.sh                   # Activate recommended env vars
bash ./scripts/safe-run.sh cargo build --locked  # Build with timeout + memory limit
bash ./scripts/safe-run.sh cargo test --locked   # Test with timeout + memory limit

  # Override defaults if needed:
  #   - AERO_TIMEOUT / AERO_MEM_LIMIT apply to safe-run's timeout + RLIMIT_AS wrapper.
  #   - AERO_CARGO_BUILD_JOBS controls Cargo parallelism (defaults to -j1 for reliability in constrained sandboxes).
  #   - RUST_TEST_THREADS controls Rust's built-in test harness parallelism (libtest). Agent helper scripts default it to CARGO_BUILD_JOBS for reliability under tight thread limits.
  #   - NEXTEST_TEST_THREADS controls cargo-nextest test concurrency (nextest has its own runner parallelism, separate from RUST_TEST_THREADS).
  #   - AERO_TOKIO_WORKER_THREADS controls Tokio runtime worker threads for supported Aero binaries (e.g. aero-l2-proxy, disk-gateway). Agent helper scripts default it to CARGO_BUILD_JOBS for reliability under tight thread limits.
  #   - AERO_ISOLATE_CARGO_HOME=1 isolates Cargo registry/cache state per checkout to avoid "Blocking waiting for file lock on package cache" on shared hosts.
  #     - `scripts/safe-run.sh` will also create `./.cargo-home` automatically when it detects that lock contention, and will auto-use it on future runs as long as you're using the default Cargo home (`$HOME/.cargo`).
  #   - AERO_RUST_CODEGEN_UNITS (alias: AERO_CODEGEN_UNITS) controls rustc per-crate codegen parallelism (`-C codegen-units=<n>`) without manually editing RUSTFLAGS.
  #   - AERO_SAFE_RUN_RUSTC_RETRIES controls how many times safe-run will retry transient rustc thread/process spawn failures (EAGAIN/WouldBlock), including panics like:
  #       - `failed to spawn helper thread (WouldBlock)`
  #       - `called Result::unwrap() on an Err value: Os { code: 11, kind: WouldBlock, message: "Resource temporarily unavailable" }`
AERO_TIMEOUT=1200 AERO_MEM_LIMIT=16G AERO_CARGO_BUILD_JOBS=2 bash ./scripts/safe-run.sh cargo build --release --locked

# Full workspace builds/tests can take longer than safe-run's default 10 minute timeout (especially
# on cold caches), so prefer a larger timeout when running `--workspace`:
AERO_TIMEOUT=3600 bash ./scripts/safe-run.sh cargo test --workspace --locked
```

Troubleshooting (some agent environments lose executable bits and/or tracked fixtures):

- If you get `Permission denied` running `./scripts/*.sh`, run via bash: `bash ./scripts/safe-run.sh …` (same for `agent-env-setup.sh`).
- If `git status` shows lots of changes that are only mode changes, or tracked files appear deleted/empty, restore the checkout:
  - `git checkout -- .` (bigger hammer), or at least:
  - `git checkout -- scripts tools/packaging/aero_packager/testdata tools/disk-streaming-browser-e2e/fixtures`
  - Non-git fallback: `find scripts -name '*.sh' -exec chmod +x {} +`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wilsonzlin/aero](https://github.com/wilsonzlin/aero) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
