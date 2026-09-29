---
trigger: always_on
description: Guidance for coding agents working in this repository. `AGENTS.md` is a symlink to this file, so Claude Code and any agent that looks for `AGENTS.md` read the same instructions. Update this file and both stay in step.
---

# Repository guide for coding agents

Guidance for coding agents working in this repository. `AGENTS.md` is a symlink to this file, so Claude Code and any agent that looks for `AGENTS.md` read the same instructions. Update this file and both stay in step.

## Project Overview

**neoserver** is a modern multi-workspace geospatial server written in Go (1.26.8+, CGO, GDAL). It publishes PostGIS, DuckDB, GeoParquet, GDAL vector-file, and GeoTIFF/COG raster data through OGC API - Features, WMS 1.3.0, WFS 2.0 (including transactions and locking), WCS 2.0.1/2.1, WMTS 1.0.0, and OGC API - Tiles. It is managed through a REST API backed by an encrypted DuckDB store and through an embedded React administration console (`web/admin`, served at `/admin`).

## Build and Development Commands

Prefer the `make` targets: on macOS they add a custom native linker (`scripts/native-linker.sh`), which a plain `go build` / `go test` does not use.

```bash
# Contributor setup
make doctor        # check Node/Go/GDAL/CGO toolchain
make dev           # isolated local catalog in .cache/dev (key abc123), backend + Vite dev server
make test-dev-tools

# Build (also builds the admin UI when node/npm are available)
make build
make release-build VERSION=v0.1.0   # requires the real UI build
make run RUN_ARGS=serve              # backend only

# Version literals (Go fallback, Dockerfile args, console package, CI smoke test)
make set-version VERSION=v0.1.0     # rewrite, then regenerate openapi.json
make check-version                  # CI enforces this

# Go tests and vet. Both run GO_PACKAGES, which is ./... minus
# web/admin/node_modules -- npm dependencies ship Go sources of their own.
make test                                            # whole suite, -p 1
make vet
make test GO_TEST_FLAGS=-count=1                     # uncached, as CI runs it
make test-race                                       # race tests for concurrent packages
make test-focused PKG=./internal/mgmt TEST=TestName  # single package/test

# Initialize the encrypted backing store (prints a one-time bootstrap JWT)
export NEOSRV_STORE_KEY="$(openssl rand -hex 32)"
./neoserver init --store-path ./data/neoserver.db

# Start the server (WMS/WFS/WCS/WMTS/Tiles/Importer/Auth are disabled by default; enable via config)
./neoserver serve
./neoserver serve --debug          # debug logging
./neoserver serve --config path.toml

# Other CLI subcommands
./neoserver create-token --role super_admin
./neoserver rotate-signing-key
./neoserver add-claim-mapping
./neoserver openapi-dump --output web/admin/openapi.json
./neoserver install-extensions   # download/load matching DuckDB extensions; no catalog needed
./neoserver version

# Docker compose (PostGIS + sample data + server)
make up        # docker compose up --build
make down
make reset-demo

# Smoke test (requires running server)
BASE_URL=http://localhost:9000 ./testing/smoke_test.sh
```

### Admin console (`web/admin`)

Requires Node 24 or 22.22+.

```bash
make ui-install    # npm ci
make ui-dev        # Vite dev server (run the backend separately)
make ui-build      # production build, embedded into the Go binary
make ui-test       # Vitest unit tests
make ui-lint       # ESLint + Prettier check
make ui-check      # shadcn provenance, tests, build, bundle budget (250 KiB gzipped initial JS)
make ui-e2e        # Playwright suite against disposable neoserver + PostGIS + Keycloak
make ui-openapi    # regenerate web/admin/openapi.json from the Go server
cd web/admin && npm run codegen   # regenerate orval React Query client + form schemas
```

When management API handlers or schemas change, run `make ui-openapi` and then `npm run codegen`. `npm run check:form-schemas` detects drift in the generated form schemas.

`make ui-e2e` rebuilds the server image and needs Docker plus free host ports 8080 (Keycloak, hard-coded in `docker-compose.console-e2e.yml`) and 19100 (`CONSOLE_HTTP_PORT`). When the local image already contains the current code, `CONSOLE_SKIP_BUILD=true` skips the rebuild; `KEEP_CONSOLE_ENVIRONMENT=true` leaves the stack running, and extra arguments pass through to Playwright (`./scripts/console/run-e2e.sh -g "pattern"`). Console wording lives in `src/lib/display.ts` and `src/lib/format.ts`; changing it breaks assertions in `e2e/` and `browser-tests/`, which the mocked suites do not catch.

### Integration and conformance tests

Require a running server with a configured `demo` workspace (see docs/getting-started.md):

```bash
make test-ogcapi-integration   # OGC API - Features suite
make test-wms-integration      # WMS suite
make test-wfs-integration      # WFS suite (incl. filter, paging, stored queries)
make test-wcs-integration      # Native live WCS 2.0.1 and 2.1 regression suite
make test-protocol-integration # All native live protocol suites in one fixture
make test-conformance          # Stock digest-pinned official OGC ETS suites
make test-conformance-derived  # Explicitly patched official-derived ETS profiles
make test-conformance-{wms13,wfs20,wcs20,wmts10,ogcapi-features10,ogcapi-tiles10}
make test-assurance-all
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tobilg/neoserver](https://github.com/tobilg/neoserver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-29 -->
