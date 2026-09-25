---
trigger: always_on
description: Single-binary Rust photo-frame app (`this-week-in-past`): indexes local images by EXIF date, stores metadata in SQLite, serves a `/api/*` + embedded vanilla-JS slideshow (`web-app/` compiled into the binary, no frontend build).
---

# Repository Guidelines

## Project Overview

Single-binary Rust photo-frame app (`this-week-in-past`): indexes local images by EXIF date, stores metadata in SQLite, serves a `/api/*` + embedded vanilla-JS slideshow (`web-app/` compiled into the binary, no frontend build).

## Architecture & Data Flow

Async only at HTTP edge (actix-web); ingest/cache/geo are blocking/parallel inside.

- Ingest (sync, rayon-parallel): `scheduler::index_resources` → `ResourceReader::read_all` (`par_iter` over `RESOURCE_PATHS`) → `filesystem_client::read_files_recursive` + `fill_exif_data` → `store.add_resources` in one rusqlite transaction → `vacuum`.
- Query (async handlers, blocking DB): endpoint → `image_cache::get` (fs hit) else `fs::read` + `image_processor::adjust_image` + `image_cache::put`; SQLite reads via `r2d2` pool (sync methods on `ResourceStore { pool }`).
- Shared state: two `Clone` structs via `web::Data` (`ResourceStore`, `ResourceReader`); no DI framework — constructor fns (`resource_store::initialize`, `resource_reader::new`).
- Background: `clokwerk` daily job at 00:05 + immediate `thread::spawn` on boot (`src/scheduler.rs`).
- Geo (offline): GeoNames `cities500.txt` bulk-loaded into `rstar::RTree` behind `OnceLock` + `tokio::Mutex` single-flight, loaded in `web::block`; `k=20` haversine scan ≤50km, PPLX district → most-populous parent ≤30km as `District, City` (`src/geo_location.rs`).
- Blocking I/O (`ureq` weather, cities500 load) always in `web::block`; CPU scan uses `rayon`, never async tasks.
- Config is env-vars only (`src/config.rs` pattern: `env::var(..).unwrap_or(default)`); `RESOURCE_PATHS` panics if missing.

## Key Directories

- `src/`: all Rust code + co-located tests (no `tests/` dir, no `docs/`, no `scripts/`).
- `migrations/`: `01-initial` … `04-drop_geo_location_cache`, each `up.sql`; loaded via `rusqlite_migration` `from-directory` (`build.rs` sets `rerun-if-changed=migrations`).
- `web-app/`: `index.html`, `script.js`, `style.css`, `images/`, `fonts/` — vanilla JS, edited directly, embedded via `include_str!/include_bytes!` (`src/web_app_endpoint.rs`).
- `.github/workflows/scripts/`: `prep-build-env.sh`, `translate-arch-to-rust-tripple.sh`, `upload-asset-to-release.sh` (CI cross-build/upload helpers).
- `.container/stage-arch-bin.sh`: picks `target/<triple>/release/` binary matching `uname -m` for scratch image.

## Development Commands

```bash
cargo fmt --all -- --check
cargo clippy --all-targets
cargo +nightly rustdoc -- -Z unstable-options --check
CITIES500_PATH=/tmp/cities500.txt cargo test
cargo test <name_substring>          # e.g. cargo test week_image
RESOURCE_PATHS=~/Pictures DATA_FOLDER=./data cargo run
docker build -f Containerfile -t this-week-in-past .
docker compose up                    # ~/Pictures:/resources:ro, 8080:8080
```

One-time native geo setup (container bakes this to `/cities500.txt`):

```bash
curl -o cities500.zip https://download.geonames.org/export/dump/cities500.zip
unzip -p cities500.zip cities500.txt > cities500.txt
CITIES500_PATH=$PWD/cities500.txt cargo test
```

Key env vars (`README.md` table is source of truth): `RESOURCE_PATHS` (required, comma-separated), `DATA_FOLDER` (default `./data`, legacy `CACHE_DIR` fallback), `PORT` (default `8080`), `CITIES500_PATH` (default `/cities500.txt`), `SLIDESHOW_INTERVAL=30`, `REFRESH_INTERVAL=360`, `WEATHER_UNIT=metric`, `OPEN_WEATHER_MAP_API_KEY` + `WEATHER_LOCATION=Berlin`.

## Code Conventions & Common Patterns

For the higher-level engineering principles (SOLID, YAGNI, error surfacing) see `## Engineering Principles` below; this section covers the file-level mechanics.

### Rust

**File structure**: Flat single files (no nested `mod` directories). Each file has a single responsibility. Module declarations live in `main.rs`.

**Error handling**: Use `thiserror`-derived `AppError` enum. Every variant maps to an HTTP status code and JSON body `{"error": "..."}` via `IntoResponse`. No `unwrap`/`expect` in non-test code. Convert `sqlx::Error` with `#[from]`, `serde_json::Error` with a manual `From` impl.

**Async pattern**: Handlers are `async fn` returning `Result<Json<T>, AppError>` or `Result<(StatusCode, Json<T>), AppError>`. DB access uses `SqlitePool` with `sqlx::query!` / `sqlx::query_as!` macros — no manual locking needed.

**State injection**: Axum `State(Arc<AppState>)` extractor. `AppState` holds a `SqlitePool`.

**Logging**: `tracing` with `#[instrument(skip(state))]` on handlers. `tracing-subscriber` with `EnvFilter` (default `info`, overridable via `RUST_LOG`).

**Validation**: `db::validate_meal()` enforces: name 1–200 chars, instructions 1–20000 chars, 1–100 ingredient lines (name ≤100 chars, quantity ≤50 chars), portions 1–10000. Both backend and frontend enforce the same limits. Validation runs inside `insert_meal` and `update_meal` before touching the DB.

**Testing**:
- DB tests: `#[tokio::test]` (async) for operations touching the DB, `#[test]` for pure validation/string helpers. Use `tempfile::TempDir` for isolated databases.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RouHim/this-week-in-past](https://github.com/RouHim/this-week-in-past) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
