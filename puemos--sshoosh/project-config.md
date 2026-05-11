---
trigger: always_on
description: These instructions apply to the whole repository. Keep them concise and durable; update this file when build, test, release, architecture, or security expectations change.
---

# AGENTS.md

## Scope

These instructions apply to the whole repository. Keep them concise and durable; update this file when build, test, release, architecture, or security expectations change.

## Project Intent

`sshoosh` is a self-hosted SSH/TUI workspace chat for small teams and operators. Users connect with SSH keys and work inside a dense terminal UI for channels, threads, DMs, notifications, mentions, reactions, unread state, search, exports, and administration.

The runtime should stay small and operator-friendly: one Rust binary, one SQLite/libSQL database, one SSH host key, and optional app-level encryption.

## Repo Map

- `src/cli/`: clap CLI definitions, command dispatch, local dev helpers, and sensitive file writing.
- `src/ssh/`: SSH server/session transport, authentication handoff, render loop integration, and SSH-facing action formatting.
- `src/app/`: TUI application state, input handling, slash commands, navigation, rendering, theme, and terminal interaction behavior.
- `src/client/`: `ClientSession` facade used by the TUI/SSH layer to call service behavior.
- `src/service/`: durable product behavior, permissions, loaders, write operations, notifications/reactions, audit/export, runtime, and `ServerState`.
- `src/domain/`: shared domain models.
- `src/db.rs`: database connection, migrations, query wrapper, encryption, backups, doctor checks, master lease, and local file permissions.
- `migrations/`: SQL migrations embedded by `src/db.rs`.
- `tests/integration.rs`: end-to-end service, persistence, SSH, auth, security, and CLI behavior coverage.
- `docs/`: Astro docs site. Use `pnpm`; do not edit `docs/node_modules`, `docs/dist`, or `docs/.astro`.
- `packaging/`, `Dockerfile`, `install.sh`, `.github/workflows/`: release, install, systemd, Docker, Pages, and CI automation.

## Working Rules

- Prefer existing module patterns over new abstractions. Keep edits narrowly scoped to the behavior being changed.
- Use `rg`/`rg --files` for search. Exclude generated or ignored paths such as `target/`, `docs/node_modules/`, `docs/dist/`, and `docs/.astro/`.
- Do not commit secrets, local databases, SSH host keys, release artifacts, or generated dependency directories.
- Protect `SSHOOSH_DB`, `SSHOOSH_DATABASE_AUTH_TOKEN`, `SSHOOSH_ENCRYPTION_KEY`, `SSHOOSH_SERVER_KEY`, exports, backups, and provider snapshots as sensitive data.
- Do not add a license file without an explicit owner decision.
- Keep user-facing commands, docs, CLI help, and TUI slash command references synchronized when behavior changes.

## Architecture Boundaries

- Durable product behavior belongs in `ServerState` and the service modules. Avoid putting persistence, permissions, audit, notification, or membership rules in render, input, SSH, or CLI code.
- `src/app` owns TUI state, rendering, input, slash command parsing/autocomplete, navigation, hit maps, and terminal affordances.
- `src/ssh` should remain transport/session glue: SSH auth, channels, render loop, terminal IO, and mapping TUI actions to client calls.
- `src/cli` should parse/administer commands and delegate behavior to service/database APIs rather than duplicating business logic.
- `ClientSession` is the app-facing facade over service behavior. TUI actions should route through it instead of reaching directly into `ServerState`.
- Database writes should go through the query wrapper in `src/db.rs` so encryption, master/standby fencing, and local file protections remain effective.
- Schema changes must update SQL migrations, the embedded migration list in `src/db.rs`, and tests that prove upgrades and persistence still work.

## Product And Security Invariants

- Unknown SSH keys must redeem a bootstrap or invite token via the SSH keyboard-interactive prompt before any account or key row is written. Tokens must never be parsed out of the SSH user field, sent over `auth_password`, or stored in plaintext in logs.
- The first activated account is bootstrapped with a one-time bootstrap token and becomes owner.
- `#general` is mandatory for activated users and must not become private, leaveable, archived, or deleted.
- Public channels are discoverable, but content visibility and searchability depend on explicit membership.
- Private channel content, mentions, notifications, search results, and source links must stay visible only to members.
- Owner/admin boundaries matter. Do not allow admins to mint owners/admins or remove the last active owner.
- Admin and security-sensitive actions should be audited.
- Unread counts, notification read state, muted threads/DMs, saved messages, and source navigation should stay consistent after edits/deletes.
- `SSHOOSH_ENCRYPTION_KEY` encrypts source content fields, but `search_index` remains plaintext by design so full-text search works. Treat search data as sensitive at rest.
- Local SQLite files, backups, generated bootstrap tokens, and server keys should use owner-only permissions where applicable.

## TUI Design Principles

- Preserve the dense, terminal-native operator UI. Avoid marketing-style copy, sparse layouts, or decorative UI.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [puemos/sshoosh](https://github.com/puemos/sshoosh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
