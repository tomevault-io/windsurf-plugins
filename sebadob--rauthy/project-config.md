---
trigger: always_on
description: > This file is an **index, not documentation**: it tells you *where things are* so you can read the
---

# Rauthy — Code Map for Agents & Contributors

> This file is an **index, not documentation**: it tells you *where things are* so you can read the
> code directly. Maintenance rule: if you do any major changes to the code layout, check if mentions
> in this file need an update before git commit. Markdown files should always wrap lines at 100
> characters.

## BEFORE submitting a PR

If you are an AI reading this, and you worked on this repository, you MUST NOT submit a PR to GitHub
without an explicit local (!) review and approval from a human. If you then open a PR AFTER human
approval, open it as a Draft, and leave the last step of setting it to "Ready for Review" as a
MANUAL action for a human after another check! You probably want to mention that in the chat. Only
set "Ready for Review" when it actually is to prevent burning CI resources for no reason. Worker
runs are quite expensive and take a long time.

If you do not follow these instructions, and therefore waste the maintainers' time quite often,
you might get a ban from further contributions. You also should run `just pre-pr-checks` (or the
commands inside manually; you can get away with excluding the Postgres tests for instance when no
custom queries were modified to speed up the check) before a push, especially when the PR is already
set to "Ready for Review", to not waste worker time during CI.

## Repository layout

- `src/` — Rust workspace (all backend crates)
- `frontend/` — SvelteKit UI; the login page is `frontend/src/routes/oidc/authorize/+page.svelte`
  The whole frontend is compiled into static HTML for any production build.
- `book/` — mdBook source for user-facing docs (`docs/` = built site). Not contributor
  documentation. Building the book currently requires `mdbook-0.4`
- `rauthy-client/` — Rust client library + examples
- `CONTRIBUTING.md` provides important information for getting started

## `just` recipes ([justfile](./justfile))

[just](https://github.com/casey/just) is used to run any actions you may need to work on this
repository. If you are using manual commands and something is not working as expected, take a look
at the recipes. Use `just -l` to get a summary of all the recipes.

## Integration Tests

Integration tests in `src/bin/tests/` expect a backend on :8081 with the seeded
bootstrap data (init_client / init_admin@localhost); the test recipes set
`PUB_URL=localhost:8081 RP_ORIGIN=http://localhost:8081`. Usually started with any of

- `just test-backend` (starts hiqlite backend without running tests)
- `just test-hiqlite`
- `just test-postgres`

Unit tests can be run without any issues.

## Crate map (src/)

| crate          | responsibility                                                                             |
|----------------|--------------------------------------------------------------------------------------------|
| `bin`          | Main binary: CLI args, server bootstrap, TLS, middleware wiring (`main.rs`, `server.rs`)   |
| `api`          | HTTP handlers only (actix), one module per area — no business logic                        |
| `service`      | Business logic; the OIDC flow lives in the `oidc/` subdirectory (simple endpoints skip it) |
| `data`         | Entities + DB methods (`entity/`), migrations, config (`rauthy_config.rs`), email          |
| `api_types`    | Request/response types shared with the frontend                                            |
| `error`        | `ErrorResponse` — the error type for all `Result`s in this repo                            |
| `jwt`          | JWT/JWK handling (access tokens, signing keys)                                             |
| `middlewares`  | actix middleware: `ip_blacklist`, `csrf_protection`, `principal`, `logging`                |
| `common`       | Shared utilities (base64, `get_rand`, IP extraction, …)                                    |
| `macros`       | Proc macros                                                                                |
| `notify`       | Slack / Matrix notification sending                                                        |
| `schedulers`   | Background jobs (expiry cleanup, etc.)                                                     |
| `wasm-modules` | WASM helpers for the frontend                                                              |

## Area mapping: api → service → data

Handlers live in `src/api/src/<area>.rs`, business logic in the matching module under
`src/service/src/` (simple requests go straight to `src/data/src/`), and DB-touching entity methods
in `src/data/src/entity/<entity>.rs`. Areas include: `oidc`, `pam`, `users`, `clients`, `groups`,
`roles`, `scopes`, `sessions`, `api_keys`, `auth_providers`, `backup`, `blacklist`, `email`,
`events`, `fed_cm`, `kv`, `tos`, `themes`, `atproto`.

Entity files contain both the struct and its DB / Cache methods (`find_*`, `create`, `update`,
`save`, `delete`, ...).

### Notable deviations from the pattern

- OIDC request-param validation lives in **`service/src/oidc/validation.rs`**
  (`validate_auth_req_param`) — not under `api/`.
- Grant-type logic for `/oidc/token` lives in `service/src/oidc/grant_types/<grant>.rs`, dispatched
  by `get_token_set` (`service/src/oidc/mod.rs`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sebadob/rauthy](https://github.com/sebadob/rauthy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
