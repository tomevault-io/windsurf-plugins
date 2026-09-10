---
trigger: always_on
description: Working notes for this repo. Written after the Audiobookshelf 2.36.0 compliance pass;
---

# CLAUDE.md

Working notes for this repo. Written after the Audiobookshelf 2.36.0 compliance pass;
the aim is that the next server release can be handled the same way without rediscovering
the same traps.

This is a Home Assistant custom integration. It polls an Audiobookshelf server with a
single static admin credential and exposes counts as sensors. It is read-mostly: the only
write is the `remove_my_progress` service.

## Checking the integration against a new Audiobookshelf release

Release notes are a starting point, not evidence. They describe intent and routinely omit
schema changes. Verify against the tagged source.

The cheapest high-confidence check is comparing blob SHAs between tags — byte-identical
files cannot have changed, which settles most questions without reading any code:

```bash
gh api repos/advplyr/audiobookshelf/compare/vOLD...vNEW --jq '.files[].filename'
```

Then read the actual hunks for anything that did change, at
`https://raw.githubusercontent.com/advplyr/audiobookshelf/vNEW/server/...`.

### Endpoints this integration depends on

Check each of these on every server release. The first six back sensors; the rest back
the `remove_my_progress` service.

| Endpoint | Used for | Notes |
| --- | --- | --- |
| `GET /api/libraries` | library discovery | |
| `GET /api/users` | `count_users` | admin only, 403 otherwise |
| `GET /api/users/online` | `count_users_online` | admin only, 403 otherwise |
| `GET /api/sessions/open` | open + recent sessions | admin only, **404** (not 403) otherwise |
| `GET /api/libraries/{id}/stats` | per-library sensors | |
| `GET /api/me/sessions` | `count_auth_sessions` | added in 2.36.0; 404s on older servers |
| `GET /api/libraries/{id}/items` | service | paginated |
| `GET`/`DELETE /api/me/progress/...` | service | |

`POST /api/authorize` is not in the table because the library calls it, not this integration —
`get_admin_client_by_token` posts it to build the client. Its response carries
`serverSettings.version`, which the library exposes as `client.server_settings.version`. That is
where the device's `sw_version` comes from, so it costs no request of its own. It is only
refreshed when the client is rebuilt, so a server upgrade is not reflected until the entry
reloads.

Audiobookshelf exposes **no update-check endpoint**. `/api/check-for-update`, `/api/update`,
`/api/version` and `/api/server-settings` all 404 on 2.36.0; the web UI queries GitHub from the
browser. Anything reporting "an update is available" has to ask GitHub directly, which is why
this integration does not do it today.

`/api/libraries/{id}/stats` omits `totalAuthors` for **podcast** libraries. `LibraryStats.total_authors`
must stay `int | None`. This is not hypothetical — it was confirmed against a real server
with a podcast library, and a required field there would break every poll.

## Authentication

Audiobookshelf 2.26.0 reworked auth. Three credential types now exist, and only some work
as a `Bearer` token:

- **API keys** (Settings > API Keys, 2.26.0+) — long-lived, the supported option. Recommend these.
- **Legacy user tokens** (Settings > Users) — still work, but the UI labels them "Legacy API
  Token" and warns of removal. Fallback for servers older than 2.26.0 only.
- **Refresh tokens** — rejected for API auth since 2.36.0. This integration never sends one.

The credential must belong to an **admin/root** user. `verify_config` uses
`get_admin_client_by_token` deliberately so a non-admin key is rejected in the config flow
rather than failing on every poll afterwards.

`verify_config` is only reachable from the config flow now. It used to run again in
`async_setup_entry`, which duplicated a request on every restart and reload over its own
`ClientSession`; `async_config_entry_first_refresh` already gates setup. A non-admin key that
somehow reaches setup is caught by the coordinator, which handles `BadUserError` ahead of
`AbsAuthError` to say so specifically.

### The exception trap

This is the single easiest thing to get wrong here.

`aioaudiobookshelf` exceptions derive from `AbsError`, **not** `aiohttp.ClientError`. Catching
only `ClientError` silently misses all of them.

Worse, because this integration configures a static `token` with no refresh token, a 401 does
**not** raise `LoginError`. The library sees `auto_refresh=True`, tries to refresh, finds no
refresh token, and raises `TokenIsMissingError`. Both are `AbsAuthError`, so catch that:

- `AbsAuthError` -> `ConfigEntryAuthFailed` (triggers the reauth flow)
- other `AbsError` -> `UpdateFailed`

`NotFoundError` (HTTP 404) is *not* an `AbsAuthError` — that is what lets `count_auth_sessions`
degrade to `None` on pre-2.36.0 servers instead of prompting for reauth.

`BadUserError` (non-admin credential) *is* an `AbsAuthError`, so it must be caught before it if
you want a message that says so rather than a generic auth failure.

There is a third clause that is easy to leave out. Every `from_json` call raises **mashumaro**
exceptions on schema drift — `MissingField` (a `LookupError`) and `InvalidFieldValue` (a
`ValueError`) — and a non-JSON body raises `JSONDecodeError` (also a `ValueError`). None of

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wolffshots/hass-audiobookshelf](https://github.com/wolffshots/hass-audiobookshelf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
