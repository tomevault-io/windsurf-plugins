---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this
repository.

## Build & Test

```bash
go build ./cmd/tspages        # build the binary
go test ./...                  # run all tests
go test ./internal/serve/...   # run tests for one package
go test -run TestHandler_ServesFile ./internal/serve/...  # run a single test
```

No linter, Makefile, or CI configuration exists. The module is `tspages` using Go 1.25.

## Logging

**Always use `log/slog` for logging. Never use the `log` standard library package.** All log calls
must include a `"component"` attribute as the first key-value pair after the message string to
identify the subsystem. Use the package name as the component value (e.g. `"deploy"`, `"serve"`,
`"multihost"`). For `tsnet.Server.UserLogf` callbacks, use `"component", "tsnet"`.

## Changelog

**Every commit that has implications for users must include a corresponding update to `CHANGELOG.md`.** Add a bullet
under the `[Unreleased]` section in the appropriate subsection (`Added`, `Changed`, `Fixed`, `Removed`). Write
entries from the user's perspective — describe the observable effect, not the implementation detail.
The changelog follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) format. At release
time, `[Unreleased]` is renamed to the new version with a date, and the comparison links at the
bottom are updated.

## Architecture

tspages is a static site hosting platform for Tailscale networks. It runs multiple tsnet servers —
one **control plane** for deploy/admin APIs and one **per site** for serving content.

```
pages.tailnet.ts.net     → control plane (deploy + admin)
docs.tailnet.ts.net      → serves docs site
demo.tailnet.ts.net      → serves demo site
```

### Package dependency flow

```
cmd/tspages/main.go
  ├── config          — TOML config loading with defaults
  ├── db              — SQLite open + unified migration runner
  ├── multihost       — manages per-site tsnet.Server lifecycle
  │     ├── serve     — static file handler (one per site)
  │     │     ├── compress — gzip/brotli compression + precompressed asset serving
  │     │     └── hints   — HTTP/103 Early Hints for HTML pages
  │     ├── auth      — WhoIs middleware + capability checks
  │     └── tsadapter — wraps tailscale LocalClient → auth.WhoIsClient
  ├── deploy          — ZIP upload + site/deployment management (control plane only)
  │     └── extract   — ZIP/tar/markdown extraction with zip-slip protection
  ├── admin           — sites, deployments, feeds (control plane only)
  ├── help        — help pages, API docs, OpenAPI spec
  ├── analytics       — SQLite-based per-request recording, queries + admin handlers
  ├── submissions     — form submission handling, storage + admin pages
  ├── webhooks        — event notifications with delivery tracking + admin pages
  ├── pushnotify      — browser push notifications for form submissions
  ├── notify          — Notifier interface + Multi fan-out (dependency inversion)
  ├── web             — shared HTML rendering, templates, response helpers
  ├── metrics         — Prometheus counters/histograms/gauges
  ├── cli             — `tspages deploy` subcommand with server auto-discovery
  └── storage         — filesystem-based site/deployment storage + site config
```

### Deployment disk layout

```
data/sites/{site}/
  current              → symlink to active deployment
  deployments/{id}/
    .complete
    manifest.json
    config.toml        ← parsed from tspages.toml (if provided)
    content/
```

### Key design decisions

- **Site name = tsnet hostname = DNS label.** `ValidSiteName` in `storage/store.go` enforces
  lowercase alphanumeric + hyphens, max 63 chars. This is the single validation point — the deploy
  handler calls it before any other work.
- **`multihost.Manager`** owns the map of `siteName → *tsnet.Server`. `EnsureServer(site)` uses
  double-check locking: checks the map under lock, releases lock for blocking tsnet startup,
  re-acquires to store. `deploy.SiteManager` is the interface the deploy handler uses — keeps the
  dependency one-directional.
- **Serve handler is per-site.** `serve.NewHandler(store, site)` bakes in the site name at
  construction. Each site's mux is just `GET /{path...}`.
- **Auth is capability-based.** `auth.Middleware` calls `WhoIs` on each request, parses capabilities
  from the tailnet policy, and stores `[]Cap` in context. Handlers check
  `CanView`/`CanDeploy`/`IsAdmin` themselves. The `WhoIsClient` interface (`auth/caps.go`) decouples
  from the real tailscale client for testability.
- **Storage is symlink-based.** Deployments live at `data/sites/{site}/deployments/{id}/`.
  Activation atomically swaps a `current` symlink. The serve handler resolves symlinks via
  `filepath.EvalSymlinks` before the path containment check.
- **Per-deployment config is TOML-based.** Deployers include `tspages.toml` in their ZIP root. At
  deploy time, it's parsed, validated, stored as `config.toml` at the deployment level (alongside
  `manifest.json`), and removed from `content/`. The serve handler reads it per-request (via the
  `current` symlink). Server-level `[defaults]` in the main config merge under deployment values.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Radiergummi/tspages](https://github.com/Radiergummi/tspages) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
