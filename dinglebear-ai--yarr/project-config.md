---
trigger: always_on
description: provides its own dynamic-discovery/Code Mode layer (e.g. Labby): in `codemode` mode
---

# yarr — Claude Code instructions

## What this project is

Rust MCP and CLI server for a media automation fleet. It wraps **11 service kinds**: Sonarr, Radarr, Prowlarr, Overseerr, Jellyfin, Plex (spec-backed, generated) plus SABnzbd, qBittorrent, Tautulli, Bazarr, Tracearr (doc-backed, curated).

| Fact | Value |
|------|-------|
| Repo | `git@github.com:dinglebear-ai/yarr.git`, default branch `main` |
| Cargo workspace | 2 members — `.` (bin+lib `yarr`) and `xtask` |
| Edition / MSRV | 2024 / Rust 1.97.1 |
| MCP crate | `rmcp = "=3.0.0-beta.2"` via `[workspace.dependencies]` (`server`, `macros`, `transport-streamable-http-server`, `transport-io`, `schemars`, `elicitation`) |
| Service port | `40070` (`YARR_MCP_PORT`) |
| npm launcher | `@dinglebear/yarr@<Cargo version>`, pinned — never `latest` |

The MCP surface is a single `yarr` tool that runs Code Mode (the `codemode` action). The 6 spec-backed services (sonarr/radarr/prowlarr/overseerr/jellyfin/plex) are reached through **generated** per-service callables (from vendored OpenAPI specs); download/stats/subtitles/trace keep curated commands; every service also has `service_status` + the `api_get/post/put/delete` generic passthrough. Services are declared via `YARR_SERVICES` plus per-service env (see Environment variables).

## Module map

**Transport (`src/yarr*`)**

| File | Role |
|------|------|
| `src/yarr.rs` | `YarrClient` — pooled HTTP transport facade over configured service identities |
| `src/yarr/auth.rs` | Per-service auth application, driven by `AuthStyle` from the `KindDescriptor` table (header / query key / cookie session / Plex / Jellyfin tokens) |
| `src/yarr/helpers.rs` | `validate_service_path` (descriptor path allowlists, S7), `query_get` (percent-encodes user text for query APIs, S6), `slim()` field selection, error-body redaction |
| `src/yarr/openapi_transport.rs` | Lossless generated-operation request serialization and response decoding, including non-JSON/binary bodies |
| `src/yarr/response.rs` | Bounded upstream body collection and sanitized response/error handling |

**Capability model**

| File | Role |
|------|------|
| `src/capability.rs` | `Capability` enum + `KindDescriptor` table (`ServiceKind::descriptor()`): api prefix, auth style, resource noun, path allowlist, `has_metadata_profiles`. SSOT for "what each kind can do" |
| `src/config/services.rs` | The `ServiceKind` enum itself plus the `KindRow` table (canonical name, default status path, `FromStr` aliases). Adding a kind is a one-row edit + enum variant |

**Business layer (`src/app*`) — all logic lives here, never in shims**

| File | Role |
|------|------|
| `src/app.rs` | `YarrService` — business-layer facade; `execute_service_action` shared dispatch entry |
| `src/app/openapi_ops.rs` + `app/openapi_ops/` | Generated-operation executor: one `(service, op, args)` → upstream request for the 6 spec-backed kinds (sonarr/radarr/prowlarr/overseerr/jellyfin/plex). No per-op code — see `src/openapi*` |
| `src/app/download.rs` + `app/download/{sab,qbit}.rs` | DownloadClient — per-client implementations (SAB query API, qBittorrent v2 REST/cookie) |
| `src/app/stats.rs` | Stats (tautulli) activity/history/users/libraries plus maintenance writes, all run immediately (`delete_image_cache` is destructive — elicited on MCP); `{response}` envelope unwrap |
| `src/app/subtitles.rs` | Bazarr subtitle status, inventory, wanted, provider, and language reads |
| `src/app/trace.rs` | Tracearr health, analytics, streams, users, violations, history, and elicited stream termination |
| `src/app/codemode_{runtime,dispatch,artifacts,snippets}.rs` | Bounded Code Mode orchestration, independently authorized inner dispatch, quota-managed artifacts, and atomic snippet lifecycle |

The 6 spec-backed kinds have **no hand-written app modules** (the old `arr`/`indexer`/
`media_server`/`requests` capability handlers were removed) — their entire API is
generated. The four doc-backed capabilities keep curated commands: `download`
(sabnzbd/qbittorrent), `stats` (tautulli), `subtitles` (bazarr), and `trace`
(tracearr). They also retain the reviewed generic passthrough.

**Generated OpenAPI surface (`src/openapi*` + `specs/`)**

The 6 spec-backed services are generated from the vendored OpenAPI specs under
`specs/` by `cargo xtask gen-openapi` — 1,352 supported operations + 808 component types
total. Inside Code Mode they are per-service callables (`sonarr.get_series()`,
`radarr.post_movie({body})`, …) dispatched through the `op` action; component types
are surfaced via `codemode.describe`.

| File | Role |
|------|------|
| `src/openapi.rs` | Lossless `OperationSpec`/parameter/request/response/`TypeDef` runtime shapes + supported/omitted per-kind registries |
| `src/openapi/generated.rs` + `generated/<svc>.rs` | GENERATED tables (`OPERATIONS`, `TYPES`) — do not edit; regenerate with `cargo xtask gen-openapi` |
| `xtask/src/gen_openapi.rs` | The generator: parse spec `paths`+`components` → emit operation table + TS-interface type catalog |

**Code Mode (`src/codemode*` + `src/app/codemode.rs`)**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dinglebear-ai/yarr](https://github.com/dinglebear-ai/yarr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
