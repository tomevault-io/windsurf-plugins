---
trigger: always_on
description: Guidance for AI assistants working in this repo. Read this first, then the relevant
---

# CLAUDE.md

Guidance for AI assistants working in this repo. Read this first, then the relevant
`docs/` file for the area you're touching.

## What this is

DarkHours: an astronomy/dark-sky planner. Predicts observing conditions
(weather, moon, milky way, satellites, light pollution) and finds nearby dark-sky sites.
The repo was originally named PyNightSkyPredictor; the package/CLI/docs have been renamed
to match the DarkHours product brand, but the CDK/AWS infra layer (stack ids, log groups,
IAM role, ECR repo, the `PYNIGHTSKY_*` env var convention) is intentionally **not yet
renamed** — deferred to the next infra overhaul, since those renames imply real resource
replacement/trust-policy risk. Don't "fix" that layer's naming without checking first.

- **CLI:** `darkhours.py` (the parity oracle — behavior here is the reference).
- **Web API:** `apps/api` (FastAPI). Long computes run async via SQS → `apps/worker`.
- **Engine:** `darkhours/` — `darksky.py` (light pollution + `find_nearby`),
  `weather.py`, `moon_events.py`, `milky_way.py`, `satellites.py`, `tle_provider.py`,
  `location.py`, `scoring.py`, `trip.py`.

## Backends (ports & adapters)

One backend is selected per process via `PYNIGHTSKY_BACKEND` (`local` default, or `aws`),
through `ports.py`. The same engine runs against local files (CLI) or cloud services
(prod). Don't bypass the seam:
- cache → `LocalFileCache` | `DynamoCache`
- geocode store → local json | DynamoDB
- rasters → local tiled grid (memmap) | S3 tiled grid via `gridraster.py` byte-range reads
  (no GDAL at runtime — see `docs/RASTERIO_REPLACEMENT.md`)
- reverse-geocode/routing → Nominatim+Overpass (local) | AWS Location (aws)

## Ways of working (these produced the good results — keep them)

- **Profile before optimizing.** Never guess a bottleneck. `PYNIGHTSKY_PROFILE=1` turns
  on per-phase timing + cache hit/miss in `find_nearby`; `cache.stats` counts lookups.
- **One variable at a time, and benchmark it.** Capture a baseline first, change one
  thing, measure again, record the before/after. See `scripts/bench_*` / `profile_*`.
- **Verify before shipping.** Tests must be green; for perf/infra claims, confirm on real
  infra (see the test-worker recipe) — don't ship on a local number alone.
- **Clean up experiments.** Tear down any throwaway AWS resources and temp files you create.
- **Surface caveats, don't bury them.** "the bump won't help", "first-container image
  tax", correctness gates — call these out explicitly.
- **Persist context.** Update the relevant `docs/` file as you go;
  that's how the next session inherits this one.

## Tests

`python -m pytest -q`. Markers (see `pytest.ini`), all skipped by default:
- `eph` — needs the de421.bsp ephemeris.
- `aws` — hits real AWS; runs only with `PYNIGHTSKY_BACKEND=aws` + cache/raster env + creds.
- `live` — hits real provider APIs; runs only with `PYNIGHTSKY_LIVE=1`
  (`tests/test_provider_smoke.py` covers Open-Meteo, 7Timer, Celestrak, Nominatim, AWS Location).

A default run stays offline and deterministic (877 tests collected as of 2026-07-23;
aws/live auto-skip). Per-file inventory: `tests/README.md`.

## Ship flow (CI/CD)

- **Branch → PR → squash-merge to `main` = deploy.** `.github/workflows/deploy.yml` fires
  on push to `main`: test gate (`pytest -q`) → OIDC → `cdk deploy PyNightSkyLambda`.
  Both the API and worker are **zip Lambdas** — CDK asset bundling pip-installs deps on
  `linux/arm64` inline during deploy; no Docker build or ECR push in CI.
- `PyNightSkyProviderHealth` (like `PyNightSkyWarmer`/`PyNightSkyCicd`) is deployed manually,
  once — `deploy.yml` only ever targets `PyNightSkyLambda`. Redeploy it by hand
  (`cdk deploy PyNightSkyProviderHealth`) if its code changes.
- **`security.yml`** runs on every branch/PR (scan-only, does not gate deploy): pytest +
  Bandit, pip-audit, Semgrep, gitleaks, Trivy (image CVEs against `Dockerfile.worker`)
  via `scripts/security_scan.sh`.
- We're on `main` by default — **branch before committing**. Commit trailer:
  `Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>`; PR body trailer:
  `🤖 Generated with [Claude Code](https://claude.com/claude-code)`.

### Security gate notes
- Trivy image CVEs are gated HIGH/CRITICAL. Accepted, **time-boxed** suppressions live in
  `.trivyignore` with a justification + removal trigger (currently 4 `libsolv` CVEs from
  the AL2023 base — remove when AWS ships the patched package).
- This is a **public repo**: never commit the S3 bucket name, DynamoDB table name, AWS
  account id, or role ARNs. They're injected via env/secrets and discovered at runtime —
  see below. `scripts/profile_aws.sh` reads them from the environment for this reason.

## Running against real AWS (local)

Needs an authenticated session and the resource names (kept out of source). Discover them
rather than hardcoding:
- Resource names: synthesized templates under `cdk/cdk.out/*.template.json`, or
  `aws lambda list-functions` / `aws ecr describe-repositories`.
- Then: `scripts/profile_aws.sh` (set `PYNIGHTSKY_RASTER_BUCKET` + `PYNIGHTSKY_CACHE_TABLE`
  first) runs one `find_nearby` against the aws backend with profiling.

### In-region perf validation (throwaway test worker)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mbeher2200/DarkHours](https://github.com/mbeher2200/DarkHours) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-10 -->
