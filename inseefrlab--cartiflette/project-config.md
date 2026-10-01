---
trigger: always_on
description: Two separate things live in this repo:
---

# cartiflette

Two separate things live in this repo:

- `cartiflette/` + `argo-pipeline/`: the **production pipeline** (dist name `cartiflette-pipeline`, not on PyPI). It fetches IGN and Insee sources, processes them with mapshaper and DuckDB, and writes GeoJSON and GeoParquet files to S3 (MinIO, SSPCloud).
- `python-package/cartiflette/`: the **client** published on PyPI as `cartiflette`. It reads those files over HTTPS with DuckDB (GeoJSON: `read_json` over the list of per-value links; GeoParquet: one link filtered in SQL), and converts to geopandas only at the end (`engine="geopandas"`, default) or returns the DuckDB relation (`engine="duckdb"`). GeoParquet is read whenever it exists (assumed from 2025 onwards, checked with a HEAD request for earlier years: `parquet_available`), even when GeoJSON is requested, with a warning; GeoJSON is only read when there is no GeoParquet or with `force=True` (also with a warning). It has its own `pyproject.toml`, `uv.lock` and tests; Python >= 3.10, DuckDB >= 1.5.

The only contract between the two is the S3 layout. Paths are built by `cartiflette/paths.py` and by `python-package/cartiflette/cartiflette/utils.py` (`create_path_bucket` for GeoJSON, `create_path_consolidated` for GeoParquet), which must stay identical. Both test suites pin the same expected strings. The `cartiflette:filter_columns` metadata of the GeoParquet files is part of the contract too.

## S3 safety: never write to production

- `projet-cartiflette/production` is read live by the Python, R and JS clients. Never write there.
- All writes go through `cartiflette/s3.py::upload`. It raises unless `CARTIFLETTE_ALLOW_PRODUCTION_WRITE=true` is set. Don't bypass it: no direct `fs.put*` and no `mc` commands.
- The default write target is `projet-cartiflette/test/v<version>` (a fresh prefix per version: `test/` already holds files from earlier tests, never write at its root), overridable with `CARTIFLETTE_WRITE_BUCKET` and `CARTIFLETTE_WRITE_PATH`.
- Argo itself archives the logs of every step to `projet-cartiflette/argo-logs/` (`templateDefaults.archiveLocation` in `pipeline.yaml`): the only S3 writes that don't go through `s3.upload`. Keep that key prefix fixed, outside `test/` and `production/`.
- Reading published files over public HTTPS is fine.
- Ask before running anything that writes to S3, even to the test location.

## Code style

- Functional code: plain functions and dicts, no classes. The previous object-oriented version (`Dataset`, `Layer`, `MasterScraper`) was removed on purpose; don't reintroduce it.
- Pass paths, `fs` and targets explicitly. No module-level side effects: no S3 client created at import, no `print`, no hard-coded `temp/` relative to the cwd.
- Call mapshaper with `subprocess.run([...])` using an argument list, never `shell=True`.
- Follow conventions based on Ruff styleguide. Use `ruff` and `vulture` to check for codebase after tasks. 

## Pipeline

`prepare_year` → `combinations` → `split_and_upload` (see `cartiflette/pipeline.py`). The Argo steps are in `argo-pipeline/src/`. The workflow (`argo-pipeline/pipeline.yaml`) runs `check-target` first, then one sub-DAG per year of the `years` JSON list. Only the years in `years` are produced, and its default is `["2025"]`: always pass `-p years='[...]'` explicitly. Writing to production takes `-p path=production -p allow_production_write=true` on the command line; the versioned YAML keeps `allow_production_write: "false"` and `path: test/v<version>`, and `tests/test_argo.py` pins this.

- **IGN source.** ADMIN EXPRESS COG CARTO, "France entière" (`FRA`) WGS84. The catalogue is an Atom feed at `https://data.geopf.fr/chunk/telechargement/resource/ADMIN-EXPRESS-COG-CARTO`. Editions 4-0: GeoParquet from 2026, GPKG only for 2025. Editions 3-x (2021-2024): shapefile only, one `.7z` with one file per layer; `ign.shapefile_to_parquet` renames their fields to the 4-0 names (`ign.SHAPEFILE_LAYERS`), so everything after `fetch_layers` is edition-agnostic.
- **Published years.** In production, only 2022 is published before 2025 (GeoJSON, former pipeline); 2021, 2023 and 2024 only hold an intermediate `preprocessed=before_cog` file that no client reads. Regenerating 2022 in production would replace files read live: it is a decision of its own.
- **Rate limit.** The Géoplateforme allows 1 request per second, so `cartiflette/http.py` retries on 429.
- **Field names.** v4 renamed every field (`code_insee`, `population`…). `prepare.py` maps them back to the historical names (`ID`, `NOM`, `INSEE_COM`, `STATUT`, `POPULATION`, `AREA`) so that the published attributes stay stable.
- **Insee TAGC.** `table-appartenance-geo-communes-{year}.zip` (two-digit year up to 2023: `-22.zip`, `-23.zip`), with the header on row 6. Zoning field vintages change over time (`BV2012` became `BV2022`), so they are resolved at runtime into `fields.json`, never hard-coded.
- **Territories.** Metropole and the 5 DROM. Saint-Pierre-et-Miquelon (`code_insee_du_departement = 'NR'`) is excluded. `mapshaper.dissolve` groups by code **and** territory (`AREA`): some codes span territories (AAV2020 `000`, communes outside any attraction area), and one feature over metropole + DROM breaks `bring_drom_closer`.
- **Formats.** GeoJSON and GeoParquet only, with two different layouts:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [InseeFrLab/cartiflette](https://github.com/InseeFrLab/cartiflette) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
