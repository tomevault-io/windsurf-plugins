---
trigger: always_on
description: Self-hosted server for health data: ingest from Apple Health, Oura and Alpha
---

# FreeReps

Self-hosted server for health data: ingest from Apple Health, Oura and Alpha
Progression, storage in PostgreSQL/TimescaleDB, a web dashboard with freely
configurable correlations, and an MCP server. It computes no scores and does no
coaching — that is a decision, not a gap ([`DECISIONS.md`](DECISIONS.md)).

Monorepo. `server/` holds the Go binary `freereps` with the web UI embedded;
`app/` holds the iOS companion (`FreeReps.xcodeproj`). Build and run
instructions live in `server/CLAUDE.md` and `app/CLAUDE.md`, which load only
when working under those paths.

## Gotchas

- **`server/specs/` is the source of truth for wire formats** —
  `hae-export-format.md`, `hae-rest-api.md`, `alpha-progression.md`,
  `hevy-api.md`, `withings-api.md`, `database-schema.md`. Read the spec before
  changing an ingest path. *Why:* the
  payloads come from third-party apps whose shape is not derivable from this
  repo, and a mismatch surfaces as silently dropped rows rather than an error
  (see [`INCIDENTS.md`](INCIDENTS.md), 2026-04-08).

- **The backend does not compile without `server/web/dist`.** `server/web.go`
  embeds it via `go:embed`. Build the frontend first, or create the stub:
  `mkdir -p server/web/dist && touch server/web/dist/.gitkeep`. *Why:* a
  missing directory fails the Go build with an embed error that names the
  directive, not the cause.

- **A metric arriving from two sources needs a unit decision and a source
  filter.** Apple Health and Oura disagree on both units and granularity.
  *Why:* both classes of failure have already occurred and neither raised an
  error — see the 2026-03-25 and 2026-03-26 entries in
  [`INCIDENTS.md`](INCIDENTS.md).

- **What you learn about building or running goes into the `CLAUDE.md` next to
  the code**, not into this file. *Why:* `server/` and `app/` instructions load
  only for sessions working there; in this file they would load in every
  session.

## Repository & CI/CD

**Forgejo is the source of truth**: `git.coydog-fence.ts.net/meltforce.net/freereps`
(`origin`). `github.com/meltforce/FreeReps` is a push mirror — never push there
directly. The mirror is `git push --mirror`: it force-pushes *and* prunes refs
that don't exist on Forgejo, so anything that must survive on GitHub has to
exist on Forgejo first (a GitHub-only branch is deleted at the next sync).

| Where | What runs | Triggered by |
|---|---|---|
| `.forgejo/workflows/ci.yml` | Go build/vet/test/lint, frontend tsc + build, document contract check, then build → `git.coydog-fence.ts.net/meltforce.net/freereps:edge` → redeploy on `freereps-lxc` | push/PR on `main` |
| `.github/workflows/ios.yml` | Xcode build (no macOS runner exists on Forgejo) | mirror push |
| `.github/workflows/release.yml` | Docker Hub image + GitHub Release with `freereps-upload` binaries | tag push, carried over by the mirror |

Deploy runs through the shared reusable workflow
`meltforce.net/ci-workflows/.forgejo/workflows/build-push-deploy.yml@v4` with
`sync_compose: false` — **the deployed compose belongs to the homelab repo**
(`docker/stacks/freereps/compose.yaml`, plus the catalog entry in
`configuration/docker-stacks/stacks/freereps.yml`, which renders `.env` and
`config.yaml`). Change the image ref, ports or volumes there, not here.
`server/docker-compose.yml` is for local development only.

Runner labels: `docker` (normal jobs), `docker-buildx` (image builds), `host`
(runs on the runner LXC itself — needed for Tailscale SSH into deploy targets).
The Forgejo org `meltforce.net` already provides `REGISTRY_USER`,
`REGISTRY_TOKEN`, `DOCKERHUB_USERNAME`, `DOCKERHUB_TOKEN`; no repo secrets.

## Data source documentation

Upstream references not reproduced in `server/specs/`:

- [Health Auto Export — export formats](https://help.healthyapps.dev/en/health-auto-export/export-format/)
- [Health Auto Export — server connection (TCP/MCP)](https://help.healthyapps.dev/en/health-auto-export/automations/server-connection/)
- [HealthyApps reference server](https://github.com/HealthyApps/health-auto-export-server) — the Grafana-based implementation this project's ingest was checked against

## Repo documents

These documents carry state over time. The axis is where a thing *is*, not what
it is about.

| File | Holds |
|---|---|
| `ROADMAP.md` | Open work only. Status token `[open]`. |
| `DECISIONS.md` | Decisions taken, including decisions not to do something — those are the ones most likely to be re-derived from scratch otherwise. |
| `INCIDENTS.md` | Postmortems for things that broke. Newest first. |

**The movement rule.** When an item closes it is *removed* from `ROADMAP.md`,
and its reasoning moves to whichever document above holds that kind of thing.
Nothing is struck through — a struck-through row is a row that should have been
moved. Status tokens are exactly `[open]`, `[done YYYY-MM-DD]`,
`[dropped YYYY-MM-DD]`; emoji never carry status.

**Before closing an item, read its entry for residual work, dates, or
triggers.** Each of those becomes its own `[open]` row before the entry leaves
the roadmap. This is the step that gets skipped, and skipping it is how
finished-looking work quietly loses its tail.

**No new top-level documents** unless the concern is genuinely orthogonal to the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [meltforce/FreeReps](https://github.com/meltforce/FreeReps) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
