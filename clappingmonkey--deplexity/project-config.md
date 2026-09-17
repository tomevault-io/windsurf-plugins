---
trigger: always_on
description: bazel build //cmd/deplexity
---

# AGENTS.md

## Build & Run

```bash
# Build
bazel build //cmd/deplexity

# Build with version stamping
bazel build //cmd/deplexity --config=release

# Run tests
bazel test //...

# Run a specific test
bazel test //internal/client:client_test

# Run the binary directly
bazel run //cmd/deplexity -- export --help

# Regenerate BUILD files after adding new files/deps
bazel run //:gazelle

# Update deps after editing go.mod
go mod tidy
bazel run //:gazelle-update-repos
bazel run //:gazelle
```

Binary entrypoint: `cmd/deplexity/main.go`. Version/buildTime stamped via `x_defs` in Bazel (or `-ldflags` for plain `go build`).

## Architecture

- `internal/api/types.go` — **raw API response structs** (JSON tags match Perplexity's undocumented internal API, verified against live responses May 2026, API v2.18).
- `internal/api/threads.go` — `ListThreads` (POST `list_ask_threads` with pagination and dual stop condition), `GetThread`.
- `internal/api/collections.go` — `ListCollections` via `GET /rest/spaces`, deduplicated by non-empty UUID, then always-on fail-soft enrichment: `GetCollection` (per-space instructions/description/suggested_queries/primers) and `ListSpaceSkills`/`GetSkillDetail` (collection-scoped skills + SKILL.md body). `ListSpaceSkills` omits `collection_uuid` when the UUID is empty (used to list account-wide global skills).
- `internal/api/skills.go` — account-wide skills. `enrichSkill` (shared fail-soft helper: fetches detail + downloads SKILL.md body, used by both space and global paths), `ListGlobalSkills` (calls `ListSpaceSkills("")`, keeps only `scope=="global"`), and `GetAccount` (wraps global skills in `models.Account`). Global skills apply to every request account-wide, so they are exported **once** at the top level, not per space.
- `internal/api/user.go` — `GetUser` via `GET /api/user`.
- `internal/models/models.go` — clean domain models used throughout the app (decoupled from API shape).
- `internal/auth/` — browser-based login via `go-rod/rod` (visible Chrome), cookie capture, session persistence. Also supports `--cookie` for manual token auth.
- `internal/client/client.go` — authenticated `net/http` client with raw `Cookie` header, adaptive rate limiting, separate HTTP (429/5xx) and network (DNS/dial/TLS) retry loops. All methods accept `context.Context`. Also provides `GetRawURL` for fetching absolute third-party URLs (pre-signed S3 skill bodies) *without* the session cookie/Origin/`x-app-*` headers; it validates the target is `https` with a non-empty host and caps the body at 10 MB (`io.LimitReader`, errors on overflow rather than truncating).
- `internal/client/transport.go` — Chrome TLS fingerprint via `refraction-networking/utls` to bypass Cloudflare.
- `internal/client/ratelimit.go` — `RetryWithBackoff`, `computeBackoff`, shared retry constants.
- `internal/export/` — JSON, Markdown, and PDF exporters. PDF uses `gpdf` (pure Go, zero dependencies). JSON exporter handles `thread_index.json` persistence for resumable exports. All exporters copy thread files into space folders for self-contained output.
- `internal/export/util.go` — shared helpers: `sanitizeFilename`, `threadDirName` (slug-readable, UUID-identity thread directories), `spaceDirNames` (identity-suffixed, collision-free space directories shared by all exporters), and `skillFilenames`/`shortID` (collision-free `.md` filenames for a space's skills, disambiguated by a short skill-ID suffix; used by both the JSON and Markdown exporters so their sidecar paths and links agree).
- CLI framework: `alecthomas/kong` (struct-tag based). Commands defined as types with `Run(ctx context.Context) error` methods in `main.go`.

## Browser Dependency

**For `login` only:** Chrome or Chromium is needed for browser-based authentication. If not found, Rod automatically downloads Chromium (~80MB, one-time, cached at `~/.cache/rod/`).

**For `export --cookie` and PDF:** No browser required at any point. `login --cookie <TOKEN>` bypasses the browser entirely. PDF export uses `gpdf` (pure Go, no CGO) in isolated helper subprocesses of the Deplexity binary so cancellation and per-thread timeouts can terminate a stuck renderer.

## Perplexity API Details

All endpoints are reverse-engineered from browser DevTools (May 2026, API version 2.18).

- Auth cookie: `__Secure-next-auth.session-token` (NextAuth.js, ~7 day expiry)
- Session stored at: `~/.config/deplexity/session.json` (mode 0600, plain JSON)
- Required headers: `User-Agent` (Chrome), `Referer`, `Origin`, `x-app-apiclient: default`, `x-app-apiversion: 2.18`
- Cloudflare TLS fingerprinting bypassed via `utls` Chrome preset

### Endpoints Used

| Method | Endpoint | Purpose |
|--------|----------|---------|
| POST | `/rest/thread/list_ask_threads` | All threads with pagination (body: `{limit, offset, ascending, ...}`) |
| GET | `/rest/thread/{uuid}?with_schematized_response=true` | Thread detail with full entries |
| GET | `/rest/spaces` | All spaces (private/shared/invited/org/saved) — list only, omits instructions/skills |
| GET | `/rest/collections/get_collection?collection_slug={slug}` | Per-space detail: instructions, description, suggested_queries, primers (param must be `collection_slug`; `collection_uuid` → 422) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [clappingmonkey/Deplexity](https://github.com/clappingmonkey/Deplexity) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-17 -->
