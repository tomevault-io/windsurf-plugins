---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Boxball builds prepopulated databases of two open-source baseball datasets (Retrosheet play-by-play + Baseball Databank/Lahman seasonal stats). Output is a family of Docker images (`doublewick/boxball:<target>-<version>`) plus flat-file Parquet/CSV downloads. The repo is the build pipeline, not a runtime app — there is no server to start; "running" the project means building images via `docker compose`.

## Workflow

Boxball 2026.0.0 cut over to `master` and Docker Hub on 2026-05-01 (PLAN.md PLE-356 cutover entry). Standard GitHub workflow now applies: feature branch → PR → squash-merge to `master`. CI runs on GitHub Actions per push/PR. Local rehearsal still available via `make ci` (act).

Self-hosted Mac runner (`droher/boxball`, labels `self-hosted, macOS, ARM64`) handles the `build-push` job in `release.yml` — see PLAN.md cutover entry for the macOS keychain workaround.

## Local CI — `act`

`make ci` runs `.github/workflows/ci.yml` locally inside Docker via [`act`](https://github.com/nektos/act). Same workflow file, same runner image (`catthehacker/ubuntu:act-latest`). High-fidelity local rehearsal — not byte-identical to GitHub-hosted runners (cache action, buildx, and docker-in-docker behavior diverge), so final source of truth is GitHub Actions when we cut over to master.

```
make ci                # full pipeline (style → int-test → e2e-test)
make ci-style          # one job
make ci-int-test
make ci-e2e-test
make ci-list           # list jobs without running
```

Requires `act` (`brew install act`) + a running Docker daemon. `.actrc` pins `linux/amd64` for reproducibility on M-series and forwards the host Docker socket so the e2e job's `docker compose build` works against the host daemon.

Schemas are SQLAlchemy-defined and authoritative. DDL for every target dialect (Postgres, Postgres+Citus columnar, MySQL, SQLite, Clickhouse) is generated from the same metadata.

## Pipeline architecture

Three stages, each its own Docker context, chained via multi-stage builds:

1. **`extract/`** — Downloads pinned Retrosheet + Baseball Databank archives (versions in `.env`), builds Chadwick from source, runs `cwdaily/cwgame/cwevent/cwsub/cwcomment` over Retrosheet event files to produce CSVs, depascalizes Databank CSVs. Output: zstd-compressed CSVs under `/extract/{retrosheet,baseballdatabank}/`. Entry points: `extract/parsers/retrosheet.py`, `extract/parsers/baseballdatabank.py`.
2. **`transform/`** — Two parallel branches from the extract image:
   - `csv.Dockerfile` — passthrough (currently identity; placeholder for cleaning steps).
   - `parquet.Dockerfile` — runs `transform/src/parquet.py` to convert each zstd-CSV into a Parquet file using the SQLAlchemy schema as the type contract (PyArrow streaming reader, zstd compression).
   - `ddl.Dockerfile` — runs `transform/src/ddl_maker.py`, which iterates `all_factories × all_metadata` and writes one `.sql` (or `.sql`-like) file per target into `/ddl/`.
3. **`load/`** — One Dockerfile (`load/Dockerfile`) with a stage per target DB. Each stage `FROM`s the official DB image, copies the matching `/ddl/<target>.sql` into `/docker-entrypoint-initdb.d/`, copies CSV or Parquet from the relevant transform stage, and lets the DB's init mechanism load on first container start. Per-target shell/SQL hooks live under `load/<target>/` (named `A_*` to run before the generated DDL, `z_*` to run after, alphabetically).

The `docker-compose.yml` wires this up — every service uses YAML anchors (`x-extract`, `x-postgres`, …) and each has a `<svc>-latest` twin that retags `${VERSION}` → `latest`. `depends_on` reflects the build DAG (`parquet`/`csv` → `extract`; DB targets → `csv`+`ddl` or `parquet`+`ddl`).

## DDL factory pattern

`transform/src/target_ddl_factory.TargetDdlFactory` is the abstract base. Each target subclass provides:
- `target_name` (output filename stem) and `dialect` (SQLAlchemy `Dialect`, or `None` if hand-rolled).
- `metadata_transform(metadata)` — overridable rewrite of the SQLAlchemy `MetaData` before DDL emit. Used to e.g. flatten schemas into table-name prefixes for SQLite, swap engines for Clickhouse, drop `dummy_id` autoincrement PKs that don't translate.
- `make_copy_ddl(metadata)` — emits the loader statements (`COPY FROM PROGRAM`, `LOAD DATA INFILE`, `.import`, etc.) tailored per dialect's NULL/bool/CSV quirks.

When adding a new target, subclass `TargetDdlFactory`, register it in `transform/src/ddl_factories/__init__.py::all_factories`, and add a stage in `load/Dockerfile` + service entry in `docker-compose.yml`.

When changing a schema column, edit `transform/src/boxball_schemas/{retrosheet,baseballdatabank}.py` — every target's DDL re-emits from there. The `dummy_id = Column(Integer, autoincrement=True, primary_key=True)` pattern is intentional: tables without natural PKs use it, and every target factory strips it because most loaders can't autoincrement during bulk load.

## Build / run

`.env` defines `VERSION`, `REPO`, dataset SHAs, and `BUILD_ENV` (`prod` pulls real data; `test` uses tiny fixtures from `extract/fixtures/raw/`). Compose reads it automatically.

```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [droher/boxball](https://github.com/droher/boxball) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
