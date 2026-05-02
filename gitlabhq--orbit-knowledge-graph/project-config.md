---
trigger: always_on
description: GitLab Knowledge Graph (Orbit). Rust service that builds a property graph from GitLab data and serves queries over gRPC/HTTP.
---

# AGENTS.md

GitLab Knowledge Graph (Orbit). Rust service that builds a property graph from GitLab data and serves queries over gRPC/HTTP. 

## Quick start

All tasks use mise. `mise build`, `mise test:fast`, `mise test:local`, `mise lint:code`, `mise server:start`, `mise server:dispatch`.
Fix linting issues: `mise lint:code:fix`. Validate docs: `mise lint:docs`. Validate ontology: `mise ontology:validate`.
Integration tests need Docker: `mise test:integration`. Correctness subset: `mise test:integration:server`.
CLI integration tests (concurrency, worktrees): `mise test:cli`.

**Worktrees:** after creating a git worktree, run `mise trust` and `git config core.hooksPath "$(git rev-parse --git-common-dir)/hooks"` so that lefthook and mise work correctly.

## How the system works

- **Read-only from the GitLab perspective.** SDLC data flows via Siphon CDC (PostgreSQL logical replication → NATS → ClickHouse). GKG only writes to its own ClickHouse tables.
- **Rails owns authorization.** GKG delegates all access decisions to Rails via gRPC (traversal IDs, resource permissions). See `docs/design-documents/security.md`.
- **ClickHouse = datalake + graph.** Datalake DB holds raw Siphon rows; graph DB holds indexed property graph tables. The indexer transforms between them.
- **Ontology-driven graph.** YAML in `config/ontology/nodes/` and `config/ontology/edges/` drives ETL, query validation, redaction, and edge table routing. New entity types start there, not in Rust. Edge YAML `table:` field + `settings.edge_tables` in `schema.yaml` control which physical table each relationship type writes to and queries from (default: `gl_edge`). Schema: `config/schemas/ontology.schema.json`.
- **Single binary, four modes.** `gkg-server --mode` runs as Webserver, Indexer, DispatchIndexing, or HealthCheck.
- **Layered configuration.** `AppConfig` in `crates/gkg-server-config/` loads three sources (lowest to highest priority): `config/default.yaml`, K8s secret files from `/etc/secrets/`, and `GKG_*` environment variables (`__` separates nested keys, e.g. `GKG_GRAPH__DATABASE`). The CLI (`orbit`) has its own clap-based config and does not use `AppConfig`. See `docs/dev/runbooks/server_configuration.md` for full reference.
- **Siphon and NATS are external.** [Siphon](https://gitlab.com/gitlab-org/analytics-section/siphon) (Go, Analytics team) and NATS are consumed, not owned. Use `/related-repositories` for local checkouts.

## What CI enforces

- `AGENTS.md` and `CLAUDE.md` must be identical (`agent-file-sync-check`)
- Clippy with all features, warnings as errors (`lint-check`)
- Ontology YAML validated against JSON schema (`ontology-schema-validate`)
- `cargo fmt` (`fmt-check`)
- `cargo audit`, `cargo deny`, `cargo geiger` (security stage)
- Unit tests via nextest, includes compiler tests (`unit-test`)
- CLI integration tests: concurrency, worktrees, content resolution (`cli-integration-test`)
- Integration tests with Docker testcontainers (`integration-test`)
- MR titles must follow conventional commit format: `type(scope): description` (`mr-title-check`)
- Markdown files must pass markdownlint, Vale, and lychee checks (`check-docs`)
- Response format version bumped when formatter code or response schema changes (`response-schema-version-check`)
- Metrics catalog regenerated in sync with `gkg-observability` source (`metrics-catalog-check`)

## Where to find things

| What | Where |
|---|---|
| Architecture and data model | `docs/design-documents/data_model.md` |
| Security / AuthZ design | `docs/design-documents/security.md` |
| Query DSL spec | `docs/design-documents/querying/` |
| SDLC indexing pipeline | `docs/design-documents/indexing/sdlc_indexing.md` |
| Code indexing pipeline | `docs/design-documents/indexing/code_indexing.md` |
| Namespace deletion pipeline | `docs/design-documents/indexing/namespace_deletion.md` |
| Schema migration strategy | `docs/design-documents/schema_management.md` |
| Observability / SLOs | `docs/design-documents/observability.md` |
| Ontology node definitions | `config/ontology/nodes/` |
| Ontology edge definitions | `config/ontology/edges/` |
| Ontology JSON schema | `config/schemas/ontology.schema.json` |
| Graph query JSON schema | `config/schemas/graph_query.schema.json` |
| Server config JSON schema | `config/schemas/config.schema.json` (generated via `mise schema:generate`) |
| Query response JSON schema | `crates/gkg-server/schemas/query_response.json` |
| Query test fixtures | `fixtures/queries/` |
| Graph DDL (ClickHouse) | `config/graph.sql` |
| Schema version file | `config/SCHEMA_VERSION` (bump when `graph.sql` or `config/ontology/` changes) |
| RAW output format version | `config/RAW_OUTPUT_FORMAT_VERSION` (semver, bump when `graph.rs` or `query_response.json` changes) |
| Graph DDL (local DuckDB) | Generated at runtime from ontology via `generate_local_tables()` + `duckdb_ddl` |
| Datalake DDL (ClickHouse) | `fixtures/siphon.sql` |
| gRPC service definition | `crates/gkg-server/proto/gkg.proto` |
| Server config structure | `crates/gkg-server-config/src/app.rs` (`AppConfig`), `config/default.yaml` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gitlabhq/orbit-knowledge-graph](https://github.com/gitlabhq/orbit-knowledge-graph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-28 -->
