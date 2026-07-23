---
trigger: always_on
description: This file is auto-loaded by Claude Code at the start of every session in this repository. Keep it concise (~150 lines, hard cap 200) and focused on **what cannot be derived from the code itself**.
---

# Project memory for `github.com/hishamkaram/geoserver`

This file is auto-loaded by Claude Code at the start of every session in this repository. Keep it concise (~150 lines, hard cap 200) and focused on **what cannot be derived from the code itself**.

## Project identity

- **Module**: `github.com/hishamkaram/geoserver`
- **Purpose**: Go client library for the GeoServer REST API (workspaces, datastores, feature types, layers, layer groups, styles, coverages, namespaces, settings).
- **Maintainer / display name**: **Hesham Karm** (note: no trailing 'a'; first name is "Hesham" not "Hisham"). Use this exact spelling in LICENSE, README authorship, AUTHORS files, commit signatures, and any user-facing credits. The legacy GitHub handle `hishamkaram` and the email `hishamwaleedkaram@gmail.com` are historical identity handles — do not "correct" them.

## Versioning regime

- **v2 (the main line) lives at the repo root on `master`** with module path `github.com/hishamkaram/geoserver/v2`. The `/v2` suffix is required by Go's semantic import versioning rule for v2+ modules; it stays even though v2 is at the root. Latest published tag is `v2.0.0` — public API is stable; no breaking changes will land in v2.x. v2 has full v1 feature parity plus surfaces v1 never had: per-service OWS settings, file-upload publishing, layer–style associations, GeoWebCache (full surface), the Importer extension, the full ACL surface, the Resource API, the OWS read-only trio (GetCapabilities + DescribeFeatureType + DescribeCoverage), templates (FTL), auth providers / filters / chains, URL checks, cascaded WMS/WMTS, manifests + system status, runtime logging, fonts, password rotation, and monitoring (gs-monitor). Run `make test-unit` and `make test-integration`.
- **v1 is end-of-feature** on the `release/v1` branch (security patches only). The latest v1 tag is `v1.1.2` (deprecation marker via `// Deprecated:` in `go.mod`; no code change from `v1.1.1`). v1's wounds that need breaking changes (exported mutable fields, four-positional-arg `PublishPostgisLayer`, etc.) are healed in v2. v1.x is **still non-breaking** for the patches that do land on `release/v1` — no signature changes; deprecate via `// Deprecated:` and add a sibling. **Why:** v1.0 callers must be able to upgrade across the whole v1 line with only a `go.mod` bump.
- **Don't auto-tag releases.** Tagging is an explicit user action — never run `git tag` or push tags from a Claude session without explicit user authorization.

## Test split

- `*_unit_test.go` (no build tag) — **fast, hermetic, httptest-based.** Run by default.
- `*_test.go` with `//go:build integration` — **integration tests against a real GeoServer + PostGIS stack.** Never invoke without `--tags=integration` AND a live compose stack.
- The `make test-unit` and `make test-integration` targets are the canonical entry points; CI mirrors them exactly.
- **Both unit and integration tests are mandatory on every PR.** Integration runs against GeoServer 2.27.4 LTS and 2.28.0 stable; both legs must pass for the PR to merge.

## Context handling (mandatory for new exports)

v2 is **context-first**: every exported method on every sub-client takes `ctx context.Context` as its first argument. No `*Context` twins, no `context.Background()` delegators — that pattern was a v1.x compat shim and does not exist in v2. Canonical shapes: `rest/workspaces/workspaces.go:38,73`, `rest/about/about.go:84,99`.

```go
func (c *Client) GetFoo(ctx context.Context, args...) (...) { /* real impl uses ctx */ }
```

If you need a no-context entry point in caller code, use `context.Background()` at the call site — the library never papers over it.

## Typed errors

- All non-2xx GeoServer responses surface as `*APIError` (defined in `errors.go`). Renamed from v1's `*Error` as part of the v2 clean break.
- Status codes map to 12 sentinels via `errors.Is`: `ErrBadRequest`, `ErrUnauthorized`, `ErrForbidden`, `ErrNotFound`, `ErrMethodNotAllowed`, `ErrConflict`, `ErrUnsupportedMediaType`, `ErrRateLimited`, `ErrServerError`, `ErrBadGateway`, `ErrServiceUnavailable`, `ErrGatewayTimeout`.
- **Never compare error strings.** `errors.Is(err, ErrNotFound)` is the only correct test.
- The `*APIError.Error()` string format is `"geoserver: <Op> <Method> <URL>: <status> <statusText>: <body-preview>"` — body capped at 8 KiB internally and previewed at ~120 bytes. v1's `"abstract:%s\ndetails:%s\n"` format was deliberately dropped at the v2 boundary.

## Logging

- v2 uses `*slog.Logger` directly. The `*Logger` wrapper and `logging.go` from v1 do not exist — that wrapper was a v1.0-source-compat shim and was deliberately dropped.
- Configure via `WithLogger(l *slog.Logger)`. Pass `slog.New(slog.DiscardHandler)` to silence; the default is the discard logger.
- Internal call sites use structured logging — `logger.Debug(msg, args...)` with key/value pairs, not printf-style.
- Library logs Debug for HTTP details, Warn for retry-exhausted, Error for protocol violations. No Info chatter.

## Concurrency


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hishamkaram/geoserver](https://github.com/hishamkaram/geoserver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
