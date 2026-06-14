---
trigger: always_on
description: middleman is a local-first maintainer console. Its original core is a dashboard for tracking pull and merge requests across a maintainer's fixed set of repositories on multiple platforms: it syncs PR/MR data into SQLite on a timer, serves a Svelte 5 SPA via an embedded Go HTTP server, and provides a focused workflow for triage, review, and merge from one place rather than each provider's notification UI. The product surface is expanding to include first-class modes for external Kata task daemons
---

# Claude Code Instructions

## Project Overview

middleman is a local-first maintainer console. Its original core is a dashboard for tracking pull and merge requests across a maintainer's fixed set of repositories on multiple platforms: it syncs PR/MR data into SQLite on a timer, serves a Svelte 5 SPA via an embedded Go HTTP server, and provides a focused workflow for triage, review, and merge from one place rather than each provider's notification UI. The product surface is expanding to include first-class modes for external Kata task daemons, markdown docs, and msgvault-backed message search without moving those domains into the provider registry.

## Architecture

```
CLI (middleman) → Config (TOML) → DB (SQLite)
                    ↓                ↓
               Sync Engine → Platform Registry → Provider Clients
                    ↓                ↓
               HTTP Server → REST API + Embedded SPA
                    ↓
          App Modes (PRs/issues, Kata, Docs, Messages)
```

- **Server**: Huma-based HTTP server on loopback (default 127.0.0.1:8091)
- **Storage**: SQLite with WAL mode (pure Go driver: modernc.org/sqlite)
- **Sync**: Periodic pull from each configured provider host (configurable, default 5m)
- **Kata**: External daemon client mode; daemon catalog comes from Kata's own `$KATA_HOME/config.toml` and runtime records, not middleman config
- **Docs**: Configured markdown folders with filesystem-safe browse/read/write/search/git publish behavior
- **Messages**: msgvault-backed message search, detail, thread, and safe HTML/image handling
- **Frontend**: Svelte 5 SPA embedded in the Go binary at build time
- **Config**: TOML at `~/.config/middleman/config.toml`; per-provider `MIDDLEMAN_<PROVIDER>_TOKEN` env vars (with optional repo-level `token_env` overrides)

## Provider Support

middleman supports GitHub, GitLab, Forgejo, and Gitea. The `gitealike` package is the shared Forgejo/Gitea adapter.

This paragraph is the single place CLAUDE.md enumerates supported providers. Do not duplicate the list elsewhere in this file: not in the architecture diagram, env-var lists, project structure, key files, or test guidance. Adding or removing a provider updates this paragraph only. Mentioning a specific provider in context (for example, GitHub-only optimizations in `internal/github/`) is fine when it describes real artifacts, not when it restates the supported set.

New features must work across every supported provider to the extent each provider's API allows. Concrete rules:

- Provider-specific capability differences go behind the capability model in `internal/platform`. Declare capabilities in `Capabilities()`, check them before mutations, and return typed `unsupported_capability` errors when a provider can't satisfy an operation. Do not silently fall back to GitHub-only behavior for other providers.
- Identity is `(platform, platform_host, owner, name)` everywhere; never owner/name/number alone. Repo-scoped routes use provider-aware paths like `/pulls/{provider}/{owner}/{name}/{number}`, with `/host/{platform_host}/...` for non-default or self-hosted instances.
- GitHub-only optimizations (GraphQL bulk fetch, ETag recovery, detailed diff behavior) stay in `internal/github/` and remain optional around the neutral persistence path.
- Frontend stores and components must thread the full provider ref (`provider`, `platformHost`, `owner`, `name`, `repoPath`) through the shared route helpers in `packages/ui/src/api/provider-routes.ts`. Do not hand-build `/api/v1` URLs or assume GitHub defaults inside components.

For package layout and the new-provider checklist, see `context/provider-architecture.md`. For identity, tokens, freshness, and route shape, see `context/platform-sync-invariants.md`. For GitHub-only sync behavior, see `context/github-sync-invariants.md`.

## Non-Provider Modes

Kata, Docs, and Messages are first-class middleman modes, but they are not platform providers and do not use provider-neutral repository identity. Do not force them through `internal/platform` or provider capability abstractions.

- Kata mode talks to external Kata daemons. Middleman reads the Kata daemon catalog from `$KATA_HOME/config.toml` (default `~/.kata/config.toml`) and resolves `local = true` daemon entries from Kata runtime records. Middleman config must not become the source of truth for Kata daemon definitions.
- Docs mode operates on explicitly configured local markdown folders. Treat folder reads, writes, deletes, browse, and git publish as local filesystem surfaces requiring explicit path safety, CSRF, and loopback-access decisions.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kenn-io/middleman](https://github.com/kenn-io/middleman) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-14 -->
