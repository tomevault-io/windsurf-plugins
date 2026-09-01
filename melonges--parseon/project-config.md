---
trigger: always_on
description: - `rtk cargo fmt --all -- --check` — verify the repository's stable Rustfmt policy.
---

# AGENTS.md

## Build & test

- `rtk cargo fmt --all -- --check` — verify the repository's stable Rustfmt policy.
- `rtk cargo clippy -q --workspace --all-targets --no-default-features --features postgres-storage --message-format=short -- -D warnings`
- `rtk cargo clippy -q --workspace --all-targets --no-default-features --features postgres-storage,webhook-sink --message-format=short -- -D warnings`
- `rtk cargo clippy -q --workspace --all-targets --no-default-features --features mongodb-storage --message-format=short -- -D warnings`
- `rtk cargo clippy -q --workspace --all-targets --no-default-features --features mongodb-storage,webhook-sink --message-format=short -- -D warnings`
- Run the same four feature combinations with `rtk cargo test -q --workspace ... --message-format=short` for service-free unit, webhook, and HTTP router tests.
- Build `parseon-server` in release mode with the same four feature combinations; the binary is `target/release/parseon`.
- `rtk cargo test -p parseon-mongodb compose_crud -- --ignored --nocapture` — optional MongoDB replica-set integration coverage after starting its Compose profile.
- Agents may run all verification commands. Prefer Cargo's `-q` flag to suppress successful compilation progress while preserving diagnostics and test failures.
- No CI exists; run format, lint, and test checks before handing off code changes.

## Git commits

- Use Conventional Commits: `<type>[optional scope][!]: <description>`.
- Keep the description imperative, lowercase, and concise; use `!` only for intentional breaking changes.
- Prefer one coherent change per commit. Common types are `feat`, `fix`, `refactor`, `docs`, `test`, `build`, and `chore`.

## Changelog

- Maintain `CHANGELOG.md` for every notable user-, operator-, API-, architecture-, performance-, or contributor-facing change.
- Add entries under `Unreleased` in the same change; do not create a version or release date unless explicitly requested.
- Use applicable `Added`, `Changed`, `Fixed`, and `Breaking` headings and describe outcomes rather than implementation details.
- Keep each entry concise and independently understandable. Omit formatting-only changes and incidental maintenance with no meaningful impact.

## Releases

- Support major, minor, and patch releases according to Semantic Versioning: breaking changes require a major bump, backward-compatible features a minor bump, and backward-compatible fixes a patch bump.
- Before `1.0.0`, use a minor bump for breaking changes and a patch bump for backward-compatible changes, consistent with Parseon's existing `0.x` release history.
- When explicitly asked to prepare a release, update the workspace version and `Cargo.lock`, move `Unreleased` entries under `## <version> - <YYYY-MM-DD>`, and leave a fresh empty `Unreleased` section.
- Use `chore(release): prepare v<version>` for the release commit. Do not publish, tag, push, or create a GitHub release unless explicitly requested.

## Development stage

- Parseon is in an early stage of development. Breaking changes are allowed when they improve the design.
- Do not preserve legacy APIs, compatibility layers, deprecated paths, or transitional code unless explicitly requested.
- Update all affected code, tests, docs, examples, and migrations together so the repository represents only the current design.

## Running Parseon

1. `docker compose up -d` — starts PostgreSQL 16 on `localhost:5432`; `docker compose --profile mongodb --profile erpc up -d` also starts the MongoDB replica set and eRPC gateway.
2. `cp .env.example .env` — `.env` is gitignored; loaded via `dotenvy` + clap env vars.
3. Run the Parseon app on the host. Its default `STORAGE_URL` connects to the
   Compose PostgreSQL instance.
4. Register RPC endpoints with `POST /chains`. Parseon discovers and stores each endpoint's chain ID and starts an enabled chain's worker immediately.
5. Chain registry changes apply without a restart: `PATCH /chains/{chain_id}` starts/stops workers on enable toggles and rotates the RPC URL in place, and `DELETE /chains/{chain_id}` stops the worker before removing its data.

PostgreSQL data is retained in the `pgdata` named volume. Check database logs
with `docker compose logs -f postgres`. The Dockerfile remains available for
building a standalone production image with `docker build -t parseon .`. Select
other adapters with `--build-arg PARSEON_FEATURES=mongodb-storage,webhook-sink`.

Default `HTTP_LISTEN=0.0.0.0:8080`. Override the listen address if the port is
taken (e.g. `HTTP_LISTEN=0.0.0.0:8081`). Both storage drivers use pool defaults.
MongoDB builds accept `STORAGE_DATABASE=parseon`; webhook builds require `WEBHOOK_URL`.

Swagger UI is served at `/swagger-ui/`; the generated OpenAPI document is at
`/api-docs/openapi.json`.
Prometheus-compatible metrics are served at `/metrics`.

The chain API validates each RPC endpoint's chain ID and `finalized` tag before
registration. The supervisor runs one finalized-only worker per enabled chain.
Base's public endpoint is rate-limited; register a private endpoint for sustained workloads.
Complete eRPC routes such as `http://localhost:4000/main/evm/8453` are registered
through the same API. See `docs/adapters.md`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [melonges/parseon](https://github.com/melonges/parseon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
