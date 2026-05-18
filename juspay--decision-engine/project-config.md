---
trigger: always_on
description: Use this file as the first-stop operating guide for work in `decision-engine`.
---

# Decision Engine Agent Guide

## Purpose

Use this file as the first-stop operating guide for work in `decision-engine`.

- Use docs for orientation, then verify behavior against code, config, and CI before making claims.
- Prefer the smallest relevant surface first: route -> service -> config -> storage -> deployment.
- When docs and code disagree, trust code, config, and CI over prose.

## Source of Truth

Use this precedence order when answering questions or planning changes:

1. Runtime code and entrypoints in `src/`
2. Config files and deployment manifests in `config/`, `docker-compose.yaml`, and `helm-charts/`
3. `justfile`, `Makefile`, `.github/workflows/`, and `scripts/ci-checks.sh`
4. Setup and API docs in `docs/`
5. `README.md` for high-level product context only

Do not treat README claims or marketing language as definitive implementation truth.

## Repo Map

- `src/bin/open_router.rs`: process entrypoint; loads config, fetches secrets, starts app and metrics servers
- `src/app.rs`: main HTTP server wiring, route registration, middleware, graceful shutdown, TLS handling
- `src/routes/`: REST handlers and endpoint behavior
- `src/config.rs`: config structs, config loading, env override behavior, validation hooks
- `src/tenant.rs`: tenant-aware app state wiring
- `src/decider/`: routing decision logic
- `src/euclid/`: routing rules and evaluation engine
- `src/feedback/`: score updates and routing feedback flows
- `docs/local-setup.md`: canonical local and on-prem style startup guide
- `docs/configuration.md`: config-oriented documentation and deployment pointers
- `docs/api-reference1.md`: human-readable API examples
- `docs/openapi.json`: machine-readable API contract
- `docker-compose.yaml`: local topology, compose profiles, supporting services
- `helm-charts/`: Kubernetes and on-prem deployment assets
- `justfile`: canonical engineering commands
- `Makefile`: convenience wrappers around common compose flows
- `.github/workflows/`: CI expectations and release automation
- `scripts/ci-checks.sh`: DB feature compile matrix enforced by CI
- `website/`: dashboard/frontend assets
- `cypress/`: frontend/end-to-end test area

## Which Docs to Open First

- Local startup or environment bring-up: `docs/local-setup.md` -> `docker-compose.yaml` -> `justfile`
- Runtime/config questions: `docs/configuration.md` -> `config/*.toml` -> `src/config.rs`
- API behavior: `docs/openapi.json` -> `docs/api-reference1.md` -> `src/routes/*`
- Server startup and wiring: `src/bin/open_router.rs` -> `src/app.rs`
- Deployment or on-prem: `helm-charts/README.md` -> `helm-charts/templates/*` -> `docker-compose.yaml`
- CI or verification expectations: `justfile` -> `.github/workflows/*` -> `scripts/ci-checks.sh`
- Routing logic: `src/decider/` -> `src/euclid/` -> `src/routes/decide_gateway.rs` -> `src/routes/rule_configuration.rs` -> `src/routes/update_gateway_score.rs`
- Tenant behavior: `src/tenant.rs` -> `src/custom_extractors.rs` -> `src/app.rs`

## Canonical Startup Paths

Use `docs/local-setup.md` as the canonical startup guide. Default to PostgreSQL unless the task is explicitly MySQL-specific.

Preferred quick-start paths:

- PostgreSQL with GHCR images:
  `docker compose --profile postgres-ghcr up -d`
- PostgreSQL with local build:
  `docker compose --profile postgres-local up -d --build`
- Convenience wrappers:
  `make init-pg-ghcr`
  `make init-pg-local`

Source-run commands:

- PostgreSQL build:
  `cargo build --release --no-default-features --features middleware,kms-aws,postgres`
- PostgreSQL run:
  `RUSTFLAGS="-Awarnings" cargo run --no-default-features --features postgres`
- MySQL build:
  `cargo build --release --features release`
- MySQL run:
  `RUSTFLAGS="-Awarnings" cargo run --features release`
- PostgreSQL migration:
  `just migrate-pg`

Verification:

- Health:
  `curl http://localhost:8080/health`

Choose the startup path based on task intent:

- local debugging: compose or source run
- deployment review: Helm and compose manifests
- API behavior inspection: route code plus health/startup verification

## Coding Expectations

- Rust edition is 2021; MSRV in `Cargo.toml` is `1.85.0`
- `unsafe` is forbidden
- Treat clippy warnings around `unwrap`, `expect`, `panic`, `todo`, and `unimplemented` as meaningful design constraints
- Verify both DB tracks when changing shared core logic; do not assume only one backend is affected
- The default feature path is MySQL; PostgreSQL uses `--no-default-features --features postgres`
- Touch the smallest relevant surface first instead of broad refactors
- When changing externally visible behavior, update the relevant docs in `docs/`
- Confirm implementation facts in `src/`, config, and deployment manifests instead of relying on README wording

## Verification Expectations

Before claiming a change is done, choose the smallest sufficient set of checks and state what was run.

Recommended command ladder:

- format check:
  `cargo +nightly fmt --all --check`
- lint:
  `just clippy`
- broad compile coverage:
  `just check`
- unit tests:
  `cargo test`
- PostgreSQL migration path when relevant:
  `just migrate-pg`
- startup/config smoke check when relevant:
  `curl http://localhost:8080/health`
- CI-sensitive Rust changes:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [juspay/decision-engine](https://github.com/juspay/decision-engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
