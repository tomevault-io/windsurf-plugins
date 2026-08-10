---
trigger: always_on
description: You are a coding agent working on the OpenStack Keystone Rust implementation.
---

# AGENTS.md

You are a coding agent working on the OpenStack Keystone Rust implementation.

## Core Constraints

- Use `cargo test -p <crate>` for unit tests.
- Use `cargo test -p test_integration` for integration tests,
  `cargo nextest -p test_integration --profile raft` for integration tests with
  raft drivers enabled.
- Always specify crate: `cargo <cmd> -p <crate_name>` when targeting specific
  crates
- Follow Domain-Driven Design: domains (identity, catalog, role, etc.) are in
  separate crates
- Policy files follow convention: `policy/<domain>/<resource>/<action>.rego`
- When checking code, always run `cargo check --message-format=short` or pipe the
  output to only show errors, e.g., `cargo check 2>&1 | grep -i "error"`.

## Workspace Structure

- `crates/keystone/`: Main service binary and API handlers (`src/api/vX/`)
- `crates/core/`: Domain providers and backend trait definitions
- `crates/core-types/`: Shared data structures across workspace
- `crates/api-types/`: API request/response models and conversions
- `crates/*-sql/`: Sea-ORM persistence drivers (e.g., `identity-sql`,
  `catalog-sql`)
- `crates/*-raft/`: OpenRaft distributed storage drivers
- `crates/storage/`: Distributed storage implementation (OpenRaft)
- `crates/config/`: Configuration parsing
- `crates/webauthn/`: WebAuthn/Passkey support extension
- `policy/`: OPA Rego policy files
- `doc/src/adr/`: Architecture Decision Records

## Tooling Commands

- **Build**: `cargo build -p <crate>` or `cargo build` (workspace)
- **Unit tests**: `cargo test -p <crate>`
- **Integration tests (raft)**:
  `cargo nextest run -p test_integration --profile raft`
- **API tests**: `cargo nextest run --profile api -p test_api` (requires SPIRE,
  OPA)
- **Format**: `cargo fmt`
- **Lint**: `cargo clippy -p <crate> --fix --allow-dirty`

## Code Quality Rules

- **Forbidden**: `unwrap()`, `expect()`, `println!`, `unsafe` (workspace lints)
- **Required**: Apache-2.0 license header on every source file
- **Error handling**: Use `thiserror` for error types, propagate with
  `Result<T, E>`
- **Async**: Heavily async codebase, built on `tokio`
- **Pass by reference** when receiver doesn't need ownership

## Backend Trait Convention

Backend traits in `crates/core/src/backend.rs` follow CRUD naming:

- Create: `create_<resource>`
- Read single: `get_<resource>`
- Read multiple: `list_<resources>`
- Update: `update_<resource>`
- Delete: `delete_<resource>`

## API Development

- One HTTP handler per module
- Unit tests live in same module's `tests` submodule
- CRUD handlers require >=3 tests: valid auth + positive/negative policy,
  invalid auth
- Policy enforcement via OPA Rego in `policy/` directory
- If a module's `tests` submodule grows past ~2000 lines, split it into
  `<module>/tests.rs` and declare it with
  `#[cfg(test)] #[path = "<module>/tests.rs"] mod tests;` in the parent file.
  Keeps large test suites from inflating the code file every read/edit
  touches (e.g. `crates/core/src/auth.rs`).

## Running `test_api` (live-server API tests)

- `cargo nextest run --profile api -p test_api` spawns SPIRE + OPA + a real
  `keystone` server via `tools/start-api.sh` as a nextest setup script, and by
  default leaves them running after the run finishes (no auto-teardown).
  Re-running without cleanup can pile up stale daemons across sessions; check
  `ps aux | grep -E 'tmp/nextest|spire-ci-test-harness'` and kill leftovers (or
  run `tools/teardown-api.sh`) before starting a fresh run if a prior run was
  interrupted/killed.
- **Never pipe the run through `tail`/`tail -N`** (e.g.
  `cargo nextest run ... | tail -200`). `tail` without `-f` buffers until EOF,
  and the setup script's long-running daemons (keystone/spire/opa) inherit the
  pipe's write fd, so it never closes even after nextest itself exits — the
  whole invocation looks permanently hung even though the test run actually
  finished. Redirect to a file instead
  (`cargo nextest run ... > /tmp/out.log 2>&1 &`) and read the file.
- The `default` domain is seeded directly in the DB at bootstrap, **not**
  created through `POST /v3/domains`. `Oauth2KeyHook` (ADR 0026) only provisions
  a domain's OAuth2 signing keys on the domain-_creation event_, so `default`
  never gets keys and `/v4/oauth2/default/jwks` and
  `/v4/oauth2/default/.well-known/openid-configuration` 404 forever. Tests
  needing real OAuth2 signing/jwks/discovery must create a fresh domain via the
  API first (key provisioning is async — poll before asserting).
- `AuthenticationResult.principal.identity` from
  `authenticate_by_password`/password auth is always `IdentityInfo::User`, never
  `IdentityInfo::Principal` (that variant is for SPIFFE/workload identities
  only). Handler code and its unit-test mocks must agree on this — a mismatch
  here doesn't fail to compile, it 500s silently at runtime with no log line (an
  unlogged `Err(_) => error_page(500, ...)` branch), so crate-level unit tests
  with a mock built the same wrong way won't catch it. Only a live-server
  request through the real password-auth path surfaces it.

## Running loadtests locally (`tests/loadtest`)

- `tools/run-loadtest-local.sh [goose args...]` sets up postgres (docker),

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openstack-experimental/keystone](https://github.com/openstack-experimental/keystone) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
