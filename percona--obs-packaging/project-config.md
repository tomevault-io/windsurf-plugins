---
trigger: always_on
description: Validates local project configuration without connecting to OBS.
---

# Percona OBS Packaging - AI Coding Instructions

## Project Purpose

This repo contains RPM and Debian **packaging metadata** for building Percona software packages via a self-hosted [OpenSUSE Build Service (OBS)](https://build.opensuse.org/) instance. It does **not** contain upstream source code — only packaging files. Sources are fetched at build time by OBS services declared in `obs/_service`.

- `osc` — the OBS CLI client (Python library, also used programmatically)
- `percona-obs` — the management script in this repo (see `requirements.txt`)
- `root/` — all packaging content lives here, mirroring the OBS project/package hierarchy

## Repository Layout

```
root/
├── project.yaml                    # OBS project config for the root project
├── common/                         # shared packages used across products
│   └── deps/
│       ├── build/                  # build-time deps (Go toolchain, OBS source services)
│       │   └── <package>/
│       │       └── obs/_aggregate  # aggregates from an external OBS project
│       └── runtime/                # runtime deps shared across products
│           └── <package>/          # e.g. percona-telemetry-agent
│               ├── debian/
│               ├── rpm/
│               └── obs/_service
└── <product>/                      # e.g. ppg/
    ├── releases/                   # release pointer files (see root/README.md)
    │   └── <name>/release.yaml
    ├── staging/                    # full package set, tag builds, QA/release candidate
    │   └── <major-version>/        # e.g. 17/
    │       ├── project.yaml        # OBS project config for this subproject
    │       ├── <package>/          # source packages
    │       │   ├── debian/         # Debian packaging (control, rules, changelog, …)
    │       │   ├── rpm/            # RPM packaging (*.spec, patches, service files)
    │       │   ├── package.yaml    # optional OBS package config (title, description)
    │       │   └── obs/
    │       │       ├── _service    # OBS build service config
    │       │       ├── _aggregate  # aggregates binaries from another OBS project
    │       │       └── _multibuild # multi-flavor builds (PostgreSQL extensions only)
    │       └── <another-package>/
    │           └── ...
    └── devel/                      # manually curated dev-branch subset (see below)
        └── <major-version>/
            └── <package>/          # Class A (full copy) or Class B (obs/_link only)
```

A directory is treated as a **package** if it contains an `obs/` subdirectory or a `package.yaml` file. Everything else is treated as a **project** (subproject grouping).

### `devel/<major-version>/` packages: Class A vs Class B

`devel/<V>/` holds a manually curated subset of `staging/<V>/`, built from development
branches so day-to-day branch work can be built and tested without disturbing staging.
Membership is manual — adding a package there also requires adding its direct dependents,
since an omitted dependent would otherwise link against staging binaries. A **Class A**
devel package is a full copy of the staging package (own `rpm/`, `debian/`, `obs/`) with
`obs/_service` retargeted from a release tag to a development branch, editable independently
of staging. A **Class B** devel package contains only an `obs/_link` pointing at the staging
package (e.g. `<link project="${OBS_ROOTPRJ}:ppg:staging:18" package="…"/>`), rebuilt in the
devel context so it links against devel binaries. See `root/README.md` for full detail.

## Two Package Archetypes

### 1. Standalone service (e.g., `common/deps/runtime/percona-telemetry-agent/`)
- Single static package name (no version placeholder)
- `obs/_service` fetches: packaging (debian + rpm subdirs) + upstream source + `go_modules` (manual)
- `debian/rules` extracts version from `.obsinfo` file at build time
- RPM `Release: 1%{?dist}`

### 2. PostgreSQL extension (e.g., `ppg/staging/17/percona-pg-telemetry/`)
- Uses `@BUILD_FLAVOR@` placeholder throughout (replaced by PG major version at build time)
- `obs/_multibuild` lists PG versions to build for: `<flavor>17</flavor>`
- `debian/pgversions` specifies min PG version (e.g., `9.3+`)
- RPM spec defines `%define pg_version @BUILD_FLAVOR@%{nil}` and uses `%{pgrel}` in `Name:`
- Built with PGXS: `USE_PGXS=1 make`

## Critical Conventions

**`obs/_service` structure** (all packages follow this pattern):
1. First `obs_scm` service: fetch `debian/` subdir from this repo — use `<param name="subdir">${DEBIAN_PACKAGE_DIRECTORY}</param>`
2. Second `obs_scm` service: fetch `rpm/` subdir from this repo — use `<param name="subdir">${RPM_PACKAGE_DIRECTORY}</param>`
3. Third `obs_scm` service: fetch upstream source from its canonical repo
4. Buildtime services: `tar`, `recompress` (gz), `set_version`
5. `go_modules` (manual mode) — only for Go projects (telemetry-agent, etcd)

**`debian/debian.dsc`** must list all tarballs in `Debtransform-Files-Tar`:
```
Debtransform-Files-Tar: debian.tar.gz vendor.tar.gz rpm.tar.gz
```

**Maintainer** (use consistently):
- `Percona Development Team <info@percona.com>` (Debian)
- `Percona LLC` (RPM)

**Epoch: 1** is set on PostgreSQL-related packages to allow version management.

## Project Configuration (project.yaml)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [percona/obs-packaging](https://github.com/percona/obs-packaging) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
