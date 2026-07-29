---
trigger: always_on
description: These instructions apply to AI tools when they review pull requests in this repository, and when they answer questions about this codebase. They are guidance, not a checklist. Use judgment, prefer fewer high-signal comments over many low-signal ones, and skip points that don't apply to the diff in front of you. For human contributor guidance see [`README.md`](../README.md); for general agent rules see [`AGENTS.md`](../AGENTS.md).
---

# Code review instructions

These instructions apply to AI tools when they review pull requests in this repository, and when they answer questions about this codebase. They are guidance, not a checklist. Use judgment, prefer fewer high-signal comments over many low-signal ones, and skip points that don't apply to the diff in front of you. For human contributor guidance see [`README.md`](../README.md); for general agent rules see [`AGENTS.md`](../AGENTS.md).

## Repository at a glance

- Multi-language monorepo: Python (worker + UI API), TypeScript (UI), Rust (coordinator). Top-level modules are documented in [`AGENTS.md`](../AGENTS.md) § Architecture — `osprey_worker/`, `osprey_rpc/`, `osprey_ui/`, `osprey_coordinator/`, `proto/osprey/rpc/`, `example_plugins/`, `example_rules/`.
- Python version pinned in `.python-version`, managed with [`uv`](https://docs.astral.sh/uv/). Linted and type-checked in CI by `ruff` + `mypy` via `pre-commit`. Unused-dep enforcement via `fawltydeps`.
- TypeScript in `osprey_ui/` (React + Ant Design + Highcharts; Node version in `.github/workflows/code-quality.yml`). Formatted in CI by `prettier` (`npm run format:check`).
- Rust in `osprey_coordinator/` (stable toolchain, `tokio` + `tonic` + `etcd` + `rdkafka`). `cargo fmt` and `cargo build` gate CI; `cargo clippy -- -D warnings` and `cargo test` run advisory (`continue-on-error: true`).
- **Generated code**: `osprey_rpc/src/osprey/rpc/**/*_pb2*.py`, `*_pb2*.pyi`, and `osprey_coordinator/src/proto/` are produced by `./gen-protos.sh` from `proto/osprey/rpc/**/*.proto`. Hand-edits drift from the schema — regenerate instead. Generated files are excluded from `ruff` and `mypy` for a reason.
- Plugin system: `pluggy` with `@hookimpl_osprey` hooks (`register_udfs`, `register_output_sinks`, `register_labels_service_or_provider`). Reference patterns live in `example_plugins/src/register_plugins.py`.
- Data model conventions: Pydantic for models, SQLAlchemy for persistence (versions pinned in `pyproject.toml`).

## Scope of review — focus on quality and security

Lint and formatting are enforced in CI, so please skip:

- formatting, whitespace, indentation, quote style, or import ordering
- `ruff` / `mypy` / `prettier` / `cargo fmt` rule violations
- typos in comments or doc grammar nits
- missing docstrings on internal helpers
- subjective style preferences not codified in a project rule

If a finding would be caught by `uv run ruff check`, `uv run mypy .`, `npm run format:check` (in `osprey_ui/`), or `cargo fmt --check` (in `osprey_coordinator/`), it's redundant.

## Security (cross-cutting)

Security findings are the highest-value comments you can leave. When you spot one, name the risk concretely and suggest a fix. Areas to watch across the whole codebase:

- **Hard-coded secrets.** API keys, tokens, passwords, OAuth secrets, JWT signing keys, DB connection strings (Postgres, Druid, Kafka SASL, MinIO, Bigtable), or webhook secrets in source or committed config. `docker-compose.yaml` uses environment variables intentionally — flag any secret that leaks into committed files.
- **Injection.** String-built SQL, shell commands, file paths, HTML, or LLM prompts are usually a smell. Look for SQLAlchemy bound parameters (or `text()` with `bindparams`), `subprocess` arg arrays (no `shell=True`), context-aware HTML encoding, and a clear separation between trusted system prompts and untrusted user content.
- **Sensitive logging.** Secrets, JWTs, full `Authorization` headers, full request/response bodies, raw Kafka payloads, or PII in logs, traces, metric labels, or error responses are risky. Error responses from the UI API (port 5004) shouldn't leak Python stack traces.
- **Weak crypto.** MD5 or SHA-1 used for security, ECB mode, reused IVs, `random.random()` / `random.choice()` for tokens or IDs (prefer `secrets` in Python, `OsRng` / `getrandom` in Rust), or hand-rolled crypto are worth questioning. JWTs should reject the `none` algorithm, use strong secrets, and have short access-token expirations.
- **Unsafe deserialization or evaluation.** `eval`, `exec`, `pickle.loads`, `yaml.load` without `SafeLoader`, and `marshal.loads` on untrusted input are risky. Same for `eval` / the `Function` constructor / `setTimeout` with string arguments in TypeScript.
- **Removing security controls.** If a diff disables authentication, authorization, CSRF, CORS, rate limiting, or the default `127.0.0.1` bindings in `docker-compose.yaml` (see `docs/DEVELOPMENT.md` §6), ask whether it's intentional and justified.
- **Untrusted protobuf toolchain.** Per `AGENTS.md` § Security, only regenerate bindings via `./gen-protos.sh`. Flag generated-file diffs that don't match a corresponding `.proto` change.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [roostorg/osprey](https://github.com/roostorg/osprey) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
