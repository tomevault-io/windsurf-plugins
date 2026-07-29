---
trigger: always_on
description: - Rust (edition and MSRV are defined in the workspace `Cargo.toml`)
---

# Coding Conventions

## Language and Framework

- Rust (edition and MSRV are defined in the workspace `Cargo.toml`)
- Web framework: Actix-web 4
- ORM: SeaORM with `DeriveEntityModel`
- Database: PostgreSQL
- API docs: utoipa (OpenAPI generation)
- Async runtime: Tokio
- Error handling: `thiserror` for enum errors, `anyhow` for ad-hoc contexts
- Serialization: serde (JSON)

## Code Style

- Follow `rustfmt` defaults — run `cargo fmt --check` before committing
- Clippy is enforced with strict flags (see the exact invocation in [Pre-commit Workflow](#pre-commit-workflow))
- `unwrap()` and `expect()` are forbidden in production code; they are allowed in tests (configured in `.clippy.toml`)
- Use `?` operator for error propagation, not `.unwrap()`
- All CI checks are run via `cargo xtask precommit` (see [Pre-commit Workflow](#pre-commit-workflow))

## Naming Conventions

- Structs: PascalCase (`SbomService`, `AdvisoryService`, `SbomSummary`)
- Functions/methods: snake_case (`fetch_sbom_summary`, `fetch_advisories`)
- Modules: snake_case (`sbom_group`, `source_document`)
- Entity models: `Model` struct inside each entity module, table names are snake_case (`sbom`, `advisory`, `sbom_group`)
- Service structs: `<Domain>Service` (e.g., `SbomService`, `AdvisoryService`)
- Endpoint functions: short verbs — `get`, `all`, `delete`, `upload`, `download`, `packages`, `related`
- API routes: `/v3/<resource>` (e.g., `/v3/sbom`, `/v3/advisory/{key}`)
- OpenAPI operation IDs: camelCase (`getSbom`, `listSboms`)
- Test functions: descriptive snake_case (`upload_with_groups`, `filter_packages`, `query_sboms_by_label`)

## File Organization

### Workspace layout

```
Cargo.toml              # workspace root
entity/src/             # SeaORM entity models (one file per table)
migration/src/          # Database migrations (m<number>_<description>.rs)
common/                 # Shared crates: common, common/auth, common/db, common/infrastructure
modules/                # Domain modules: fundamental, analysis, ingestor, importer, storage, ui, user
query/                  # Query framework and derive macro
server/                 # HTTP server assembly
trustd/                 # CLI binary
test-context/           # trustify_test_context crate; `TrustifyContext` with `#[test_context]`
e2e/                    # End-to-end tests (hurl files)
```

### Domain module structure (e.g., `modules/fundamental/src/sbom/`)

Each domain area follows the same three-submodule pattern:

```
<domain>/
  mod.rs                # Re-exports: pub mod endpoints, service, model
  endpoints/
    mod.rs              # configure() function, endpoint handlers
    test.rs             # Endpoint integration tests (#[cfg(test)])
    label.rs            # Label sub-endpoints (if applicable)
    query.rs            # Query parameter structs
    config.rs           # Endpoint config structs
  service/
    mod.rs              # <Domain>Service struct with pub methods
    test.rs             # Service integration tests (#[cfg(test)])
    <submodule>.rs      # Additional service logic
  model/
    mod.rs              # API response/request models (DTOs)
    details.rs          # Detailed model variants
```

### Entity files

One file per database table in `entity/src/` (e.g., `sbom.rs`, `advisory.rs`, `sbom_group.rs`).

### Migration files

Named `m<7-digit-number>_<description>.rs` (e.g., `m0002030_create_ai.rs`). SQL files go in a directory with the same name when needed.

## Error Handling

- Each module defines its own `Error` enum in `error.rs`, using `#[derive(Debug, thiserror::Error)]`
- Common error variants: `Database(DbErr)`, `Query(query::Error)`, `NotFound(String)`, `BadRequest(...)`, `Any(anyhow::Error)`
- Every module error implements `actix_web::ResponseError` to map errors to HTTP status codes
- `From<DbErr>` is implemented manually (not via `#[from]`) to handle `RecordNotFound` → `NotFound` conversion
- Use `?` with automatic `From` conversions throughout service and endpoint code
- Endpoints return `actix_web::Result<impl Responder>`

## Testing Conventions

- Integration tests use `#[test_context(TrustifyContext)]` from the `trustify_test_context` crate
- Test functions are `async fn` annotated with `#[test(actix_web::test)]`
- Tests return `anyhow::Result<()>` for ergonomic error handling
- Tests live in `test.rs` files alongside the code they test, gated by `#[cfg(test)] mod test;`
- Endpoint tests use `TestRequest` builder pattern to construct HTTP requests and `call_service` to execute
- Service tests call service methods directly against a test database
- Test data is ingested via `TrustifyContext` methods such as `ingest_document` / `ingest_documents`; use the crate-level `document_bytes` helper when you need raw fixture bytes
- The `TrustifyContext` provides: `db`, `graph`, `storage`, `ingestor` fields
- Inline unit tests (e.g., in `sbom.rs`) use `#[cfg(test)] mod test { ... }` blocks

## Commit Messages

- Follow Conventional Commits: `<type>[optional scope]: <description>`
- Types: `feat`, `fix`, `refactor`, `test`, `docs`, `chore`
- Reference the Jira issue in the commit footer (e.g., `Implements TC-123`)
- AI-assisted commits include `--trailer="Assisted-by: Claude Code"`

## Pre-commit Workflow


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [guacsec/trustify](https://github.com/guacsec/trustify) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
