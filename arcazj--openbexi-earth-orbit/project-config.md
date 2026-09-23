---
trigger: always_on
description: This file provides repository guidance for Claude Code and other coding agents working on OpenBEXI Earth Orbit.
---

# CLAUDE.md

This file provides repository guidance for Claude Code and other coding agents working on OpenBEXI Earth Orbit.

## Commands

Install the locked dependency graph:

```powershell
npm ci
```

Run the complete policy/build checks and test suites:

```powershell
npm run check
npm test
```

Run focused suites when iterating:

```powershell
npm run test:unit
npm run test:python
npm run test:browser
npm run benchmark:full-catalog -- --output artifacts/full-catalog-benchmark.json
npm run benchmark:v21-service -- --output artifacts/v21-service-benchmark.json
```

Python-backed npm commands share `scripts/python-discovery.mjs`: `OPENBEXI_PYTHON_COMMAND` is tried first, followed by `py -3` on Windows, `python3`, `python`, and recognized installed Windows Python directories. Only Python 3 is accepted. `npm run test:browser` reuses an existing healthy loopback server on `OPENBEXI_TEST_PORT` when one is available; otherwise its lifecycle runner starts Python directly and terminates only the server child it created.

Supply-chain and static-artifact commands:

```powershell
npm run audit:dependencies
npm run sbom -- --output release/evidence/openbexi-node-sbom-2.3.2-development.cdx.json
npm run build
npm run check:release-tree -- --tree HEAD --require-clean
npm run rehearse:rollback
py -m http.server 8001 --bind 127.0.0.1 --directory dist
```

Serve the source application with the optional Python API:

```powershell
npm run serve
npm run serve:update
# http://127.0.0.1:8000/index.html
```

The v2.1 full-catalog workspace requires at least one configured token; job submission requires an analyst or administrator token. Values must be unique and at least 24 characters. `server.py` reads the process environment and does not load `.env.example`:

```powershell
$env:OPENBEXI_API_VIEWER_TOKEN = "replace-with-an-independent-random-viewer-token"
$env:OPENBEXI_API_ANALYST_TOKEN = "replace-with-an-independent-random-analyst-token"
$env:OPENBEXI_API_ADMIN_TOKEN = "replace-with-an-independent-random-admin-token"
npm run serve
```

Use `--no-v21-service` for the legacy/static API boundary. Keep `runtime/`, tokens, cursor secrets, database files, and runner artifacts private and uncommitted.

For static source hosting without API routes:

```powershell
py -m http.server 8000 --bind 127.0.0.1
```

Data maintenance:

```powershell
py tools/satellite_data_tools.py export-gp --dry-run
py tools/satellite_data_tools.py export-gp --force
py tools/satellite_data_tools.py export-tle --dry-run
py tools/satellite_data_tools.py export-tle --all
py tools/satellite_data_tools.py build-launches --dry-run
py tools/satellite_data_tools.py build-decayed-db --all
py tools/satellite_data_tools.py build-decayed-db --refresh-satcat --force
py tools/satellite_data_tools.py build-tracked --dry-run
py tools/satellite_data_tools.py build-tracked --all
py tools/satellite_data_tools.py maybe-update --dry-run --interval-hours 24 --reconciliation-interval-hours 24
py tools/satellite_data_tools.py stage-update --dry-run
py tools/satellite_data_tools.py import-candidate
py tools/satellite_data_tools.py validate-candidate <candidate_id>
py tools/satellite_data_tools.py promote-candidate <candidate_id>
```

`export-gp` remains the position source in Version 2.3.2. It requests `active`, `fengyun-1c-debris`, `iridium-33-debris`, and `cosmos-2251-debris`; the event groups are only a partial positioned-debris subset. `build-tracked` derives the searchable SATCAT inventory from local snapshots and makes no provider request. It must record the GP groups that produced the accepted catalog, not merely the configured desired groups. `export-tle` is deprecated compatibility coverage. Numeric and Space-Track Alpha-5 fields are decoded to canonical full NORAD strings, but TLE remains an incomplete subset of current six-digit GP/OMM coverage. Direct maintenance commands above operate on their supplied data root. The opt-in server scheduler instead seeds a private revisioned candidate, runs maintenance only there, validates the complete candidate closure, and atomically promotes a private runtime pointer; it must not rewrite checked-in release data.

For startup performance diagnostics, open `http://127.0.0.1:8000/index.html?perf=1`, then run:

```javascript
window.openbexiStartupPerformance.summary()
```

## Architecture

### Runtime and deployment

The source application is plain HTML, CSS, and browser ES modules. There is no JavaScript bundler, but `npm run build` creates the curated deterministic `dist/` artifact defined by `release/static-artifact.json`.

`js/dependencyBootstrap.js` prefers exact integrity-checked Three.js and satellite.js files under `vendor/`. Source/server mode can use exact-version CDN URLs only as an explicit fallback. The generated static artifact is packaged-only, same-origin-only, and must not execute remote runtime code. Never use `file://`; modules, Workers, JSON, textures, and models require HTTP.

### Coordinate and scientific boundaries

Visualization positions use an Earth-centered frame with Three.js Y/Z axes swapped:

```text
scene = (orbital.x, orbital.z, orbital.y) * KM_TO_SCENE_UNITS
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [arcazj/openbexi_earth_orbit](https://github.com/arcazj/openbexi_earth_orbit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
