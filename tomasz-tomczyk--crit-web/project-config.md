---
trigger: always_on
description: Hosted Phoenix LiveView app that receives shared reviews from the [Crit](https://github.com/tomasz-tomczyk/crit) local CLI and renders them at `/r/:token`. Same review surface as the local tool — see `../CLAUDE.md` for the parity contract.
---

# Crit Web — Development Guide

Hosted Phoenix LiveView app that receives shared reviews from the [Crit](https://github.com/tomasz-tomczyk/crit) local CLI and renders them at `/r/:token`. Same review surface as the local tool — see `../CLAUDE.md` for the parity contract.

## Project map

```
crit-web/
├── lib/
│   ├── crit/                        # Domain logic
│   │   ├── application.ex           # OTP app supervision tree
│   │   ├── repo.ex                  # Ecto repo
│   │   ├── schema.ex                # Base schema module
│   │   ├── review.ex                # Review schema (token, delete_token, last_activity_at, review_round, cli_args)
│   │   ├── comment.ex               # Comment schema (review_id, parent_id, start_line, end_line, body, scope, resolved, author_identity, author_display_name, file_path, quote, external_id)
│   │   ├── review_round_snapshot.ex # Per-round snapshot of review files
│   │   ├── reviews.ex               # Context: create/get/delete reviews with comments (10 MB total limit)
│   │   ├── review_cleaner.ex        # Periodic cleanup of inactive reviews (30 days)
│   │   ├── output.ex                # Formats review data for API responses
│   │   ├── display_name.ex          # Author display name logic (40-char max)
│   │   ├── integrations.ex          # Integration metadata (editors, AI tools)
│   │   ├── changelog.ex             # GenServer: fetches and caches GitHub releases
│   │   ├── rate_limit.ex            # Hammer-based rate limiting
│   │   ├── release.ex               # Release migration helpers
│   │   ├── config.ex                # Runtime config helpers
│   │   ├── statistics.ex / statistic.ex   # Usage statistics
│   │   ├── accounts.ex + accounts/scope.ex # Phoenix 1.8 scope-based auth
│   │   ├── user.ex / user_api_token.ex     # Authenticated user + CLI bearer tokens
│   │   ├── device_codes.ex / device_code.ex / device_code_cleaner.ex # OAuth device flow
│   │   ├── sentry_filter.ex / sentry_http_client.ex # Sentry plumbing
│   ├── crit_web/
│   │   ├── router.ex                # Routes: marketing, /r/:token, /dashboard, /settings, /overview, /api/*, /api/device/*, /api/auth/*, /auth/cli/*
│   │   ├── endpoint.ex              # Phoenix endpoint
│   │   ├── user_auth.ex             # Auth plug + on_mount hooks; sets current_scope
│   │   ├── controllers/             # page, review, api, auth, oauth, health, device, device_api, auth_api
│   │   ├── live/
│   │   │   ├── review_live.ex       # LiveView for /r/:token
│   │   │   ├── review_live.html.heex # Review page template (uses crit-* CSS classes)
│   │   │   ├── dashboard_live.ex    # User dashboard
│   │   │   ├── settings_live.ex     # User settings
│   │   │   ├── overview_live.ex     # Selfhost admin overview
│   │   │   └── tokens_live.ex       # CLI token management
│   │   ├── components/              # core_components.ex, layouts.ex
│   │   └── plugs/                   # security_headers, rate_limit, api_auth, require_bearer_auth, localhost_cors, canonical_host
│   └── mix/tasks/
│       └── crit.refresh_integrations.ex
├── assets/
│   ├── js/
│   │   ├── app.js                   # Phoenix JS setup + LiveView hooks
│   │   └── document-renderer.js     # Port of crit local's rendering logic
│   └── css/
│       └── app.css                  # Review page CSS (crit-* classes) + Tailwind
├── priv/repo/migrations/
├── config/                          # Dev/test/prod/runtime config
├── test/                            # ExUnit tests
└── .github/workflows/ci.yml         # CI: format, compile, sobelow, audit, test
```

## Key architecture

1. **Review page rendering** — the LiveView loads review data, then `document-renderer.js` (a Phoenix hook) renders the markdown client-side using markdown-it + highlight.js + mermaid. Mirrors `crit` local's rendering.
2. **API for CLI uploads** — `POST /api/reviews` accepts review files + comments + metadata from the CLI's Share button. `PUT /api/reviews/:token` upserts updates and bumps `review_round`. Returns `{url, delete_token}`.
3. **Delete via token** — reviews are deleted by passing the `delete_token` (not auth). The CLI stores this in the review file.
4. **Rate limiting** — Hammer-based via `CritWeb.Plugs.RateLimit`, applied across browser + API pipelines.
5. **Auth + identity** — Phoenix 1.8 scope pattern: `Crit.Accounts.Scope` carries either an authenticated `user` (OAuth or selfhost password) or an anonymous `identity` (session-bound visitor ID), plus a `display_name`. `CritWeb.UserAuth` plug + `on_mount` hooks set `conn.assigns.current_scope` / `socket.assigns.current_scope`. `user` and `identity` are mutually exclusive — see `lib/crit_web/CLAUDE.md` for the full scope contract.
6. **CLI auth** — OAuth device flow (`/api/device/*`, `/auth/cli/*`) issues bearer tokens (`UserApiToken`) used by the CLI. `Plugs.RequireBearerAuth` gates `/api/auth/*`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tomasz-tomczyk/crit-web](https://github.com/tomasz-tomczyk/crit-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
