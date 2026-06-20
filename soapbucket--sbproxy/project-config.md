---
trigger: always_on
description: *Last modified: 2026-06-17*
---

# sbproxy (Rust workspace)
*Last modified: 2026-06-17*

The active implementation of sbproxy. Cargo workspace with ~20
crates under `crates/`, an e2e suite under `e2e/`, examples under
`examples/`, and an internal observability/cache/AI/security stack.

## Pre-commit checks

Before committing any change, run all checks. Each one corresponds to a
required CI gate; if any fails locally, CI will fail too.

| Check | Command |
|---|---|
| Format | `cargo fmt --all -- --check` |
| Build | `cargo build --workspace` |
| Test | `cargo nextest run --workspace --exclude sbproxy-e2e --locked --profile ci` |
| Doctest | `cargo test --workspace --exclude sbproxy-e2e --locked --doc` |
| Clippy | `cargo clippy --workspace --all-targets -- -D warnings` |
| Docs | `RUSTDOCFLAGS="-D warnings" cargo doc --workspace --no-deps --document-private-items` |

Fix the issue before pushing. Do not paper over with `#[allow(...)]`
unless you also write a one-line comment explaining the deliberate
exception.

The equivalent local runner is `scripts/check.sh`. It uses
`cargo-nextest` when installed (`cargo install cargo-nextest --locked`)
and falls back to plain `cargo test` otherwise. The default path mirrors
the required PR lane: non-e2e workspace tests in the dev profile plus
doctests. This keeps the local target directory materially smaller than
full release/e2e runs. Set `SBPROXY_RELEASE_TESTS=1` to compile test
binaries in release mode, and `SBPROXY_CHECK_E2E=1` to include the
`sbproxy-e2e` package.

By default, `scripts/check.sh` runs `scripts/cleanup-build-artifacts.sh`
on exit to prune `target/doc`, nextest output, incremental directories,
and other high-churn artifacts while keeping dependency build outputs
available for reuse. Set `SBPROXY_CLEAN_AFTER_BUILD=0` only when you
are deliberately preserving every artifact for local debugging.

## Faster inner-loop alternatives

For day-to-day editing, these run in seconds against just the slice
you're working in:

- `cargo check -p <crate>` - single-crate type check, ~1-5s
- `cargo test -p <crate> --lib <prefix>` - unit tests by name prefix
- `cargo test -p sbproxy-config --tests` - config tests + example +
  v1-compat sweep, ~3s
- `cargo test -p sbproxy-modules --lib <policy_name>` - per-policy
  unit tests
- `cargo test -p sbproxy-e2e --release --test <name>` - one e2e test
  file (release build of the proxy is reused if present)

## Workspace layout

```
sbproxy-rust/
  crates/
    sbproxy/            - binary entry point (cmd line, signal handling, server boot)
    sbproxy-core/       - request pipeline (request_filter, response_filter,
                          response_body_filter), Pingora glue
    sbproxy-config/     - config schema, compile_config(), example sweep,
                          v1 schema-compat regression test
    sbproxy-modules/    - all action / auth / policy / transform modules
                          (plugin-style registry, register-via-init pattern)
    sbproxy-plugin/     - public plugin trait surface
    sbproxy-httpkit/    - HTTP request/response helpers shared by plugin authors
    sbproxy-platform/   - circuit breaker, dns, health, messenger, kv storage
                          (redb embedded KV; SQLite for relational state)
    sbproxy-cache/      - response cache, KV stores (memory/file/memcached/redis)
    sbproxy-ai/         - AI gateway path (providers, routing, guardrails,
                          streaming, budgets, cost tracking)
    sbproxy-extension/  - scripting (CEL, Lua, JavaScript, WASM via
                          wasmtime + WASI preview-1), MCP server,
                          feature flags
    sbproxy-observe/    - metrics (sbproxy_*), events, structured logging
    sbproxy-security/   - crypto (HKDF), hostfilter, IP/CIDR utilities,
                          PII redactor, SSRF guard; optional headless-detect
                          (TLS fingerprint) and agent-verify (reverse DNS)
    sbproxy-tls/        - TLS config, mTLS
    sbproxy-transport/  - HTTP/1.1, H2, H3, websockets, gRPC, GraphQL
    sbproxy-vault/      - secret backends + interpolation
    sbproxy-middleware/ - middleware chain (CORS, HSTS, compression, ...)
    sbproxy-openapi/    - OpenAPI emission from live config
    sbproxy-k8s-operator/ - CRDs + reconcile loop
    sbproxy-classifiers/  - ONNX-backed text classifiers (prompt injection v2)
  e2e/
    Cargo.toml          - e2e harness crate (sbproxy-e2e)
    src/                - ProxyHarness lib used by e2e tests
    tests/              - Rust-native e2e (one file per feature)
    cases/              - per-feature config fixtures used by Rust tests
    conformance/        - vendored curl-and-bash conformance suite
                          (93 cases). See e2e/conformance/HOW-TO-RUN.md.
  examples/             - ~90 dir-style examples; every sb.yml here is
                          swept by validate_examples test
  scripts/              - dev-loop helpers (run-e2e.sh, perf-compare.sh,
                          install.sh, generate-certs.sh)
  docker/               - docker-compose stack (sbproxy + Redis +
                          Jaeger) for local dev
  dashboards/           - Grafana dashboards + Prometheus alerts that
                          consume the sbproxy_* metrics

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [soapbucket/sbproxy](https://github.com/soapbucket/sbproxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-20 -->
