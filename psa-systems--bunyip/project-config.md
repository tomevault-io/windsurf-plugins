---
trigger: always_on
description: Guidance for AI agents working in this repository.
---

# CLAUDE.md

Guidance for AI agents working in this repository.

## What this is

bunyip is the PSA Systems SaaS platform: a Cargo **workspace** with two server
apps plus the domain it owns.

```
bunyip/
├── bunyip-web/             bunyip-web - Axum SSR frontend (Maud + htmx). The browser-facing BFF.
├── bunyip-api/             bunyip-api - actix-web backend binary (wiring + main.rs + migrations).
└── crates/
    ├── bunyip-domain         models, repositories, business services, app Config, email templates.
    ├── bunyip-oci          OCI registry vertical.        (depends on bunyip-domain)
    └── bunyip-oidc         OIDC / OAuth 2.1 vertical.    (depends on bunyip-domain)
```

bunyip **owns all domain-specific code**. The generic, domain-free kernel
(errors, responses, validation, request_id/security_headers middleware, and the
generic jwt/encryption/password services) is `dunite-core`, consumed as a
**git dependency** from the Forgejo repo
`https://dev.a8n.run/psa-systems/dunite`. The dunite repo is anonymously
readable, so builds need no token (an optional `DUNITE_GIT_TOKEN` / buildkit
secret `dunite_token` is honoured for mirrors that require auth). Nothing in
dunite is bunyip-specific; nothing domain-specific lives in dunite.

All four dunite crates are pinned by `rev` (not `branch = "main"`), so moving
bunyip onto a newer dunite is an explicit one-line manifest diff in a bunyip PR
rather than a silent lockfile change (BUNYIP-426 F6). Bumping means editing the
`rev` in `crates/bunyip-{domain,oci,oidc}/Cargo.toml` and re-running
`cargo update --package dunite-core --package dunite-download --package dunite-oci --package dunite-oidc`.

Dependency direction (strictly downward): `bunyip-api -> bunyip-oci/oidc -> bunyip-domain -> dunite-core (git)`. `bunyip-web` is a standalone binary (talks to bunyip-api over /v1).

Ports: bunyip-api listens on `APP_PORT=4401`; bunyip-web on `4400`. bunyip-api
is also bunyip's OIDC issuer (it serves `/.well-known/*` + `/oauth2/*`).

## Build / dev

`just` drives everything (see `justfile`):

- `just dev` / `just dev-detach` - full local stack (postgres + api + web) via `compose.dev.yml`.
- `just dev-sso` - Traefik-routed stack on `*.a8n.run` (layers `compose.dev-sso.yml` on top). Cross-repo (bunyip + mokosh-server + mokosh-apps), Nebula topology, OIDC client registration, and every spin-up obstacle are documented in `docs/dev-sso-three-repo-runbook.md` - read it before touching dev-sso infra or onboarding a dev box.
- `just check` - fmt + clippy + build + docker builder stage. `just test`, `just typecheck`, `just lint`, `just fmt`.
- `just build-docker` - both production images (`build-docker-export` extracts the api static binary). `just migrate` / `migrate-revert`.
- `just create-release <major|minor|hotfix>` - bump `[workspace.package].version`, push the branch, open the release PR. The member-scoped `cargo update` that syncs `Cargo.lock` runs inside the pinned rust-builder image (dev boxes have no local cargo; online, to resolve the dunite-core git dep), NOT on the host; it is deliberately NOT `--workspace`, which would also roll the dunite git dep forward. Every git/fj step stays on the host, so the recipe needs docker (it fails fast if docker is missing).

Production runs the published images via `compose.yml` (api + web + postgres,
images under `dev.a8n.run/psa-systems-private/{bunyip-api,bunyip-web}`).

## Toolchain / checks on toolchain-less dev boxes

The canonical Rust toolchain is pinned in `rust-toolchain.toml` (currently
1.94.1, matching the `ghcr.io/niceguyit/rust-builder-*:v1.0.0-rust1.94-*`
images and CI). Bumping it means fixing any newly-promoted clippy/rustfmt
lints in the same PR so `just check` stays green everywhere.

Dev boxes have **no local Rust toolchain**, so run `just check-container`. It
wraps fmt + clippy + workspace lib tests in the pinned rust-builder image with
named cache volumes for the cargo registry and target dir (so repeated runs
stay incremental).

The image's rustup honours `rust-toolchain.toml`, so the pin (not the image
default) decides the compiler version. CI (`.forgejo/workflows/check.yml`)
runs the same fmt/clippy/build/test sequence on every PR and push to main.

## Critical conventions

- **sqlx**: only `bunyip-oidc` uses compile-time `sqlx::query!` macros. They resolve against the workspace-root `.sqlx/` offline cache; build with `SQLX_OFFLINE=true` (the justfile/Dockerfiles set it). After changing those queries, regenerate `.sqlx/` and commit it.
- **Migrations** live in `bunyip-api/migrations/` and run on api startup. **Committed migrations are immutable.** sqlx checksums every applied migration in `_sqlx_migrations` and a deployed database refuses to boot once a migration's on-disk content disagrees with the recorded checksum (`migration <version> was previously applied but has been modified`). Never edit, rename, or delete a migration already on `main`: fix forward with a NEW migration file. CI enforces this via `scripts/check-migration-immutability.nu` (BUNYIP-293); `scripts/reconcile-sqlx-checksums.md` covers recovering a DB that was broken by an in-place edit.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [psa-systems/bunyip](https://github.com/psa-systems/bunyip) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
