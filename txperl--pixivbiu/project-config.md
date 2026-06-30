---
trigger: always_on
description: [PixivBiu](https://github.com/txperl/PixivBiu) is a Pixiv artwork browsing, searching, and downloading tool. **v3 is its next major version**, adopting Go as the implementation language and evolving the project structure to a decoupled backend + frontend monorepo.
---

# AGENTS.md

## Project Overview

[PixivBiu](https://github.com/txperl/PixivBiu) is a Pixiv artwork browsing, searching, and downloading tool. **v3 is its next major version**, adopting Go as the implementation language and evolving the project structure to a decoupled backend + frontend monorepo.

The current backend covers **auth + read-only browsing + bookmark/follow + download + SSE event stream** (all backed by [github.com/txperl/pixivgo](https://github.com/txperl/pixivgo)), plus **version reporting & one-click self-update from a minisign-signed release feed on Cloudflare R2 (CDN)**, and a **same-origin image proxy with on-disk cache**. Search cache and SauceNAO remain intentionally deferred.

## Architecture Overview

| Component | Tech Stack | Default Port | Description |
|-----------|-----------|--------------|-------------|
| **Backend** | Go + chi + oapi-codegen | 4001 | REST API server (binds `127.0.0.1` by default). OpenAPI-first: routes and types generated from `api/openapi.yaml`. |
| **Frontend** | React 19 + Vite + TypeScript | 5173 (dev) | SPA. Talks to backend via `/api/v1/*`. |

For **production the built SPA is embedded into the Go binary** (`go:embed`, `internal/web`) and served by the same server at `/`, so a release is a single self-contained executable and the frontend calls `/api` same-origin (no CORS). The Vite dev server (5173) only applies during development, proxying `/api` → 4001. See [Build & Release](#build--release).

## Tech Stack

### Backend (Go)

- **Language**: Go 1.26
- **Router**: [chi v5](https://github.com/go-chi/chi) — lightweight, idiomatic HTTP router
- **API codegen**: [oapi-codegen v2](https://github.com/oapi-codegen/oapi-codegen) — generates types + chi server interface from OpenAPI 3 spec. Installed as a `go tool` (Go 1.24+ tool directive).
- **Config**: [koanf v2](https://github.com/knadh/koanf) — providers for confmap (defaults + file layer) and env. The settings file is JSON (`usr/settings.json`), loaded by an in-package `Store` instead of koanf's file/yaml providers; the same Store + a reflected schema power the `/config/*` REST surface.
- **Logging**: `log/slog` (stdlib) + [go-chi/httplog v3](https://github.com/go-chi/httplog) — HTTP access logs go through httplog middleware; all events aligned to ECS schema (`SchemaECS` + `ReplaceAttr` on the slog handler)
- **Decode hooks**: `github.com/go-viper/mapstructure/v2` (for `time.Duration` parsing)
- **Animated WebP**: [HugoSmits86/nativewebp](https://github.com/HugoSmits86/nativewebp) — pure Go, zero cgo, used for ugoira → animated WebP

### Frontend

- **Framework**: React 19 + react-router 7 (SPA, no SSR)
- **Build**: Vite 8 + TypeScript (strict, bundler resolution)
- **UI**: shadcn/ui (base-nova) on `@base-ui/react` primitives + Tailwind 4; Material You dynamic color via `@material/material-color-utilities`. Compose classNames via `cn(...)` from `@/lib/utils`, not backtick template literals.
- **i18n**: [Paraglide JS](https://inlang.com/m/gerre34r/library-inlang-paraglideJs) — compile-time, tree-shakeable message functions. Source + config + generated output consolidated under `src/i18n/` (barrel `index.ts`; `messages/` + `project.inlang/` + `generated/` (gitignored) + `react/{locale-provider,use-messages}`). Locales: `en` (baseLocale + fallback), `zh-CN`, `ja`. The rules below are load-bearing:
  - **Read via `const m = useMessages()`** inside render paths — it subscribes to `LocaleContext`, so a language switch rerenders without remounting. **Do not** `import { m }` from `@/i18n/generated/messages` inside components (those importers don't subscribe); module-level UI-text constants must move inside the component or a `useMemo`, or they freeze the locale at import time.
  - **Dynamic keys** use an **explicit static map** (`{ key: () => m.key() }`), never `m[dynamicKey]()` (breaks tree-shaking + types).
  - **Two-stage locale resolution.** (1) On mount, `LocaleProvider` resolves `auto` against `navigator.languages` *only* when Paraglide's localStorage cache is empty (first visit) — otherwise the cache wins, preserving the last explicit choice. Prefix matching is hand-rolled (`zh*`→`zh-CN`, `ja*`→`ja`, `en*`→`en`) because Paraglide's `extractLocaleFromNavigator` matches only exact configured locales. (2) After auth, `<LocaleSync>` (inside `AuthProvider`) fetches `GET /config` and applies the persisted `app.language`; a `PATCH` touching `app.language` calls `applyLanguage` synchronously, so a save switches the UI without reload. The unauthenticated login page relies on stage 1.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [txperl/PixivBiu](https://github.com/txperl/PixivBiu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
