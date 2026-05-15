---
trigger: always_on
description: Brewlog is a self-hosted coffee logging platform built in Rust: HTTP server (Axum + Askama + Datastar), REST API, CLI client, SQLite database.
---

# Claude Code Guidelines for Brewlog

## Project Overview

Brewlog is a self-hosted coffee logging platform built in Rust: HTTP server (Axum + Askama + Datastar), REST API, CLI client, SQLite database.

## Build & Test Commands

```bash
prek run -av                      # All lints, tests, formatters
cargo build                       # Build
cargo test                        # Tests
cargo clippy --allow-dirty --fix  # Lint + auto-fix
nix fmt                           # Format
sqlx migrate add <name>           # New migration → migrations/NNNN_<name>.sql
```

## Workflow Requirements

**Before finishing any task**, always:

1. Run `prek run -av`
1. Consider if the test coverage needs updating
1. Update `README.md` if the change adds/removes/renames CLI commands, env vars, or user-facing features
1. Update `scripts/bootstrap-db.sh` if the change affects CLI commands, flags, or entity fields used by it
1. Provide a **draft commit message** using Conventional Commits format

## Architecture

Clean Architecture / DDD with four layers:

```
src/
├── domain/              # Pure business logic, no external deps
│   ├── errors.rs        # RepositoryError enum
│   ├── ids.rs           # Typed ID wrappers (RoasterId, BagId, BrewId, etc.)
│   ├── listing.rs       # Pagination & sorting (SortKey, ListRequest, Page, PageSize)
│   ├── repositories.rs  # Repository traits
│   ├── countries.rs     # Country name → ISO code, flag emoji
│   ├── formatting.rs    # format_relative_time(), format_weight()
│   ├── coffee/          # roasters, roasts, bags, brews, cups, gear, cafes
│   ├── auth/            # users, sessions, tokens, passkeys, registration_tokens
│   └── analytics/       # timeline, stats, country_stats, ai_usage
├── infrastructure/      # DB, HTTP clients, third-party APIs
│   ├── repositories/    # SQL impls of repository traits (coffee/, auth/, analytics/)
│   ├── client/          # HTTP client for CLI
│   ├── ai.rs            # OpenRouter LLM integration
│   ├── foursquare.rs    # Foursquare Places API
│   ├── backup.rs        # Database backup/restore
│   └── database.rs      # Database pool + SQLite pragmas
├── application/         # HTTP server, routes, middleware, services
│   ├── routes/          # Axum handlers (api/ for REST, app/ for web UI)
│   ├── services/        # Entity services (create + timeline event)
│   └── errors.rs        # HTTP error mapping
└── presentation/        # User interfaces
    ├── cli/             # CLI commands
    └── web/             # View models for templates
```

**Dependency flow**: `presentation → application → domain ← infrastructure`

## Gotchas

These are non-obvious footguns that will cause bugs if missed.

**1. `datastar-fetch` event bubbles through the DOM.** Every `data-on:datastar-fetch` handler must guard with its own in-progress signal, or it fires for events from _any_ `@post`/`@get` in the DOM tree:

```html
<form
  data-on:submit="$_extracting = true; @post(...)"
  data-on:datastar-fetch="if (!$_extracting) return;
    if (evt.detail.type === 'finished') { $_extracting = false }
    else if (evt.detail.type === 'error') { $_extracting = false; $_extractError = 'Failed.' }"
></form>
```

**2. No `data-model` in Datastar v1** — silently ignored. Use `data-bind:_signal-name`.

**3. Signal patching requires JSON, not HTML.** Use `render_signals_json()`, not `data-signals` in DOM fragments.

**4. List partial must be OUTSIDE the form section** — sibling of the form `<section>`, not nested inside it.

**5. Table wrapper must be `<section>`, not `<div>`** — `<section class="rounded-lg border bg-surface">`.

**6. Infinite scroll sentinel needs `md:hidden`** — `<div class="infinite-scroll-sentinel h-4 md:hidden">`.

**7. Use token-based text classes, never `text-stone-*`.** Use `text-text`, `text-text-secondary`, `text-text-muted`.

**8. Static assets need explicit routes and cache headers.** Embedded via `include_str!()`/`include_bytes!()` with explicit routes in `application/routes/app/mod.rs`. All under `/static/`. Every handler must return `cache-control: public, max-age=604800`.

**9. CSP must be updated when adding external resources.** Set in `application/routes/mod.rs`. Datastar requires `'unsafe-inline'` and `'unsafe-eval'` in `script-src`.

**10. Cookie `Secure` flag is on by default.** Set `BREWLOG_INSECURE_COOKIES=true` for local HTTP dev.

**11. URL fields must validate scheme server-side.** Reject non-`http(s)` schemes to prevent XSS. See `is_valid_url_scheme()` in `domain/coffee/roasters.rs`.

**12. Datastar create handlers must check referer for fragment targets.** If a `@post` can fire from pages lacking the target element, check `Referer` and return a reload-script. See `create_brew` in `application/routes/api/coffee/brews.rs`.

## Backend Patterns

### Repository Pattern

Repositories defined as traits in `domain/repositories.rs`, SQL impls in `infrastructure/repositories/`. Each uses a private `Record` struct with `to_domain()`. Use typed ID wrappers from `domain/ids.rs` — never raw `i64`.

### Service Layer


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jnsgruk/brewlog](https://github.com/jnsgruk/brewlog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
