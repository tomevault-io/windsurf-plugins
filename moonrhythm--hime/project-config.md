---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Hime is a minimalist Go web framework built on top of `net/http`. Its core design principle (see `README.md`): it ships **no router or middleware of its own** so it composes with any `net/http`-compatible router, middleware, or handler. Hime only adds an app container, a request `Context` with response helpers, named routes, globals, and an `html/template`-based view/component system.

The module is `github.com/moonrhythm/hime` and requires **Go 1.25**. It is a library (no `main` package).

## Commands

```bash
go test ./...                       # run all tests
go test -run TestApp ./...          # run a single top-level test
go test -run 'TestApp/Clone' ./...  # run a single subtest (tests use t.Run subtests heavily)
go vet ./...                        # static checks (part of CI)
```

CI (`.github/workflows/test.yaml`) runs `go vet` then `go test ./...` on Go 1.25. Tests rely on fixtures in `testdata/` (templates, YAML configs, `server.crt`/`server.key`). Some tests bind real ports (`:8081`, `:8082`) and `time.Sleep` — they run with `t.Parallel()`.

## Architecture

The framework is a flat package (`package hime`); each `*.go` file has a matching `*_test.go`. The pieces fit together as follows:

### App and the context-injection pattern (`app.go`)
`App` is the central container, wrapping a `*parapet.Server` (`github.com/moonrhythm/parapet`) rather than `http.Server`. It holds routes, globals (`sync.Map`), templates, components, and a `parent *template.Template`.

The key mechanism: `App.ServeHTTP` / `App.ServeHandler` wrap the downstream handler and **inject the app into every request's `context.Context`** under `ctxKeyApp{}`. Everything else (`NewContext`, package-level `Route`/`Global`) retrieves it via `getApp(ctx)`, which **panics with `ErrAppNotFound`** if the app isn't present. So any hime `Context` or helper only works for requests that passed through the app's handler chain.

`App.Clone()` deep-copies server settings, routes, globals, and templates — used to derive variant apps (e.g. per-host) from a base.

### Handler and the error-as-panic convention (`handler.go`)
`type Handler func(*Context) error` adapts to `http.Handler`. In `ServeHTTP`, a returned error is treated as fatal: `nil` and `context.Canceled` are swallowed, **everything else panics**. There is no built-in error rendering — recovery is expected to come from parapet/middleware upstream. When adding handler logic, return errors only for truly exceptional cases; use the `Context` result helpers for normal responses.

### Context and result helpers (`context.go`, `request.go`)
`Context` embeds `*http.Request` and itself implements `context.Context` (delegating to the request's context). Construct it with `NewContext(w, r)`.

Response "results" are methods that write to the response and return `error` (meant to be returned from a `Handler`): `View`, `Component`, `Render`, `JSON`, `XML`, `HTML`, `String`, `Bytes`, `CopyFrom`, `File`, `Redirect`/`SafeRedirect`/`RedirectTo`/`RedirectBack`, `Error`, `NotFound`, `NoContent`, `StatusText`. Status is set fluently via `ctx.Status(code)` and read back via `ctx.StatusCode()`. Request bodies bind via `BindJSON`/`BindXML`.

- **ETag**: when enabled (`App.ETag` or `ctx.ETag(true)`), `View`/`Component`/`Render`/`JSON`/`XML`/`HTML` compute a weak ETag over the rendered bytes and return `304` on `If-None-Match` match (`setETag`). This forces buffering of output.
- **`filterRenderError`** swallows broken-pipe / `net.OpError` / `EPIPE` errors so client disconnects aren't treated as handler failures.
- `request.go` adds typed value helpers that trim spaces and strip commas, in three parallel families — `FormValue*` (query+body), `PostFormValue*` (body), and `QueryValue*` (query only) — plus multi-value slice getters (`FormValues`/`PostFormValues`/`QueryValues`) and `FormFileNotEmpty`/`FormFileHeader`.
- `cookie.go` adds opt-in signed cookies: a `CookieSigner` interface plus an HMAC-SHA256 reference impl (`NewHMACCookieSigner`). Set `app.CookieSigner` (a public field, like `app.ETag`) to enable `AddSignedCookie`/`SignedCookieValue`, which read the signer off the app and panic if it's unset. The signer binds the cookie name into the MAC and verifies in constant time; it signs but does not encrypt. No lock-in — you supply your own signer.
- `htmx.go` adds opt-in [htmx](https://htmx.org) helpers on `Context`: `IsHTMX()` (detect the `HX-Request` header), `HTMXRedirect`/`HTMXRefresh`, and chainable `HTMXReswap`/`HTMXRetarget`/`HTMXTrigger` that set `HX-*` response headers. Thin wrappers, no client runtime beyond htmx itself.
- `flash.go` adds stateless flash messages (`AddFlash`/`Flashes`) for the post/redirect/get pattern, carried in a plain base64-JSON cookie (`flash`) that's cleared on read. Unsigned — non-sensitive UX only; the `flash` field on `Context` accumulates outgoing messages within a request.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [moonrhythm/hime](https://github.com/moonrhythm/hime) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
