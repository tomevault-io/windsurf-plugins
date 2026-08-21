---
trigger: always_on
description: This file provides guidance to AI coding agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents when working with code in this repository.

## Build & Test Commands

```bash
# Build (SQLX_OFFLINE=true needed when no live database is available)
SQLX_OFFLINE=true cargo build --workspace

# Format + lint (always run before committing)
cargo fmt --all
SQLX_OFFLINE=true cargo clippy --all-targets --all-features -- -D warnings

# Unit tests (no database needed)
cargo test --workspace

# Integration tests (requires live PostgreSQL)
export DATABASE_URL=postgres://postgres:testpassword@localhost:5432/pgroles_test
cargo test --workspace -- --include-ignored

# Run a single test
cargo test -p pgroles-core --lib diff::tests::diff_creates_new_roles -- --exact

# CRD drift check (CI compares every committed CRD copy — chart and k8s/ —
# against crdgen output)
scripts/check-crd-drift.sh

# Regenerate CRDs after modifying crd.rs. crdgen writes into the chart; the
# k8s/ copies are duplicates that must be refreshed by hand, and
# check-crd-drift.sh fails if any copy drifts.
cargo run --bin crdgen -- --output-dir charts/pgroles-operator/crds/
cp charts/pgroles-operator/crds/postgrespolicies.pgroles.io.yaml k8s/crd.yaml
cp charts/pgroles-operator/crds/postgrespolicyplans.pgroles.io.yaml k8s/postgrespolicyplan-crd.yaml
cp charts/pgroles-operator/crds/postgrespolicycandidates.pgroles.io.yaml k8s/postgrespolicycandidate-crd.yaml
cp charts/pgroles-operator/crds/ephemeralaccesspolicies.pgroles.io.yaml k8s/ephemeralaccesspolicy-crd.yaml
cp charts/pgroles-operator/crds/ephemeralaccessrequests.pgroles.io.yaml k8s/ephemeralaccessrequest-crd.yaml
```

### Local PostgreSQL for integration tests

```bash
docker run --rm --name pgroles-pg16 \
  -e POSTGRES_PASSWORD=testpassword \
  -e POSTGRES_DB=pgroles_test \
  -p 5432:5432 \
  postgres:16
```

## Architecture

### Data Pipeline

```
YAML → parse_manifest() → PolicyManifest
     → expand_manifest() → ExpandedManifest (profiles × schemas resolved)
     → RoleGraph::from_expanded() → RoleGraph (desired)
     → [operator only] compose_effective_graph() → RoleGraph (effective desired)
       overlays memberships from active EphemeralAccessRequests

DB   → inspect() → RoleGraph (current)
     → detect_pg_version() → SqlContext

diff(current, effective desired) → Vec<Change> → sql::render_all_with_context() → SQL
```

### Workspace Crates

- **pgroles-core** — Pure library, no IO. Manifest parsing, profile expansion, diff engine, SQL rendering, manifest export. All collections use `BTreeMap`/`BTreeSet` for deterministic output.
- **pgroles-inspect** — Async database introspection via `sqlx`/`pg_catalog`. Version detection, cloud provider detection (RDS, Cloud SQL, AlloyDB, Azure), drop-role safety preflight.
- **pgroles-cli** — Binary crate. Thin orchestration over core + inspect. Subcommands: `validate`, `diff`/`plan`, `apply`, `inspect`, `generate`.
- **pgroles-operator** — Kubernetes operator. Reconciles `PostgresPolicy` CRDs (`pgroles.io/v1alpha1`). Has a `crdgen` binary for generating the CRD YAML.
  - Kubernetes identifiers: every name and label value derived from user input goes through `k8s_names`. Do not hand-roll truncation or character filtering elsewhere — a cut that lands on a separator yields a value the API server rejects, which surfaces as a policy that silently stops reconciling. Invariants are enforced by property tests in `tests/identifier_properties.rs`.
  - Health endpoints: `/livez`, `/readyz`
  - Reconciliation modes: `apply`, `plan`
  - Metrics/telemetry: prefer OTLP export via OpenTelemetry Collector; do not add a built-in Prometheus scrape endpoint by default unless the change explicitly requires it.

### Diff Change Ordering

`diff()` assembles changes in dependency order: creates → alters/comments → grants → default privileges → membership removes → membership adds → default privilege revocations → revocations → drops. Retirement steps (terminate sessions, reassign owned, drop owned) are inserted immediately before the matching `DropRole` by `apply_role_retirements()`. `apply` executes the whole plan in a single transaction.

## CI

`.github/workflows/ci.yml` runs:

- **Lint** — `cargo fmt --check`, `clippy -D warnings`, CRD drift check, helm-docs drift check
- **Unit Tests** — `cargo test --workspace`
- **Integration Tests** — PG 16/17/18 matrix, `cargo test --workspace -- --include-ignored`
- **Docker and example smoke tests** — verifies the documented container and example flows work end-to-end
- **Operator E2E** — kind cluster, deploys the operator plus an OpenTelemetry Collector, runs happy-path plus conflict/invalid/missing-secret/insufficient-privilege/secret-rotation scenarios, verifies roles in the database, and verifies OTLP metrics export
- **Plan lifecycle and load E2E** — plan approval flows, plus generated-policy convergence at higher object counts and ephemeral-request load
- **Ephemeral access E2E** — runs twice: once for the trusted-writer posture, once for the Kyverno secure-admission profile in `k8s/security/`

The heavier fairness/load coverage lives in `.github/workflows/operator-fairness-load.yml` and runs on a nightly schedule when `main` has changed.

## Agent Skills


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thepartly/pgroles](https://github.com/thepartly/pgroles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
