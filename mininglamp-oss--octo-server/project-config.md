---
trigger: always_on
description: This file provides guidance to coding agents when working in this repository.
---

# AGENTS.md

This file provides guidance to coding agents when working in this repository.

## Project Overview

Octo-server is the Go backend for DMWork, an enterprise IM platform. It handles
business logic on top of WuKongIM for messaging transport.

- Go module: `github.com/Mininglamp-OSS/octo-server`
- Go version: 1.25
- Shared library: `github.com/Mininglamp-OSS/octo-lib` for config, wkhttp,
  testutil, register, and model packages
- Default branch: `main`

## Common Commands

```bash
# Build
docker build -t octo-server .

# Run tests for a module
go test ./modules/group/...
go test ./modules/message/ -run TestSendMsg

# Run all tests
go test ./...

# Lint
golangci-lint run ./...
```

## Architecture

### Request Flow

```text
HTTP (Gin/wkhttp) -> Auth Middleware (pkg/auth/) -> Space Middleware -> API Handler -> Service -> DB (MySQL/DBR)
                                                                                       |
                                                                                       v
                                                                                  WuKongIM (gRPC)
```

### Module System

Modules live in `modules/`. Each module is auto-registered with `init()` and
`register.AddModule()`.

Standard module layout:

- `1module.go`: registration entry using `init()` and `register.AddModule()`
- `api*.go`: HTTP handlers implementing `register.APIRouter.Route(r *wkhttp.WKHttp)`
- `service.go`: business logic, usually with an `IService` interface
- `db*.go`: database operations using `gocraft/dbr`
- `model.go`: data models and response structs
- `sql/`: SQL migrations embedded with `//go:embed sql`

### Key Packages

| Package | Purpose |
| --- | --- |
| `pkg/auth/` | Token parsing, `CacheTokenParser`, auth middleware |
| `pkg/errcode/` | Error code definitions per module, such as group, message, user, and OIDC |
| `pkg/httperr/` | `ResponseErrorL` and `ResponseErrorLWithStatus` error facades |
| `pkg/i18n/` | Localization SDK: code registry, localizer, renderer, language negotiation, and `locales/` |
| `internal/` | Internal wiring and module imports |
| `modules/base/event/` | Async event system |

## Error Handling and i18n

All user-facing error responses go through the i18n error envelope. Do not use
`c.ResponseError(errors.New(...))`, `c.ResponseErrorf(...)`,
`c.AbortWithStatusJSON(...)`, or non-OK `c.JSON(...)`; these are legacy patterns
and bypass the localized envelope.

Use the facades in `pkg/httperr`:

| Facade | Wire status | Use for |
| --- | --- | --- |
| `ResponseErrorL(c, code, params, details)` | Pinned 400 for D14 compatibility; real status in `error.http_status` | Default for legacy-bearing endpoints |
| `ResponseErrorLWithStatus(c, code, params, details)` | The code's real `HTTPStatus` | New endpoints only when no clients depend on fixed 400; maintainer sign-off is required when diverging from D14 |

```go
httperr.ResponseErrorL(c, errcode.ErrGroupQueryFailed, nil, nil)
```

Register error codes in `pkg/errcode/<module>.go`:

```go
ErrXxx = register(codes.Code{
    ID:             "err.server.<module>.<reason>", // or reuse err.shared.* codes
    HTTPStatus:     http.StatusBadRequest,
    DefaultMessage: "English source (D4).",
    SafeDetailKeys: []string{"field"},
    Internal:       false,
})
```

Error-code rules:

- 5xx codes must set `Internal=true`; 4xx codes must not. The renderer hides
  messages and details for internal errors, so log the cause with `zap.Error`
  before responding.
- Auth and verification failures use one generic anti-enumeration code, such as
  a single 401. Specific failure reasons go to logs only.
- `params` interpolate into the message template. `details` are structured
  client-facing fields filtered by `SafeDetailKeys`.
- Per-module helpers live in `modules/<module>/api_i18n.go`.
- `mustLookupSharedCode` resolves shared codes at init and should panic loudly
  when a shared code is not registered.

After adding or changing error codes, run:

```bash
make i18n-extract
make i18n-extract-check
make i18n-lint
```

Then add the zh-CN translation to `pkg/i18n/locales/active.zh-CN.toml` with one
`["id"]` block and `other = "..."` value per code.

Each migrated module has a `Test<Module>NoLegacyResponseError` source guard
that forbids legacy and raw error responses. Add new handler files to that
guard's file list. Protocol endpoints that intentionally keep raw responses,
such as OAuth2/OIDC browser-redirect flows, are exempted through
`tools/lint-direct-error-response/baseline.txt`.

Localized email templates live in
`modules/base/common/emailtmpl/templates/{lang}/`. Send functions take a `lang`
argument resolved through `i18n.OutboundLanguage(ctx)`; do not hardcode
localized subject or body strings.

## Rate Limiting

Use the shared middleware in octo-lib `pkg/wkhttp/ratelimit.go`. Do not
hand-roll Redis `INCR` and TTL counters for generic request-frequency limiting.

The rate limit layers set `X-RateLimit-Limit`, `X-RateLimit-Remaining`,
`X-RateLimit-Scope`, and `X-RateLimit-Retry-After` headers, return the i18n
`rate.limited` response, and fail open on Redis errors.

| Middleware | Scope header | Dimension | Use for |
| --- | --- | --- | --- |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Mininglamp-OSS/octo-server](https://github.com/Mininglamp-OSS/octo-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
