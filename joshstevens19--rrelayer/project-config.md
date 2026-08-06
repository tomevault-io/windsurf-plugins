---
trigger: always_on
description: Guidance for coding agents (and humans) working in this repo. Everything here is
---

# AGENTS.md — working on rrelayer

Guidance for coding agents (and humans) working in this repo. Everything here is
grounded in the actual code/CI as of v0.12.1 — when in doubt, the referenced file
is the source of truth.

rrelayer is an EVM transaction relay service written in Rust: an axum REST API +
an in-process transaction engine (queues, nonce management, gas bumping) backed by
Postgres, with pluggable signing providers (raw mnemonic, AWS KMS, AWS/GCP Secrets
Manager, Privy, Turnkey, Fireblocks, PKCS#11, private keys). Clients are a CLI, a
Rust SDK, and a TypeScript SDK.

---

## 1. Non-negotiable rules

1. **Every user-visible change MUST add a changelog entry** in
   `documentation/rrelayer/docs/pages/changelog.mdx` under `## Changes Not Deployed`,
   in the same commit/PR as the change. "User-visible" = anything a user of the
   server, CLI, SDKs, config file, or docker image could notice: features, bug
   fixes, behavior changes, config options, API/SDK surface changes. Pure
   refactors, CI tweaks, and doc typo fixes are exempt. Format is strict — see
   section 4 before touching the file.
2. **Pass the exact CI gates locally before pushing** (CI = `.github/workflows/ci.yml`):
   ```bash
   cargo fmt --all                                            # fix, then:
   cargo fmt --all -- --check
   cargo clippy -- -D warnings -A clippy::uninlined_format_args
   cargo test --exclude rust-sdk-playground --workspace
   ```
   Clippy denies ALL warnings (only `uninlined_format_args` is allowed). rustfmt
   uses `rustfmt.toml` (`use_small_heuristics = "Max"`, `reorder_imports`,
   `use_field_init_shorthand`) — IDE-default formatting will fail the check; always
   run `cargo fmt --all` from the repo root.
3. **Never put the string `Release v` (or `release/`) in a commit message or PR
   title.** CI greps master HEAD for `Release v[0-9]*\.[0-9]*\.[0-9]*` to trigger
   the release pipeline, and skips normal builds on messages containing
   `Release v`/`release/`. Branch names `release/*` are reserved for releases.
4. **The HTTP API has three hand-synced consumers.** Any change to a route path,
   request/response shape, or a serde rename in `crates/core` must be mirrored in
   the Rust SDK (`crates/sdk`), the TypeScript SDK (`sdk/typescript`), and the docs
   (`documentation/rrelayer/docs/pages/integration/`). There are no shared
   constants or codegen — grep is your safety net.
5. **New API handlers must do their own auth.** There is no route-level auth
   middleware; a handler that skips the `AppState` auth helpers is publicly
   reachable. There are two, and picking the wrong one is a security bug — see
   the endpoint recipe in section 6.

---

## 2. Repo map

| Path | What it is |
|---|---|
| `crates/core` | The server (`rrelayer_core`): axum API + transaction engine + DB + signing providers. Almost all behavior lives here. |
| `crates/cli` | The CLI (`rrelayer_cli` — the shipped binary; installers rename it to `rrelayer`). `rrelayer start` boots the core server in-process. |
| `crates/sdk` | The **Rust SDK**, published to crates.io as `rrelayer`. Depends on `rrelayer_core` for types. |
| `sdk/rust` | README pointer only — the Rust SDK is `crates/sdk`. Never add code here. |
| `sdk/typescript` | The **TypeScript SDK**, published to npm as `rrelayer` (versioned independently: npm 1.2.0 vs crates 0.12.1). |
| `crates/e2e-tests` | The real test suite: a standalone `e2e-runner` binary (~49 tests) that orchestrates anvil + Postgres + an embedded server. **Not run in CI.** |
| `playground/rust-sdk-playground` | Workspace member with compile-checked SDK usage snippets + runnable examples. Excluded from `cargo test` in CI, but it must still build and pass clippy. |
| `playground/*` (example, local-node, base, sepolia, …) | rrelayer *project fixtures* (`rrelayer.yaml` + docker-compose) used to run the server locally. `playground/base` is a TS consumer app using the *published* npm SDK. |
| `documentation/rrelayer` | Vocs docs site (npm). Contains **the changelog** and all user docs. CI never builds it — verify with `npm run build`. |
| `helm/rrelayer`, `providers/railway`, `Dockerfile`, `docker-compose.yml` | Deployment artifacts. Root docker-compose is dev Postgres only (port 5446). |
| `scripts/publish-rrelayer.sh` | Manual, interactive crates.io publish. Never run casually. |

Crate/package name traps: directory `crates/sdk` = crate `rrelayer`; `crates/core`
= `rrelayer_core`; `crates/cli` = `rrelayer_cli` (not on crates.io — ships as
GitHub Release binaries). Both SDKs are named `rrelayer` on their registries.

---

## 3. Commands

Rust (repo root unless noted):

```bash
cargo build                        # build the whole workspace
cargo build -p rrelayer_core       # just the server crate
cargo fmt --all -- --check         # CI fmt gate
cargo clippy -- -D warnings -A clippy::uninlined_format_args   # CI lint gate
cargo test --exclude rust-sdk-playground --workspace           # CI test gate (CI adds --release --target <triple>)
```

Run the server locally (needs Docker; see section 8):

```bash
cp .env.example .env               # once
docker-compose up -d               # Postgres 16 on host port 5446

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joshstevens19/rrelayer](https://github.com/joshstevens19/rrelayer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
