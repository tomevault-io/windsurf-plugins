---
trigger: always_on
description: BranchBox is a distributed development environment orchestrator managing git worktrees and devcontainers. Milestone 0 is complete—core workflow orchestration for feature worktrees is implemented in Rust. Milestone 1 shipped the Rust agent daemon (macOS/Linux/devcontainer) plus CLI bridge + telemetry so long-running workflows can run in the background. Milestone 2 added the control-plane HTTP drain, `branchbox agent status`, and a SwiftUI macOS preview app that rides on the same gRPC surface as t
---

# Repository Guidelines

## Project Overview & Current State
BranchBox is a distributed development environment orchestrator managing git worktrees and devcontainers. Milestone 0 is complete—core workflow orchestration for feature worktrees is implemented in Rust. Milestone 1 shipped the Rust agent daemon (macOS/Linux/devcontainer) plus CLI bridge + telemetry so long-running workflows can run in the background. Milestone 2 added the control-plane HTTP drain, `branchbox agent status`, and a SwiftUI macOS preview app that rides on the same gRPC surface as the CLI. Next up: Windows transport, full Rails control plane UX, and tighter Tailscale coordination so the offline-first contract holds across devices.

## Project Structure & Module Organization
The workspace roots at `Cargo.toml` and currently ships two members: the core library in `core/` and the CLI in `cli/`. Core modules live under `core/src/` (notably `adapters/`, `modules/`, `bootstrap/`, and cross-cutting helpers like `git.rs`). The CLI entry point is `cli/src/main.rs`, exporting the `branchbox` binary on behalf of the library. Shared documentation sits in `docs/`, CI workflows in `.github/workflows/`, and reproducible tooling in `.devcontainer/`. Future members (agent, control-plane, macos) are commented out in the root workspace until their respective milestones.

## Build, Test, and Development Commands
Run workspace builds with `cargo build` and optimize releases via `cargo build --release`. Execute the CLI locally with `cargo run -p branchbox-cli -- --help` to validate argument wiring. Use `cargo fmt --all -- --check` to enforce formatting, `cargo clippy --all-targets --all-features -- -D warnings` for linting, and `cargo check` for quick iteration. Security and dependency scanning is covered by `cargo audit`.

## Coding Style & Naming Conventions
Rust files follow `rustfmt` defaults (4-space indentation, 100-column soft limit). Modules and files use `snake_case`; types are `UpperCamelCase`; constants are `SCREAMING_SNAKE_CASE`. Prefer explicit `Result<T, Error>` aliases plus the `?` operator for flow control, and leverage `thiserror` for rich domain errors. Branch names should stay action-oriented, e.g., `feature/bootstrap-cleanup` or `fix/git-lock-race`.

## Testing Guidelines
Unit tests live beside their modules under `#[cfg(test)]`; grow integration coverage in a `core/tests/` harness when cross-cutting behaviour warrants it. Run `cargo nextest run --all-features --no-fail-fast` for the default gate, `cargo test --doc` to validate examples, and `cargo nextest run --all-features --run-ignored ignored-only` when you need parity with CI’s integration configuration. CI enforces 90% line coverage via `cargo llvm-cov`, so periodically run `cargo llvm-cov --all-features --workspace --lcov --output-path lcov.info` to catch regressions.

### Manual CLI regression requirement
Before any PR is marked ready or a release branch is cut, run the CLI smoke harness in all supported modes to cover `branchbox init`, multi-feature devcontainer sync, tunnel module permutations (manual fallback, Cloudflared, credential-loss), and teardown end-to-end:

```bash
./scripts/manual-cli-e2e.sh
./scripts/manual-cli-e2e.sh --mode verbose
./scripts/manual-cli-e2e.sh --mode pretend
# Target other stacks (e.g., generic, rails, node)
STACK=generic ./scripts/manual-cli-e2e.sh
STACK=rails ./scripts/manual-cli-e2e.sh
STACK=node ./scripts/manual-cli-e2e.sh
```

Follow up with `./scripts/manual-agent-e2e.sh --cp-stub` to exercise the control-plane HTTP drain. The flag spins up a disposable stub endpoint, feeds it the agent’s batched events, and prints the ack cursor so you can confirm retries/metadata before cutting a release. For quick spot checks (without rerunning the harness) use `branchbox agent status --json`—it reports whether the drain is configured/connected plus the last delivery/failure timestamps.

If you touch devcontainer auth/signing bootstrap (1Password PAT + SSH signing flow from issue #45), also run:

```bash
ORIGIN_SSH_URL='git@github.com:<org>/<repo>.git' \
OP_GITHUB_REF='op://<vault>/<item>/token' \
OP_SIGNING_KEY_REF='op://<vault>/<item>/private key' \
./scripts/manual-1password-e2e.sh --check-failure-path
```

### Devcontainer auth/signing guardrails (issue #45)
- Treat Docker Desktop on macOS as **no SSH-agent socket pass-through** for 1Password keys; prefer the PAT + mounted-file strategy.
- Keep host-only secret retrieval in devcontainer `initializeCommand` (`op read`), and keep container-only git/gh/signing setup in `postStartCommand`.
- Ensure mounted secret files exist before `docker compose up` (`touch` placeholders) or first-run compose startup will fail.
- Keep secret files (`.github-token.env`, `.git-signing-key`, `.gitconfig.env`) in `.gitignore` and template scaffolding so new repos are safe by default.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [branchbox/branchbox](https://github.com/branchbox/branchbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
