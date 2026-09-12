---
trigger: always_on
description: Ultimo is a modern Rust web framework: REST + JSON-RPC in one app, automatic
---

# Ultimo — Developer Reference (for Codex / coding agents)

Ultimo is a modern Rust web framework: REST + JSON-RPC in one app, automatic
**TypeScript client generation** from Rust API definitions, and RFC 6455
WebSocket support with built-in pub/sub. Built on **Hyper 1.0 + Tokio**.

- Homepage: https://ultimo.dev · Docs: https://docs.ultimo.dev · Roadmap: https://docs.ultimo.dev/roadmap
- Repo: https://github.com/ultimo-rs/ultimo · Issues: https://github.com/ultimo-rs/ultimo/issues · Board: https://github.com/orgs/ultimo-rs/projects/1
- Current version: **0.9.1** · Edition 2021 · MSRV **1.86.0** · License MIT

## ⚠️ THESE ARE PUBLISHED CRATES — read before changing any public code

Ultimo ships two crates to crates.io that real users depend on: **`ultimo`**
(the library) and **`ultimo-cli`** (the binary; depends on `ultimo`). Every
change is a potential breaking change. **The public API contract** — changing
any of it requires a deliberate version bump:

- Any `pub` item (types, fns, traits, enum variants, fields) and the `prelude`
- **Cargo feature names** (`websocket`, `testing`, `session`, `csrf`, `database`, `sqlx-*`, `diesel-*`, `test-helpers`) — renaming/removing breaks `features = [...]` downstream
- **MSRV** (`rust-version`) — raising it is breaking (we're at 1.86.0)
- **Direct dependency floors** — raising one (e.g. `bytes`, `diesel`) constrains downstream resolution; a dep type re-exported in our API (e.g. `bytes::Bytes` in WebSocket messages) makes that dep's version part of _our_ API
- CLI subcommands / flags of `ultimo-cli`

### On EVERY change

1. **Decide SemVer impact.** Pre-1.0 (0.x): breaking → bump **minor** (0.3.x → 0.4.0); additive/fix → bump **patch**. CI's `semver-checks` enforces this against the published crate — respect its verdict.
2. **Don't hand-edit `CHANGELOG.md`** — release-plz regenerates it from your conventional commits. Write a good commit message; that _is_ the changelog entry.
3. **Keep `README.md` accurate** — it's the crates.io landing page.
4. **Don't break docs.rs** — public items need doc comments; doctests must compile (`cargo test --doc`).

### On a RELEASE (automated by release-plz — rarely done by hand)

release-plz opens a release PR that bumps `version` in root `Cargo.toml`
(`[workspace.package]`, shared by both crates) and writes the per-crate
`ultimo/CHANGELOG.md` / `ultimo-cli/CHANGELOG.md` from commits. Merging it
publishes **`ultimo` then `ultimo-cli`** to crates.io and tags a release. A
published vuln/serious bug → **`cargo yank`** it and ship a fixed patch.

**Version must be in sync EVERYWHERE — one source of truth (`Cargo.toml`).** The
`version-sync` CI gate (`scripts/check-versions.sh`) fails a PR if any surface
drifts: `website/package.json`, `docs-site/package.json`, the website hero badge,
the `ultimo = "x.y"` install snippets across README + docs, `CLAUDE.md`/`AGENTS.md`,
the blog comparison post, and the root `CHANGELOG.md` + `docs-site/.../changelog.mdx`
(plus the git tag + GitHub release, done by release-plz). release-plz only bumps
`Cargo.toml` + the per-crate changelogs, so the release workflow auto-runs
`scripts/sync-versions.sh` on the release PR to propagate the rest (it mirrors the
new changelog section from whichever crate — library or CLI — was released). If
`version-sync` ever fails on a release PR, run `bash scripts/sync-versions.sh` and
commit — don't hand-edit the surfaces one by one.

## Workspace layout

Cargo workspace (`resolver = "2"`). Members:

| Crate / path     | Role                                                                                                                                                                   |
| ---------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ultimo/`        | **The framework.** Core library — everything below is a module here.                                                                                                   |
| `ultimo-cli/`    | CLI binary (clap). Subcommands: `generate` (TS client), `new` (scaffold), `dev` (hot-reload server), `build`.                                                          |
| `coverage-tool/` | Custom coverage runner (`ultimo-coverage`), invoked by `make coverage`.                                                                                                |
| `examples/*`     | Runnable example apps. `Cargo.toml` `members` is the source of truth — some dirs (`react-app`, `benchmark`, `database-with-openapi`) exist on disk but aren't members. |

### `ultimo/src` modules (`lib.rs` is the map)

- `app.rs` (`Ultimo` builder), `context.rs` (`Context`), `router.rs`, `handler.rs`, `middleware.rs`
- `rpc.rs` — JSON-RPC registry + **TypeScript client codegen** (`RpcRegistry`)
- `response.rs`, `error.rs` (`UltimoError`/`Result`), `validation.rs` (`validate`), `openapi.rs` (+ `openapi/docs.rs`)
- `cookie.rs` (core), `session/` (`session`), `csrf.rs` (`csrf`), `testing/` (`testing`)
- `database/` = `sqlx.rs` + `diesel.rs` (behind `database`); `websocket/` (behind `websocket`)

Public API surface (keep stable): `Ultimo`, `Context`, `Result`, `UltimoError`,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ultimo-rs/ultimo](https://github.com/ultimo-rs/ultimo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-12 -->
