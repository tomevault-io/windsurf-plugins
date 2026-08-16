---
trigger: always_on
description: This directory contains the Django REST Framework backend for the YPPF WeChat
---

## Overview

This directory contains the Django REST Framework backend for the YPPF WeChat
mini-program. These scoped instructions supplement the repository-wide rules
in [`../AGENTS.md`](../AGENTS.md); both apply to files under `api/`. The
mini-program frontend is maintained separately at
https://github.com/HelloWorldZTR/YPPF-mini.

## Development

When debugging is enabled, inspect the generated drf-spectacular schema at
`/api/schema/`, Swagger UI at `/api/docs/`, or ReDoc at
`/api/docs/redoc/`.

Use the development environment and project-wide test workflow documented in
[`../AGENTS.md`](../AGENTS.md). API-only tests may be run inside the configured
container with `python manage.py test api`, but changes that touch shared
models or website services require the full `python manage.py test` suite.
Mini-program configuration is declared in `api/config.py` and stored under
the `wx_miniapp` section of the local `config.json`; follow the root
configuration rules and never hard-code or commit credentials.

## File Structure

`boot/urls.py` mounts `api/urls.py` at `/api/`. The API root router then mounts
feature modules under `/api/v2/<module>/`. Do not place endpoint
implementations directly in the `api/` root. Each feature module should own
its `urls.py`, `views.py`, serializers, tests, and feature-specific helpers,
then be registered with `include()` in `api/urls.py`.

| Path | Responsibility | Route prefix |
| --- | --- | --- |
| `urls.py` | Root mini-program router and debug-only OpenAPI, Swagger, and ReDoc registration. | `/api/`; feature APIs begin at `/api/v2/`, while documentation uses `/api/schema/`, `/api/docs/`, and `/api/docs/redoc/`. |
| `authentication.py` | Shared DRF authentication classes for JWT and one-time webview tickets, plus schema integration. | Used by views and authentication settings; it has no routes of its own. |
| `config.py` | Typed `WXMiniappConfig` access to app ID, secret, WeChat session endpoint, token lifetimes, and ticket TTL. | No routes. |
| `auth/` | WeChat code login, account binding/unbinding, login checks, account listing, and one-time webview ticket exchange. | `/api/v2/auth/` |
| `user/` | Current-user profile and daily-login reward. | `/api/v2/user/` |
| `notification/` | Notification listing, statistics, and read/status updates. | `/api/v2/notification/` |
| `feedback/` | Feedback metadata, creation, listing, detail, and updates. | `/api/v2/feedback/` |
| `appoint/` | Room availability, appointment creation/cancellation/renewal, checkout, violations, agreements, and user search. | `/api/v2/appoint/` |
| `activity/` | Activity homepage/listing and activity signup/check-in operations. | `/api/v2/activity/` |
| `library/` | Library welcome/configuration, book search, reader data, and lending records. | `/api/v2/library/` |
| `YQpools/` | YQPoint balance, prize-pool listing, exchange, lottery, and random purchase. Preserve the directory and URL capitalization. | `/api/v2/YQpools/` (case-sensitive) |
| `org/` | Organization subscription listing and subscription status updates. | `/api/v2/org/` |
| `generic/` | Cross-feature mini-program data, currently the homepage carousel. | `/api/v2/generic/` |

Many modules use DRF `DefaultRouter`; routes generated from ViewSets and
`@action` methods may therefore not appear as explicit `path()` entries. When
adding an endpoint, inspect both the module router and `api/urls.py`, preserve
existing basename and route naming conventions, add serializers for request
and response contracts, and add tests in the same feature module.

## Authentication and Interaction

The mini-program uses three separate credentials. They are not
interchangeable:

| Credential | Purpose | Issued by | Lifetime and storage | Transport |
| --- | --- | --- | --- | --- |
| `signed_openid` | Proves the WeChat identity only during first-time account binding. It is not an API login credential. | `_sign_openid()` after an unbound `wx.login` exchange. | `signed_openid_ttl_minutes` (10 minutes by default). Signed with Django's `TimestampSigner`; it is short-lived but not consumed on use. | JSON body of `POST /api/v2/auth/wx/bind/`. |
| JWT access token | Authenticates normal mini-program API requests and identifies the currently selected person or organization account. | `_issue_jwt_for_user()` after login or binding. | `token_expire_minutes` (120 minutes by default). The client stores it and obtains a new token by logging in again; there is currently no refresh-token endpoint. | `Authorization: Bearer <token>`. Never place it in a URL. |
| WebView ticket | Converts an authenticated mini-program identity into a Django session for a website WebView without exposing the JWT in the URL. | `POST /api/v2/auth/ticket/`, which requires a valid JWT. | `ticket_ttl_seconds` (60 seconds by default). Stored in Django cache and deleted when consumed. | Query parameter to `/redirect/?ticket=<ticket>&to=<path>`. |

The relevant implementation is split across `api/auth/views.py` (flows and
token issuance), `api/auth/ticket.py` (ticket creation/consumption),
`api/authentication.py` (DRF authenticators), `generic/models.py`
(`UserWechatProfile`), and `generic/views.py` (the WebView redirect bridge).

### First-time binding


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Yuanpei-Intelligence/YPPF](https://github.com/Yuanpei-Intelligence/YPPF) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
