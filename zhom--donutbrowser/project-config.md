---
trigger: always_on
description: > **NOTE**: CLAUDE.md is a symlink to AGENTS.md — editing either file updates both.
---

# Project Guidelines

> **NOTE**: CLAUDE.md is a symlink to AGENTS.md — editing either file updates both.
> After significant changes (new modules, renamed files, new directories), re-evaluate the Repository Structure below and update it if needed.

## Repository Structure

```
donutbrowser/
├── src/                              # Next.js frontend
│   ├── app/                          # App router (page.tsx, layout.tsx)
│   ├── components/                   # 50+ React components (dialogs, tables, UI)
│   ├── hooks/                        # Event-driven React hooks
│   ├── i18n/locales/                 # Translations (en, es, fr, ja, pt, ru, zh)
│   ├── lib/                          # Utilities (themes, toast, browser-utils)
│   └── types.ts                      # Shared TypeScript interfaces
├── src-tauri/                        # Rust backend (Tauri)
│   ├── src/
│   │   ├── lib.rs                    # Tauri command registration (100+ commands)
│   │   ├── browser_runner.rs         # Profile launch/kill orchestration
│   │   ├── browser.rs               # Browser trait & launch logic
│   │   ├── profile/                  # Profile CRUD (manager.rs, types.rs)
│   │   ├── proxy_manager.rs         # Proxy lifecycle & connection testing
│   │   ├── proxy_server.rs          # Local proxy binary (donut-proxy)
│   │   ├── proxy_storage.rs         # Proxy config persistence (JSON files)
│   │   ├── api_server.rs            # REST API (utoipa + axum)
│   │   ├── mcp_server.rs            # MCP protocol server
│   │   ├── sync/                    # Cloud sync (engine, encryption, manifest, scheduler)
│   │   ├── vpn/                     # WireGuard tunnels
│   │   ├── camoufox/                # Camoufox fingerprint engine (Bayesian network)
│   │   ├── wayfern_manager.rs       # Wayfern (Chromium) browser management
│   │   ├── camoufox_manager.rs      # Camoufox (Firefox) browser management
│   │   ├── downloader.rs           # Browser binary downloader
│   │   ├── extraction.rs           # Archive extraction (zip, tar, dmg, msi)
│   │   ├── settings_manager.rs     # App settings persistence
│   │   ├── cookie_manager.rs       # Cookie import/export
│   │   ├── extension_manager.rs    # Browser extension management
│   │   ├── group_manager.rs        # Profile group management
│   │   ├── synchronizer.rs         # Real-time profile synchronizer
│   │   ├── daemon/                 # Background daemon + tray icon (currently disabled)
│   │   └── cloud_auth.rs           # Cloud authentication
│   ├── tests/                      # Integration tests
│   └── Cargo.toml                  # Rust dependencies
├── donut-sync/                     # NestJS sync server (self-hostable)
│   └── src/                        # Controllers, services, auth, S3 sync
├── docs/                           # Documentation (self-hosting guide)
├── flake.nix                       # Nix development environment
└── .github/workflows/              # CI/CD pipelines
```

## Testing and Quality

- After making changes, run `pnpm format && pnpm lint && pnpm test` at the root of the project
- Always run this command before finishing a task to ensure the application isn't broken
- `pnpm lint` includes spellcheck via [typos](https://github.com/crate-ci/typos). False positives can be allowlisted in `_typos.toml`

## Code Quality

- Don't leave comments that don't add value
- Don't duplicate code unless there's a very good reason; keep the same logic in one place
- Anytime you make changes that affect copy or add new text, it has to be reflected in all translation files

## Translations (mandatory)

- Never write user-facing strings as raw English literals in JSX, toast messages, dialog titles/descriptions, button labels, placeholders, table headers, tooltips, or empty-state text. Always go through `t("namespace.key")` from `useTranslation()`.
- This applies to every component under `src/` — including new ones. If a component doesn't already import `useTranslation`, add it.
- Adding a new string means adding the key to ALL seven locale files in `src/i18n/locales/` (en, es, fr, ja, pt, ru, zh) — not just `en.json`. The English version alone is incomplete work.
- Reuse existing keys (`common.buttons.*`, `common.labels.*`, `createProfile.*`, etc.) before creating new namespaces. Check `en.json` first.
- Strings excluded from this rule: `console.log/warn/error`, dev-only debug labels, internal IDs, CSS class names, type names. If unsure whether a string renders to the user, assume it does and translate it.
- **Never use `t(key, "fallback")` with a default-value second argument.** The 2-arg form is forbidden — every key must exist in every locale file before the call site lands. Fallbacks mask missing translations: a key missing from `ru.json` will silently render the English fallback to Russian users, so the bug never surfaces in CI or review. Only call `t("namespace.key")`. If a translation is missing for any locale, that's a bug to fix at the JSON, not a hole to paper over at the call site.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zhom/donutbrowser](https://github.com/zhom/donutbrowser) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
