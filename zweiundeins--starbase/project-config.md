---
trigger: always_on
description: Community gallery for Datastar Rocket web components. Go 1.27, templ, SQLite (modernc, pure Go), Datastar v1.0.4 + Rocket (vendored in `static/vendor/datastar-rocket.js`).
---

# Starbase: notes for Claude

Community gallery for Datastar Rocket web components. Go 1.27, templ, SQLite (modernc, pure Go), Datastar v1.0.4 + Rocket (vendored in `static/vendor/datastar-rocket.js`).

## Commands
- `go tool task live`: dev server (air, `-tags dev`, live reload via `/dev/reload`)
- `go tool task test`: `go vet` + `go test ./...`. Run `go tool templ generate` after editing `.templ` (the `_templ.go` files are committed).
- `go tool task new -- <slug> --category <cat>`: scaffold a component
- `go tool task manifests`: regenerate all `manifest.json` in headless Chrome (`cmd/manifests`, runs the app in-process); `--check` in CI. Chrome keeps its sandbox (submitted code runs there); on machines without one, set `STARBASE_CHROME_NO_SANDBOX=1` (never in CI).

## Performance and SEO
- Responses to GET/HEAD are compressed (brotli/zstd/gzip, `web/compress.go`); render streams (POST) compress themselves and bypass it. Set `Vary` with `Add`, never `Set`.
- Hashed static files are `immutable`. `robots.txt`, `sitemap.xml` (pages + active components), `/og.png`, `/apple-touch-icon.png` and `/favicon.ico` (PNG, rendered from the pixel art: `pixelart/png.go`) are in `web/seo.go`. Every page has a canonical URL, Open Graph/Twitter tags and JSON-LD (`WebSite` with a search action; component pages add `SoftwareSourceCode` via `view.Schema`).

## Deploy
- `deploy/`: host files (systemd unit, env, Caddy snippet), `starbase-deploy` (root; verifies, restarts, checks `/healthz`, rolls back) and `setup-host.sh` (the restricted `starbase-deploy` user with a forced-command key). `.github/workflows/deploy.yml` runs after CI on `main`, gated by the repository variable `DEPLOY_HOST`.

## Architecture rules (CQRS)
- State changes are **commands** (`internal/commands`): structs with `Apply(ctx, *sql.Tx)`, optional `Validate()`, and `Scope()` (session id) when only that session's views change. Handlers call `bus.Send` and return 204. Commands never render HTML.
- Only the `cqrs.Bus` writes to the database (single writer, `db.W`). Queries use `db.R` through `queries.Queries.View` (one read tx per render).
- Pages are `pageFunc`s in `internal/web` registered with `s.page(mux, pattern, fn)`, which gives the GET document and the POST render stream. A page must be a pure function of `renderCtx`.
- Per-tab UI state goes in `tab_state` via commands, never in handler memory or client-only signals.
- Exception: the site theme is a browser preference, kept by `sb-theme-switch` in the `sb-theme` cookie. The layout renders it (`Shell.Theme`, `siteTheme`) on `<html data-sb-theme>`, so there is no flash. "auto" means no attribute: `:root` is deep-space, and `/theme/auto.css` (generated from the daylight block in `css/themes/showcase.css`) applies daylight on light systems.

## Frontend rules
- 8-bit details are opt-out: pixel-corner clip-paths scale with `--sb-notch` (1 or 0; at 0 use a border radius), frames with `--sb-frame-step`, display text uses `--sb-font-display`. `[data-sb-style="smooth"]` (theme.css) sets all three; the Themes page switch is `tab_state.PreviewSmooth`.
- CSS: `@layer reset, tokens, theme, base, layout, components, utilities`. Pages load one minified bundle (`/bundle/site.css`, built at startup from `siteStylesheets` in `web/assets.go`, font URLs rewritten to hashed names); a new stylesheet must be added there. The playground runner gets `/bundle/runner.css`.
- Syntax colours are semantic tokens (`--sb-code-keyword|function|tag|string|number`), with light values in the daylight block; don't use palette primitives (`--sb-violet-3`…) for anything that must work on light themes. Use semantic `--sb-*` tokens (theme.css) in components, not primitives. Prefer container queries over media queries.
- **Wire components declaratively:** events use `data-on:*` with local `action()`s (`@name()`), state is `$$` signals with `data-bind`/`data-show`/`data-class`/`data-attr`/`data-text`/`data-effect`, lists use `<template data-for>`, and elements come from `data-ref:x` → `onFirstRender({ refs })` (refs are not `$$` signals). `data-bind:x` in rendered markup binds the local `$$x`. Plain `addEventListener` is only for things without an attribute form (`matchMedia`, Intersection/ResizeObserver).
- Rocket components (`components/<slug>/`): shadow DOM (default mode), `adoptStyles(host, css)`, `--_x: var(--sb-x, fallback)` locals, interaction state in `$$` signals (never reflected to attributes, because server morphs reset attributes), `emit()` for events, `.docs()` on every prop.
- **Known upstream issue:** Datastar's morph is not re-entrant, and Rocket renders synchronously in `connectedCallback`. Never put `id`s on repeated or reordered elements that contain Rocket components (e.g. gallery cards). The morph would park and move them and crash (`Maximum call stack` / `moveBefore` HierarchyRequestError). Minimal repro and a proven fix are in `docs/repro/rocket-morph-reentrancy/` (upstream: starfederation/datastar#1209).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zweiundeins/starbase](https://github.com/zweiundeins/starbase) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
