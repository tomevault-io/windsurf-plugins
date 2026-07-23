---
trigger: always_on
description: This file is auto-loaded by Claude Code at the start of every session in this repository. Keep it concise (~150 lines, hard cap 200) and focused on **what cannot be derived from the code itself**.
---

# Project memory for `github.com/hishamkaram/gismanager`

This file is auto-loaded by Claude Code at the start of every session in this repository. Keep it concise (~150 lines, hard cap 200) and focused on **what cannot be derived from the code itself**.

## Project identity

- **Module**: `github.com/hishamkaram/gismanager`
- **Purpose**: Go library + CLIs covering two GIS workflows. (1) **Publish pipeline** (`cmd/gismanager`): walk a directory of GIS data files (shapefile / GeoJSON / GeoPackage / KML), load them into PostGIS via GDAL bindings, then publish the resulting tables as GeoServer feature types — the product layer over [`hishamkaram/geoserver`](https://github.com/hishamkaram/geoserver). (2) **Conversion subsystem** (v1.2+ format-conversion + v1.3+ `Rasterize` / `BuildVRT` / `DEMProcessing`, plus `cmd/gisconvert`): stateless wrappers over `gdal.VectorTranslate` / `gdal.Translate` / `gdal.Warp` / `gdal.Rasterize` / `gdal.BuildVRT` / `gdal.DEMProcessing` exposing the full `ogr2ogr` / `gdal_translate` / `gdalwarp` / `gdal_rasterize` / `gdalbuildvrt` / `gdaldem` surface as a Go API plus a CLI.
- **Maintainer / display name**: **Hesham Karm** (note: no trailing 'a'; first name is "Hesham" not "Hisham"). Use this exact spelling in LICENSE, README authorship, AUTHORS files, commit signatures, and any user-facing credits. The legacy GitHub handle `hishamkaram` and the historical email are intentional — do not "correct" them.

## Build & dev — Docker only

**This project does NOT install GDAL on the host machine.** All build, test, integration, and CI work runs inside the multi-stage Dockerfile rooted at the repo's `Dockerfile`, based on `ghcr.io/osgeo/gdal:ubuntu-small-3.12.4`.

- `make dev` opens an interactive bash inside the dev container.
- Every other `make` target shells into the same container non-interactively (`docker compose run --rm -T dev <cmd>`).
- VS Code users: `.devcontainer/devcontainer.json` lets the editor open the source tree inside the container so `gopls` resolves the GDAL CGo headers.
- CI runs the same Docker image as the build container.
- **Never** add `apt-get install gdal-dev` (or equivalent) to host docs, READMEs, or scripts. If a contributor lacks Docker, the entry point is "install Docker", not "install GDAL".

## Versioning regime

- **First stable tag is `v1.0.0`.** Module path stays `github.com/hishamkaram/gismanager` (no `/v2` suffix — gismanager has no released versions to preserve, so the post-revival API is the v1.0 baseline).
- The repo before this revival was last tagged in 2018, never released, and broken on Go ≥ 1.13 (issue #1). The post-revival v1.0.0 is the first-ever release; previous source is reachable only by pre-revival commit SHAs.
- **Don't auto-tag releases.** Tagging is an explicit user action — never run `git tag` or push tags from a Claude session without explicit user authorization.

## Test split

- `*_unit_test.go` (no build tag) — **fast, hermetic, httptest-based.** Run by default via `make test-unit`.
- `*_integration_test.go` with `//go:build integration` — **integration tests against a real GeoServer + PostGIS stack.** Never invoke without `--tags=integration` AND a live compose stack via `make compose-up`.
- The `make test-unit` and `make test-integration` targets are the canonical entry points; CI mirrors them.
- **Both unit and integration tests are mandatory on every PR.** Integration runs against GeoServer 2.27.4 LTS and 2.28.0 stable matching the upstream `hishamkaram/geoserver` client's matrix; both legs must pass for the PR to merge.

## GeoServer client integration

gismanager publishes layers via [`github.com/hishamkaram/geoserver/v2`](https://github.com/hishamkaram/geoserver) — the **v2 line** at the repo root. Key idioms:

- Constructor: `geoserver.New(serverURL, geoserver.WithBasicAuth(user, pass), geoserver.WithLogger(slogLogger))`.
- No `Exists` methods. Test for presence via `Get` + `errors.Is(err, geoserver.ErrNotFound)`.
- Hierarchical scoping: `c.Datastores.InWorkspace(ws).Create(ctx, ...)`, `c.FeatureTypes.InWorkspace(ws).InDatastore(ds).Create(ctx, ft)`.
- Errors surface as `*geoserver.APIError`; sentinel matching via `errors.Is`.
- Every method takes `ctx context.Context` as the first argument. gismanager mirrors this — every public method here also takes `ctx` first.

## Context handling (mandatory for new exports)

gismanager is **context-first**: every exported method on `*Manager` takes `ctx context.Context` as its first argument. No `*Context` twins, no `context.Background()` delegators inside the library. If a caller has no context, they pass `context.Background()` at the call site.

## Typed errors

- Sentinels live in `errors.go`: `ErrConfigInvalid`, `ErrUnsupportedFormat`, `ErrInvalidLayer`, `ErrInvalidDatasource`, `ErrPostGISConnect`, `ErrGeoServerPublish`, `ErrNoSourcesFound`.
- Typed `*GISError` with `Op`, `Source`, `Sentinel`, `Cause` fields and `errors.Is`/`As` semantics. `Unwrap` returns the underlying cause; `Is` matches the sentinel. The `*geoserver.APIError` from the v2 client is recoverable via `errors.As` against the wrapped cause.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hishamkaram/gismanager](https://github.com/hishamkaram/gismanager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
