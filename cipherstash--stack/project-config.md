---
trigger: always_on
description: > **Repository ownership:** EQL is maintained in the `cipherstash/stack`
---

# CLAUDE.md

> **Repository ownership:** EQL is maintained in the `cipherstash/stack`
> monorepo. Track all EQL work in `cipherstash/stack` GitHub Issues. The former
> `cipherstash/encrypt-query-language` repository and its issues are historical
> provenance only; do not create or update active work there.

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

This project uses `mise` for task management. Common commands:

- `mise run build` (alias: `mise r b`) - Build SQL into single release file
- `mise run test` (alias: `mise r test`) - Build, reset and run tests
- `mise run postgres:up` - Start PostgreSQL container
- `mise run postgres:down` - Stop PostgreSQL containers
- `mise run reset` - Reset database state
- `mise run clean` (alias: `mise r k`) - Clean release files

### Documentation
- `mise run docs:generate` - Generate API documentation (requires doxygen)
  - Outputs XML (primary) and HTML (preview) formats
  - XML suitable for downstream processing/website integration
  - Output is written to `docs/api/` (generated, gitignored)
- `mise run docs:generate:markdown` - Convert XML to Markdown API reference
  - Generates single-file API reference: `docs/api/markdown/API.md` (all documented functions with parameters, return values, and source links)
- `mise run docs:validate` - Validate documentation coverage and tags
- `mise run docs:package` - Package XML docs for distribution (~230KB archive)

### Testing
- Run all tests: `mise run test`
- Run SQLx tests directly: `mise run test:sqlx`
- Run SQLx tests in watch mode: `mise run test:sqlx:watch`
- Tests are located in `tests/sqlx/` using Rust and SQLx framework
- Property-based tests for the `eql_v3` encrypted scalar domains live in three suites — **catalog** (pure-Rust catalog invariants, no DB), **fixture** (oracle over committed ciphertext), and **e2e** (oracle over fresh end-to-end encryption, gated behind the `proptest-e2e` cargo feature). The structure, the shared all-pairs oracle engine, and the conventions/footguns (e.g. why they must not live under `scalars::`) are documented in `tests/sqlx/tests/encrypted_domain/property/README.md`.
- Verify the scalar matrix coverage snapshot: `mise run test:matrix:inventory` (no database required). Committed token-normalized baselines under `tests/sqlx/snapshots/` pin the set of `scalars::<T>::*` test names so a silently dropped/renamed/`#[cfg]`-gated test fails CI's `matrix-coverage` job. The task discovers the present scalar types from the test binary's `--list` and cross-checks them against `cargo run -p eql-codegen -- list-types`, so a catalog type missing its matrix wiring also fails. When you change which matrix tests the macro emits, regenerate and commit the affected baseline in the same change. `tests/sqlx/snapshots/README.md` is the source of truth for which baselines exist and how each is regenerated.

### Build System
- Dependencies are resolved using `-- REQUIRE:` comments in SQL files
- Build outputs to `release/` directory (a single installer + uninstaller, assembled from `src/v3` alone):
  - `cipherstash-encrypt.sql` - The sole installer: the self-contained `eql_v3` surface, globbed from `src/v3` only (no `eql_v2`; installable into a DB with no `eql_v2` present)
  - `cipherstash-encrypt-uninstall.sql` - Matching uninstaller

There are no longer separate Main / Supabase / Protect / v3-only build variants. The combined `eql_v2` build that previously produced multiple artefacts has been removed; the v3 surface now ships as one self-contained installer under the canonical `cipherstash-encrypt.sql` name (`eql-codegen order` walks `src/v3` only). Because the surface owns no `eql_v2` dependency, it is already Supabase / managed-Postgres compatible (functional indexes over extractors, no superuser-only operator classes) without a dedicated subset build. Self-containment — no `-- REQUIRE:` edge pointing outside `src/v3`, no `eql_v2.<symbol>` anywhere in the surface — is enforced at build time by `surface_order`'s `OutsideSurface` error (`crates/eql-codegen/src/ordering.rs`) and CI-gated by `mise run test:self_contained_v3`.

## Project Architecture

This is the **Encrypt Query Language (EQL)** - a PostgreSQL extension for searchable encryption. Key architectural components:

### Core Structure

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cipherstash/stack](https://github.com/cipherstash/stack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
