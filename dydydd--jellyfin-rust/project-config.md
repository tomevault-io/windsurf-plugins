---
trigger: always_on
description: - This repository reimplements the Jellyfin server in Rust while preserving compatibility with the official Jellyfin API and web client.
---

# Jellyfin Rust contributor guide

## Scope

- This repository reimplements the Jellyfin server in Rust while preserving compatibility with the official Jellyfin API and web client.
- The checked-out official server source in `jellyfin/` is the behavioral reference. Prefer matching its externally visible behavior, defaults, validation, authorization, ordering, and error semantics over inventing new behavior.
- Current optimization priorities are media-library management and scanning, users and policies, metadata scraping/providers, and PostgreSQL-backed data access.
- Do not work on Live TV unless a task explicitly asks for it. Avoid incidental changes under `src/jellyfin-live-tv`.
- Treat `Emby.ApiClients/Clients/Go/api/swagger.yaml` and its generated Java and Swift clients as
  the Emby wire contract. Keep Emby endpoints under `/emby`, register protocol-specific adapters
  before the shared Jellyfin fallback, and never change an unprefixed Jellyfin response merely to
  satisfy an Emby-only DTO.
- Keep the checked-in Emby operation inventory synchronized with the generated client source.
  Exclude only plugin operations, `/LiveTv` routes, and QuickConnect unless a task explicitly
  includes them;
  ordinary playback `LiveStreams` routes remain in scope. Track every other missing method/path in
  the explicit gap ledger, remove entries only with route and response-shape tests, and keep the
  combined server test proving `/emby` and Jellyfin root routes remain isolated.
- Normalize empty successful shared-handler responses to HTTP 200 only inside the `/emby` tree,
  because the generated 4.10.0.40 document declares 200 as the sole success status for all 548
  operations. Preserve Jellyfin's root and `/api` 204 mutation responses.
- Bind Emby collection and playlist mutation `Ids`/`EntryIds` query strings case-insensitively,
  using only the last repeated scalar value before applying their comma-delimited collection
  semantics. Keep create-route ids optional, require add/remove ids, and do not change Jellyfin root
  or `/api` query binding.
- Require a nonblank, case-insensitively bound, last-duplicate-wins query `Container` on Emby's
  extensionless and filename-form Audio/Video progressive routes and on their master/live/main HLS
  manifests. Path-container stream routes satisfy the generated contract from their suffix, while
  Universal audio and subtitle/BIF routes must not acquire this requirement; preserve Jellyfin root
  and `/api` behavior.
- Require `Size` on `/emby/Playback/BitrateTest`, bind it case-insensitively with the last duplicate
  winning, and accept API keys as generated-client authentication. Preserve Jellyfin root and
  `/api` default size 102400 and the shared inclusive 1..100000000 validation.
- Require one nonblank, case-insensitively bound, last-duplicate-wins `Id` for Emby's DELETE
  `/Devices` and POST `/Devices/Delete`. Keep Jellyfin's root and `/api` repeated/comma-separated
  device-id binder and its omitted-id no-op behavior unchanged.
- Bind the required `Id` query on `/Devices/Info` and `/Devices/Options` case-insensitively with the
  last duplicate winning on Jellyfin root, `/api`, and `/emby`. Apply the same semantics to every
  known `DeviceOptionsDto` JSON property, including `CustomName`, and ignore unknown properties
  without changing the existing authorization and success-status differences between protocols.
- Require one nonempty, strictly valid comma-delimited `Ids` scalar on Emby's DELETE `/Items` and
  POST `/Items/Delete`. Bind its name case-insensitively with the last duplicate scalar winning,
  reject invalid or empty collection elements before deleting anything, and preserve Jellyfin root
  and `/api` omitted/repeated-query behavior.
- Require a valid JSON `PlaybackInfoRequest` body on Emby's POST
  `/Items/{Id}/PlaybackInfo`, while preserving the optional body accepted by Jellyfin root and
  `/api`. Do not impose this POST-only body requirement on the generated GET operation.
- Never serialize Jellyfin GUID strings into Emby's `NameLongIdPair.Id` fields. Project BaseItem
  `Studios` and `GenreItems` on `/emby` with the stable private `item_values.emby_id` BIGINT identity,
  preserving the accompanying string `Genres`; relations without a stable numeric mapping such as
  `TagItems` and `Collections` must remain omitted. Keep names and GUID ids unchanged on every
  unprefixed Jellyfin response, batch-load the numeric ids with the normal relation query, and avoid
  whole-response buffering to adapt these fields.
- Project Emby's top-level BaseItem `Size`, `Bitrate`, and `FileName` only on `/emby`: source them
  from persisted media metadata and the local item path, with inferred projected-source bitrate as
  a fallback. Keep Jellyfin root and `/api` responses unchanged. Persist the local `.strm` shortcut
  file length during scans and rescans, matching the official server; do not stat the remote target
  or perform filesystem I/O in the item-detail request path.
- Keep Emby BaseItem enum adaptation protocol-local as well: omit unsupported Person `Type` values,
  filter Jellyfin-only `Lyric` media streams, omit the Jellyfin-only `Drop` subtitle delivery method,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dydydd/jellyfin-rust](https://github.com/dydydd/jellyfin-rust) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
