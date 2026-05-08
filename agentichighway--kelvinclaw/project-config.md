---
trigger: always_on
description: This file defines default expectations for coding agents working in this repository.
---

# KelvinClaw Agent Instructions

This file defines default expectations for coding agents working in this repository.

## Priorities

1. Security

- Protect secrets and sensitive information.
- Avoid introducing vulnerabilities or attack vectors.
- Ensure safe defaults and fail-closed behavior.
- Avoid over-permissioning or excessive access.

2. Stability
3. Reliability
4. Simplicity
5. Size/maintainability

## Safety Rules

- Never commit secrets, keys, tokens, hostnames, or private IPs.
- Keep `.env` / local machine details out of commits.
- Avoid destructive git commands unless explicitly requested.
- Do not revert user-authored unrelated changes.

## Architectural Principles

- All Crates must be self-contained and not directly reference each other, except for the SDK which can reference all.
- The Core SDK and the Memory SDK should be the only interfaces to the crates from the outside.
- All WASM plugins must be loaded through the SDK and not directly from the root or other crates.
- All network access must be mediated through the SDK with explicit allowlists and not directly from the root or other crates.
- All configuration must be validated and fail closed on missing or invalid values, with clear error messages.
- Keep everything as simple as possible, but no simpler. Avoid unnecessary complexity or abstractions.
- Bear in mind at all times the OWASP Top 10, NIST CSF / AI, MITRE ATT&CK, ISO 42001, and other relevant security frameworks and best practices.
- Follow Rust best practices for safety, error handling, and code quality.
- Follow general software engineering best practices for testing, documentation, and maintainability.
- Prioritize security and stability over new features or optimizations.
- Always consider the potential impact of changes on users and the ecosystem.
- Communicate clearly and proactively about changes, especially breaking ones, with users and stakeholders.
- Continuously monitor and improve the security, stability, and reliability of the system over time.

## Build and Test

- Only run tests/builds relevant to the change being made, but ensure all tests pass before finalizing.
- Prefer Docker-based verification, remote server first if available, only fallback to local if needed.
- Remote Docker preflight (required when using a vanilla `rust:*` image):
    - `rustup component add rustfmt clippy`
    - If missing: `cargo install cargo-audit --locked`
    - If missing: `cargo install cargo-outdated --locked`
- Docker cache policy:
    - Iteration/testing: use cached layers (`scripts/test-docker.sh`).
    - Final push validation: run a clean rebuild from zero (`scripts/test-docker.sh --final`).
- Standard SDK lane:
    - `cargo test -p kelvin-sdk`
- Targeted Rust lane:
    - `cargo test -p kelvin-core -p kelvin-wasm -p kelvin-brain -p kelvin-sdk --lib`
- Run formatting checks before finalizing:
    - `cargo fmt --all -- --check`
- Run linting checks before finalizing:
    - `cargo clippy --all -- -D warnings`
- Run security checks before finalizing:
    - `cargo audit`
- Run dependency checks before finalizing:
    - `cargo outdated`
- Run integration tests before finalizing:
    - `cargo test --workspace --tests`
- Run end-to-end tests before finalizing:
    - `scripts/test-e2e.sh`
- Run Docker-based tests before finalizing:
    - `scripts/test-docker.sh`
- If a referenced script is not present in the repo, report it as `MISSING` and continue with the remaining checks.

## Plugin Architecture Guardrails

- Keep model/tool plugins on the SDK path, not direct root coupling.
- Fail closed on missing or invalid plugin configuration.
- Enforce manifest capability/runtime parity and import allowlists.
- Keep network access host-mediated with explicit allowlists.

## Local-First Testing Policy

**Run tests locally before pushing.** CI runners cost money and time. Do not use the remote CI as your personal test harness.

Required local checks before any push:
1. `cargo fmt --all -- --check` — formatting must pass
2. `cargo clippy -p kelvin-cli -- -D warnings` — no warnings in touched packages
3. `cargo test -p kelvin-cli` — all unit tests in touched packages must pass
4. If you changed other crates, run `cargo test --workspace --lib` as well

If a test passes locally but fails in CI, the most common causes are:
- Parallel test interference (env var races, shared temp files)
- Different filesystem behavior (`/tmp` handling, symlinks)
- Missing binaries that the e2e tests require (e.g., `kelvin-gateway`)

When in doubt, use an ephemeral Docker container to replicate the CI environment:
```bash
# Build and test in a clean container
scripts/test-docker.sh
```

## CI Troubleshooting

### `Path::exists()` is unreliable in ephemeral CI runners

We hit a race condition where `Path::exists()` returned `true` for a file in `/tmp` that did not actually exist, causing a "failed to parse trust policy" error instead of the expected "no trust policy found" error.

**Root cause:** `exists()` is a separate syscall from `read()`. In ephemeral CI environments (overlayfs, shared tmpfs, etc.), the filesystem state can change between the two calls — or the filesystem itself can give stale answers.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AgenticHighway/kelvinclaw](https://github.com/AgenticHighway/kelvinclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
